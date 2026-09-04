# L'Unico Anello

![L'Unico Anello](src/main/resources/static/images/logo.png "L'Unico Anello")

Applicazione Spring Boot 3.5.x per la gestione delle compagnie e i diari di viaggio del GdR L'Unico Anello - Seconda Edizione, edito in Italia da NeedGames. Il software contiene anche la Knowledge Base, comodamente divisa per argomenti e copiata dal manuale base del gioco. L'admin/maestro del sapere si occupa anche della creazione di mappe, aggiungere personaggi/NPC, luoghi, dicerie, avversari e tesori, attingendo da altro materiale o dalla propria fantasia. 

> Buon viaggio nell'Eriador.

Il software comprende:

- Una base dati *MariaDB*
- Una GUI *Thymeleaf*
- Un set di *API* per eventuali sviluppi futuri
- Un addon per *Home Assistant*

### **Database** 

E'costituito da una base dati MariaDB, dove sono mappate le entità principali del gioco e memorizzati i valori che caratterizzano lo stato dei giocatori, le loro compagnie ed i loro viaggi. L'accesso ai dati è regolato tramite *Spring Data JPA 3.5.x*. Prerequisito all'utilizzo è quindi la presenza di uno schema. Lo [Schema](src/main/resources/schema.sql) costituisce il punto di partenza, includendo anche tutta la KB "non statica" del gioco.

### **GUI**

L'interfaccia Web è esposta sulla porta 8080. L'accesso è regolato tramite *Spring Security 6.x* e prevede l'autenticazione di utenti tramite username/password e l'assegnazione di determinate authorities (ADMIN, GM, PLAYER, USER). Le principali tecnologie di frontend utilizzate sono:

- [Bootstrap 5](https://getbootstrap.com/docs/5.3) (tramite webjar) - Per la struttura delle pagine
- [Fontawesome 6](https://fontawesome.com/v6/search?ic=free-collection) (tramite webjar) - Per le icone
- [Leaflet](https://leafletjs.com/reference.html) - Per la visualizzazione delle mappe

### **API**

Usando Spring JPA API Rest sono state esposte via API (/api/**) le Entity mappate usando i Repository JPA.

### **Mappe**

Le mappe disponibili sono state create usando **MapTiler Engine 14**. Questa è la procedura: 
- scaricare una mappa ad una risoluzione sufficientemente elevata da potere essere spezzettata in tiles (2K +). Eventualmente dovesse essere necessario fare upscaling, si può usare [waifu2x](https://www.waifu2x.net/)
- Caricare l'immagine PNG ad alta risoluzione su MapTiler Engine
- Come Geographical Location scegliere Bounding Box (West South East North) ed impostare:
    * West = 0
    * South = 0
    * East = larghezza in pixel dell'immagine (ES: 12000)
    * North = altezza in pixel dell'immagine (ES: 8000)
- Come Input Coordinate System Selezionare **WGS 84 /  Pseudo-Mercator (EPSG:3857)**
- Caricare la mappa complessiva nella cartella [mappe](src/main/resources/static/images/mappe/) e salvarla con l'ID della mappa
- Caricare le tiles generate da MapTiler in [tiles](src/main/resources/static/tiles/) all'interno di una sottocartella avente come nome l'ID della mappa

### **Home Assistant**

Il progetto può essere deployato come App Home Assistant, usando il repository https://github.com/shardik86/unicoanello-ha-addons. Nel deploy non è incluso MariaDB, che deve essere fornito separatamente. Nota: il repository costituisce una proiezione pubblica del repository "unicoanello" privato: i deu repository, pur rimanendo separati, sono collegati ed uniformati tramite workflow:

                        shardik86/unicoanello

    git tag vX.x.x (tramite Maven Release Plugin - mvn release:prepare)
                                  │
                                  ▼
                            GitHub Actions
                                  │
                     ┌────────────┴────────────┐
                     │                         │
                     ▼                         ▼
                Maven build              Docker Buildx
                     │                         │
                     │                         ▼
                     │                    GHCR :X.x.x
                     │                         │
                     └────────────┬────────────┘
                                  │
                                  │ repository_dispatch
                                  │
                                  ▼
    
                  shardik86/unicoanello-ha-addons
                  ─────────────────────────────────
    
                           update-addon.yml
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                    ▼             ▼             ▼
                 config        README       CHANGELOG
                 X.x.x         vX.x.x        vX.x.x
                    │             │             │
                    └─────────────┼─────────────┘
                                  │
                                  ▼
                        commit + tag vX.x.x
                                  │
                                  ▼
                           Home Assistant
                                  │
                                  ▼
                   ghcr.io/shardik86/unicoanello:X.x.x

	
