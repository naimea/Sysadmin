# Solaris
Algunos consejos sobre ciertas actividades administrativas de los equipos Solaris

## Cambiar el nombre del equipo
Principalmente el nombre del equipo se encuentra en el archivo /etc/nodename, aunque puede haber otras referencias en el /etc/hostname (aquí dependerá si el equipo tiene un sólo archivo hostname, ya que puede haber uno por interface de red, ejemplo, "hostname.ce1"), el /etc/inet /ipnodes y /etc/inet/hosts.
Para cambiar el nombre del equipo, se requiere:

1) Modificar el archivo /etc/nodename y el /etc/hostname o los /etc/hostname.* que correspondan.

2) Reiniciar el equipo o ejecutar el comando "uname -S {nuevo nombre}". Para ver el cambio en el prompt del shell se debe cerrar y reiniciar la sesión o cargar nuevamente el profile.

3) Aunque el nombre del equipo cambia con los pasos 1 y 2, es altamente recomendable revizar todos los archivos mencionados inicialmente para detectar y correguir inconsistencias, incluyendo adicionalmente, consultas a los DNS.

## Comandos para visualizar la configuración del sistema
Para ver los parámetros de configuración de un sistema Solaris existen los siguientes comandos:

1) prtconf: Muestra la configuración del sistema, incluyendo la cantidad total de memoria y la configuración de los periféricos del sistema presentada en forma de árbol.

2) prtdiag: Muestra información de configuración y diagnóstico. Su uso más frecuente es de la forma: 
```Bash
prtdiag -v
```

3) sysdef: Muestra la definición actual del sistema en formato de tabla. Presenta todos los dispositivos de hardware, seudodispositivos, dispositivos del sistema, módulos cargables y valores de determinados parámetros ajustables del núcleo. Este comando genera su salida analizando el archivo de nombres (listanombres) del sistema operativo de arranque y extrayendo de él la información de configuración. La listanombres predeterminada del sistema es /dev/kmem.

## Consultar el WWN de la HBA en Solaris
```Bash
$ prtconf -vp | grep wwn
port-wwn: 21000003.ba19415b
node-wwn: 20000003.ba19415b
```

Visualizar las interfaces de red instaladas
Usamos el comando "dladm show-link". Ejemplo:
```Bash
# dladm show-link
ge0 type: legacy mtu: 1500 device: ge0
eri0 type: legacy mtu: 1500 device: eri0
ce0 type: legacy mtu: 1500 device: ce0
ce1 type: legacy mtu: 1500 device: ce1
```

## Configurar una interface de red en Solaris
Pasos previos:
1) Con "dladm show-link" podemos ver las interfaces que están instaladas en el sistema. Al
menos, la que el kernel ha detectado. Ejemplo:
```Bash
# dladm show-link
ge0 type: legacy mtu: 1500 device: ge0
eri0 type: legacy mtu: 1500 device: eri0
ce0 type: legacy mtu: 1500 device: ce0
ce1 type: legacy mtu: 1500 device: ce1
```

2) Con "ifconfig -a" podemos ver las interfaces que están activas y sus valores configurados.

Ejemplo:
```Bash
# ifconfig -a
lo0: flags=2001000849 mtu 8232 index 1
inet 127.0.0.1 netmask ff000000
ce0: flags=1000843 mtu 1500 index 2
inet 167.134.219.21 netmask ffffff00 broadcast 167.134.219.255
ether 0:14:4f:1e:70:f1
```

Nota: En nuestro ejemplo vemos que el sistema tiene una interface identificada como "ce1", pero que la misma no está configurada.
Configuración:
3) Levantar la interface de red "ifconfig {interface} plumb up" (donde {interface} es el identificador de la NIC que queremos levantar). Ejemplo (posteriormente vemos con "ifconfig" qe la NIC ha levantado y sólo falta asignarle su configuración):
```Bash
# ifconfig ce1 plumb up
# ifconfig -a
lo0: flags=2001000849 mtu 8232 index 1
inet 127.0.0.1 netmask ff000000
ce0: flags=1000843 mtu 1500 index 2
inet 167.134.219.21 netmask ffffff00 broadcast 167.134.219.255
ether 0:14:4f:1e:70:f1
ce1: flags=1000843 mtu 1500 index 3
inet 0.0.0.0 netmask ff000000
ether 0:14:4f:1e:70:f1
```

Nota: Hay varias manera de asignar los parámetros de la interfaces de red, una de ellas es usando el comando ifconfig (pero estos cambios no son permanentes) y otra es modificando los archivos de configuración (aquí los cambios serán permanentes). Aquí mostraremos la segunda:

4) Solaris puede mantener un archivo hostname por cada interface de red (NIC), para casos como éste, en el que tenemos varias interfaces de red (es sumamente importante el formato de nombre, sino la interface no tomará los parámetros).
```Bash
# vi hostname.ce0
167.134.219.21 netmask 255.255.255.0
# vi hostname.ce1
167.134.219.22 netmask 255.255.255.0
```

5) Indicar el nombre asociado a las direcciones IP en el /etc/inet/hosts y en el /etc/inet/ipnodes.
A continuación mostramos, a modo de ejemplo, el /etc/inet/ipnodes:
```Bash
# vi /etc/inet/ipnodes
#
# Internet host table
#
::1 localhost
127.0.0.1 localhost
167.134.219.21 metcam106 loghost
167.134.219.22 metcam106-adm
```

6) Es conveniente chequear la configuración del gateway por defecto y la tabla de rutas:
```Bash
# cat /etc/defaultrouter
167.134.219.1
# netstat -rn
Routing Table: IPv4
Destination Gateway Flags Ref Use Interface
-------------------- -------------------- ----- ----- ------ ---------
167.134.219.0 167.134.219.21 U 1 11 ce0
224.0.0.0 167.134.219.21 U 1 0 ce0
default 167.134.219.1 UG 1 9
127.0.0.1 127.0.0.1 UH 19 130187 lo0
```
