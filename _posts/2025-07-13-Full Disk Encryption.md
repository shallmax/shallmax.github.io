---
title: Full Disk Encryption (FDE)
date: 2025-07-13 14:10:00 +0800
categories: [cifrado , proteccion de datos en reposo]
tags: ["bitlocker , fde "]     # TAG names should always be lowercase
author: owen
image:
    path: assets/img/FDE/header_fde.webp
    alt: fde
---

## Full Disk Encryption (FDE) ##

El cifrado completo de disco (FDE) consiste en utilizar un criptosistema simetrico para cifrar todos los datos que se encuentran en el disco cuando el sistema está apagado. Este tipo de tecnicas protegen los datos en reposo (data-at-rest) cuando un sistema se apaga y lo descifran cuando se enciede de manera transparente para el usuario.

## BitLocker ##

BitLocker es una aplicacion nativa de Windows que cifra el disco duro con una clave simetrica, ya sea con la informacion del momento o todo el disco segun la opcion que seleccionemos para que así, al momento de que alguien quiera conectar el disco apagado a algun programa como Autopsy con la idea de recuperar la informacion, esta se encuentre cifrada sin poder verse. De esta forma aseguramos los datos que se encuentran en reposo en caso de que el computador o el dispositivo USB se pierda y alguien quiera saber que informacion contiene.
![untitled](/assets/img/FDE/BitLocker01.png)

Es importante guardar la clave de recuperacion en algun lugar seguro para que si en algun momento necesitemos recuperar la informacion con esa clave podamos tener acceso.
![untitled](/assets/img/FDE/BitLocker02.png)


