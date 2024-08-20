# BorazuwarahCTF

## Port Enumeration

To begin our scan, we use the Nmap tool in our discovery phase. As we can see, we have the following open ports:

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

```bash
┌──(root㉿kali)-[/home/kali]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2  
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

## Examining the Web Page and Its Infrastructure
We access the web page hosted on the Apache server and find this:

![Descripción de Borazu](Imagenes/Borazu_1.png)

The first thing that comes to mind is to analyze the web page in search of strange links or any clues. As I don't find anything, I decide to investigate if there are hidden files or directories. For this, I'm going to use the Gobuster application.

```shell
gobuster dir -u http://172.17.0.2/ -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html

```

## Examining the Egg Image
It doesn't find anything, so the only thing I can think of is that the Kinder egg image might have some secret, including information in the metadata or an embedded file.

To find out, I'm going to use the Exiftool and Steghide tools, including the script created in the other section to perform a brute force attack on the image.

![Descripción de Borazu](Imagenes/Borazu_2.png)

We see that both in the description and in "Title" a username or password appears. Additionally, using the script to extract a possible embedded file, we extract the file "secreto.txt" with the following message.

![Descripción de Borazu](Imagenes/Borazu_3.png)

## Intrusion

Finally, everything points to the need for a brute force attack on the SSH service with the username "borazuwarah" and a password dictionary.

For this, I'm going to use the Hydra tool

```bash
hydra -l borazuwarah -P rockyou.txt -t 10 -w 1 ssh://172.0.17.2
```

![Descripción de Borazu](Imagenes/Borazu_4.png)

Hydra obtains the password for that given user, so we can now connect to the SSH service.
