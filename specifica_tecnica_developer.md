# MY.BBLO Profiler — Specifica tecnica: funzione serverless

Documento per un developer freelance. Descrive il comportamento richiesto, non impone una piattaforma specifica — la scelta tra Vercel, Netlify Functions o Cloudflare Workers è a discrezione dello sviluppatore, in base a costo e familiarità.

## Contesto

MY.BBLO Profiler è un quiz diagnostico B2B per centri estetici. Il centro riceve via email un briefing HTML (`mybblo_calcolatore.html`) con i dati del cliente e un calcolatore. La titolare del centro non ha un account Brevo personale: tutte le azioni che oggi richiederebbero un login a una dashboard devono avvenire tramite link cliccabili dentro le email che riceve già.

## Cosa deve fare la funzione

Tre azioni distinte, ciascuna richiamata da un link unico per contatto. La pagina di conferma (passaggio intermedio prima dell'azione vera) è già costruita: `mybblo_conferma_azione.html`. Gestisce tutte e tre le azioni in un unico file, distinte tramite un parametro nell'URL — non va costruita da zero, va solo collegata al backend.

**Come funziona oggi (frontend già pronto):** la pagina legge dall'URL i parametri `azione` (valori: `rifiuto`, `confermato`, `blocca`), `nome` (nome del cliente, per il messaggio) e `token` (l'identificativo univoco del contatto). Mostra il messaggio giusto per ciascuna azione, e al click su "Confermo" chiama la funzione JavaScript `conferma()`. Dentro quella funzione, nel file, c'è un commento `TODO developer` che indica esattamente dove va inserita la chiamata reale (oggi è un placeholder che simula sempre un esito positivo, da rimuovere). Non eseguire mai l'azione al primo caricamento della pagina (al solo GET), perché scanner antispam e client email talvolta visitano automaticamente i link nelle email — un'azione eseguita così rischierebbe falsi positivi; ecco perché il flusso richiede sempre il click esplicito su "Confermo" prima di chiamare il backend.

### Azione 1 — Marca rifiuto/rinvio (avvia la sequenza follow-up)

- `azione=rifiuto` nella pagina di conferma.
- Il token identifica il contatto (email cliente, inestetismo dichiarato, centro di riferimento).
- Alla conferma, chiama l'API di Brevo per aggiungere il contatto all'automazione "Follow-up tiepidi", nel ramo corrispondente all'inestetismo dichiarato (il token deve portare con sé questo dato, così la funzione sa in quale dei 4 rami inserirlo).

### Azione 2 — Check-up confermato (tracciamento positivo)

- `azione=confermato` nella pagina di conferma.
- Alla conferma, applica un tag (es. `convertito` o `checkup_confermato`) al contatto su Brevo, senza avviare nessuna automazione — è solo un dato da tracciare.

### Azione 3 — Blocca Email B (notifica del giorno 10)

- `azione=blocca` nella pagina di conferma.
- Alla conferma, deve **annullare l'invio della prossima email programmata (Email B, giorno 11)** per quello specifico contatto, senza interrompere il resto dell'automazione per altri contatti. Verificare la modalità corretta lato API Brevo per rimuovere/saltare un singolo step pianificato per un singolo contatto (può richiedere la rimozione del contatto da quello specifico step dell'automazione, o un tag che la condizione del workflow controlla — la soluzione esatta è a discrezione tecnica del developer, il comportamento richiesto è quello descritto). Questa azione è richiamata dal bottone dentro `mybblo_notifica_giorno10.html`, il template dell'email che Brevo invia il giorno 10 della sequenza.

## Sicurezza

- Ogni link deve contenere un token univoco e non indovinabile (es. un identificativo casuale generato per contatto, non un ID sequenziale).
- La chiave API di Brevo non deve mai essere esposta lato client — tutte le chiamate a Brevo passano dalla funzione serverless, mai dal codice HTML/JS delle pagine.
- Il token dovrebbe avere una scadenza ragionevole (es. 60 giorni) per evitare che link vecchi restino utilizzabili indefinitamente.

## Quarta funzione — Contatore licenze, blocco Email B per prova gratuita

Indipendente dalle tre azioni sopra, ma stessa infrastruttura: un contatore lato server (non `localStorage`) per il consumo delle licenze (P10, P50, P100, P200, ILLIMITATO), perché il link del prodotto viene condiviso anche via WhatsApp su dispositivi diversi, e un contatore solo-browser non sarebbe condiviso tra dispositivi. Comportamento richiesto: ad ogni completamento del quiz, incrementare un contatore associato alla licenza; bloccare l'accesso quando il contatore raggiunge il limite del piano, mostrando la pagina di blocco già prevista nei file di licenza esistenti.

**Blocco aggiuntivo specifico per la licenza P10_prova (prova gratuita):** quando la sequenza follow-up tiepidi è attiva per un contatto associato a una licenza P10_prova, l'Email B (giorno 11) non deve partire se la titolare non ha ancora acquistato una licenza a pagamento entro quel momento. L'Email A (giorno 5) può partire comunque — è soft e non contiene proposte commerciali. Questo blocco è semplice da implementare: è una condizione aggiuntiva nel trigger dell'Email B che verifica il tipo di licenza attiva per quel centro prima dell'invio.

## Quinta funzione — Link monouso per il quiz cliente

### Contesto e problema da risolvere

Il Profiler viene inviato dalla titolare alla propria cliente tramite un link. Quel link non deve essere riutilizzabile: se la cliente lo condivide con un'amica o lo apre più volte, non deve consumare quiz aggiuntivi dalla licenza della titolare né generare compilazioni non autorizzate. Il controllo di chi riceve il Profiler deve restare sempre e solo in mano alla titolare.

### Comportamento richiesto

Ogni link generato dalla titolare verso il quiz ha uno **stato univoco** gestito lato server. Gli stati possibili sono quattro:

- `generato` — il link è stato creato ma non ancora aperto
- `aperto` — la cliente ha aperto il quiz ma non ha ancora completato la compilazione con i suoi dati
- `completato` — la cliente ha completato il quiz e lasciato i dati (nome, telefono, email)
- `scaduto` — il link è stato disattivato per uno dei motivi sotto

**Regole di gestione:**

1. Un link aperto ma non completato (stato `aperto`) si disattiva automaticamente — passa a `scaduto` — ma **non consuma quiz dalla licenza**. La titolare ritrova il quiz disponibile nel suo contatore.

2. Il contatore della licenza scala **solo al completamento confermato** — ovvero quando la cliente invia il form con nome, telefono e email. Non al momento dell'apertura del link, non durante la compilazione.

3. Un link già in stato `completato` o `scaduto`, se aperto di nuovo, mostra una pagina dedicata (da costruire) con messaggio neutro tipo: *"Questo link non è più attivo. Se hai bisogno di assistenza, contatta il centro."* — senza esporre dettagli tecnici.

4. Un link non può essere completato due volte. Una volta raggiunto lo stato `completato`, è chiuso permanentemente.

### Pagina di stato link non attivo

Va costruita una pagina leggera (`mybblo_link_scaduto.html`) con messaggio neutro, coerente con il design system del Profiler, senza spiegazioni tecniche. La titolare non viene coinvolta automaticamente — è solo un dead-end pulito per chi prova ad accedere a un link non più valido.

### Integrazione con il contatore licenze

Questa funzione è strettamente collegata alla quarta (contatore lato server). Il momento in cui il contatore scala deve coincidere esattamente con il momento in cui il link passa allo stato `completato`. Le due operazioni devono avvenire in modo atomico — o entrambe riescono, o nessuna delle due viene registrata — per evitare disallineamenti tra quiz consumati e link completati.

### Generazione del link

La titolare non genera il link manualmente. Il sistema deve esporre un endpoint semplice (chiamabile dal briefing o da una futura dashboard) che:
1. Riceve l'identificativo della licenza della titolare
2. Genera un token univoco e non indovinabile
3. Registra il link in stato `generato` nel database
4. Restituisce l'URL completo da inviare alla cliente

La modalità con cui la titolare attiva questo endpoint (bottone nel briefing, link in email, futura dashboard) è da definire con BBLO nella fase successiva — per ora il developer deve solo costruire l'endpoint e documentarne l'interfaccia.

## Variabili d'ambiente necessarie

- Chiave API Brevo (da richiedere a Luca/BBLO, non generarla autonomamente).
- Eventuale secret per la firma/validazione dei token, se si usa un approccio a token firmati piuttosto che un database di lookup.

## Mappa dei file coinvolti

- `mybblo_calcolatore.html` — pagina ospitata (non un'email vera), letta dalla titolare. Contiene i due bottoni con i placeholder `{{LINK_CONFERMATO}}` e `{{LINK_MARCA_RIFIUTO}}`.
- `mybblo_conferma_azione.html` — pagina ospitata, già completa lato frontend. Riceve `azione`, `nome`, `token` come parametri nell'URL. Il TODO per il developer è dentro questo file, nella funzione `conferma()`.
- `mybblo_notifica_giorno10.html` — questo invece è il corpo reale dell'email che Brevo invia il giorno 10, non una pagina ospitata: usa i merge tag di Brevo ({{NOME_CENTRO}}, {{NOME_CLIENTE}}, {{LINK_BLOCCA_EMAIL_B}}), che Brevo sostituisce da solo all'invio.
- `mybblo_link_scaduto.html` — **da costruire** — pagina leggera mostrata quando un link monouso non è più attivo (stato `completato` o `scaduto`). Messaggio neutro, design coerente con il Profiler.

## Consegna finale

Il punto sui placeholder è già risolto, non resta più una scelta aperta: `{{LINK_CONFERMATO}}`, `{{LINK_MARCA_RIFIUTO}}` e `{{LINK_BLOCCA_EMAIL_B}}` nelle email vanno sostituiti (da Brevo, al momento dell'invio, oppure da chi genera il link nel caso di `mybblo_calcolatore.html`) con l'URL completo di `mybblo_conferma_azione.html`, con i parametri giusti in coda — ad esempio: `mybblo_conferma_azione.html?azione=rifiuto&nome=Maria&token=abc123`. Non sono tre endpoint diversi da costruire, sono tre varianti dello stesso URL verso la stessa pagina.

Alla fine del lavoro, il developer deve fornire:
1. La funzione serverless funzionante, collegata al `TODO developer` dentro `mybblo_conferma_azione.html`, con le tre azioni (rifiuto, confermato, blocca) che chiamano correttamente l'API di Brevo.
2. Il sistema di link monouso con gestione degli stati (generato → aperto → completato/scaduto), endpoint di generazione documentato, e pagina `mybblo_link_scaduto.html`.
3. Il contatore licenze lato server integrato con il sistema di link monouso — le due operazioni di scala contatore e transizione stato link devono essere atomiche.
4. Conferma di come viene generato il token per ciascun contatto, e di come viene costruito l'URL completo verso `mybblo_conferma_azione.html`.
5. Breve nota su dove è ospitata la funzione e come accedere ai log in caso di problemi.
