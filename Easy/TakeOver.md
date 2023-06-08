Bienvenidos a otra sala de tryhackme, en esta ocasion enumeraremos subdominios y enumeraremos un poco para obtener posibles pistas al obtener la flag, para luego ver el certificado y observar su DNS 

# Enumeration

Antes que nada , agregaremos el host a nuestro `/etc/hosts`. Utilizaremos `sudo nano /etc/hosts` y pegaremos `futurevera.thm` con su respectiva IP.

```ruby
IPTARGET   futurevera.thm
```

Iniciaremos el proceso de enumeracion de subdominios con la herramienta  **ffuf** , que nos permitira descubrir subdominios en el host. Ejecutaremos el siguiente comando
```bash
ffuf -H "HOST: FUZZ.futurevera.thm" -u https://10.10.217.35 -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -fs 0,4605

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://IPTARGET
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.futurevera.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 0,4605
________________________________________________

[Status: 200, Size: 1522, Words: 367, Lines: 34, Duration: 226ms]
    * FUZZ: support

[Status: 200, Size: 3838, Words: 1326, Lines: 81, Duration: 227ms]
    * FUZZ: blog

[WARN] Caught keyboard interrupt (Ctrl-C)

```

Como podemos observar, nos encontro dos subdominios que pertenecen al host que acabamos de consultar `support & blog`. Nuevamente, tendremos que modificar nuestro `/etc/hosts` con los subdominios encontrados recientemente y nos quedaria de la siguiente manera : 

```ruby
IPTARGET   futurevera.thm
IPTARGET   blog.futurevera.thm
IPTARGET   support.futurevera.thm
```

Luego, al dirigirnos a subdominio **support**, nos saldra como advertencia "**Warning: Potential Security Risk Ahead**", del cual procedemos a clickear en "**Advanced**" para luego mirar el certificado de la pagina.

Una vez nos reedirige al certificado, podremos ver informacion valiosa como "**Subject Name,  Public Key Info, Subject Alt Names**", entre otras cosas. Podremos ver un DNS que nos llama la atencion y lo agregaremos nuevamente a nuestro `/etc/hosts` que nos quedaria de la siguiente manera 
```ruby
IPTARGET   futurevera.thm
IPTARGET   blog.futurevera.thm
IPTARGET   support.futurevera.thm
IPTARGET   secrethelpdesk934752.support.futurevera.thm
```

Una vez dirigido al nuevo DNS que pudimos encontrar, obtendremos la flag 
# 