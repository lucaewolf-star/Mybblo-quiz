# MY.BBLO Profiler — Guida operativa per la configurazione

Questa guida è pensata per chi configura il sistema senza basi di programmazione. Copre tutto ciò che si può fare tramite interfacce (Brevo, GitHub, pannello del dominio) — non copre la scrittura di codice, che è oggetto di un documento separato per un developer esterno.

Segui le fasi in ordine: ognuna spiega cosa fare, perché, e cosa serve prima di iniziarla.

---

## Fase 0 — Accesso al dominio

Il dominio mybblo.com è registrato tramite una società terza. Prima di qualsiasi altra cosa, richiedi le credenziali di accesso al pannello di gestione del dominio (di solito si chiama "pannello DNS" o "area clienti" del registrar). Senza questo accesso non si può:
- configurare l'invio email da Brevo con il proprio dominio (altrimenti le email rischiano lo spam);
- eventualmente collegare un dominio personalizzato alle pagine ospitate.

**Cosa richiedere:** username e password (o le credenziali di accesso) all'area di gestione DNS del dominio mybblo.com.

---

## Fase 1 — Hosting dei file (GitHub Pages)

I file HTML del prodotto (quiz, briefing, calcolatore, email) sono ospitati su GitHub Pages — un servizio gratuito che pubblica file HTML come pagine web raggiungibili da un link pubblico.

**Come funziona, in breve:** i file vivono in una cartella (un "repository") su GitHub. Ogni volta che un file viene caricato o modificato in quella cartella, la pagina online si aggiorna da sola, di solito in pochi minuti.

**Cosa devi sapere fare:**
1. Accedere al repository GitHub del progetto (richiedi le credenziali a Luca se non le hai già).
2. Caricare un file nuovo o sostituire un file esistente (su GitHub questo si fa dal sito stesso, senza bisogno di programmi particolari — basta trascinare il file nella cartella giusta tramite l'interfaccia web di GitHub).
3. Verificare che il link pubblico del file funzioni, aprendolo nel browser dopo l'aggiornamento.

**File principali che troverai in questa cartella** (elenco di riferimento, vedi anche la tabella finale):
- `mybblo_briefing_test.html` — il quiz/briefing principale
- `mybblo_calcolatore.html` — il briefing con calcolatore che riceve il centro
- `mybblo_email_utente.html` — l'email che riceve il cliente finale
- i file di licenza (P10, P50, P100, P200, ILLIMITATO)

---

## Fase 2 — Configurazione DNS

Questa fase serve a due cose: far sì che Brevo possa inviare email dal dominio mybblo.com senza finire nello spam, ed eventualmente collegare un dominio personalizzato alle pagine ospitate.

**Per l'invio email (obbligatorio):** Brevo, una volta collegato il dominio nelle sue impostazioni, fornisce 2-3 record DNS da aggiungere (in genere di tipo TXT e CNAME, per l'autenticazione SPF/DKIM). Vanno copiati esattamente come Brevo li mostra e incollati nel pannello DNS del dominio (Fase 0). Brevo stesso guida passo-passo questa procedura dalla propria interfaccia — segui le sue istruzioni al momento del collegamento dominio, senza bisogno di altre conoscenze tecniche oltre a copiare e incollare con attenzione.

**Per un dominio personalizzato sulle pagine GitHub (opzionale):** se in futuro si vuole un indirizzo tipo `app.mybblo.com` invece del link standard di GitHub Pages, serve aggiungere un record CNAME nel pannello DNS — GitHub fornisce le istruzioni esatte nelle impostazioni del repository quando si configura un dominio personalizzato.

---

## Fase 3 — Configurazione Brevo

Questa è la parte più consistente. Brevo gestisce due automazioni email separate, più il tracciamento degli esiti delle chiamate.

### 3.1 — Liste e contatti

Crea (se non esiste già) una lista per i contatti "centro estetico" (i destinatari delle email di onboarding) e una lista o segmento per i contatti "cliente finale" tiepidi (i destinatari della sequenza follow-up). Tienile separate: sono due pubblici diversi con contenuti diversi.

### 3.2 — Automazione "Onboarding centro estetico" (30 giorni)

Contenuto: vedi il file `sequenza_onboarding_brevo.md` — 9 email su 30 giorni, a partire dall'acquisto della licenza.

**Trigger:** acquisto di una licenza **a pagamento** (P10, P50, P100, P200 o ILLIMITATO) da parte del centro — il contatto entra nell'automazione in quel momento. **La prova gratuita NON deve attivare questa sequenza** — è un evento distinto e va trattato come tale nella configurazione del trigger in Brevo.

**Per la prova gratuita:** al posto dell'onboarding automatico, Luca invia manualmente una singola email con le 4 azioni fondamentali per usare bene i 10 quiz in 7 giorni. Il testo completo è nel file `email_prova_gratuita.md`.

**Timing:** giorni 0, 1, 2-3, 4-5, 9-10, 12, 14-16, 22-24, 30 dall'ingresso. Imposta ogni email come "ritardo" (delay) dal trigger iniziale, secondo questi giorni.

**Importante:** nessuna condizione di uscita prevista per questa sequenza — tutte le 9 email vanno inviate indipendentemente da cosa fa il centro nel frattempo, è una sequenza di accompagnamento, non commerciale.

### 3.3 — Automazione "Follow-up contatti tiepidi"

Contenuto: vedi il file `sequenza_followup_tiepidi.md` — 8 email (2 per ciascuno dei 4 inestetismi), più una notifica di controllo interna.

**Trigger:** il contatto entra nell'automazione quando la titolare clicca il link "Rifiuto o rinvio del check-up" nell'email di briefing (`mybblo_calcolatore.html`) e confirma sulla pagina di conferma. Questo click arriva alla funzione serverless (vedi documento per il developer), che è lei a far entrare il contatto nell'automazione Brevo tramite l'API — non è un'azione che fai manualmente dentro Brevo.

**Timing:**
- Email A — giorno 5 dall'ingresso (contenuto secondo l'inestetismo dichiarato dal cliente nel quiz)
- Notifica di controllo (interna alla titolare) — giorno 10
- Email B — giorno 11 (contenuto secondo l'inestetismo)

**Segmentazione per inestetismo:** la sequenza deve mandare il contenuto giusto (adipe / cellulite / ritenzione / lassità) in base all'inestetismo dichiarato dal cliente nel quiz — questo dato arriva insieme al contatto quando entra in automazione (il developer te lo passerà come campo/tag insieme al trigger). Configura 4 rami dell'automazione, uno per inestetismo, ciascuno con la propria Email A e Email B.

**Blocco dell'Email B:** se la titolare clicca il link nella notifica del giorno 10 e confirma, l'invio dell'Email B per quel contatto deve essere cancellato. Anche questa azione passa dalla funzione serverless via API, non da un intervento manuale tuo in Brevo — il tuo compito è solo assicurarti che l'automazione supporti la rimozione di uno step già programmato per un singolo contatto (le piattaforme di automazione email lo permettono tramite API; verifica con il developer come viene implementato lato Brevo).

### 3.4 — Tracciamento "Check-up confermato"

Quando la titolare clicca "Check-up confermato" nel briefing, il contatto viene taggato come convertito (sempre tramite la funzione serverless). Non serve costruire nessuna automazione per questo — è solo un dato che si accumula nei contatti Brevo, utile più avanti per analisi e leve commerciali. Assicurati solo che il tag esista e sia leggibile/filtrabile dentro Brevo (per costruire in futuro segmenti o report).

---

## Fase 4 — Collegamento con la funzione serverless

Il developer che costruirà la funzione serverless (vedi documento tecnico separato) ti fornirà alla fine 2-3 URL (link) — uno per ciascuna azione (marca rifiuto, segnala confermato, blocca Email B). Il tuo compito è inserire quei link nei punti giusti:
- nel file `mybblo_calcolatore.html`, dove oggi ci sono i placeholder `{{LINK_CONFERMATO}}` e `{{LINK_MARCA_RIFIUTO}}`;
- nella notifica del giorno 10, dove c'è il placeholder per il link di blocco.

Il developer ti dirà anche se questi link devono essere passati come parametro nell'URL della pagina (cioè uniti al link che apre il briefing, con i dati del cliente) oppure inseriti come tag fissi — è una scelta tecnica che dipende da come è costruita la pagina, te la confermerà lui prima della consegna.

---

## Fase 5 — Test prima del lancio

Prima di attivare tutto su contatti reali, fai un giro di test completo:
1. Simula un nuovo lead, verifica che il briefing arrivi e che il calcolatore funzioni.
2. Clicca "Rifiuto o rinvio" su un contatto di prova, verifica che l'Email A arrivi al giorno corretto (puoi accelerare i tempi per il test, poi rimetterli a regime).
3. Verifica che la notifica del giorno 10 arrivi, e che cliccando il link di blocco l'Email B non parta.
4. Verifica che cliccando "Check-up confermato" il tag venga applicato correttamente al contatto.
5. Fai partire un'automazione di onboarding di prova e controlla che le 9 email arrivino ai giorni giusti.

Solo dopo aver verificato tutti questi punti, passa a contatti reali.

---

## Riferimento rapido — file del progetto

**Da hostare su GitHub Pages (sono pagine vere, devono avere un link pubblico funzionante):**

| File | Cosa fa |
|---|---|
| `mybblo_briefing_test.html` | Quiz e briefing principale (prodotto interno) |
| `mybblo_calcolatore.html` | Briefing + calcolatore ricevuto dal centro, con i link di esito chiamata |
| `mybblo_email_utente.html` | Email ricevuta dal cliente finale dopo il quiz |
| Licenze P10/P50/P100/P200/ILLIMITATO | File HTML con banner e blocco a esaurimento — verificare che siano caricati su GitHub prima del test, non sono ancora presenti nel repository al momento di scrivere questa guida |

**Solo testo di riferimento (non serve che siano online, servono per copiare i contenuti dentro Brevo):**

| File | Cosa fa |
|---|---|
| `sequenza_onboarding_brevo.md` | Testi delle 9 email di onboarding (30 giorni) |
| `sequenza_followup_tiepidi.md` | Testi delle 8 email di follow-up + notifica giorno 10 |

Tenerli comunque nel repository va benissimo, per comodità e backup — ma non è un requisito come per i file HTML.
