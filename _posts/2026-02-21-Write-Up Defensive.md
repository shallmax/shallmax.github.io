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
	Para responder esta pregunta primero realizaremos un analisis basico del documento con el siguiente comando;
	`python3 zipdump.py /root/Desktop/ChallengeFiles/Employees_Contact_Audit_Oct_2021.docx`
    ![untitled](/assets/img/write-up_def/write-up1.png)
    `zipdum-py` enumera los archivos contenidos en el .docx y le asigna un "nombre de archivo indice". Esto es ya que un archivo .docx es básicamente un archivo zip.

	Para dumpear todos los archivos se realiza con la bandera -D mostrando lo siguiente;
	`python3 zipdump.py -D root/Desktop/ChallengeFiles/Employees_Contact_Audit_Oct_2021.docx`
    ![untitled](/assets/img/write-up_def/write-up2.png)

    como muestra demasiado informacion canalizaremos con un pipe ( | ) la salida a otra herramienta llamada `re-search.py` donde se utilizan expresiones regulares para buscar entre archivos y así combinando zipdump para el volcado y `re-search.py -n -u ipv4` para buscar la direccion IP que nos pide la pregunta. El comando final seria;
	`python3 zipdump.py -D /root/Desktop/ChallengeFiles/Employees_Contact_Audit_Oct_2021.docx | python3 re-search.py -n -u ipv4`
    ![untitled](/assets/img/write-up_def/write-up3.png)
    y la respuesta a la pregunta es 175.24.190.249

- Pregunta 2; Examinando el archivo Employee_W2_Form.docx, ¿qué es el dominio malicioso en el archivo docx?
	Utilizaremos el mismo comando anterior pero ahora le agregaremos `domaintld` segun indicando en la libreria de github de DidierStevenSuite quedando con el siguiente comando;
	`python3 zipdump.py -D /root/Desktop/ChallengeFiles/Employee_W2_Form.docx | python3 re-search.py -u -n domaintld`
    ![untitled](/assets/img/write-up_def/write-up4.png)
    y la respuesta a la pregunta es arsenal.30cm.tw

- Pregunta 3; Examinando el archivo Work_From_Home_Survey.doc, ¿cuál es el dominio malicioso en el archivo doc?
	En esta pregunta nos pregunta por un archivo .doc por lo que segun google nos tenemos que centrar en lo siguiente;
    ![untitled](/assets/img/write-up_def/write-up5.png)

    por lo que el comando será `python3 zipdump.py /root/Desktop/ChallengeFiles/Work_From_Home_Survey.com`
    ![untitled](/assets/img/write-up_def/write-up6.png)
    donde dumpearemos el Index 10 con el comando `python3 zipdump.py /root/Desktop/ChallengeFiles/Work_From_Home_Survey.doc -s 10 -d`
    ![untitled](/assets/img/write-up_def/write-up7.png)

    el resultado esta codificado pero creo que en lo seleccionado se encontraria el dominio malicioso por lo que utilizaremos una tercera herramienta de DidierStevensSuit llamada "numbers-to-string.py" (aunque podriamos tambien utilizar CyberChef) `python3 zipdump.py /root/Desktop/ChallengeFiles/Work_From_Home_Survey.doc -s 10 -d | python3 numbers-to-string.py`
    ![untitled](/assets/img/write-up_def/write-up8.png)
    donde la respuesta a la pregunta tres seria trendparlye.com

- Pregunta 4; Examinando el income_tax_and_benefit_return_2021.docx, ¿cuál es el dominio malicioso en el archivo docx?
	Haremos lo mismo que en la pregunta 2 pero en lugar de usar `domaintld` utilizaremos `url-domain` solo para ver si tiene otro resultado `python3 zipdump.py -D /root/Desktop/ChallengeFiles/income_tax_and_benefit_return_2021.docx | python3 re-search.py -n -u url-domain`
    ![untitled](/assets/img/write-up_def/write-up9.png)

- Pregunta 5; ¿Cuál es la vulnerabilidad que explotaron los archivos anteriores?
	Para abordar cual es la vulnerabilidad comun de los 4 archivos primero tendremos que obtener los hashes de todos los archivos y analizarla con VirusTotal para poder ver las vulnerabilidades que tienen y eso se hace con el comando;
	`sha256sum *`
    ![untitled](/assets/img/write-up_def/write-up10.png)
    ![untitled](/assets/img/write-up_def/write-up11.png)
    ![untitled](/assets/img/write-up_def/write-up12.png)
    ![untitled](/assets/img/write-up_def/write-up13.png)
    ![untitled](/assets/img/write-up_def/write-up14.png)
    La respuesta a la pregunta 5 seria CVE-2021-40444

Herramientas y referencias:

Hoja de trucos de SANS para analizar documentos maliciosos:
[https://www.sans.org/posters/cheat-sheet-for-analyzing-malicious-documents]
