# Mapeo URL-archivo

Por defecto se concatena el *URL path* al contenido de ***DocumentRoot*** como mapeo básico. Se puede definir un ***DocumentRoot*** distinto para cada *virtual host*.

Al especificar así una carpeta en la *URL*, podemos indicar qué archivo será el índice que se retornará en la respuesta:

```
DirectoryIndex index.html index.php
```

En este caso indicamos que preferentemente sea ***index.html***, y si no lo encuentra, ***index.php***. Si tampoco, se retornará un índice del directorio (siempre y cuando esté habilitado para este).

Esta directiva pertenece al módulo ***mod_dir***.

## Acceso fuera del árbol

Para acceder a archivos fuera del árbol ***DocumentRoot***, podemos crear enlaces simbólicos (en *Unix*). Aunque solo se seguirán estos si están permitidos. Para ello hay que incluir ***FollowSymLinks*** o ***SymLinksIfOwnerMatch*** en la directiva `Options`. En el segundo caso, solo se seguirá el enlace simbólico si el usuario propietario de dicho enlace es el mismo que el propietario del archivo o directorio final.

Alternativamente se puede usar la directiva `Alias` para mapear una *URL* a un archivo del sistema de archivos local:

```
Alias "/docs" "/var/web"
```

En este caso se mapea la *URI* ***docs*** a la carpeta cuya ruta absoluta es ***/var/web***. La directiva sustituye fragmentos completos de rutas *URL*, es decir, la *URL* ***http://servidor.com/docs/dossier.txt*** se mapeará al archivo ***/var/web/dossier.txt***, pero ***http://servidor.com/docsitos/dossier.txt*** o ***http://servidor.com/data/docs/dossier.txt*** no realizará mapeo alguno.

Hay que recalcar que si indicamos el *trailing slash* final en el patrón, también hay que indicarlo en el *string* a sustitir:

```
Alias "/docs/" "/var/web/"
```

Si queremos usar expresiones regulares, disponemos de la directiva `AliasMatch`. Ambas directivas pertenecen al módulo ***mod_alias***.

```
AliasMatch "^/docs/(.*)$" "/var/web/$1"
```

Este ejemplo equivale al ejemplo anterior. En cambio si hacemos:

```
Alias "/docs(.*)" "/var/web$1"
```

Esto sustituirá cualquier parte de la *URI* que contenga ***/docs***.

## Redirección

En el caso de la redirección se informa al cliente que el recurso está en otra *URL*, e instruye al navegador para que realice una nueva *request* a la nueva *URL*. Es decir, mapea una *URL* a otra *URL*. Las directivas `Redirect` pertenecen al módulo ***mod_alias***.

La *URL* de entrada es una *%-decoded URL-path* de la *request*, es decir, fragmento justo después del nombre de *host* (y puerto), y justo antes de la *query string* (si la hay).

La directiva puede redirigir a una *URL* absoluta (tipo ***http://servidor.com***), o relativa al *document root* (empezando con ***/***: ***/docs/index.html***).

```
Redirect permanent "/foo/" "http://www.example.com/bar/"
```

En este caso, todas las peticiones con un *URL-path* que empiece por ***/foo/***, se redirigirán a la *URL* indicada (puede ser en un servidor externo), de tal modo que la *URL* original, incluido ***/foo/*** se sustituirá con la nueva *URL*. Dado que se trata de una sustitución literal, si el primer parámetro termina con una barra, la *URL* de destino también deberá hacerlo.

Podemos usar expresiones regulares en la directiva `RedirectMatch`:

```
RedirectMatch permanent "^/$" "http://www.example.com/startpage.html"
RedirectMatch temp ".*" "http://othersite.example.com/startpage.html"
```

En este caso, la primera redirección (***permanent***, permanente, estado 301) se refiere a la página raíz ***/***, y la segunda (***temp***, temporal, estado 302, opción por defecto) redirigirá todas las solicitudes a la página especificada (en otro servidor). En estos casos se pueden usar las directivas `RedirectPermanent` o `RedirectTemp` (equivalente a `Redirect` sin argumentos) respectivamente.

Existen otros dos tipos de redirección: ***seeother*** retorna estado *See Other* (303), que reemplaza un recurso (normalmente *POST*) a una página *GET*; y ***gone*** retorna estado *Gone* (410), indicando que el recurso ya no está, en cuyo caso no debería indicarse el campo *URL*.

También es posible indicar numéricamente el estado a retornar. Si este es entre 300 y 399 se debe indicar el campo *URL*.

Dentro de **un mismo nivel**, si existen tanto directivas `Redirect`/`RedirectMatch` y `Alias`/`AliasMatch`, las primeras (todas) se ejecutarán **antes** que las segundas, por lo que si se produce una coincidencia con alguna `Redirect`/`RedirectMatch`, ninguna de las `Alias`/`AliasMatch` tendría efecto.

También **dentro del mismo contexto** (nivel), y **dentro de las del mismo tipo** (tipo `Redirect`/`RedirectMatch` o tipo `Alias`/`AliasMatch`), se ejecutarán en el orden indicado, con lo que se deberán indicar las más específicas antes de las más generales. Por ejemplo:

```
Alias /foo/bar /var/web/bar
Alias /foo     /var/web
```

## Otros

Es posible definir la apariencia de una respuesta 404 (no encontrado) mediante la directiva `ErrorDocument`.

Por otro lado, el módulo ***mod_rewrite*** ofrece mecanismos de reescritura de las *URLs*. Se verá en su propio apartado.
