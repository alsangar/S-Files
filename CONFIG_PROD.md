Fichero config_prod.yml
=======

# Tabla de contenidos
- config.yml
    - [Monolog](#monolog)
        - [Mejores email de logs](#mejores-email-de-logs)
            - [Activar envío de emails](#activar-envío-de-emails)
            - [Enviar registros relativos a errores 500](#enviar-registros-relativos-a-errores-500)
            - [Enviar registros relativos a errores 400 excluyendo 404](#enviar-registros-relativos-a-errores-400-excluyendo-404)
        - [Logs organizados](#logs-organizados)
            - [Usar diferentes ficheros de regitro para cada canal](#usar-diferentes-ficheros-de-regitro-para-cada-canal)
            - [Habilitar registro de autenticaciones](#habilitar-registro-de-autenticaciones)
        - [Excluir errores 404](#excluir-errores-404)

## Monolog

Este comoponente genera los logs de la aplicación y es esencial para administrar la plataforma web. Symfony incluye Monolog para esta tarea.
La configuración inicial es correcta para el entorno de desarrollo pero es insuficiente para el de producción. Con estos cambios conseguimos dos objetivos:
* Enviar los errores al administrador web por email(los registros de nivel "error")
* Registrar todas las autenticaciones, puesto que estan definidas a nivel "info" y estas no se registran por defecto.


### Mejores email de logs

Fuente: [documentación Symfony](http://symfony.com/doc/3.4/logging/monolog_email.html)

#### Activar envío de emails

```
monolog:
    handlers:
        swift:
            type:               swift_mailer
            from_email:         FQN@foo.com
            to_email:           webmaster@company.com
            subject:            "OOps"
            level:              debug
```

#### Enviar registros relativos a errores 500

El parámetro *action_level* indica el tipo de error a enviar: los 500 están definidos como "critical" mientras que los 400 estan definidos como "error".

```
monolog:
    handlers:
        mail:
            type:               fingers_crossed
            action_level:       critical
            handler:            buffered
        buffered:
            type:               buffer
            handler:            swift
```

#### Enviar registros relativos a errores 400 excluyendo 404

```
monolog:
    handlers:
        mail:
            type:               fingers_crossed
            action_level:       error
            handler:            deduplicated
            excluded_404s:
                        - ^/
        deduplicated:
                    type:    deduplication                    
                    time: 10
                    handler: swift
```

El manejador *deduplicated* simplemente guarda todos los mensajes para una solicitud y luego los pasa al manejador anidado de una sola vez, pero solo si los registros son únicos durante un período de tiempo dado (60 segundos por defecto). Si los registros son duplicados, simplemente se descartan. Agregar este controlador reduce la cantidad de notificaciones a un nivel manejable, especialmente en escenarios de fallo crítica. Puede ajustar el período de tiempo usando la opción "time".

### Logs organizados

#### Usar diferentes ficheros de regitro para cada canal

```
monolog:
    handlers:
        security:
            level:    debug
            type:     stream
            path:     '%kernel.logs_dir%/security-%kernel.envirnment%.log'
            channels: [security]
        main:
            level:    debug
            type:     stream
            path:     '%kernel.logs_dir%/%kernel.envirnment%.log'
            channels: ['!event', '!security']
```
Por ejemplo, en *main* se registra información de todos los canales excepto de event y security.

Fuente: [documentación Symfony](http://symfony.com/doc/3.4/logging/channels_handlers.html)

### Habilitar registro de autenticaciones

```
monolog:
    handlers:
        login:
            type:               stream
            path:               "%kernel.logs_dir%/auth.log"
            level:              info
            channels:           security
```

### Excluir errores 404

```
monolog:
    handlers:
        main:
            # ...
            type: fingers_crossed
            handler: ...
            excluded_404s:
                - ^/phpmyadmin
```

Fuente: [documentación Symfony](http://symfony.com/doc/3.4/logging/monolog_regex_based_excludes.html)

