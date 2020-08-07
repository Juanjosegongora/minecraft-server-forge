---
title: "INSTALACION DE MINECRAFT SERVER CON FORGE EN LINUX"
author: [Juan José Góngora Contreras]
subject: "Markdown"
keywords: [Markdown, README]
lang: "en"
toc-own-page: "true"
---

NOTA: Las siguientes sentencias de comando es recomendable hacerlas como root, yo en los comandos NO pondré `sudo`

# INSTALACION Y CONFIGURACION DE JAVA

Para empezar se necesita java tanto como para iniciar el juego y para el servidor.
```
sudo apt install openjdk-8-jre-headless
```
Si tienes otra versión de java mas actualizada tendrás que usar la 8 para que el servidor funcione, podemos cambiarlo de la siguiente forma...
```
update-alternatives --config java
```
Sale algo parecido a esto...
```
  Selección   Ruta                                            Prioridad  Estado
------------------------------------------------------------
  0            /usr/lib/jvm/java-13-openjdk-amd64/bin/java      1311      modo automático
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      modo manual
  2            /usr/lib/jvm/java-13-openjdk-amd64/bin/java      1311      modo manual
* 3            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      modo manual

Pulse <Intro> para mantener el valor por omisión [*] o pulse un número de selección: 
```
En mi caso la versión 8 es el numero 3 y elijo esa opción.

# INSTALACION DE SERVIDOR MINECRAFT FORGE

Elegir una ruta donde instalarlo, se puede hacer en cualquier sitio, yo lo haré en `/opt/`
```
mkdir /opt/minecraft
cd /opt/minecraft
```
Ahora hay que descargar el archivo server de forge desde la pagina, yo haré el servidor con la versión `1.12.2` de minecraft porque actualmente tiene muchos mods. Pero en esa pagina encontramos todas las versiones.
```
https://files.minecraftforge.net/
```
Llegó el momento de la configuración, dentro de la carpeta donde esta nuestro server meteremos el archivo descargado y lo iniciaremos.
```
java -jar [NOMBRE_DE_NUESTRO_ARCHIVO] --installServer
```
Esto nos va a generar mas archivos, los de forge y también nos da el de minecraft server, ahora nos toca iniciar ese. Este archivo es parecido al otro, así que primero eliminar el archivo antiguo para evitar confusiones. (Como ejemplo el archivo que tengo que eliminar yo es `forge-1.12.2-14.23.5.2838-installer.jar`)
```
rm [NOMBRE_ARCHIVO_ANTERIOR]
```

Ahora iniciar el otro que en mi caso es `forge-1.12.2-14.23.5.2838-universal.jar`

```
java -Xms1024M -Xmx2000M -jar [NUESTRO_ARCHIVO] nogui  
```
Esto nos generará aun mas archivos, el server no se ha iniciado porque tenemos un archivo nuevo que se llama `eula.txt` que hay que modificar.
```
sed -i 's/false/true/' eula.txt
```
Volvemos a ejecutar la orden anterior...
```
java -Xms1024M -Xmx2000M -jar [NUESTRO_ARCHIVO] nogui  
```
Ahora se habrn generado mas archivos incluidos el `server.properties`

Resaltaré las lineas mas importantes...
- gamemode
    - El modo de juego, se pone con números
        - 0=Survival
        - 1=Creative
        - 2=Adventure
- max-player
    - Numero máximo de jugadores en el servidor (al mismo tiempo claro)
- server-port
    - El puerto donde escuchará por defecto tiene el `25565` recomiendo dejarlo así
- server-ip
    - La IP de la maquina donde estará el servidor.
- level-name
    - Nombre de la carpeta donde está el "mundo" si queréis un mundo nuevo dejarlo por defecto es la mejor opción, te generará uno automáticamente y lo meterá en una carpeta llamada default.
- online-mode
    - Tiene dos modos:
        - `true` para los que tienen minecraft comprado
        - `false` para ambos jugadores tanto los que compraron como los que no.

De esta forma tenemos minecraft server configurado, solo con iniciar el servidor ya podrías jugar
```
java -Xms1024M -Xmx2000M -jar [NUESTRO_ARCHIVO] nogui  
```

# AUTOMATIZACION Y CONFIGURACION INICIAL

Si queremos una forma mas fácil de iniciar minecraft tenemos la opción de crear un script.

## CREAR SCRIPT PARA INICIAR `MINECRAFT_SERVER`

Crearé un archivo en `/bin` es donde se alojan los comandos del sistema
```
cd /bin
nano minecraft_server
```
Dentro de ese archivo pondré lo siguiente
```
#! /bin/bash

# Ir a la carpeta server
cd [RUTA_DONDE_ESTA_SERVER]

# Iniciar el server
sudo java -Xms1024M -Xmx2000M -jar [NOMBRE_ARCHIVO_INICIACION_SERVER] nogui
```
Para mi caso se quedaría una cosa así...
```
#! /bin/bash

# Ir a la carpeta server
cd /opt/minecraft/

# Iniciar el server
sudo java -Xms1024M -Xmx2000M -jar forge-1.12.2-14.23.5.2847-universal.jar nogui
```
Con esto creado solo basta con poner el nombre del script en un terminal como `root`
```
minecraft_server
```
## CONFIGURACION DE `ops.json`

Este archivo contiene los miembros del servidor que tienen acceso para realizar comandos dentro del juego.

Para hacer que sea administrador, con el server iniciado en el terminal ponemos lo siguiente.
```
op playername
```
Y en el archivo nos generará algo parecido a esto.
```
[
  {
    "uuid": "005c74a1-755e-3894-b0f7-9077abefd15f",
    "name": "playername",
    "level": 4,
    "bypassesPlayerLimit": false
  },
```
Cuando esto este hecho ya se podra realizar cualquier comando de servidor desde el propio juego, con el chat.

## INSERCION DE MODS EN EL SERVER.

Para insertar mods en el servidor hay que descargarlos ojo con la versión de los mods que coincidan con la del server, y meterlos en `/opt/minecraft/mods/`

Ojo porque algunos mods necesitan librerías para ser utilizados, en este caso descargas las librerías y las metes en esa carpeta.

Los jugadores deberán tener los mismos mods para poder jugar en el servidor. Que para insertarlos seria ir a la carpeta `$HOME/.minecraft/mods`. Insertar aquí los mismos que al servidor y reiniciar server.

# CREACION DE UN SERVICIO CON SYSTEMD
Para facilitar el uso del servidor podemos crear un servicio con `systemd`.

Lo primero es lo siguiente, cogemos el script que habiamos creado antes y lo movemos a la carpeta de nuestro servidor, y de paso le cambiamos el nombre si queremos.
```
mv /bin/minecraft_server /opt/mincraft/run.sh
```
Lo siguiente es la creacion del servicio, que eso se hace generando un archivo en `/etc/systemd/system/`
```
touch /etc/systemd/system/minecraft.service
```
Con el siguiente contenido:
```
[Unit]
Description=Minecraft server
After=network.target

[Service]
User=minecraft
ExecStart=/opt/minecraft/run.sh

[Install]
WantedBy=multi-user.target
```
- Unit
  - Descripcion: Una breve descripcion de que se trata el servicio.
  - Afer=network.target: Aqui le indicamos que ese servicio sera iniciado una vez la tarjeta de red este iniciada.
- Service
  - User: El usuario que controlara ese servicio, se puede usar uno local del sistema o crear un usuario que controle ese servicio.
  - ExecStart=/opt/minecraft/run.sh: Aqui lo que le estamos indicando es que va a hacer el servicio para iniciar el servidor de minecraft, que simplemente pues iniciar el script que habiamos movido a la carpeta del juego antes, simplemente le damos la ruta.
- Install
  - WantedBy=multi-user.target: esto se encarga de hacer que podamos usar el comando `systemctl` o `service` directamente sin tener que configurar enlaces simbolicos, es mucho mas facil ponerlo.

Ahora toca controlar ese servicio que lo podemos hacer con el comando `systemctl`.
```
systemctl [parametro] minecraft
```
Los parametros pueden ser:
- start: Iniciar servicio.
- stop: Parar servicio.
- restart: Reiniciar servicio.
- status: Ver el etado del servidor.
- enable: Hacer que se ejecute automaticamente cuando el sistema se inicie.
- disable: Hacer que no se ejecute automaticamente cuando el sistema se inicie.

# COPIAS DE SEGURIDAD.
Imaginemos que hay algun mod que no se ha metido aun o simplemente queremos hacer una copia de seguridad del juego, para eso podemos generar un script que al ejecutarlo nosotros cree una copia de seguridad.

Podemos llamar al archivo `backup.sh` y meterlo en nuestra carpeta personal por ejemplo.
```
touch $HOME/backup.sh
```
Lo primero en el script sera comprobar si la carpeta donde queremos guardar la copia existe y sino, que lo cree. 

Por ejemplo yo escogi la ruta `$HOME/minecraft_backup` que es en la carpeta personal de ese mismo usuario una carpeta que se llame `minecraft_backup`.
```
# COMPROBAR SI EXISTE LA CARPETA DONDE GUARDAR LAS COPIAS
RUTA=$HOME/minecraft_backup
if test -d $RUTA; then
  echo
else
  mkdir $RUTA
fi
```

Lo siguiente seria parar el servidor para que haga una copia sin que se pueda modificar nada en lo que la hace.
```
# PARAR SERVICIO DE MINECRAFT
systemctl stop minecraft
```

Lo siguiente es realizar la copia, que de la siguiente forma la copia se guardara con el nombre y la fecha y hora de ese momento, para saber cuando se hizo.
```
# REALIZAR COPIA DE SEGURIDAD
FECHA=$(date -Idate)
tar cvzf $RUTA$FECHA.tar.gz /opt/minecraft
echo "La copia se realizó correctamente"
```

Lo siguiente seria volver a iniciar el servidor o no, que podemos aprovechar para poner algo nuevo y luego iniciarlo a mano, o si es solo copia le decimos que si y lo inicia solo.

```
# INICIAR EL SERVIDOR
read -n1 -p "Do you want to start the server? (y/n)" RESP
if [ $RESP = y ]; then
	echo "starting server..."
	service start minecraft
	echo
	echo "Ready"
else
	echo 
	echo "Ready"
fi
```

El script completo seria el siguiente.
```
#! /bin/bash

# COMPROBAR SI EXISTE LA CARPETA DONDE GUARDAR LAS COPIAS
RUTA=$HOME/minecraft_backup
if test -d $RUTA; then
  echo
else
  mkdir $RUTA
fi

# PARAR SERVICIO DE MINECRAFT
systemctl stop minecraft

# REALIZAR COPIA DE SEGURIDAD
FECHA=$(date -Idate)
tar cvzf $RUTA$FECHA.tar.gz /opt/minecraft
echo "La copia se realizó correctamente"

# INICIAR EL SERVIDOR
read -n1 -p "Do you want to start the server? (y/n)" RESP
if [ $RESP = y ]; then
	echo "starting server..."
	systemctl minecraft start
	echo
	echo "Ready"
else
	echo 
	echo "Ready"
fi
```