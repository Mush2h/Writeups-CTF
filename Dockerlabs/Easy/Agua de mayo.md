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

 (Foto comentarios pagina web)

Parece que nos encontramos con código brainfuck por lo tanto necesitamos un traductor brainfuck a Ascii. Nos aparece esta palabra que puede ser una posible contraseña.

```ruby
bebeaguaqueessano
```
Foto del traductor

Ahora hacemos un reconocimiento de subdomninios con la herramienta dirbuster obteniendo lo siguiente:

```ruby
gobuster dir -u http://172.17.0.2/ -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html

```

Foto de agua ssh

## Examining the agua_ssh.jpg Image

Descargamos la imagen y usamos herramientas como exiftool o stehide para comprobar si existe algun secreto en su interior.
Sin embargo no encontramos nada.

foto exiftool 




## Intrusion

Finalmente vamos a intentar identificarnos con:

``` ruby
user: agua
password: bebeaguaqueessano
```

foto de intrusion


