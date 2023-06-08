Bienvenidos a otra sala de TryHackMe. Esta ocasion realizaremos una maquina blue team, en el area forense visitando registros log dentro del sistema asi como tambien explorando malware creado y como usualmente realizan ataques los malos actores siguiendo sus huellas (comandos) dentro del sistema.

# Task3 

 ## The user installed a package on the machine using elevated privileges. According to the logs, what is the full COMMAND?
 
 ```bash
 Le echamos un vistaso a los logs de autenticacion alojados en /var/log/auth.log
 y grepeamos con la palabra "sudo". Para facilitar la busqueda, grepeamos con el respectivo nombre de usuario
 
	cat /var/log/auth.log | grep "sudo"
	cat /var/log/auth.log | grep "sudo" | grep "cybert"
	
```

## What was the present working directory (PWD) when the previous command was run?

```bash
Una vez encontramos el paquete podremos ver la ruta (PWD).
```

# Task 4 

## Which user was created after the package from the previous task was installed?

```bash
Lo encontramos en los logs de la tarea anterior
```

## A user was then later given sudo priveleges. When was the sudoers file updated? (Format: Month Day HH:MM:SS)

```bash
Entre los logs de la tarea anterior, tendremos que grepear por "visudo", que es llamado cuando es editado el archivo /etc/sudoers
	cat /var/log/auth.log | grep "visudo"
```

## A script file was opened using the "vi" text editor. What is the name of this file?

```bash
Lo encontramos en los logs anteriormente utilizados grepeando por "vi"
	cat /var/log/auth.log | grep "vi"
```


# Task 5

## What is the command used that created the file bomb.sh?

```bash
Nos dirigimos al directorio /home/it-admin y listaremos los registros de comandos que se alojan en .bash_history

	cat .bash_history
	
```

## The file was renamed and moved to a different directory. What is the full path of this file now?

```bash
Anteriormente en nuestro historial de comandos, podremos ver como crea una tarea cron, por lo que tendremos que listar las actividades cron alojadas en /etc/crontab

	cat /etc/crontab
	
```

## When was the file from the previous question last modified? (Format: Month Day HH:MM)

```bash
Listamos el archivo para mostrarnos informacion detallada acerca de cuando se creo o modifico el archivo como tambien los permisos que posee.

	ls -lah /bin/os-update.sh
	
```

## What is the name of the file that will get created when the file from the first question executes?

```bash
le echaremos un vistazo a nuestro archivo malicioso para ver como esta armado y ver que mensaje interesante nos responde.

	cat /bin/os-update.sh
	
```

# Task 6

## At what time will the malicious file trigger? (Format: HH:MM AM/PM)

```bash
Podremos obtener la respuesta listando las actividades cron fijandonos en la parte inferior izquierda. Estara listado como
# m h
  0 8
```
