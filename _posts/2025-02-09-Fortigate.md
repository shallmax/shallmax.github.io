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

## SD-WAN

permite hacer un uso efectivo de internet mediante diferente algoritmos de balanceos. Es decir SD-WAN es la evolucion de ECMP porque incorpora diferentes mecanismos de medicion de estos vinculos, de manera tal que mediante esos indicadores será capaz de enviar el trafico por el enlace mas optimo. Para poder asociar los puertos 2 y 3 (ISP1 y ISP2) a nuestra interfaz SD-WAN los puertos no deben estar en uso. Para poder deshacer las configuraciones de estas interfaces se tiene que ir a "Network" y en "Interfaces" a la derecha indica "Ref" y el numero indica la cantidad de configuraciones que tienen las interefaces y de ahí se pueden eliminar directamente haciendo click con boton derecho y "delete". Con esto eliminamos las todas las configuraciones a estas interfaces como las politicas y rutas estaticas donde hacen referencia a estas interfaces pero las interfaces aun seguiran configuradas. Debido a que borramos las rutas estaticas ya no tendria internet el equipo y tampoco los hosts conectados a este.
Primero crearemos una zona de SD-WAN que es una interfaz de SD-WAN. Para eso vamos a "Network" luego a "SD-WAN" y ahí pinchamos en "Create New" y seleccionamos "SD-WAN Zone" y le asignamos un nombre como por ejemplo SDWAN-LAB para identificarla. Hacemos click en "OK" y volvemos a entrar en la recien creada para poder agregar asociaciones en "Interface members" que en este caso serian las interfaces 2 y 3
![untitled](/assets/img/fortigate/forti66.png)

Se necesitan marcar las interfaces para que salgan en las "Interface Members" como en la imagen
![untitled](/assets/img/fortigate/forti67.png)

con esto creamos una zona DS-WAN o una interfaz virtual de tipo SD-WAN y le asociamos nuestros ISP1 y ISP2 con sus respectivos gateways a la zona.
![untitled](/assets/img/fortigate/forti68.png)

Lo que queda por hacer es volver a crear las rutas estaticas para que pueda salir a internet y las politicas para que los hosts puedan navegar.
Para crear la ruta estatica es como siempre en "Network" "Static Routes" pinchamos en "Create New" y acá como es para la salidad de internet el destino es "0.0.0.0/0.0.0.0" pero no pondremos Gateway debido a que ya lo ingresamos en la zona SD-WAN y en "Interface" seleccionamos "SDWAN-LAB" que es la que creamos recien y con esto el fortiget tendrá salida a internet 
![untitled](/assets/img/fortigate/forti69.png)

Ahora crearemos la Policy para que los hosts puedan navegar por internet estableciendo el puerto de salida nuestra SDWAN-LAB creada... nota; una vez que un puerto es asociado como miembro a una SD-WAN no podrá ser referenciado individualmente
![untitled](/assets/img/fortigate/forti70.png)

ya con esto tenemos mas distribuido el trafico mediante por los 2 ISPs con el metodo de balanceo basado en origen y destino seleccionado en la pestaña SD-WAN Rules 
![untitled](/assets/img/fortigate/forti71.png)

## SD-WAN Perfomance SLAs

Esto es muy parecido a Link Health Monitor donde utilizamos los dns de google como testigo para determinar si el ISP1 estaba funcionando de forma correcta. En este caso fortigate trae por defecto testigos muy confiables como aws.amazon.com o los DNS de fortigate entre otros para determinar no solo el funcionamiento de nuestros enlaces, sino que en este caso excede a lo que vimos con Link health Monitor, ya que en ese caso solo nos interesaba si el PING respondia o no, en cambio con Perfomance SLAs ademas de ver si responde o no tambien podemos determinar el "Packet Loss", la "Latency" y el "Jitter". Con estos criterios fortigate puede determinar si el ISP2 es mas confiable que el ISP1 y enviar el trafico por ese enlace. Esa es la finalidad de este servicio muy utilizado en las empresas.
![untitled](/assets/img/fortigate/forti72.png)

En este caso utilizaremos el de Amazon donde primero encontraremos el nombre. "Probe mode" significa de qué forma hará las pruebas contra amazon, en la opcion "Active" fortigate envia paquetes contra el destino (aws.amazon.com), en la opcion "Pasive" fortigate no envia paquetes de prueba si no que utiliza el trafico que nosotros generemos para hacer la medicion, y por ultimo en la opcion "Prefer Passive" es igual que "Pasive" pero en el caso de que nosotros no generemos el trafico contra amazon el fortinet empezará a enviar sus propios paquetes como si estuviera en modo "Active". En protocolo son las pruebas ya sea en tipo "Ping", paquetes tipo "HTTP" y paquetes "DNS". En "Server" ponemos el testigo que es a quien se le hará las pruebas y se pueden agregar como maximo 2 testigos. En "participants" se define que miembros pertenecientes de SD-WAN vamos a estar utilizando para hacer la medicion, pudiendo especificar si es el ISP1 o el ISP2 o seleccionar "ALL SD-WAN Members" para seleccionar ambos. "SLA Target" por ahora lo desactivaremos para verlo mas adelante. En "Link Status" es para determinar si el vinculo está activo o no (muy parecido a Link Health Monitor). En "Check interval" significa que en intervalos de 1000ms (1 segundo) si caen 5 paquetes "Failures before inactive" den por caido el enlace (tardará 5 segundos en que caiga en enlace por los 5 paquetes solicitados cada 1 segundo). "Update Static route" indica que quite de las rutas estaticas ese vinculo que falló. Y por ultimo "Restore link after" significa que pasaran 10 check para que vuelta estar arriba el enlace (osea 10 segundos)
![untitled](/assets/img/fortigate/forti73.png)

y con esto verificamos la estabilidad de los enlaces junto con el porcentaje de paquetes perdidos o si hay algun enlace caído
![untitled](/assets/img/fortigate/forti74.png)

## SD-WAN Rules

SD WAN Rules, es muy parecido a Policy Route que lo utilizamos donde para un cierto origen que quería navegar hacia un destino podemos definir por qué enlace saldrá. En este caso SD WAN Rules tiene mas criterios para poder elegir por cual enlaces salir al destino especificado. Para crear la regla se tiene que ir a “Network” y luego “SD-WAN” y ahí abrir la pestaña “SD-WAN Rules” para luego en “Create New” ingresar los parámetros necesarios donde en “Name” ingresamos el nombre de la política, “Source Address” podemos definir una dirección IP o un grupo de usuarios (“Source address” y “user group” respectivamente), en el apartado de destination tenemos 3 opciones “address” que tenemos que ingresar la dirección de destino, “Internet Service” que podríamos ingresar algún servicio y por ultimo “Aplication” que es un software que queramos enrutar que en este caso la Suite de Google (G Suite) Esto significa que cuando nuestro Active Directori quiera navegar a través de la Suit de Google hará match con esta regla y hará lo que le configuremos en la sección de Outgoing Interfaces que puede ser lo siguiente; en la sección “Manual” significa que el trafico siempre se hará cuando se cumpla esta regla por la interfaz que pondremos en “Interface preference” (ISP2 por ejemplo) junto con “Zone preference” (SDWAN-LAB que tenemos creado). Ya con esta política creada siempre que el active direcotory navegue a través de Outlook.web.app se hará por el ISP2. En la siguiente foto sale que quedó marcado el ISP2
![untitled](/assets/img/fortigate/forti75.png)

Con la siguiente imagen se ve donde se encuentra el Tilde que es la interfaz seleccionada para esa regla que sacará cierto tipo de trafico por esa interfaz seleccionada
![untitled](/assets/img/fortigate/forti76.png)

La siguiente opción de Outgoing Interfaces es “Best Quality” que significa que se seleccionará la interfaz con la mejor performance para el enlace, por lo tanto en “Interface Preference” tenemos que seleccionar por lo menos 2 interfaces para que pueda elegir por cual salir. En “Measured SLA” que es donde estará haciendo las pruebas de los parámetros del enlace. En “Quality criteria” seleccionamos el parámetro que queremos medir ya sea Latency, Jytter, Bandwich, entre otros. Y ya con esto, el enlace con mejores mediciones será el seleccionado para la política creada
![untitled](/assets/img/fortigate/forti77.png)

En la sección de “Performance SLAs” se pueden ver los parámetros en tiempo real de los enlaces donde aparece que el Port2 tiene menos latencia que el Port3 por lo tanto Port2 es el seleccionado automaticamente
![untitled](/assets/img/fortigate/forti78.png)

La tercera opción en Outgoing Interfaces es “Lowest Cost SLA” y SLA significa que saca los criterios de la pestaña “Performance SLAs” donde al activar una vez dentro de la performance creada (Default_AWS) la pestaña que dice SLA Target se mostraran 3 valores para editar, que son el Latency threshold (umbral de latencia), Jitter threshold y Packet Loss threshold (umbral de paquetes perdidos) y estos parámetros serán de forma que genere un aviso en los parámetros que mostrará de color rojo de los enlaces si está bajo los criterios indicados y así utilizará el enlace que se encuentre por encima de los criterios indicados. Y con esto el enlace que se utilice será el de menor costo si ambos enlaces tienen su SLA en parámetros buenos y si también tienen el mismo costo verá cual tiene menor prioridad y si tienen la misma prioridad se utilizará el enlace que se encuentre primero en el apartado de “Interface preference” dentro de las “SD Wan Rules”. Y así seleccionará 1 de los enlaces indicados ya que lowest cost no hace balanceo de carga.
![untitled](/assets/img/fortigate/forti79.png)

En la imagen se aprecia que el ISP1 sobrepasó el umbral de 120 de latencia por lo tanto marca en rojo como alerta y seleccionó para el trafico de la regla el ISP2
![untitled](/assets/img/fortigate/forti80.png)

La ultima opción que nos tiene “Outgoing Interfaces” es “Maximize Bandwidth SLA” y este a diferencia de “Lowest Cost SLA” si hace balanceo de carga siempre y cuando los enlaces estén por encima de los criterios indicados en “SLA Target” sin importarle cual tiene menor costo o prioridad
![untitled](/assets/img/fortigate/forti81.png)

## Virtual Domains (VDOMs)

Es una funcionalidad de Fortigate que nos permite dividir un equipo físico en múltiples fortigate virtuales y a cada uno se le puede asignar su propia red virtual con su respectiva interfaz asociada, con su propio set de políticas, tablas de enrutamiento, direccionamiento, objetos, etc. con la idea de que el trafico que pase por el VDOMs1 no pueda ser visto por el VDOMs2 o VDOMs3 haciendo el equivalente a tener 3 fortigate independiente en un mismo equipo físico. Por defecto fortigate soporta hasta 10 VDOMs a excepción de los modelos de alta gama.

En las empresas se pueden ocupar VDOMs cuando se quiere un total aislamiento en sectores críticos o específicos por ejemplo separar el área contable del área de producción.

### Idependent VDOMs

No es una forma muy practica debido a que se necesita una interfaz física por cada VDOM que creemos para que pueda salir a internet, lo que la hace bastante limitada.

### Routing Through a Single VDOM

Es una evolución al anterior modelo agregando un VDOM “To_Internet” que tiene como finalidad que este sea el que está conectado a internet independiente si tenemos 1 o mas ISPs todos saldrán a través de este VDOM y atras de este VDOM tendremos conectados todos los VDOMs de la empresa mediante los Inter-VDOM links pero aun con esto no tenemos conexion entre los VDOMs

### Meshed VDOMs

Este modelo todos los VDOMs se conectan con todos mediante los Inter-VDOM link.

## Habilitar VDOMs

desde la Interfaz grafica se realiza en "System" y luego "Settings" y acá se encuentra una opcion para habilitar el "Multi-VDOM Mode".
desde CLI es;
- `config system global;`
- `set vdom-mode multi-vdom;`
- `end;`
- `y;`

## Creacion de VDOMs para Single VDOM

en "System" en la opcion "VDOM" click en "+Create New" y en "Virtual Domain" crearemos el de internet con el nombre "INTERNET" y tambien el VDOM "LEGALES" y "CONTABLE". El "Management VDOM" siempre tiene que ser el que salga a internet debido a que este se encarga de actualizar el fortigate con las firmas o virus nuevos, actualizaciones de Software, base de datos, etc. y para asignar el Management VDOM se selecciona el VDOM que en este caso es "INTERNET" y se hace click en "Switch Management" y ya con esto seleccionamos el VDOM INTERNET como Management y queda con un puntito en rojo para que sobresalga indicando que ese es el Management

## Asignacion de interfaces 

Desde la seccion de configuracion global (arriba donde aparecen todos los VDOMs) vamos a "Network", "Interfaces" y tenemos que retirar las interfaces que utilizaremos de la seccion "Hardware Switch" para que se encuentren habilitadas para los VDOMs creados. Para asignar la interfaz al VDOM INTERNET se abre el puerto que queremos asignar a "INTERNET" y en "Virtual domain" se le asigna la VDOM correspondiente (INTERNET) y así con la interfaz de LEGALES y  CONTABLE. Ahora si nos cambiamos a los diferentes VDOMs veremos el VDOM INTERNET, LEGALES y CONTABLE con sus respectivas interfaces y direccionamiento IP configurado. 

## Vincular VDOMs mediante inter-VDOM links

Se crean desde global "Network" "Interfaces" al pinchar "Create New" tenemos la opcion de "VDOM Link" y en nombre le pondremos "L - int" (de legales a internet debido a que tiene un limite maximo de caracteres por defecto) y en "Virtual Domain" de la "Interface 0" seleccionamos LEGALES con el direccionamiento "IP/Netmask" por ejemplo 10.10.10.1/30 (solo 2 ip con esa mascara) y habilitamos el PING. En "Virtual Domain" de la "Interface 1" asignamos INTERNET y la "IP/Netmask" 10.10.10.2/30 y tambien habilitamos el PING. y con esto se crea el VDOM Link y tenemos que hacer lo mismo con CONTABLE

## Crear ruta para salida a internet

En el VDOM LEGALES vamos a "Network" y "Static Routes" y en "Destination" la ip 0.0.0.0/0.0.0.0 debido a que es para salida a internet, en "Gateway Address" 10.10.10.2 que es el extremo del VDOM Link y en "Interface" tiene que detectar el VDOM Link creado (L - int0) y ahora tenemos que crear la policy para que los hosts detras de LEGALES puedan navegar a internet. Vamos a "Policy & Objects" y "Firewall Pollicy" y "Create New" con el nombre de INTERNET LEGALES, en "Incoming Interface" asignamos el "internal1" y en "outgoing Interface" asignamos "L - int"  y todo lo demas igual que siempre salvo en que en esta politica aunque sea para salir a internet no habilitamos el NAT ya que el VDOM de legales no es el de salida a internet. Por ultimo necesitamos la configuracion del VDOM de INTERNET para que conozca la red de LEGALES en "Static Routes" creamos con la IP 172.17.0.0/24 (direccion de red de LEGALES) el "Gateway Address" 10.10.10.1 y en "Interface" tiene que detectarse el L - int1 y por ultimo creamos la policy para la salida a internet desde el VDOM de INTERNET la creamos con nombre INTERNET, "Incomming Interface" L - int1, "Outgoing Interface" wan2 y acá si va el NAT habilitado ya que este es el VDOM de cara a internet y con esto ya tiene que tener internet los hosts de la red LEGALES

## Creacion de VDOMs para Meshed VDOM

con lo ya configurado tendremos que crear un ultimo VDOM link entre el vdom de LEGALES y CONTABLE junto con la creacion de unas rutas y nuevas politicas para la comunicaciones entre las 2 redes. 
Crearemos el VDOM Link faltante desde la seccion de GLOBAL "Network" "Interfaces" "Create New" "VDOM Link" y en "Name" lo llamaremos L - C (de legales a contable) en "Virtual Domain" de la "interface 0" seleccionamos LEGALES, en "IP/Netmask" 10.30.0.1/30 Habilitamos el PING y en "Interface 1" en "Virtual Domain" seleccionamos CONTABLE con "IP/Netmask" 10.30.0.2/30 y habilitamos el PING y con esto ya tenemos el vinculo entre los VDOMs. Ahora queda configurar en LEGALES crear una una ruta para que aprenda como enrutar hacia CONTABLE y para eso nos vamos en la seccion LEGALES "Interfaces" "Static Routes" "Create New" y en "Destination Subnet" detallamos la 10.0.1.0/24 y en "Gateway Address" la IP del extremo VDOM Link 10.30.0.2 y con eso "Interface" tiene que detectar L - C0 automaticamente y debemos crear la misma ruta en el VDOM de CONTABLE con la nueva "Static Route" 172.17.0.0/24 con el "Gateway Address" 10.30.0.1 con el "Interface" automatico de L - C1. Ahora que ya estamos en CONTABLE crearemos la politica de ida hacia el sector de LEGALES con el nombre de To_Legales, el Incomming Interface es Internal2, el Outgoing Interface es VDOM Link "L - C1" y sin NAT obviamente. Al crearla tomamos esa politica y le creamos la reversa para el trafico de vuelta con el boton derecho "Clone Reverse" con el nombre de From_Legales y por ultimo la habilitamos. Y ahora hacemos lo mismo en el VDOM de LEGALES con el nombre To_Contable, Incoming Internal1, Outgoing L - C0 y sin NAT, realizamos la reversa tambien y la habilitamos y ya con esto la red LEGALES y CONTABLE deberia alcanzarse 

## Gestion de Usuarios administradores en VDOM

Por defecto el unico usuario que tiene permisos para hacer configuraciones en todos los VDOMs, tomar backup y restaurarlos es el usuario admin o algun otro usuario que tenga el perfil de super admin. Tambien es posible crear usuario y asignarle directamente la administracion de un VDOM o varios VDOMs pero no podran acceder a las configuraciones globales ya que eso solo lo puede hacer el usuario admin o algun otro usuario que tenga el perfil de super admin.
Desde la seccion de Global en "System", "Administrators" y en "Create New" creamos un nuevo usuario por ejemplo Contable en Type "Local User" en "Administrator Profile" se puede seleccionar si es prof_admin o super_admin y en "Virtual Domains" podemos seleccionar el VDOM que queramos que administre (vdom contable) y para que este usuario pueda entrar tiene que hacerlo desde la red de CONTABLE.

## VLANs en equipos fortigate

Con las VLANs podemos subdividir una o mas interfaces fisicas en distintas interfaces o redes logicas, haciendo completamente independiente una de otra teniendo su propio dominio de broadcast. Con esto tambien podemos hacer un mejor aprovechamiento de un equipo que tiene pocas interfaces. Esto se lleva a cabo mediante la inserción de los vlantags en los frame de Ethernet (capa 2). Los equipos de capa 2 (switch) tiene la capacidad de agregar o remover los tags de sus propias vlans mientras que los equipos de capa 3 (routers y firewalls) puede sobreescribir estos tag para poder enrutar el trafico a la vlan correspondiente. Se presenta la siguiente topologia;
![untitled](/assets/img/fortigate/forti82.png)

### creacion de vlans

nos vamos a "Network", luego a "Interfaces" y en "Create New" seleccionamos "Interface" y aca creamos una vlan donde en "VLAN ID" tenemos que ingresar el TAG de la VLAN (numero de vlan) entre otros datos segun la imagen;
![untitled](/assets/img/fortigate/forti83.png)

creamos la segunda vlan (Contable)
![untitled](/assets/img/fortigate/forti84.png)

y por ultimo la vlan de Sistemas en el puerto 3
![untitled](/assets/img/fortigate/forti85.png)

y ahora podemos ver las vlans creadas tanto en el puerto 2 como en el puerto 3
![untitled](/assets/img/fortigate/forti86.png)

### Creacion de politicas para las vlans

Para que se puedan comunicar entre las vlans hace falta la politica correspondiente y esto se hace en "Policy & Objetcts", "Firewall Policy" y "Create New" y la configuramos sin NAT 
![untitled](/assets/img/fortigate/forti87.png)

y ahora ya creada le hacemos un "Clone Reverse" para el trafico de vuelta para así tener comunicacion entre las vlans configuradas. Acemos esto para cada una de las vlans tanto con el trafico de ida como con el trafico de vuelta
![untitled](/assets/img/fortigate/forti88.png)

## Virtual Switchs en equipos fortigate

Es cuando hacemos que el equipo fortigate trabaje como un switch mediante Software Switch y así al momento de conectarle en sus interfaces los hosts se puedan comunicar entre ellos directamente y para configurarlo tenemos que ir a "Network", "Interfaces" y "Create New" y seleccionamos "Interface" y lo importante en la seccion de "Type" seleccionar "Software Switch" y ahora seleccionamos las interfaces que estarán como Switch y para nuestro caso crearemos una red y habilitaremos el "DHCP Server" con los rangos por defecto para que el fortigate nos dé un rango de IP
![untitled](/assets/img/fortigate/forti89.png)

una vez creado se muestra en "Interfaces" el apartado de Software Switch donde muestra el tipo que es, los puertos, la ip de la red, los permisos (en este caso solo PING), el rango de DHCP Server configurado. Ya con esto al momento de conectar los equipos a esos puertos estaran configurados por DHCP y tendran comunicacion como si fuera un switch
![untitled](/assets/img/fortigate/forti90.png)

## Fortigate modo transparente

Hasta ahora siempre se a utilizado el fortigate en modo NAT que significa que el equipo enruta el trafico de acuerdo a la capa 3 del modelo OSI, con esto actúa como un router teniendo IP en cada interfaz a excepción de cuando creamos un Software-Switch. En el modo transparente o modo Bridge el equipo reenvía los paquetes de acuerdo a la capa 2 del modelo OSI (MAC Address) lo que hace que el equipo no tenga direcciones IP en sus interfaces ya que actúa como un Switch. La gran ventaja de este modo es que no requiere ningún cambio ni configuración adicional en la red ya existente.

Para pasar de un equipo en modo NAT a uno Transparente se realiza de la siguiente manera;
- `Config system interface;`
- `Edit port1;` editar el acceso del puerto conectado
- `Append allowaccess http;` para tener acceso mediante GUI
- `End;` para guardar
- `Config system settings;` para entrar en el contexto de cambiar de NAT a BRIGE
- `Set opmode transparent;` o NAT si así se requiere
- `End;`

### Error al pasar a modo Brige

Puede dar un error debido a que no se puede tener switches administrados o switch con vlans en el equipo por lo tanto tenemos que borrar esas configuraciones. En el caso de nosotros no tenemos eso pero sí tenemos una interfaz configurada con fortilink por lo tanto tenemos que deshabilitarla
![untitled](/assets/img/fortigate/forti91.png)

- `Config system interface;` para entrar en el contexto
- `Show;` para ver cual interfaz está con fortilink

![untitled](/assets/img/fortigate/forti92.png)

Se podría eliminar esa configuración con el comando “delete fortilink” pero no lo permite debido a que esa interfaz está en uso por lo tanto una opción sencilla es ir a la interfaz grafica y abrir "Network", "Interfaces" hacer click en el numero de la columna de "Ref" donde aparecen todos los servicios referenciados a fortilink y eliminarlos
![untitled](/assets/img/fortigate/forti93.png)

pero lo haremos mediante la CLI;

- `Show | grep -fi fortilink;` la flag -f es para que muestre el contexto en donde encontró la cadena y la -i para que ignore mayúsculas y minúsculas

Nos muestra 4 entradas
![untitled](/assets/img/fortigate/forti94.png)

la primera es la que tenemos que eliminar y las otras 2 (NTP y DHCP) tenemos que deshabilitar y la ultima es solo una descripcion que encontró el comando grep por lo tanto la ignoramos. Para que deje eliminar la primera se hace de la siguiente manera;

- `Config system ntp;`
- `Show;` para ver lo que tenemos que deshabilitar
- `Set ntpsyn disable;` para desactivar
- `Set server-mode disable;` para desactivar
- `Show;` para confirmar que esté desactivado
- `End;` para guardar
- `Config system dhcp servier;` para entrar en ese contexto
- `Show;` para ver lo configurado
- `Delete 1;` para eliminar la entrada
- `End;`
- `Config system interface;`
- `Delete fortilink;` para eliminar el fortilink y ahora deja ponerlo en modo transparente
- `End;`
- `Get system interface;` para ver la ip que nos dio el DHCP y así poder dejar esa ip de administración
- `Config system settings;`
- `Set opmode transparent;` para dejarlo en modo bridge
- `Set manageip 192.168.1.60/24;` podriamos poner la que nos dio el DHCP o poner alguna otra del rango para así dejar esta ip de administración del equipo
- `Set Gateway 192.168.1.1/24;` para que pueda salir a internet y así actualizarse
- `End;` para guardar

Se puede verificar con

- `Get system status;` y en operation mode; indica “Transparent”

Y ahora utilizando la ip de administración vamos al navegador y accedemos al fortigate, conectamos a los puertos del fortigate el router y el host y creamos la política para que el host pueda salir a internet
![untitled](/assets/img/fortigate/forti95.png)

## Alta Disponibilidad (HA)

Mediante la alta disponibilidad se puede configurar un cluster de equipos fortigate (entre 2 a 4 equipos). La ventaja es que si uno falla los otros equipos tomarán su lugar automáticamente sin que los usuarios lo noten.

Existen 2 modalidades de alta disponibilidad, la primera es “Activo Pasivo” donde tendremos un equipo designado como primario y los otros equipos serán secundarios y solo el primario procesará datos y los otros están en stand by a la espera de que el primario falle. En este modo el primario envia su configuración y mantiene en sincronia a los demás fortigates de tal manera que si agregamos alguna policy, ruta o un objeto esto se ve reflejado en los equipos secundarios.

El otro modo “Activo Activo” hace que todos los equipos del clauster procesan trafico sin embargo el equipo primario es el que balancea la carga con los demás fortigates. En caso de que el primario falle un equipo secundario pasa a ser primario

### Requerimientos para un HA

- Tener entre 2 a 4 equipos del mismo modelo de fortigate con un licenciamiento ideal del mismo tipo
- Definir un link de comunicación entre los fortigate llamada heartbeat y que idealmente sean 2 conexiones para tener una de respaldo y este permite la comunicación entre los distintos equipos del clouster para la distribución de trafico
- Se necesita configurar las mismas interfaces en todos los equipos del clauster

Haremos la configuración en el primario para que lo replique en el secundario;

- `Config system interface;`
- `Edit port2;` para salir a internet
- `Set ip 45.32.12.1/30;`

Configuramos la ip del las interfas de managment de ambos equipos donde se encuentran conectados los fortigate (que se recomiendan 2 interfaces) siempre tiene que estar en static y nunca por DHCP por lo tanto la configuramos de la siguiente manera

- `Config system interface;`
- `Edit port1;` donde esta conectado el otro fortigate
- `Set mode static;` para dejarla estática
- `Set ip 192.168.1.21/24;` tiene que ser una ip disponible en la red
- `Append allowaccess http;` para poder entrar mediante GUI y administrarlo
- `End;`
- `Config system global;` para entrar en el contexto y cambiar el nombre
- `Set hostname HA-2;` para dejarlo con ese nombre
- `End;`

Hacemos lo mismo en el fortigate principal y también la IP Static correspondiente
- `config system interface;`
- `edit port1;`
- `set mode static;`
- `set ip 192.168.1.20/24;`
- `append allowaccess http;`
- `end;`
- `config system global;`
- `set hostname HA-1;`
- `end;`

### configuración HA Activo Pasivo

En la GUI vamos a System, luego HA y acá en el apartado de “Mode” que se encuentra en “Standalone” seleccionamos Active-Passive. El campo de “Device priority” hace que la prioridad mas alta será el principal. Se tiene que poner un nombre al clouster en el campos de “Group name” junto con la contraseña y este tiene que ser igual en todos los fortigate del clauster. “Session pickup” se refiere a que si está activado el fortigate principal guardará las sesiones de los usuarios en todos los fortigates del claster para que en caso de caída los otros equipos ya tengan las sesiones y el usuario no tenga que volver a conectarse a sus programas. “Monitor interfaces” es un elemento muy importante ya que si no está activo y en caso de desconexion del cable con la LAN como el fortigate seguirá encendido no se tomará como caido y no tomaría eso como un evento de alta disponibilidad, en cambio si se agrega el puerto de la LAN en “Monitor interfaces” en caso de desconexion del cable levantará la alerta de evento de alta disponibilidad y podrá realizar el cambio de equipo automáticamente haciendo que el otro equipo sea prioritario y así tomar el control. “Heartbeat interfaces” son las interfaces que se utiliza para que los fortigate se comuniquen entre sí por lo tanto tenemos que seleccionar la interfaz o las interfaces que están directamente conectadas entre los fortigate. “Management Interface Reservation” se tiene que detallar la interfaz que sea de administración para que no sean sincronizadas y en “Destination subnet” se ingresa la red correspondiente. Ya con esto tenemos el primer equipo del clauster configurado. En el otro equipo tenemos que hacer exactamente lo mismo y una vez esperar unos minutos para que el primario le envíe toda la configuración al equipo secundario.
![untitled](/assets/img/fortigate/forti96.png)

![untitled](/assets/img/fortigate/forti97.png)

### primary Fortigate Election: Override Disabled

En caso de apagar el equipo primario pasará al secundario y cuando vuelva a encender el primario este quedara como secundario debido a que fortigate sigue el siguiente orden de selleccion; el equipo con mas puertos monitorizados será el prioridad, en caso de coincidir la misma cantidad de puertos conectados evalua el HA Uptime que es el tiempo de pertenencia de ese equipo al claster y es por esta razón que queda el otro equipo como prioritario debido a que tiene mas minutos en el claster que el otro que reiniciamos, de tener el mismo tiempo en el clauster verificará el “Priority” y el que tenga mayor numero será el primario y si en caso de que todo esto sea igual entre los equipos se selecciona con el numero de serie mas alto de cada equipo.


### primary Fortigate Election: Override Enable

Es un modelo que a diferencia del que viene por defecto (Override Disable) con este podemos dejar siempre un equipo como primario una vez que este vuelva a estar disponible y esto se hace debido a que primero al igual que el otro modo mira las interfaces conectadas (el que tenga mayor numero de interfaz es el prioridad) pero como segundo factor toma en cuenta la “Priority” y así al darle mayor prioridad al primario este apenas vuelva a estar disponible será utilizado como primario. El tercer paso que toma es el HA uptime que mira cual tiene mayor tiempo en el claster (a diferencia del disable este está en tercer lugar y el priority está en segundo lugar).

Para programar esta opción se tiene que hacer en todos los equipos ya que no se actualiza;
- `Config system ha;`
- `Set override enable;`
- `End;`

### HA Activo Activo

Para poder pasar de Activo Pasivo a Activo Activo como ya tenemos toda la configuración de interfaces e IP solo se modifica en “System”, luego “HA”, en “Mode” seleccionamos “Active-Active” haciendo lo mismo en todos los equipos y con esto ya queda en el modo para realizar balanceo de carga con los equipos del claster. Los fortigate cuentan con MAC virtuales por lo tanto cuando el principal se cae los otros ya tienen la MAC virtual por lo tanto los hosts nunca se enteran cuantos equipos hay en el clauster. También podemos hacer HA  con claster con VDOMs. Una cosa importante es que para actualizar los equipos solo basta con subir la imagen de actualización al equipo primario y este primero actulizará todos los equipos del claster y una vez que todos los secundarios estén actualizados el primario se pasa como secundario y ahí recién se actualiza para luego volver como primario

### HA Comandos utiles en la CLI

- `diagnose sys ha status;` muestra informacion importante como los serial numbers de los equipos del claster (FGVMEVVEZLHQHNC4 - FGVMEVORDBOYDBD6), junto con quien es el primario y el secundario, tambien la prioridad de cada uno (130 y 128), los hostname, y la prioridad de los serial numbers. Otra informacion importante es la direccion IP de la interfaz de heartbeat que se encuentra en "primary_ip" (169.254.0.1) y esta es asignada de forma automatica dependiendo de quien tenga el serial number mas alto recibe la direccion IP mas baja.
![untitled](/assets/img/fortigate/forti98.png)

- `diagnose sys ha checksum cluster;` con este comando se ve el checksum (calculo) que hace el equipo para verificar si se encuentran sincronizados teniendo ambos calculos el mismo resultado
![untitled](/assets/img/fortigate/forti99.png)

- `execute ha manage 1 admin;` para conectarme desde la CLI del equipo primario al equipo secundario, una vez ingresado ese comando pide la contraseña del equipo secundario y asi configurar lo que se necesite 
![untitled](/assets/img/fortigate/forti100.png)

- `execute ha failover set 1;` sirve para testear el failover de un claster para así no estar apagando el equipo y con este comando pasa al siguiente equipo como primario.
execute ha failover status; sirve para ver si el test del failover está habilitado y si muestra "set" significa que está habilitado
![untitled](/assets/img/fortigate/forti101.png)

- `get system ha status;` en la seleccion de "Primary selected using:" nos muestra el motivo por el cual un equipo se encuentra como primario y en este caso muestra que el equipo con la serial number "FGVMEVORDBOYDBD6"  fue seleccionado como primario debido a que se ejecutó "EXE_FAIL_OVER flag" el equipo con la serial number "FGVMEVVEZLHQHNC4". Tambien muestra el "Configuration Status" que indica que los equipos se encuentran en sincronia. Tambien los recursos del sistema utilizados por cada equipo como memoria, CPU, sistema. Tambien muestra los index del claster que son los numeros de cada equipo
![untitled](/assets/img/fortigate/forti102.png)

- `execute ha failover unset 1;` sirve para deshacer el test failover

## IPsec

