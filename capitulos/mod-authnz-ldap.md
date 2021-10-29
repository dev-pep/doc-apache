# Módulo mod_authnz_ldap

Este módulo proporciona la posibilidad de autenticación (de ahí la ***n***) y autorización (de ahí la ***z***) contra servidores *LDAP*.

El módulo utiliza mecanismos de *cache* a través del módulo ***mod_ldap***, que debe estar habilitado.

Se invoca este módulo a través de la directiva `AuthBasicProvider`, cuyo valor debe contener el proveedor ***ldap***.

## Funcionamiento

La primera fase es de autenticación, también llamada *binding*, en la que se comprueba en el directorio *LDAP* que las credenciales sean válidas. Si lo son, el usuario especificado queda *bound* (autenticado contra el *LDAP*).

La segunda fase es la de autorización, también llamada *compare*, y es donde se hace uso de las directivas `Require`.

La autenticación se produce enviando el nombre de usuario y su contraseña al servidor *LDAP* proporcionado por la directiva `AuthLDAPURL`.

Para realizar la búsqueda del usuario en el servidor *LDAP* es necesario especificar un nombre de usuario y contraseña mediante las directivas `AuthLDAPBindDN` y `AuthLDAPBindPassword` (a no ser que el servidor permita buscar anónimamente).

En cuanto a la autorización, a parte de las expresiones habituales (como `Require valid-user`), pueden usarse algunas extensiones de la directiva `Require`:

- ***ldap-user*** seguido de uno o más nombres de usuario dan acceso a los usuarios especificados. Se considera nombre de usuario el campo especificado en ***atributo***, al especificar la *URL* con `AuthLDAPURL`.
- ***ldap-dn*** seguido de uno o más *distinguished names* dan acceso a los usuarios cuyo *dn* (*distinguished name*) coincide con los indicados.
- ***ldap-group*** da acceso si el usuario pertenece al grupo especificado.
- ***ldap-attribute*** permite el acceso si existe alguna coincidencia con los atributos especificados. Estos se indican del modo ***attr1=val1 attr2=val2 ...***.
- ***ldap-filter*** sirve para especificar un filtro de búsqueda.

Recordemos que los campos *LDAP* son en realidad *arrays* con varios valores. Supongamos que definimos el atributo *cn* (*common name*) en `AuthLDAPURL`, y que hay un usuario que tiene varios valores en dicho atributo: ***Pablo Mármol***, ***P. Mármol***, y ***Paul Marble***. Supongamos que tenemos esta directiva en la configuración:

`Require ldap-user "P. Mármol"`

Al introducir sus credenciales puede utilizar cualquiera de sus *common names*, ya que al realizarse la búsqueda, el servidor *LDAP* retornará el registro completo, incluyendo sus tres nombres *cn*. Solo que uno de esos tres coincida con el de la directiva, el acceso es concedido.

## Directiva AuthLDAPBindDN

Se utiliza para hacer búsquedas en el directorio *LDAP* cuando no hay un *binding* (no se ha autenticado ningún usuario todavía). En este caso se le debe pasar el *dn* de un usuario (y una contraseña con `AuthLDAPBindPassword`), pues de lo contrario hará las búsquedas en modo anónimo, lo cual solo sería válido si el servidor permitiese búsquedas anónimas.

**Contexto:** directorio, *htaccess*.

## Directiva AuthLDAPBindPassword

Esta directiva funciona junto con `AuthLDAPBindDN` para el acceso al directorio *LDAP* para realizar búsquedas.

**Contexto:** directorio, *htaccess*.

## Directiva AuthLDAPRemoteUserAttribute

Especifica qué campo se usará para dar valor a la variable de entorno ***REMOTE_USER***. Dicho campo debe estar entre los atributos especificados en `AuthLDAPURL`. Si solo especificamos un atributo en `AuthLDAPURL`, se usará ese, y no hará falta indicar `AuthLDAPRemoteUserAttribute`.

## Directiva AuthLDAPURL

A esta directiva se le pasa un *string* con el *host*, puerto, *dn* base, y otros parámetros de búsqueda. El formato del *string* es:

```
ldap://<host>:<port>/<basedn>?<atributo>?<scope>?<filtro>
```

Es posible utilizar más de una *URL*, separándolas con espacios. En ese caso, el *string* entero debe ir entrecomillado (la directiva admite un solo argumento).

Veamos las distintas partes de este *string*:

- El protocolo especificado puede ser ***ldap*** o ***ldaps***.
- El *host* y puerto definen la localizació del servidor *LDAP* y son por defecto ***localhost:389*** para *ldap*, y ***localhost:636*** para *ldaps*.
- El *dn* base es un *dn* que representa al nodo donde deben empezar las búsquedas. Por defecto es el nodo raíz del servidor.
- El atributo es el atributo o atributos (separados por comas, aunque solo se usará el primero) por los que buscar. Por defecto es ***uid***.
- El *scope* indica el ámbito de búsqueda: ***one*** busca solo en los nodos hijos inmediatos del *dn* base, mientras que ***sub*** (valor por defecto) busca en todo el subárbol (sin incluir el *base dn* en sí).
- El filtro es una expresión (por defecto ***objectClass=\****, es decir, **todo**) que filtra los nodos para la búsqueda. En algunos servidores *LDAP* antiguos se debe indicar ***none*** para deshabilitar el filtrado.

Si queremos dejar algún campo en blanco pero no un campo posterior, podemos dejarlo en blanco escribiendo interrogantes consecutivos (***??***).

Se pueden indicar otros argumentos tras la *URL*:

- ***NONE*** indica conexión insegura (*ldap* en puerto 389).
- ***SSL*** indica conexión segura (*ldaps* en puerto 636).
- ***TLS*** o ***STARTTLS*** indica conexión segura en puerto 389.

Para configuraciones adicionales (certificado, etc.) el módulo ***mod_ldap*** proporciona directivas de configuración de seguridad.

**Contexto:** directorio, *htaccess*.
