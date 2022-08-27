# Docker
Instalación de Docker
## Debian
```Bash
echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" | tee /etc/apt/sources.list.d/docker.list

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

apt update

apt install docker-ce docker-ce-cli containerd.io pigz

usermod -aG docker $USER

curl -L "https://github.com/docker/compose/releases/download/1.26.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

## Ubuntu, Mint y otras variantes
```Bash
echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" | sudo tee /etc/apt/sources.list.d/docker.list

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io pigz

sudo usermod -aG docker $USER

sudo curl -L "https://github.com/docker/compose/releases/download/1.26.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

# Comandos básicos Docker
## Descarga una imagen al repositorio local
```Bash
docker pull <imagen>
```
Ejemplo:
```Bash
docker pull postgres
```

## Ejecutar un contenedor
```Bash
docker run -d <imagen>
```

Ejemplo:
```Bash
docker run --name postgresDB -e POSTGRES_PASSWORD=******** -d postgres
```

## Iniciar un contenedor
```Bash
docker start <id_contenedor>
```

## Listar imagenes locales
```Bash
docker images
```

## Listar contenedores en ejecución
```Bash
docker ps
```
## Listar todos los contenedores en elsistema (incluidos los apagados)
```Bash
docker ps -a
```

## Ejecutar un comando en el contenedor
```Bash
docker exec -it <container-id> <comando> 
```

Ejemplo (inicia una shell bash en el contenedor):
```Bash
docker exec -it <container-id> bash 
```

## Detener un contenedor
```Bash
docker stop <id_contenedor>
```

Ejemplo:
```Bash
docker stop postgresDB
```

## Borrar o elimina el contenedor
```Bash
docker rm <id_contenedor>
```

Ejemplo:
```Bash
docker rm -f rstudio
```

#### Borrar la imagen
```Bash
docker rmi <imagen>
```

Ejemplo:
```Bash
docker rmi rocker/verse:latest
```

## Resplaldar el contenedor en un archivo
```Bash
docker export -o respaldo.tar nombre_contenedor
```

Tambien es valido:
```Bash
docker export nombre-contenedor > respaldo.tar
```

## Restaurar un contenedor desde el respaldo
```Bash
docker import respaldo.tar nuevo_nombre_contenedor
```

# LVM 

## Instalación 
```Bash
aptitude install lvm2
```

## Comandos de gestion
## Creando un Volume Group

1) Seleccionar el dispositivo:

1.a) Si es un disco o partición local.
```Bash
# fdisk -l
Disco /dev/sda: 146.1 GB, 146163105792 bytes
255 heads, 63 sectors/track, 17769 cylinders
Units = cilindros of 16065 * 512 = 8225280 bytes
Disk identifier: 0x00047557
Disposit. Inicio Comienzo Fin Bloques Id Sistema
/dev/sda1 * 1 122 979933+ 83 Linux
/dev/sda2 123 1095 7815622+ 83 Linux
/dev/sda3 1096 1338 1951897+ 82 Linux swap / Solaris
/dev/sda4 1339 17769 131982007+ 5 Extendida
/dev/sda5 1339 1581 1951866 83 Linux
/dev/sda6 1582 6444 39062016 83 Linux
/dev/sda7 6445 8037 12795741 83 Linux
/dev/sda8 8038 17769 78172258+ 83 Linux
```
Nota: En este caso asignaremos el /dev/sda8 para administrarlo con LVM

1.b) Si es un LUN asignado desde la SAN 
```Bash
# cat /proc/scsi/scsi
Attached devices:
Host: scsi0 Channel: 00 Id: 00 Lun: 00
Vendor: HITACHI Model: DF600F Rev: 0000
Type: Direct-Access ANSI SCSI revision: 03
# ls -la /dev/disk/by-id/
total 0
drwxr-xr-x 2 root root 240 feb 15 06:32 .
drwxr-xr-x 5 root root 100 feb 15 06:32 ..
lrwxrwxrwx 1 root root 16 feb 15 06:32 cciss-3600508b1001045395356423159580009 -> ../../cciss/c0d0
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part1 -> ../../cciss/c0d0p1
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part2 -> ../../cciss/c0d0p2
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part3 -> ../../cciss/c0d0p3
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part4 -> ../../cciss/c0d0p4
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part5 -> ../../cciss/c0d0p5
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part6 -> ../../cciss/c0d0p6
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part7 -> ../../cciss/c0d0p7
lrwxrwxrwx 1 root root 18 feb 15 06:32 cciss-3600508b1001045395356423159580009-part8 -> ../../cciss/c0d0p8
lrwxrwxrwx 1 root root 9 feb 15 06:32 scsi-1HITACHI_770141020124 -> ../../sda 
...
```
Nota: En este caso podemos ver que el disco puede ser identificado como /dev/sda

2) Inicializamos el dispositivo (disco o partición)
```Bash
pvcreate /dev/sda8
```

3) Creamos un 'Volume Group' llamado "vg" con este dispositivo. Se puede colocar una lista de dispositivos separados por un espacio en blanco, si previamente han sido inicializados.
```Bash
vgcreate vg /dev/sda8
```

4) Podemos ver los 'Volume Group' definidos con el comando vgdisplay
```Bash
vgdisplay
--- Volume group ---
VG Name vg
System ID
Format lvm2
Metadata Areas 1
Metadata Sequence No 1
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 0
Open LV 0
Max PV 0
Cur PV 1
Act PV 1
VG Size 74,55 GB
PE Size 4,00 MB
Total PE 19084
Alloc PE / Size 0 / 0
Free PE / Size 19084 / 74,55 GB
VG UUID i3ip9K-5yD9-36Vy-bcKJ-653v-bQz2-BmKQIF
```

## Crear un Volúmen Lógico (Logical Volume)
1) Para crear un Volumen Lógico (Logical Volume) llamado data de 1GB en este Volume Grup llamado "vg":
```Bash
lvcreate -n data --size 1g vg
```
2) Para formatear este volumen lógico como un File System ext3:
```Bash
mkfs.ext3 /dev/vg/data
```
3) Para montarlo. Ejemplo:
```Bash
mkdir /mnt/data
mount /dev/vg/data /mnt/data
```
4) Si se quiere que se monte automaticamente al iniciar el equipo, se debe editar el /etc/fstab y agregar una línea como esta:
```Bash
/dev/vg/data /mnt/data ext3 defaults 0 2
```

## Agregar un dispositivo a un Volume Group
1) Veamos la información previa de nuestro VG (el cual tiene 14,55GB libres)
```Bash
test03:/var/www# vgdisplay
--- Volume group ---
VG Name vg
System ID
Format lvm2
Metadata Areas 1
Metadata Sequence No 4
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 1
Open LV 1
Max PV 0
Cur PV 1
Act PV 1
VG Size 74,55 GB
PE Size 4,00 MB
Total PE 19084
Alloc PE / Size 15360 / 60,00 GB
Free PE / Size 3724 / 14,55 GB
VG UUID i3ip9K-5yD9-36Vy-bcKJ-653v-bQz2-BmKQIF
```

2) Para agregar un dispositivo al Volume Group usamos 'vgextend'
```Bash
test03:/var/www# vgextend vg /dev/sda6
Volume group "vg" successfully extended
```

3) Verificamos y podemos observar que ahora tiene 51,80 GB libres
```Bash
test03:/var/www# vgdisplay
--- Volume group ---
VG Name vg
System ID
Format lvm2
Metadata Areas 2
Metadata Sequence No 5
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 1
Open LV 1
Max PV 0
Cur PV 2
Act PV 2
VG Size 111,80 GB
PE Size 4,00 MB
Total PE 28620
Alloc PE / Size 15360 / 60,00 GB
Free PE / Size 13260 / 51,80 GB
VG UUID i3ip9K-5yD9-36Vy-bcKJ-653v-bQz2-BmKQIF
```

4) Para ver los dispositivos físicos que conforman el VG, usamos pvdisplay
```Bash
test03:/var/www# pvdisplay
--- Physical volume ---
PV Name /dev/sda8
VG Name vg
62 of 95
6/6/18 1:52 p. m.Manuales
file:///home/naimea/Documentos/AIT/HG/Platafor...
PV Size 74,55 GB / not usable 4,10 MB
Allocatable yes
PE Size (KByte) 4096
Total PE 19084
Free PE 3724
Allocated PE 15360
PV UUID DyzSNR-6NpR-Qz40-ZhZV-8KFv-8yDJ-64Xdtk
--- Physical volume ---
PV Name /dev/sda6
VG Name vg
PV Size 37,25 GB / not usable 2,50 MB
Allocatable yes
PE Size (KByte) 4096
Total PE 9536
Free PE 9536
Allocated PE 0
PV UUID vOT1ds-lYi2-2Wy3-xTsl-2ZDL-pfUW-0qIew8
```

## Ampliar el tamaño de un Volumen Lógico
1) Para ampliar un volumen lógico (darle más espacio en disco). Un ejemplo con /var
```Bash
lvresize -L+10G /dev/vg/var (sustituir 10G por el tamaño correcto, en este caso aumentamos
10GB)
```

2) Expandir el filesystem para que coincida con el tamaño del Volumen Lógico
Nota: haré la explicación con /var, por ser un filesystem complejo)

a) Desmontar el filesystem.
Nota, si algún proceso tiene tomado archivos dentro de este filesystem, no se podrá desmontar.
Para consultar los procesos que tienen archivos abiertos dentro de este filesystem usamos 'lsof':
```Bash
# lsof /var
COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME
syslogd 2582 root 2w REG 254,0 20480 1826852 /var/log/auth.log
syslogd 2582 root 3w REG 254,0 4096 1826899 /var/log/syslog
syslogd 2582 root 4w REG 254,0 706 1826846 /var/log/daemon.log
syslogd 2582 root 5w REG 254,0 79295 1826887 /var/log/kern.log
syslogd 2582 root 6w REG 254,0 0 1826912 /var/log/lpr.log
syslogd 2582 root 7w REG 254,0 1075 1826913 /var/log/mail.log
syslogd 2582 root 8w REG 254,0 0 1826910 /var/log/user.log
syslogd 2582 root 9w REG 254,0 1075 1826911 /var/log/mail.info
syslogd 2582 root 10w REG 254,0 1075 1826874 /var/log/mail.warn
syslogd 2582 root 11w REG 254,0 1075 1826878 /var/log/mail.err
syslogd 2582 root 12w REG 254,0 0 1826833 /var/log/news/news.crit
syslogd 2582 root 13w REG 254,0 0 1826834 /var/log/news/news.err
syslogd 2582 root 14w REG 254,0 0 1826832 /var/log/news/news.notice
syslogd 2582 root 15w REG 254,0 26765 1826858 /var/log/debug
syslogd 2582 root 16w REG 254,0 61440 1826888 /var/log/messages
xenstored 3092 root 3uW REG 254,0 5 442383 /var/run/xenstore.pid
xenstored 3092 root 14u REG 254,0 40960 2056202 /var/lib/xenstored/tdb
python 3097 root 2w REG 254,0 84 1826917 /var/log/xen/xend-debug.log
python 3097 root 12w REG 254,0 6184 1826916 /var/log/xen/xend.log
xenconsol 3099 root 3uW REG 254,0 5 442387 /var/run/xenconsoled.pid
python 3101 root 2w REG 254,0 84 1826917 /var/log/xen/xend-debug.log
python 3101 root 12w REG 254,0 6184 1826916 /var/log/xen/xend.log
cron 3163 root cwd DIR 254,0 4096 3055633 /var/spool/cron
cron 3163 root 3u REG 254,0 5 442393 /var/run/crond.pid
apache2 3177 root 2w REG 254,0 4096 1826845 /var/log/apache2/error.log
apache2 3177 root 7w REG 254,0 0 1826843 /var/log/apache2/other_vhosts_access.log
apache2 3177 root 8w REG 254,0 8192 1826841 /var/log/apache2/access.log
apache2 3178 www-data 2w REG 254,0 4096 1826845 /var/log/apache2/error.log
apache2 3178 www-data 7w REG 254,0 0 1826843 /var/log/apache2/other_vhosts_access.log
apache2 3178 www-data 8w REG 254,0 8192 1826841 /var/log/apache2/access.log
apache2 3184 www-data 2w REG 254,0 4096 1826845 /var/log/apache2/error.log
apache2 3184 www-data 7w REG 254,0 0 1826843 /var/log/apache2/other_vhosts_access.log
apache2 3184 www-data 8w REG 254,0 8192 1826841 /var/log/apache2/access.log
apache2 3193 www-data 2w REG 254,0 4096 1826845 /var/log/apache2/error.log
apache2 3193 www-data 7w REG 254,0 0 1826843 /var/log/apache2/other_vhosts_access.log
apache2 3193 www-data 8w REG 254,0 8192 1826841 /var/log/apache2/access.log
```
En este caso, como es de esperar, hay demonios del sistema que tienen ocupado el filesystem /var (principalmente por los logs, pero también hay otros casos como lo son los pid). Conclusión, parar estos servicios. En nuestro caso el servidor, ahora, está corriendo Xen, Apache, Crond y Syslogd. Como son servicios es mejor bajarlos que matar los procesos con un simple kill -9 aunque tambien funciona ;-)
```Bash
/etc/init.d/apache2 stop
/etc/init.d/cron stop
/etc/init.d/xend
/etc/init.d/sysklogd stop
```

También podemos ejecutar el lsof de esta forma:
```Bash
# lsof | grep var
acpid 2600 root 4u unix 0xffff88000173d940 5886 /var/run/acpid.socket
```

Así nos damos cuenta de que el demonio acpi, tambien tiene un socket en uso y paramos el demonio.
/etc/init.d/acpid stop

Verificamos nuevamente que nadie tiene ocupado el FS con lsof y de estar todo bien, desmontamos el FS
umount /var

b) Ampliar el filesystem
Nota: es conveniente chequear el filesysten, antes de hacer el 'resize'. Para chequearlo se usa:
```Bash
e2fsck -f /dev/vg/var
```
Si es ext2 o ext3
```Bash
resize2fs /dev/vg/var
```
Ejemplo:
```Bash
# e2fsck -f /dev/vg/var
e2fsck 1.41.3 (12-Oct-2008)
Paso 1: Verificando nodos-i, bloques y tamaños
Paso 2: Verificando la estructura de directorios
Paso 3: Revisando la conectividad de directorios
Paso 4: Revisando las cuentas de referencia
Paso 5: Revisando el resumen de información de grupos
/dev/vg/var: 77436/3932160 ficheros (2.9% no contiguos), 15728632/15728640 bloques

# resize2fs /dev/vg/var
resize2fs 1.41.3 (12-Oct-2008)
Resizing the filesystem on /dev/vg/var to 20971520 (4k) blocks.
El sistema de ficheros en /dev/vg/var tiene ahora 20971520 bloques.
```

3) Montar de nuevo el filesystem y levantar los servicios
```Bash
mount /var
```

4) Levantar los servicios que se hayan bajado

## Reducir en tamaño de un volumen lógico
NOTA: Es importante recordar reducir el tamaño del filesystem o lo que este residiendo en el volumen antes de encoger el volumen, sino se perderán datos (data). Un ejemplo con /var

1) Desmontar el filesystem:
```Bash
umount /var
```

2) Reducir el filesystem:
Si es Ext2/Ext3 (colocar el device correcto):
```Bash
e2fsck -f /dev/vg/var (siempre es recomendable chequear antes el filesystem)
resize2fs /dev/vg/var XX (XX es el nuevo tamaño)
```
Si es Reiserfs:
```Bash
resize_reiserfs -s-XXG /dev/vg/var (En este caso XX es el tamaño a restarle)
```
No es posible reducir Xfs ni Jfs.

3) Reducir el Volumen:
```Bash
lvreduce -L-XXG /dev/vg/var (Donde XX es el tamaño a reducir)
```
ó
```Bash
lvreduce -LXXG /dev/vg/var (Donde XX es el tamaño con el que quedará el volumen)
```

4)Volver a montar el filesystem:
```Bash
mount /var
```

## Eliminar un Volumen Lógico
```Bash
lvremove
```
Ejemplo:
```Bash
# lvremove /dev/vg/samba-swap
Do you really want to remove active logical volume "samba-swap"? [y/n]: y
Logical volume "samba-swap" successfully removed
```
