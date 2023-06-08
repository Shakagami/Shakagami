Bueno, en esta sala exploraremos la vulnerabilidad IDOR. Es una vulnerabilidad de seguridad que ocurre cuando una aplicación web expone referencias directas a objetos internos, como bases de datos, archivos o registros, y no impone restricciones adecuadas para verificar si el usuario tiene acceso legítimo a esos objetos.

Por lo que podremos abusarnos manipulando los identificadores en la parte de la URL para asi acceder como un usuario privilegiado o si bien queremos ver dicho archivos.

Nos dirigiremos a la ip proporcionada por la plataforma, una vez alli nos dira que podremos utilizar una cuenta "guest" si apretamos `CTRL+U`.

Credenciales:

>Guest
>Guest

Al entrar, podremos ver en la parte de la URL nuestro nombre de usuario "guest". Para utilizar nuestro IDOR attack, modificaremos la parte de guest y la reemplazaremos por "admin". Una vez logrado esto, obtendremos la flag.

La URL nos quedaria: http://IPTARGET/profile.php?user=admin