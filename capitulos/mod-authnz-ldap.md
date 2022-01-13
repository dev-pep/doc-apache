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

## Directiva LDAPReferrals (módulo ***mod_ldap***)

Una organización puede dividir su directorio en varios dominios (por ejemplo por provincias). Cuando el dominio se traspasa (un usuario de una provincia intentando acceder a datos de usuarios en otra provincia, por ejemplo), se puede permitir el acceso automático al servidor adecuado (*referral*).

Si el directorio está configurado para no permitir los *referrals*, si *Apache* intenta usar un *referral* se producirá error. Por lo tanto habrá que indicar que no está permitido el *referral chasing* mediante `LDAPReferrals off`.

Por otro lado, la directiva (también de ***mod_ldap***) `LDAPReferralHopLimit` indica el máximo número de saltos máximo durante el *referral*.

## Directiva AuthLDAPBindDN

Se utiliza para hacer búsquedas en el directorio *LDAP* cuando no hay un *binding* (no se ha autenticado ningún usuario todavía). En este caso se le debe pasar el *dn* de un usuario (y una contraseña con `AuthLDAPBindPassword`), pues de lo contrario hará las búsquedas en modo anónimo, lo cual solo sería válido si el servidor permitiese búsquedas anónimas.

En el caso concreto de un servidor de **Directorio Activo**, es posible indicar, en lugar de un *dn* completo, otro campo, como ***sAMAccountName***, ***userPrincipalName***, y otros (es un mecanismo de *DA* llamado *ambiguous name resolution*, o *ANR*).

**Contexto:** directorio, *htaccess*.

## Directiva AuthLDAPBindPassword

Esta directiva funciona junto con `AuthLDAPBindDN` para el acceso al directorio *LDAP* para realizar búsquedas.

**Contexto:** directorio, *htaccess*.

## Directiva AuthLDAPCompareAsUser

Esta directiva puede estar activa (***on***) o inactiva (***off***). Si está activa, la comparación se realiza bajo el usuario autenticado en lugar de las credenciales configuradas con `AuthLDAPBindDN` y `AuthLDAPBindPassword`.

Los `Require` ***ldap-attribute***, ***ldap-user*** y ***ldap-group*** utilizan comparación (autorización). En el caso de comparación con grupos anidados, solo se puede hacer en el primer nivel. Si de desean más niveles, se debe activar también la directiva `AuthLDAPSearchAsUser`.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***off***.

## Directiva AuthLDAPGroupAttributeIsDN

Si está activada esta directiva (***on***), para comprobar si el usuario pertenece a un grupo determinado, se comprobará si el grupo contiene un miembro que coincide con el *dn* del usuario. De lo contrario, se comprobará si el grupo contiene un miembro igual al nombre de usuario proporcionado al hacer *login*.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***on***.

## Directiva AuthLDAPInitialBindAsUser

Por defecto, el acceso al servido *LDAP* se hace de forma anónima o usando el nombre de usuario y contraseña establecidos con `AuthLDAPBindDN` y `AuthLDAPBindPassword` en la búsqueda inicial, cuando todavía no hay usuario autenticado.

Activando esta directiva (***on***), se utiliza el nombre de usuario y contraseña entrantes para obtener el *dn* que se usará pará hacer la búsqueda, haciendo innecesario aplicar las directivas `AuthLDAPBindDN` y `AuthLDAPBindPassword`.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***off***.

## Directiva AuthLDAPMaxSubGroupDepth

Indica el número de niveles para las búsquedas en grupos anidados.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***10***.

## Directiva AuthLDAPRemoteUserAttribute

Especifica qué campo se usará para dar valor a la variable de entorno ***REMOTE_USER***. Dicho campo debe estar entre los atributos especificados en `AuthLDAPURL`. Si no se especifica esa directiva, se usará el nombre de usuario proporcionado por el mismo al autenticarse.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***none***.

## Directiva AuthLDAPRemoteUserIsDN

Si se establece en ***on***, el valor de la variable de entorno ***REMOTE_USER*** se establecerá al *dn* completo del usuario autenticado, en lugar de el nombre de usuario que proporcionó el mismo al autenticarse.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***off***.

## Directiva AuthLDAPSearchAsUser

Esta directiva puede estar activa (***on***) o inactiva (***off***). Si está activa, las búsquedas se realizan bajo el usuario autenticado en lugar de las credenciales configuradas con `AuthLDAPBindDN` y `AuthLDAPBindPassword`.

Los `Require` ***ldap-filter*** y ***ldap-dn*** utilizan comparación (autorización). En el caso de comparación con grupos anidados, solo se puede hacer si la directiva `AuthLDAPCompareAsUser` está activa también.

**Contexto:** directorio, *htaccess*.

**Por defecto:** ***off***.

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
- El atributo es el atributo o atributos (separados por comas, aunque para la autenticación solo se usará el primero) por los que buscar. Por defecto es ***uid***.
- El *scope* indica el ámbito de búsqueda: ***one*** busca solo en los nodos hijos inmediatos del *dn* base, mientras que ***sub*** (valor por defecto) busca en todo el subárbol (sin incluir el *base dn* en sí).
- El filtro es una expresión (por defecto ***objectClass=\****, es decir, **todo**) que filtra los nodos para la búsqueda. En algunos servidores *LDAP* antiguos se debe indicar ***none*** para deshabilitar el filtrado.

Si queremos dejar algún campo en blanco pero no un campo posterior, podemos dejarlo en blanco escribiendo interrogantes consecutivos (***??***).

Se pueden indicar otros argumentos tras la *URL*:

- ***NONE*** indica conexión insegura (*ldap* en puerto 389).
- ***SSL*** indica conexión segura (*ldaps* en puerto 636).
- ***TLS*** o ***STARTTLS*** indica conexión segura en puerto 389.

Para configuraciones adicionales (certificado, etc.) el módulo ***mod_ldap*** proporciona directivas de configuración de seguridad.

> Si usamos autenticación con este módulo, el atributo o atributos especificados en la URL quedan disponibles en variables de entorno con prefijo ***AUTHENTICATE_*** (***AUTHENTICATE_SAMACCOUNTNAME***, ***AUTHENTICATE_DISPLAYNAME***, etc.).
>
> Igualmente sucede si se usa para autorización, en cuyo caso disponemos de variables de entorno con prefijo ***AUTHORIZE_***.
>
> Todas estas variables quedan disponibles a los *scripts* mediante el *array* ***$_SERVER***.

**Contexto:** directorio, *htaccess*.
