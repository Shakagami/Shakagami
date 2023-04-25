Bienvenidos nuevamente a otra sala de TryHackMe. En este capitulo, enumeraremos el servicio con __gobuster__, buscaremos posibles brechas que nos deje para acceder a la maquina windows y escalaremos privilegios  mediante una vulnerabilidad en el ejecutable #hhpud que se encuentra dentro de la maquina victima. Sin mas preambulos, empecemos!.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap` 

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-04-24 20:39:20 EDT for 37s
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       REASON          VERSION
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/
```

Nuestro escaneo de nmap nos muestra 2 puertos abiertos con versiones microsoft. El puerto **3389** es el puerto por defecto usado para **RDP** (Remote Desktop Protocol), por lo que si encontramos sus credenciales podremos acceder via acceso remoto a su computadora microsoft.

# Enumeration

La enumeracion es clave, y si que es cierto!. Utilizaremos la herramienta __gobuster__ para enumerar directorios ocultos dentro del sitio web. Ejecutaremos : `gobuster dir -u http://IPTARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
```ruby
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://IPTARGET
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/04/24 20:47:07 Starting gobuster in directory enumeration mode
===============================================================
/retro                (Status: 301) [Size: 149] [--> http://10.10.216.45/retro/]
[!] Keyboard interrupt detected, terminating.

===============================================================
2023/04/24 20:53:36 Finished

```

Nos mostrara un directorio oculto, navegaremos en el y seguiremos enumerando hasta encontrar algo que nos llame la antencion. Una vez encontrado lo que queremos ver, obtenemos las credenciales para acceder a su maquina windows.

# Exploit

Con las credenciales obtenidas, usaremos **RDP**  para acceder a su maquina, puedes encontrar muchos servicios que te permitan realizar rdp, en mi caso utilizare __remminia__ que viene incluida en kali linux.

Una vez configurado, accedemos a su maquina y podremos ver en su escritorio un ejecutable nombrado #hhpud .  Si realizamos una busqueda a este ejecutable, nos encontraremos el nombre de su CVE #CVE-2019-1388,  utilizaremos esta informacion para abusarnos y escalar privilegios a la autoridad mas alta en microsoft windows __NT/AUTHORITY__ .

# Priv. Escalation

Seguiremos el paso a paso para poder escalar privilegios abusandonos de lo mencionado anteriormente. Ejecutaremos #hhpud :

>.-Show more details
>.-Show information certificate
>.-Verisign Commercial

Nos dirigira a un enlance, en el cual debemos apretar `CTRL+S`. Utilizaremos una wildcard para poder ver la consola , `C:\Windows\System32\`, dentro de __file name__ pondremos la wildcard `*.*` y ejecutaremos cmd.exe con `click derecho + open`.

# Respuestas

#credentials = wade : parzival

