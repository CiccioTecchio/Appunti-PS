# DVWA_low - Cross Site Scriptiong(XSS) Stored

## Obiettivo della sfida
Iniettare statement arbitrari JS in form HTML

## Strategia di attacco
Iniziamo con l'inviare una richiesta corretta e notiamo che l'input viene riflesso nella pagina HTML.  
Proviamo ad inserire codice HTML nella form **"Message"** e notiamo che il codice HTML viene eseguito, possiamo iniettare anche codice JS utilizzando il tag `<scipt> istruzioni JS </scipt>`.   
Possiamo ottenere i **cookie** posseduti dal browser vittima iniettando la funzione JS `document.cookie`.  
Gli attacchi visti non sono sfruttabili dato che la stampa degli alert avviene sul browser vittima non sul browser dell'attaccante, inoltre, la vittima si accorgerebbe immediatamente dell'attacco.


### XSS Stored un attacco reale
Iniettiamo uno script che imposta `document.location` ad un nuovo URL

## Debolezze
1. L'applicazione non neutralizza l'input inserito in una pagina web

## Mitigazione
1. Inserire un filto basato su **whitelist** facendo scegliere l'input in una lista di valori fidati(menu a tendina),oppure, neutralizzare i caratteri speciali
