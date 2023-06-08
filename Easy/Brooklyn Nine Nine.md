Bienvenidos a otra sala de TryHackMe, tendremos otra maquina con dificultad facil en el que  abusaremos el servicio #ftp para obtener informacion. Luego, haremos uso de #Hydra para averiguar las credenciales, accederemos al sistema via #ssh y escalaremos privilegios con #less.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-04-26 19:06:43 EDT for 23s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Nuestro escaneo en nmap nos muestra 3 puertos abiertos. Podremos navegar y nos encontraremos una imagen. Si miramos la fuente de la pagina, podremos ver un mensaje oculto que nos dirce si alguna vez escuchamos sobre la steganografia. No pude descargar la imagen, por lo que no intente utilizar **steghide**. Nos centraremos en el servicio #ftp para acceder como usuario anonimo y poder inspeccionar el servicio.

# Instrusion

Ejecutaremos `ftp IPTARGET`, nos pedira un nombre de usuario el cual le especificaremos que entremos como usuario **Anonymous** y nos pedira una password, lo dejaremos en blanco ya que, como usuario anonimo, no necesitaremos password para poder acceder. 
Listaremos con `ls` y nos transferimos el archivo `get Note_to_jake.txt` o simplemente le echamos un vistazo `more Note_to_jake.txt`. La nota nos la envia **amy** para **jake**, diciendole que su password en muy debil. Nosotros, como atacantes, podremos utilizar #Hydra utilizando una wordlist basica para poder intentar descubrirla y acceder via #ssh .

Ejecutamos hydra : `hydra -l jake -P /usr/share/wordlists/rockyou.txt IPTARGET ssh`


Una vez obtenidas las credenciales, utilizamos #ssh para acceder al sistema como usuario jake `ssh jake@IPTARGET`.

# Priv. Escalation

Listaremos los permisos de superusuario `sudo -l`
Averiguamos comandos que puede ejecutar jake **(ALL)** , alojado en **/usr/bin/less**.
Nos dirigimos a https://gtfobins.github.io/, y buscaremos **/less. Podremos escalar privilegios a usuario root utilizando el comando que nos proporciona la pagina . 

Nos movemos al directorio `cd /usr/bin` y ejecutamos :

> `sudo less /etc/profile` 
> `!/bin/sh`

Este proceso nos abrira una shell privilegiada como raiz del sistema.

# Respuestas

#credentials = jake:987654321
