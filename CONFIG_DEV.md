Fichero config_dev.yml
=======

# Tabla de contenidos
- config.yml
    - [Monolog](#monolog)
        - [Oculta la información de eventos](#oculta-la-información-de-eventos-(si-quieres))
        - [Activar formateador de logs pesonalizado](#activar-formateador-de-logs-pesonalizado)
    - [Mailer](#mailer)
        - [Definir una lista de emails permitidos](#definir-una-lista-de-emails-permitidos)

## Monolog

Este comoponente incluido en Symfony genera los logs de la aplicación, recurso esencial para administrar la plataforma web.

### Oculta la información de eventos (si quieres)

El siguiente bloque evita que se registre la información de eventos haciendo más legible la información del log.

```
monolog:
    handlers:
        main:
            type: stream
            path: '%kernel.logs_dir%/%kernel.environment%.log'
            level: debug
            channels: ['!event']
```

### Activar formateador de logs pesonalizado

Activa el formateador configurado en el fichero [config.yml](CONFIG.md#formato-personalizado-en-los-logs)

```
monolog:
    handlers:
        main:
            type: stream
            path: '%kernel.logs_dir%/%kernel.environment%.log'
            level: debug
            formatter: app.log.formatter
```


## Mailer

### Definir una lista de emails permitidos

Las direcciones de correo que concuerden con la expresión regular serán enviados, incluso en el entorno dev.
```
swiftmailer:
    delivery_address: dev@example.com
    delivery_whitelist: 
        - "/notifications@example.com$/" 
        - "/^admin@*$/""
``` 