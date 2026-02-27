# MODULAR CATSHIRE CITADEL
> **The On-Premise, AI-Augmented Modular Monolith Blueprint**

La **Modular Catshire Citadel** e' l'implementazione fisica, sicura e on-premise del *Modular Monolith Architecture Manifesto v5.3*. E' un ecosistema di sviluppo chiuso e sovrano, dove l'intelligenza artificiale open-source (Lo Stregatto) agisce come guardiano e partner di un team snello, garantendo che le regole architetturali vengano rispettate e che la velocita' di sviluppo sia massimizzata grazie allo stack Node-native (Platformatic, Fastify, Watt).

---

## 1. OVERVIEW DELL'IDEA

L'obiettivo della Catshire Citadel non e' solo scrivere codice, ma creare una **fabbrica di software automatizzata e strutturata** che prevenga il debito tecnico alla radice. 

In un tipico progetto, la pressione sui rilasci porta a violare i confini architetturali (il "falso confine" o l'accesso diretto ai dati altrui). La Catshire Citadel risolve questo problema alla base creando un ambiente on-premise dove:
1. **La Sovranita' del Dato e' assoluta:** Nessuna riga di codice, specifica di business o dato di produzione viene inviato a server cloud esterni (OpenAI, Anthropic). Tutto gira sui server aziendali tramite modelli locali.
2. **L'Architettura e' un vincolo fisico:** L'Intelligenza Artificiale (Lo Stregatto) viene istruita con il Manifesto e agisce da supervisore durante il Pair Programming, bloccando proattivamente codice che viola la purezza del dominio o le regole di accoppiamento.
3. **Dal Requisito al Codice senza frizioni:** Il flusso di lavoro parte da specifiche formali (BMAD) che guidano sia la generazione dell'infrastruttura (OpenAPI/Platformatic) sia lo scaffolding del codice da parte dell'AI.

---

## 2. ARCHITETTURA DEL SISTEMA

L'architettura della Cittadella si divide in due macro-livelli: l'infrastruttura operativa (dove risiede l'AI e il tooling) e l'architettura applicativa (il monolite vero e proprio).

### 2.1 Livello Infrastrutturale e di Tooling (On-Premise)
* **Compute Node (Docker Swarm / k3s):** Il cluster locale che ospita l'intero ambiente di sviluppo, test e produzione.
* **AI Engine (Ollama):** Il motore di inferenza locale. Fa girare modelli specializzati come *DeepSeek Coder V2* per la generazione del codice e *Llama 3.1* per il ragionamento sui log e sulle specifiche.
* **Orchestratore AI (Cheshire Cat AI):** Il "cervello" del team. Dispone di memoria a lungo termine (dove risiedono il Manifesto e gli ADR) e interviene tramite IDE o chat integrata per validare il lavoro degli sviluppatori.
* **Spec-Driven Engine (BMAD - Nearform):** Lo strumento utilizzato dal Product Owner e dal Tech Lead per scrivere e validare le specifiche di comportamento (BDD) prima che vengano tradotte in codice.

### 2.2 Livello Applicativo (Il Monolite Modulare)
* **Orchestratore di Processo (Watt):** Gestisce il ciclo di vita dei vari moduli indipendenti all'interno della stessa istanza fisica, permettendo al monolite di scalare o di essere diviso in futuro.
* **Livello API e Persistenza (Platformatic DB + Fastify):** Ogni modulo espone interfacce OpenAPI autogenerate. Platformatic gestisce la mappatura sul database locale (es. PostgreSQL) fornendo CRUD veloci per le operazioni di lettura (CQRS asimmetrico).
* **Bus degli Eventi Interno (RabbitMQ / Redis):** Il tessuto connettivo del monolite. Tutti i moduli comunicano in modo asincrono pubblicando eventi tipizzati con `evento_id`, `correlazione_id` e `causazione_id`.
* **Testing "Zero-Mock" (Testcontainers + TAP):** Nessun database finto. I test di integrazione avviano container reali effimeri per garantire che la logica di persistenza e i flussi asincroni funzionino esattamente come in produzione.

---

## 3. PROSSIMI PASSI DI SVILUPPO (BOOTSTRAPPING)

Come si avvia fisicamente la Cittadella? Utilizzando i tool stessi per auto-generare l'infrastruttura. Ecco la roadmap dei primi Sprint operativi.

### Fase 1: Spec-Driven Inception con BMAD
Prima di scrivere qualsiasi file sorgente o configurare i database, il Product Owner e il Tech Lead definiscono il "Modulo di Riferimento" (es. Gestione Anagrafica Base) utilizzando BMAD.
* **Azione:** Scrittura degli scenari `Given-When-Then` in linguaggio naturale.
* **Output Automatico:** BMAD genera lo scheletro dei test end-to-end e la prima bozza del contratto OpenAPI (`openapi.json`). Questo file diventa il contratto intoccabile del modulo.

### Fase 2: Inizializzazione del Core Engine (Watt & Platformatic)
Il DevOps e il Senior Backend Developer preparano il terreno di esecuzione.
* **Azione:** Creazione del monorepo strutturato secondo le regole della Sezione 12.1 del Manifesto.
* **Output Automatico:** Configurazione di `watt.json` per orchestrare il backend e inizializzazione di `platformatic.db.json` per mappare il database sulle specifiche OpenAPI generate da BMAD nella Fase 1.

### Fase 3: Addestramento dello Stregatto (Cheshire Cat Setup)
Si accende l'Intelligenza Artificiale e le si danno le "leggi" della Cittadella.
* **Azione:** Deploy di Ollama e Cheshire Cat sul cluster k3s. Caricamento del file `Manifesto v5.3.md` nella memoria vettoriale dello Stregatto.
* **Creazione Plugin:** Sviluppo di un plugin custom per lo Stregatto (in Python) che agganci le API di linting locale. Il plugin istruisce il Gatto a rifiutare prompt o generazioni che superano la Complessita' Cognitiva di 15 o che tentano di importare librerie dentro la cartella `nucleo/dominio/`.

### Fase 4: Scaffolding Autoguidato e TDD
Lo Junior o il Senior Developer iniziano l'implementazione del dominio lavorando in Pair Programming con lo Stregatto.
* **Azione:** Lo sviluppatore fornisce allo Stregatto i test end-to-end (generati da BMAD) e gli chiede di preparare i Test Unitari (TDD) per il nucleo del dominio.
* **Validazione:** Una volta scritti i test, lo sviluppatore implementa la logica pura in JavaScript/TypeScript. La pipeline locale esegue i test TAP e i test di mutazione.
* **Integrazione:** L'adattatore di uscita viene collegato al client di Platformatic generato nella Fase 2. La funzionalita' e' completa, documentata, e architettonicamente pura.
