# Manual de configuración de servicios de AWS para notificación de generación de Backups
## Objetivo
El objetivo del presente documento es describir el paso a paso de la configuración de los servicios de AWS para enviar una notificación por correo electrónico del estado de la generación de los backups configurados en el servicio de AWS Backup.
## Consideraciones Previas
- En la cuenta origen, cree la política: **auna-eventbridge-procesos-aws-backups-prd-policy.**
- En la cuenta origen, cree el rol: **auna-eventbridge-procesos-aws-backups-prd-role**, incluyendo la política: **auna-eventbridge-procesos-aws-backups-prd-policy.**
- En la cuenta Shared Services, configure el bus de eventos Default de Amazon EventBridge para agregar la política de acceso al rol creado en la cuenta origen.
- En la cuenta origen, cree la regla de Amazon EventBrige: auna-procesos-aws-notificar-backups-prd-rule con destino hacia el bus de eventos (default) de la cuenta destino.

## Arquitectura
- La utilización centralizada de AWS Backup para generar backups en base a los planes de backup permitirá centralizar la notificación del estado de la generación de los backups en la cuenta Shared Services. Los estados controlados son los siguientes: completado, ocurrió un error y cancelado.
- La configuración de las suscripciones para notificación por email, se realiza en el tema de **SNS: ProcesosAWS-NotificarBackups**.  

IMAGEN
## Procedimiento

Crear la política: **eventbridge-procesos-aws-backups-prd-policy**.

- En la cuenta origen, cree la política: **eventbridge-procesos-aws-backups-prd-policy** con los siguientes permisos hacia el bus de eventos (Default) de la cuenta Shared Services:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "events:PutEvents"
            ],
            "Resource": [
                "arn:aws:events:us-east-1:748816464213:event-bus/default"
            ]
        }
    ]
}
```
Crear el rol: **eventbridge-procesos-aws-backups-prd-role**.
- En la cuenta origen, cree el rol: **eventbridge-procesos-aws-backups-prd-role** con la siguiente política de confianza personalizada: 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "events.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
- En la página Agregar Permisos, busque y seleccione la política: **eventbridge-procesos-aws-backups-prd-policy creada.**
- En la página Asignar nombre, revisar y crear, ingrese el nombre del rol:**eventbridge-procesos-aws-backups-prd-role**.

Configurar el bus de eventos Default en Amazon EventBridge de la cuenta Shared Services:
- En la cuenta Shared Services, ingrese al servicio de Amazon EventBridge y ubique el bus de eventos Default, luego en el botón de Acciones, seleccione la opción Administrar permisos.
- En la ventana Administrar permisos, ingrese en la sección AWS de Principal, el ARN del rol creado en la cuenta origen.
Ejemplo: **arn:aws:iam::189902947303:role/eventbridge-procesos-aws-backups-prd-role**.

Crear la regla:**procesos-aws-notificar-backups-prd-rule**.
- En la cuenta origen, cree la regla de Amazon EventBridge: **procesos-aws-notificar-backups-prd-rule**, incluyendo la siguiente descripción: Gestiona el cambio de tipo de estado de la generación de backups para notificar el estado de cada uno.

- En la página Crear patrón de eventos, seleccione en la sección Patrón de eventos, el servicio de AWS Backup, luego seleccione el Tipo de evento Backup Job State Change.
- En la página Seleccionar destinos, seleccione en la sección Tipos de destino Bus de eventos de EventBridge.
- A continuación, seleccione el botón de elección Bus de eventos en una cuenta o región diferente.

- En la sección Bus de eventos como destino ingrese el ARN del bus de eventos Default de la cuenta Shared Services:**arn:aws:events:us-east-1:748816464213:event-bus/default**.
- En la sección Rol de ejecución, seleccione el botón de elección Usar el rol existente, luego seleccione el rol: **eventbridge-procesos-aws-backups-prd-role** de la lista de roles.
- En la página Revisar y crear, verifique las configuraciones realizadas y haga clic en el botón Crear Regla.


