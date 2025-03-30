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
