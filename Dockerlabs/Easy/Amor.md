# Agua de Mayo

## Port Enumeration

To begin our scan, we use the Nmap tool  during our discovery phase. As we can see, we have the following open ports:

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

```ruby
┌──(root㉿kali)-[/home/kali]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2  
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

## Examining the Web Page and Its Infrastructure
We access the web page hosted on the Apache server and find this:

 (Foto pagina web)

```ruby
Users:
Juan
Carlota
```

Now we perform a subdomain enumeration using the tool GoBuster, obtaining the following:

```ruby
gobuster dir -u http://172.17.0.2/ -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html

```

Sin embargo no encotramos nada por lo tanto vamos a realizar un ataque de fuerza bruta con la herramienta hydra con los dos usuarios.

## Intrusion

Finally, we will try to authenticate to the SSH service.
``` ruby
user: carlota
password: babygirl
```

foto de intrusion


## Examining the imagen.jpg 

 
Si navegamos por los directorios nos encontramos una imagen, para analizarla, mejor vamos a descargarla con el siguiente comando:

```ruby
scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/kali/Desktop/amor/
```

We download the image and use tools like ExifTool or Steghide to check if there is any secret inside it.

Vemos que si no introducimos ninguna contraseñas nos encontramos con un fichero secreto.txt que contiene este texto:

```ruby
ZXNsYWNhc2FkZXBpbnlwb24=
```

Parece que es texto en base64 por lo tanto vamos a decodificarlo con:

```ruby
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 --decode
```
Ese comando nos devuelve el siguiente texto:

```ruby
lacasadepingypong
```
Parece una posible contraseña de algun usuario.

Si hacemos el comando hydra pero con el rockyou como usuario y la anterior contraseña obtenemos una coinicdencia 
```ruby
user: oscar
pwd: lacasadepingypong
```
por lo tanto accedemos con esas credenciales.

## Escalation privilege


If we run sudo -l, we see that we can execute 



Finally, we see that we are the root user.