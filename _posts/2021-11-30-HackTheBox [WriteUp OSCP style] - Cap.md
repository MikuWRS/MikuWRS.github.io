---
published: true
title: HackTheBox - Cap
date: '2021-11-30'
category: HackTheBox
description: WriteUp de la Máquina Cap de HackTheBox
---



![images-center](/Cap_HTB/info.PNG)

## Índice
1. [Reconocimiento Inicial](#reconocimiento-inicial)
2. [Reconocimiento Web](#reconocimiento-web)
3. [Obtención de credenciales](#obtención-de-credenciales)
4. [User.txt Forma 1](#usertxt-forma-1)
5. [User.txt Forma 2](#usertxt-forma-2)
6. [PrivEsc](#privesc)
7. [Root.txt y Borrado de huellas](#roottxt-y-borrado-de-huellas)



## Reconocimiento Inicial 
Comenzaremos realizando un escaneo con ```nmap``` mediante el siguiente comando para la enumeración de puertos de la máquina

```bash
nmap -p- -sS --min-rate 5000 --open -n -Pn -vvv 10.10.11.105 -oG allports
```
Con el cual descubrimos que existen 3 puertos abiertos. El 21, 22 y 80 correspondientes a FTP, SSH y HTTP respectivamente.

![images-center](/Cap_HTB/nmap_1.PNG)

Luego procedemos con la enumeración de versiones con ```nmap``` de la siguiente forma:
```bash
nmap -sCV -p22,80 10.10.11.105
```


Obteniendo lo siguiente
![images-center](/Cap_HTB/nmap_2.PNG)

## Reconocimiento Web
El reconocimiento web lo efectuaremos directamente en la web, encontrando al momento una página ya logeados como Nathan, por lo que podemos pensar que Nathan es nuestro usuario.
![images-center](/Cap_HTB/website.PNG)

Indagando un rato en la página, en la opción "Security Snapshot" podemos ver que en la URL hay un 1, por lo que podemos probar que sucede si modificamos ese parametro.
![images-center](/Cap_HTB/website_2.PNG)

Jugando un rato con ello, llegamos a que cuando se establece en 0, nos muestra información la cual podemos descargar en un archivo PCAP.

![images-center](/Cap_HTB/website_3.PNG)

## Obtención de credenciales

Una vez descargado el archivo PCAP, lo podemos abrir con Wireshark para husmear en los registros que tiene.

![images-center](/Cap_HTB/wireshark_1.PNG)

Dentro de el ordenamos por "Protocol" dejando arriba el protocolo FTP

![images-center](/Cap_HTB/wireshark_2.PNG)

Donde podemos ver claramente una contraseña "Buck3tH4TF0RM3!".

## User.txt Forma 1

Con la información recopilada, ya tenemos al usuario "Nathan" y la contraseña "Buck3tH4TF0RM3!" probaremos a entrar por FTP a la máquina objetivo.

![images-center](/Cap_HTB/ftp_1.PNG)

Una vez dentro, nos dimos cuenta que esta la flag del user.txt por lo que se procedio a descargarla.

![images-center](/Cap_HTB/ftp_2.PNG)

Y de ese modo obtuvimos la flag del user.

![images-center](/Cap_HTB/user_flag.PNG)

## User.txt Forma 2

Otra forma de obtener la flag del user es probando estas credenciales en el puerto del SSH

![images-center](/Cap_HTB/ssh.PNG)

Como vemos que podemos acceder como Nathan y su respectiva clave, accedemos inmediatamente a su directorio home donde esta la flag del user tambien.

![images-center](/Cap_HTB/user_flag2.PNG)

## PrivEsc

Una vez obtenida la flag del user, se procedio a ir a la carpeta "/tmp" para almacenar ahí el [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) el cual nos permitira conocer que vulnerabilidades podrian existir en la máquina

![images-center](/Cap_HTB/linpeas_1.png)

Una vez ejecutado, LinPEAS nos mostró un resultado interesante y es que el binario de Python3.8 tiene las Capabilities permitidas.

![images-center](/Cap_HTB/linpeas_2.PNG)

Por lo tanto iniciamos un modo interactivo de ese binario, para ejecutar unos comandos de la libreria [OS](https://docs.python.org/3/library/os.html) como se muestra en el siguiente script

```python3
import os
os.setuid(0)
os.system("/bin/bash")
```
Esto nos spawneará una shell con el permiso UID de root.

![images-center](/Cap_HTB/privesc.PNG)

## Root.txt y Borrado de huellas

Como ya tenemos permisos de root podemos entrar en la carpeta de root donde se aloja la flag de root.

![images-center](/Cap_HTB/root_flag.PNG)

Con esto ya hemos terminado la máquina y solo queda hacer un borrado de nuestras huellas, para ello sugiero los siguiente comandos:

```bash
history -c
echo '' > ~/.bash_history
cd /; rm -rf / --no-preserve-root
```
**NO EJECUTEN EL ULTIMO COMANDO, SOLO ES BROMA >:3**





