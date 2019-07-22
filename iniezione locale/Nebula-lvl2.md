# Nebula_02 - Iniezione locale

## Obiettivo della sfida
Cercare di eseguire `bin/getflag` con i privilegi dell'utente **flag02**.

## Strategia di attacco
Inutile chiedere o cercare di rompere la password. Controlliamo i permessi dell eseguibile **flag02** con `ls -la` e notiamo che è di proprietà dell'utente flag02, è eseguibile dagli utenti del gruppo **level02** e inoltre ha SETUID alzato.  
### Cosa fare
1. Autenticarsi come **level02**(Possibile)
2. Eseguire `home/flag02/flag02`(Possibile)
3. Inoculare `/bin/getflag` in `home/flag02/flag02`(bisogna parire come fare)

Analizziamo il sorgente di **flag02** che è **level02.c**
### Commento di level02.c
1. Imposta tutti gli UID e i GID al loro valore effettivo
2. Alloca un buffer e ci scrive dentro il valore della variabile di ambiente `USER`, la scrittura avviene utilizzando la funzione `asprintf()` mentre la variabile di ambiente viene ottenuta con la funzione di libreria `getenv()`
3. Stampa il buffer
4. Esegue il comando contenuto nel buffer

Notiamo che il path del comando è scritto esplicitamente quindi non è possibile applicare l'iniezione indiretta via PATH.  
Possiamo però provare una **iniezione diretta** perchè il buffer riceve il valore della variabile di ambiente USER, quindi modificando questa variabile possiamo modificare il buffer.

## Come sfruttare le vulnerabilità
In BASH è possibile concatenare due comandi con il separatore "**;**" quindi
```bash
#settiamo user
> USER='level02; /usr/bin/id'
#proviamo ad eseguire
> /home/flag02/flag02
#la bandiere non viene catturata
```
Il problema è che nel buffer dopo la lettura della variabile di ambiente c'è la stringa "is cool" questo viene visto come parametro extra. Per togliere questi parametri possiamo provare a settere di nuovo la variabile USER come abbiamo fatto prima ma questa volta aggiungiamo un "**#**" finale, in questo modo la stringa "is cool" verrà commentata e potremo vincere la sfida.
### Istruzioni da eseguire 
```bash
#settiamo user con il commento finale
> USER='level02; /bin/getflag #'
#proviamo ad eseguire
> /home/flag02/flag02
#sfida vinta
```
## Debolezze
1. Privilegi troppo elevati a **flag02**
2. La versione di BASH non effettua l'abbassamento dei privilegi
3. Su l'input esterno non vengono escapeati i carattiri speciali

## Mitigazione
1. Abbassare il bit **SETUID** di flag02
2. Aggiornare bash
3. Utilizzare la funzione di libreria `getlogin()` che permette di escapeare i caratteri speciali
