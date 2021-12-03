# Configuración de *Apache 2*

Dentro de un archivo de configuración, la directiva `Include` añade otros archivos. Esta directiva admite *wildcards*.

Tras un cambio en la configuración, el servidor debe ser reiniciado.

Cada directiva ocupa una línea. Si termina en barra invertida (***\\***) proseguirá en la siguiente línea. Si un argumento contiene espacios se debe encerrar entre comillas dobles (***\"***). Las directivas son *case-insensitive*, pero los argumentos suelene ser *case-sensitive*.

Se ignoran las líneas en blanco y el espacio de indentación. Las líneas que **empiezan** por almohadilla (***\#***) son comentarios.

Se pueden usar variables de entorno o variables definidas con la directiva `Define` (tienen preferencia sobre variables de entorno) usando la sintaxis ***\${VAR}***. Solo se expanden variables del sistema que existan antes del inicio del servidor. Si son variables de sistema definidas en el mismo archivo de configuración (`SetEnv`), no se expandirán.

## Módulos

*Apache* es un servidor modular. Se compila con un conjunto básico de módulos. Si dicha compilación se hace permitiendo la carga dinámica de módulos, estos se pueden añadir o desactivar fácilmente, gracias a la directiva `LoadModule`. Por otro lado, la sección `<IfModule>` aplica directivas condicionalmente a la presencia de un módulo determinado. Al ejecutar el binario principal (que puede ser ***/usr/sbin/apache2***) con el *flag* `-l`, se muestran los módulos compilados estáticamente. Con el *flag* `-M`, vemos todos los módulos activos (estáticos y dinámicos).

## Ámbito

Para que las directivas se apliquen solo en ciertos directorios, archivos o *URLs*, disponemos de las secciones `<Directory>`, `<DirectoryMatch>`, `<Files>`, `<FilesMatch>`, `<Location>` y `<LocationMatch>`. Para que se apliquen solo a ciertas *requests*, tenemos las secciones par los *virtual hosts* `<VirtualHost>`.

## Archivos de configuración descentralizada

Para especificar el nombre del archivo de acceso (por defecto ***.htaccess***) existe la directiva `AccessFileName`. Para controlar qué tipo de directivas pueden indicarse en estos archivos de configuración descentralizada, se usa `AllowOverride`. El formato de este archivo es el mismo que el de los archivos de configuración.

## Secciones

Las secciones pueden anidarse.

### Secciones condicionales

La mayoría de secciones se evalúa para cada *request*. Pero las secciones `<IfDefine>`, `<IfModule>`, `<IfFile>` y `<IfVersion>` solo lo hacen al iniciarse el servidor. Si la condición es cierta, se aplicarán en cada *request*; de lo contrario serán siempre ignoradas sus directivas.

En el caso de `<IfModule>`, será aplicado solo si el módulo referido está compilado estáticamente, o lo está dinámicamente y su activación (sección `<LoadModule>` es anterior a `<IfModule>`).

Estas secciones pueden aplicar la condición a la inversa si se prefija ***!*** antes de la condición.

### Archivos, directorios y espacio *web*

Estas secciones sirven para confinar la acción de una directiva a ciertos ámbitos.

`<Directory>` y `<Files>` especifican respectivamente un directorio (y recursivamente todos sus elementos) o archivo (independientemente del directorio donde resida). Se admiten *wildcards*: asterisco (***\****), interrogante (***?***) y grupo de caracteres entre corchetes. El carácter barra (***/***) no coincide con ningún *wildcard*.

Para ceñirse a ciertos archivos dentro de un directorio, por ejemplo, se puede anidar una sección `<Files>` dentro de una sección `<Directory>`.

```
<Directory "/var/web/dir1">
    <Files "private.html">
        # Directivas a aplicar
    </Files>
</Directory>
```

Colocadar directivas en una sección `<Directory>` equivaldría a colocarlas en el archivo ***.htaccess*** del directorio en cuestión. Aunque es preferible lo primero, ya que el archivo ***.htaccess*** se carga y procesa a cada *request*, mientras `Directory` solo se lee y parsea al iniciar el servidor.

Se puede utilizar una expresión regular, tras el carácter ***~***:

```
<Directory ~ "^/var/.*/[0-9]{3}"
    # Directivas
</Directory>
```

La sección `<Location>` circunscribe a los *URL path* que coincidan (total o parcialmente) con lo indicado. No tiene por qué coincidir con un nombre de archivo. Por ejemplo, esta sección asocia el *URL-path* ***/estado*** a un *handler Apache* que proporciona el módulo ***mod_status***:

```
<Location "/server-status">
    SetHandler server-status
</Location>
```

Hay que tener en cuenta el orden de las directivas cuando hay solapamiento:

```
<Location "/foo">
    # ...
</Location>

<Location "/foo/bar">
    # ...
</Location>
```

Ese es el orden correcto: cuando hacemos una petición de ***/foo/bar***, la primera sección coincide, con lo que se aplican las directivas de esta. Sigue hacia abajo, y también coincide la segunda sección, con lo que se aplicarán esas directivas, sobrescribiendo las primeras.

En cambio, al definir *alias* (directiva `Alias`, explicada más adelante) se hace al revés.

A parte del uso de *wildcards*, si queremos más flexibilidad podemos usar expresiones regulares con las correspondientes secciones `<DirectoryMatch>`, `<FilesMatch>` o `<LocationMatch>`.

### Expresiones booleanas

Podemos usar también la directiva `<If>` con una expresión *Apache* que retorne un valor booleano.

Este directiva no se puede anidar dentro de otra sección `<If>`. Disponemos también de las directivas asociadas `<Else>` y `<ElseIf>`.

### Directivas permitidas

Cada directiva tiene un contexto, que marca en qué secciones puede usarse y en qué secciones no.

### Orden de aplicación de las directivas

En general, las directivas se van aplicando en orden de aparición. Dentro de un nivel, se aplican primero todas las directivas que no son contenedores, y luego se ejecutan los contenedores que procedan. Pero hay algunos contenedores que se aplican con un orden concreto (unos tipos antes que otros):

1. `<Directory>` y ***.htaccess*** (en caso de conflicto, el último *overrides*).
2. `<DirectoryMatch>` (y `<Directory "~">` si existe).
3. `<Files>` y `<FilesMatch>`.
4. `<Location>` y `<LocationMatch>`.
5. `<If>`.

Dentro de cada tipo de contenedor, estos se aplicarán en el orden de aparición, excepto `<Directory>` (no `<Location>`), que se aplicará en el orden más correcto, es decir,según la jerarquía de las rutas (las rutas más exteriores primero) para que se aplique *overriding* correctamente. Las directivas en archivos incluidos se tratarán como si estuvieran escritas en el archivo que llama a `Include`.

Como ejemplo, supongamos que el *document root* es ***/var/www/public***, y que existe un subdirectorio ***paises***, y dentro de este, un subdirectorio ***vaticano***. Supongamos que tenemos estos tres contenedores:

```
<Directory /var/www/public/paises>
   #...
</Directory>

<Directory /var/www/public/paises/vaticano>
   #...
</Directory>

<Directory /var/www/public>
   #...
</Directory>
```

Cuando hacemos una solicitud a ***http://servidor.com/paises/vaticano***, se ejecutarán las directivas de estos tres contenedores, pero primero se ejecutarán las correspondientes a ***/var/www/public***, luego las de ***/var/www/public/paises***, y finalmente las de ***/var/www/public/paises/vaticano***, de tal modo que las opciones de los directorios más interiores reemplazarán a las de los directorios más exteriores. Como vemos en el ejemplo, no es necesario ordenar adecuadamente estos contenedores, ya que el mecanismo es independiente del orden indicado. Si la solicitud fuese ***http://servidor.com/paises***, se ejecutarían primero las directivas correspondientes a ***/var/www/public*** y luego las de ***/var/www/public/paises***.
