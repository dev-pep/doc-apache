# *Rewrite engine*

El módulo ***mod_rewrite*** permite reescribir *URLs* entrantes dinámicamente para que se adapten al sistema interno de *URLs* de nuestro sitio.

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

El patrón es una expresión regular:

- En contexto *server config* o *virtual host* busca coincidencia en el *%-decoded URL-path* de la *request*, es decir, el fragmento justo después del nombre de *host* (y puerto), y justo antes de la *query string* (si la hay).
- En contexto directorio o *htaccess*, solo el fragmento del *%-decoded URL-path* relativo al directorio en el que está definida la regla.

Para acceder a la *URL* completa será necesario acceder a las variables ***%{HTTP_HOST}***, ***%{SERVER_PORT}*** o ***%{QUERY_STRING}*** en una `RewriteCond`.

Una vez se ha realizado una sustitución, las subsiguientes reglas se aplicarán a este nuevo valor sustituido, no a la *URL* inicial. Así, se pueden encadenar sustituciones.

Las sustituciones pueden ser de tres tipos:

- Ruta absoluta al *filesystem*: ```RewriteRule "^/games" "/usr/local/games/web"``` (equivalente a un `Alias`).
- Ruta *web*: ```RewriteRule "^/foo$" "/bar"``` (se produce sustitución).
- *URL* completa: ```RewriteRule "^/product/view$" "http://site2.example.com/seeproduct.html" [R]```. En este caso se indica que el cliente realice una nueva *request* a la *URL* indicada (redirección).

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

En la cadena de sustitución de una `RewriteRule` podemos hacer, como hemos visto, *backreferences* mediante ***\$1***, ***\$2***, etc. Pero también podemos hacerlas de las expresiones regulares de las `RewriteCond` pertinentes usando ***%1***, ***%2***, etc.

```
RewriteCond "%{HTTP_HOST}" "(.*)"
RewriteRule "^/(.*)"       "/sites/%1/$1"
```

En este ejemplo, si la *request* es ***http://example.com/foo/bar***, entonces ***\%1*** contendría ***example.com*** y ***\$1*** sería ***foo/bar***.

### RewriteMap

Esta directiva permite proporcionar una función externa, es decir, programar nuestra propia reescritura.

## Redirección y remapeo

Ejemplos de uso.

### De antiguo a nuevo

Hemos renombrado ***foo.html*** a ***bar.html***. Queremos *backward compatibility*.
