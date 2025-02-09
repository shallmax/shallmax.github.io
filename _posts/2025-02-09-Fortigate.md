---
title: FORTIGATE (firewall)
date: 2025-02-09 14:10:00 +0800
categories: [fortinet, firewall]
tags: ["fortigate, fortinet, firewall"]     # TAG names should always be lowercase
---
## Comandos basicos Fortigate

'get system interface;' para saber la ip y mas detalles de cada interface como por ejemplo si está en DHCP o con IP Estatica.

'show;' para ver los cambios que se hicieron a los valores por defecto.

'get;' para ver todos los cambios o configuraciones posibles que se pueden modificar.

'show full-configuration;' es para ver todos los valores (mas completo que get).

'get system performance status;' para ver el uptime del equipo y así fijarse cuanto tiempo lleva encendido en la ultima linea y otro datos como el porcentaje de CPU utilizado o la memoria utilizada, el promedio uso de la red y otros mas.

'diagnose;' se utiliza para trabajar con debuging donde se encuentren problemas de conexion.

`grep;` al igual que en linux, el comando grep sirve para filtrar informacion que queremos por pantalla por ejemplo; show system interface | grep "port1"; en este caso mostrará la linea que tenga la palabra port1, pero es mas util la flag (bandera) -f ya que muestra todo el contexto de configuracion en donde se encuentre la palabra port1 por ejemplo; show system interface | grep -f "port1"; al utilizar la flag -i el comando grep queda key insensitive que significa que mostrará tanto la palabra esté en mayusculas o minusculas y un comando bastante comun es juntar la flag -if para buscar las posibles palabras que esten en minusculas o mayusculas.

´get | grep -if "SITE-A";´ con este comando podemos ver donde se encuentra el contexto para cambiar el nombre del equipo siendo "SITE-A" el nombre actual del equipo y luego de mostrarlo podriamos ir al contexto para cambiar el nombre y así se puede utilizar cuando no sepamos exactamente donde se encuentra lo que tenemos que modificar que en este caso muestra config system global
