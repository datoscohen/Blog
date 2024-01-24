---
title: 'Trackear Actualizaciones de Librerías con renv y Github Actions'
date: 2024-01-23T21:51:23Z
draft: false

categories: ['R']
tags: ['github actions', 'slack', 'renv']
toc: true
author: 'Juan P. Dugo'

featuredImage: 'articles/actualizacion-librerias/Portada.png'
---

> TL;DR: Automatiza la búsqueda de actualizaciones de librerías en proyectos de R con renv, slackr y GitHub Actions.

<!--more-->

Realizar un seguimiento de las nuevas actualizaciones disponibles para las librerías de un proyecto no suele ser una tarea sencilla, sobretodo a medida que aumentan las dependencias y la cantidad de repositorios de la organización. Para automatizar este proceso tedioso de revisar manualmente cada Proyecto y estar siempre el dia con las ultimas novedades, se puede utilizar github actions junto con slack para realizar un chequeo de actualizaciones periódicamente, lo ideal es configurar un canal con todos los miembros del grupo de trabajo para mantener a todos los integrantes informados.

## Requisitos

- [R](https://www.r-project.org/)
- [renv](https://rstudio.github.io/renv/articles/renv.html): Permite crear ambientes reproducibles para tus proyectos, donde las dependencias de R utilizadas en el proyecto se guardan en un archivo `renv.lock` que luego se utiliza como referencia por el paquete para poder instalar las versiones exactas necesarias. Adicionalmente, permite llevar un registro **explícito** de las librerias utilizadas en el proyecto, lo cual se hace desde un archivo DESCRIPTION, o también se puede optar por llevar un registro implícito, en el cual renv se encarga de hacer un crawl por todo el codigo buscando dependencias y las captura de manera semi-automática. Nosotros optamos por un registro explícito.
- [Github Actions](https://docs.github.com/es/actions): Permite automatizar flujos de trabajo mediante eventos en el repositorio.
- [slackr](https://github.com/mrkaye97/slackr): Vamos a utilizar la librería de R que permite acceder a la api de slack para enviar mensajes. Para poder conectarse se necesita generar las credenciales, esto se puede realizar de multiples manera y recomendamos que se consulten los [vignettes](https://github.com/mrkaye97/slackr#vignettes) disponibles.

### Estructura general del proyecto

``` bash
.
├── .github
│   ├── .gitignore
│   └── workflows
│       ├── check_updates.yaml
├── .gitignore
├── actualizacion_librerias.Rproj
├── DESCRIPTION
├── renv
│   ├── activate.R
│   ├── settings.json
│   └── staging
├── renv.lock
```

## Pasos a seguir

- Crear un repositorio en github y clonarlo localmente. En caso de haber utilizado Rstudio van a tener un archivo `.Rproj`.
- Dentro del directorio del proyecto, ejecutar `renv::init()` para inicializar renv
- Crear un description file, se puede usar `usethis::use_description()`
- Para agregar una nueva librería al description, se puede utilizar `usethis::use_package("package_name")` y luego `renv::snapshot()`
- Finalmente agregamos un yaml con el workflow. Para esto también se puede utilizar usethis::use_github_actions()

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
          
          update_lock <- as.logical(Sys.getenv("UPDATE_LOCK"))

          sink("logs")
          renv::update(check = !update_lock)
          slackr::slackr(readLines("logs"))
          if (isTRUE(update_lock)) {renv::snapshot()}

        shell: Rscript {0}

      - name: Auto Commit
        if : ${{ inputs.update_lock }}
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Auto Snapshot
          file_pattern: 'renv.lock'
```

### Explicación del flujo de trabajo:

- El flujo de trabajo se activa cuando hay un push a las ramas principales `main` o `master`, manualmente con `workflow_dispatch` y también se ejecuta automáticamente cada 3 horas (`cron: '0 9-23/3 * * *'`).
- Se instalan las dependencias necesarias usando `sudo apt-get install -y libsodium-dev`. En los casos donde las librerías se compilan desde código fuente, es necesario contar con librerías de sistema. A modo de ejemplo puse `libsodium-dev` que se utiliza para compilar el paquete `sodium`.
- Se configuran R y `renv` utilizando las acciones `r-lib/actions/setup-r@v2` y `r-lib/actions/setup-renv@v2`. El action de `renv` se encarga de instalar las librerías contenidas en el `renv.lock` y adicionalmente configura un caché para no tener que instalar desde cero cada vez que corre el workflow.
- El paso `check_for_updates` realiza las siguientes tareas:
    1. Configura `slackr` para enviar mensajes a un canal de Slack específico.
    2. Registra la salida en un archivo de registros (`sink("logs")`).
    3. Actualiza las librerías utilizando `renv::update()`.
    4. Envía los registros de actualización a Slack utilizando `slackr::slackr(readLines("logs"))`.
- El paso `Auto Commit` realiza un commit automático si se ha activado la opción `update_lock` durante la ejecución anterior.
- Configuración de Secretos y Variables de Entorno:
- El flujo de trabajo utiliza secretos y variables de entorno para garantizar la seguridad y la configuración personalizada. Estos incluyen el token de GitHub (`GITHUB_TOKEN`), el canal de Slack (`SLACK_CHANNEL`), el token de Slack (`SLACK_TOKEN`), y la URL del webhook de Slack (`SLACK_WEBHOOK_URL`).

Ahora si, ¡hemos finalizado los preparativos y estamos listos para utilizar nuestro action! 🚀

## Resultados

Se enviará un mensaje de slack con el siguiente formato cuando existan librerías disponibles.
![Slack Message](/articles/actualizacion-librerias/check_updates.png)

En los próximos artículos, vamos a estar hablando sobre cómo configurar slackr y enviar mensajes personalizados utilizando la sintaxis propia de Slack.