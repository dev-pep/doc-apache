# *Rewrite engine*

El módulo ***mod_rewrite*** permite reescribir *URLs* entrantes dinámicamente. Se pueden usar para mapear una *URL* entrante a una localización de nuestro *filesystem*, o para mapear una *URL* a otra *URL* sobre la marcha.

Se puede aplicar un número ilimitado de reglas a una *URL*, y cada regla puede tener asociado un número ilimitado de condiciones

Las reglas de reescritura se pueden aplicar a cualquier parte de la *URL*, y circunscribirse a un contexto global del servidor, dentro de un *virtual host*, un directorio o un archivo ***.htaccess***.

El módulo realiza *logging* útil para depuración en los niveles ***trace1*** a ***trace8***. No se deberían usar estos niveles en producción, ya que ralentizarían sobremanera el servidor.

Las directivas, por defecto, no se heredan en niveles inferiores. Para hacer que se hereden, véase la directiva `RewriteOptions`. Sin embargo, las reglas en un archivo ***.htaccess*** actuarán sobre el directorio actual y sobre todos sus descendientes, hasta el descendiente que tenga directivas del motor de reescrituras. En tal caso, esas directivas serán las que afectarán en tal directorio e inferiores (hasta encontrar un directorio con tales directivas).

Por lo tanto, para que no heredar mecanismos de reescritura, es suficiente indicar `RewriteEngine On` (o `RewriteEngine Off`) sin más reglas. Los directorios inferiores tampoco heredarán nada al respecto.

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
- **Desde contexto directorio o** ***htaccess***, el resultado será siempre una *URI* (el *flag* ***PT*** va implícito siempre), con lo que dicho resultado se podrá procesar con directivas como `Alias` o `Redirect`. En todo caso, hay dos posibilidades:
    - El *string* de sustitución empieza con una barra (***/***). En este caso, el *URL-path* indicado es **absoluto**, lo que en la práctica significa que es relativo al raíz del servidor (*document root*).
    - El *string* no empieza con una barra (***/***), en cuyo caso la *URI* indicada es **relativa**, es decir, se refiere a una ruta relativa al directorio desde el que estamos trabajando (véase la directiva `RewriteBase` para más detalles).

Si se usa el *flag* ***PT*** en una regla en contexto servidor o *virtual host*, la reescritura se realizará como en contexto directorio (relativo siempre a *document root*).

Hay que tener en cuenta una cosa importante: en un contexto de directorio o *htaccess*, el patrón es el relativo al directorio actual, al igual que un *string* de sustitución **relativo**. Si esta configuración actúa en un directorio inferior, seguirán siendo relativos al directorio donde están definidas las directivas, no en ese directorio inferior.

Es posible realizar *backreferences* en la *substitution string*:

```
RewriteRule "^/product/(.*)/view$" "/var/web/productdb/$1"
```

En el caso de sustituciones en contexto directorio (y *htaccess*), hay que tener en cuenta:

- Para que funcione, a parte de `RewriteEngine On` deberemos activar el seguimiento de enlaces simbólicos con `Options FollowSymLinks`.
- Si el directorio no está en el árbol del *document root*, se debe configurar adecuadamente el prefijo que se añadirá a las sustituciones **relativas al directorio** con la directiva `RewriteBase`.
- Si se desea tener acceso al *URL-path* completo, necesitaremos la variable ***%{REQUEST_URI}*** en una `RewriteCond`.

### *Flags*

Veremos aquí algunos *flags* útiles. Es importante recalcar que a la hora de especificar varios de ellos, deben ir separados por comas, y **no se deben insertar espacios**.

#### C | chain

Sirve para indicar una serie de reglas consecutivas que forman una unidad (para fragmentar una regla muy compleja en varias). Si una de ellas falla, el resto se salta.

#### CO | cookie

Si la regla se ejecuta, se crea una *cookie*. Es formato es:

```
CO=NAME:VALUE:DOMAIN:lifetime:path:secure:httponly:samesite
```

Si es necesario especificar el carácter dos puntos (***:***) en alguno de los *strings*, se puede usar punto y coma (***;***) en su lugar, precediendo el nombre de la *cookie* con un punto y coma:

```
CO=;NAME;VALUE;DOMAIN;lifetime;path;secure;httponly;samesite
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
- ***samesite*** establece el atributo ***SameSite*** de la *cookie*, el cual sirve para prevenir ataques *cross-site request forgery* (*CSRF*). Los valores de este atributo pueden ser ***Strict***, ***Lax*** y ***None***.

Veamos el atributo ***samesite*** con más detalle. Si este no está definido, se entenderá ***None***. En este caso la *cookie* se enviará a todos los sitios necesarios para mostrar la página. Veamos un ejemplo:

Supongamos que accedemos a una página de un sitio ***A*** que muestra una imagen que reside en otro sitio ***B***, y que el sitio ***B*** necesita una *cookie* concreta para retornar (mostrar) esa imagen. En este caso, hacemos una petición al sitio ***A***, pero enviamos a dicho sitio, no solo las *cookies* que en su momento creó y nos envió dicho sitio, sino también las *cookies* del sitio ***B*** (llamadas *third-party cookies*), que serán del tipo *samesite* ***None*** en este ejemplo, para que se pueda mostrar toda la página, incluyendo la imagen del sitio ***B***. De esta forma, el sitio ***A*** recibe sus propias *cookies* más las *cookies* del sitio ***B***. Cuando el sitio ***A*** necesita recuperar la imagen del sitio ***B***, realiza la *request*, y le envía las *cookies* necesarias, las cuales posee también porque se las hemos enviado nosotros.

En cambio, supongamos que las *cookies* que tenemos del sitio ***B*** son de tipo ***Strict***. En ese caso, al hacer la petición desde el cliente, solo enviaremos las *cookies* del sitio ***A***. Este es el uso de ***Strict***: solo se envían desde el cliente hacia el servidor las *cookies* que se han originado en ese servidor.

Un punto intermedio es ***Lax***, que permite enviar la mayoría de *cookies* a sitios que no las originaron, siempre y cuando se trate de una petición *GET* de nivel superior, es decir dentro del mismo dominio. En el modo ***Strict***, dos subdominios del mismo dominio se consideran sitios distintos.

Los navegadores exigen que por lo menos se establezca el atributo ***secure***, o que ***samesite*** no sea ***None***.

Si los últimos campos se dejan en blanco, se pueden obviar los dos puntos pertinentes (***:***).

#### END

Al ejecutar una regla con el *flag* ***END***, se dará por terminado el proceso de reescritura, aunque esté en contexto directorio o *htacces* (ver explicación de ***L***). Sin embargo, esto solo es así para redirecciones internas, ya que si se produce una redirección externa, la *request* se vuelve a procesar desde el principio.

#### E | env

Si la regla se ejecuta, establece el valor de una variable de entorno del servidor. Puede tener estas formas:

- `E=VAR:valor` establece la variable ***VAR*** al valor especificado.
- `E=VAR` establece la variable ***VAR*** a un valor vacío.
- `E=!VAR` *unsets* la variable ***VAR***.

Estas variables son variables del servidor (*server variables*) válidas solo en la sesión actual, no se trata de variables de entorno del sistema. Para acceder a ellas desde *PHP* se debe usar ***\$_SERVER***.

#### F | forbidden

Directamente produce una respuesta 403 (*forbidden*).

#### G | gone

Directamente produce una respuesta 410 (*gone*).

#### L | last

Es importante saber que las reglas de reescritura **dentro de un contexto directorio o** ***htaccess***, al terminar de procesarse, se pasan al *URL parser* para seguir su tratamiento. En ocasiones es posible volvamos a entrar en ese contexto directorio o *htaccess*. Esto sucede cuando una regla produce una redirección (interna o externa), en cuyo momento se reinicia todo el proceso de la *request* nuevamente.

Si una regla con el *flag* ***L*** se ejecuta **fuera** de los contextos mencionados, será la última en hacerlo, de la forma esperada.

Pero si es dentro de uno de estos contextos, no se ejecutarán más reglas de dicho contexto, y se pasará la *URL* reescrita al *URL parser* que volverá a empezar si la *URL* se ha reescrito de tal modo que se debe hacer una nueva *request* (redirección interna o externa), pudiendo producirse un bucle.

Si deseamos que esto no suceda, en un contexto directorio o *htaccess* se usa el *flag* ***END***, que dará por terminada la reescritura inmediatamente, sin pasarla al *URL parser*, es decir, dando por terminado el proceso de reescritura (siempre que no haya una redirección externa).

Fuera de los contextos directorio y *htaccess*, ***L*** y ***END*** son equivalentes.

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

Con frecuencia se usa junto con el *flag* ***L*** (`[R,L]`) para que termine la ejecución del conjunto de reglas actual, puesto que si no, la *URL* se pasa a la siguiente regla (la redirección se produce después de haberse procesado todas las reglas).

## Directiva RewriteBase

Esta directiva se utiliza solamente en el caso de que especifiquemos un *string* de sustitución relativo. Como sabemos, esto solo es posible en entorno directorio o *htaccess*, cuando dicho *string* no empieza por barra (***/***). En este caso, se prefijará la ruta absoluta del directorio actual, respecto al *document root*. Esto suele funcionar. Sin embargo, cuando la petición hace referencia a un alias, ya no se mapea correctamente, y en ese caso es necesario definir `RewriteBase`.

Por ejemplo, supongamos un servidor (***servidor.com***) con *document root* en ***/var/www/html*** tiene esta configuración:

```
<Directory /var/www/html/info>
    RewriteEngine On
    RewriteRule "^index\.html$" "welcome.html"
</Directory>
```

Entonces la petición ***http://servidor.com/info/index.html*** reescribirá internamente a ***http://servidor.com/info/welcome.html*** de forma correcta, pues el módulo de reescritura interpreta correctamente que el directorio ***/var/www/html/info*** corresponde a la *URI* ***/info*** (relativa al *document root*).

Pero qué pasa si hay un *alias*:

```
Alias /info /opt/varios/info

<Directory /opt/varios/info>
    RewriteEngine On
    RewriteRule "^index\.html$" "welcome.html"
</Directory>
```

En este caso, el módulo no puede interpretar la localización de ***/opt/varios/info*** relativa al *document root*, con lo que interpretará esa ruta de *filesystem* como una *URI* en sí, y reescribirá ***http://servidor.com/info/index.html*** como ***http://servidor.com/opt/varios/info/welcome.html***, lo cual es una *URL* incorrecta. Para ello, usaremos `RewriteBase` para indicarle al motor de reescritura qué es lo que debe prefijar a la ruta relativa:

```
Alias /info /opt/varios/info

<Directory /opt/varios/info>
    RewriteEngine On
    RewriteBase /info/
    RewriteRule "^index\.html$"  "welcome.html"
</Directory>
```

Ahora ***http://servidor.com/info/index.html*** ya se reescribe a ***http://servidor.com/info/welcome.html*** correctamente.

Este problema se da también en las redirecciones. Supongamos que el mismo servidor contiene en su *document root* un archivo ***.htaccess*** con esta directiva:

```
RewriteRule ^bar\.html$ foo.html [R,L]
```

Al redirigir, nuevamente tendremos que la petición ***http://servidor.com/bar.html*** redirige a ***http://servidor.com/var/www/html/foo.html***, confundiendo la ruta absoluta de la carpeta donde está el *document root* con una *URI*. Así, deberemos indicar la base de reescritura para esta directiva:

```
RewriteBase /
RewriteRule ^bar\.html$ foo.html [R,L]
```

Así, ***http://servidor.com/bar.html*** redirigirá a ***http://servidor.com/foo.html***.

Por otro lado, supongamos que existe un subdirectorio ***info*** dentro de ***/var/www/html***. Supongamos que en ***/var/www/html*** tenemos el archivo ***.htaccess*** indicado anteriormente, con la base de reescritura correcta. Como esas directiva afectarán también al directorio ***info***, la petición ***http://servidor.com/info/bar.html*** redirigirá también a ***http://servidor.com/foo.html***, cuando lo que quisiéramos en nuestro ejemplo sería una redirección a ***http://servidor.com/info/foo.html***. En este caso, el directorio hijo deberá definir una nueva base de reescritura:

```
RewriteBase /info/
RewriteRule ^bar\.html$ foo.html [R,L]
```

Obsérvese que dado que el *string* de `RewriteBase` se concatena a la ruta relativa para formar una *URI* completa, tal *string* debe empezar y terminar en barra.

El uso de `RewriteBase` no sería necesario en el caso que las rutas absolutas del sistema de archivos se correspondieran con una *URI* válida. Esto no tiene mucho sentido, ya que sería en el caso de tener un *document root* en el directorio raíz del *filesystem*.

En definitiva, `RewriteBase` se utiliza para transformar una *URI* relativa en una *URI* absoluta, prefijándole lo indicado.

**Contexto:** directorio y *htaccess*.

**Por defecto:** ***None***, lo cual implica el directorio actual.

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
- ***OR*** o ***ornext*** indica que se tiene que aplicar la operación booleana *OR* con la siguiente condición, en lugar de *AND* (por defecto).

## Variables y *backreferences*

Tanto las reglas como las condiciones pueden usar variables de entorno (en cualquiera de sus *strings*). Estas se indican con el formato ***%{NOMBRE_VAR}***. Además, pueden contener *backreferences* a la regla más reciente (***\$1***, ***\$2***,...) o a la condición más reciente (***%1***, ***%2***,...). Si la *backreference* se produce en el segundo argumento, se referirá al primer argumento, si procede. De lo contrario, a la regla o condición más reciente.

Lista de algunas variables útiles: ***HTTP_USER_AGENT***, ***REMOTE_ADDR***, ***REMOTE_HOST*** (será igual que ***REMOTE_ADDR*** si el servidor está configurado para no hacer *IP address lookups*, que es lo habitual), ***REMOTE_PORT***, ***REMOTE_USER***, ***HTTP_REFERER*** (*URL* de origen), ***HTTP_COOKIE***, ***HTTP_HOST***, ***REQUEST_METHOD***, ***SCRIPT_FILENAME***, ***PATH_INFO***, ***QUERY_STRING***, ***AUTH_TYPE*** (***BASIC*** o ***DIGEST***), ***DOCUMENT_ROOT***, ***SERVER_ADMIN***, ***SERVER_NAME***, ***SERVER_ADDR***, ***SERVER_PORT***, ***SERVER_PROTOCOL***, ***SERVER_SOFTWARE***, ***TIME_YEAR***, ***TIME_MON***, ***TIME_DAY***, ***TIME_HOUR***, ***TIME_MIN***, ***TIME_SEC***, ***TIME_WDAY***, ***TIME***, ***REQUEST_URI***, ***HTTPS***.

También es posible usar el contenido de una cabecera *HTTP* mediante el formato ***%{HTTP:nombre_cabecera}***. Por ejemplo, podríamos recuperar las credenciales de autenticación, que el servidor recibe en una *request* que contiene una cabecera ***Authorization***. Podríamos recuperar esas credenciales si el sistema de autenticación es *basic* (no *digest*). Entonces solo deberíamos establecer una variable de entorno del servidor para que esta estuviera disponible en un *script*.

```
RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

La primera directiva será verdadera si la cabecera ***Authorization*** coincide con cualquier carácter, es decir, si tal cabecera está presente en la petición. En caso de ser verdadera, pues, se ejecutará la regla siguiente, que siempre coincidirá, pues se compara con cualquier conjunto de caracteres. En este caso, no se produce ninguna reescritura, pero sí se establece una variable de entorno llamada ***HTTP_AUTHORIZATION*** a la cual se le da el valor que tiene la cabecera ***Authorization*** recibida.

El valor de tal cabecera consiste en un *string* con dos *substrings* separados por un espacio. El primero de ellos es el tipo de autenticación, y puede ser ***Basic*** o ***Digest***. El segundo es una cadena del tipo ***usuario:password*** codificada mediante *base64*. En caso de desear obtener el usuario y la contraseña solo hay que decodificar tal *substring* (con `base64_decode()` de *PHP*, por ejemplo).

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

Supongamos ahora que nuestro recurso as accesible mediante ***http://example.com/verduras.php?pimiento*** pero queremos que tenga la forma más atractiva ***http://example.com/verduras/pimiento***. Se haría así:

```
RewriteRule ^/verduras/(.*) /verduras.php?$1 [PT]
```

El *flag* ***PT*** no es estrictamente necesario, pero es útil, y no está de más usarlo a discreción, a no ser que deseemos el efecto contrario, es decir, especificar una ruta de archivo. En alguna ocasión podríamos tener un problema si no usamos el *flag* y se interpreta como ruta de archivo. En nuestro caso es improbable que en el directorio raíz del sistema de archivos del *host* exista un archivo ***/verduras.php***, pero en algún otro caso podría no ser así.

Supongamos ahora que deseamos forzar el *hostname* canónico (***www.dominio.com***):

```
RewriteCond %{HTTP_HOST} !^www\.dominio\.com$ [NC]
RewriteRule (.*) http://www.example.com$1 [R,L]
```

Si además no queremos perder la *query string*, añadiremos también el *flag* ***QSA***.

Por otro lado, si queremos forzar conexión *HTTPS*:

```
RewriteCond %{HTTPS} !=on
RewriteRule ^(.*) https://%{SERVER_NAME}$1 [R,L]
```

Dentro de un contexto directorio o *htcaccess*:

```
RewriteCond %{HTTPS} !=on
RewriteRule ^directorio(.*) https://%{SERVER_NAME}directorio$1 [R,L]
```

Supongamos ahora que el sitio está en mantenimiento y queremos redirigir todas las páginas al aviso de mantinimiento:


```
RewriteCond %{REQUEST_URI} !^/mantenimiento.html
RewriteRule (.*) /mantenimiento.html [R]
```
La condición evita que se produzca redirección una y otra vez. Recordemos que en un contexto de directorio o *htaccess* debería escribirse así:

```
RewriteCond %{REQUEST_URI} !^mantenimiento.html
RewriteRule (.*) /mantenimiento.html [R]
```

En estos ejemplos, es más rápido escribir la regla así:

```
RewriteRule . /mantenimiento.html [R]
```

Esto hace lo mismo, y ahorra tiempo (no hace *match* con la *URI* entera, ni captura en un grupo).

Veamos ahora cómo mostrar contenido específico dependiendo de la fecha/hora:

```
RewriteCond %{TIME} >20050701080000
RewriteCond %{TIME} <20050705170000
RewriteCond %{HTTP_COOKIE} !promovista=si
RewriteRule /index.html /promocion.html [R,CO=promovista:si:.dominio.com:1440,L]
```

En este caso, al entrar a la página principal, si el rango de fechas/horas es el indicado, se redirigirá a una página que anuncia una promoción, siempre y cuando no exista la *cookie* ***promovista*** con el valor ***si***, que indica que la promo se ha visto ya. Al redirigir también se crea la mencionada *cookie* para evitar que el cliente la reciba otra vez en las próximas 24 horas.

También podemos incluir reglas para que una vez haya terminada la promoción, si el cliente intenta entrar en esa página, se le redirija el acceso a otra página que indique que ha terminado, o que se le deniegue el acceso a la página:

```
RewriteCond %{TIME} >20050705170000
RewriteRule /contest.html - [F]
```

Si queremos condiciones relacionadas con el software cliente, usaremos la variable ***HTTP_USER_AGENT***, que puede tener valores como ***MSIE***, ***Safari***, ***Mozilla***, etc. Puede ser útil hacer el *test* ignorando mayúsculas (*flag* ***NC***). También puede ser útil la *IP* del cliente (variable ***REMOTE_ADDR***).

Para obtener el valor de una variable de entorno, se puede usar ***%{ENV:nombrevar}***.
