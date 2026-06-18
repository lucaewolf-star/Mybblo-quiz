# MY.BBLO — Sequenza Follow-up Contatti Tiepidi

## Logica generale

**Destinatario:** cliente finale che ha completato il quiz e lasciato i dati, ma non ha confermato il check-up dopo la chiamata della titolare.

**Trigger:** manuale, legato all'esito della telefonata e non a un timer automatico dopo il quiz. La sequenza esiste solo per chi ha completato il quiz e lasciato i dati (senza quel passaggio non arriva nessun briefing alla titolare, quindi non c'è nulla da attivare). La titolare chiama il lead entro 3 giorni dal completamento del quiz come da prassi, e solo se l'esito della chiamata è un rifiuto o un rinvio del check-up segna l'esito per attivare la sequenza. Funziona allo stesso modo indipendentemente dal canale con cui il quiz è stato completato (in istituto o a casa via WhatsApp), perché la notifica alla titolare passa comunque per Brevo.

**Struttura:** 2 email per ciascuno dei 4 inestetismi (adipe, cellulite, ritenzione, lassità), segmentate per inestetismo principale dichiarato nel quiz — non sui 20 profili R01-R20.

- **Email A** — 5 giorni dopo l'esito della chiamata. Rinforza la diagnosi, nessuna vendita, soft mention della categoria di prodotto domiciliare (senza brand).
- **Email B** — 11 giorni dopo l'esito della chiamata. Leva commerciale: acquisto del prodotto domiciliare con seduta omaggio inclusa, più frase di sicurezza per chi ha già prenotato il check-up nel frattempo.

**Nota tecnica per il socio (Brevo):** la titolare non ha un account Brevo personale, quindi non può assegnare tag o gestire l'automazione da una dashboard. La sequenza si attiva tramite un link d'azione protetto (token univoco per contatto) inserito nell'email di briefing che la titolare già riceve (vedi `mybblo_calcolatore.html`) — il click porta a una pagina di conferma esplicita, e solo dopo la conferma una funzione serverless attiva la sequenza su Brevo. Per fermare l'Email B non serve un tag generico controllato prima di ogni invio: basta una notifica dedicata, inviata il giorno 10 (il giorno prima dell'Email B), con un link che — se cliccato e confermato — cancella direttamente l'invio dell'Email B per quel contatto specifico. È un'azione puntuale, non una condizione da verificare ad ogni passaggio della sequenza, perché l'unica email successiva da poter bloccare è proprio quella.

---

## Notifica di controllo — Giorno 10

Email interna alla titolare (non al cliente finale), inviata automaticamente a tutti i contatti ancora nella sequenza al giorno 10, indipendentemente dall'esito — è la titolare a decidere se agire o ignorarla.

**Oggetto:** {{nome}} ha già prenotato il check-up?

**Corpo:** Ciao {{nome_centro}}, tra un giorno {{nome}} riceverà una proposta commerciale legata al check-up che aveva rinviato. Se nel frattempo ha già prenotato o si è presentata, clicca qui per fermarla: [link]. Se non hai ancora notizie da lei, non devi fare nulla — il sistema procederà in automatico.

---

## Adipe

### Email A — 5 giorni
**Oggetto:** {{nome}}, una cosa che vale la pena ricordare sul tuo adipe

**Corpo:** Ciao {{nome}}, ci siamo parlate qualche giorno fa e capisco perfettamente che a volte serve del tempo prima di decidere di fare qualcosa per sé — nessuna fretta da parte nostra. Volevo solo lasciarti un pensiero su quello che è emerso dal tuo profilo: l'adipe non è sempre uguale — può avere una consistenza più compatta oppure una componente più legata alla circolazione, e il modo in cui si lavora cambia in base a questo. È per questo che il tuo caso ha caratteristiche precise, diverse da quelle di un'altra persona. Una cosa che in tante non sanno è che il lavoro fatto in cabina rende ancora meglio se accompagnato da una piccola routine quotidiana pensata per il proprio tipo di tessuto — non la sostituisce, ma la sostiene nel tempo. Conoscere davvero come funziona il proprio corpo è già un passo avanti — il resto lo scegli tu, quando sarai pronta. A presto, {{nome_centro}}

### Email B — 11 giorni
**Oggetto:** Un modo concreto per iniziare, {{nome}}

**Corpo:** Ciao {{nome}}, abbiamo pensato a qualcosa di speciale per te: acquistando il tuo prodotto per la cura a casa, ricevi in omaggio il primo trattamento di modellamento corporeo, pensato per farti percepire da subito una sensazione di maggiore tonicità e compattezza nella zona che ti interessa. È un modo concreto per toccare con mano quanto può cambiare la tua percezione di benessere già dal primo passo, senza dover decidere tutto in una volta. Se nel frattempo hai già prenotato il tuo check-up con noi, ignora questo messaggio (perché scoprirai qualcosa di unico pensato per chi inizia la sua trasformazione) — ti aspettiamo. A presto, {{nome_centro}}

---

## Cellulite

### Email A — 5 giorni
**Oggetto:** {{nome}}, una cosa che vale la pena ricordare sulla tua cellulite

**Corpo:** Ciao {{nome}}, ci siamo parlate qualche giorno fa e capisco perfettamente che a volte serve del tempo prima di decidere di fare qualcosa per sé — nessuna fretta da parte nostra. Volevo solo lasciarti un pensiero su quello che è emerso dal tuo profilo: la cellulite ha quasi sempre una componente legata alla circolazione e al drenaggio dei liquidi, non solo all'alimentazione. Una cosa che in tante non sanno è che il lavoro fatto in cabina rende ancora meglio se accompagnato da una piccola routine quotidiana pensata per il proprio tipo di tessuto — non la sostituisce, ma la sostiene nel tempo. Conoscere davvero come funziona il proprio corpo è già un passo avanti — il resto lo scegli tu, quando sarai pronta. A presto, {{nome_centro}}

### Email B — 11 giorni
**Oggetto:** Un modo concreto per iniziare, {{nome}}

**Corpo:** Ciao {{nome}}, abbiamo pensato a qualcosa di speciale per te: acquistando il tuo prodotto per la cura a casa, ricevi in omaggio la prima seduta di mobilizzazione dei liquidi, pensata per farti percepire da subito un cambiamento nella sensazione di leggerezza e nel volume percepito delle tue circonferenze. È un modo concreto per toccare con mano quanto può cambiare la tua percezione di benessere già dal primo passo, senza dover decidere tutto in una volta. Se nel frattempo hai già prenotato il tuo check-up con noi, ignora questo messaggio (perché scoprirai qualcosa di unico pensato per chi inizia la sua trasformazione) — ti aspettiamo. A presto, {{nome_centro}}

---

## Ritenzione

### Email A — 5 giorni
**Oggetto:** {{nome}}, una cosa che vale la pena ricordare sulla tua ritenzione

**Corpo:** Ciao {{nome}}, ci siamo parlate qualche giorno fa e capisco perfettamente che a volte serve del tempo prima di decidere di fare qualcosa per sé — nessuna fretta da parte nostra. Volevo solo lasciarti un pensiero su quello che è emerso dal tuo profilo: la ritenzione è legata soprattutto al modo in cui i liquidi circolano e vengono drenati nei tessuti, non solo a quanta acqua si beve o al sale nella dieta come spesso si pensa. Una cosa che in tante non sanno è che il lavoro fatto in cabina rende ancora meglio se accompagnato da una piccola routine quotidiana pensata per il proprio tipo di tessuto — non la sostituisce, ma la sostiene nel tempo. Conoscere davvero come funziona il proprio corpo è già un passo avanti — il resto lo scegli tu, quando sarai pronta. A presto, {{nome_centro}}

### Email B — 11 giorni
**Oggetto:** Un modo concreto per iniziare, {{nome}}

**Corpo:** Ciao {{nome}}, abbiamo pensato a qualcosa di speciale per te: acquistando il tuo prodotto per la cura a casa, ricevi in omaggio il primo trattamento drenante, pensato per farti percepire da subito una sensazione di maggiore leggerezza nella zona che ti interessa. È un modo concreto per toccare con mano quanto può cambiare la tua percezione di benessere già dal primo passo, senza dover decidere tutto in una volta. Se nel frattempo hai già prenotato il tuo check-up con noi, ignora questo messaggio (perché scoprirai qualcosa di unico pensato per chi inizia la sua trasformazione) — ti aspettiamo. A presto, {{nome_centro}}

---

## Lassità

### Email A — 5 giorni
**Oggetto:** {{nome}}, una cosa che vale la pena ricordare sulla tua lassità

**Corpo:** Ciao {{nome}}, ci siamo parlate qualche giorno fa e capisco perfettamente che a volte serve del tempo prima di decidere di fare qualcosa per sé — nessuna fretta da parte nostra. Volevo solo lasciarti un pensiero su quello che è emerso dal tuo profilo: la lassità ha a che fare con il supporto strutturale della pelle e dei tessuti, non è solo una questione di età come spesso si pensa. Una cosa che in tante non sanno è che il lavoro fatto in cabina rende ancora meglio se accompagnato da una piccola routine quotidiana pensata per il proprio tipo di tessuto — non la sostituisce, ma la sostiene nel tempo. Conoscere davvero come funziona il proprio corpo è già un passo avanti — il resto lo scegli tu, quando sarai pronta. A presto, {{nome_centro}}

### Email B — 11 giorni
**Oggetto:** Un modo concreto per iniziare, {{nome}}

**Corpo:** Ciao {{nome}}, abbiamo pensato a qualcosa di speciale per te: acquistando il tuo prodotto per la cura a casa, ricevi in omaggio il primo trattamento tonificante, pensato per farti percepire da subito una sensazione di maggiore tonicità e sostegno nella zona che ti interessa. È un modo concreto per toccare con mano quanto può cambiare la tua percezione di benessere già dal primo passo, senza dover decidere tutto in una volta. Se nel frattempo hai già prenotato il tuo check-up con noi, ignora questo messaggio (perché scoprirai qualcosa di unico pensato per chi inizia la sua trasformazione) — ti aspettiamo. A presto, {{nome_centro}}
