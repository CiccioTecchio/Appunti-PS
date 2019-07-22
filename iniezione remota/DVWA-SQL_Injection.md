# DVWA_low - SQL Injection

## Obiettivo della sfida
Iniettare comandi SQL arbitrari tramiti form HTML

## Strategia di attacco
Utilizziamo il **fuzz testing** ovvero inviamo rischieste anomale con lo scopo di scoprire malfunzionamenti del programma.  
Passo 1: invio di una richiesta leggittima, valida e non maliziosa ed osserviamo il comportamento del sistema   
Passo 2: invio di una richiesta **non leggittima**, **non valida** e **maliziosa** 
con lo scopo di ottenere più informazioni possibili per la costruzione di un albero di attacco.

### Cosa fare
Inseriamo nella form **_"UserID"_: 1**, l'input è sia sintatticamente che semanticamente corretto quindi ci aspettiamo una risposta corretta e infatti la otteniamo dato che vengono stampate le informazioni dell'utente con id 1.  
Proviamo adesso a dare un input sintatticamente corretto ma scorretto semanticamente, ci aspettiamo una risposta scorretta.  
Inseriamo **_"UserID"_: -1** e il sistema non ci restituisce niente, proviamo anche ad inserire **_"UserID"_: 1.0** ma la risposta è la stessa di quando abbiamo provato ad inserire 1 quindi il sistema converte in double in intero

### Riflessione dell'input
La riflessione dell'input è l'atto di un server di includere l'input di un utente nella risposta, la presenza della riflessione permetta all'attaccante di vedere il risultato dell'attacco quindi il fatto che un sistema presenti la riflessione dell'input è negativo.  
Ci accorgiamo che(in questa sfida) la riflessione è presente perchè se facciamo **_"UserID"_: 2-1** abbiamo una risposta che presenta al campo ID il valore 2-1.

## Come sfruttare le vulnerabilità
Cerchiamo di sfruttare questa vulnerailità dando come input ad UserID una striga che termina con apice singolo, notiamo che ci viene restituito un **errore di sintassi SQL**. Ci sembra di capire che la query SQL converte il valore in input in intero e preleva la riga corrispondente nella tabella.  
Proviamo ad iniettare un input che trasformi la query SQL in un'altra query che stampi tutte le tuple della tabella.  
Per fare questo effettuiamo l'iniezione di una **tautologia** essa è una condizione sempre vera a prescindere dall'input dell'utente ad esempio
```sql
SELECT t
FROM tbl
WHERE t = <valore> OR '1' = '1'
```
La clausola WHERE sarà sempre vera a prescindere dal valore di t.

### Istruzioni da eseguire
Iniettiamo una tautologia inserendo:  
`"UserID": 1' OR '1'= '1`  
Vengono stampati tutti gli utenti

### Commenti sull'iniezione di tautologie
Questo tipo di attacco presenta diverse limitazioni, dato che esso, non permette di ricostruire la struttura della query, non permette di selezionare altri campi, non permette di eseguire comandi SQL arbitrati.  
Possiamo migliorare questo tipo di attacco utilizzando la _UNION_ di SQL in maniera tale da poter concatenare due query(ovviamente la seconda sarà la nostra query arbitraria). Nella seconda query possiamo anche cercare di ottenere informazioni riguardo alla struttura del db alcuni esempi
- ottenere la versione del db:
```sql
1' UNION SELECT 1, version()#
```
- ottenere nome utente e host
```sql
1' UNION SELECT 1, user()#
```
- ottenere nome del db
```sql
1' UNION SELECT 1, database()#
```
- ottenere il nome delle tabelle
```sql
1' UNION SELECT 1, table_name
FROM information_schema.tables
WHERE table_schema = 'dvwa'#
```
## Debolezze
1. L'applicazione non neutralizza i caratteri speciali di SQL

## Mitigazione
1. Implementare un filtro per i caratteri speciali
2. attivare la script security a livello "high"
