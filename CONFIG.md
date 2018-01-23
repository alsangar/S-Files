Fichero config.yml
=======

Existens ficheros específicos para los entornos de [producción](CONFIG_PROD.md) y [desarrollo](CONFIG_DEV.md). 


# Tabla de contenidos
- config.yml
    - [Habilitar profiler en producción](#habilitar-profiler-en-producción)
    - []()
    - [Doctrine](#doctrine)
        - [Estrategia de nombrado](#estrategia-de-nombrado)
        - [Configurar la opción schema_filter](#configurar-la-opción-schema_filter)        
    - [Monolog](#monolog) 
        - [Deshabilitar precision de microsegundos](#deshabilitar-precision-de-microsegundos)
        - [Formato personalizado en los logs](#formato-personalizado-en-los-logs)
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


## Alternativa a variables de entorno

1. Crear un fichero de credenciales en el servidor de producción

```
#/etc/credentials/example.com.yml
parameters:
    database_host:          ...
    database_port:          ...
    database_name:          ...
    database_user:          ...
    database_password:      ...

    mailer_user: ...
    mailer_password:        ...

    secret:                 ...
    
    github_client_id:       ...
    github_client_secret:   ...
```

2. Cargar el fichero de credenciales en el fichero de configuración

 ```
 #/app/config/config.yml
imports:
    - { resource: parameters.yml }
    - { resource: security.yml }
    - { resource: services.yml }
    - { resource: '/etc/credentials/example.com.yml',
                  ignore_errors: true }
 ```

3. En la máquina "dev" el fichero no existe pero no se notifca el error por la etiqueta "ignore_errors" indicada. La aplicación usará el fichero parameters.yml habitual.
4. En la máquina "prod" conseguimos los siguiente:
- El fichero de credenciales sobreescribe a *parameters.yml*.
- Los parámetros correctos están disponibles en cualquier lugar.
- Los desarrolladores no ven las opciones de configuración de producción.
- Requiere disciplina para agregar/eliminar opciones.

  
## Doctrine

### Estrategia de nombrado

En las entidades, definir mediante annotation el campo name usando snake_case y el nombre de variable con camelCase

```
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
/**
 * @ORM\Table(name="api_users")
 */
class ApiUsers
{    
    /**
     * @ORM\Column(type="text", name="api_token")
     */
    protected $apiToken;
    
    /**
    * @ORM\Column(type="datetime", name="created_at")
    */
    protected $createdAt;
}
```

Se puede configurar el parámetro de esta manera:

```
doctrine:
    dbal:
        default_connection: default
        # ...
    orm:
        naming_strategy: doctrine.orm.naming_strategy.underscore
```

El resultado es el mapeo automático entre las propiedades de la entidad y los campos de la tabla

<table style="border: 1px solid #ccc">
<tr>
<td>Entidad</td>
<td>BBDD</td>
</tr>
<tr>
<td>ApiUsers</td>
<td>api_users</td>
</tr>
<tr>
<td>apiToken</td>
<td>api_token</td>
</tr>
<tr>
<td>createdAt</td>
<td>created_at</td>
</tr>
</table>


### Configurar la opción schema_filter

```
doctrine:
    dbal:
        default_connection: default
        # ...
        schema_filter: ~^(?!legacy_)~
```

### Configurar las opciones de tabla por defecto

```
default_table_options:
            charset: utf8mb4
            collate: utf8mb4_unicode_ci
            engine: InnoDB
```

Con **utf8mb4** damos soporte a los emojis =)


## Monolog

### Deshabilitar precision de microsegundos

Esta configuración esta recomendada para los sistemas que generen una gran cantidad de logs puesto que mejora el tiempo de
generación de logs. Si se configura, forzamos al logger a reducir la precisión de *datetime* de microsegundos a segundos 
evitando la llamada al *microtime(true)* y su consiguiente parseo.
 
```
monolog:
    use_microseconds: false
```

### Definir canales personalizados

Se pueden definir canales personalizados para registrar datos de forma fácil:

```
monolog:
    channels: ["mi_canal"]
```

Se puede volcar información desde el código fácilmente:

```
$this->get('monolog.logger.mi_canal')->info('....');
```

Incluso se pueden procesar estos mensajes a parte del resto de logs:

```
# app/logs/dev.log
[2018-01-22 17:31:05] mi_canal.INFO: .......[][] 
```

El nombre del canal puede ser un parámetro

```
parameters:
    app_channel: 'mi_canal' 

monolog:
    channels: ["%app_channel%"]
    
services:
    app_logger:
        class: ...
        tags: [{name: monolog.logger, channel: %app_channel% }]
```

### Formato personalizado en los logs
```
services:
    app.log.formatter:
        class: 'Monolog\Formatter\LineFormatter'
        public: false
        arguments:
            - "%%level_name%% [%%datetime%%] [%%channel%%] %%message%%\n %%context%%\n\n"
            - 'H:i:s'
            - false
```
El primer argumento indica el formato de cada linea del log. El segundo es el formato del *placeholder* para *%datetime%*. 
Y el tercero indica si se ignoran los saltos de línea (\n) dentro de los mensajes de log.

Se debe habilitar el formateador personalizado en el fichero de configuración de entorno, por ejemplo: [config_dev.yml](CONFIG_DEV.MD#activar-formateador-de-logs-pesonalizado) 

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
 