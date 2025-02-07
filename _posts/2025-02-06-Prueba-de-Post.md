---
title: Blueprint - Write up (TryHackMe)
author: ELIAS CAMPOS
date: 2021-10-23 23:40:00 -03000 
categories: [TryHackMe]
tags: [write-up, windows]
math: true
mermaid: true


PRUEBA DEL PRIMER POST COPIADO.




## TryHackMe y conexiones por VPN

> Como ya sabrán algunos, para auditar máquinas en TryHackMe debemos conectarnos a una VPN mediante OpenVPN. Esto nos permitirá hacer como si nuestro equipo y el de TryHackMe estuvieran en la misma red.



Bueno sin más preambulos, comencemos con la máquina...


## **Enumeración de puertos**

Comenzaré haciendo un *Port discovery* o bien, en Español, un *Escaneo de puertos*. Para ello haré uso de nmap:

```bash
nmap -vvv -sS --open -n -Pn -p- --min-rate 5000 10.10.28.212 -oG ports
```

* **-vvv**: verbose con máximo despliegue de información.
* **-sS**: indicamos que queremos hacer conexiones por TCP SYN. Esto significa que no se entablarán completamente las conexiones por TCP, esto se hace no completando el 
* **\--open**: filtramos por los puertos que se detecten con estado *open*, o sea que están abiertos (no filtrados, ni cerrados).
* **-n**: deshabilitamos la 
* **-Pn**: deshabilitamos el *host discovery*, o *descubrimiento de equipos* por medio del 
* **-p-**: indicamos que haga escaneo de los 65535 puertos de una máquina.
* **--min-rate**: señalamos la cantidad mínima de paquetes a enviar por segundo (en este caso indicamos 5000).
* **-oG**: solicitamos que output de la captura se almacene en formato grepeable en el fichero indicado (en mi caso es el archivo *ports*). 



**Ya con los puertos abiertos en mano, procederé a hacer un escaneo más exhaustivo de de estos:**

```bash
nmap -p80,135,139,443,445,3306,8080,49152,49154,49158,49159 -sV -sC 10.10.28.212 -oN nmap
```

* **-p**: me permite indicarle los puertos a escanear.
* **-sV**: detección de servicios y la versión de estos. 
* **-sC**: me permite utilizar los scripts por defecto que tiene la herramienta nmap para enumerar los servicios que estén corriendo en los puertos.
* **-oN**: solicitamos que el output del escaneo se almacene en un fichero en un formato normal (en mi caso lo almaceno en el fichero *nmap*).  


## **Enumeración de servicio SMB**

Como se puede notar en la captura de nmap, tenemos el puerto 139 y 445 corriendo el servicio SMB. Por esto mismo recurriremos a 
la herramienta [smbclient]() para enumerar los recursos compartidos que tenga la máquina (aunque también hay otras como 
[crackmapexec]() o [smbmap]()). 
Comenzaré listando las comparticiones sin hacer uso de credenciales (*null-session*):

```bash
smbclient -L 10.10.28.212 -N
```

* **-L**: solicitamos el listado de recursos compartidos.
* **-N**: indicamos el *null-session*   

Como podemos ver tenemos dos recursos asequibles; aquellos que no tienen el símbolo de dolar ($).
