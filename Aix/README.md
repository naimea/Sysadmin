Para crear colas de impresión en AIX se tienen dos métodos, uno vía comandos y otra a través de la herramienta 'System Management' (smit). En ambos casos se requiere que el nombre de la impresora esté en los DNS referenciados por el servidor o éste este registrado en el /etc/host. De lo contrario, el nombre de la cola quedará asociado a la dirección IP, lo que resulta incómodo ya que los comandos sólo mostrarán el nombre de la cola hasta el primer punto, lo que hará que muchas colas de impresión a simple vista no puedan diferenciarse por el usuario.
Una vez asociado el nombre de la impresora a la IP, se ejecuta:
```Bash
# > /usr/lib/lpd/pio/etc/piomkjetd mkpq_jetdirect -p 'generic (PCL Emulation)' -D pcl -q 'PR3_VIG3' -h 'PR3_VIG3' -x '9100'
```
donde,
-p 'generic (PCL Emulation)' # Representa el driver a utilizar
-q 'PR3_VIG3' # Representa el nombre de la cola
-h 'PR3_VIG3' # Representa el nombre de la impresora (en el /etc/host o en el DNS)
-x '9100' # Representa el puerto de impresión (en este caso es el estandar JetDirect)

Para crearla por el 'System Management', se escribe:
```Bash
# > smit
```
Luego se sigue las opciones (se desplaza con las flechas y se selecciona con "Enter"):
"Print Spooling" --> "AIX Print Spooling" --> "Add a Print Queue"

Posteriormente se colocan los datos de la cola según el caso. Con la herramienta 'System Management' se pueden realizar todas las actividades administrativas sobre las colas de impresión siguiendo los menús.

Para monitorear el estado de las colas por comando (vía rápida), usamos:
```Bash
# > lpstat -W
```
Para habilitar una cola de impresión, usamos:
```Bash
# > enable {nombre de la cola}
```
Nota: Si la cola esta en estado "DOWN", pero tiene trabajos no despachados por algún problema, la misma no pasa a estado "READY". Igualmente, no se puede eliminar una cola de impresión que tenga trabajos encolados.
