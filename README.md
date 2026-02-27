# Modular Monolith Architecture Manifesto

> "L'architettura non e' cio' che costruiamo oggi. E' cio' che rendiamo possibile domani senza rimpiangere le scelte di ieri."

**Versione 5.0 -- 2026**

**Autore:** Antonio Cittadino -- awesome.cit.dev@gmail.com

Questo documento definisce lo stile architetturale e la metodologia di sviluppo adottati per la costruzione di sistemi software complessi. Non e' una guida teorica: e' un contratto operativo tra sviluppatori, tech lead e stakeholder su come il codice deve essere scritto, organizzato e fatto evolvere nel tempo. Non prescrive tecnologie, linguaggi o framework specifici: definisce principi, pratiche e strutture che ogni team puo' implementare con gli strumenti piu' adatti al proprio contesto.

**Framework e metodologie di ispirazione:** Domain-Driven Design (Eric Evans, Vaughn Vernon), Hexagonal Architecture -- Ports and Adapters (Alistair Cockburn), Clean Architecture (Robert C. Martin), Event-Driven Architecture, Extreme Programming (Kent Beck), Trunk-Based Development (Paul Hammant), Continuous Delivery (Jez Humble, Dave Farley), Infrastructure as Code (Kief Morris), Specification by Example (Gojko Adzic), Semantic Versioning (Tom Preston-Werner).

---

## Indice

1. Perche' questo documento
2. Il ruolo del Tech Lead e la relazione con lo stakeholder
3. Extreme Programming come metodo di lavoro
4. Il modello di riferimento
5. I cinque principi fondamentali
6. Regole operative
7. Strategia di testing: TDD, BDD e Spec-Driven Development
8. Strategia di versionamento: Trunk-Based Development con Feature Flags
9. Versionamento semantico: progetto, moduli e artefatti
10. Analisi automatica della qualita' e complessita' cognitiva
11. Infrastructure as Code
12. Struttura di progetto
13. Moduli esportabili: portabilita' cross-progetto
14. Anti-pattern e trappole comuni
15. Rischi operativi e sfide pratiche
16. Il percorso di crescita
17. Il contratto del team
18. Boost allo sviluppo con agenti LLM
19. AI-Augmented Development: Agenti e Architettura
20. Bibliografia

---

## 1. Perche' questo documento

Ogni sistema software cresce. Quello che inizia come un insieme coerente di moduli, senza regole esplicite e condivise, tende inevitabilmente verso il caos: dipendenze circolari, logica di business dispersa nell'infrastruttura, moduli che si parlano direttamente bypassando ogni confine architetturale. Il risultato e' un sistema che nessuno vuole toccare.

Questo manifesto nasce dall'osservazione che la maggior parte dei problemi architetturali non deriva da scelte tecnologiche sbagliate, ma dall'assenza di regole esplicite e condivise su come il codice deve essere strutturato. Un team che scrive codice senza un contratto condiviso produce inevitabilmente un'architettura che riflette le abitudini individuali di ciascun membro, non una visione coerente del sistema.

L'architettura Modular Monolith con principi selezionati da DDD, Hexagonal Architecture ed Event-Driven Design rappresenta la risposta pragmatica: rigore dove serve, semplicita' dove basta, e un percorso di crescita naturale verso microservizi se e quando il business lo richiedera'. Ogni modulo e' progettato per essere esportabile: puo' vivere dentro il monolite, essere estratto come servizio autonomo, o essere riutilizzato in un progetto completamente diverso con un dominio differente.

**Esempio concreto.** Immagina un ospedale che costruisce un sistema per la gestione delle sale operatorie. Il primo anno il team e' composto da quattro persone e il sistema gestisce una singola struttura. Senza un manifesto, ogni sviluppatore organizza il codice secondo le proprie abitudini: uno mette la logica di prenotazione insieme alla logica di fatturazione perche' "tanto si usano insieme", un altro accede direttamente ai dati dei pazienti dal modulo di pianificazione chirurgica perche' "e' piu' veloce". Dopo diciotto mesi il sistema deve essere adottato da tre ospedali aggiuntivi, ciascuno con regole diverse. Modificare la logica di fatturazione per un ospedale rompe la prenotazione per gli altri. Il manifesto previene questo scenario stabilendo fin dal giorno uno le regole che rendono ogni modulo indipendente e sostituibile.

---

## 2. Il ruolo del Tech Lead e la relazione con lo stakeholder

Il Tech Lead non e' un capo gerarchico che distribuisce compiti. E' il ponte tra il team tecnico e lo stakeholder di business. Questa funzione e' strutturale, non opzionale: senza un canale di comunicazione diretto e continuo tra chi costruisce il software e chi ne riceve il valore, il team rischia di costruire la cosa sbagliata nel modo giusto, o la cosa giusta nel modo sbagliato.

---

### 2.1 Contatto diretto con lo stakeholder

Il Tech Lead mantiene un canale di comunicazione diretto con lo stakeholder, senza intermediari. Questo non significa bypassare il product owner: significa che quando una decisione tecnica ha conseguenze di business (e quasi tutte le hanno), il Tech Lead puo' verificare direttamente con lo stakeholder se il compromesso e' accettabile.

La comunicazione avviene in cicli brevi. Il Tech Lead non aspetta la fine di un'iterazione per mostrare il progresso: condivide incrementi visibili e funzionanti il piu' frequentemente possibile. L'obiettivo e' che lo stakeholder non debba mai chiedersi "a che punto siamo?" perche' la risposta e' sempre visibile.

**Esempio concreto.** Il team sta costruendo il modulo di reportistica per una catena di cliniche. Il Tech Lead ha una conversazione settimanale con il direttore sanitario (stakeholder). Durante la terza settimana, il direttore menziona casualmente che il report settimanale deve essere disponibile entro lunedi' mattina alle sette, non "entro lunedi'" genericamente. Questa informazione cambia una decisione tecnica: il calcolo dei report deve essere asincrono e schedulato nella notte di domenica, non on-demand quando l'utente apre la pagina. Senza il contatto diretto, questa informazione sarebbe emersa in produzione come un bug di performance.

---

### 2.2 Puntare al valore, non alla funzionalita'

Ogni decisione del team -- tecnica, architetturale, organizzativa -- deve essere valutata in base al valore che porta allo stakeholder, non in base alla complessita' tecnica o alla "eleganza" della soluzione. Questo principio ha conseguenze pratiche quotidiane.

Il Tech Lead chiede "quale problema risolviamo?" prima di "come lo implementiamo?". Le storie vengono prioritizzate in base al valore di business, non alla sequenza tecnica piu' comoda. Se due funzionalita' hanno lo stesso costo di sviluppo ma una porta dieci volte piu' valore, si fa quella. Se una funzionalita' porta poco valore ma richiede molto lavoro, il Tech Lead ha il dovere di dirlo allo stakeholder e proporre alternative.

**Esempio concreto.** Lo stakeholder chiede un cruscotto con quaranta metriche in tempo reale. Il Tech Lead, dopo aver analizzato l'uso effettivo del sistema, scopre che il team medico guarda solo cinque metriche con regolarita'. Propone di costruire prima un cruscotto con quelle cinque metriche, rilasciarlo in produzione, e poi aggiungere le altre in base all'uso reale. Lo stakeholder accetta. Il risultato e' che il valore arriva in due settimane invece di due mesi, e meta' delle metriche originali non verranno mai richieste perche' il cruscotto semplificato copre gia' le esigenze reali.

---

### 2.3 Feedback continuo e cicli brevi

Il feedback non e' un evento pianificato: e' un flusso continuo. Il team rilascia incrementi piccoli e frequenti, lo stakeholder li vede e li usa, il feedback ritorna al team e influenza le decisioni successive. Questo ciclo deve essere il piu' breve possibile.

Le pratiche concrete che abilitano il feedback continuo sono: rilasci frequenti in ambienti accessibili allo stakeholder (almeno settimanali, idealmente quotidiani), sessioni dimostrative brevi (quindici minuti, non un'ora) alla fine di ogni iterazione, canale di comunicazione informale per domande rapide (il Tech Lead risponde entro la giornata, non entro la settimana), e metriche di utilizzo reale visibili al team per capire se cio' che e' stato costruito viene effettivamente usato.

Il Tech Lead e' responsabile di proteggere il team dalle interruzioni non strutturate (lo stakeholder che chiama lo sviluppatore alle tre del pomeriggio con una richiesta urgente), ma anche di garantire che il team non lavori in isolamento costruendo qualcosa che nessuno ha chiesto.

**Esempio concreto.** Il team rilascia la prima versione del modulo prenotazione sale con la funzionalita' base: prenotazione, visualizzazione calendario, annullamento. Lo stakeholder la usa per una settimana e torna con un feedback: "funziona, ma il flusso di prenotazione urgente e' troppo lento, servono tre passaggi e in emergenza ne serve uno solo". Il Tech Lead porta il feedback al team, che aggiunge una funzionalita' "prenotazione rapida" nell'iterazione successiva. Il ciclo completo -- rilascio, uso reale, feedback, correzione -- e' avvenuto in due settimane.

---

## 3. Extreme Programming come metodo di lavoro

L'Extreme Programming (XP) e' la metodologia di sviluppo adottata dal team. Non e' stata scelta per moda: e' stata scelta perche' i suoi valori -- comunicazione, semplicita', feedback, coraggio e rispetto -- sono esattamente quelli che servono per mantenere un'architettura modulare pulita nel tempo. Un'architettura fatta di moduli indipendenti richiede un team che comunica costantemente, che ha il coraggio di refactorare, e che riceve feedback rapido sulla qualita' delle proprie decisioni.

---

### 3.1 Le pratiche XP adottate

**Pair Programming.** Due sviluppatori lavorano insieme sullo stesso problema. Non e' un lusso: e' un investimento. Quando uno sviluppatore scrive il codice del domain layer e l'altro osserva, le violazioni dei principi del manifesto vengono intercettate in tempo reale, prima che diventino debito tecnico. Il pair programming e' particolarmente efficace quando si lavora sui confini tra moduli, dove il rischio di accoppiamento e' piu' alto.

Esempio concreto. Lo sviluppatore A sta implementando la logica di emissione fattura e ha bisogno di sapere il nome del paziente. Il suo istinto e' andare a leggere direttamente dal modulo anagrafica. Lo sviluppatore B, che sta osservando, segnala che questo viola il principio dei confini tra moduli. Insieme trovano la soluzione corretta: il modulo fatturazione ascolta un evento che contiene gia' il dato necessario, oppure espone una richiesta al modulo anagrafica attraverso l'interfaccia pubblica. Questo scambio dura due minuti e risparmia ore di refactoring futuro.

**Test-Driven Development (TDD).** Il codice nasce dai test, non il contrario. Si scrive prima il test che descrive il comportamento atteso, si verifica che fallisca, si scrive il codice minimo per farlo passare, si migliora la struttura senza cambiare il comportamento. Questa pratica e' il motore dello sviluppo del domain layer ed e' approfondita nella sezione 7.

**Continuous Integration.** Il codice viene integrato nel ramo principale almeno una volta al giorno. L'integrazione frequente e' possibile solo se ogni integrazione e' verificata da una suite di test automatici completa. Il manifesto prescrive Trunk-Based Development come strategia di versionamento (sezione 8) proprio per rendere l'integrazione continua una pratica naturale, non un obiettivo aspirazionale.

**Small Releases.** Il sistema viene rilasciato in produzione in incrementi piccoli e frequenti. Ogni rilascio aggiunge valore visibile e verificabile. I feature flag (sezione 8) rendono questo possibile anche quando una funzionalita' non e' ancora completa: il codice va in produzione ma la funzionalita' rimane nascosta fino a quando non e' pronta.

**Simple Design.** L'architettura giusta e' la piu' semplice che risolve il problema di oggi mantenendo aperta la strada per il problema di domani. Questo significa che non si introducono architetture distribuite, code di messaggi o database separati per lettura e scrittura finche' non servono davvero. Il percorso di crescita (sezione 16) definisce esattamente quando e perche' fare questi salti.

Esempio concreto. Il team sta valutando se usare un sistema di messaggistica distribuito per la comunicazione tra moduli. Il modulo di sala operatoria pubblica eventi quando un intervento inizia o finisce, e il modulo di fatturazione li ascolta. Con dieci interventi al giorno, un sistema di messaggistica distribuito e' sovra-ingegnerizzazione: un meccanismo di eventi in-process basta e avanza. La regola del Simple Design dice di usare il meccanismo piu' semplice che funziona, sapendo che l'architettura a porte e adattatori permette di sostituire il meccanismo di trasporto senza toccare la logica di business quando i volumi cresceranno.

**Refactoring.** Il codice viene continuamente migliorato nella sua struttura interna senza cambiarne il comportamento esterno. Il refactoring non e' un'attivita' pianificata come uno sprint separato: e' parte del ciclo quotidiano di sviluppo, integrato nel ritmo Red-Green-Refactor del TDD. La suite di test automatici garantisce che ogni refactoring sia sicuro.

**Collective Code Ownership.** Ogni membro del team puo' modificare qualsiasi parte del sistema. Non esistono "proprietari" di moduli. Questo e' cruciale per un'architettura modulare: se solo una persona capisce il modulo di fatturazione, il team ha un collo di bottiglia umano che e' peggiore di qualsiasi collo di bottiglia tecnico. Il manifesto e i test sono gli strumenti che rendono possibile la proprieta' collettiva -- un nuovo sviluppatore puo' leggere i test di un modulo e capire cosa fa senza chiedere a nessuno.

**Coding Standards.** Il team segue standard di codifica condivisi e verificati automaticamente. Il manifesto stesso e' lo standard architetturale; gli strumenti di analisi statica ne automatizzano il rispetto. Nessuna variazione personale sulle convenzioni di naming, sulla struttura delle cartelle, o sulla modalita' di comunicazione tra moduli.

**Sustainable Pace.** Il team lavora a un ritmo sostenibile nel lungo periodo. Le pratiche XP -- TDD, pair programming, integrazione continua -- non sono "extra" che si fanno quando c'e' tempo: sono il modo in cui si lavora sempre. Un team che salta i test per consegnare prima accumula debito che rallenta il team dopo. Il manifesto lo dice chiaramente: nessuna proposta di modifica viene accettata se viola i principi, indipendentemente dall'urgenza.

**Planning Game.** Il lavoro viene pianificato in iterazioni brevi con il coinvolgimento diretto del product owner e, attraverso il Tech Lead, dello stakeholder. Le storie vengono stimate in punti relativi, non in ore. La velocita' del team viene misurata empiricamente e usata per pianificare le iterazioni successive. Le specifiche scritte in linguaggio naturale (sezione 7) sono lo strumento di comunicazione tra product owner e sviluppatori.

---

### 3.2 Il ritmo di lavoro quotidiano

Una giornata tipo del team segue questo ritmo: il team si riunisce brevemente al mattino per allinearsi sulle priorita'. Le coppie si formano e iniziano a lavorare sulle storie dell'iterazione corrente, partendo sempre dalla specifica condivisa con il product owner. Si scrivono i test, si implementa il codice, si refactora. Il codice viene integrato nel ramo principale piu' volte durante la giornata. Prima di integrare, la pipeline automatica verifica che tutto sia verde. A fine giornata il team ha prodotto incrementi piccoli, testati e integrati.

Esempio concreto. La coppia di sviluppatori sta lavorando sulla storia "il sistema deve impedire la prenotazione di una sala operatoria gia' occupata in quella fascia oraria". La mattina, insieme al product owner, scrivono la specifica con gli scenari: cosa succede se la sala e' libera, cosa succede se e' occupata, cosa succede se c'e' una sovrapposizione parziale. Poi scrivono i test per il domain layer partendo dalla regola piu' semplice (sala completamente libera), li fanno passare, e procedono con i casi piu' complessi. A fine mattinata la logica di domain e' completa e testata. Nel pomeriggio collegano la logica agli adattatori di persistenza e verificano che il flusso funzioni dall'esterno del modulo. Prima di sera, il codice e' integrato nel ramo principale con la pipeline verde.

---

## 4. Il modello di riferimento

L'architettura e' un **Modular Monolith**: un'unica applicazione deployata, ma internamente organizzata in moduli indipendenti con confini espliciti, interfacce pubbliche definite e comunicazione attraverso eventi interni. Ogni modulo e' un Bounded Context nel senso del Domain-Driven Design: ha il proprio modello, il proprio linguaggio e le proprie regole, e puo' esistere in isolamento completo dagli altri.

Ogni modulo adotta internamente un'**architettura esagonale** (Ports and Adapters): il core del dominio e' circondato da porte -- interfacce astratte che definiscono come il mondo esterno interagisce con il modulo -- e da adattatori che implementano quelle porte per una specifica tecnologia o un specifico contesto di business. Questo rende ogni modulo non solo indipendente dal framework, ma portabile tra progetti diversi.

E' importante chiarire come i concetti di Hexagonal Architecture e Clean Architecture si relazionano in questo manifesto, perche' la confusione tra i due e' uno degli errori piu' comuni. L'architettura esagonale definisce la forma del modulo: un nucleo circondato da porte e adattatori. La Clean Architecture di Robert Martin definisce la direzione delle dipendenze: sempre verso il centro, mai verso l'esterno. In questo manifesto i due concetti si compongono senza conflitto: la forma e' esagonale (porte in ingresso, porte in uscita, adattatori sostituibili) e la regola delle dipendenze e' quella della Clean Architecture (il dominio non conosce l'infrastruttura). Non usiamo i "cerchi concentrici" della Clean Architecture come struttura di cartelle perche' la metafora dell'esagono con porte e adattatori comunica piu' chiaramente l'intento di sostituibilita' e portabilita'.

**Esempio concreto.** Pensa a un modulo di gestione prenotazioni in un sistema ospedaliero. Il nucleo del modulo sa cos'e' una prenotazione, quali regole di business la governano (non puoi prenotare una sala gia' occupata, un intervento di chirurgia maggiore richiede almeno tre ore), e quali eventi pubblica quando qualcosa accade (prenotazione confermata, prenotazione annullata). Questo nucleo non sa se i dati vengono salvati su un database relazionale, su un database documentale, o su un foglio di carta. Non sa se la richiesta arriva da un'interfaccia web, da un'app mobile, o da un messaggio di integrazione con un altro sistema. Le porte definiscono cosa il nucleo offre e cosa si aspetta dal mondo esterno. Gli adattatori implementano il come. Se domani l'ospedale cambia il database, si sostituisce l'adattatore di persistenza. Se si aggiunge un canale di accesso mobile, si aggiunge un nuovo adattatore di ingresso. Il nucleo rimane intatto.

---

## 5. I cinque principi fondamentali

Ogni principio e' selezionato perche' porta un beneficio concreto e misurabile senza introdurre complessita' operativa sproporzionata. Sono ordinati per priorita' di adozione: i primi tre vanno integrati dal primo giorno di sviluppo, gli altri due nel secondo ciclo.

---

### P1 -- Bounded Context con interfaccia pubblica esplicita (ispirato a Domain-Driven Design)

Ogni modulo espone un'interfaccia pubblica che costituisce il suo contratto con il mondo esterno. Questa interfaccia e' l'unico punto di accesso lecito al modulo. Nessun altro modulo puo' accedere direttamente alle strutture interne, ai meccanismi di persistenza, ai dettagli implementativi o ai modelli interni di un altro modulo. Tutto cio' che non e' esplicitamente esposto e' privato per definizione.

**Regola operativa.** Se uno sviluppatore e' tentato di accedere direttamente ai dati di un altro modulo -- perche' "e' piu' semplice" o "piu' veloce" -- sta violando un confine. La soluzione e' sempre una delle tre: esporre un servizio pubblico nel modulo che possiede il dato, ascoltare un evento che trasporta il dato necessario, oppure chiedere al team se il confine tra i due moduli e' tracciato nel punto giusto.

Esempio concreto. In un sistema di gestione clinica, il modulo di pianificazione chirurgica ha bisogno di sapere se un paziente ha allergie a farmaci specifici. L'informazione sulle allergie appartiene al modulo anagrafica pazienti. Lo sviluppatore non deve andare a leggere la tabella delle allergie nel database dell'anagrafica: deve usare l'interfaccia pubblica del modulo anagrafica, che espone un servizio "ottieni profilo allergie per paziente". In questo modo, se l'anagrafica cambia il modo in cui memorizza le allergie, il modulo di pianificazione chirurgica non ne risente.

---

### P2 -- Regola delle dipendenze unidirezionale (ispirato a Clean Architecture e Hexagonal Architecture)

All'interno di ogni modulo le dipendenze puntano sempre verso il centro: gli adattatori dipendono dalle porte, le porte dipendono dal dominio. Il dominio non conosce nulla del mondo esterno -- non sa quale framework viene usato, quale database, quale protocollo di comunicazione. E' pura logica di business espressa nel linguaggio del dominio.

Questo principio si applica rigorosamente: il nucleo del dominio non contiene annotazioni del framework applicativo, non lancia eccezioni specifiche del protocollo di comunicazione, non importa librerie di persistenza. Se il domain layer dipende da qualcosa che non e' pura logica di business, la dipendenza e' nella direzione sbagliata.

**Beneficio immediato.** Il domain layer e' testabile in millisecondi, senza avviare l'applicazione, senza un database, senza simulazioni complesse.

Esempio concreto. La regola di business "uno sconto fedelta' del 10% si applica ai clienti con piu' di cinque acquisti nell'ultimo anno" vive nel domain layer. E' espressa in pura logica: riceve il numero di acquisti, calcola lo sconto, restituisce l'importo scontato. Non sa se il conteggio degli acquisti viene da un database, da un servizio esterno, o da un file. Questa purezza rende la regola testabile con un test unitario che dura un millisecondo e leggibile da chiunque senza conoscere il framework.

---

### P3 -- Eventi di dominio interni (ispirato a Event-Driven Architecture)

I moduli comunicano tra loro attraverso eventi, mai attraverso chiamate dirette. Quando un modulo completa un'operazione rilevante per il dominio, pubblica un evento. Gli altri moduli interessati lo ascoltano e reagiscono in autonomia, senza che il modulo pubblicante sappia della loro esistenza.

Questa e' una distinzione fondamentale: il modulo che pubblica l'evento non "chiama" gli altri moduli. Annuncia un fatto -- qualcosa che e' accaduto nel suo dominio. Chi ascolta e cosa fa con quell'informazione non e' responsabilita' del pubblicante. Questo disaccoppiamento rende il sistema estensibile: aggiungere un nuovo modulo che reagisce a un evento esistente non richiede alcuna modifica al modulo che lo pubblica.

Gli eventi trasportano dati sufficienti affinche' il sottoscrittore possa reagire senza dover interrogare il pubblicante. Ogni evento ha un identificativo univoco e un timestamp. Gli eventi non trasportano l'intero modello dell'entita': trasportano solo i dati rilevanti per comunicare il fatto accaduto.

Esempio concreto. Quando un intervento chirurgico viene completato, il modulo di sala operatoria pubblica un evento "intervento completato" che contiene l'identificativo dell'intervento, l'identificativo del paziente, il tipo di intervento, la durata effettiva e l'esito. Il modulo di fatturazione ascolta questo evento e genera la fattura. Il modulo di cartella clinica ascolta lo stesso evento e aggiorna lo storico del paziente. La sala operatoria non sa che fatturazione e cartella clinica esistono. Se domani si aggiunge un modulo di ricerca clinica, bastera' aggiungere un nuovo ascoltatore -- zero modifiche ai moduli esistenti.

---

### P4 -- Separazione comandi e query (ispirato a CQRS)

All'interno di ogni modulo, le operazioni di scrittura (comandi) e le operazioni di lettura (query) sono separate in percorsi distinti. Questo non significa avere database separati o infrastrutture duplicate: significa semplicemente che il codice che modifica lo stato del sistema e il codice che legge lo stato del sistema vivono in classi, file e percorsi logici separati.

Questa separazione e' "leggera" nella Fase 1: stesso database, percorsi logici diversi. La separazione fisica diventa necessaria solo quando i volumi lo richiedono, e a quel punto il punto di innesto e' gia' pronto senza dover ristrutturare il codice.

Una conseguenza pratica importante: la separazione comandi/query consente un'asimmetria intenzionale nello sforzo architetturale. I comandi (scritture) attraversano sempre il dominio puro con tutte le sue regole e validazioni. Le query (letture) possono bypassare il dominio e interrogare direttamente il livello di persistenza per restituire modelli di trasferimento, senza passare per la ricostruzione dell'entita' di dominio. Questo dimezza il boilerplate di mappatura per le operazioni di sola lettura, mantenendo la purezza del dominio solo dove serve -- sulle scritture che modificano lo stato.

Esempio concreto. Nel modulo di prenotazione sale, il comando "prenota sala per intervento" valida la richiesta contro le regole di business, verifica la disponibilita', modifica lo stato della sala, e pubblica l'evento "sala prenotata". La query "mostra disponibilita' sale per la prossima settimana" legge lo stato corrente delle prenotazioni e produce una vista ottimizzata per la visualizzazione a calendario, senza ricostruire le entita' di dominio. Queste due operazioni vivono in percorsi separati perche' hanno esigenze diverse: il comando richiede validazione rigorosa e consistenza transazionale; la query richiede velocita' e puo' tollerare dati leggermente non aggiornati.

---

### P5 -- Segregazione delle interfacce sui modelli di trasferimento (ispirato a Interface Segregation Principle)

Ogni modulo espone modelli di trasferimento (i dati che escono dall'interfaccia pubblica) che rappresentano solo cio' che il consumatore ha il diritto e la necessita' di conoscere. I modelli di trasferimento non espongono mai la struttura interna delle entita' del dominio. La conversione tra il modello interno e il modello esposto avviene al confine del modulo.

Esempio concreto. Il modulo anagrafica pazienti ha internamente un modello complesso con storico delle residenze, relazioni familiari, assicurazioni, e consensi firmati. Quando il modulo di pianificazione chirurgica chiede i dati di un paziente, non riceve l'intero modello: riceve un modello "paziente per pianificazione" che contiene solo nome, data di nascita, codice fiscale, allergie note e gruppo sanguigno. Ogni consumatore riceve esattamente cio' di cui ha bisogno, niente di piu'.

---

### Priorita' di adozione

Se il team deve scegliere solo tre principi con cui partire, in ordine di impatto: primo, Bounded Context con interfaccia pubblica -- impedisce l'accoppiamento prima che nasca; secondo, eventi di dominio interni -- disaccoppia i moduli nel tempo senza infrastruttura aggiuntiva; terzo, regola delle dipendenze unidirezionale -- rende la logica di business testabile e indipendente dal framework fin dal giorno uno.

---

## 6. Regole operative

Le regole operative traducono i principi in decisioni quotidiane. Non sono negoziabili a meno che un Architecture Decision Record (ADR) non documenti una deroga approvata dal team.

---

### 6.1 Accesso tra moduli: solo attraverso l'interfaccia pubblica

L'unico punto di contatto che un modulo puo' avere con un altro modulo e' la sua interfaccia pubblica. Questa interfaccia espone servizi, modelli di trasferimento ed eventi. Tutto il resto -- entita' interne, meccanismi di persistenza, logica di validazione, strutture intermedie -- e' invisibile dall'esterno.

Il rispetto di questa regola viene verificato automaticamente tramite strumenti di analisi statica che bloccano gli accessi non autorizzati in tempo reale nell'editor dello sviluppatore.

Esempio concreto. Il modulo di reportistica ha bisogno di dati aggregati sugli interventi eseguiti nell'ultimo mese. Lo sviluppatore potrebbe essere tentato di eseguire una query direttamente sulle tabelle del modulo sala operatoria perche' "e' solo una lettura". Questa e' una violazione. La soluzione corretta e' che il modulo di reportistica costruisca una propria proiezione dei dati ascoltando gli eventi che la sala operatoria gia' pubblica.

---

### 6.2 Il nucleo del dominio e' puro

Il nucleo del dominio non contiene annotazioni, decoratori, o dipendenze dal framework applicativo. Non lancia eccezioni legate al protocollo di comunicazione. Non importa librerie di persistenza. E' scritto nel linguaggio di programmazione puro, usando solo i costrutti del linguaggio e le librerie standard.

Le entita' del dominio hanno costruttori controllati e metodi di fabbrica che validano gli invarianti di business al momento della creazione. E' fisicamente impossibile creare un'entita' in uno stato inconsistente. I value object sono immutabili: una volta creati non possono essere modificati, solo sostituiti con nuove istanze.

Esempio concreto. L'entita' "Intervento Chirurgico" nel suo dominio conosce le seguenti regole: un intervento non puo' durare meno di quindici minuti, un intervento di chirurgia maggiore richiede almeno un chirurgo primario e un assistente, un intervento non puo' essere completato se non e' stato prima avviato. Queste regole sono espresse nel linguaggio del dominio chirurgico, non nel linguaggio del framework o del database.

---

### 6.3 Eventi tipizzati, tracciabili, e gestori idempotenti

Ogni evento e' una struttura dati con campi espliciti e tipizzati, mai un contenitore generico con una mappa chiave-valore. La struttura base di ogni evento nel sistema contiene obbligatoriamente tre identificativi:

L'**identificativo dell'evento** e' un valore univoco generato al momento della creazione dell'evento. Serve a garantire l'idempotenza dei gestori: il gestore verifica se un evento con quell'identificativo e' gia' stato elaborato prima di procedere.

L'**identificativo di correlazione** (Correlation ID) e' un valore che nasce nel punto di ingresso della richiesta originale (ad esempio quando un utente effettua un'azione dall'interfaccia) e viaggia identico attraverso tutti gli eventi generati a cascata da quella richiesta. Serve a ricostruire l'intera catena di operazioni che una singola azione utente ha scatenato nel sistema. Se un utente prenota una sala e questo genera un evento "sala prenotata" che a sua volta genera un evento "notifica inviata" che a sua volta genera un evento "log registrato", tutti e tre gli eventi condividono lo stesso identificativo di correlazione.

L'**identificativo di causazione** (Causation ID) e' l'identificativo dell'evento specifico che ha scatenato l'evento corrente. Serve a ricostruire la relazione padre-figlio tra gli eventi. Nell'esempio precedente, l'evento "notifica inviata" ha come causazione l'identificativo dell'evento "sala prenotata".

Pseudocodice della struttura base:

```
STRUTTURA EventoDiDominio:
    evento_id         : identificativo univoco
    correlazione_id   : identificativo della richiesta originale
    causazione_id     : identificativo dell'evento che ha causato questo
    timestamp         : momento in cui l'evento e' stato creato
    tipo              : nome qualificato dell'evento
    versione_schema   : versione della struttura dati dell'evento
```

La combinazione di questi tre identificativi rende ogni flusso di eventi completamente tracciabile. Uno strumento di tracciamento distribuito puo' usare l'identificativo di correlazione per visualizzare l'intera cascata di operazioni scatenata da una singola azione utente. Questo risolve il problema dell'opacita' della comunicazione ad eventi: il flusso non e' visibile nelle chiamate dirette, ma e' completamente ricostruibile dagli identificativi.

Ogni gestore di eventi deve essere **idempotente**: ricevere lo stesso evento due volte non deve produrre effetti collaterali. L'idempotenza si realizza verificando, prima di ogni operazione, se l'evento con quell'identificativo e' gia' stato elaborato.

Esempio concreto. Un utente annulla una prenotazione. Il modulo prenotazioni pubblica l'evento "prenotazione annullata" con un Correlation ID generato dalla richiesta web. Il modulo fatturazione ascolta l'evento e genera una nota di credito: prima di procedere, verifica che non esista gia' una nota di credito per quell'evento (idempotenza tramite evento_id). Il modulo notifiche ascolta lo stesso evento e invia un'email al paziente. Tre giorni dopo, il team di supporto deve capire perche' il paziente non ha ricevuto l'email. Con il Correlation ID, lo strumento di tracciamento mostra l'intera catena: richiesta web, evento "prenotazione annullata", evento "nota di credito emessa" (successo), evento "notifica email" (fallimento per indirizzo non valido). Il problema e' identificato in secondi, non in ore.

---

### 6.4 Nessun accesso diretto ai dati tra moduli

Un modulo non puo' eseguire query che coinvolgono strutture dati di un altro modulo. Nessuna giunzione tra tabelle di moduli diversi, nessuna lettura diretta dei dati altrui. Se un modulo ha bisogno di dati da un altro modulo, le strategie sono tre: invocare il servizio pubblico del modulo che possiede il dato (approccio sincrono), ascoltare gli eventi e costruire una propria copia locale dei dati rilevanti (approccio asincrono), oppure utilizzare un modulo aggregatore dedicato che ascolta eventi da piu' moduli e costruisce viste di sola lettura.

Esempio concreto. La direzione sanitaria vuole un cruscotto che mostra, per ogni sala operatoria, il numero di interventi eseguiti nel mese, il tasso di occupazione, e il fatturato generato. Questi dati provengono da tre moduli diversi. La soluzione corretta e' un modulo "cruscotto direzionale" che ascolta gli eventi dei tre moduli e costruisce una propria proiezione locale dei dati, ottimizzata per la visualizzazione. Il cruscotto diventa autonomo: se uno dei tre moduli e' temporaneamente rallentato, il cruscotto mostra dati leggermente non aggiornati ma continua a funzionare.

---

## 7. Strategia di testing: TDD, BDD e Spec-Driven Development

Il testing non e' un'attivita' che si fa "dopo". E' il driver dello sviluppo. Ogni modulo segue una piramide di test a tre livelli, ciascuno con il proprio approccio metodologico e il proprio scopo. I test sono la documentazione vivente del sistema: quando un test passa, documenta un comportamento garantito; quando fallisce, documenta un contratto violato.

---

### 7.1 Livello 1 -- Test unitari con TDD (nucleo del dominio)

Il nucleo del dominio si sviluppa in Test-Driven Development puro. La sequenza e' sempre: scrivi il test che descrive il comportamento atteso, verifica che fallisca (rosso), scrivi il codice minimo per farlo passare (verde), migliora la struttura senza cambiare il comportamento (refactoring). Poi ripeti.

**Cosa si testa a questo livello.** Invarianti di business, transizioni di stato, calcoli, validazioni dei value object, e generazione corretta degli eventi di dominio.

**Cosa non si testa a questo livello.** Persistenza, comunicazione di rete, serializzazione, comportamento del framework.

**Obiettivo di copertura:** novanta percento o superiore sul nucleo del dominio.

Esempio concreto. Lo sviluppatore deve implementare la regola "una sala operatoria non puo' essere prenotata se e' gia' in manutenzione programmata". Inizia scrivendo un test che crea una sala in stato "in manutenzione" e tenta di prenotarla, aspettandosi un rifiuto. Il test fallisce. Lo sviluppatore scrive il codice minimo. Poi aggiunge un test per il caso in cui la manutenzione finisce prima dell'orario di prenotazione. Poi un terzo per la sovrapposizione parziale. Ad ogni passo, il codice cresce solo quanto basta per far passare il nuovo test.

---

### 7.2 Livello 2 -- Test di integrazione con BDD (porte e adattatori)

Il livello di integrazione verifica che le porte e gli adattatori funzionino correttamente insieme. Qui si adotta il Behavior-Driven Development con sintassi "dato-quando-allora" (Given-When-Then), perche' i test a questo livello descrivono comportamenti osservabili dall'esterno del modulo.

I test di integrazione si scrivono prima come specifiche leggibili in linguaggio naturale strutturato, poi si implementano come test automatici.

**Obiettivo di copertura:** ottanta percento o superiore sul livello applicativo.

Esempio concreto -- specifica in linguaggio naturale:

Funzionalita': Emissione fattura con numerazione progressiva. Come sistema di fatturazione, voglio emettere fatture con numero progressivo annuale per garantire la tracciabilita' fiscale.

Scenario: emissione di una fattura in bozza. Dato che esiste una fattura in stato "bozza" con importo di mille euro, e dato che il prossimo numero progressivo disponibile e' "2026/042", quando emetto la fattura, allora la fattura passa allo stato "emessa", e il numero assegnato e' "2026/042", e viene pubblicato un evento "fattura emessa" con l'importo e il numero, e l'evento contiene un Correlation ID valido.

Scenario: tentativo di emettere una fattura gia' emessa. Dato che esiste una fattura in stato "emessa" con numero "2026/041", quando provo a emetterla nuovamente, allora ricevo un errore "la fattura e' gia' stata emessa", e lo stato rimane invariato, e nessun evento viene pubblicato.

---

### 7.3 Livello 3 -- Test end-to-end con Spec-Driven Development (modulo completo)

I test end-to-end verificano il modulo dall'esterno, esattamente come lo userebbe un client reale. Questi test sono Spec-Driven: la specifica viene scritta prima dell'implementazione e funge da criterio di accettazione della storia.

I test end-to-end di un modulo avviano solo quel modulo, non l'intera applicazione. Se un test end-to-end del modulo fatturazione richiede che anche il modulo anagrafica sia avviato, il modulo fatturazione non e' veramente indipendente.

I test end-to-end non misurano copertura di codice ma copertura di specifiche. Ogni scenario nella specifica deve avere un test end-to-end corrispondente.

---

### 7.4 Il flusso completo: dalla specifica al codice

Primo passo -- Specifica (Spec-Driven). Il product owner e lo sviluppatore scrivono insieme gli scenari in linguaggio naturale strutturato.

Secondo passo -- Scheletro dei test end-to-end. Lo sviluppatore crea i test end-to-end corrispondenti. Falliscono tutti perche' il codice non esiste ancora.

Terzo passo -- TDD nel dominio. Lo sviluppatore implementa la logica di business usando TDD classico.

Quarto passo -- BDD per l'integrazione. Si scrivono i test di integrazione "dato-quando-allora" che verificano il collegamento tra porte e adattatori.

Quinto passo -- Verifica end-to-end. I test end-to-end creati al secondo passo dovrebbero ora passare tutti.

Sesto passo -- Refactoring finale. Con tutti i test verdi a tutti e tre i livelli, lo sviluppatore puo' refactorare in sicurezza.

---

## 8. Strategia di versionamento: Trunk-Based Development con Feature Flags

Il team adotta **Trunk-Based Development** come strategia di ramificazione del codice sorgente. Esiste un unico ramo principale e tutto il codice viene integrato in questo ramo il piu' frequentemente possibile -- idealmente ogni giorno, al massimo ogni due giorni.

---

### 8.1 Perche' Trunk-Based e non strategie con rami longevi

Le strategie con rami multipli longevi funzionano per team grandi con rilasci schedulati a mesi di distanza. Per team piccoli-medi che praticano consegna continua, introducono un costo senza beneficio proporzionale: conflitti di integrazione dolorosi, rami obsoleti che divergono dal ramo principale, integrazione ritardata che scopre problemi troppo tardi.

---

### 8.2 Le regole del Trunk-Based

**Rami di breve durata.** Se un ramo vive piu' di due giorni, sta vivendo troppo a lungo.

**Pipeline automatica su ogni integrazione.** Ogni integrazione nel ramo principale scatena la pipeline completa: analisi statica, controllo dei tipi, test unitari, test di integrazione, test end-to-end, analisi della complessita' cognitiva, test di mutazione, costruzione dell'artefatto. Se la pipeline fallisce, il team ferma tutto e corregge. Il ramo principale non e' mai in stato di errore.

**Nessun ramo di rilascio permanente.** I rilasci avvengono dal ramo principale tramite etichette semantiche. Se serve una correzione urgente su una versione in produzione, si crea un ramo temporaneo dall'etichetta, si applica la correzione, si rilascia, e si riporta la correzione nel ramo principale.

---

### 8.3 Feature Flags: il rilascio e' separato dal deployment

Il codice viene integrato nel ramo principale anche se la funzionalita' non e' completa. Questo e' possibile grazie ai feature flag: interruttori che separano il concetto di "deployment" (il codice e' in produzione) da "rilascio" (la funzionalita' e' visibile agli utenti).

Un feature flag e' un'interfaccia che risponde alla domanda "questa funzionalita' e' attiva?". L'implementazione puo' essere banale (una variabile d'ambiente) o sofisticata (un servizio di configurazione remoto con attivazione graduale). In entrambi i casi, il nucleo del dominio non ne sa nulla -- il flag vive nell'adattatore, mai nel dominio.

**Ciclo di vita di un feature flag.** Il flag nasce quando la funzionalita' inizia lo sviluppo, vive durante il rilascio progressivo, e muore quando la funzionalita' e' stabile. I flag dimenticati -- quelli che sopravvivono piu' di trenta giorni dopo il rilascio completo -- sono debito tecnico e vanno rimossi attivamente.

**Tipologie di flag.** I flag di rilascio nascondono funzionalita' incomplete (durata: giorni o settimane). I flag operativi sono interruttori permanenti per disattivare integrazioni esterne in caso di problemi. I flag sperimentali permettono di testare varianti su un sottoinsieme di utenti. I flag di permesso controllano l'accesso a funzionalita' basate sul piano o sul ruolo.

---

## 9. Versionamento semantico: progetto, moduli e artefatti

Il versionamento non e' un'attivita' burocratica: e' il meccanismo che comunica al team, allo stakeholder e ai sistemi automatici la natura di ogni cambiamento. Il manifesto adotta un sistema di versionamento a due livelli -- progetto e moduli -- con una strategia di tagging che collega il codice sorgente agli artefatti di deployment.

---

### 9.1 Conventional Commits con scope dichiarati

Ogni integrazione nel ramo principale segue il formato Conventional Commits. Il formato e' composto da un tipo, uno scope, e una descrizione.

I tipi ammessi sono: **feat** (nuova funzionalita'), **fix** (correzione difetto), **refactor** (modifica struttura senza cambiare comportamento), **test** (aggiunta o modifica test), **docs** (aggiornamento documentazione), **chore** (manutenzione), **perf** (miglioramento prestazioni), **ci** (modifiche pipeline), **infra** (modifiche Infrastructure as Code).

Gli scope dichiarati corrispondono ai moduli del sistema e alle aree trasversali. Ogni progetto mantiene una lista degli scope validi nella configurazione degli strumenti di analisi, e l'integrazione viene rifiutata automaticamente se lo scope non e' nella lista.

Pseudocodice del formato:

```
TIPO(SCOPE): descrizione breve in minuscolo senza punto finale

[corpo opzionale con spiegazione dettagliata]

[footer opzionale]
BREAKING CHANGE: descrizione del cambiamento incompatibile
```

Esempi concreti:

```
feat(fatturazione): aggiunta emissione fattura con numero progressivo
fix(prenotazioni): correzione sovrapposizione fasce orarie notturne
refactor(anagrafica): estrazione value object per codice fiscale
test(sala-operatoria): aggiunta scenari BDD per intervento urgente
infra(pipeline): aggiunta step analisi complessita' cognitiva

feat(fatturazione)!: nuovo formato modello di trasferimento fattura

BREAKING CHANGE: il campo "importo" e' stato sostituito da "importo_lordo"
e "importo_netto" per supportare lo split payment
```

---

### 9.2 Versionamento semantico di primo livello: il progetto

Il progetto segue il Semantic Versioning (SemVer) con il formato MAGGIORE.MINORE.PATCH. La versione viene calcolata automaticamente dalla pipeline analizzando i commit dal tag precedente.

Pseudocodice del flusso di calcolo:

```
FUNZIONE calcola_prossima_versione(versione_corrente, lista_commit):
    ha_breaking = QUALSIASI commit IN lista_commit CON tipo "!" O footer "BREAKING CHANGE"
    ha_feature = QUALSIASI commit IN lista_commit CON tipo "feat"
    ha_fix = QUALSIASI commit IN lista_commit CON tipo "fix" O "perf" O "refactor"

    SE ha_breaking:
        RITORNA versione_corrente.incrementa_maggiore()
    ALTRIMENTI SE ha_feature:
        RITORNA versione_corrente.incrementa_minore()
    ALTRIMENTI SE ha_fix:
        RITORNA versione_corrente.incrementa_patch()
    ALTRIMENTI:
        RITORNA versione_corrente
```

---

### 9.3 Versionamento semantico di secondo livello: i moduli interni

Ogni modulo mantiene la propria versione semantica indipendente. La versione viene calcolata automaticamente filtrando i commit per scope. Ogni modulo tiene un file di manifesto nella propria directory radice che dichiara la versione corrente.

Pseudocodice:

```
FUNZIONE calcola_versione_modulo(modulo, versione_corrente_modulo, lista_commit):
    commit_del_modulo = FILTRA lista_commit DOVE scope == modulo.nome

    SE commit_del_modulo E' VUOTA:
        RITORNA versione_corrente_modulo

    RITORNA calcola_prossima_versione(versione_corrente_modulo, commit_del_modulo)
```

---

### 9.4 Tagging a due livelli: codice sorgente e artefatti

**Primo livello: tag sul repository del codice sorgente.** Quando la pipeline calcola una nuova versione del progetto, applica un tag al commit corrispondente (formato: v2.5.0).

**Secondo livello: tag sull'immagine di deployment.** L'artefatto costruito viene etichettato con due tag: la versione semantica del progetto e l'identificativo breve del commit. Il doppio tag permette di risalire sia dalla versione al codice sorgente, sia dall'immagine in esecuzione al commit esatto.

Pseudocodice del flusso completo:

```
FUNZIONE pipeline_rilascio(commit):
    versione_progetto = calcola_prossima_versione(tag_corrente, commit_da_ultimo_tag)
    PER OGNI modulo IN moduli_del_progetto:
        modulo.versione = calcola_versione_modulo(modulo, commit_da_ultimo_tag)
        aggiorna_file_manifesto(modulo)

    crea_tag_repository("v" + versione_progetto, commit.id)
    artefatto = costruisci_immagine(commit)
    tagga_immagine(artefatto, versione_progetto)
    tagga_immagine(artefatto, commit.id_breve)
    tagga_immagine(artefatto, "latest")
    pubblica_immagine(artefatto)
    genera_changelog(commit_da_ultimo_tag)
    notifica_team(versione_progetto, changelog)
```

---

### 9.5 Il changelog automatico

Il changelog e' generato automaticamente dai messaggi dei commit, raggruppato per tipo e per modulo.

Pseudocodice della struttura:

```
## v2.5.0 (2026-03-15)

### Cambiamenti incompatibili
- fatturazione: nuovo formato modello di trasferimento fattura

### Nuove funzionalita'
- fatturazione: aggiunta emissione fattura con numero progressivo
- prenotazioni: aggiunto supporto prenotazione multi-sala

### Correzioni
- sala-operatoria: risolto calcolo durata interventi notturni

### Moduli aggiornati
- fatturazione: v1.8.2 -> v1.9.0
- prenotazioni: v1.3.0 -> v1.4.0
- sala-operatoria: v2.1.3 -> v2.1.4
```

---

## 10. Analisi automatica della qualita' e complessita' cognitiva

La qualita' del codice non puo' dipendere solo dalla disciplina individuale e dalle revisioni manuali. Il manifesto richiede strumenti automatici che misurano, segnalano e bloccano le violazioni in tempo reale -- nell'editor dello sviluppatore, nella pipeline, e nei report del team.

---

### 10.1 Complessita' cognitiva

La complessita' cognitiva misura quanto e' difficile per un essere umano leggere e comprendere un pezzo di codice. E' diversa dalla complessita' ciclomatica (che conta i percorsi logici): la complessita' cognitiva pesa di piu' l'annidamento, le interruzioni di flusso, e le strutture che richiedono al lettore di tenere piu' cose in mente contemporaneamente.

Il manifesto stabilisce soglie massime di complessita' cognitiva per unita' di codice: complessita' massima 15 per funzione o metodo, massima 50 per classe o modulo, massima 100 per file sorgente. Queste soglie vengono verificate automaticamente nell'editor e nella pipeline.

Esempio concreto. Lo sviluppatore scrive un gestore di eventi che contiene un ciclo annidato con tre livelli di condizionali. Lo strumento di analisi mostra immediatamente che la complessita' cognitiva e' 23, sopra la soglia di 15. Lo sviluppatore decompone la funzione in tre funzioni piu' piccole: una che valida l'input, una che esegue la trasformazione, e una che gestisce la persistenza. La complessita' di ciascuna e' sotto 10.

---

### 10.2 Le metriche monitorate automaticamente

**Copertura dei test (classica).** La pipeline verifica che il nucleo del dominio abbia copertura almeno al novanta percento e il livello applicativo almeno all'ottanta percento. La copertura non e' un obiettivo in se': e' un indicatore che il TDD viene praticato correttamente. Per ridurre il rumore, gli strumenti di misurazione escludono automaticamente dalla copertura i modelli di trasferimento, i file di configurazione puramente dichiarativi, e le costanti. Pretendere il cento percento su un file di costanti non ha alcun valore e distorce le metriche.

**Test di mutazione.** La copertura classica misura se una riga di codice e' stata eseguita da un test, ma non verifica se il test sta effettivamente controllando qualcosa di significativo. Un test che chiama una funzione senza verificare il risultato produce copertura ma non produce sicurezza. I test di mutazione risolvono questo problema: uno strumento automatico modifica il codice sorgente in modo controllato (ad esempio inverte una condizione, cambia un operatore aritmetico, rimuove una riga) e rilancia i test. Se i test rimangono verdi dopo la modifica, significa che non stanno testando il comportamento modificato -- il "mutante" e' sopravvissuto. Un alto tasso di mutanti sopravvissuti e' un segnale inequivocabile che la suite di test e' superficiale.

Il manifesto richiede un tasso di eliminazione dei mutanti (mutation score) di almeno l'ottanta percento sul nucleo del dominio. Il test di mutazione viene eseguito nella pipeline su ogni integrazione, ma solo sul nucleo del dominio dove il costo computazionale e' contenuto (perche' il dominio e' puro e i test sono veloci). Per i livelli di integrazione e end-to-end, il test di mutazione viene eseguito periodicamente (ad esempio una volta alla settimana) perche' il costo e' significativamente piu' alto.

Esempio concreto. Il team ha una copertura classica del novantacinque percento sul modulo fatturazione. La pipeline di test di mutazione modifica la condizione "se l'importo e' maggiore di zero" in "se l'importo e' minore di zero" e rilancia i test. Tutti i test rimangono verdi. Questo significa che nessun test sta verificando il caso di importo negativo -- una regola di business fondamentale ("una fattura non puo' avere importo negativo") che si pensava fosse coperta. Il mutante viene segnalato, lo sviluppatore aggiunge il test mancante, e alla successiva esecuzione il mutante viene eliminato.

**Duplicazione del codice.** Blocchi di logica duplicati in piu' punti sono un segnale di astrazione mancante. La pipeline segnala duplicazioni superiori a un numero configurabile di righe consecutive.

**Profondita' delle dipendenze.** Catene di dipendenze troppo lunghe rendono il sistema fragile.

**Conformita' ai confini dei moduli.** Nessun modulo importa file interni di un altro modulo, non esistono dipendenze circolari, la cartella condivisa non cresce oltre una dimensione ragionevole.

**Dimensione delle integrazioni.** Integrazioni troppo grandi sono un segnale che il ramo ha vissuto troppo a lungo.

---

### 10.3 Il flusso automatico di check nella pipeline

La pipeline esegue i controlli dal piu' veloce al piu' lento. Se un controllo fallisce, la pipeline si ferma e segnala il problema senza eseguire i controlli successivi.

Pseudocodice del flusso della pipeline:

```
FUNZIONE pipeline_integrazione(commit):
    // Gate 1: Analisi statica (secondi)
    ESEGUI analisi_stile_codice(commit.file_modificati)
    ESEGUI verifica_tipi(commit.file_modificati)
    ESEGUI verifica_confini_moduli(commit.file_modificati)
    ESEGUI verifica_scope_commit(commit.messaggio, lista_scope_validi)
    SE ERRORI: FALLISCI CON report_errori

    // Gate 2: Complessita' e qualita' (secondi)
    ESEGUI analisi_complessita_cognitiva(commit.file_modificati)
    ESEGUI analisi_duplicazione(commit.file_modificati)
    ESEGUI analisi_profondita_dipendenze(commit.file_modificati)
    SE SOGLIE_SUPERATE: FALLISCI CON report_metriche

    // Gate 3: Test unitari (secondi a minuti)
    risultati_unit = ESEGUI test_unitari()
    copertura_domain = MISURA copertura(livello="domain")
    SE copertura_domain < 90%: FALLISCI CON "Copertura domain insufficiente"
    SE risultati_unit.falliti > 0: FALLISCI CON report_test

    // Gate 4: Test di mutazione sul dominio (minuti)
    risultati_mutazione = ESEGUI test_mutazione(livello="domain")
    SE risultati_mutazione.mutation_score < 80%:
        FALLISCI CON "Mutation score insufficiente: " + risultati_mutazione.mutanti_sopravvissuti
    // Nota: il gate 4 verifica la QUALITA' dei test, non solo la quantita'.
    // Se la copertura e' 95% ma il mutation score e' 60%, i test non stanno
    // verificando il comportamento in modo significativo.

    // Gate 5: Test di integrazione (minuti)
    risultati_integration = ESEGUI test_integrazione()
    copertura_app = MISURA copertura(livello="application")
    SE copertura_app < 80%: FALLISCI CON "Copertura application insufficiente"
    SE risultati_integration.falliti > 0: FALLISCI CON report_test

    // Gate 6: Test end-to-end (minuti)
    risultati_e2e = ESEGUI test_e2e()
    copertura_spec = MISURA copertura_specifiche()
    SE copertura_spec < 100%: SEGNALA "Scenari specifica non coperti"
    SE risultati_e2e.falliti > 0: FALLISCI CON report_test

    // Gate 7: Costruzione e versionamento (minuti)
    versione = calcola_prossima_versione(tag_corrente, commit_da_ultimo_tag)
    artefatto = costruisci_immagine(commit)

    // Gate 8: Suggerimento versione e tag
    MOSTRA "Versione suggerita: " + versione
    MOSTRA "Moduli aggiornati:" + lista_moduli_cambiati_con_versioni

    SE ramo == ramo_principale:
        applica_tag_repository(versione, commit)
        tagga_e_pubblica_immagine(artefatto, versione, commit.id_breve)
        genera_e_pubblica_changelog()

    RITORNA SUCCESSO
```

---

### 10.4 Suggerimento automatico del versionamento

La pipeline non si limita a calcolare la versione: la suggerisce attivamente al team. Se lo sviluppatore ha fatto un commit di tipo "fix" ma la pipeline rileva che un'interfaccia pubblica e' cambiata in modo incompatibile, la pipeline segnala il conflitto e suggerisce di rietichettare il commit come breaking change.

---

## 11. Infrastructure as Code

L'infrastruttura del sistema -- ambienti, configurazioni, reti, servizi di supporto, permessi -- e' definita come codice sorgente, versionata insieme all'applicazione, e gestita con gli stessi processi di revisione e test del codice applicativo. Non esistono configurazioni manuali che vivono solo nella memoria di un operatore.

---

### 11.1 Il principio fondamentale

L'infrastruttura viene trattata come un altro modulo del sistema: ha i propri file sorgente, la propria suite di test, e il proprio ciclo di vita. Ogni ambiente e' definito dallo stesso codice sorgente con parametri diversi. Le modifiche seguono lo stesso flusso del codice applicativo: ramo breve, revisione del team, pipeline automatica.

---

### 11.2 Cosa viene codificato

Ambienti e risorse computazionali. Servizi di supporto (database, cache, code di messaggi, monitoraggio). Configurazioni e segreti (i segreti sono gestiti tramite un sistema dedicato, mai come testo in chiaro). Pipeline di costruzione e rilascio. Permessi e sicurezza.

---

### 11.3 Il test dell'infrastruttura

I **test statici** verificano correttezza sintattica e conformita' alle policy di sicurezza. I **test di piano** verificano che le modifiche proposte siano quelle attese prima di applicarle. I **test di conformita'** verificano dopo l'applicazione che l'infrastruttura reale corrisponda alla definizione.

Esempio concreto. Il team deve aggiungere un ambiente di staging per un nuovo cliente. Lo sviluppatore duplica la definizione dell'ambiente di produzione, cambia i parametri di scala, e lancia la pipeline. In quindici minuti l'ambiente e' pronto, identico alla produzione nella struttura, e la configurazione e' versionata.

---

## 12. Struttura di progetto

La struttura delle cartelle e' il primo strumento di comunicazione dell'architettura. Un nuovo sviluppatore che entra nel progetto dovrebbe poter leggere la struttura delle directory e capire immediatamente i confini del dominio, la separazione tra nucleo e adattatori, e dove trovare i test -- senza leggere una riga di codice.

---

### 12.1 Vista generale

```
radice-progetto/
|
|-- sorgenti/
|   |-- moduli/
|   |   |
|   |   |-- _riferimento/                <-- MODULO DI RIFERIMENTO (Reference Implementation)
|   |   |   |-- LEGGIMI                   <-- spiega che questo e' il modello da seguire
|   |   |   |-- MANIFESTO_MODULO          <-- versione, descrizione, dipendenze
|   |   |   |-- interfaccia-pubblica      <-- unico punto di accesso dall'esterno
|   |   |   |-- nucleo/
|   |   |   |   |-- dominio/
|   |   |   |   |   |-- entita/           <-- esempio completo con invarianti
|   |   |   |   |   |-- oggetti-valore/   <-- esempio completo immutabile
|   |   |   |   |   |-- eventi/           <-- esempio con Correlation/Causation ID
|   |   |   |   |   +-- servizi-dominio/
|   |   |   |   +-- porte/
|   |   |   |       |-- ingresso/         <-- esempio caso d'uso comando + query
|   |   |   |       +-- uscita/           <-- esempio repository + gateway
|   |   |   |-- adattatori/
|   |   |   |   |-- ingresso/             <-- esempio adattatore web
|   |   |   |   +-- uscita/               <-- esempio adattatore persistenza
|   |   |   +-- test/
|   |   |       |-- unitari/              <-- esempio TDD completo Red-Green-Refactor
|   |   |       |-- integrazione/         <-- esempio BDD Given-When-Then
|   |   |       |-- e2e/                  <-- esempio test modulo isolato
|   |   |       +-- specifiche/           <-- esempio specifica linguaggio naturale
|   |   |
|   |   |-- fatturazione/                <-- bounded context reale
|   |   |   |-- MANIFESTO_MODULO
|   |   |   |-- interfaccia-pubblica
|   |   |   |-- nucleo/
|   |   |   |   |-- dominio/
|   |   |   |   |   |-- entita/
|   |   |   |   |   |-- oggetti-valore/
|   |   |   |   |   |-- eventi/
|   |   |   |   |   +-- servizi-dominio/
|   |   |   |   +-- porte/
|   |   |   |       |-- ingresso/
|   |   |   |       +-- uscita/
|   |   |   |-- adattatori/
|   |   |   |   |-- ingresso/
|   |   |   |   +-- uscita/
|   |   |   +-- test/
|   |   |       |-- unitari/
|   |   |       |-- integrazione/
|   |   |       |-- e2e/
|   |   |       +-- specifiche/
|   |   |
|   |   |-- prenotazioni/                <-- stessa struttura
|   |   |-- anagrafica/
|   |   |-- sala-operatoria/
|   |   +-- cruscotto-direzionale/
|   |
|   +-- condiviso/
|       |-- oggetti-valore/               <-- identificativi, moneta, date
|       |-- eventi-base/                  <-- struttura base con Correlation/Causation ID
|       +-- eccezioni-base/
|
|-- infrastruttura/
|   |-- ambienti/
|   |   |-- sviluppo/
|   |   |-- staging/
|   |   +-- produzione/
|   |-- pipeline/
|   |-- script/
|   +-- test-infrastruttura/
|
|-- documentazione/
|   |-- manifesto.md                      <-- questo documento
|   |-- decisioni-architetturali/         <-- ADR
|   |-- guide/
|   +-- specifiche-condivise/
|
+-- strumenti/
    |-- generatori/                       <-- scaffolding CLI per nuovi moduli
    |-- configurazione-analisi/           <-- regole linting, confini moduli, scope validi
    +-- script-sviluppo/
```

Il modulo di riferimento (prefissato con underscore per apparire in cima all'elenco) e' un modulo completo e funzionante che implementa un dominio semplice -- ad esempio una gestione anagrafica di base -- seguendo alla lettera tutti i principi del manifesto. Non e' un modulo di produzione: e' un modello vivente. Contiene esempi commentati di ogni pattern prescritto: un'entita' con costruttore privato e factory method, un value object immutabile, un evento con i tre identificativi (evento, correlazione, causazione), un caso d'uso comando e uno query, un adattatore di persistenza e uno di ingresso, test TDD con il ciclo Red-Green-Refactor esplicito, test BDD con specifiche in linguaggio naturale, e un test end-to-end che dimostra l'isolamento del modulo. Quando uno sviluppatore deve creare un nuovo modulo, il modulo di riferimento e' il punto di partenza concreto.

La cartella strumenti/generatori contiene lo scaffolding CLI del team: uno script che, dato il nome di un nuovo modulo, genera automaticamente tutta la struttura delle cartelle, le interfacce base, le classi di configurazione e i test scheletro. Questo azzera l'ansia da "foglio bianco" per lo sviluppatore e garantisce aderenza alla struttura del manifesto fin dalla prima riga. Il generatore usa il modulo di riferimento come template, sostituendo i nomi e rimuovendo gli esempi commentati.

---

### 12.2 Regole di convivenza nella struttura

**Ogni modulo e' autocontenuto.** Tutti i file di un modulo vivono all'interno della sua cartella. Non esistono file trasversali che referenziano le strutture interne di piu' moduli.

**La cartella condivisa e' minimale.** Solo i concetti veramente trasversali finiscono nella cartella condivisa. La struttura base degli eventi (con Correlation ID e Causation ID) vive qui perche' e' un contratto che tutti i moduli rispettano.

**I test vivono accanto al codice che testano.** Se estrai il modulo come pacchetto indipendente, i test vengono con lui.

**L'infrastruttura vive separata dall'applicazione.**

---

## 13. Moduli esportabili: portabilita' cross-progetto

Un modulo ben costruito non e' legato al progetto in cui nasce: il suo nucleo e' universale, mentre i suoi adattatori sono specifici del contesto in cui opera.

---

### 13.1 Il principio di portabilita'

Immagina di aver costruito un modulo di fatturazione per una clinica privata. Ora arriva un secondo progetto per la Pubblica Amministrazione con regole completamente diverse. Se il modulo e' costruito correttamente con porte e adattatori, la risposta e': tutto il nucleo, nessun adattatore.

---

### 13.2 La separazione tra nucleo riusabile e adattatori specifici

Il nucleo del modulo (dominio + porte) diventa un pacchetto pubblicabile e distribuibile. Quando un nuovo progetto lo adotta, guarda le porte di uscita e implementa un adattatore per ciascuna nel proprio contesto.

Esempio concreto -- clinica privata. Adattatore di persistenza per il proprio database, adattatore di invio fatture al Sistema di Interscambio formato privati, generatore di numeri progressivi annuali, adattatore di validazione per le regole delle prestazioni sanitarie esenti.

Esempio concreto -- Pubblica Amministrazione. Stesso nucleo con: adattatore di persistenza per un database diverso, adattatore di invio nel formato PA, generatore di numeri integrato con il protocollo generale dell'ente, adattatore di validazione codici di gara tramite l'ente regolatore, adattatore aggiuntivo per il flusso di approvazione gerarchica.

---

### 13.3 Estendere il dominio per un contesto specifico

Il nucleo definisce il modello base che funziona ovunque. Ogni progetto puo' estendere le entita' aggiungendo concetti specifici del proprio contesto, senza modificare il nucleo. Le estensioni vivono nel progetto, non nel pacchetto condiviso. Se un'estensione si rivela utile in piu' progetti, viene promossa nel nucleo dopo averla validata in almeno due contesti reali.

---

### 13.4 Il test di portabilita'

Ogni modulo deve poter essere avviato e testato in completo isolamento, senza nessun altro modulo del sistema presente, usando adattatori di test minimali. Se il test fallisce, il modulo ha dipendenze nascoste.

---

## 14. Anti-pattern e trappole comuni

---

### 14.1 Il "falso confine" tra moduli

**Sintomo.** Due moduli si chiamano direttamente, condividono query incrociate, o hanno dipendenze circolari. **Causa.** Confini tracciati per criteri tecnici o organizzativi, non di dominio. **Soluzione.** Rivedere i confini con l'analisi del linguaggio ubiquo del DDD.

---

### 14.2 Il "dio modulo" -- condiviso che diventa un modulo di dominio

**Sintomo.** La cartella condivisa cresce e contiene servizi di business. **Causa.** Il team la usa come posto comodo. **Soluzione.** La cartella condivisa contiene esclusivamente: tipi primitivi del dominio, interfaccia base degli eventi (con i tre identificativi), eccezioni base.

---

### 14.3 L'evento come chiamata remota mascherata

**Sintomo.** Un modulo pubblica un evento e attende sincronamente la risposta. Oppure l'evento contiene un campo "azione richiesta". **Causa.** Confusione tra eventi e comandi. **Soluzione.** Se serve una risposta sincrona, usare l'interfaccia pubblica del modulo. Se si vuole comunicare un fatto, pubblicare un evento. L'anti-pattern dell'evento-che-richiede-risposta crea un accoppiamento temporale nascosto.

---

### 14.4 L'astrazione prematura delle porte

**Sintomo.** Venti porte di uscita con un solo adattatore ciascuna. **Causa.** Applicazione meccanica del principio. **Soluzione.** "Posso descrivere almeno due implementazioni plausibili?" Se no, l'interfaccia e' prematura.

---

### 14.5 La separazione comandi/query applicata al dominio

**Sintomo.** Entita' di dominio separate per lettura e scrittura. **Causa.** Confusione tra pattern applicativo e modello di dominio. **Soluzione.** Il dominio ha un unico modello. La separazione si applica alle porte di ingresso. Le proiezioni di lettura ottimizzate vivono negli adattatori.

---

### 14.6 L'Infrastructure as Code senza test

**Sintomo.** Definizioni codificate ma non testate. **Causa.** IaC trattato come "configurazione". **Soluzione.** Stessa piramide di test del codice applicativo.

---

### 14.7 I feature flag che non muoiono mai

**Sintomo.** Condizionali ovunque, nessuno ricorda perche'. **Causa.** Ciclo di vita non gestito. **Soluzione.** Ogni flag nasce con data di scadenza. Registro revisionato ad ogni retrospettiva.

---

### 14.8 Il versionamento semantico senza disciplina

**Sintomo.** Breaking change rilasciati come patch, versioni moduli non aggiornate. **Causa.** Commit senza convenzione, scope non dichiarati. **Soluzione.** Pipeline rifiuta commit con scope non validi. Suggerimento automatico segnala incoerenze.

---

## 15. Rischi operativi e sfide pratiche

Per quanto teoricamente solida, l'applicazione rigorosa di questo manifesto si scontrera' con la realta' operativa. Questa sezione documenta i rischi principali che il team incontrera' e le strategie strutturali per ridurli a un livello gestibile fin dal primo giorno. Un framework che non anticipa le proprie debolezze e' un documento teorico; un framework che le anticipa e fornisce contromisure e' pronto per la produzione.

---

### 15.1 Barriera all'ingresso e carico cognitivo

**Il rischio.** Chiedere a uno sviluppatore -- soprattutto se junior o appena assunto -- di padroneggiare simultaneamente TDD, BDD, Bounded Context, architettura esagonale, separazione comandi/query, Conventional Commits con scope, tracciamento eventi con Correlation ID, e tutte le regole operative di questo manifesto e' una richiesta enorme. Il rischio concreto e' una paralisi da analisi nei primi mesi: lo sviluppatore passa piu' tempo a chiedersi "sto rispettando il manifesto?" che a scrivere codice utile. La velocity del team cala drasticamente con ogni nuovo membro, la frustrazione cresce, e il manifesto stesso diventa percepito come un ostacolo piuttosto che un aiuto.

**Le contromisure strutturali.**

Il **modulo di riferimento** (descritto nella sezione 12.1) e' la prima contromisura. Il codice parla piu' dei documenti. Uno sviluppatore che deve creare un'entita' di dominio non deve rileggere le sezioni 5 e 6 del manifesto: apre il modulo di riferimento, trova l'entita' di esempio, ne studia la struttura, e la usa come modello. Il modulo di riferimento non e' documentazione: e' codice funzionante e testato che implementa ogni pattern del manifesto con commenti esplicativi dove serve. E' il "dillo col codice" applicato all'architettura.

Lo **scaffolding automatico** (descritto nella sezione 12.1) e' la seconda contromisura. Quando lo sviluppatore deve creare un nuovo modulo, non parte da una cartella vuota: esegue lo script generatore che produce tutta la struttura delle cartelle, le interfacce base, la configurazione, e gli scheletri dei test. Lo sviluppatore deve riempire gli spazi, non creare la struttura. Questo azzera l'ansia da foglio bianco e riduce le decisioni architetturali che il nuovo membro deve prendere nei primi giorni a zero.

Il **pair programming obbligatorio nelle prime quattro settimane** e' la terza contromisura, gia' prevista dalla pratica XP (sezione 3.1). Ma qui il manifesto e' prescrittivo: non "consigliato", non "quando possibile". Obbligatorio. Un nuovo membro del team non lavora mai da solo per le prime quattro settimane di calendario. Lavora sempre in coppia con un membro senior, alternando il ruolo di chi scrive e chi osserva. La teoria del manifesto si impara guardandola applicare a problemi reali, non leggendo il documento. Al termine delle quattro settimane, il nuovo membro ha implementato almeno tre storie complete end-to-end -- dalla specifica al codice -- e ha ricevuto feedback architetturale su ciascuna.

La **progressione graduale dei concetti** e' la quarta contromisura. Non tutti i concetti del manifesto hanno la stessa priorita' di apprendimento. L'onboarding del nuovo membro segue un ordine preciso: nella prima settimana si concentra sui confini dei moduli (P1) e sulla struttura delle cartelle; nella seconda settimana aggiunge la purezza del dominio (P2) e il TDD; nella terza settimana introduce gli eventi (P3) e il BDD; nella quarta settimana affronta i concetti avanzati (CQRS, versionamento, feature flag). Questa progressione evita il sovraccarico cognitivo del "tutto insieme dal giorno uno".

Esempio concreto. Arriva nel team una sviluppatrice con tre anni di esperienza ma nessuna familiarita' con DDD o architettura esagonale. Il primo giorno legge il manifesto. Il secondo giorno apre il modulo di riferimento e studia la struttura. Il terzo giorno, in pair con un membro senior, usa lo scaffolding per generare un nuovo modulo e implementa un caso d'uso semplice: un comando che crea un'entita' e pubblica un evento. Alla fine della prima settimana ha una comprensione pratica dei confini dei moduli e della struttura delle cartelle, perche' li ha usati, non solo letti. Alla fine del mese ha implementato tre storie complete e ha ricevuto feedback architetturale ad ogni revisione del codice. Il manifesto non e' piu' un documento intimidatorio: e' la struttura dentro cui lavora ogni giorno.

---

### 15.2 Il costo del dominio puro: eccesso di boilerplate

**Il rischio.** Mantenere il nucleo del dominio agnostico rispetto all'infrastruttura significa scrivere molti livelli di traduzione: le strutture di persistenza vengono tradotte in entita' di dominio, le entita' di dominio vengono tradotte in modelli di trasferimento, e viceversa. In un modulo con decine di entita', questo boilerplate di mappatura diventa una parte significativa del codice -- e una fonte significativa di frustrazione, specialmente per le operazioni banali di tipo CRUD (creazione, lettura, aggiornamento, eliminazione) dove non c'e' logica di business significativa.

**Le contromisure strutturali.**

Il **CQRS asimmetrico** (gia' descritto nella sezione P4) e' la contromisura principale. La separazione comandi/query consente di applicare la purezza del dominio solo dove serve -- sulle scritture. Le operazioni di sola lettura possono bypassare il dominio: l'adattatore di lettura interroga direttamente il livello di persistenza e restituisce il modello di trasferimento senza passare per la ricostruzione dell'entita' di dominio. Questo dimezza il boilerplate di mappatura per le operazioni di lettura, che in molti sistemi rappresentano il settanta-ottanta percento delle operazioni totali.

La **deroga esplicita per moduli di supporto senza logica di dominio** e' la seconda contromisura. Non tutto il codice del sistema ha logica di business. Alcuni moduli sono puramente dizionari di supporto: liste di nazioni, categorie fiscali, tabelle di lookup, configurazioni di sistema. Per questi moduli, applicare la purezza del dominio, le porte e gli adattatori separati, e il TDD con copertura al novanta percento e' sovra-ingegnerizzazione. Il manifesto ammette esplicitamente che un modulo di supporto senza logica di dominio puo' adottare un pattern piu' semplice (ad esempio accesso diretto ai dati senza livello di dominio), purche' questa deroga sia documentata in un Architecture Decision Record che spiega perche' il modulo non ha bisogno della struttura completa. La regola e': se il modulo non ha invarianti di business da proteggere e non pubblica eventi di dominio, puo' usare un pattern semplificato. Se ha anche un solo invariante, usa la struttura completa.

Esempio concreto. Il modulo "categorie fiscali" contiene una lista di codici IVA con le relative aliquote. Non ha logica di business: non ci sono invarianti da proteggere (un codice IVA e' un dato di riferimento che cambia per decreto, non per logica del sistema), non pubblica eventi, non ha transizioni di stato. Applicare l'architettura esagonale completa a questo modulo significherebbe creare un'entita' di dominio "CategoriaFiscale", un value object "AliquotaIva", un repository con interfaccia e implementazione, un caso d'uso "OttieniCategorie", un modello di trasferimento, e un mapper tra le tre rappresentazioni -- per un dato che e' essenzialmente una tabella di lookup. Il team approva un ADR che consente per questo modulo un accesso diretto ai dati con un pattern semplificato. Il modulo fatturazione (che ha logica di business complessa) usa la struttura completa.

---

### 15.3 La trappola degli "spaghetti events": perdita di tracciabilita'

**Il rischio.** La comunicazione tramite eventi disaccoppia i moduli in modo efficace, ma oscura il flusso di esecuzione. In un sistema con chiamate dirette, uno sviluppatore puo' seguire il flusso da un modulo all'altro navigando il codice nell'editor: "questo metodo chiama quel metodo che chiama quell'altro". In un sistema a eventi, il flusso e' invisibile nell'editor: l'evento viene pubblicato in un punto e ascoltato in un altro, senza nessun collegamento navigabile. Quando il numero di eventi cresce, capire "cosa succede nel sistema quando un utente esegue questa azione" diventa un esercizio di archeologia.

**Le contromisure strutturali.**

Il **Correlation ID e Causation ID obbligatori** (gia' prescritti nella sezione 6.3) sono la contromisura fondamentale. Non sono un'aggiunta opzionale: fanno parte della struttura base di ogni evento del sistema. Ogni flusso di operazioni scatenato da un'azione utente e' ricostruibile al cento percento seguendo il Correlation ID. Ogni relazione padre-figlio tra eventi e' visibile tramite il Causation ID. Senza questi identificativi, il sistema a eventi e' un labirinto; con questi identificativi, e' una mappa.

Il **tracciamento distribuito interno al monolite** e' la seconda contromisura. Anche se il sistema e' un monolite, ogni transazione che attraversa i confini di un modulo dovrebbe generare una traccia di esecuzione strutturata. Questo si implementa tramite uno strumento di tracciamento che intercetta la pubblicazione e la ricezione di eventi, registra i tempi, e costruisce una visualizzazione della cascata di operazioni. Lo strumento non richiede infrastruttura distribuita: nel monolite puo' essere semplice come un log strutturato che viene aggregato e visualizzato.

Il **catalogo degli eventi auto-generato** e' la terza contromisura. Uno strumento di analisi statica del codice puo' identificare automaticamente tutti i punti dove gli eventi vengono pubblicati e tutti i punti dove vengono ascoltati, e generare una mappa delle relazioni tra pubblicanti e sottoscrittori. Questa mappa viene generata ad ogni integrazione e resa disponibile come documentazione vivente. Lo sviluppatore che vuole capire "chi ascolta l'evento X" non deve cercare nel codice: consulta il catalogo auto-generato.

Esempio concreto. Il team di supporto riceve una segnalazione: "il paziente Mario Rossi ha ricevuto due email di conferma per la stessa prenotazione". Lo sviluppatore apre lo strumento di tracciamento, cerca il Correlation ID della richiesta originale, e vede l'intera cascata: la richiesta web ha generato un evento "prenotazione confermata"; il modulo notifiche ha ricevuto l'evento e inviato l'email; ma il modulo notifiche ha ricevuto lo stesso evento due volte (perche' un retry automatico lo ha riconsegnato). Il gestore delle notifiche non era idempotente: non controllava l'identificativo dell'evento prima di procedere. Il bug e' identificato in tre minuti grazie al tracciamento strutturato. Senza il Correlation ID, lo sviluppatore avrebbe dovuto correlare manualmente i log di tre moduli diversi, un processo che richiede tipicamente un'ora o piu'.

---

### 15.4 L'ossessione per la copertura: metriche che mentono

**Il rischio.** Fissare target rigidi di copertura dei test (novanta percento sul dominio, ottanta percento sull'applicazione) crea un incentivo perverso: sotto pressione di consegna, gli sviluppatori scrivono test che eseguono il codice senza verificare nulla di significativo. Il test chiama la funzione, non esplode, la riga e' contata come "coperta". La pipeline e' verde, la copertura e' al novantacinque percento, il team e' rassicurato -- ma il sistema e' fragile perche' i test non stanno proteggendo nulla. Una modifica al codice che introduce un difetto non fa fallire nessun test, perche' nessun test stava verificando quel comportamento.

**Le contromisure strutturali.**

Il **test di mutazione** (gia' descritto nella sezione 10.2) e' la contromisura definitiva. Lo strumento di mutazione modifica il codice in modo controllato e rilancia i test. Se i test rimangono verdi dopo che una condizione e' stata invertita o un'operazione e' stata rimossa, quei test non stanno facendo il loro lavoro. Il mutation score e' l'antidoto ai test superficiali: non misura se il codice e' stato eseguito, misura se i test avrebbero fallito se il codice fosse stato sbagliato. Il manifesto richiede un mutation score di almeno l'ottanta percento sul nucleo del dominio (sezione 10.2).

L'**esclusione intelligente dalla copertura** e' la seconda contromisura. Gli strumenti di misurazione devono essere configurati per escludere automaticamente dalla copertura i file che non contengono logica testabile: modelli di trasferimento (che sono pure strutture dati senza comportamento), file di configurazione dichiarativi, costanti, e boilerplate di framework. Pretendere copertura su un file di costanti distorce le metriche e spinge gli sviluppatori a scrivere test inutili per raggiungere la soglia. Il target del novanta percento deve essere chirurgicamente puntato sul nucleo del dominio -- dove la logica di business vive e dove un difetto ha conseguenze reali.

La **revisione della qualita' dei test nella retrospettiva** e' la terza contromisura. Ogni quattro iterazioni, durante la retrospettiva architetturale (sezione 17.4), il team dedica tempo all'analisi dei risultati del test di mutazione: quanti mutanti sopravvivono, in quali aree del codice, e perche'. Se un'area del dominio ha alta copertura ma basso mutation score, il team sa dove i test sono superficiali e puo' migliorarli prima che un difetto reale si presenti.

Esempio concreto. Il team ha una copertura del novantacinque percento sul modulo fatturazione. Il Tech Lead esegue il test di mutazione e scopre che il mutation score e' solo del sessantadue percento. L'analisi mostra che i mutanti sopravvissuti si concentrano nella logica di calcolo degli arrotondamenti fiscali: i test verificano che il calcolo non esploda, ma non verificano che il risultato sia corretto. Lo sviluppatore aggiunge asserzioni specifiche sui valori attesi (ad esempio: "una fattura con imponibile 100.50 e aliquota 22% deve avere IVA pari a 22.11, non 22.10 e non 22.12"). Il mutation score sale all'ottantasei percento. Un mese dopo, una modifica accidentale all'algoritmo di arrotondamento viene immediatamente catturata dai test che ora verificano i valori precisi.

---

## 16. Il percorso di crescita

Un Modular Monolith ben strutturato non e' una scelta definitiva: e' una scelta reversibile.

**Fase 1 -- Adesso.** Modular Monolith con i cinque principi e architettura esagonale. Deploy unico. Il meccanismo di eventi e' in-process. La separazione comandi/query e' logica. L'infrastruttura e' definita come codice. Il team pratica XP. Trigger per la fase successiva: il volume di eventi supera la capacita' del meccanismo in-process.

**Fase 2 -- Crescita.** Il meccanismo di eventi passa a un sistema di messaggistica dedicato. Il nucleo dei moduli non cambia: cambia solo l'adattatore della porta "pubblicazione eventi". Trigger per la fase successiva: le query piu' pesanti rallentano sotto carico.

**Fase 3 -- Scaling.** I casi d'uso di sola lettura con alti volumi vengono serviti da una replica di lettura dedicata o da proiezioni materializzate. La separazione comandi/query diventa fisica per i moduli che lo richiedono. Trigger per la fase successiva: un modulo ha bisogno di un ciclo di sviluppo indipendente o un Service Level Agreement diverso.

**Fase 4 -- Estrazione.** Un modulo viene estratto come servizio autonomo. Il nucleo rimane invariato. Gli adattatori si collegano via rete. Il test di portabilita' garantisce che l'estrazione funzionera'. Trigger per la fase successiva: lo stesso nucleo e' richiesto da un progetto completamente diverso.

**Fase 5 -- Riuso.** Il nucleo del modulo viene pubblicato come pacchetto indipendente e adottato da un altro progetto con adattatori completamente diversi. Il nucleo e' gia' versionato semanticamente e ha una storia di versioni chiara.

Il segnale piu' importante per passare alla fase successiva non e' tecnico: e' organizzativo. Si passa a una fase piu' complessa quando la complessita' organizzativa supera il costo operativo aggiuntivo della nuova fase. Non prima.

---

## 17. Il contratto del team

Questo manifesto e' efficace solo se il team lo tratta come un contratto vivo.

---

### 17.1 Revisione del codice

Ogni proposta di modifica che viola un principio deve essere bloccata, indipendentemente dall'urgenza. Il revisore cita il principio violato e propone la soluzione corretta.

La checklist di revisione include: gli accessi tra moduli passano solo dall'interfaccia pubblica; la logica di business e' nel nucleo del dominio; gli eventi sono strutture tipizzate con i tre identificativi (evento, correlazione, causazione); i gestori di eventi sono idempotenti; i test seguono la piramide; il modulo passa il test di portabilita'; i feature flag hanno una data di scadenza; le modifiche infrastrutturali sono codificate e testate; i commit seguono la convenzione con scope dichiarato; la complessita' cognitiva e' sotto soglia; il mutation score sul dominio e' sopra l'ottanta percento.

Le eccezioni richiedono un Architecture Decision Record documentato e approvato.

---

### 17.2 Onboarding

Ogni nuovo sviluppatore legge questo documento il primo giorno. Il secondo giorno apre il modulo di riferimento e ne studia la struttura. Il terzo giorno, in pair programming con un membro senior, usa lo scaffolding automatico per generare un nuovo modulo e implementa il primo caso d'uso. Le prime quattro settimane sono esclusivamente in pair programming con un membro senior.

La progressione dei concetti segue un ordine preciso: prima settimana confini dei moduli e struttura delle cartelle (P1); seconda settimana purezza del dominio e TDD (P2); terza settimana eventi con Correlation ID e BDD (P3); quarta settimana concetti avanzati (CQRS, versionamento, feature flag). Al termine delle quattro settimane, il nuovo membro ha implementato almeno tre storie complete end-to-end con revisione architetturale dedicata su ciascuna.

---

### 17.3 Definition of Done

Una storia non e' "done" finche' non soddisfa: test unitari TDD verdi con copertura al novanta percento sul nucleo del dominio, mutation score al ottanta percento sul nucleo del dominio, test di integrazione BDD verdi con copertura all'ottanta percento sul livello applicativo, test end-to-end corrispondenti a ogni scenario della specifica, pipeline verde (inclusa analisi complessita' cognitiva sotto soglia), revisione del codice approvata con checklist del manifesto verificata, commit con Conventional Commits e scope dichiarato, eventi con Correlation ID e Causation ID, e feature flag configurato se la funzionalita' non e' pronta per il rilascio.

---

### 17.4 Retrospettiva architetturale

Ogni quattro iterazioni il team dedica una sessione alla revisione dell'architettura. Si analizzano: le violazioni intercettate nelle revisioni, i punti di attrito ricorrenti, le metriche di complessita' cognitiva in crescita, i risultati del test di mutazione (aree con alto coverage ma basso mutation score), lo stato del registro dei feature flag, la coerenza tra versioni semantiche e cambiamenti effettivi, e l'efficacia del modulo di riferimento e dello scaffolding per i nuovi membri.

---

### 17.5 Evoluzione del manifesto

Il manifesto puo' essere modificato. Qualsiasi proposta viene discussa in team durante la retrospettiva architetturale e documentata con motivazione. Nessun principio viene aggiunto senza essere stato testato nella pratica per almeno due iterazioni, e nessun principio viene rimosso senza un ADR.

---

### 18 Boost allo sviluppo con agenti LLM

Il manifesto fornisce un "boost" naturale agli agenti LLM per tre ragioni principali:

- **Contesto Delimitato (Bounded Context):** Poiché ogni modulo ha confini espliciti e un'interfaccia pubblica definita (P1), è possibile fornire a un LLM il contesto di un singolo modulo senza "confonderlo" con l'intero sistema. Questo riduce drasticamente il rumore e i limiti di token.
- **Purezza del Dominio:** La regola delle dipendenze unidirezionale (P2) e la purezza del nucleo del dominio (6.2) garantiscono che la logica di business sia espressa in linguaggio puro (Plain Old Objects), privo di boilerplate di framework. Questo permette agli LLM di generare e ragionare sulla logica di business in modo molto più accurato.
- **Spec-Driven Development:** L'uso di specifiche in linguaggio naturale (BDD) come driver per i test (7.2, 7.4) crea un ponte perfetto tra l'input umano e il codice. Un LLM può facilmente trasformare una specifica "Dato-Quando-Allora" in uno scheletro di test e nella relativa implementazione.

---

### 19. AI-Augmented Development: Agenti e Architettura

Questa sezione definisce come il team integra gli agenti LLM nel flusso di lavoro per accelerare la consegna senza compromettere la qualità o l'integrità del manifesto.

------

### 19.1 L'Agente come "Junior Partner" nel Pair Programming

L'uso di agenti LLM non sostituisce il Pair Programming umano, ma lo integra. L'agente viene trattato come un partner che propone implementazioni basate sulle specifiche (sezione 7). L'umano agisce come supervisore e garante dei principi architettonici (P1-P5).

- **Generazione dei Test:** L'agente genera i test unitari (TDD) e di integrazione (BDD) partendo dalle specifiche scritte dal Tech Lead e dal Product Owner.
- **Rilevamento violazioni:** L'agente viene istruito con il manifesto per segnalare proattivamente tentativi di accoppiamento tra moduli o perdite di astrazione (es. logica di infrastruttura nel dominio).

------

### 19.2 Prompt Engineering basato sul Dominio

Per massimizzare l'efficacia degli agenti, i prompt devono riflettere la struttura del Modular Monolith:

- **Context Injection per Modulo:** Quando si lavora su un modulo, l'agente deve ricevere solo l'interfaccia pubblica (P1) e il nucleo del dominio (6.2) del modulo interessato.
- **Uso del linguaggio Ubiquo:** I prompt devono utilizzare esclusivamente i termini definiti nel linguaggio del dominio per garantire che il codice generato sia coerente con il modello mentale del team.

------

### 19.3 Automazione del Refactoring e Debito Tecnico

Gli agenti LLM vengono utilizzati per eseguire refactoring meccanici guidati dalla suite di test esistente.

- **Modernizzazione e Pulizia:** L'AI può essere incaricata di estrarre Value Object o di separare Query e Comandi (P4) in moduli legacy o meno rifiniti, garantendo la sicurezza tramite l'esecuzione automatica della pipeline.
- **Rimozione Feature Flag:** Gli agenti possono automatizzare la rimozione dei branch condizionali legati a feature flag obsoleti (8.3) una volta che la funzionalità è diventata stabile.

------

### 19.4 Integrità Architetturale e Allucinazioni

Il team accetta proposte dall'AI solo se verificate dalla pipeline (10.2). Se l'agente propone una soluzione che viola la regola delle dipendenze (P2) o che accede direttamente ai dati di un altro modulo (6.4), l'umano ha il dovere di scartare la proposta e istruire l'agente sul motivo del rifiuto, rafforzando il contratto operativo del team.

------

### 19.5 Configurazione dell'IDE Agentico (Windsurf)

Per garantire che l'automazione non degradi la qualità architettonica, l'IDE deve essere configurato con regole che agiscano da "vincolo fisico" per l'agente LLM. Questa configurazione trasforma il manifesto in un set di istruzioni operative non negoziabili.

------

### 19.5.1 Definizione delle regole locali (.windsurfrules)

Nella radice del progetto deve essere presente un file .windsurfrules per istruire l'agente sul comportamento atteso e sui limiti invalicabili.

------

### 19.5.2 Modular Monolith Architecture Manifesto - AI Rules

Tu agisci come un esperto Tech Lead. Ogni tua azione deve rispettare il Manifesto v5.1.

#### 1. Regole Architetturali Inviolabili

- **Confini (P1):** È vietato l'accesso diretto ai file interni di un modulo. Usa solo 'interfaccia-pubblica'.
- **Purezza (P2):** Il codice in 'nucleo/dominio' deve essere privo di annotazioni di framework o librerie esterne.
- **Eventi (P3):** Ogni evento deve includere evento_id, correlazione_id e causazione_id.
- **CQRS (P4):** Separa fisicamente i Comandi dalle Query nelle porte di ingresso.

#### 2. Standard di Progetto

- **Struttura:** Segui fedelmente la gerarchia definita nella sezione 12.1.
- **Esempio:** Prima di generare codice, analizza il modulo '_riferimento' come gold standard.
- **Qualità:** Rifiuta o rifattorizza funzioni con Complessità Cognitiva > 15.

#### 3. Workflow Operativo

1. Analizza la specifica BDD in 'test/specifiche'.
2. Applica il ciclo TDD: scrivi il test unitario nel dominio prima dell'implementazione.
3. Assicurati che ogni modifica sia coperta da test e che il mutation score del dominio sia >= 80%.
19.5.2 Master System Prompt per lo Sviluppo Assistito
Questo prompt deve essere utilizzato per inizializzare l'agente LLM all'inizio di ogni sessione o task complesso.

#### 4. System Prompt:

Sei l'Agente AI "Junior Partner" integrato nel flusso Extreme Programming del team. Il tuo obiettivo è la consegna di valore attraverso codice testato e modulare.

Prima di ogni generazione:

1. Identifica il Bounded Context del modulo attuale.
2. Verifica che la logica di business sia protetta nel nucleo del dominio.
3. Implementa l'idempotenza nei gestori eventi verificando l'identificativo univoco.
4. Assicurati che il commit segua il formato Conventional Commits con lo scope del modulo corretto.
5. Non generare codice infrastrutturale (DB, API) finché la logica di dominio non è validata dai test unitari.

------

### 19.5.3 Esempio di Input Operativo per l'Agente

Per attivare correttamente l'agente su un nuovo requisito, utilizzare il seguente formato di input strutturato:

**Input Task**:

"Agente, implementa il requisito: 'Un intervento chirurgico non può essere avviato se la sala non è in stato Prenotata'.

Scaffolding: Se necessario, usa lo strumento di generazione per il modulo 'sala-operatoria'.

Specifica: Scrivi lo scenario BDD in linguaggio naturale in 'test/specifiche/avvio_intervento.feature'.

Dominio: Implementa l'invariante nell'entità di dominio 'Sala' usando TDD.

Tracciabilità: Assicurati che l'evento 'InterventoAvviato' propaghi correttamente il Correlation ID della richiesta.

Verifica: Esegui i test unitari e conferma che la complessità cognitiva sia sotto la soglia di 15.

Procedi in modalità Pair Programming: fermati dopo la scrittura dei test e attendi il mio feedback prima di procedere con l'implementazione.".

---

### 20 Bibliografia

#### Opere Principali

- **Evans, E.** (2003). *Domain-Driven Design: Tackling Complexity in the Heart of Software*. Addison-Wesley. https://www.ibs.it/domain-driven-design-tackling-complexity-libro-inglese-eric-evans/e/9780321125217
- **Vernon, V.** (2013). *Implementing Domain-Driven Design*. Addison-Wesley. https://www.lafeltrinelli.it/implementing-domain-driven-design-libro-inglese-vaughn-vernon/e/9780321834577
- **Cockburn, A.** (2005). *Hexagonal Architecture (Ports and Adapters)*. Articolo/pattern originale. https://jmgarridopaz.github.io/content/interviewalistair.html
- **Martin, R. C.** (2017). *Clean Architecture: A Craftsman's Guide to Software Structure and Design*. Prentice Hall. https://www.lafeltrinelli.it/clean-architecture-craftsman-s-guide-libro-inglese-robert-martin/e/9780134494166
- **Beck, K.** (2004). *Extreme Programming Explained: Embrace Change* (2ª ed.). Addison-Wesley. https://hamersoft.com/2024/08/31/2711/

#### Metodologie di Sviluppo

- **Hammant, P.** (2015+). *Trunk-Based Development*. Paul Hammant blog. https://hammant1.rssing.com/chan-8523407/all_p2.html
- **Humble, J., & Farley, D.** (2010). *Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation*. Addison-Wesley. https://www.youtube.com/watch?v=ikrNZxKOrJQ
- **Morris, K.** (2025). *Infrastructure as Code* (3ª ed.). O'Reilly Media. https://helion.pl/ksiazki/infrastructure-as-code-3rd-edition-kief-morris,e_4d9x.htm
- **Adzic, G.** (2011). *Specification by Example: How Successful Teams Deliver the Right Software*. Manning. https://books.google.it/books?id=fDszEAAAQBAJ
- **Preston-Werner, T.** (2011). *Semantic Versioning Specification*. https://livefront.com/writing/lies-damn-lies-and-semantic-versioning/

- **Chen, M. et al.** (2024). *From LLMs to LLM-based Agents for Software Engineering*. arXiv preprint arXiv:2408.02479. https://arxiv.org/html/2408.02479v2
  Agenti LLM per code generation, testing e workflow SE.
- **Graphite Team.** (2024). *Best practices for pair programming with AI assistants*. Graphite Guides. https://graphite.com/guides/ai-pair-programming-best-practices
  Ruoli umano-AI nel pair programming e iterazioni.
- **Andovar Blog.** (2023). *The Role of Domain Knowledge in Effective Prompt Engineering*. https://blog.andovar.com/the-role-of-domain-knowledge-in-effective-prompt-engineering
  Prompt engineering con conoscenza di dominio (DDD-like).
- **Alshahani, A. et al.** (2025). *Code Refactoring with LLM: A Comprehensive Evaluation*. arXiv preprint arXiv:2511.21788. https://arxiv.org/abs/2511.21788
  LLM per refactoring multi-linguaggio e metriche qualità.
- **Understanding Data.** (2026). *DDD Bounded Contexts: Clear Domain Boundaries for LLM Code Generation*. https://understandingdata.com/posts/ddd-bounded-contexts-for-llms/
  Bounded Contexts DDD per migliorare accuratezza LLM.
- **Lassala, M.** (2025). *From Story to Solution: AI-Enhanced BDD/TDD Workflow*. YouTube/Improving. https://www.youtube.com/watch?v=z2DD9PLb7-E
  AI per BDD/TDD, da user story a codice test-driven.
- **Qian, C. et al.** (2024). *Large Language Model-Based Agents for Software Engineering*. arXiv preprint arXiv:2409.02977. https://arxiv.org/html/2409.02977v1
  Agenti LLM per end-to-end SE, inclusi testing e debugging.
- **AugmentedCode.** (2025). *Augment-Aider: AI pair programming tool*. GitHub Repository. https://github.com/augmentedcode/augment-aider
  Tool per AI pair programming con context awareness.
- **Reddit r/DomainDrivenDesign.** (2025). *AI Prompts within DDD Domain*. Community Discussion. https://www.reddit.com/r/DomainDrivenDesign/comments/1knv17r/ai_prompts_within_ddd_domain/
  Prompt LLM per modellazione DDD e domain objects.
- **Alshahani, A. et al.** (2025). *Code Refactoring with LLM Framework*. arXiv preprint arXiv:2511.21788 (v1). https://arxiv.org/html/2511.21788v1
  Framework LLM per refactoring con prompt engineering.
  

## Changelog

| Versione | Data | Note |
|---|---|---|
| v1.0 | 2025 | Prima stesura -- cinque principi, regole operative, percorso di crescita |
| v2.0 | 2025 | Aggiunta TDD/BDD/Spec-Driven Development, Trunk-Based con Feature Flags, moduli esportabili con porte e adattatori, test di portabilita' |
| v3.0 | 2025 | Riscrittura technology-agnostic. Aggiunta Extreme Programming. Aggiunta Infrastructure as Code. Aggiunta struttura di progetto. Aggiunta sezione anti-pattern. Corretti anti-pattern: relazione Clean/Hexagonal, applicazione CQRS, differenza eventi/comandi |
| v4.0 | 2026 | Aggiunta ruolo Tech Lead e relazione stakeholder. Aggiunta versionamento semantico a due livelli con Conventional Commits e scope. Aggiunta tagging a due livelli (codice sorgente e immagine deployment). Aggiunta analisi complessita' cognitiva con soglie e pipeline completa. Aggiunto suggerimento automatico versionamento |
| v5.0 | 2026 | Aggiunta sezione rischi operativi e sfide pratiche (barriera all'ingresso, boilerplate dominio puro, spaghetti events, metriche che mentono). Aggiunto Correlation ID e Causation ID come requisiti strutturali nella definizione degli eventi (sezione 6.3). Aggiunto test di mutazione nella pipeline e nelle metriche monitorate (sezioni 10.2, 10.3). Aggiunto modulo di riferimento e scaffolding CLI nella struttura di progetto (sezione 12.1). Aggiornato onboarding con progressione graduale dei concetti e pair obbligatorio quattro settimane (sezione 17.2). Aggiornata Definition of Done con mutation score e Correlation ID (sezione 17.3). Aggiunta deroga ADR per moduli di supporto senza logica di dominio. Aggiunto CQRS asimmetrico come strategia anti-boilerplate in P4 |
| v5.1 | 2026 |	Aggiunta sezione 19.5: Configurazione IDE Agentico (Windsurf), regole locali, system prompt e input operativo.

---

Modular Monolith Architecture Manifesto -- v5.0

Autore: Antonio Cittadino -- awesome.cit.dev@gmail.com
