# *Rewrite engine*

El módulo ***mod_rewrite*** permite reescribir *URLs* entrantes dinámicamente. Se pueden usar para mapear una *URL* entrante a una localización de nuestro *filesystem*, o para mapear una *URL* a otra *URL* sobre la marcha.

Se puede aplicar un número ilimitado de reglas a una *URL*, y cada regla puede tener asociado un número ilimitado de condiciones

Las reglas de reescritura se pueden aplicar a cualquier parte de la *URL*, y circunscribirse a un contexto global del servidor, dentro de un *virtual host*, un directorio o un archivo ***.htaccess***.

El módulo realiza *logging* útil para depuración en los niveles ***trace1*** a ***trace8***. No se deberían usar estos niveles en producción, ya que ralentizarían sobremanera el servidor.

## Directiva RewriteEngine

Esta directiva sirve para activar o desactivar la reescritura de *URLs*. Se puede establecer en ***On*** u ***Off***.

**Contexto**: todos.

**Por defecto:** ***Off***.

## Expresiones regulares

Las reescrituras utilizan las acostumbradas expresiones regulares tipo *Perl*. A grandes rasgos, acepta los patrones ***.*** (cualquier carácter), ***+*** (repetición 1 o más, *greedy*), ***+?*** (repetición 1 o más, *non-greedy* o *lazy*), ***\**** (repetición 0 o más, *greedy*), ***\*?*** (repetición 0 o más, *lazy*), ***?*** (anterior elemento opcional), ***\\*** (escape del siguiente carácter), ***^*** (principio), ***\$*** (final), ***( )*** (grupos), ***[ ]*** conjuntos de caracteres y ***[^ ]*** (conjuntos negados).

Adicionalmente, prefijar un signo de admiración (***!***) delante de una expresión regular sirve para negarla.

Para *backreferences* se usa ***\$N*** o ***%N***, como se verá más adelante.

## Reglas de reescritura (RewriteRule)

Esta directiva define una regla de reescritura y recibe 3 argumentos: *pattern* (patrón: qué *URLs* se verán afectadas), *substitution* (sustitución: cómo se reescribirá la *URL*) y opcionalmente *flags* (opciones de procesamiento).

En el caso de reglas consecutivas, la salida de cada una será la entrada de la siguiente.

### Patrón (*pattern*)

El **patrón*** es una expresión regular:

- En contexto *server config* o *virtual host* el *pattern* es el *%-decoded URL-path* de la *request*. Por ejemplo, si la petición es ***http://servidor.com:8000/url/path/pagina.html?parm1=11&parm2=12***, la *pattern* será ***/url/path/pagina.html***. Como puede verse, se elimina, por delante, el protocolo, servidor y puerto, sin incluir la barra tras el puerto, que queda **siempre** como primer carácter de la *pattern*. También se elimina la *query string*, incluyendo el interrogante inicial (***?***) que no formará parte del *pattern*.
- En contexto directorio o *htaccess*, solo recibe el fragmento del *%-decoded URL-path* relativo al directorio en el que está definida la regla. Supongamos otra vez que la *request* es ***http://servidor.com:8000/url/path/pagina.html?parm1=11&parm2=12***, y que nuestro *document root* es ***/var/www/sitio***. Si estamos dentro de una sección `<Directory /var/www/sitio/url>`, entonces *pattern* será ***path/pagina.html***. Obsérvese que en este caso, el patrón no empieza por barra (***/***).

Para acceder a la *URL* completa será necesario acceder a las variables ***%{HTTP_HOST}***, ***%{SERVER_PORT}*** o ***%{QUERY_STRING}*** en una `RewriteCond`.

Una vez se ha realizado una sustitución, las subsiguientes reglas se aplicarán a este nuevo valor sustituido, no a la *URL* inicial. Así, se pueden encadenar sustituciones.

### Cadena de sustitución (*substitution*)

El argumento *substitution* sustituye por entero la *URL* de entrada, no solamente la parte coincidente (para sustituir partes disponemos de las *backreferences*). Este *string* de sustitución puede tener varias formas:

En primer lugar, puede ser una ***URL*** **completa:**:

```
RewriteRule "^/product/view$" "http://site2.example.com/seeproduct.html" [R]
```

En este caso se trata de una redirección (el navegador volverá a hacer una solicitud).

En segundo lugar, puede ser simplemente **un guión** (***-***). En este caso no se produce ninguna sustitución. Este tipo de reglas se utiliza cuando simplemente necesitamos realizar una acción a través de los *flags*.

Y en tercer lugar, puede ser una **ruta** a un archivo de nuestro *filesystem*, o una ruta *URL*. En este caso, hay dos escenarios distintos:

- **Desde contexto servidor o** ***virtual host***, el *string* de sustitución deberá empezar **siempre** con una barra (***/***). La reescritura resultante es siempre una referencia a un archivo local, ya no es una *URI*, por lo que si deseamos utilizar directivas como `Alias` o `Redirect`, que trabajan con una *URI* de entrada, deberemos usar el *flag* ***PT*** para que el resultado siga siendo una *URI* que se pueda pasar (*pass through*) a otro *handler* que mapee *URI* a archivo local. En todo caso, suponiendo que no se haya usado el *flag* ***PT***, pueden darse dos casos:
    - El primer fragmento de la ruta especificada en la cadena de sustitución **existe** en el sistema de archivos, en cuyo caso, la cadena representará una **referencia absoluta a un archivo local** (equivale a hacer un `Alias`).
    - El primer fragmento de la ruta **no existe** en el sistema de archivos. En ese caso, la cadena representa una **referencia a un archivo local**, aunque esta vez se le prefija la ruta del *document root* (excepto si indicamos el *flag* ***PT***).
- **Desde contexto directorio o** ***htaccess***, el resultado será siempre una *URI* (el *flag* ***PT*** va implícito siempre), con lo que dicho resultado se podrá procesar con directivas como `Aias` o `Redirect`. En todo caso, hay dos posibilidades:
    - El *string* de sustitución empieza con una barra (***/***). En este caso, el *URL-path* indicado es relativo al *document root*.
    - El *string* no empieza con una barra (***/***), en cuyo caso la *URI* indicada es relativa al directorio desde el que estamos trabajando (o a lo indicado por `RewriteBase`, como se verá más adelante).

Si se usa el *flag* ***PT*** en una regla en contexto servidor o *virtual host*, la reescritura se realizará como en contexto directorio (relativo siempre a *document root*).

Es posible realizar *backreferences* en la *substitution string*:

```
RewriteRule "^/product/(.*)/view$" "/var/web/productdb/$1"
```

En el caso de sustituciones en contexto directorio (y *htaccess*), hay que tener en cuenta:

- Para que funcione, a parte de `RewriteEngine On` deberemos activar el seguimiento de enlaces simbólicos con `Options FollowSymLinks`.
- Si el directorio no está en el árbol del *document root*, se debe configurar adecuadamente el prefijo que se añadirá a las sustituciones **relativas al directorio** con la directiva `RewriteBase`.
- Si se desea tener acceso al *URL-path* completo, necesitaremos la variable ***%{REQUEST_URI}*** en una `RewriteCond`.

### *Flags*

Veremos aquí algunos *flags* útiles.

#### C | chain

Sirve para indicar una serie de reglas consecutivas que forman una unidad (para fragmentar una regla muy compleja en varias). Si una de ellas falla, el resto se salta.

#### CO | cookie

Si la regla se ejecuta, se crea una *cookie*. Es formato es:

```
CO=NAME:VALUE:DOMAIN:lifetime:path:secure:httponly:samesite
```

Los tres primeros campos son obligatorios:

- ***NAME*** es el nombre de la *cookie*.
- ***VALUE*** es el valor dado a la misma.
- ***DOMAIN*** indica en qué ámbito es activa la *cookie*, y puede ser un nombre de *host* (un nombre al que corresponde una *IP*, como ***www.ejemplo.com***), o un dominio o subdominio (***.ejemplo.com***, ***.users.ejemplo.com***, etc.).

Los cinco siguiente son opcionales (se pueden dejar en blanco):

- ***lifetime*** es el tiempo de vida en minutos. Por defecto es 0 (vive solo durante la sesión actual).
- ***path*** es la ruta dentro del sitio (*URI*) desde la cual se enviará la *cookie* al navegador. Por defecto es ***/***, indicando el sitio completo (la ruta se entiende recursivamente).
- ***secure*** indica, si está activado (***secure***, ***true*** o ***1***), que solo será procesada en conexiones *HTTPS*.
- ***httponly*** indica, si está activado (***HttpOnly***, ***true*** o ***1***), que la *cookie* tendrá activado el *flag* ***HttpOnly***, lo cual hace (si el navegador lo soporta) que sea inaccesible desde *Javascript*.
- ***samesite*** establece el atributo ***SameSite*** de la *cookie*. Puede ser ***Lax***, ***Strict*** o ***None***.

Si los últimos campos se dejan en blanco, se pueden obviar los dos puntos pertinentes (***:***).

#### END

Al ejecutar la presente regla, no se ejecuta ninguna más, aunque se entre en otros contextos que contengan reglas de reescritura.

#### E | env

Si la regla se ejecuta, establece el valor de una variable de entorno. Puede tener estas formas:

- `E=VAR:valor` establece la variable ***VAR*** al valor especificado.
- `E=VAR` establece la variable ***VAR*** a un valor vacío.
- `E=!VAR` *unsets* la variable ***VAR***.

#### F | forbidden

Directamente produce una respuesta 403 (*forbidden*).

#### G | gone

Directamente produce una respuesta 410 (*gone*).

#### L | last

En cuanto la presente regla se ejecute (si coincide), la reescritura se dará por finalizada **en el contexto actual**, ignorando cualquier otra regla en él. Tras esto, la *URL* se entrega nuevamente al parseador de *URLs* para su posterior tratamiento. Si esto sucede en una sección directorio o *htaccess*, tras encontrar el *flag* ***L*** (o tras ejecutar la última regla de la sección), se ignorarán el resto de reglas de la sección, pero tras la posterior gestión del *parser*, es posible que la *URL* vuelva a encontrarse con la misma sección (por ejemplo tras una redirección, interna o externa), se volverán a invocar las mismas reglas desde el principio, una y otra vez.

Si deseamos que esto no suceda, se usa el *flag* ***END***.

#### NC | nocase

No tiene en cuenta mayúsculas/minúsculas a la hora de hacer *matching*.

#### NE | noescape

Los caracteres especiales en un *string* de sustitución suelen convertirse a su correspondiente código hexadecimal. Para que esto no suceda se usa este *flag*.

#### PT | passthrough

Las reescrituras producen por defecto una ruta de archivo del *filesystem*. Para evitarlo, y que el resultado sea considerado a su vez una *URL*, utilizaremos este *flag*. De este modo, la *URL* producida puede usarse en directivas cuya entrada es una *URL*, como `Alias` o `Redirect`.

Este *flag* implica también el *flag* ***L*** automáticamente.

> Este *flag* está implícito en las reglas dentro de un contexto de directorio o *htaccess*.

#### QSA

La *query string* es retenida tal cual está, y es añadida al final del *string* de sustitución.

#### R | redirect

Reescribe la *URL* a modo de redirección, de forma análoga a la directiva `Redirect`. Si se proporciona una *URL* completa, se hará la redirección allí. De lo contrario, se usará el protocolo, nombre de servidor y puerto actuales. Al especificar la *URI* no es necesario prefijarle barra (***/***).

Por defecto realiza una redirección 302, pero se le puede indicar cualquier otro código (`R=305`), ***temp***, ***permanent*** o ***seeother***.

Con frecuencia se usa junto con el *flag* ***L*** (`[R,L]`) para que termine la ejecución del conjunto de reglas actual.

## Directiva RewriteBase

Como se ha comentado, la directiva añade el prefijo indicado (debe ser una *URL-path*, es decir, empezará siempre con una barra) al *string* de sustitución, siempre y cuando:

- Estemos en contexto directorio o *htaccess*, y
- El *string* de sustitución sea relativo, es decir, no empiece con barra (***/***).

**Contexto:** directorio y *htaccess*.

**Por defecto:** ***None***, lo cual implica que el prefijo resultante es el directorio actual.

## Condiciones de reescritura (RewriteCond)

Se puede indicar una o más condiciones con `RewriteCond` para restringir los tipos de *request* a los que se aplicará la `RewriteRule` inmediatamente siguiente a las condiciones. Si hay varias condiciones, se deben cumplir todas ellas para que se aplique la regla. Si tras las condiciones hay más de una regla, solo la primera de ellas es afectada por dichas condiciones. Sin embargo, si la primera regla se ejecuta, su salida será la entrada de la segunda regla.

### Cadena *test*

El primer argumento de la condición es la expresión (*test string*) a considerar, la cual puede contener variables. El segundo argumento (*pattern string*) es una expresión regular que se comparará con el primero. El tercer argumento (opcional) son los *flags* que modifican la evaluación de la comparación.

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

### *Pattern string*

A la hora de comparar la expresión con la *pattern string*, es posible prefijar operadores numéricos ***\<***, ***>***, ***\<=***, ***>=*** y ***=*** que forman parte de dicha *pattern string*. En ese caso los *strings* no se tratarán como expresiones regulares, sino que se hace una comparación a nivel de *string* plano.

Por otro lado, la *pattern string* también puede tener puede tener los prefijos ***-eq***, ***-ge***, ***-gt***, ***-le***, ***-lt***, ***-ne*** para comparaciones numéricas.

La *pattern string* puede consistir simplemente en:

- ***-d*** la *test string* es un directorio existente.
- ***-f*** la *test string* es un archivo existente.
- ***-s*** la *test string* es un archivo existente con tamaño distinto de cero.

En todo caso, siempre es posible prefijar ***!*** **al principio de todo** para negar la comparación.

### *Flags*

Estos son algunos *flags* útiles para usar en condiciones:

- ***NC*** o ***nocase*** sirve para hacer una comparación *case-insensitive*.
- ***OR*** o ***ornext*** indica que se tiene que aplicar *OR* con la siguiente condición, en lugar de *AND*.

## Variables y *backreferences*

Tanto las reglas como las condiciones pueden usar variables de entorno (en cualquiera de sus *strings*). Estas se indican con el formato ***%{NOMBRE_VAR}***. Además, pueden contener *backreferences* a la regla más reciente (***\$1***, ***\$2***,...) o a la condición más reciente (***%1***, ***%2***,...). Si la *backreference* se produce en el segundo argumento, se referirá al primer argumento, si procede. De lo contrario, a la regla o condición más reciente.

Lista de algunas variables útiles: ***HTTP_USER_AGENT***, ***REMOTE_ADDR***, ***REMOTE_HOST*** (será igual que ***REMOTE_ADDR*** si el servidor está configurado para no hacer *IP address lookups*, que es lo habitual), ***REMOTE_PORT***, ***REMOTE_USER***, ***HTTP_REFERER*** (*URL* de origen), ***HTTP_COOKIE***, ***HTTP_HOST***, ***REQUEST_METHOD***, ***SCRIPT_FILENAME***, ***PATH_INFO***, ***QUERY_STRING***, ***AUTH_TYPE*** (***BASIC*** o ***DIGEST***), ***DOCUMENT_ROOT***, ***SERVER_ADMIN***, ***SERVER_NAME***, ***SERVER_ADDR***, ***SERVER_PORT***, ***SERVER_PROTOCOL***, ***SERVER_SOFTWARE***, ***TIME_YEAR***, ***TIME_MON***, ***TIME_DAY***, ***TIME_HOUR***, ***TIME_MIN***, ***TIME_SEC***, ***TIME_WDAY***, ***TIME***, ***REQUEST_URI***, ***HTTPS***.

También es posible usar el contenido de una cabecera *HTTP* mediante el formato ***%{HTTP:nombre_cabecera}***.

## Directiva RewriteMap

Esta directiva permite proporcionar una función externa, es decir, programar nuestra propia reescritura.

## Directiva RewriteOptions

Aquí podemos indicar algunas opciones para el motor de reescritura. Veamos algunos valores útiles (el argumento puede ser **uno** de ellos):

- ***Inherit***: se hereda la configuración de reescritura del contexto padre. Un *VH* hereda la configuración global, un directorio la del directorio que lo contiene. La configuración heredada se copia **después** de la local, independientemente de la posición que ocupe `Rewrite Options` en el contexto actual.
- ***InheritBefore***: igual que la anterior, pero la configuración heredada se copia **antes** de la local.
- ***InheritDown*** fuerza que todos los hijos hereden la configuración local (como ***Inherit***).
- ***InheritDownBefore*** fuerza que todos los hijos hereden la configuración local (como ***InheritBefore***).
- ***IgnoreInherit*** ignora la herencia de un padre que haya indicado ***InheritDown*** o ***InheritDownBefore***.
- ***MergeBase*** copia la directiva `RewriteBase` a todos los descendientes que no la definan.

## Ejemplos de uso

Supongamos que hemos renombrado ***foo.html*** a ***bar.html***. Queremos *backward compatibility*, con lo que la antigua *URL* debería retornar también la página:

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

> Exceptuando las redirecciones, las reescrituras suelen ser internas.

Si se ha movido el archivo a un servidor (dominio) distinto, veamos tres formas equivalentes de redirigir:

```
RewriteRule   "^/docs/(.+)"  "http://new.example.com/docs/$1"  [R,L]

RedirectMatch "^/docs/(.*)" "http://new.example.com/docs/$1"

Redirect "/docs/" "http://new.example.com/docs/"
```

Supongamos ahora que nuestro recurso as accesible mediante ***http://example.com/vegetables.php?carrots*** pero queremos que tenga la forma más atractiva ***http://example.com/vegetables/carrots***. Se haría así:

```
RewriteRule ^/vegetables/(.*) /vegetables.php?$1 [PT]
```
