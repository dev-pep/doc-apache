# *Virtual Hosts*

Permiten ejecutar varios sitios *web* en una sola máquina.

En el caso de *IP-based virtual hosts*, tenemos cada *VH* en una *IP* distinta de la máquina. En el caso de *name-based VHs* tenemos varios nombres de *host* distintos en una misma *IP*.

Cada *VH* se corresponderá con una *container directive* `<VirtualHost>`.

El uso de *virtual hosts* no afecta a qué direcciones y puertos escucha *Apache*. Esto debe configurarse globalmente con la directiva `Listen`.

## *Name-based virtual hosts*

Se aprovecha el hecho de que el cliente debe incluir en los encabezados de la petición el nombre del *host* servidor.

En la directiva `<VirtualHost>` se debe indicar la *IP* (que admite *wildcards*). En el caso de indicar simplemente un asterisco (***\****) en su lugar, la *IP* será irrelevante. Igualmente podemos indicar (o no) el número de puerto (por ejemplo ***\*:80***).

Cuando llega una petición, *Apache* comprueba primero los *VH* cuya *IP* coincida de la forma más específica (las *IPs* sin *wildcards* tienen precedencia sobre las que tienen *wildcards*). Si existen varias coincidencias con la misma especificidad, comprobará esas coincidencias comparando el nombre de *host* de la petición (cabecera *HTTP* ***Host***) con el nombre del *VH*, especificado con las directivas `ServerName` o `ServerAlias`.

Si se omite `ServerName` (no recomendado) se heredará el `ServerName` (será el *fully qualified domain name*, o *FQDN*) del servidor global.

Al llegar una solicitud, se busca coincidencia de *IP* con la mayor especificidad posible. Una vez establecida la especificidad de dicha coincidencia, si entre los *virtual hosts* **con esa especificidad concreta** no hay coincidencia **de nombre**, se usará el primero de dichos *VHs*.

Cada bloque `<VirtualHost>` deberá contener, como mínimo, una directiva `ServerName` y una `DocumentRoot`.

> Cuando no se encuentra coincidencia con ningún *VH* basándose en la *IP*, se usa la configuración global. Sin embargo si hay coincidencia de *IP* pero no de nombre, no se usará la configuración global sino el *default virtual host* para esa combinación de *IP* y puerto especificada en la petición. En este caso, se recomienda que este tenga el mismo nombre que el servidor base, para emular el mecanismo comentado inicialmente.

Si deseamos tener una serie de *VHs* en una *IP* de la máquina, y otros en otra *IP*, se debe indicar la *IP* en la directiva, así como los nombres de los *VHs*.

Para indicar otros nombres a través de los cuales se puede alcanzar es *VH* a parte del nombre definido en `ServerName`, usaremos `ServerAlias`, pasándole una lista de nombres alternativos (se acepta el uso de *wildcards*).

La búsqueda de coincidencias se hace por orden de aparición en la configuración. Dentro de cada *VH*, la búsqueda se hace también por orden de aparición de las directivas `ServerName` y `ServerAlias`.

Las directivas de configuración dentro de `<VirtualHost>` sobrescribirán la configuración global para ese *VH* concreto (siempre que sean directivas con contexto *virtual host*).

Si en la directiva `<VirtualHost>` se omite el esquema (protocolo), se usará el que viene en la petición. Omitir el número de puerto es como indicar como puerto un asterisco (***\****). En este caso, se asumirá el que viene con la petición. Por lo tanto, coincidirá con cualquier puerto en el que escuche el servidor. Si embargo, hay que tener en cuenta la regla de especificidad:

```
<VirtualHost *:80>
    # configuración del VH
</VirtualHost>

<VirtualHost *>
    # configuración de otro VH
</VirtualHost>
```

En este caso, una petición hacia el puerto 80 coincidirá con ambos *VHs*, pero solo se considerará el más específico, es decir, el primero, ya que el segundo no especifica puerto. El segundo *VH* se considerará solo para peticiones hacia puertos distintos del 80.

## *IP-based virtual hosts*

Si la máquina tiene varias interfaces de red (conexiones físicas o virtuales) podemos asociar uno o más *VHs* a cada combinación *IP*/puerto.

En este caso hay dos aproximaciones:

- Instalar un servidor *Apache* para cada combinación *IP*/puerto, configurando la directiva `Listen` de cada instalación adecuadamente.
- Instalar un solo servidor, estableciendo los correspondientes *IP-based VHs*.
