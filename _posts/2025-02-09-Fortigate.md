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

Ahora crearemos la politica para el evío de trafico y el recibimiento de trafico de esa red vecina, en Name pondremos "TO_CONTABILIDAD" debido a que queremos enviar A contabilidad el trafico por lo tanto recibiremos el trafico desde (Incoming Interface) "LAN(port4)" y lo sacaremos (Outgoing Interface) por el puerto de salida "LEGALES(port7)", el origen (Source) será el objeto creado "RED LOCAL", el destino (Destination) será CONTABILIDAD, junto con las otras configuraciones de la imagen pero lo importante es que NAT se encuentre desactivado ya que como son redes locales no necesita realizar el nateo de la IP y los fortigate saben como llegar a las redes por la configuracion que le hicimos en Static Routes por lo tanto eso tiene que desactivarse
![untitled](/assets/img/fortigate/forti34.png)

Ahora la policy anterior permite el trafico de ida (TO) pero ahora necesitamos crear una policy con el trafico de vuelta y esto se hace con click derecho en la policy en "Clone Reverse" y con esto invertirá los puertos y solo se tiene que configurar el nombre y habilitarla... Nota; no se hace reversa en la politica de INTERNET debido a que esa politica tiene la configuracion NAT habilitada 
![untitled](/assets/img/fortigate/forti35.png)

Ahora que ya le asignamos el nombre "FROM_CONTABILIDAD" queda habilitarla en Set Status Enable y con esta politica reversiva se invertió el origen (source) y el Destino junto con los puertos correspondientes (del puerto4 al 7 y del puerto7 al 4) y una vez haciendo lo mismo en el otro foriget con los respectivos cambios ya tendria comunicacion las 2 redes locales
![untitled](/assets/img/fortigate/forti36.png)

## Enrutamiento basado en politicas

la "Policy Routes" sirve para poder reenviar cierto tipo de trafico por cierta interfaz indicada, por ejemplo en estos momentos todo el trafico de "contabilidad" o "Site-B" viaja a travez de la configuracion de ruta estatica que hicimos anteriormente por el puerto de la red 172.10.10.0/30 (Port7) y lo que queremos hacer con esta Policy Routes es que solo el protocolo TCP HTTP(80) viaje por el ISP1 (Port2). Nota; es importante señalar que esto no es configuracion de ruta, mas bien es un criterio que nosotros definimos para reenviar el trafico de un lado a otro por la ruta que nosotros queramos
![untitled](/assets/img/fortigate/forti37.png)

de la siguiente forma donde Incoming interface es LAN, Source Address es la IP del origen del objeto que se quiere desviar que en este caso el pc Workstation, el Destination Address es la direccion del servidor, luego el puerto que queremos desviar que en este caso HTTP y escribimos el puerto a desviar en destino (80) y por ultimo Forward Trafic (reenvío de trafico) y activamos Outgoing Interface (interfaz saliente) escribiendo el Gateway de esa interfaz y ya con esto tenemos que todos los servicios viajan por el puerto de la red 172.10.10.0 y solo el servicio http viaja por ISP1 por la red 60.89.123.0
![untitled](/assets/img/fortigate/forti38.png)

Con esta imagen vemos que aun no hemos realizado ninguna prueba con el protocolo 80 debido a que el Hit Count se encuentra en 0;
![untitled](/assets/img/fortigate/forti39.png)

y ahora que intentamos de acceder al servidor por el navegador por http (puerto 80) nos marca el Hit Count en "1"  eso es una prueba de que el trafico si se está desviando y el resto del trafico como el ping se sigue realizando de forma normal por la Static Routes que configuramos anteriormente por la interfaz directa;
![untitled](/assets/img/fortigate/forti40.png)

## monitoreo de rutas por GUI

Sirve para ver las rutas activas, tambien mediante el Route Lookup sirve para ver por donde se encuentra saliendo la solicitud por ejemplo veremos como se llega al host 10.0.2.10
![untitled](/assets/img/fortigate/forti41.png)

y el resultado será la interfaz del puerto 7
![untitled](/assets/img/fortigate/forti42.png)

tambien podriamos ver por donde saldriamos si ponemos los DNS de google donde en el destino ponemos los DNS de google luego el puerto que dns es el 53 TCP y desde la red LAN. Esto es muy importante para ver cuando se tienen muchas rutas y queremos saber por donde estamos saliendo, y tambien es bueno para algun debug en caso de que estemos saliendo por alguna ruta equivocada. Tambien es importante entender que aca se ven las rutas activas independiente si están conectadas o no ya que por ejemplo no se ve la ruta ISP2 debido a que la ISP1 es la que se encuentra activa con una menor distancia administrativa
![untitled](/assets/img/fortigate/forti43.png)

## Monitoreo de rutas por CLI

- `get router info routing-table;` y al escribir el signo de pregunta tendremos las siguientes opciones donde nosotros mas veremos "details, all, static, connected y database"
- `get router info routing-table connected;` veremos las rutas del tipo conectadas
- `get router info routing-table static;` veremos las rutas estaticas
- `get router info routing-table all;` veremos todas las rutas (static y connected)
- `get router info routing-table database;` salen las mismas que en all pero acá sale la inactiva del puerto 3 (ISP2) que es la que no tiene el signo "asterisco>" cosa que en all no aparece la inactiva 
cuando un puerto esta caido muestra despues del puerto "inactive" que significa que el puerto esta caido como en la siguiente imagen
![untitled](/assets/img/fortigate/forti44.png)

## Monitoreo de policy route por cli

- `diagnose firewall proute list;` se puede ver la policy route creada donde la primera linea muestra los criterios programados, source wildcard muestra la direccion de origen, destination wildcar es el destino, gwy es el getway, dport es el destination port (80), protoclo 6 es TCP
![untitled](/assets/img/fortigate/forti45.png)

## Atributos de las rutas

unos de los atributos ya mencionados es la distancia administrativa que a menor distancia es la prioridad y en este caso le pusimos a ISP-1 una distancia de 10 y a ISP-2 una distancia de 15 para que siempre el trafico viaje por ISP-1. Ahora aparte de eso está la metrica y priority que prority es el parametro que tomará en cuenta el fortigate en caso de que la distancia sea la misma y al igual que la distancia entre menor es el numero de priority esa es la que tomará el fortiget como ruta principal, y el campo de metrica es utilizado por los protocolos de enrutamiento dinamico OSPF, RIP, BGP y eso nosotros no lo vemos por el momento. En la imagen se ve que ambos tienen la misma distancia
![untitled](/assets/img/fortigate/forti46.png)

Ahora le cambiamos la Priority a ISP-1 a 3 para que tome como principal el ISP-2, recordar que entre menor priority es el principal y en este caso ISP-2 tendrá priority 0 por lo tanto ese quedará como principal
![untitled](/assets/img/fortigate/forti47.png)

Ahora crearemos una politica para salir a internet por el ISP-2 ya que eso no lo tenemos y solo tenemos la politica de ISP 1 como se ve en la siguiente imagen
![untitled](/assets/img/fortigate/forti48.png)

la politica quedaria de la siguiente manera;
![untitled](/assets/img/fortigate/forti49.png)

Ahora que ya están las 2 politicas y le pusimos el contador en 0 iremos al active directory a navegar por internet y veremos si aumentan los bytes en INTERNET ISP2 segun la priority indicada
![untitled](/assets/img/fortigate/forti50.png)

y ahora se muestra que la que tiene trafico es la que está funcionando y es la ISP-2
![untitled](/assets/img/fortigate/forti51.png)

la metrica en la CLI aparecen con el comando;
- `get router info routing-table database;` donde el primer 15 es la distancia luego el 0 siguiente es la metrica que en este caso lo dejamos en 0, luego el 3 es priority y por ultimo el siguiente 0 es el wild o peso que mas adelante lo veremos. En el port3 no aparece la priority porque lo dejamos por defecto en 0 y solo modificamos el port2
![untitled](/assets/img/fortigate/forti52.png)

## Routing para servicios de Internet

Si queremos enviar cierto trafico de internet por alguna interfaz especifica por ejemplo queremos que los servidores de DNS de google salgan solo por la interfaz de ISP2 tenemos que ir a Network, Static Routes y en Internet Service agregar Google-DNS, y escribimos el Gateway Address de ISP2 y ya con esto cuando necesitemos los DNS de google saldrá por esa interfaz grafica. (no hay imagen debido a que no se puede realizar el laboratorio debido a que solo contamos con la licencia de evaluacion)

## ECMP

EMCP (multiples rutas de igual costo) mas conocido como balanceo de carga mediante ECMP significa que el trafico de la red sea distribuido entre el ISP1 y el ISP2, de tal manera que ambos se encuentren activos simultáneamente. Para poder activar el balanceo de carga ICMP se tiene que cumplir con 3 condiciones que son la misma distancia administrativa, misma priorida y la misma metrica y por ultimo tener el mismo destino (en este caso internet 0.0.0.0)    
![untitled](/assets/img/fortigate/forti53.png)

el primer corchete es la distancia administrativa junto con la metrica [10/0] y el segundo corchete es la prioridad junto con wild o peso [5/0]
![untitled](/assets/img/fortigate/forti54.png)

aun con esto no estamos balanceando la carga debido a que ICMP por defecto viene con modo basado en ip de origen (source-ip-based) por lo tanto siempre enviará por la misma ruta segun la direccion ip de origen que sea, los otros modos son basado en peso (weight-based), basado en uso o spill-over (usage-based) y por ultimo basado en direccion ip de origen y destino (source-dest-ip-based). El modo por defecto source-ip-based (basado en ip de origen) significa que al momento de llegar el primer intento de navegar por internet (o primera peticion) por parte del host, el fortigate hace una especie de sorteo y determina en por cual ISP o interfaz le tocará siempre a ese host salir hacia internet hasta que las sesiones de ese host expire y ECMP tenga que realizar el "sorteo" nuevamente.
![untitled](/assets/img/fortigate/forti55.png)

el otro metodo mas utilizado es el source-dest-ip-based que es parecido al anterior pero este aparte de enfocarse en la ip de origen para seleccionar la interfaz de salida tambien se fija en la direccion de destino. Por ejemplo, si el host quiere ir a la pagina google.com en el sorteo podria establecer que la ruta SIEMPRE será la interfaz 2 (ISP1) para esa direccion ip de destino (google.com) y el mismo host si quiere navegar en fortinet.com podria perfectamente establecer la ruta de la interfaz 3 (ISP2) y así hasta que nuevamente tenga que realizar el "sorteo" nuevamente. La forma de seleccionar este modo es;
- `config system settings;` para posicionarse en el contexto de configuracion
- `get | grep ecmp;` aplicamos grep para filtrar solamente ecmp y muestre en que modo está
- `set v4-ecmp-mode source-dest-ip-based;` para seleccionar este modo de balanceo de carga
- `end;` para guardar la modificacion
![untitled](/assets/img/fortigate/forti56.png)

y para probar haremos ping a distintos destinos y veremos en "Firewall Policy" por donde sale el trafico y nos daremos cuenta de que ambos proveedores se están utilizando;
![untitled](/assets/img/fortigate/forti57.png)

y si vamos a "Dashboard" en "FortiView Sessions" y hacemos que muestre "Destination Interface" veremos que han salido por ISP1 y tambien por ISP2 (aunque igual en menor medida)
![untitled](/assets/img/fortigate/forti58.png)

Para configurar el balanceo de carga de ECMP basado en peso (weight-based) se puede hacer en la interfaz o en la ruta (SOLO EN UNO) y en este caso lo haremos en la ruta. En lo que respecta al peso este es el unico caso en fortigate que el mayor numero de peso es mayor prioridad
- `config system settings;` para entrar en ese contexto de configuracion
- `get | grep ecmp;` para ver en que modo está
- `set v4-ecmp-mode weight-based;` para ponerlo en modo basado en peso
- `end;` para guardar los cambios
- `config router static;` para entrar en las rutas y ahí asignarle el peso
- `show;` muestra las rutas de los puertos 
- `edit 2;` para editar esa ruta (ISP2)
- `get;` para ver el weight por defecto que se encuentra en 0
- `set weight 20;` es el numero del peso que puede ir de 0 a 255
- `next;` para guardar
- `edit 1;` para editar esta ruta (ISP1)
- `set weight 10;` para que esta tenga menor peso y así la ruta 2 es prioridad en el balanceo
- `next;` para guardar
- `show;` para mostrar las rutas con los pesos configurados
- `end;` para guardar 

![untitled](/assets/img/fortigate/forti59.png)
![untitled](/assets/img/fortigate/forti60.png)

Para ver la ruta que está como prioridad y verificar los pesos de cada ruta (port3 y port2) se realiza con el siguiente comando donde lo vemos en el segundo digito del segundo corchete y donde la primera ruta que aparece es la prioridad
- `get router info routing-table database;` para ver las rutas de las interfaces
![untitled](/assets/img/fortigate/forti61.png)

ya con esto nos damos cuenta que el ISP2 es prioridad y tiene mayor navegacion por ese puerto
![untitled](/assets/img/fortigate/forti62.png)

y en las sesiones tambien se puede observar por cual interfaz sale mas trafico
![untitled](/assets/img/fortigate/forti63.png)

como comenté que tambien se puede hacer a nivel de interfaz (recuerda que solo se puede hacer o a nivel de rutas o a nivel de interfaz pero nunca ambos) y para modificarlo en la interfaz se realiza lo siguiente;
- `config system interface;` para entrar en el contexto de configuracion de interfaz
- `edit port2;` para editar el puerto de internet que es el ISP1
- `get | grep weight;` para verificar el numero de peso que tiene
- `set weight 50;` puede ir entre 0 a 255 y el mayor numero es mayor prioridad
- `next;` para guardar
- `end;` para salir

![untitled](/assets/img/fortigate/forti64.png)

por ultimo queda el modo basado en volumen o en uso y acá lo que se configura es el umbral de trafico y una vez que ese trafico llega al umbral o limite se cambia al otro proveedor. En esta configuracion solo se puede hacer en la interfaz y no en la ruta como el anterior modo
- `config system setting;`
- `get | grep ecmp;` para ver en que modo se encuentra
- `set v4-ecmp-mode usage-based;` para modificarlo a este modo basado en uso o volumen
- `end;` para salir y guardar
- `config system interface;` 
- `edit port2;`
- `set spillover-threshold 300;` puede ir de 0 a 16776000
- `next;` guardarmos
- `edit port3;` para editar ahora el ISP2
- `set spillover-threshold 500;` este es prioridad y una vez que llegue al umbral pase al ISP1
- `end;` guardamos y salimos
![untitled](/assets/img/fortigate/forti65.png)

Para deshacer los cambios de ECMP;
- `config system interface;` para editar los puertos
- `edit port2;`
- `unset spillover-thashold;` el commando unset hace que los valores queden por defecto
- `get | grep spill;` para verificar que el valor se encuentre en 0 (por defecto)
- `next;` para guarda
- `edit port3;`
- `unset spillover-thashold`
- `config router static;` para ir a las rutas estáticas
- `show;` para ver el peso de las rutas, la prioridad y volver a la distancia como estaba 10 y 15
- `edit 1;`
- `unset weight;`
- `unset distance;`
- `unset prority;`
- `next;`
- `edit 2;`
- `unset priority;`
- `unset weight;`
- `set distance 15;` para dejar el ISP1 como ruta por defecto y esta como backup
- `end;`
- `get router info routing-table database;` para confirmar que solo se navega por port2 (ISP1)

## LINK HEALTH MONITOR

Esta herramienta sirve para que el fortiget mediante PING hacia por ejemplo los DNS de Google pueda ir monitoreando si se encuentra con internet (ping exitoso) y en caso de que el ping no sea exitoso hará el cambio de ISP al puerto 3 y bajará el puerto 2 debido a que asume que hay un problema de conexión

- `config system link-monitor;`
- `show;` para ver que no tenemos nada configurado
- `edit 1;` para agregar una entrada y entrar en el cotexto correcto
- `get;` para ver todas las configuraciones que podríamos detallar
- `set srcintg port2;` le indicamos que este es el ISP1
- `set server 8.8.8.8;` hacia donde se realizarán las pruebas de ping
- `set Gateway-ip 200.212.31.2;` el Gateway de ISP1
- `show;` para ver lo que hemos configurado
- `get;` ya vemos todo configurado y lo importante es que update-static-route se encuentre habilitado (enable) para que actualice las rutas estáticas
- `end;` para guardar

Para borrar el link health monitor se hace de la siguiente manera;
- `config system link-monitor;`
- `show;` para ver el numero de entrada de contexto
- `delete 1;` para borrar esa entrada
- `show;` para verificar que no esté
- `end;` para guardar y salir

## Packet sniffer por CLI

Esto es muy util para realizar debug y se necesite correr un sniffer. El detalle del comando es; “diagnose sniffer” y el primer parámetro de este comando es qué es lo que queremos capturar y en este caso es “packet”, el segundo parámetro es en qué interfaz queremos hacer la escucha y en este caso “por4” o también se puede poner “any” para ecuchar todas las interfaces, y el siguiente parámetro son los filtros y en este caso pondremos el protocolo “ICMP”
- `diagnose sniffer;` para hacer una escucha en tiempo real por la línea de comando y ver si estamos capturando algún criterio
- `diagnose sniffer packet port4 icmp;` y ya con este comando el fortigate está a la escucha del trafico ICMP

Para detallar mas la escucha haremos por ejemplo que solo muestre los paquetes de cierta computadora en especifica y esto se realiza agregando comillas “” y && host x.x.x.x ;
- `diagnose sniffer packet port4 “icmp && host 10.0.1.10”;`

En este caso se captura tanto el trafico de ida como de vuelta “request” y “reply” y ahora haremos que solo nos muestre el trafico de solicitud “request” con el comando “src host” que significa que muestre solo los paquetes de solicitud de esa IP
- `diagnose sniffer packet port4 “icmpt && src host 10.0.1.10”;`

Si queremos que muestre solo el trafico del destino de la IP especifica se realiza con el comando “dst host”
- `diagnose sniffer packet port4 “icmp && dst host 10.0.1.10”;`

Ahora haremos una escucha del protocolo UDP del puerto 53 que corresponde a DNS y al realizar el ping mostrará ese trafico con la resolución de DNS de la pagina
- `diagnose sniffer packet port4 “udp && port 53 && host 10.0.1.10”;`

Y lo mismo que el anterior si en caso de que solo queremos capturar los paquetes que se enviar agregamos el comando “src”
- `diagnose sniffer packet port 4 “udp && port 53 && src host 10.0.1.10”;`

Ademas de los filtros ya mencionados también podemos especificar el nivel de verbosidad que significa que tanta información se requiere mostrar por pantalla y esto va por niveles desde el 1 hasta el 6 siendo el 4 el mas utilizado ya muestra la cabecera IP y el nombre de la interfaz
- `diagnose sniffer packet port4 “icmpt && host 10.0.1.10” 4;` muestra el trafico que está siendo originado y enviado por el puerto correspondiente (port4)

En lugar de indicar el puerto 4 (port4) ponemos “any” mostrará el puerto de donde entra el trafico y por donde sale siendo este comando muy importante para verificar errores de rutas. El “port4 in” significa que el trafico icmp está entrando por esa interfaz con esa dirección IP. Por otra parte el “port2 out” está enviado hacia afuera (internet) desde esa direccion IP hacia la otra dirección IP. Luego el “port2 in” significa que recibe respuesta desde esa IP con el destino a otra IP. Y por ultimo en por el puerto 4 sale la respuesta a la pagina desde la pagina hacia el host de destino.
- `diagnose sniffer packet any “icmp” 4;`

Y si luego del numero de verbosidad se agrega otro numero por ejemplo el 5 significa que solo se mostrará los 5 primeros paquetes de esa escucha
- `diagnose sniffer packet port4 “icmpt && src host 10.0.1.10” 4 5;`
