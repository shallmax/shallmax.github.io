---
title: FORTIGATE (firewall)
date: 2025-02-09 14:10:00 +0800
categories: [fortinet, firewall]
tags: ["fortigate, fortinet, firewall"]     # TAG names should always be lowercase
---
## Comandos basicos Fortigate

`get system interface;` para saber la ip y mas detalles de cada interface como por ejemplo si está en DHCP o con IP Estatica.

`show;` para ver los cambios que se hicieron a los valores por defecto.

`get;` para ver todos los cambios o configuraciones posibles que se pueden modificar.

`show full-configuration;` es para ver todos los valores (mas completo que get).

`get system performance status;` para ver el uptime del equipo y así fijarse cuanto tiempo lleva encendido en la ultima linea y otro datos como el porcentaje de CPU utilizado o la memoria utilizada, el promedio uso de la red y otros mas.

`diagnose;` se utiliza para trabajar con debuging donde se encuentren problemas de conexion.

`grep;` al igual que en linux, el comando grep sirve para filtrar informacion que queremos por pantalla por ejemplo; show system interface | grep port1; en este caso mostrará la linea que tenga la palabra port1, pero es mas util la flag (bandera) -f ya que muestra todo el contexto de configuracion en donde se encuentre la palabra port1 por ejemplo; show system 
interface \| grep -f port1; al utilizar la flag -i el comando grep queda key insensitive que significa que mostrará tanto la palabra esté en mayusculas o minusculas y un comando bastante comun es juntar la flag -if para buscar las posibles palabras que esten en minusculas o mayusculas.

`get | grep -if "SITE-A";` con este comando podemos ver donde se encuentra el contexto para cambiar el nombre del equipo siendo "SITE-A" el nombre actual del equipo y luego de mostrarlo podriamos ir al contexto para cambiar el nombre y así se puede utilizar cuando no sepamos exactamente donde se encuentra lo que tenemos que modificar que en este caso muestra config system global.

### Activar HTTP debido a que los Fortigate vienen solo con SSH Y HTTPS habilitado

`config system interface;` para acceder a la configuracion de las interface. 

`edit port1;` para entrar al puerto 1 y poder habilitar HTTP en ese puerto. 

`show;` para ver un breve resumen del puerto y se puede apreciar que se encuentra habilitado en set allowaccess el PING, HTTPS, SSH FGFM (fgfm es para administrar el fortigate desde el puerto fortimanagment).

`append allowaccess http;` para habilitar http en ese puerto y poder entrar a la configuracion mediante GUI y ahora con un show se puede ver HTTP.

`end;` es para grabar la configuracion modificada y vuelve al contexto principal.

`next;` es para grabar la configuracion pero queda en el contexto anterior al guardado.

`exec ping google.com;` para hacer ping.
(se desactiva default gateway from server y override internal DNS para que el laboratorio no tenga internet por las interfaces port1 de ambos fortigate)

en system luego en setting modificaremos la zona horaria correspondiente y modificamos en esa misma pagina el Idle timeout para que no se bloquee la sesion luego de inactividad poniendo el maximo de minutos (480) y activar la opcion auto file system check por si se apaga mal el equipo al iniciarlo intentará a nosotros hacerle un check system al sistema y con esa casilla activada el equipo lo hará

`get system status;` nos mostrará la version del equipo, la version de la BD de antivirus, el serial number del equipo, el estado de la licencia, la fecha de expiracion de la licencia, la informacion de los recursos  de CPU RAM, el hostname del equipo, si está en modo NAT o bridge, la fecha y hora y por ultimo la razon del ultimo reinicio del equipo.

### Para cambiar ip dinamica (DHCP) a estatica del puerto de administracion (port1)

`config system interface;`
`edit port1;`
`set mode static;` para cambiar de modo DHCP a estatica
`show;` mostrará la direccion ip que quedó con el DHCP
`set ip X.X.X.X/24;` se ingresa la ip que se quiera asignar en este caso (192.168.1.51/24)
`show;` mostrará la direccion ip ingresada
`end;` para guardar los cambios

### asignar IP a interfaces

config system interface;
edit port2;
set ip X.X.X.X/30;
set alias ISP-1; para asignarle un alias a la interface
set role wan; para asignarle un rol que en este caso es wan
next; para guardar y poder editar el siguiente puerto
edit port3;
set ip 45.32.12.1/30;
set alias ISP-2;
set role wan;
next;
edit port4;
set ip 10.0.2.254/24;
set allowaccess ping; para poder habilitar la funcion PING
set alias LAN;
set role lan;
end;