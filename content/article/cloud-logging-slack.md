---
title: "Enviar Notificaciones de Google Cloud Logging a Slack"
date: 2024-01-30T17:33:49Z
draft: false

categories: ['gcp']
tags: ['cloud logging', 'slack']
toc: true
author: 'Juan Cruz Parise'

featuredImage: 'articles/cloud-logging-slack/portada.png'
---

> TL;DR: Configura notificaciones automáticas de Google Cloud Logging a Slack para recibir alertas específicas de tus aplicaciones en GCP.

<!--more-->

Cuando gestionas proyectos en Google Cloud Platform (GCP), contar con un control efectivo de lo que sucede en tu proyecto es esencial. En este artículo, exploraremos el paso a paso de cómo poder notificar alertas de logs de Cloud Logging directamente a un canal de Slack.

## ¿Qué es Google Cloud Logging?

Es una herramienta que permite administrar los logs en tiempo real, con búsqueda, análisis y almacenamiento.

Para ingresar a Google Cloud Logging se debe:
1. Ingresar a [Google Cloud](https://console.cloud.google.com/)
2. Debemos buscar "Logging"
3. Una vez que ingresamos veremos una consola de SQL Log y los logs en tiempo real

### Logs Explorer
La consola tiene una [sintaxis particular](https://cloud.google.com/logging/docs/view/building-queries?hl=es-419) y permite filtrar los logs de una manera sencilla ya sea con palabras o [expresiones regulares](https://cloud.google.com/blog/products/management-tools/cloud-logging-gets-regular-expression-support).

![Logs explorer](/articles/cloud-logging-slack/log-explorer.png).

## Pasos previos para crear una alerta

### Google Cloud Monitoring en Slack

En el apartado de aplicaciones dentro de Slack debemos buscar **Google Cloud Monitoring** y agregarla a nuestro espacio de trabajo. Luego, se debe ingresar el siguiente comando en el canal para invitar a la app:

![Invitar Google Cloud Monitoring a Slack](/articles/cloud-logging-slack/invitar-cloud-monitoring.png)


### Permisos necesarios

Los permisos de GCP necesarios para este procedimiento son:
- [Error Reporting User](https://cloud.google.com/iam/docs/understanding-roles#errorreporting.admin).
- [Monitoring Editor](https://cloud.google.com/iam/docs/understanding-roles#monitoring.editor).

### Agregar canal de Slack a Cloud Logging

Para agregar un canal, se debe hacer clic en **Manage Notification Channel** y abrirá una nueva ventana donde se debe agregar un nuevo canal de Slack

![Botón para crear alerta](/articles/cloud-logging-slack/notification-channel.png)

Luego se debe permitir que Google Cloud Monitoring tenga acceso a Slack

![Aprobacion](/articles/cloud-logging-slack/aprobacion.png)

Una vez permitido, se debe ingresar el nombre del canal que queremos agregar.

## Parámetros para crear una alerta

Una vez detectado en el panel de Log explorer los logs que queremos notificar, procedemos a crear la alerta. Para ello, simplemente hacemos clic en el botón 'Crear Alerta' ubicado en el mismo panel.

![Botón para crear alerta](/articles/cloud-logging-slack/button-create-alert.png)

Luego debemos completar un formulario, el cual contiene los siguientes parámetros:

**Sección 1: Detalles de la alerta**
- `Alert Policy Name`: Nombre que aparecerá en la alerta en Slack.
- `Policy Severity Lever`: Aquí se selecciona la gravedad. Las opciones son "No Severity", "Critical", "Error" y "Warning"
- `Documentation`: Es la documentación que queremos incluir con el mensaje, podría ser cómo resolver la alerta o puede referenciar a una wiki, también se le puede [dar formato a los mensajes](https://cloud.google.com/monitoring/alerts/doc-variables?_ga=2.148459511.-194555348.1625487700) e incluir campos específicos sobre la alerta.

**Sección 2: Elija logs para incluir en la alerta**

- `Define log entries to alert on`: Son los filtros que va a tener en cuenta Cloud Logging para encontrar la alerta. Para este ejemplo buscaremos logs donde se avise que se superó la memoria limite de una app corriendo en CloudRun:
  ```yaml
  resource.type = "cloud_run_revision"
  severity=ERROR
  textPayload: "Memory limit of"
  ```

**Sección 3: Establecer la frecuencia de notificación y la duración del cierre automático**

- `Set notification frequency and autoclose duration`: Es la frecuencia con la cual se envían las notificaciones, actualmente el tiempo mínimo son 5 minutos.
- `Incident autoclose duration:` Es el período de tiempo que transcurre antes de que un incidente se cierre automáticamente.

**Sección 4: ¿A quién se debe notificar?**

- `Who should be notified?:` Aquí se marca el canal de Slack a donde enviaremos la notificación.

![Paso a paso](/articles/cloud-logging-slack/parametros.png)

## Resultado
Cuando se active la alerta se enviará un mensaje de Slack con el contenido (link a el log pre-filtrado en Log Explorer, documentación, nombre de la alerta, etc), algo parecido al siguiente ejemplo. 🚀

![Slack Message](/articles/cloud-logging-slack/resultado.png)
