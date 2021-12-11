# *Logging*

Los mecanismos de *logging* son útiles para comprobar la actividad del servidor, detectar problemas o depurar errores.

## *Log* de errores

La directiva `ErrorLog` define la ubicación y nombre del principal archivo de *log* del servidor (o *virtual host*): el *log* de errores. Se especifica en el primer parámetro. Si no es una ruta absoluta, se considera relativa a ***ServerRoot***.

El formato de los mensajes de *log* queda definido con la directiva `ErrorLogFormat`, a la que se pasa un *string* de formato, con la definición de los campos deseados. Añadiendo un token ***%L*** al *string* de formato del *log* de error y al del *log* de accesos, se incluirá un ID que permitirá correlacionar entradas en ambos.

Si no se invoca la directiva `ErrorLogFormat`, *Apache* le da un formato por defecto.

Por otro lado, la directiva `LogLevel` indica el nivel mínimo de los mensajes de *log*. Es posible especificar el nivel a nivel general, o por módulos.

```
LogLevel info rewrite:trace5
```

En este ejemplo se registrarán los mensajes de importancia ***info*** o superior, pero los mensajes producidos por el módulo ***mod_rewrite*** se registrarán si tienen importancia ***trace5*** o superior. Algunos módulos (como el mencionado) producen numerosos mensajes de la categoría ***trace*** para facilitar la depuración de errores.

Para indicar un módulo se puede escribir su nombre de archivo fuente (***mod_ssl.c***), el identificador del módulo (***ssl_module***) o su identificador sin el sufijo ***\_module*** (***ssl***).

También es posible definir un nivel distinto a nivel de directorio (con la directiva `<Directory>`).

Estos son los niveles admitidos, de mayor a menor importancia: ***emerg***, ***alert***, ***crit***, ***error***, ***warn***, ***notice***, ***info***, ***debug***, ***trace1***, ***trace2***, ***trace3***, ***trace4***, ***trace5***, ***trace6***, ***trace7*** y ***trace8***.

## *Log* de accesos

El servidor almacena todos los accesos en el *log* de accesos. Este está controlado por la directiva `CustomLog`.

La directiva `LogFormat` sirve para definir el formato de los mensajes en este *log*. En este caso, el primer parámetro de la directiva es el *string* de formato, mientras que el segundo es un alias, o identificador de ese *string*. La directiva `CustomLog` define el nombre y la ruta de un archivo de *log* (igual que `ErrorLog`), así como un formato para este. Este formato puede indicarse mediante un *string* de formato o un identificador creado con `LogFormat`.

Se pueden definir varios *logs* de accesos con varias directivas `CustomLog`.

Tanto la directiva `CustomLog` como `LogFormat` pertenecen al módulo ***mod_log_config***. Ambas tienen contexto *server config* y *virtual host*.

## Rotación de *logs*

Un *log* puede ocupar 1 MB o más cada 10.000 peticiones, por lo que periódicamente hay que borrar o mover (comprimidos) estos los archivos. Sin embargo, el servidor debe ser reiniciado tras mover o eliminar los *logs* antiguos, para que el servidor abra archivos nuevos. Otra opción es utilizar *piped logs* hacia aplicaciones que reciban estos datos en lugar de archivos (se hace prefijando el carácter *pipe* al ejecutable de tal aplicación), y de este modo no hay que reiniciar el servidor.

## Servidores virtuales

Cada servidor virtual puede enviar sus mensajes de *log* a distintos archivos. En cambio, si no define su propia configuración de *logs*, la heredará del contexto del servidor principal.

## Directivas

### CustomLog

Esta directiva actúa sobre el *log* de accesos al servidor.

El primer argumento es el archivo de *log* (relativo al *server root*).

El segundo argumento es el formato de la entrada de *log*. Puede ser el nombre de un formato definido mediante una directiva **previa** `LogFormat`, o puede ser un *string* de formato directamente.

El tercer argumento, opcional, indica una condición que debe cumplirse para que se produzca esa entrada el el *log*. Esta condición puede ser una expresión booleana, o la existencia o ausencia de una variable de entorno del servidor (definida con anterioridad o no). Este tercer argumento tiene la forma ***env=nombrevar*** que precisa la existencia de la variable ***nombrevar***, o ***env=!nombrevar***, que hace lo propio con la ausencia de tal variable.

```
SetEnvIf Request_URI \.gif$ gif-image
LogFormat "%h %l %u %t \"%r\" %>s %b" common
CustomLog "logs/access_log_nogif" common env=!gif-image
CustomLog "logs/access_log_gif" common env=gif-image
```

Este ejemplo define un formato con el nombre ***common***, y registra el acceso en un archivo de *log* u otro dependiendo de si la *request* es de un archivo ***.gif*** o no.

**Contexto:** *server config* y *virtual host*.

### ErrorLog

Define el archivo de *log* como parámetro. Véase apartado de *logging*.

**Contexto:** *server config* y *virtual host*.

**Por defecto:** ***logs/error_log*** (Unix), ***logs/error.log*** (Windows y Mac).

### LogFormat

Existen dos formas de esta directiva. La primera solo tiene un argumento, el cual es un *string* de formato, o un nombre definido en una directiva **previa** `LogFormat`. Este tipo de directiva tiene efecto sobre la directiva `TransferLog`.

La segunda forma acepta dos argumentos, siendo el primero un *string* de formato y el segundo un nombre al que se asociará a ese *string*.

**Contexto:** *server config* y *virtual host*.

**Por defecto:** ***\"%h %l %u %t \\"%r\\" %>s %b\"***

### LogLevel

Define el nivel de *log* mínimo que se registrará en el *log* de errores. También se puede definir en entorno directorio.

**Contexto:** *server config*, *virtual host* y directorio.

**Por defecto:** ***warn***.

### TransferLog

Esta directiva tiene el mismo uso y efecto que `CustomLog`, con la diferencia que solo acepta un argumento (el primero), que indica el archivo de *log*. Como formato se usará el indicado en la última directiva previa `CustomLog` con un solo argumento. Si no hay tal directiva previa, se usará el valor por defecto de esa.

Esta directiva no acepta argumento condicional.

**Contexto:** *server config* y *virtual host*.
