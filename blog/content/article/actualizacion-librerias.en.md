---
title: 'Tracking Library Updates with renv and Github Actions'
date: 2024-01-23T21:51:23Z
draft: false
featuredImage: 'articles/actualizacion-librerias/Portada.png'
categories: ['R']
tags: ['github actions', 'slack', 'renv']
toc: true
author: 'Juan P. Dugo'
resizeImages: false
---

Tracking new updates for libraries in a project is not always an easy task, especially as dependencies and the number of organization repositories increase. To automate the tedious process of manually checking each project for updates and staying up-to-date with the latest changes, you can use GitHub Actions along with Slack to periodically check for updates. It's ideal to configure a channel with all team members to keep everyone informed.

## Requirements

- [R](https://www.r-project.org/)
- [renv](https://rstudio.github.io/renv/articles/renv.html): Allows you to create reproducible environments for your projects, where R dependencies used in the project are saved in a `renv.lock` file, which is then used as a reference by the package to install the exact necessary versions. Additionally, it allows explicit tracking of libraries used in the project, either from a DESCRIPTION file or implicitly by crawling through the code to find dependencies and capturing them semi-automatically. We opt for explicit tracking.
- [Github Actions](https://docs.github.com/en/actions): Enables workflow automation through events in the repository.
- [slackr](https://github.com/mrkaye97/slackr): We'll use the R library that accesses the Slack API to send messages. To connect, credentials need to be generated, and various methods are available. We recommend consulting the [vignettes](https://github.com/mrkaye97/slackr#vignettes) for details.

## Project's General Structure

```bash
.
├── .github
│   ├── .gitignore
│   └── workflows
│       ├── check_updates.yaml
├── .gitignore
├── library_updates.Rproj
├── DESCRIPTION
├── renv
│   ├── activate.R
│   ├── settings.json
│   └── staging
├── renv.lock
```

## Steps to Follow

- Create a repository on GitHub and clone it locally.
- Within the project directory, run `renv::init()` to initialize renv.
- Create a description file using `usethis::use_description()`.
- To add a new library to the description, use `usethis::use_package("package_name")` and then `renv::snapshot()`.
- Finally, add a YAML file for the workflow. You can also use `usethis::use_github_actions()`.

``` yaml
on:
  push:
    branches: [main, master]
  workflow_dispatch:
    inputs:
      update_lock:
        default: false
        type: boolean

  schedule:
  - cron: '0 9-23/3 * * *'

name: Check Updates

jobs:
  Updates:
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
      SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      SLACK_TOKEN: ${{ secrets.SLACK_TOKEN }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      UPDATE_LOCK: ${{ inputs.update_lock }}
    steps:
      - run: |
          sudo apt-get install -y libsodium-dev

      - uses: actions/checkout@v3
      - uses: r-lib/actions/setup-r@v2
      - uses: r-lib/actions/setup-renv@v2
          
      - name: check_for_updates
        run: |
          slackr::slackr_setup(
            channel = Sys.getenv("SLACK_CHANNEL"), 
            token   = Sys.getenv("SLACK_TOKEN"),
            incoming_webhook_url = Sys.getenv("SLACK_WEBHOOK_URL")
          )
          
          sink("logs")
          updates <- renv::update(check = as.logical(Sys.getenv("UPDATE_LOCK")))
          slackr::slackr(readLines("logs"))

        shell: Rscript {0}

      - name: Auto Commit
        if: ${{ inputs.update_lock }}
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Auto Snapshot
          file_pattern: 'renv.lock'

```

### Workflow Explanation:

- The workflow is triggered on a push to the `main` or `master` branches, manually with `workflow_dispatch`, and also runs automatically every 3 hours (`cron: '0 9-23/3 * * *'`).
- Necessary dependencies are installed using `sudo apt-get install -y libsodium-dev`. In cases where libraries are compiled from source, system libraries are required. As an example, `libsodium-dev` is used, which is required for compiling the `sodium` package.
- R and `renv` are configured using the actions `r-lib/actions/setup-r@v2` and `r-lib/actions/setup-renv@v2`. The `renv` action installs the libraries contained in `renv.lock` and additionally sets up a cache to avoid installing from scratch each time the workflow runs.
- The `check_for_updates` step performs the following tasks:
  - Configures `slackr` to send messages to a specific Slack channel.
  - Logs the output to a log file (`sink("logs")`).
  - Updates the libraries using `renv::update()`.
  - Sends the update logs to Slack using `slackr::slackr(readLines("logs"))`.
- The `Auto Commit` step performs an automatic commit if the `update_lock` option was activated during the previous run.
- Configuration of Secrets and Environment Variables:
  - The workflow uses secrets and environment variables to ensure security and personalized configuration. These include the GitHub token (`GITHUB_TOKEN`), Slack channel (`SLACK_CHANNEL`), Slack token (`SLACK_TOKEN`), and Slack webhook URL (`SLACK_WEBHOOK_URL`).
