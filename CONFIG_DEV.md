Fichero config_dev.yml
=======

## Mejores logs con Monolog

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
