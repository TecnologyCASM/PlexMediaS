# Mi HomeLAB con Plex en Raspberry PI 4B con Docker-Compose:
![image](https://github.com/TecnologyCASM/MiHomeLAB-CASM/assets/107158068/1cf71482-f40d-4716-ba3c-0023a0ddce0c)

Con este proyecto, podras crear tu propio servidor de descargas de peliculas y series de forma automatica, moviendo estas de formorma automatica al directorio `media/`, donde PLEX las agregara su bibiliteca para que la puedas disfrutar. Todo el tutorial es explicado en el canal de `Pelado Nerd` => [Youtube](https://www.youtube.com/playlist?list=PLqRCtm0kbeHCEoCM8TR3VLQdoyR2W1_wv)

# Prerequisitos:
* `Raspberry Pi 4 Modelo B 2GB`. https://amzn.to/3K7diXR
* `64GB tarjeta MicroSD`. https://amzn.to/3ynPiNz
* `Lector de tarjetas SD USB`. https://amzn.to/3wGN8bs
  
Nota: Dicho esto, este proceso debería funcionar en cualquier Raspberry Pi. La carpeta Docker-Container contiene otros contenedores en caso de querer instalarlos, como son Portainer, Plex y Nextcloud.

# Instalacion de Sistema Operativo en Raspberry Pi:
1) Descarga la aplicacion "[RaspberryOS](https://www.raspberrypi.com/software/)" de la pagina oficial.
2) Conecta a la PC el lector SD con la memoria micro y segue los pasos como se muestra en la imagen mas abajo.
3) Elige el sistema operativo recomendado por raspberry.
![image](https://github.com/TecnologyCASM/PiHoleUnbound/assets/107158068/4173438b-eca6-497a-85d0-ec96bf698629)
![image](https://github.com/TecnologyCASM/PiHoleUnbound/assets/107158068/3a84ef2b-4204-4585-a62f-6c5adf6b9236)
4) Una vez instalado el sistema operativo en la raspberry, conecta esta a la red via cable y conecta un monitor para completar las configuraciones iniciales.
5) En este paso, debemos realizar los ajustes de `Network`, ya que estaremos brindando los servicios mencionados, debemos fijar un direccionamiento IP al equipo, donde estaremos ejecutando el siguiente comando, para que nos arroje un asistente:
```shell
sudo nmtui
```
![image](https://github.com/TecnologyCASM/PiHoleUnbound-WG/assets/107158068/35b590d2-8eab-44af-9d5b-ab48f9270ff5)

6) En este asistente, estaremos eligiendo la opcion `Modificar una conexion`, luego debemos elegir la terjeta de red que dice `Conexion Cableada 1` y presionamos editar:
![image](https://github.com/TecnologyCASM/PiHoleUnbound-WG/assets/107158068/cc2a65fd-be60-46c5-9aec-0b39bc97c184)
10) Alli aplicamos los ajuste de direccionamiento estatico que sean necesarios, como son:
  * Direcciones: En esta opcion debemos colocar la IP estatica que debera llevar la raspberry, el cual debe llevar el siguiente formato `192.168.1.10/24`.
  * Puerta de Enlace: En esta opcion debemos colocar la IP de nuestro router de internet, el cual se encuentra en el mismo segmento mensionado mas arriba, por ejemplo `192.168.1.1`.
  * Servidores de DNS: En esta opcion colocamos los DNS de nuestra eleccion, en mi caso coloco los de google `8.8.8.8` y `8.8.4.4`.
  * Busqueda de Dominio: Esto es cuando estamos trabajando con un dominio, por lo que es opcional.
7) Despues que la Raspberry Pi inicie el OS despues del reinicio, entonces procederemos con los ajustes de algunos parametros en esta, para ello debemos iniciar el siguiente asistente, con el comando:
```shell
sudo raspi-config
```
Este comando nos llevara a la siguente pantalla:
![image](https://github.com/TecnologyCASM/PiHoleUnbound-WG/assets/107158068/c138d6d4-2f87-4468-bd1f-2c13102bac31)

8) En este asistente, tendremos que realizar los siguientes ajustes:
  *  `1-System Options=>Boot/Auto Login=>Desktop Autologin`: Opcion utilizada para que la rasperrypi no solicite credenciales al iniciar sesion. (Solo Raspberry Pi Desktop).
  *  `2-Display Options=>VNC Resolutions`: Opcion utilizada para elegir la resolucion de la pantalla de conexion remota a la raspberry pi. (Solo Raspberry Pi Desktop). 
  *  `3-Interface Options=>VNC`: Opcion habilitada para conexion remota por GUI a la raspberry pi. (Solo Raspberry Pi Desktop).
  *  `5-Localisation Options`:
      - Locale: Configuracion del idioma. En mi caso es `es_DO.UTF-8`.
      - Timezona: Configuracion de zona horaria. En mi caso es `America/Santo_Domingo`.
      - Keyboard: Configuracion del Teclado. Solo elegir para autoreconocimiento.
      - WLAN Country: Configuradion de la ciudad para la red wireless. En mi caso `DO Dominican Republic`.
  *  `6-Advanced Options=>Expand FileSystem`: Opcion para expandir por completo el almaceniamiento de la SD de la raspberry.

 Nota: Para que estos ajustes se apliquen debemos presionar `Finish` y este solicitara un reinicio del equipo.
 
# Proceso Actualiacion OS, Instalacion de Docker y agregar usuario a grupo docker: 
1) Una vez la raspberry pi halla iniciado, procederemos a aplicar los siguientes comandos:
   - Actualiazar la lista de repositorios, OS e instalacion de binarios:
```shell
sudo apt update && sudo apt-get full-upgrade -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common \
     vim \
     fail2ban \
     ntfs-3g
```
   - Proceder con la instalacion de Docker:
```shell
sudo sudo curl -fsSL https://get.docker.com/ -o get-docker.sh && sudo sh get-docker.sh &&
sudo usermod -aG docker ${USER} && sudo reboot
```
   - Agregar el usuario al grupo SUDO:
```shell
sudo useradd ${USER} -G sudo
```
2) Una vez la raspberry pi halla iniciado y para validar que el servicio de docker esta instalado, procederemos a descargar un contenedor de prueba llamado `Helo-Wold`:
```shell
docker run hello-world
```
![image](https://github.com/TecnologyCASM/PiHoleUnbound-WG/assets/107158068/58f35f2b-9c35-4381-8186-8f37298e170a)

# Proceso de Montar disco duro Externo:
1) Abriremos la terminal y nos colocaresmos en modo root con el siguiente comando:
```shell
sudo su
```
![image](https://github.com/TecnologyCASM/MiHomeLAB-CASM/assets/107158068/09e483c7-5a33-4f33-8b27-3e6d828273b0)

2) Comando para visualizar el disco que necesitamos montar:
```shell
lsblk
```
![image](https://github.com/TecnologyCASM/MiHomeLAB-CASM/assets/107158068/117d28ef-6b14-4a0f-be62-867266c70da2)

```shell
fdisk -l
```
![image](https://github.com/TecnologyCASM/MiHomeLAB-CASM/assets/107158068/26ed80dd-72c5-47fb-bf34-3a911eef2b1d)

3) El siguiente comando es para poder visualizar cual es el UUID que identifica el disco:
```shell
ls -l /dev/disk/by-uuid/
```
![image](https://github.com/TecnologyCASM/MiHomeLAB-CASM/assets/107158068/94a31d18-0fd6-480b-82c8-35bbb14e5e86)

4) A continuacion proceso de montar el disco agregando la sigueinte entrada en la ruta `/etc/fstab`:
```shell
UUID="029EF4C89EF4B4EF"  /mnt/storage  ntfs-3g  defaults,auto  0  0 
```
![image](https://github.com/TecnologyCASM/MiHomeLAB-CASM/assets/107158068/efe03d0a-ad63-4b6a-89f1-e591daaf1fdc)

luego de esto solo resta reiniciar.
