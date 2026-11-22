---
title: Bash-Scripting
date: 2026-11-21 14:10:00 +0800
categories: []
tags: [scripting, bash, linux, bash-scripting]     # TAG names should always be lowercase
author: owner
image:
    path: /assets/img/bash/header_Bash.jpg
    alt: Bash-Scripting
---

## Decompressor.sh ##

```bash
#!/bin/bash

# Función que se ejecuta al interrumpir el script (Ctrl+C)
function ctrl_c(){
  echo -e "\n\n[!] Saliendo ... \n"
  tput cnorm; exit 1   # Restaura el cursor antes de forzar la salida con código de error
}

# Captura la señal de interrupción (SIGINT / Ctrl+C) y la redirige a ctrl_c
trap ctrl_c INT

# Oculta el cursor de la terminal para una salida visual más limpia
tput civis

# Nombre del archivo comprimido inicial
first_file_name="prueba.gz"

# Lista el contenido interno del archivo, filtra la antepenúltima línea y extrae la última columna (nombre del archivo)
decompressed_file_name="$(7z l "$first_file_name" | tail -n 3 | head -n 1 | awk '{print $NF}')"

# Descomprime el archivo inicial de forma silenciosa usando el estándar POSIX (>/dev/null 2>&1)
7z x "$first_file_name" >/dev/null 2>&1

# Bucle que se repite mientras la cadena "$decompressed_file_name" no esté vacía (-n)
while [ -n "$decompressed_file_name" ]; do
  echo -e "\n [+] Nuevo archivo descomprimido: $decompressed_file_name"
  
  # Extrae el archivo actual de forma silenciosa
  7z x "$decompressed_file_name" >/dev/null 2>&1
  
  # Guarda el nombre del archivo válido antes de buscar el siguiente (evita perderlo cuando la última comprobación falle)
  last_file="$decompressed_file_name"
  
  # Intenta obtener el nombre del siguiente archivo interno; si ya es texto plano, falla en silencio y vacía la variable
  decompressed_file_name="$(7z l "$decompressed_file_name" 2>/dev/null | tail -n 3 | head -n 1 | awk '{print $NF}')"
  
  # Pausa de 1 segundo entre iteraciones para visualizar el progreso
  sleep 1
done

# Imprime el contenido del último archivo extraído y devuelve el cursor a la normalidad
echo -e "\n[+] El contenido del archivo final ($last_file):"
cat "$last_file"
tput cnorm

```
