---
title: Write-Up Defensive
date: 2026-02-21 14:10:00 +0800
categories: []
tags: []     # TAG names should always be lowercase
author: owner
image:
    path: /assets/img/write-up_def/header_Write-Up_Def.jpeg
    alt: Write-Up Defensive
---

## MSHTML ##

Este desafio consta de analizar cuatro documentos maliciosos (maldoc), descubrir direcciones IP y Dominios ocultos y usar la informacion para averiguar qué vulnerabilidad o CVE está explotando el actor de amenaza. 

Vamos a utilizar varias herramientas de "DidierStevensSuite" como por ejemplo zipdum, re-search y numbers-to-string para extraer lo necesario.

- Pregunta 1; Examinando el archivo Employees_Contact_Audit_Oct_2021.docx, ¿cuál es la IP maliciosa en el archivo docx?
	para responder esta pregunta primero realizaremos un analisis basico del documento con el siguiente comando;
	`python3 zipdump.py /root/Desktop/ChallengeFiles/Employees_Contact_Audit_Oct_2021.docx`
    ![untitled](/assets/img/write-up_def/write-up1.png)
    `zipdum-py` enumera los archivos contenidos en el .docx y le asigna un "nombre de archivo indice". Esto es ya que un archivo .docx es básicamente un archivo zip.

	Para dumpear todos los archivos se realiza con la bandera -D mostrando lo siguiente;
	`python3 zipdump.py -D root/Desktop/ChallengeFiles/Employees_Contact_Audit_Oct_2021.docx`
    