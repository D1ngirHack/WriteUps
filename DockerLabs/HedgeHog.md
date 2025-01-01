
![](img/Pasted%20image%2020250101103808.png#center)


Compruebo si está activa
```
ping -c 1 172.17.0.2
```

![](img/Pasted%20image%2020250101105520.png#center)


### Escaneo de puertos
- Primero hago un reconocimiento de puertos silencioso de los puertos abiertos
```
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2
```

![](img/Pasted%20image%2020250101105541.png#center)

**Resultados del escaneo:**

| Puerto | Estado | Servicio |
| ------ | ------ | -------- |
| 22/tcp | open   | ssh      |
| 80/tcp | open   | http     |


Realizamos un segundo escaneo al puerto abierto, lanzando una serie de script por defecto de `nmap` y reconocimiento de servicios.
```
nmap -p22,80 -sVC --min-rate 5000 -n -Pn 172.17.0.2
```

![](img/Pasted%20image%2020250101105737.png#center)


| Puerto | Estado | Servicio | Versión                          |
| ------ | ------ | -------- | -------------------------------- |
| 22/tcp | open   | ssh      | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 |
| 80/tcp | open   | http     | Apache httpd 2.4.58 ((Ubuntu))   |

---


<h3><center> Análisis del servidor web HTTP (puerto 80)</center></h3>

Al introducir la IP como la dirección URL, la web nos muestra lo siguiente:
![](img/Pasted%20image%2020250101110000.png#center)

Nos muestra una palabra `tails`, que en español significa `cruz`. Puede que sea algún nombre de usuario, pero antes realizamos `fuzzing web` para intentar enumerar fichero y directorios que estén alojados en el servidor web.


#### Fuzzing Web

Primero vamos a usar la herramienta `dirb`, que realiza un escaneo rápido.
```
dirb http://172.17.0.2
```

![](img/Pasted%20image%2020250101110314.png#center)


No encuentra mucho, así que realizamos un segundo escaneo con la herramienta `gobuster`.
```
gobuster -dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php,txt,asp,aspx
```

![](img/Pasted%20image%2020250101111237.png#center)

Tampoco nos encuentra mucho. Así que realizaremos un ataque de fuerza bruta para comprobar que `tails` es un usuario y poder ver si conseguimos su contraseña.

#### Hydra
Si tomamos como usuario la palabra que nos indica la web `tails`, vamos a realizar un ataque de fuerza bruta para conseguir la contraseña del mismo por el protocolo `SSH` que también está abierto.
```
hydra -l tails -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 64
```


