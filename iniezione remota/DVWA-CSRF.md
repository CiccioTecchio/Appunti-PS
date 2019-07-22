# DVWA_low - Cross Site Request Forgery(CSRF)

## Obiettivo della sfida
Cambiare la password senza passare dalla form
## Strategia di attacco
Nei campi **"New password"** e **"Confirm password"** inseriamo la stringa _pswd_. La richiesta è leggittima e infatti l'output è corretto dato che ci viene restituito "password changed" ma notiamo due errori:
1. Le password immesse dall'utente vengono riflesse in chiaro quindi se c'è un attaccante che sta monitorando il traffico sarà in grado di vedere le password.
2. Si suppone che l'operazione venga effettuata da un "utente fidato" dato che dato che la modifica avviene senza verificare nessun parametro legato all'utente come ad esempio la verifica della vecchia password.

Per vincere la sfida possiamo provare ad effettuare una **richiesta contraffatta** modificando i parametri della URL _password\_new_ e _password\_conf_, la richiesta contraffatta viene poi nascosta in un'immagine, la vittima viene indotta a caricare l'immagine che una volta caricata provoca la modifica della password della vittima.

## La richiesta contraffatta
```html
<img 
src = "http://127.0.0.1:80/dvwa/vulnerabilities/csrf/?password_new=pippo&password_conf=pippo&Change=Change#"
width="0" height="0" />
```
Quando il browser valuta questa richiesta(inconsciamente) si collega alla pagina del cambio password e modifica con successo la password.

## Debolezze
1. L'applicazione non è in grado di verificare se una richiesta valida e leggittima sia stata eseguita intenzionalmente dell'utente che l'ha inviata.

## Mitigazione
1. Introdurre un elemento di casualità degli URL associati alle azioni allo scopo di distinguere le richieste lecite dalle illecite cosicchè se l'attaccante genera una URL senza passare da una form la richiesta viene scartata.
