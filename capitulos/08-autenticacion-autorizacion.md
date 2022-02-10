# Autenticación y autorización

Veremos aquí los mecanismos *core* de autenticación y autorización de usuarios en el servidor.

## Autenticación básica (módulo mod_authn_core)

Este módulo proporciona mecanismos de autenticación. Las directivas de este módulo son comunes a todos los proveedores de autenticación.

Los datos del usuario autenticado se guardan internamente en el servidor. Al terminar la sesión actual (cerrando el navegador o limpiando datos de sesión), el navegador volverá a solicitar autenticación.

### Directiva \<AuthnProviderAlias>

**Contexto:** *server config*.

Para crear un proveedor de autenticación configurado adecuadamente, podemos crear un alias del proveedor genérico, añadiéndole nuestra configuración.

```
<AuthnProviderAlias file file1>
    AuthUserFile "/www/conf/passwords1"
</AuthnProviderAlias>

<AuthnProviderAlias ldap ldap-alias1>
    AuthLDAPBindDN cn=youruser,o=ctx
    AuthLDAPBindPassword yourpassword
    AuthLDAPURL ldap://ldap.host/o=ctx
</AuthnProviderAlias>
```

En el ejemplo utilizamos un proveedor de tipo ***file*** y otro de tipo ***ldap***, y configuramos sus opciones (específicas de cada tipo). Los proveedores resultantes se accederán con los alias ***file1*** y ***ldap-alias1*** respectivamente.

Ahora ya podemos incluir cualquiera de estos proveedores a las partes de nuestro sitio que precisen autienticación. Dependiendo del tipo de autenticación que deseemos, usaremos la directiva `AuthBasicProvider` (básica, sin encriptar los datos) o `AuthDigestProvider` (*digest*, los datos viajan encriptados).

```
<Directory "/var/web/pages/secure">
    AuthBasicProvider file1
    AuthType Basic
    AuthName "Protected Area"
    Require valid-user
</Directory>
```

Si no utilizamos alias, debemos incluir la configuración cada vez:

```
<Directory "/var/web/pages/secure">
    AuthBasicProvider file
    AuthUserFile "/www/conf/passwords1"
    AuthType Basic
    AuthName "Protected Area"
    Require valid-user
</Directory>
```

### Directiva AuthBasicProvider

La directiva `AuthBasicProvider`, del módulo ***mod_auth_basic***, admite una lista de varios proveedores (separados por espacios). Se deben incluir junto a esta, las directivas para configurar el proveedor en cuestión.

**Contexto:** directorio y *htaccess*.

**Por defecto:** ***file***.

### Directiva AuthName

Esta directiva marca el *realm* de la autenticación. Páginas y recursos en el mismo *realm* mantendrán las mismas credenciales, siempre dentro de la misma sesión. Dado que visitar una página en la que cambia el protocolo, el *host name* y/o el puerto, pertenece a otra sesión, los datos de autenticación se volverán a solicitar (siempre que esa nueva página esté configurada para autenticación). Así, si cambiamos alguno de estos elementos en el navegador, nos volverá a solicitar las credenciales, aunque el ***AuthName*** sea el mismo.

En este contexto, una visita a un subdominio implica otra sesión. Es decir, aunque estemos estemos correctamente autenticados en ***ejemplo.com***, visitar ***sub.ejemplo.com*** nos volverá a solicitar credenciales (si ese subdominio precisa de autenticación).

Por el contrario, si dos partes de un mismo sitio (mismo protocolo, nombre de *host* y puerto) definen *realms* distintos, las credenciales de una parte no serán válidas para la otra, y se volverán a solicitar, autenticándose/autorizándose con el mismo proveedor o con uno diferente, según nuestras necesidades.

**Contexto:** directorio y *htaccess*.

### Directiva AuthType

En este caso, indicamos el tipo de autenticación, normalmente ***Basic*** o ***Digest***. Si indicamos ***None*** significa sin autenticación. Este último caso puede ser útil si deseamos *override* un tipo de autenticación heredado, para que no solicite credenciales:

```
<Directory "/www/docs">
    AuthType Basic
    AuthName Documents
    AuthBasicProvider file
    AuthUserFile "/usr/local/apache/passwd/passwords"
    Require valid-user
</Directory>

<Directory "/www/docs/public">
    AuthType None
    Require all granted
</Directory>
```

**Contexto:** directorio y *htaccess*.

## Autorización básica (módulo mod_authz_core)

Proporciona mecanismos de autorización para usuarios autenticados, típicamente las directivas `Require`, a través de las cuales se definen requisitos de acceso, según el proveedor de autorización definido.

### Contenedores de autorización

Las *container directives* `<RequireAny>`, `<RequireAll>` y `<RequireNone>` se evalúan a verdadero si se cumple al menos uno, todos, o ninguno de los requisitos (`Require`) que contienen, respectivamente. Estos contenedores se pueden anidar para definir condiciones complejas.

**Contexto:** directorio y *htaccess*.

### Directiva Require

Algunos proveedores proporcionados por el módulo:

- `Require all granted` proporciona acceso incondicional.
- `Require all denied` acceso denegado incondicional.
- `Require env <var1> [<var2> ...]` acceso solo si una de las variables de entorno está definida.
- `Require method <metodo-http1> [<metodo-http2> ...]` solo para los métodos *HTTP* indicados (*get*, *post*,...). Los métodos *get* y *head* se tratan como sinónimos.

Los módulos ***mod_authz_user***, ***mod_authz_host*** y ***mod_authz_groupfile*** proporcionan sintaxis como estas:

- `Require user <ID1> [<ID2> ...]` solo son autorizados los usuarios indicados.
- `Require group <grupo1> [<grupo2> ...]` solo se autoriza a usuarios que pertenezcan a los grupos indicados.
- `Require valid-user` autoriza a todos los usuarios válidos autenticados.
- `Require ip 10 172.20 192.168.2` autoriza a los clientes cuyas *IPs* empiezan por las direcciones indicadas.
- `Require local` autoriza peticiones de ***localhost***.
- `Require forward-dns un.ejemplo.com` autoriza peticiones provenientes de un cliente cuya *IP* se resuelva a partir del nombre indicado.

Existen varios módulos que permiten trabajar con distintos tipos de proveedores de usuarios, como ***mod_authnz_ldap*** que permite autenticar contra un directorio *LDAP*.

**Contexto:** directorio y *htcaccess*.

Ejemplo:

```
<RequireAny>
     Require method GET POST OPTIONS
     Require valid-user
</RequireAny>
```

En este caso, cualquier *request* *get*, *head*, *post* u *options* dejará pasar sin que haga falta siquiera autenticarse. Para el resto de métodos se necesita autenticación válida.

En muchas ocasiones, para una correcta autenticación y autorización será necesario combinar la directiva `Require` con directivas de otros módulos, como `AuthName`, `AuthType` y `AuthBasicProvider`/`AuthDigestProvider`, así como directivas específicas de cada proveedor, como por ejemplo `AuthUserFile` y `AuthGroupFile` (estas definen usuarios y grupos en el caso de un proveedor de tipo ***file***).

A la hora de definir un requisito podemos añadir `not` para indicar que el requisito debe no cumplirse.

```
<RequireAll>
    Require group alpha beta
    Require not group reject
</RequireAll>
```

En este caso estamos dando autorización a usuarios que pertenezcan al grupo ***alhpa*** y/o ***beta*** que **no pertenezcan** a ***reject***.

Si varias directivas de autorización están en una sección de configuración sin estar dentro de una *container directive* de autorización (como `<RequireAll>`), funcionarán como si estuvieran dentro de una directiva `<RequireAny>`, es decir si una de ellas se cumple, se dará la autorización.

> Independientemente de las directivas de acceso, el sistema operativo debe forzosamente permitir el acceso a los archivos por parte del usuario *Apache* (***www-data*** en algunos sistemas). En el caso de *Linux*, por ejemplo, todos los directorios desde el raíz hasta los archivos deberán permitir ejecución para **otros**. Esto significa que deberían tener, por ejemplo, permisos 755. En cuanto al archivo en sí, el permiso 644 permite lectura a **otros**.
