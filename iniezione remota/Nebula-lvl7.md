# Nebula_07 - Iniezione remota

## Obiettivo della sfida
Esecuzione del programma `/bin/getflag`con i privilegi dell'utente **flag07**.
Non cosideremo attacchi con login diretto, iniezione tramite variabili di ambiente e librerie condivise
dato che questi attacchi non avranno successo. Ci concetriamo sulla **iniezione diretta di comandi**

## Strategia di attacco
La cartella flag07 contiene dei file di configurazione di BASH e _index.cgi_ e _thttp.conf_.  
Controlliamo i permessi di _index.cgi_ e scopriamo che esso è leggibile ed eseguibile da tutti gli utenti e che(a differenza delle sfide precedenti) ha SETUID abbassato. 

### Commento di index.cgi
1. Crea lo scheletro di una pagina HTML
2. Con il comando `ping -c 3 IP 2>&1` invia 3 pacchetti all'host il cui indirizzo è IP
3. Stampa l'output sulla pagina HTML

### Esecuzione di index.cgi
Eseguiamo lo script in locale passando come Host 8.8.8.8 quindi:
1. loggiamo come **level07**
2. `> /home/flag07/index.cgi Host=8.8.8.8`
3. Il risultato è la stampa del ping

### Primo tentativo
Proviamo l'esecuzione sequenziale di due comandi usando il separatore "**;**"
1. loggiamo come **level07**
2. `> /home/flag07/index.cgi Host=8.8.8.8; /bin/getflag`
3. _getflag_ viene eseguito... ma non con i permessi di **flag07**

Proviamo una iniezione locale modificando il secondo punto mettendo i parametri fra viroglette, ma questa volta _getflag_ non viene eseguito.

### Perchè non funziona
La funzione `param()` fetcha solo i valori del parametro descritto nella funzione, dato che in _index.cgi_ param() viene invocata nel seguente modo `ping(param("Host"));` essa preleva solo il valore assegnato ad Host e ingora il resto. Inoltre scopiamo che il carattere "**;**" consente di separare i parametri

### Iniezione locale
Sostituiamo i caratteri speciali "**;**" e "**/**" con i rispettivi valori esadecimali essi sono rispettivamente **%3B** e **%2F** quindi tentiamo nuovamente l'attacco
```bash
> /home/flag07/index.cgi "Host=8.8.8.8%3B%2Fbin%2Fgetflag"
```
Questa volta l'iniezione locale ha successo ma _getflag_ non viene eseguito con i permessi di **flag07**.

### Iniezione remota
Per effettuare una iniezione remota bisogna identificare un server WEb che esegua _index.cgi_ con SETUID, se esiste un server del genere la sfida è vinta

## Come sfruttare le vulnerabilità
Visualizziamo i metadati di _thttpd.conf_ e scopriamo che è leggibile da tutti e modificabile solo da root, leggendo il file scopriamo che il server
1. ascolta sulla porta 7007
2. la sua directory radice è `/home/flag07`
3. vede l'intero file system dell'host
4. esegue con i permessi di **flag07**

Verifichiamo se  il server sia veramente in esecuzione sulla porta 7007 con
```bash
# verifichiamo se esistono processi di nome pgrep
> pgrep -l thttpd
# esistono
# verifichiamo se sono in ascolto sulla 7007
> netstat -ntl | grep 7007
# un processo è in ascolto su 7007
```
Non abbiamo certezza del fatto che il processo sia thttpd e non possiamo usare `netstat -ntlp` per sapere il nome del processo perchè non siamo root, quindi dobbiamo interagire direttamente con il server Web inviadogli richieste tramite il comando `nc` quindi facciamo
```bash
> nc localhost 7007
> GET / HTTP/1.0
```
L'accesso a root(/) ci è proibito ma scopriamo che il server è effettivamente _thttpd_

### Istruzioni da eseguire
Quindi il nostro vettore d'attacco verrà costruito utilizzando `nc localhost 7007` ed effettuando una GET verso _index.cgi_ passandogli come parametro Host 8.8.8.8  contatenato a /bin/getflag con i rispettivi punti e virgola e slash in esadecimanle
```bash
> #login come level07
> nc localhost 7007
> GET /index.cgi Host=8.8.8.8%3B%2Fbin%2Fgetflag
#sfida vinta
```
## Debolezze
1. _thttpd_ esegue con privilegi troppo elevati
2. _index.cgi_ non neutralizza i caratteri speciali

## Mitigazione
1. riconfigurare _thttpd_ con privilegi più bassi
2. possiamo implementare nello script Perl un filtro basato su **blacklist** quindi se un input è conforme a un formato presente nella blacklist esso viene scartato.