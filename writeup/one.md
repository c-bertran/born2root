1. Connect two VM on virtualBox with a nat network

Open two vm, and go to the Debian Live

Install nmap and execute this:
nmap -T4 -A -v 10.0.2.0/24
for scanning all port

We see open ports:
22 -> ssh
143 -> tcp imap
80 -> tcp apache
993 -> ssl imap
21 -> ftp
443 -> tcp ssl apache
53

We see inside the ssl of 443 is created by BornToSec (/etc/hosts), so we have an user

2. Scanning the apache server

Install the feroxbuster and get a dictionnary
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/raft-medium-directories.txt
./feroxbuster -u 10.0.2.4 -w raft-medium-directories.txt --insecure

We get list of existing url (access it with ssl):
/
/cgi-bin
/forum
/phpmyadmin
/server-status
/webmail

3. Let's make an analyze of services installed on mahcine

- The forum is based under my little form
- Webmail is based under SquirrelMail version 1.4.22
- PhpMyAdmin is imself ^^
- The front server is an apache

Let's testing the most probable vuln software,the forum

After a check, we see 6 vuln inside Exploit Database, but urls no working, so we skip it. Let's continue the analyze of forum

4. The forum

We get this list of users:
- admin
- lmezard
- qudevide
- zaz
- wandre
- thor

A thread named Probleme login contain a extract of log file. This format is really recognizable, is /var/log/auth.log. Is contain trace of auth attempt. Is containing interessing parts if we focus on invalid user return :
```txt
Oct 5 08:44:52 BornToSecHackMe sshd[7488]: input_userauth_request: invalid user PlcmSpIp [preauth]
Oct 5 08:44:55 BornToSecHackMe sshd[7488]: Failed password for invalid user PlcmSpIp from 161.202.39.38 port 54827 ssh2
[...]
Oct 5 08:45:26 BornToSecHackMe sshd[7547]: input_userauth_request: invalid user adam [preauth]
Oct 5 08:45:29 BornToSecHackMe sshd[7547]: Failed password for invalid user !q\]Ej?*5K5cy*AJ from 161.202.39.38 port 57764 ssh2

```

Dans ces deux cas, on peut supposer que l'utilisateur a utilisé son mot de passe plutôt que le pseudonym.

A l'inverse, avec `cat ./born2root/writeup/log | grep 'session opened for user'`, on voit que trois utilisateurs sont en ssh, root, lmezard et admin.

On peut essayer les différents combinaisons en SSH, mais rien ne fonctionne. On se rabat sur le forum et lmezard arrive à se connecter via !q\]Ej?*5K5cy*AJ

En fouillant, on tombe sur la page user de admin, et on découvre qu'il possède un mail `laurie@borntosec.net`. Essayons avec le peu d'info de s'y connecter. PlcmSpIp ne fonctionne pas, mais !q\]Ej?*5K5cy*AJ si.

5. Le webmail
c'est très rapide, un mail DB Access contient l'id et de password pour phpmyadmin `root/Fg-'kKXBj87E:aJ$`