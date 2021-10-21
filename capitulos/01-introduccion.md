# Servidor *HTTP* Apache

Para estos apuntes se asume que se dispone de un servidor *Apache 2* instalado en un sistema operativo *Ubuntu Linux*.

Para controlar el funcionamiento del servidor, disponemos del comando `systemctl`. Para operaciones básicas, disponemos de los comandos `service` y `/etc/init.d/apache2`.

Inicio del servidor:

```
service apache2 start
/etc/init.d/apache2 start
```

Los dos comandos son equivalentes.

A parte de ***start***, también están disponibles órdenes como ***stop*** (parada), ***restart*** (reinicio), ***status*** (estado) o ***reload*** (recarga).

Si *Apache* está sirviendo en puertos estándar (inferior a 1024), las operaciones de arranque y parada precisan de privilegios de administrador.

La *URL* que le llega al servidor consta de protocolo, nombre de servidor, *URI* (ruta de la *URL*) y *query string* opcional. El recurso indicado por la *URI* puede ser un *handler Apache*, un archivo, o un programa (*PHP* por ejemplo). Con esta *request*, el servidor retorna una *response*.

La misma *IP* puede corresponderse con varios *host names* (*virtual hosts*). A parte, se pueden asociar varias *IP* a un servidor físico.

Para resolver nombres de servidor de forma privada, se usa el archivo ***hosts***. En *Linux* suele ser ***/etc/hosts***, y en *Windows* ***C:\\Windows\\system32\\drivers\\etc\\hosts***. Para asociar un nombre arbitrario a la interfaz local de red:

```
127.0.0.1   localhost ejemplo.com otroejemplo.com
```

## Configuración

La configuración del servidor se halla en el directorio ***/etc/apache2***. Esta localización puede ser distinta en otros sistemas operativos (por defecto, el código fuente de *Apache 2* establece el directorio ***/usr/local/apache2/conf***).

En el archivo ***envvars*** se definen algunas variables que definen cosas como directorios, o el usuario que utiliza *Apache* (por defecto ***www-data***).

El archivo principal de configuración es ***apache2.conf***, y está formado por directivas de configuración. Esta configuración se puede fragmentar en varios archivos, con la ayuda de la directiva `Include`. Es posible indicar otras configuraciones (directivas) directamente en los directorios de contenido, mediante el archivo ***.htaccess***.

El directorio del contenido raíz del sitio se indica con la directiva `DocumentRoot`. A parte de archivos, *Apache* dispone de varios *handlers* que generan contenido.

Por otro lado, la localización del *log* se indica mediante la directiva `ErrorLog`.
