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
