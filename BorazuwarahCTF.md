## Enumeración de puertos

Para  empezar el escaneo usamos la herramienta [[Nmap]] en nuestra fase de descubrimiento , como se puede apreciar tenemos los siguientes puertos abiertos:

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

```bash
┌──(root㉿kali)-[/home/kali]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2  
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

## Examinar la página web y su infraestructura
Accedemos a la web que esta montada en el servidor apache y nos encontramos con esto:

![[Pasted image 20240806190718.png]]

Lo primero que se me ocurre es analizar la página web en búsqueda de enlaces raros  o alguna pista, como no encuentro nada me voy a decantar por investigar si existen archivos o directorios ocultos , para ello voy a utilizar la aplicación de gobuster.

```shell
gobuster dir -u http://172.17.0.2/ -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html

```

## Examinar imagen del huevo
No encuentra nada por lo tanto lo único que se me ocurre es que la imagen del huevo kinder
tenga algún secreto esto incluye información en los metadatos o un fichero incrustado.

Para averiguarlo, voy a utilizar las herramientas Exiftool e steghide incluyendo el script creado en el otro apartado para hacer un ataque por fuerza bruta a la imagen.

![[Pasted image 20240808103308.png]]

Vemos que tanto en descripción como en "Tittle" aparece un usuario o contraseña, además utilizando el script para extraer un posible fichero incrustado.
Extraemos el fichero "secreto.txt" con el siguiente mensaje.

![[Pasted image 20240808104002.png]]

## Intrusión

Por último todo apunta a que vamos a tener que hacer un ataque de fuerza bruta en el servicio ssh con el usuario "borazuwarah" y con un diccionario de contraseñas.

Para ello voy a utilizar la herramienta [[Hydra]]

```bash
hydra -l borazuwarah -P rockyou.txt -t 10 -w 1 ssh://172.0.17.2
```

![[Pasted image 20240808105649.png]]

Hydra obtiene la contraseña con ese usuario dado por lo tanto ya podemos conectarnos al servicio ssh.