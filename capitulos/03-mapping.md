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

Alternativamente se puede usar la directiva `Alias` para mapear una carpeta a otra:

```
Alias "/docs" "/var/web"
```

En este caso se mapea la carpeta ***docs*** perteneciente al *document root* a la carpeta cuya ruta absoluta es ***/var/web***.

Si queremos usar expresiones regulares, disponemos de la directiva `AliasMatch`. Ambas directivas pertenecen al módulo ***mod_alias***.

## Redirección

En el caso de la redirección se informa al cliente que el recurso está en otra *URL*, e instruye al navegador para que realice una nueva *request* a la nueva *URL*. Las directivas `Redirect` pertenecen al módulo ***mod_alias***.

```
Redirect permanent "/foo/" "http://www.example.com/bar/"
```

En este caso, todas las peticiones con un *URL-path* que empiece por ***/foo/***, se redirigirán a la *URL* indicada (puede ser en un servidor externo), de tal modo que la *URL* original, incluido ***/foo/*** se sustituirá con la nueva *URL*.

Podemos usar expresiones regulares en la directiva `RedirectMatch`:

```
RedirectMatch permanent "^/$" "http://www.example.com/startpage.html"
RedirectMatch temp ".*" "http://othersite.example.com/startpage.html"
```

En este caso, la primera redirección (permanente, estado 301) se refiere a la página raíz ***/***, y la segunda (temporal, estado 302) redirigirá todas las solicitudes a la página especificada (en otro servidor). En este caso se pueden usar las directivas `RedirectPermanent` o `RedirectTemp` respectivamente.

## Otros

Es posible definir la apariencia de una respuesta 404 (no encontrado) mediante la directiva `ErrorDocument`.

Por otro lado, el módulo ***mod_rewrite*** ofrece mecanismos de reescritura de las *URLs*. Se verá en su propio apartado.
