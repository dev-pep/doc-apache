# Otros

## Habilitación/deshabilitación de componentes

En el directorio de configuración, existe la carpeta ***mods-available***. En esta carpeta existen archivos de configuración para cada módulo *Apache* disponible. Existen por un lado los archivos con el nombre del módulo en cuestión y la extensión ***.load***, que simplemnte contienen la directiva `LoadModule` que carga dicho módulo. Por otro lado, está el archivo con el mismo nombre y extensión ***.conf*** con configuraciones adicionales para el módulo. No siempre existen ambos archivos para cada módulo.

Sin embargo, estos archivos no son cargados en la configuración de *Apache*. Para agregar un módulo es necesario usar el comando `a2enmod`, pasándole el nombre del módulo como parámetro. Lo que hace esto es simplemente crear en la carpeta ***mods-enabled*** enlaces simbólicos del archivo o archivos de configuración del módulo presentes en ***mods-available***. Los archivos de esta carpeta sí quedan incluidos en la configuración.

Para deshabilitar un módulo, se usa `a2dismod`, que elimina el enlace o enlaces simbólicos del módulo en ***mods-enabled***.

De forma parecida, podemos habilitar o deshabilitar configuraciones específicas disponibles (archivos ***.conf***) en la carpeta ***conf-available***, mediante el comando `a2enconf`, que crea enlaces simbólicos en ***conf-enabled***, y `a2disconf` que los elimina.

Finalemente, se puede hacer lo mismo para *virtual hosts*, con los comandos `a2ensite` y `a2dissite`, relacionados con las carpetas ***sites-available*** y ***sites-enabled***.

Los enlaces simbólicos se cargan por orden alfabético, con lo que, por ejemplo, en la lista de *virtual hosts*, existe un archivo ***000-default.conf*** que se carga el primero, y es el *default virtual host*, usado cuando no coincide ninguno de los nombre de servidor de los *virtual hosts* siguientes.

Tras realizar cualquier cambio en la configuración se debe reiniciar el servidor.

## Directiva DirectoryIndex (mod_dir)

Indica el archivo o archivos por defecto que se usará como índice. Los nombres indicados se comparan como prefijo ('empieza por').

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

**Por defecto:** ***index.html***.

## Directiva FallbackResource (mod_dir)

Indica el *handler* (*script*) a utilizar cuando una *URI* no se puede mapear a ningún archivo. En ese caso, se proporciona como único argumento la *URI* de dicho *script*.

Si el argumento es ***disabled*** (por defecto), el servidor retornará simplemente el código 404 (*not found*). Se debe indicar este valor si deseamos desactivar esta configuración en un directorio concreto que puede heredar esta característica:

```
<Directory "/var/www/htdocs/blog">
    FallbackResource /blog/notfound-404.php
</Directory>

<Directory "/var/www/htdocs/blog/images">
    FallbackResource disabled
</Directory>
```

El *script* obtiene la *URI* original (no la nueva) al leer la variable de servidor ***REQUEST_URI***.

Al retornar el recurso *fallback*, el código de retorno del servidor es 200 (*OK*). Si no deseamos que sea así, deberemos usar, en su lugar, la directiva `ErrorDocument`.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

**Por defecto:** ***disabled***.

## Directiva \<IfVersion> (mod_version)

Comprueba el número de versión de *Apache* con una versión indicada. Se puede negar la expresión e incluir operadores de comparación.

```
<IfVersion >= 2.3>
    # directivas
</IfVersion>
```

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.
