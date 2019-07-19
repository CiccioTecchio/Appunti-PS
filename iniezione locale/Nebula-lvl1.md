# Nebula_01 - Iniezione locale

## Obiettivo della sfida
Cercare di eseguire `bin/getflag` con i privilegi dell'utente **flag01**.

## Strategia di attacco
Sappiamo che è molto difficile rompere la password e che l'amministratore non è intenzionato a fornircela,
quindi dobbiamo trovare una strategia alternativa.  
Controlliamo i permessi dell'eseguibile di `flag01` con `ls -la /home/flag01/flag01` e notiamo che l'eseguibile
è di proprietà dell'utente **flag01** ed è esguibile dal gruppo **level01**, ma notiamo anche che l'eseguibile ha il bit **SETUID** alzato.  
**IDEA**: provare a _inoculare_ l'esecuzione di `/bin/getflag` sfruttando il binario `/home/flag01/flag01`.

### Cosa fare
1. Autenticarsi come **level01** (Possibile)
2. Eseguire `/home/flag01/flag01` (Possibile)
3. Incoculare `/bin/getflag` in `/home/flag01/flag01` (bisogna capire come fare)

Per effettuare lo step 3 analizziamo il sorgente di **level1.c**
### Commento di level1.c
Esso imposta tutti gli UID e i GID al loro valore effettivo e poi tramite la funzione di libreria `system()` esse esegue un comando shell passato come argomento(restituisce -1 in caso di errore). L'ultima istruzione di **level1.c** è la seguente:
`system("/usr/bin/env echo and now what?");`.
Dal manuale della funzione `system` scopriamo che essa non funziona correttamente se`/bin/bash` punta a `bash` e su questo tipo di macchina ci troviamo proprio in questa situazione.  
Quindi la causa del malfunzionamento è il fatto che BASH quando invocata come sh **non effettua** l'abbassamento dei privilegi.
Sappiamo che la chiamata a system effettua una `echo` possiamo **inoculare** qualcosa di diverso dalla echo, questo è possibile modificando le variabili di ambiente. 

## Come sfruttare le vulnerabilità
Possiamo modificare indirettamente la stringa eseguita da system copiando `/bin/getflag` in una cartella temporanea e dandole il nome `echo` e alterando la variabile d'ambiente in modo da anticipare `tmp` a `/usr/bin` facendo  
```bash
> PATH=/tmp:$PATH
````

### Conseguenza
Il comando env prova a caricare la echo, sh individua `/tmp/echo` come primo candidato e lo esegue con i privilegi dell'utente flag01

### Istruzioni da eseguire
```bash
> cp /bin/getflag /tmp/echo   
> PATH=/tmp:$PATH
> /home/flag01/flag01
#sfida vinta
```
## Debolezze
1. i privilegi dell' eseguibile **flag01** ingiustamente elevati
2. il binario `bin/sh` non abbassa i propri privilegi
3. manipolando una variabile d'ambiente è possibile sostituire il comando `echo` con un comando che esegue lo stesso codice di `/bin/getflag`

## Mitigazione
Essendo la **vulnerabilità** un AND di debolezze è sufficiente mitigarne una per inibire le restanti(ovviamente è meglio mitigarle tutte).
Per mitigare la **prima** debolezza basta abbassare il bit di SETUID sull'eseguibile **flag01**.  
Per mitigare la **seconda** debolezza basta installare una nuova versione di BASH che eviti il mancato abbassamento dei privilegi
Per mitigare la **terza** debolezza basta impostare in maniera sicura PATH. Questo è possibile modificando il codice di **level1.c** usando la funzione `putenv()` che modifica la variabile di ambiente già impostata prima della chiamata a `system()` 