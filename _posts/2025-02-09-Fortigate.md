---
title: FORTIGATE (firewall)
date: 2025-02-09 14:10:00 +0800
categories: [fortinet, firewall]
tags: ["fortigate, fortinet, firewall"]     # TAG names should always be lowercase
---
## Comandos basicos Fortigate

- `get system interface;` para saber la ip y mas detalles de cada interface como por ejemplo si está en DHCP o con IP Estatica.
- `show;` para ver los cambios que se hicieron a los valores por defecto.
- `get;` para ver todos los cambios o configuraciones posibles que se pueden modificar.
- `show full-configuration;` es para ver todos los valores (mas completo que get).
- `get system performance status;` para ver el uptime del equipo y así fijarse cuanto tiempo lleva encendido en la ultima linea y otro datos como el porcentaje de CPU utilizado o la memoria utilizada, el promedio uso de la red y otros mas.
- `diagnose;` se utiliza para trabajar con debuging donde se encuentren problemas de conexion.
- `grep;` al igual que en linux, el comando grep sirve para filtrar informacion que queremos por pantalla por ejemplo; show system interface | grep port1; en este caso mostrará la linea que tenga la palabra port1, pero es mas util la flag (bandera) -f ya que muestra todo el contexto de configuracion en donde se encuentre la palabra port1 por ejemplo; show system 
interface \| grep -f port1; al utilizar la flag -i el comando grep queda key insensitive que significa que mostrará tanto la palabra esté en mayusculas o minusculas y un comando bastante comun es juntar la flag -if para buscar las posibles palabras que esten en minusculas o mayusculas.
- `get | grep -if "SITE-A";` con este comando podemos ver donde se encuentra el contexto para cambiar el nombre del equipo siendo "SITE-A" el nombre actual del equipo y luego de mostrarlo podriamos ir al contexto para cambiar el nombre y así se puede utilizar cuando no sepamos exactamente donde se encuentra lo que tenemos que modificar que en este caso muestra config system global.

## Activar HTTP debido a que los Fortigate vienen solo con SSH Y HTTPS habilitado

- `config system interface;` para acceder a la configuracion de las interface.
- `edit port1;` para entrar al puerto 1 y poder habilitar HTTP en ese puerto.
- `show;` para ver un breve resumen del puerto y se puede apreciar que se encuentra habilitado en set allowaccess el PING, HTTPS, SSH FGFM (fgfm es para administrar el fortigate desde el puerto fortimanagment).
- `append allowaccess http;` para habilitar http en ese puerto y poder entrar a la configuracion mediante GUI y ahora con un show se puede ver HTTP.
- `end;` es para grabar la configuracion modificada y vuelve al contexto principal.
- `next;` es para grabar la configuracion pero queda en el contexto anterior al guardado.
- `exec ping google.com;` para hacer ping.
(se desactiva default gateway from server y override internal DNS para que el laboratorio no tenga internet por las interfaces port1 de ambos fortigate)

en system luego en setting modificaremos la zona horaria correspondiente y modificamos en esa misma pagina el Idle timeout para que no se bloquee la sesion luego de inactividad poniendo el maximo de minutos (480) y activar la opcion auto file system check por si se apaga mal el equipo al iniciarlo intentará a nosotros hacerle un check system al sistema y con esa casilla activada el equipo lo hará

- `get system status;` nos mostrará la version del equipo, la version de la BD de antivirus, el serial number del equipo, el estado de la licencia, la fecha de expiracion de la licencia, la informacion de los recursos  de CPU RAM, el hostname del equipo, si está en modo NAT o bridge, la fecha y hora y por ultimo la razon del ultimo reinicio del equipo.

## Para cambiar ip dinamica (DHCP) a estatica del puerto de administracion (port1)

- `config system interface;`
- `edit port1;`
- `set mode static;` para cambiar de modo DHCP a estatica
- `show;` mostrará la direccion ip que quedó con el DHCP
- `set ip X.X.X.X/24;` se ingresa la ip que se quiera asignar en este caso (192.168.1.51/24)
- `show;` mostrará la direccion ip ingresada
- `end;` para guardar los cambios

## Asignar IP a interfaces

- `config system interface;`
- `edit port2;`
- `set ip X.X.X.X/30;`
- `set alias ISP-1;` para asignarle un alias a la interface
- `set role wan;` para asignarle un rol que en este caso es wan
- `next;` para guardar y poder editar el siguiente puerto
- `edit port3;`
- `set ip 45.32.12.1/30;`
- `set alias ISP-2;`
- `set role wan;`
- `next;`
- `edit port4;`
- `set ip 10.0.2.254/24;`
- `set allowaccess ping;` para poder habilitar la funcion PING
- `set alias LAN;`
- `set role lan;`
- `end;`

## Asignar IP a interfaces mediante GUI

Hacemos doble click en la interfaz que se quiera configurar en Network y luego Interfaces
![Untitled](/assets/img/fortigate/forti01.png)

Le asignamos un "Alias (ISP 2)", indicamos que es WAN y le asignamos la direccion IP de esa interface
![untitled](/assets/img/fortigate/forti02.png)

Le configuramos una IP al puerto 4 que en este caso el "alias" es LAN, en el role tambien le indicamos LAN y le ingresamos la IP 10.0.1.254/255.255.255.0 y en accesos administrativos le habilitamos el PING para poder hacer ping desde el servidor que se encuentra en esa red
![untitled](/assets/img/fortigate/forti03.png)

## asignar ruta estatica mediante CLI para poder habilitar la navegacion por internet llamada Default Route

- `config router static;` para entrar a la configuracion de rutas estaticas
- `show;` para ver si hay rutas estaticas
- `edit 1;` para crear la primera ruta estatica
- `show;` para ver y no saldrá que tenemos nada configurado en esa ruta creada
- `get;` para ver las opciones que tenemos para configurar
- `set gateway 60.89.123.2;` para configurar el gateway
- `set device port2;`
- `show;` para ver los cambios
- `next;`
- `edit 2;`
- `show;`
- `set gateway 45.32.12.2;`
- `set device port3;`
- `set distance 15;` para que sea prioridad el ISP1 (entre menor distancia mayor prioridad) por el puerto 2 y en caso de que falle se activa el puerto 3 ISP2
- `end;`

## asignar las Default Route mediante GUI

Ingresamos la primera default Route en "Create New"
![untitled](/assets/img/fortigate/forti04.png)

y acá como es para salida de internet se le pone en el "Destination" 0.0.0.0/0.0.0.0, en el Gateway Address la IP del Gateway que en este caso la IP de la interfaz de salida 200.212.31.2 y automaticamente se asignará la "Interface" 200.212.31.1 en el numero del puerto que se encuentra esa IP de salida, en la "Administrative Distance" es para ver cual interface es principal y cual es de backup (la menor en distancia administrativa es la principal y la mayor es la de backup en caso de que se interrupa la otra interfaz) en este caso el puerto 2 es el principal con una distancia de 10 y el puerto 3 (ISP 2) es la de backup.
![untitled](/assets/img/fortigate/forti05.png)

En este caso en la distancia administrativa se le ingresa 15 para que este sea de backup en caso de que el ISP1 falle
![untitled](/assets/img/fortigate/forti06.png)

## Backups & restores por interfaz grafica de usuario (GUI)

En la parte superior derecha en "Configuration" seleccionamos Backup y se descargará un archivo de texto con el nombre del HOSTNAME, la version de Fortiguet, la fecha y la hora en que se hizo el backup y al abrirlo está la configuracion del Fortinet 
![untitled](/assets/img/fortigate/forti07.png)

Podemos Encriptar el archivo con una contraseña y descargarlo en texto plano en el computador
![untitled](/assets/img/fortigate/forti08.png)

en el archivo de backup si se quiere copiar la configuracion a otro Fortiget se tiene que cerciorar que en la primera linea aparezca el mismo nombre del archivo Backup Fortiget de destino y en ese caso se recomienda hacer un backup del destino para ver la primera linea y luego copiar esa primera linea y pegarla en la linea del archivo que queramos copiar y tambien se tiene que fijar que las interfases sean la misma cantidad y de ser menor cantidad la del destino se tiene que eliminar el nombre de las interfaces para que calcen igual
![untitled](/assets/img/fortigate/forti09.png)

para restaurar el dispositivo se realiza lo siguiente en Configuration en Restore y seleccionamos el archivo de backup que queremos restaurar y en caso de que le hallamos puesto contraseña se ingresa la contraseña donde dice "Password" y luego de que el equipo se reinicia ya se encuentra restaurado
![untitled](/assets/img/fortigate/forti10.png)

## Revision por interfaz grafica (GUI)

Es mas que nada un backup pero queda guardada en el mismo equipo aunque no es recomendable tomarlo como backups debido a que si el equipo falla no se podrá sacar la configuracion del equipo. Pero si se puede utilizar para ver si se pide a alguien que haga tal tarea se puede comparar entre 2 backups cuales fueron los cambios que se realizaron en el equipo. Para ingresar en la seccion "Revision" se ingresa en admin, luego "Configuration" y "Revision". Para la primera revision se hace click en Save Changes y con esto guarda la configuracion en un backup y al crearla con el boton derecho se puede poner "Revert" para aplicar ese backup
![untitled](/assets/img/fortigate/forti11.png)

para ver el cambio, cambié el alias del ISP del puerto 2
![untitled](/assets/img/fortigate/forti12.png)

ahora ingreso en revision para guardar una segunda revision con los cambios realizados 
![untitled](/assets/img/fortigate/forti13.png)

y al seleccionar las 2 revision y al hacer click en "Diff" se verá en rojo la configuracion antigua y en verde el cambio que se realizó en la ultima configuracion (modificacion del nombre del puerto 2)
![untitled](/assets/img/fortigate/forti14.png)

ahora haremos una aplicacion de una revision para volver a como estaba el nombre del puerto haciendo click en la revision que queremos y seleccionar "Revert" y con eso reiniciará el equipo con esa configuracion avisandonos siempre que el equipo se reiniciará
![untitled](/assets/img/fortigate/forti15.png)
nota al aplicar los backups no alteran o borran las revisiones que se hicieron en ese rango de tiempo es por eso que es buena practica tener revisiones pero importante descargarlas por si equipo se daña

## Backups & restores por interfaz linea de comando (CLI)

execute backup config ftp SITE-A-25.12.2024 200.212.31.2 ftp ftp; donde el primer ftp es para indicar que el archivo se copiará mediante el protocolo FTP, luego pide el nombre del archivo que será creado que en este caso SITE-A-25.12.2025 le indiqué el hostname y la fecha en que se realizó el backup luego indica la direccion IP del FTP que en este caso es la de ISP-1 200.212.31.2 y los 2 ultimos ftp son el primer el usuario ftp y el segundo la password que tambien es ftp y listo
![untitled](/assets/img/fortigate/forti16.png)

para restaurar es lo mismo pero se reemplaza backup por restore y luego se reiniciará para cargar el archivo de configuracion nuevo
execute restore config ftp SITE-A-25.12.2024 200.212.31.2 ftp ftp
![untitled](/assets/img/fortigate/forti17.png)

## Gestionando usuarios administradores

en "System, Admin Profiles" encontraremos los perfiles que detalla que permisos le daremos a cada usuario y eso se crea en "Create New" y ahí le asignamos que puede hacer y que no puede hacer. En "Permit usage of CLI diagnostic commands" es para permitir o no el uso de los comandos de diagnosticos por CLI y el "Override Idle Timeout" es para desconectar cada cierta cantidad de minutos la sesion.
![untitled](/assets/img/fortigate/forti18.png)

ahora vamos a crear un usuario para agregarlo a ese perfil creado en "System, Administrators" y click en "Create New" y seleccionar "Administrator"
![untitled](/assets/img/fortigate/forti19.png)

le daremos un perfil predeterminado donde solo pueda leer las configuraciones
![untitled](/assets/img/fortigate/forti20.png)

luego se podria habilitar el segundo factor de autenticacion (Two-factor Authentication), tambien se puede restringir a que solo se conecte desde una red especifica (Restrict login to trusted hosts) de la red empresarial que en este caso podria ser la 10.0.1.0/24 para así impedir que se conecte desde internet o de una red que sea insegura y por ultimo restringir el admin a un grupo de huesped unicamente (Restrict admin to guest account provisionaling only)
![untitled](/assets/img/fortigate/forti21.png)

para poder crear el segundo factor de autenticacion se realiza una vez ya creado el usuario y se tiene que editar ese usuario para que pueda habilitar por la CLI la creacion del segundo factor de autenticacion
![untitled](/assets/img/fortigate/forti22.png)

al realizar un get se ve la variable "two-factor" que se encuentra disable y que tenemos que editar
get two-factor; y al presionar el signo de pregunta se veran todas las opciones de autenticacion que se encuentran disponibles. El fortitoken es una aplicacion que viene con el fortigate para tener la segunda autenticacion, el sms es para hacerlo mediante mensaje de texto tambien el email para poder tener el codigo como segundo factor de autenticacion siendo este gratis a diferencia de fortitoken que es con la licencia comprada del fortigate
![untitled](/assets/img/fortigate/forti23.png)

## Primera policy (politica) para poder permitir que los host salgan a internet GUI

Se crean y configuran en "Policy & Objects" donde "Firewll Policy" se crean y configuran las politicas y "Addresses, Internet Services Database, Services Schedules, Virtual IPs, IP Pools, Protocol Options" son Objetos y esos objetos luego se podrian agregar a politicas. Ahora haremos una politica que permita el trafico de internet desde la LAN hacia internet ya que Fortiget trabaja denegando con la regla implicita que viene y se tiene que permitir lo que uno quiera agregar
![untitled](/assets/img/fortigate/forti24.png)

tenemos que crear una regla que se encuentre por encima de esta regla para que pueda generar trafico a internet. Es recomendado tener una vista de las politicas por par de interfaces para así poder entender con mayor claridad como se encuentran las politicas por interfaces de entrada y por interfaces de salidas y eso se selecciona con "Interface Pair View". Para crear una nueva politica se hace en "Create New" y en el primer campo "Name" se le asigna un nombre identificativo a la politica creada, en el segundo campo "Incoming Interface" es el numero de puerto en donde ingresará o recibirá el trafico el Fortiget para esa politica (en este caso port4 LAN), en "Outgoing Interface" es por donde saldrá el trafico de esta politica (que en este caso port2 ISP1), en "Source" se podria especificar que usuario, direccion IP o que equipo será el que enviará el trafico (en este caso le pondremos el objeto "all" que corresponde a cualquiera), en "Destination" tambie pondremos el objeto "all" debido a que queremos permitir llegar a todo internet, en "Schedule" es el rango u horario que queremos que funcione está politica (en este cado seleccionamos el objeto "all"), en "Services" se refiere a puertos y servicios que hayamos definidos en la seccion de "Services" de la parte de objetos y en este caso ocuparemos "all", en "Action" indicamos si está politica se deniega o se acepta y en este caso pondremos accept, "Firewall/Network Options" como es una salida a internet tendremos que natear la direccion de origen para que todas las peticiones se hagan con la direccion ip de la interfaz del Fortigate y podriamos poner una IP o utilizar un pool de direcciones IP en caso de tenerlas, en "Logging Options" seleccionaremos que logee todo el trafico en todas las sesiones "All Sessions", que genere logs cuando inicien las sesiones "Generate Logs when Session Starts" y por ultimo habilitar la politica en "Enable this Policy"
![untitled](/assets/img/fortigate/forti25.png)
![untitled](/assets/img/fortigate/forti26.png)

## Politica para permitir que los hosts salgan a internet por CLI

- `config firewall policy;` para entrar en el contexto de politicas
- `show;` para ver si se encuentra alguna politica creada y en este caso ninguna
- `show full-configuration;` tampoco vemos nada creado
- `edit 1;` para crear la primera politica 
- `get;` como ya creamos la politica ahora saldran posibles configuraciones 
- `set name INTERNET;` para crear el nombre de la politica
- `set srcintf port4;` significa el puerto de entrada de trafico (LAN), en la GUI esta como incoming interface
- `set dstintf port2;` la interfaz de salida (ISP1), en la GUI esta como Outgoing Interface
- `set srcaddr all;` el origen del trafico, en la GUI esta como Source
- `set dstaddr all;` el destino del trafico, en la GUI esta como Destination
- `set schedule always;` el rango horario que está habilitada la policy 
- `set service ALL;` todos los servicios y puertos
- `set action accept;` para que la politica permita
- `set nat enable;` para habilitar el NAT 
- `show full-configuration;` para verificar que inspection-mode flow se encuentre en flow
- `end;` ya con esto guardamos y creamos la politica mediante CLI
![untitled](/assets/img/fortigate/forti27.png)

## Configurando rutas estaticas en red LAN

tenemos la siguiente topologia y tenemos que configurar primero las interfaces de los fortigate en el puerto 7 de ambos equipos;
![untitled](/assets/img/fortigate/forti28.png)

Solo dejaré la configuracion del fortigate del lado de "legales", en este caso habilitamos PING para poder probar la conexion, y le asignamos la ip 172.10.10.1/24 al puerto 7 con el alias "LEGALES" e indicamos el role LAN;
![untitled](/assets/img/fortigate/forti29.png)

Ahora crearemos dos objetos "Addresses" para despues utilizar estos objetos en todas las configuraciones y así tener un orden en las configuraciones, el primer objeto será la red vecina "CONTABILIDAD" le cambié el color, de tipo (Type) Subnet, IP/Netmask la ip de la red de CONTABILIDAD (recuerda que esta red que estoy configurando es "LEGALES" con la IP 10.0.1.0/24 y en esta parte estoy agregando la red vecina que necesito configurar para poder llegar a ella), Static route configuration lo habilitaremos para poder despues este objeto utilizarlo en el ruteo de la red y con esto cuando necesitemos modificar la red solo modificamos el objeto correspondiente y en todas las demas configuraciones que se encuentra ese objeto se modificará automaticamente; 
![untitled](/assets/img/fortigate/forti30.png)

Ahora crearemos un segundo objeto llamado "RED LOCAL", con la ip de la red 10.0.1.0/24, y tambien le habilitamos el Static route configuration para poder utilizar este objeto en el ruteo de la red
![untitled](/assets/img/fortigate/forti31.png)

Ahora volvemos a Network y en Static Routes agregamos la red "CONTABILIDAD" para indicarle al Fortiget donde se encuentra esa red y así poder comunicarse agregando el objeto recien creado "CONTABILIDAD" con la IP del Gateway 172.10.10.2 y automaticamente aparece la interface "LEGALES (port7)" que es la interfaz del fortiget local llamada legales
![untitled](/assets/img/fortigate/forti32.png)

En caso de no agregar el objeto se tiene que hacer en vez de Named Address en Subnet y asignar la IP correspondiente a la red vecina y el Gateway de como llegar a esa red pero es mas recomendable y limpio hacerlo mediante objetos Address
![untitled](/assets/img/fortigate/forti33.png)


