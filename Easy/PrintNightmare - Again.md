Buenas, en esta ocasion traemos un analizis de un ataque llamado "PrintNightmare" que es un ataque abusando el servicio spooler "spoolsv.exe" para escalar privilegios dentro de una maquina windows y asi podes crear, modificar e incluso borrar archivos dentro de una maquina. Haremos uso del Event viewer utilizando unos filtros que nos hara la tarea mucho mas facil y rapida a la hora de analizar una amenaza.

Esta seccion estara dividida por preguntas y explicaremos como es resolvido a medida que avanzan las preguntas. Pero antes, filtraremos los datos para encontrar mas rapido el incidente. 

Abrimos nuestro EventViewer > Custom Views + `click derecho` > Create Custom View

En la seccion de Event Level seleccionaremos todos los campos.
Luego, en "By log" tendremos que seleccionar los siguientes campos 
>Microsoft-Windows-PrintService/Admin
>Microsoft-Windows-PrintService/Operational
>Microsoft-Windows-PrintService/Operational
>Microsoft-Windows-PrintService/Admin
>Microsoft-Windows-PrintService/Operational
>Microsoft-Windows-SMBClient/Security
>Windows System
>Microsoft-Windows-Sysmon/Operational(**Sysmon**)
>Microsoft-Windows-Sysmon/Operational(**Sysmon**)
>Microsoft-Windows-Sysmon/Operational(**Sysmon**)

Por ultimo, filtramos los siguientes Events ID : **316,808,811,31017,7031,3,11,23,26**
El  resultado final seria:

  ![[PrintNighmare - Image.png]]


# The user downloaded a zip file. What was the zip file saved as?

Utilizamos `CTRL+F` y buscamos por archivo .zip


# What is the full path to the exploit the user executed?
El mismo metodo utilizado anteriormente, pero esta vez buscamos la ruta donde encontramos el archivo zip y buscamos algun metodo donde se pudo haber ejecutado el archivo 

Al buscar por `C:\Users\bmurphy\` nos indicara una ruta bastante interesante `C:\Users\bmurphy\Downloads\CVE-2021-1675-main\` que al buscarla con esta nueva ruta, encontraremos la ruta donde se ejecuto el archivo malicioso.


# What was the temp location the malicious DLL was saved to? 
En nuestras busquedas anteriores, encontraremos el sitio temporal donde se guardo el .dll malicioso. Lo encontraremos dentro de la ruta `C:\Users\bmurphy\AppData\Local\Temp`


# What was the full location the DLL loads from?
Al buscar por el nombre `nightmare.dll` encontraremos la ruta #HINT=drivers

# What is the primary registry path associated with this attack?

Buscaremos por `spoolsv.exe` y veremos en la parte de `Details` ahi nos inficara la ruta primaria del registro asociado con este ataque.



# What was the PID for the process that would have been blocked from loading a non-Microsoft-signed binary?

Tendremos que agregar un nuevo filtro a nuestro log `Microsoft-Windows-Security-Mitigations` una vez filtrado, nos dirigimos al resultado que nos marco y podremos ver el PID.



# Last Questions
Para las ultimas preguntas, tendremos que observar el registro en powershell, que se aloja en la siguiente ruta `C:\Users\bmurphy\AppData\Roaming\Microsoft\Windows\Powershell\PSReadLine`
Ahi encontraremos su usuario y password mas los comandos que utilizo durante su ataque.

Si no ves la carpeta "appdata" seguramente no tengas habilitado la opcion de ver "archivos y carpetas ocultas", esto se arregla yendo a la barra de tareas en la parte de "view > options > view" y seleccionamos la opcion "show hidden files..."