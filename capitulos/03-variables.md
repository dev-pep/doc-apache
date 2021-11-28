# Variables de entorno

Las variables de entorno del sistema operativo están ya establecidas antes del inicio del servidor *HTTP*. En *scripts PHP* son recogidas en el *array* ***\$_ENV***. Por otro lado, *Apache* utiliza una serie de variables que pasa a los *scripts*, los cuales las recogen en ***\$_SERVER***.

En definitiva, las variables con las que trabaja *Apache* no son en realidad variables de entorno del sistema operativo. Cuando nos refiramos a variables de entorno, nos referiremos a estas variables que maneja el servidor.

Para una lista de variables manejadas por el servidor por defecto, véase el apartado de reescritura (***mod_rewrite***).

### *Logging* condicional

***TODO*** #######################################################



## Directivas

### PassEnv

Esta directiva pasa como variable del servidor (***\$_SERVER***) las variables del sistema indicadas (una o más).

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

### SetEnv

Establece una variable de entorno. Si no se indica valor, la establece a *string* vacío. Esto se ejecuta **antes** que la mayor parte de directivas que procesan una *request*, como los mapeos *URI* a archivo. Si deseamos que la variable esté disponible a directivas como las de reescritura, se debería establecer la variable con `SetEnvIf`.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

### SetEnvIf y SetEnvIfNoCase

Esta directiva establece variables de entorno en función de características de la *request*. `SetEnvIfNoCase` es igual que `SetEnvIf`, con la diferencia de que en el primer caso las coincidencias se harán sin tener en cuenta mayúsculas/minúsculas.

El primer argumento es la característica a comprobar, y puede ser una de estas cosas:

- Un *header HTTP*, como por ejemplo ***Host***, ***User-Agent***, ***Referer***, o ***Accept-Language***. Se puede especificar una expresión regular para que coincidan varios *headers*.
- Uno de los siguientes valores (características de la *request*):
    - ***Remote_Host***: nombre del *host* cliente (si está disponible).
    - ***Remote_Addr***: la dirección *IP* del cliente.
    - ***Server_Addr***:  la dirección *IP* del servidor.
    - ***Request_Method***: método *HTTP* (***GET***, ***POST***, etc.).
    - ***Request_Protocol***: nombre/versión del protocolo (***HTTP/0.9***, ***HTTP/1.1***, etc.).
    - ***Request_URI***: la *URI* de la solicitud.
- El nombre de una variable de entorno del servidor. Solo están disponibles las variables que se hayan definido en directivas `SetEnvIf` o `SetEnvIfNoCase` anteriores (es decir, en niveles más externos, o más arriba en el mismo nivel).

El segundo argumento es el valor con el que comparar, y consiste en una expresión regular. Si coincide con el valor del primer argumento, se ejecutarán el resto de argumentos de la directiva (puede haber uno o más), que tendrán una de estas tres formas:

- ***nombre***: nombre de la variable a establecer (se establece con valor 1).
- ***!nombre***: nombre de la variable a eliminar.
- ***nombre=valor***: nombre de la variable a establecer, junto con su valor. Se pueden usar *backreferences* a la expresión regular del segundo argumento: ***\$0*** representa el *string* entero que ha coincidido con el patrón, mientras que ***\$1***, ***\$2***,... se refieren a los grupos de la expresión regular.

Ejemplos:

```
SetEnvIf Request_URI "\.gif$" object_is_image=gif
SetEnvIf Request_URI "\.jpg$" object_is_image=jpg
SetEnvIf Request_URI "\.xbm$" object_is_image=xbm

SetEnvIf Referer www\.mydomain\.example\.com intra_site_referral

SetEnvIf object_is_image xbm XBIT_PROCESSING=1

SetEnvIf Request_URI "\.(.*)$" EXTENSION=$1

SetEnvIf ^TS  ^[a-z]  HAVE_TS
```

Los tres primeros ejemplos comparan características de la solicitud (en este caso la *URI*). El cuarto compara con un *header*. El quinto con una variable (establecida anteriormente). El penúltimo ejemplo compara con la *URI* otra vez, y el último lo hace con todos los *headers HTTP* que empiecen por ***TS***.

A diferencia de `SetEnv`, las variables definidas tendrán efecto para las directivas incluidas en niveles más interiores, o más tarde en el mismo nivel.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

### BrowserMatch y BrowserMatchNoCase

Son casos específicos de `SetEnvIf` y `SetEnvIfNoCase` respectivamente. Equivalen a comparar el contenido del *header* ***User-Agent***.

```
BrowserMatch ^Mozilla var1 var2=valor !var3
```

equivale a:

```
SetenvIf User-Agent ^Mozilla var1 var2=valor !var3
```

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

### UnsetEnv

Elimina una variable de entorno. Esto se ejecuta **antes** que la mayor parte de directivas que procesan una *request*, como los mapeos *URI* a archivo.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.
