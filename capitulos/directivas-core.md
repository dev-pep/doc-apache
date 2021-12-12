# Resumen de directivas *core*

Resumen de directivas útiles del módulo *core*.

En general, los nombres de directivas son *case-insensitive*, pero algunos parámetros pueden ser *case-sensitive*. Los parámetros con espacios deben ir entrecomillados. Los parámetros sin espacios pueden entrecomillarse también.

Las directivas entre paréntesis angulares (***\<>***) suelen denominarse *container directives*, ya que contienen otras directivas en su interior.

## AcceptPathInfo

En ocasiones, la *URL* añade un *path information* tras el nombre del archivo, del tipo ***https://server.com/carpeta/index.php/un/path/cualquiera***. En este ejemplo, se mapeará al archivo ***/carpeta/index.php***, con un *path info* ***/un/path/cualquiera***. Con esta directiva activada (***On***) se aceptará este tipo de información, en cuyo caso el *script* puede acceder a esta información mediante la variable de entorno ***PATH_INFO***.

Si no se acepta esta información (***Off***), la *URL* debe ser una ruta literal que exista realmente.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

**Por defecto:** ***Default***, lo cual implica que se acepta *path info* o no dependiendo del *handler* que atienda la *request*. Normalmente se rechaza para archivos normales y se acepta para *scripts*.

## AccessFileName

Nombre o nombres separados por espacios (se busca la primera coincidencia) del archivo de configuración asociado al directorio donde reside el documento. El servidor buscará en ese directorio y en todos los directorios superiores, hasta el raíz del servidor (***DocumentRoot***). Sin embargo, si el directorio donde está el documento no tiene permitido el *override* de configuración (`AllowOverride none`), no se hará nada.

**Contexto:** *server config* y *virtual host*.

**Por defecto:** ***.htaccess***.

## AllowOverride

Configura la posibilidad de *override* local de la configuración. Si se establece en ***All***, se permitirá el *override* de todas las directivas que tengan contexto *htaccess*. Con ***None*** no se permite ninguna. A parte, se pueden especificar categorías de directivas.

**Contexto:** directorio.

**Por defecto:** ***None***.

## AllowOverrideList

Permite dar permiso de *override* a directivas específicas (siempre que tengan contexto *htaccess*).

```
AllowOverride None
AllowOverrideList Redirect RedirectMatch
```

Este ejemplo permite únicamente el *override* de directivas `Redirect` y `RedirectMatch`.

**Contexto:** directorio.

**Por defecto:** ***None***.

## Define

Permite definir el valor de una variable, equivalente a llamar al ejecutable de *Apache* con el flag `-D`. Afecta a las secciones `<IfDefine>`.

El primer parámetro es el nombre de la variable. El segundo, opcional, es el valor.

**Contexto:** *server config*, *virtual host* y directorio.

## \<Directory> y \<DirectoryMatch>

Véase capítulo de configuración.

**Contexto:** *server config* y *virtual host*.

## DocumentRoot

Indica la carpeta raíz del servidor o *virtual host*. Si no es una ruta absoluta, se considera relativa a ***ServerRoot***.

**Contexto:** *server config* y *virtual host*.

**Por defecto:** ***/usr/local/apache/htdocs***.

## Error

Produce un mensaje de error (primer parámetro) y detiene el parseo de la configuración.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

## ErrorDocument

Esta directiva especifica qué debe hacer el servidor cuando la *request* encuentra un problema. El primer argumento es el código de respuesta *HTTP* correspondiente al error, y el segundo indica la acción a realizar.

En cuanto a los códigos de respuesta, no cuentan los informativos (1xx), de éxito (2xx) o redirecciones (3xx). Sí podemos usar los códigos de error en el cliente (4xx) o en servidor (5xx). Los más frecuentes son:

- ***401 Unauthorized***: es necesaria la autenticación (se procederá a autenticarse).
- ***403 Forbidden***: el usuario no posee los permisos necesarios.
- ***404 Not Found***: no se puede encontrar el contenido solicitado.
- ***500 Internal Server Error***.
- ***503 Service Unavailable***.

En caso de encontrar un problema, el servidor puede hacer una de estas cuatro cosas:

1. Retornar el mensaje de error por defecto (*hardcoded*) relacionado con cada código de error retornado.
2. Retornar un mensaje personalizado.
3. Realizar una redirección interna a un *URL-path* local.
4. Realizar una redirección externa.

La primera es la opción por defecto para todos los códigos de respuesta de error. Si se quiere indicar explícitamente para un código de respuesta determinado (para evitar heredar el valor de un contexto superior), se indicará mediante ***default***:

```
ErrorDocument 404 default
```

La segunda opción se indica proporcionando el mensaje a retornar.

```
ErrorDocument 403 "Lo siento, no tienes privilegios suficientes"
```

En caso de indicar una ruta *web* local, se indicará con una *URI* que empiece por barra (***/***), la cual corresponderá a un archivo relativo al *document root*.

Si se indica una *URL* completa, deberá empezar por el indicador de protocolo (***http://*** o ***https://***). Aunque la *URL* corresponda al servidor local, al indicar la *URL* completa se realiza una redirección externa.

Cuando se realiza una redirección externa, el cliente no recibe el código de respuesta del error original, sino el código de la redirección. En el caso concreto de recibir un código 401, el cliente solicita las credenciales para autenticarse. Si configuramos para realizar una redirección externa al producirse un error 401, el cliente no recibirá ese 401 sino el código de la redirección, con lo cual no solicitará credenciales. Por lo tanto, una directiva `ErrorDocument 401` no debería realizar una redirección externa.

En el caso específico del estado 404, si lo que queremos es que las *URLs* que no se puedan mapear a un archivo sean tratadas por un único archivo controlador (*front controller*, como hace *Laravel* o *MediaWiki*), y que por lo tanto no retorne un estado de error sino de éxito (200), deberíamos usar la directiva `FallbackResource` en lugar de `ErrorDocument`.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

## \<Files> y \<FilesMatch>

Véase capítulo de configuración.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

## \<If>, \<Else> y \<ElseIf>

Directivas condicionales que trabajan con expresiones *Apache*.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

## \<IfDefine>, \<IfDirective>, \<IfFile>, \<IfModule>, \<IfSection>

Directivas condicionales según la existencia o no de una variable, directiva, archivo, módulo o sección.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

## Include y IncludeOptional

`Include` incluye un archivo de configuración. Se admiten *wildcards* para incluir varios archivos. Si el nombre especificado es el de un directorio, se incluirán todos los archivos de configuración contenidos en él y en sus subdirectorios. Si la ruta indicada no es absoluta, se considera relativa a ***ServerRoot***.

Si la ruta especificada no se encuentra, se produce un fallo. Para evitarlo se puede usar `IncludeOptional`, que ignora las rutas no encontradas.

**Contexto:** *server config*, *virtual host* y directorio.

## \<Location> y \<LocationMatch>

Véase capítulo de configuración.

**Contexto:** *server config* y *virtual host*.

## Options

Especifica ciertas opciones. Se pueden definir las opciones exactas para un directorio, por ejemplo, o añadir/quitar opciones heredadas de un directorio superior. Lo segundo se hace mediante los signos ***+*** y ***-***. Una sola directiva `Options` no puede tener opciones con signo y sin signo.

```
<Directory "/web/docs">
  Options Indexes FollowSymLinks
</Directory>

<Directory "/web/docs/spec">
  Options Includes
</Directory>
```

En este caso, el directorio ***/web/docs/spec*** solo tiene activada la opción ***Includes***. Veamos otro ejemplo:

```
<Directory "/web/docs">
  Options Indexes FollowSymLinks
</Directory>

<Directory "/web/docs/spec">
  Options +Includes -Indexes
</Directory>
```

En este caso, tendrá activadas las opciones ***Includes*** y ***FollowSymLinks***.

Veamos algunas opciones útiles:

- ***FollowSymLinks***: permite seguir enlaces simbólicos. Esta opción solo funciona en contexto directorio y *htaccess*.
- ***Indexes***: visualiza un índice formateado de la carpeta si no se encuentra el archivo índice en ella.
- ***SymLinksIfOwnerMatch***: como ***FollowSymLinks*** pero solo si el usuario propietario del enlace coincide con el del *target*.

**Contexto:** *server config*, *virtual host*, directorio y *htaccess*.

**Por defecto:** ***FollowSymLinks***.

## ServerAdmin

Normalmente, el correo electrónico del administrador del sitio, aunque puede ser también una *URL* personal del administrador (no recomendado). La dirección indicada puede verse incluida en algunos mensajes de error enviados al cliente.

**Contexto:** *server config* y *virtual host*.

## ServerAlias

Nombres alternativos por los que se puede llegar al *VH*. Permite el uso de *wildcards*. Por ejemplo, es útil para capturar todos los subdominios (tercer nivel).

**Contexto:** *virtual host*.

## ServerName

Indica el protocolo (*scheme*), nombre de *host* (o *IP*) y puerto que utiliza el servidor para identificarse. El esquema y puerto son opcionales. Si no se especifica puerto, se utiliza el puerto que viene en la *request*.

Se utiliza para identificar el *virtual host*.

**Contexto:** *server config* y *virtual host*.

## ServerRoot

Raíz del servidor en el *filesystem* local.

**Contexto:** *server config*.

**Por defecto:** ***/usr/local/apache***.

## TimeOut

Tiempo en segundos en el que se producirá un *time out*.

**Contexto:** *server config* y *virtual host*.

**Por defecto:** 60.

## UnDefine

Deshace la definición de una variable.

**Contexto:** *server config*.

## \<VirtualHost>

Véase el capítulo sobre *virtual hosts*.

**Contexto:** *server config*.
