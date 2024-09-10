---
title: Utilizar Slack para Automatizar Procesos Internos
date: 2024-09-10
draft: false

categories: ['Automatización', 'Slack', 'API']
tags: ['Slack', 'API', 'automatización']
toc: true
author: 'Juan Cruz Parise'

featuredImage: 'articles/slack-automatizacion/portada.png'
---

> TL;DR: Automatiza procesos internos con Slack y APIs integradas, agilizando tareas cotidianas mediante notificaciones automáticas y botones interactivos, sin necesidad de cambiar de plataforma.

<!--more-->

En el área de datos, buscamos constantemente formas de mejorar la eficiencia en nuestras tareas diarias. Una de las soluciones que hemos implementado es la automatización de procesos internos utilizando **Slack**, lo que nos ha permitido gestionar ciertas tareas de manera más ágil, como la **actualización de permisos de usuarios** en nuestras herramientas internas.

En este artículo, te contaremos cómo hemos integrado Slack con nuestras APIs para automatizar procesos puntuales, eliminando la necesidad de realizar acciones manuales repetitivas y mejorando el flujo de trabajo en nuestro equipo.

## Slack como herramienta para simplificar tareas internas

Uno de los desafíos que enfrentábamos era la necesidad de actualizar permisos de usuarios en nuestros sistemas cada vez que alguien dejaba de formar parte de la organización. Realizar estos cambios manualmente podía ser lento y susceptible a errores, por lo que decidimos automatizar este proceso específico integrando Slack con nuestras herramientas de administración de usuarios.

Ahora, cuando ocurre un cambio en los permisos de un usuario, **Slack envía una notificación automática** a nuestro canal de trabajo, donde podemos elegir entre dos opciones:

- **Actualizar permisos**: Revocar los permisos del usuario en los sistemas correspondientes.
- **Desestimar**: Si no es necesario realizar cambios, podemos ignorarlos de inmediato.

Este flujo resuelve un problema concreto en nuestro equipo, pero la flexibilidad de Slack nos permitiría automatizar otros tipos de tareas de manera similar, según las necesidades de cada área.

## ¿Cómo funciona la integración?

El proceso que diseñamos es simple desde el punto de vista del usuario, pero es importante mencionar algunos detalles clave que pueden aplicarse en diferentes escenarios. 

### 1: Notificación automática
Cuando un usuario es desvinculado de la organización, se activa una notificación en Slack. Utilizamos los **componentes interactivos** de Slack, que permiten añadir botones en los mensajes enviados al canal.

### 2: Interacción con botones
En el mensaje, añadimos botones interactivos configurados a través de **Interactive Components & Shortcuts** de Slack. Esto se gestiona en el panel de configuración de la app de Slack, en la sección "Interactivity & Shortcuts". Aquí proporcionamos un **URL de solicitud** que apunta a nuestra **API interna**, encargada de procesar las interacciones cuando los usuarios hacen clic en los botones.

![Panel para activar interactividad](/articles/automatizacion-procesos-slack/interactivity-shortcuts.png)

![Mensaje con botones interactivos](/articles/automatizacion-procesos-slack/mensaje-interactivo.png)

### 3: Envío de solicitud a la API
Al hacer clic en cualquiera de los botones (por ejemplo, "Actualizar permisos" o "Desestimar"), Slack envía una **solicitud HTTP POST** a nuestra API interna. Esta solicitud contiene información sobre la acción realizada por el usuario, como el botón seleccionado y los datos necesarios para procesar la solicitud.

### 4: Procesamiento en la API
- Nuestra API recibe la solicitud y valida su autenticidad mediante el **Signing Secret** y el **timestamp** que Slack envía para garantizar que la solicitud es reciente y legítima.
- Dependiendo de la acción seleccionada (actualizar permisos o desestimar), la API ejecuta la operación correspondiente y actualiza los permisos del usuario en los sistemas internos o desestima la acción.

### 5: Actualización en Slack
Una vez completada la operación, el mensaje en Slack se actualiza automáticamente para informar si los permisos fueron actualizados correctamente o si la acción fue desestimada. Todo este proceso ahorra tiempo y minimiza los errores manuales.

Este flujo ha sido útil para nuestra área en un caso específico, pero se puede adaptar a diferentes situaciones y procesos internos que requieran notificaciones y acciones rápidas y seguras.

![Permisos actualizados](/articles/automatizacion-procesos-slack/permisos-actualizados.png)

![Permisos desestimados](/articles/automatizacion-procesos-slack/permisos-desestimados.png)

## ¿Por qué elegimos Slack?

Elegimos Slack porque ya es una herramienta central en nuestra comunicación diaria. Al integrarlo con nuestras APIs internas, logramos **centralizar** la gestión de procesos relevantes sin necesidad de cambiar de plataforma.

Entre las principales ventajas de esta integración específica se encuentran:

- **Simplicidad**: Gestionamos todo directamente desde Slack, sin alternar entre múltiples herramientas.
- **Acciones rápidas**: Los botones interactivos permiten ejecutar cambios importantes con solo un clic.
- **Seguridad**: Las solicitudes enviadas a través de Slack están verificadas mediante sistemas de firmas y timestamps, asegurando que las acciones sean válidas y seguras.

## Un flujo que nos ahorra tiempo

Este proceso de automatización ha reducido significativamente el tiempo que antes invertíamos en la gestión manual de permisos. Lo que antes podía tomar varios minutos ahora se resuelve en cuestión de segundos con un clic, liberando tiempo para que el equipo se enfoque en tareas más estratégicas.

Aunque esta automatización fue diseñada para un caso específico, **es escalable** y podría adaptarse fácilmente para otros flujos de trabajo que requieran acciones rápidas y seguras en Slack, como la ejecución de reportes o la actualización de otros datos clave.

## Conclusión

Integrar Slack en nuestros procesos internos ha sido un paso clave para mejorar la eficiencia del equipo de datos. Al automatizar tareas específicas como la gestión de permisos mediante notificaciones y botones interactivos, hemos simplificado el trabajo diario y mejorado nuestra capacidad de respuesta ante cambios.

**Si tu equipo utiliza Slack regularmente, es una herramienta excelente para explorar la posibilidad de simplificar tareas repetitivas, adaptando el enfoque a los procesos que mejor se ajusten a las necesidades de tu equipo.**

### Links útiles

- [Documentación de Slack API](https://api.slack.com/docs)
- [Interactive Components & Shortcuts](https://api.slack.com/interactivity)
- [Block Kit Builder](https://app.slack.com/block-kit-builder)