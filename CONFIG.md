Fichero config.yml
=======

Existens ficheros específicos para los entornos de [producción](CONFIG_PROD.md) y [desarrollo](CONFIG_DEV.md). 


# Tabla de contenidos
- config.yml
    - [Habilitar profiler en producción](#habilitar-profiler-en-producción)
    - [Doctrine](#doctrine)
        - [Configurar la opción schema_filter](#configurar-la-opción-schema_filter)
    - [Monolog](#monolog) 
        - [Deshabilitar precision de microsegundos](#deshabilitar-precision-de-microsegundos)
    - [Entorno de trabajo](#entorno-de-trabajo)
        - [Abrir archivos en PHPStorm](#abrir-enlaces-de-volcado-de-variables-directamente-en-phpstorm)
    - [Mailers](#mailers)
        - [Mailers instantáneos y pospuestos](#mailers-instantáneos-y-pospuestos)
        
    
## Habilitar profiler en producción

La manera fácil
```
framework: 
    # ... 
    profiler: 
        matcher: 
            path: ^/admin/ 
```

La manera correcta: [enlace 1](https://image.slidesharecdn.com/symfony-tips-and-tricks-141128100207-conversion-gate01/95/symfony-tips-and-tricks-44-638.jpg?cb=1417169555) y [enlace 2](https://image.slidesharecdn.com/symfony-tips-and-tricks-141128100207-conversion-gate01/95/symfony-tips-and-tricks-45-638.jpg?cb=1417169555)


## Doctrine

### Configurar la opción schema_filter

```
doctrine:
    dbal:
        default_connection: default
        # ...
        schema_filter: ~^(?!legacy_)~
```


## Monolog

### Deshabilitar precision de microsegundos

Esta configuración esta recomendada para los sistemas que generen una gran cantidad de logs puesto que mejora el tiempo de
generación de logs. Si se configura, forzamos al logger a reducir la precisión de *datetime* de microsegundos a segundos 
evitando la llamada al *microtime(true)* y su subsiguiente parseo.
 
```
monolog:
    use_microseconds: false
```

## Entorno de trabajo

### Abrir enlaces de volcado de variables directamente en PHPStorm

Los editores soportados son phpstorm*, sublime, textmate, macvim y emacs

*PHPStorm requiere seguir [instrucciones](https://symfony.com/doc/3.4/reference/configuration/framework.html#ide) específicas.
```
framework:
    ide: 'phpstorm://open?file=%%f&line=%%l'
```

## Mailers

**Recomendación**
- Aplicaciones pequeñas o medianas: usar mailer propio es correcto.
- Aplicaciones medianas o grandes: mejor usar un proveedor (AWS, Mailgun, Postmark...)

### Mailers instantáneos y pospuestos

```
swiftmailer:
    default_mailer: delayed
        mailers: 
            instant: 
                # ...
                spool: ~ 
            delayed: 
                # ... 
                spool: 
                    type: file 
                    path: "%kernel.cache_dir%/swiftmailer/spool"
```

Uso de los mailers priorizados: 
- Alta prioridad: registro, recuperación de contraseña,...
- Prioridad normal: notificaciones, newsletters, <s>spam</s>ofertas... 

```
//Alta prioridad
$container->get('swiftmailer.mailer.instant')->...

//Emails normales
$container->get('swiftmailer.mailer')->...
$container->get('swiftmailer.mailer.delayed')->...
```
 