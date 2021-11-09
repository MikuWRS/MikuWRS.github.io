---
published: true
title: HackTheBox - Horizontall [WriteUp OSCP style]
date: '2021-11-02'
category: HackTheBox
description: WriteUp de la Máquina Horizontall de HackTheBox
---

![images-center](/assets/images/Horizontall_HTB/info.PNG)

## Reconocimiento Inicial 
Comenzaremos realizando un escaneo con ```nmap``` mediante el siguiente comando para la enumeración de puertos de la máquina

```bash
nmap -p- -sS --min-rate 5000 --open -n -Pn -vvv 10.10.11.105 -oG allports
```
![images-center](/assets/images/Horizontall_HTB/nmap-portscan.PNG)

Luego se procede a la enumeración de versiones con ```nmap``` de la siguiente forma:
```bash
nmap -sCV -p22,80 10.10.11.105
```
Obteniendo lo siguiente
![images-center](/assets/images/Horizontall_HTB/nmap-verscan.PNG)

## Reconocimiento Web
Antes de comenzar con este reconocimiento, como es típico en las máquinas de HTB se realizó un virtual hosting en el /etc/hosts incorporando en él la dirección ip y el hosting horizontall.htb

Luego se procedió a realizar una enumeración con ```GoBuster```, sin embargo, no fue posible dado que arroja un error. Por lo que se procedió a abrir el navegador para buscar información en ella.
![images-center](/assets/images/Horizontall_HTB/web-web_inicial.PNG)

Indagando un rato en la página, no se encontró nada útil, por lo que se procedió a inspeccionar el código fuente, luego de un rato se llegó a que existe un llamado a una URL, por lo que podemos pensar que se puede realizar un virtual hosting a esa dirección e inspeccionarla.
![images-center](/assets/images/Horizontall_HTB/web-web_script_sub_domain.PNG)

## Reconocimiento SubWeb
Una vez realizado el virtual hosting, se procedió a enumerarla con ```GoBuster``` encontrando que existen algunos directorios.
![images-center](/assets/images/Horizontall_HTB/gobuster-subdomain.PNG)

Luego se procedió a acceder a la subweb encontrando lo siguiente:
![images-center](/assets/images/Horizontall_HTB/subweb-inicial.PNG)

Probando con la ruta /admin nos encontramos con un login bajo un CMS llamado Strapi, buscando exploits para este CMS se llegó al CVE-2019-18818 y CVE-2019-19609 los cuales poseen un [exploit](https://www.exploit-db.com/exploits/50239) en ExploitDB que se basa en ambos para ejecutar un RCE.

![images-center](/assets/images/Horizontall_HTB/exploit-CVE-2019-18818-19609.PNG)

Sin embargo, no tenemos clara la versión del CMS por lo que con ```BurpSuite``` indagamos en las rutas de la página a más profundidad encontrando un archivo bajo el directorio admin llamado "init", donde se indica la versión del CMS que corresponde a la misma versión del exploit encontrado.

![images-center](/assets/images/Horizontall_HTB/burp-admin_init.PNG)

## Explotación

Con la información recopilada, se procedió a la explotación de la vulnerabilidad. Para ello se descargó el exploit mencionado y se ejecutó con el comando
```bash
python3 CVE-2019-18818-19609.py http://api-prod.horizontall.htb
```
el cual nos brinda una pseudo shell para ejecutar comandos.

![images-center](/assets/images/Horizontall_HTB/exploits-ejecucion.PNG)

Luego de un rato probando comandos, se descubrió que es posible realizar una reverse shell con bash de la siguiente forma:

![images-center](/assets/images/Horizontall_HTB/exploit-revshell.PNG)

## User.txt

Una vez dentro se procedió a realizar la sanitización de la TTY y luego enumerar la máquina con LinPeas... sin embargo no se encontró mucha información útil. Por lo que se procedió a buscar la flag del user.

![images-center](/assets/images/Horizontall_HTB/user-find_flag_user.PNG)

Procediendo con una enumeración manual, se llegó a un archivo el cual posee un usuario y una password.

![images-center](/assets/images/Horizontall_HTB/user-BD_Creds.PNG)

Como la base de datos esta en el localhost en el puerto 3306, se indago en la red para ver que otros puertos estan abiertos encontrado lo siguiente:

![images-center](/assets/images/Horizontall_HTB/user-puertos.PNG)

## Port Forwarding

Como ya sabemos que existen algunos puertos abiertos se procedio a realizar un curl a cada uno, encontrando en el puerto 8000 un dato relevante para continuar con la máquina.

![images-center](/assets/images/Horizontall_HTB/user-curl_localhost8000.PNG)

Resulta que en el código que muestra el curl hay un div que indica la version de Laravel que utiliza. Esta version es vulnerable segun el CVE-2021-3129. Buscando en internet algun exploit se encontraron varios, uno de ellos fue este https://github.com/nth347/CVE-2021-3129_exploit, pero para ejecutarlo es necesario realizar un Port Forwarding, ya que si lo ejecutamos directamente podriamos hacer saltar alguna alerta suponiendo un caso real.

![images-center](/assets/images/Horizontall_HTB/exploit-CVE-2021-3129.PNG)

El Port Forwarding podemos realizarlo creando un par de claves con ```ssh-keygen``` una vez creadas, subimos la clave pública a la máquina, se puede hacer con 
```bash
python3 -m http.server 80
```
una vez subida la clave, en nuestra equipo de atacante asignamos privilegios 600 a la llave privada y realizamos el port con ssh tal que:
```bash
ssh -i llave_privada -L 8000:127.0.0.1:8000 strapi@horizontall.htb
```
realizado lo anterior podemos ver en el puerto 8000 de nuestra localhost la página de Laravel.

## Root.txt

Con lo anterior hecho, podemos ejecutar el exploit que encontramos. Durante el ejecución se intento realizar una reverse shell, sin emgargo, no se logro. Por lo que se procedio a ir directamente por la flag de root.

![images-center](/assets/images/Horizontall_HTB/root_flag.PNG)



