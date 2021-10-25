# *Rewrite engine*

El módulo ***mod_rewrite*** permite reescribir *URLs* entrantes dinámicamente. Se pueden usar para mapear una *URL* entrante a una localización de nuestro *filesystem*, o para mapear una *URL* a otra *URL* sobre la marcha antes de ser procesada.

Se puede aplicar un número ilimitado de reglas a una *URL*, y cada regla puede tener asociado un número ilimitado de condiciones

Las reglas de reescritura se pueden aplicar a cualquier parte de la *URL*, y circunscribirse a un contexto global del servidor, dentro de un *virtual host*, un directorio o un archivo ***.htaccess***.

## Introducción

### Directiva RewriteEngine

Esta directiva sirve para activar o desactivar la reescritura de *URLs*. Se puede establecer en ***On*** u ***Off***.

**Contexto**: todos.

**Por defecto:** ***Off***.

### Expresiones regulares

Las reescrituras utilizan las acostumbradas expresiones regulares tipo *Perl*. A grandes rasgos, acepta los patrones ***.*** (cualquier carácter), ***+*** (repetición 1 o más, *greedy*), ***+?*** (repetición 1 o más, *non-greedy* o *lazy*), ***\**** (repetición 0 o más, *greedy*), ***\*?*** (repetición 0 o más, *lazy*), ***?*** (anterior elemento opcional), ***\\*** (escape del siguiente carácter), ***^*** (principio), ***\$*** (final), ***( )*** (grupos), ***[ ]*** conjuntos de caracteres y ***[^ ]*** (conjuntos negados).

Adicionalmente, prefijar un signo de admiración (***!***) delante de una expresión regular sirve para negarla.

Para *backreferences* se usa ***\$N*** o ***%N***, como se verá más adelante.

### Reglas de reescritura (RewriteRule)

Esta directiva define una regla de reescritura y recibe 3 argumentos: *pattern* (patrón: qué *URLs* se verán afectadas), *substitution* (sustitución: cómo se reescribirá la *URL*) y opcionalmente *flags*.

El **patrón*** es una expresión regular:

- En contexto *server config* o *virtual host* el *pattern* es el *%-decoded URL-path* de la *request*, es decir, el fragmento justo después del nombre de *host* (y puerto), y justo antes de la *query string* (si la hay). Por ejemplo, si la petición es ***http://servidor.com:8000/url/path/pagina.html?parm1=11&parm2=12***, la *pattern* será ***/url/path/pagina.html***. Como puede verse, se elimina, por delante, el protocolo, servidor y puerto, sin incluir la barra tras el puerto, que queda **siempre** como primer carácter de la *pattern*. También se elimina la *query string*, incluyendo el interrogante inicial (***?***) que no formará parte del *pattern*.
- En contexto directorio o *htaccess*, solo recibe el fragmento del *%-decoded URL-path* relativo al directorio en el que está definida la regla. Supongamos otra vez que la *request* es ***http://servidor.com:8000/url/path/pagina.html?parm1=11&parm2=12***, y que nuestro *document root* es ***/var/www/sitio***. Si estamos dentro de una sección `<Directory /var/www/sitio/url>`, entonces *pattern* será ***path/pagina.html***. Obsérvese que en este caso, el patrón no empieza por barra (***/***).

Para acceder a la *URL* completa será necesario acceder a las variables ***%{HTTP_HOST}***, ***%{SERVER_PORT}*** o ***%{QUERY_STRING}*** en una `RewriteCond`.

Una vez se ha realizado una sustitución, las subsiguientes reglas se aplicarán a este nuevo valor sustituido, no a la *URL* inicial. Así, se pueden encadenar sustituciones.




substitution en dir vs. server contexts??????????????????
cuándo la substition empieza o no con barra???????????

El argumento *substitution* sustituye por entero a la *URL* de entrada, no solamente a la parte coincidente¿?¿? (para eso existen las *backreferences*). Esta cadena de sustitución puede ser un *URL* completa. Por ejemplo:

```
RewriteRule "^/product/view$" "http://site2.example.com/seeproduct.html" [R]
```

En este caso se trata de una redirección (el navegador volverá a hacer una solicitud).

Otra opción es indicar, en lugar de una *URL*, una *URI*. En un contexto *server config* o *virtual host*, deberá empezar por barra (***/***), pues indicará . En este caso, se considerará que



- Ruta al *filesystem*: ```RewriteRule "^/games" "/usr/local/games/web"``` (equivalente a un `Alias`).
- Ruta *web*: ```RewriteRule "^/foo$" "/bar"```.







Para realizar *backreferences*:

```
RewriteRule "^/product/(.*)/view$" "/var/web/productdb/$1"
```

En cuanto a los *flags*, sirven para modificar el comportamiento de una regla. Se verán más adelante.

#### Contexto directorio (y *htaccess*)

En el caso de este contexto, hay que tener en cuenta:

- Para que funcione, a parte de `RewriteEngine On` deberemos activar el seguimiento de enlaces simbólicos con `Options FollowSymLinks`.
- Se debe configurar adecuadamente el prefijo que se añadirá a estas sustituciones relativas (directiva `RewriteBase`).
- Si se desea tener acceso al *URL-path* completo, necesitaremos la variable ***%{REQUEST_URI}*** en una `RewriteCond`.
- El prefijo eliminado contiene siempre una barra final, con lo que el *string* con el que compara la regla nunca empieza por ***/*** (al contrario que en contexto servidor o *virtual host*).
- ***[TODO ------------ RewriteOptions Inherit merging]***

### Condiciones de reescritura (RewriteCond)

Se puede indicar una o más condiciones con `RewriteCond` para restringir los tipos de *request* a los que se aplicará la próxima `RewriteRule`. Si hay varias condiciones, se deben cumplir todas ellas para que se aplique la regla.

El primer argumento indica la variable que habrá que considerar (una característica de la *request*). El segundo argumento es una expresión regular que se comparará con el primero. El tercer argumento (opcional) son los *flags* que modifican la evaluación de la comparación.

```
RewriteCond "%{QUERY_STRING}" "hack"
RewriteCond "%{HTTP_COOKIE}"  !go
RewriteRule "."               "-"   [F]
```

En este ejemplo, se rechazan todas las *URLs* que contengan ***hack***, excepto si existe una *cookie* con el valor ***go***.

En la cadena de sustitución de una `RewriteRule` podemos hacer, como hemos visto, *backreferences* mediante ***\$1***, ***\$2***, etc. Pero también podemos hacerlas de las expresión regular de la última `RewriteCond` *matched* (si la hay) usando ***%1***, ***%2***, etc.

```
RewriteCond "%{HTTP_HOST}" "(.*)"
RewriteRule "^/(.*)"       "/sites/%1/$1"
```

En este ejemplo, si la *request* es ***http://example.com/foo/bar***, entonces ***\%1*** contendría ***example.com*** y ***\$1*** sería ***foo/bar***.

### RewriteMap

Esta directiva permite proporcionar una función externa, es decir, programar nuestra propia reescritura.

## Redirección y remapeo

Ejemplos de uso.

### De antiguo a nuevo (*internal/external rewrite*)

Hemos renombrado ***foo.html*** a ***bar.html***. Queremos *backward compatibility*, con lo que la antigua *URL* debería retornar también la página:

```
RewriteRule "^/foo\.html$" "/bar.html" [PT]
```

En este caso, la petición ***http://servidor.com/foo.html*** se cambiará **internamente** a ***http://servidor.com/bar.html***. Se trata de un *internal rewrite*, es decir, se hará internamente, sin informar al navegador de que se trata de una *URL* nueva. Por lo tanto, en la barra de dirección del navegador seguirá apareciendo la antigua *URL*. El flag ***PT*** hace que la reescritura se considere una nueva *URL*, y no una ruta de archivo. Si nos da igual, podemos prescindir de este *flag*.

En cambio, si queremos que se produzca un *external rewrite*, es decir, que se informe al navegador de la nueva *URL* que debe solicitar (es decir, una redirección en toda regla), se debe haría así:

```
RewriteRule "^/foo\.html$" "/bar.html" [R]
```

Lo cual equivale a:

```
Redirect "/foo.html" "/bar.html"
```




















## *Flags*

### END

Al ejecutar la presente regla, no se ejecuta ninguna más, aunque se entre en otros contextos que contengan reglas de reescritura.

### L | last

En cuanto la presente regla se ejecute (si coincide), la reescritura se dará por finalizada, ignorando cualquier otra regla. Sin embargo, si posteriormente se entra a una sección `<Directory>`, o un archivo ***.htaccess*** que contienen reglas de reescritura (por ejemplo tras una redirección), estas se ejecutarán. Si no deseamos que se ejecute ninguna reescritura más, se usa el *flag* ***END***.

### PT | passthrough

Las reescrituras producen por defecto una ruta de archivo del *filesystem*. Para evitarlo, y que el resultado sea considerado a su vez una *URL*, utilizaremos este *flag*. De este modo, la *URL* producida puede usarse en directivas cuya entrada es una *URL*, como `Alias` o `Redirect`.

Este *flag* implica también el *flag* ***L*** automáticamente.

> Este *flag* está implícito en las reglas dentro de un contexto de directorio o *htaccess*.

### R | redirect

Reescribe la *URL* a modo de redirección, de forma análoga a la directiva `Redirect`. Si se proporciona una *URL* completa, se hará la redirección allí. De lo contrario, se usará el protocolo, nombre de servidor y puerto actuales. Al especificar la *URI* no es necesario prefijarle barra (***/***).

Por defecto realiza una redirección 302, pero se le puede indicar cualquier otro código (`R=305`), ***temp***, ***permanent*** o ***seeother***.

Con frecuencia se usa junto con el *flag* ***L*** (`[R,L]`) para que termine la ejecución del conjunto de reglas actual.
