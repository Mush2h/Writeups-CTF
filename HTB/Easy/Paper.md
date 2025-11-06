# Paper
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Paper.png" alt="Paper" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>Information Leakage</li>
        <li>Abusing WordPress - Unauthenticated View Private/Draft Posts</li>
        <li>Abusing Rocket.Chat Bot</li>
        <li>Polkit (CVE-2021-3560) [Privilege Escalation]</li>
      </ul>
    </td>
  </tr>
</table>


## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.245 -oG allports
```

```shell
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 63
80/tcp  open  http    syn-ack ttl 63
443/tcp open  https   syn-ack ttl 63
```


# Cap 

• Insecure Directory Object Reference (IDOR)
• Information Leakage
• Abusing Capabilities (Python3.8) [Privilege Escalation]

## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.245 -oG allports
```

```shell
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

Vamos a realizar un escaneo más exaustivo de los siguiente puertos encontrados:


```shell
nmap -sCV -p21,22,80 10.10.10.245 -oN targeted
```

Se puede comprobar que no encontramos nada interesante o vulnerable.

```shell
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-title: Security Dashboard
|_http-server-header: gunicorn
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Revisamos la pag web

Con el siguiente comando voy a comprobar que nos encontramos en la página web.

```shell
whatweb http://10.10.11.143
http://10.10.11.143 [403 Forbidden] Apache[2.4.37][mod_fcgid/2.3.9], Country[RESERVED][ZZ], Email[webmaster@example.com], HTML5, HTTPServer[CentOS][Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9], IP[10.10.11.143], MetaGenerator[HTML Tidy for HTML5 for Linux version 5.7.28], OpenSSL[1.1.1k], PoweredBy[CentOS], Title[HTTP Server Test Page powered by CentOS], UncommonHeaders[x-backend-server], X-Backend[office.paper]
```

Encontramos en la cabecera X-backend[office.paper] por lo tanto editamos el fichero /etc/passwd para poder mostrar el contenido.

nos encontramos que nos lleva a un blog desarrollado en `Wordpress` con unos comentarios que son relevantes junto con un usuario `Prisonmike`el comentario es el siguiente 

```text
Michael, you should remove the secret content from your drafts ASAP, as they are not that secure as you think!
-Nick
```

Podemos acceder a post que no se encuentra en la página pero estan expuestos para ello me sirvo de la siguiente vulnerabilidad: 

```
http://wordpress.local/?static=1&order=asc
```
vemos que usando esta URL se puede acceder a una URL con filtración de otros post.

por lo tanto si lo adaptamos a nuestro caso obtengo un post donde comentan que hay un enlace de registro.
```text
# Secret Registration URL of new Employee chat system
http://chat.office.paper/register/8qozr226AhkCHZdyY
```
Podemos apreciar que hay un subdominio que se agrega al fichero hosts y vemos que es un panel de login.

En ella te puedes registrar y entrar a un chat donde se encunetra un `bot` que te proporciona datos interesantes entre ellos ofrece un usuario y una contrase de un fichero `.env`

```shell
 <!=====Contents of file ../hubot/.env=====>
export ROCKETCHAT_URL='http://127.0.0.1:48320'
export ROCKETCHAT_USER=recyclops
export ROCKETCHAT_PASSWORD=Queenofblad3s!23
export ROCKETCHAT_USESSL=false
export RESPOND_TO_DM=true
export RESPOND_TO_EDITED=true
export PORT=8000
export BIND_ADDRESS=127.0.0.1
<!=====End of file ../hubot/.env=====>
```

Tambien nos deja ver el contenido del fichero `/etc/passwd` en el que se destaca la existencia de dos usuarios relevantes
```
rocketchat❌1001:1001::/home/rocketchat:/bin/bash
dwight❌1004:1004::/home/dwight:/bin/bash
```

## Acceso ssh

Con estas credenciales intentamos acceder al servicio ssh y comprobar si podemos acceder.

```shell
ssh dwight@10.10.11.143
```
Vemos que podemos acceder con éxito, donde podemos ver la primera la flag `user.txt`

## Escalada de privilegios

Para la escalada he intnetado los comandos básicos sin embargo no he conseguido encontrar nada de manera manual por lo tanto voy a utilizar la herramienta `linpeas.sh`
```
https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```

para pasarlo he creado un server con python 

```
python3 -m http.server 8000
```
y posteriormente me he conectado a ese recurso.


Después de hacerme el escaneo  vemos que `sudo` es vulnerable, y lo podemos explotar con el `CVE-2021-3560`que utiliza el comando sudo para crear un ususario y contraseña temporal con todos los permisos por lo tanto de esta manera podemos averiguar la flag de root.txt

el enlace del exploit es el siguiente:

```
https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation/blob/main/poc.sh
```