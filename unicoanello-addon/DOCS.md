# L'Unico Anello - HA App

[L'Unico Anello][unicoanello] è una App per Home Assistant che permette di installare una Companion App per giocare al GdR 
L'Unico Anello - Seconda Edizione.

L'Unico Anello permette di tenere traccia dello stato degli Eroi Giocanti, di tenere il diario dei viaggi effettuati dalla compagnia e 
di visualizzare in maniera semplice e indicizzata la Knowledge Base completa del gioco.

## Installazione

L'installazione di questa app è semplice ed analoga all'installazione di una qualunque altra app per Home Assistant.

1. Cliccare il bottone My Home Assistant per aprire la propria istanza HA.

   [![Apri questa app nella tua istanza Home Assistant.][addon-badge]][addon]

1. Cliccare sul bottone "Install" per avviare l'installazione.
1. Avviare l'app con il bottone "Start".
1. Effettuare un Check dei logs per verificare che l'app sia partita correttamente.
1. Pronti! Per accedere: http://<IP_HOME_ASSISTANT>:8080
1. Per la configurazione SSL e l'accesso HTTPS vedere la sezione apposita.

## Configurazione

**Nota**: _L'app va riavviata ogni volta che viene modificata la configurazione._

Configurazione di esempio:

```yaml
external: info
db_url: jdbc://mariadb://homeassistant:3306/unicoanello
db_username: unicoanello
db_password: password
external_url: https://lacompagniadelrogo.duckdns.org
java_opts: -Xmx 512m
log_level: info
mail_host: smtp.gmail.com
mail_port: 587
mail_auth: true
mail_starttls: true
mail_username: test@gmail.com
mail_password: password
storage_root: /share/unicoanello
```

**Nota**: _Questo è solo un esempio, Non copiare!_

### DB

Queste 3 options sono legate alla connessione dell'app con il database esterno 
Mariadb.

#### Option: `db_url`

L'opzione `db_url` rappresenta la JDBC connection string per connettersi
al database. La sintassi da utilizzare è la seguente: _jdbc:mariadb://[host][:port]/[database]_

#### Option: `db_username`

L'opzione `db_username` rappresenta l'utenza Mariadb da utilizzare per connettersi
al database.

#### Option: `db_password`

L'opzione `db_password` rappresenta la credenziale associata all'username da utilizzare 
per connettersi al database.

### Option: `external_url`

L'opzione `external_url` rappresenta l'url utilizzabile per raggiungere l'app dalla rete 
internet pubblica. 

**Nota**: _Qualora si decida di utilizzare un SSL Proxy, vedere la sezione apposita._

### Option: `java_opts` (opzionale)

L'opzione `java_opts` rappresenta la stringa di parametri che possono essere passati alla VM Java
tramite CLI all'avvio dell'applicazione. 

### Option: `log_level`

L'opzione `log_level` controlla il livello dei log in output e può essere 
modificato per essere più o meno verboso, ad esempio per debuggare eventi
insoliti. Valori possibili:

- `TRACE`: Livello di dettaglio massimo.
- `DEBUG`: Mostra informazioni dettagliate.
- `INFO`: Livello base, mostra solo informazioni normalmente utili.
- `WARN`: Eventi eccezzionali non frequenti.
- `ERROR`: Errori di runtime che potrebbero non richiedere azioni immediate.
- `FATAL`: Qualcosa di incredibilmente storto. App non utilizzabile.
- `OFF`: Root logging disabilitato.

**Nota**: _Ogni livello include automaticamente messaggi di log di un livello
ad una severità maggiore: `debug` mostra anche messaggi `info`. Per default,
`log_level` è impostato ad `info`, che è l'impostazione consigliata a meno che non
si stia effettuando troubleshooting_

### Mail

Queste options sono legate alle funzionalità legate all'invio di email di 
notifica da parte dell'app.

#### Option: `mail_host`

Hostname/Indirizzo IP del server di posta SMTP utilizzato per l'invio delle mail
di notifica.

#### Option: `mail_port`

Numero di porta del protocollo di trasporto utilizzato per la connessione al server
di posta.

#### Option: `mail_auth`

Specifica se il server di posta richieda o meno una procedura di autenticazione.

#### Option: `mail_starttls`

Specifica se il server di posta supporti il protocollo di sicurezza STARTTLS.

#### Option: `mail_username`

Utenza utilizzata per l'autenticazione con il server di posta.

#### Option: `mail_password`

Credenziale associata all'utenza usata per l'autenticazione col server di posta.

#### Option: `storage_root` (opzionale)

Directory dove sono memorizzate le risorse statiche (images/tiles) dell'applicazione.

## Utilizzo con Proxy SSL (Advanced Usage)

E'possibile configurare l'istanza Home Assistant per pubblicare l'applicazione su 
Internet usando un certificato Letsencrypt ed un SSL Proxy NginX. La configurazione
consigliata è quella di utilizzare le app [DuckDNS][duckdns] e [NGINX SSL Proxy][nginx].

### Configurazione DuckDNS

- Registrarsi su [DuckDNS](https://duckdns.org), ottenere il token di autenticazione e copiarlo
sull'opzione `Token`.
- Aggiungere alla lista dei `domains` l'hostname che si vuole utilizzare come indirizzo esterno
(esempio: lacompagniadelrogo.duckdns.org). 
- Abilitare Letsencrypt accettando le condizioni di utilizzo.

### Configurazione NGINX SSL Proxy

Creare un file di configurazione NGINX analogo a questo template:

```nginx
server {
	server_name lacompagniadelrogo.duckdns.org;

	listen 80;
	return 301 https://$host$request_uri;
}

server {
	server_name lacompagniadelrogo.duckdns.org;

	ssl_session_timeout 1d;
	ssl_session_cache shared:MozSSL:10m;
	ssl_session_tickets off;
	ssl_certificate /ssl/fullchain.pem;
	ssl_certificate_key /ssl/privkey.pem;

	# dhparams file
	ssl_dhparam /data/dhparams.pem;
	listen 443 ssl;
	http2 on;
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

	proxy_buffering off;

	location / {
		proxy_pass http://homeassistant:8080;
		proxy_set_header Origin $http_origin;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header Host $http_host;
		proxy_redirect http:// https://;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection $connection_upgrade;
		proxy_set_header X-Forwarded-Host $http_host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
```

Copiare questo file all'interno della cartella _/share/ngin_proxy_, assicurandosi che questa sia
configurata per i `servers` dell'app NGINX SSL Proxy.


## Supporto

Domande? Richieste?

Puoi [aprire una issue qui][issue] su GitHub.

## Autore

Il setup di questo repository è stato fatto da [Filippo Tosti][filippo].

## Licenza

MIT License

Copyright (c) 2025-2026 Filippo Tosti

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

[addon-badge]: https://my.home-assistant.io/badges/supervisor_addon.svg
[addon]: https://my.home-assistant.io/redirect/supervisor_addon/?addon=f0a20dc8_unico_anello&repository_url=https%3A%2F%2Fgithub.com%2Fhassio-addons%2Frepository
[duckdns]: https://github.com/home-assistant/addons/tree/master/duckdns
[filippo]: https://github.com/shardik86
[issue]: https://github.com/shardik86/unicoanello-ha-addons/issues
[nginx]: https://github.com/home-assistant/addons/tree/master/nginx_proxy
[unicoanello]: https://github.com/shardik86/unicoanello-ha-addons/tree/master/unicoanello-addon