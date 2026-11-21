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

## Function_Ctrl+C ##

`#!/bin/bash`

`function ctrl_c(){`

   `echo -e "\n\n[!] Saliendo ... \n"`
   `exit 1`

`}`

`#Ctrl+C`

`trap ctrl_c INT`

`sleep 10`

![untitled](/assets/img/bash/bash1.png)
