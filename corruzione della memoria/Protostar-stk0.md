# Protostar_stack 0 - Buffer overflow

## Obiettivo della sfida
Modifica del valore della variabile `modified` a tempo di esecuzione

## Strategia di attacco
Come prima cosa otteniamo informazioni sul **S.0.** della macchina che dobbiamo attaccare, quindi digitiamo:
```bash
> lsb_release -a
# è un sistema Debian v. 6.0.3
```
per ottenere informazioni riguardo l'**architettura** digitiamo:
```bash
> arch
#i686 a 32bit Pentium II
```
per ottenere info sul **processore** montato digitiamo:
```bash
> cat /proc/cpuinfo
# info sul processore
```
### Esecuzione lecita
Come prima cosa vediamo come si comporta l'eseguibile di _stack0_ digitiamo:
```bash
> cd /opt/protostar/bin
> ./stack0
> stringa a caso
#risposta
Try again?
```
### Commento di stack0.c
Il programma stampa un messaggio di conferma se la variabile _modified_ è diversa da 0. Notiamo anche che le variabili _modified_ e _buffer_ sono spazialmente vicine, se due variabili sono contigue in memoria possiamo sovrascrivere _modified_ sfruttando la vicinanza con _buffer_.

### Ipotesi
Per verificare la fattibilità di questo attacco bisogna verificare due ipotesi:
1. `gets(buffer)` permette l'input di una stringa più lunga di 64 byte
2. _modified_ e _buffer_ sono contigue in memoria

### Verifica delle ipotesi

**Prima ipotesi**: leggiamo il manuale di `gets()` e nella sezione BUG scopriamo che `gets()`è stata deprecata e sostituita con `fgets()` perchè la prima non effettuava controlli sul numero di caratteri inseriti a differenza della seconda, **ipotesi verificata**.  
**Seconda ipotesi**: utilizziamo il comando `pmap` che stampa il layout di memeoria di un processo in esecuzione
```bash
> pmap $$
# output
```
L'output di pmap ci dice che lo **stack** del programma è piazzato su indirizza alti, l'area del codice(**TEXT**) è piazzata sugli indirizzi bassi e l'area del programma(**Global data**)è piazzata nel mezzo. Ma ancora non siamo in grado di capire se le due variabili siano contigue, per capirlo dobbiamo recuperare [informazioni sullo stack layout](http://duartes.org/gustavo/blog/post/journey-to-the-stack/).  
Scopiramo che lo stack cresce verso gli indirizzi bassi e che le variabili definite per ultime stiano in cima allo stack e che lo stack cresce verso gli indirizzi bassi, quindi allocando 68 byte alla variabile _buffer_ essa(crescendo verso il basso) andrà a riempire se stessa e a sovrascrivere il contenuto di _modified_


### Istruzioni da eseguire
Per vincere la sfida eseguiamo
```bash
> python -c 'print "a" * 65' | /opt/protostar/bin/stack0
# sfida vinta
```

## Debolezze
1. `gets()` non effettua controlli sul buffer overflow

## Mitigazione
1. sostituire la funzione`gets()` con `fgets()`
