# Guida utente — cfai-code

Questa guida spiega come installare e usare `cfai-code`, la chat da terminale costruita sul motore Rust di questo
progetto. Non è un documento di ricerca: per lo stato del porting, le prove di conformità e le decisioni tecniche,
vedi [README.md](README.md) e [research/](research/). Le voci senza data sono stabili da più aggiornamenti; quando
qualcosa cambia nel binario, questa guida va aggiornata insieme al codice.

## 1. Installazione

Serve la toolchain Rust (`cargo`) e, la prima volta, una connessione per scaricare le dipendenze.

```powershell
cargo build --release --offline -p cfai-code
```

Il binario risulta in `target\release\cfai-code.exe`. Copialo dove preferisci, oppure lancialo da lì restando nella
cartella del progetto su cui vuoi lavorare: `cfai-code` legge e scrive sempre rispetto alla cartella da cui parte
(`--cwd` per indicarne un'altra).

## 2. Accesso

`cfai-code` deve sapere con quale modello parlare. Ci sono tre strade, in ordine di semplicità:

1. **Account CFAI (consigliato).** Al primo avvio, se non è impostata nessuna chiave, compare la schermata di
   benvenuto e viene chiesta email e password dell'account CFAI. Il token ricevuto resta salvato nella cartella di
   configurazione dell'utente e viene riusato ai prossimi avvii: non va più chiesto nulla. Per accedere o uscire in
   qualsiasi momento, senza aprire la chat:
   ```powershell
   cfai-code login
   cfai-code logout
   ```
2. **Una chiave Anthropic propria.** `cfai-code config` e scegli `provider: anthropic`, poi inserisci la chiave
   (`api_key`) e, se vuoi, il modello. Nella build che viene distribuita la richiesta passa comunque dal servizio
   CFAI (la tua chiave viaggia in un'intestazione dedicata e non è mai l'account che paga); i dettagli tecnici sono
   in `research/sdk-porting.md`.
3. **Un modello tuo, anche locale.** `provider: custom`, con `base_url` che punta a un endpoint che parla l'API
   Messages di Anthropic (per esempio un server locale su `http://localhost:8080`) e, se il server lo richiede,
   `api_key`.

In alternativa alle impostazioni salvate, l'ambiente ha sempre la precedenza: `ANTHROPIC_API_KEY`,
`ANTHROPIC_MODEL` e `ANTHROPIC_BASE_URL`, letti dall'ambiente del terminale o da un file `.env` nella cartella di
lavoro (`--env-file FILE` ne indica un altro).

## 3. Le impostazioni (`cfai-code config`)

```powershell
cfai-code config            # apre un menu interattivo
cfai-code config list       # mostra i valori correnti (la chiave è mascherata)
cfai-code config get <nome>
cfai-code config set <nome> <valore>
cfai-code config reset      # cancella tutte le impostazioni salvate
```

Lo stesso menu è raggiungibile dentro la chat con `/config` (o `/settings`). Le impostazioni disponibili:

| Nome | Valori | Significato |
| --- | --- | --- |
| `provider` | `cfai` (predefinito) · `anthropic` · `custom` | il servizio CFAI, una chiave Anthropic propria, o un modello proprio |
| `model` | testo libero | nome del modello; vuoto tiene il predefinito del provider |
| `base_url` | testo libero | solo per `custom`: l'indirizzo dell'endpoint |
| `api_key` | segreto | solo per `anthropic` e `custom` (un modello locale che non la richiede può lasciarla vuota) |
| `temperature` | numero da 0 a 2 | vuoto tiene il predefinito del provider |
| `reasoning` | `default` · `low` · `medium` · `high` | quanto ragiona il modello del servizio CFAI prima di rispondere (solo `cfai`) |
| `verify_command` | testo libero | il comando che controlla il lavoro, per esempio `python -m unittest -q` o `cargo test`; vuoto: nessun controllo |
| `verify_every` | numero da 0 a 1000 | esegue il controllo da solo dopo tante chiamate a un tool dall'ultima volta (0: solo quando il modello dice di aver finito) |
| `verify_tries` | numero da 0 a 20 | quante volte un controllo fallito torna al modello prima che subentri quello più forte |
| `escalate_model` | testo libero | il modello che subentra se i controlli continuano a fallire (`cfai-escalate` sul servizio CFAI); vuoto: nessuno |
| `profile` | `local` · `gateway` | solo `cfai`: se il prompt per il modello piccolo viene accorciato qui (`local`) o dal servizio (`gateway`, richiede che il servizio lo supporti) |

Le impostazioni salvate valgono come predefinite: una variabile d'ambiente già impostata dall'utente vince sempre su
di esse.

## 4. Avviare la chat

```powershell
cfai-code                        # apre la chat nella cartella corrente
cfai-code --cwd C:\percorso       # apre la chat in un'altra cartella
cfai-code -p "fai questo"         # risponde una volta e esce, senza aprire la chat
```

Alcune opzioni utili all'avvio (l'elenco completo è in `cfai-code --help` e in
[research/inventory/cli-flags.md](research/inventory/cli-flags.md)):

- `--model <nome>` — usa un modello diverso da quello predefinito per questa sessione.
- `-c`, `--continue` — riprende l'ultima sessione di questa cartella.
- `-r`, `--resume [id]` — elenca le sessioni salvate, o riprende quella con l'id dato.
- `--permission-mode <modo>` — `default` (chiede conferma), `acceptEdits`, `plan`, `dontAsk`, `bypassPermissions`.
- `--add-dir <cartelle...>` — concede l'accesso anche a cartelle fuori da quella di lavoro.
- `--no-project-instructions` — non legge i file `CLAUDE.md` della cartella e delle sue superiori.

Dentro la chat: Invio manda la riga; una barra rovesciata (`\`) alla fine di una riga continua sulla riga
successiva senza inviare. Ctrl-C interrompe la risposta in corso senza chiudere il programma.

## 5. Comandi della chat

```
/help                 questo elenco
/exit, /quit          esce
/clear, /new          apre una nuova sessione dal prossimo messaggio
/compact [note]       riassume la sessione per liberare contesto (le note indirizzano il riassunto)
/compact list          numera i messaggi scritti, per poter scegliere dove tagliare
/compact from N [note] riassume dal messaggio N in poi, tenendo quelli precedenti
/compact upto N [note] riassume quanto viene prima del messaggio N, tenendo il resto
/model [nome]          mostra o cambia il modello
/permissions [modo]    mostra o cambia il modo dei permessi
/resume [numero|id]    elenca le sessioni salvate, o ne riprende una
/cost                  token e costo di questa chat
/status                modello, cartella, modo e sessione in uso
/config [list|set <nome> <valore>]   le impostazioni descritte sopra
!comando               esegue un comando di shell qui, senza passare dal modello
/nome argomenti        esegue un comando di progetto (.claude/commands), una skill o un prompt MCP
@file                  allega un file al messaggio
```

## 6. Uso senza interazione (`-p`)

Per usare `cfai-code` da uno script o in una pipeline, senza aprire la chat:

```powershell
cfai-code -p "aggiungi un test per la funzione X" --output-format json
echo "testo del prompt" | cfai-code -p
```

`--output-format text|json|stream-json` sceglie il formato della risposta (`stream-json` richiede `--verbose`);
`--allowedTools`/`--disallowedTools` restringono o escludono singoli tool; `--max-turns` limita i giri di
conversazione. Il codice di uscita del processo segue l'esito della risposta.

## 7. Dove vengono salvati i dati

- **Impostazioni** (`cfai-code config`): `cfai-settings.json` nella cartella di configurazione dell'utente.
- **Accesso salvato**: `cfai-session.json`, stessa cartella; contiene solo il token del servizio, non la password.
- **Sessioni della chat**: `.cfai/sessions` dentro la cartella di lavoro (`--journal-dir` per cambiarla). È il
  formato nativo del progetto, non quello del CLI `claude` originale.

Nessuno di questi file viene inviato da `cfai-code` a un server diverso dal servizio scelto in `provider`.

## 8. Problemi comuni

- **«set --model or ANTHROPIC_MODEL»**: nessun modello è stato scelto né con `--model` né con `ANTHROPIC_MODEL` né
  con `cfai-code config`. Imposta uno dei tre.
- **Il programma chiede email e password a ogni avvio**: il login non si è salvato, probabilmente perché la
  cartella di configurazione dell'utente non è scrivibile. Prova `cfai-code login` ed controlla l'errore stampato.
- **Il modello non vede un file fuori dalla cartella di lavoro**: aggiungi la cartella con `--add-dir`, oppure
  riavvia con `--cwd` sulla cartella giusta.
- **Una chiave API sembra non essere quella attesa**: l'ambiente (`ANTHROPIC_API_KEY`/`ANTHROPIC_AUTH_TOKEN`/
  `ANTHROPIC_BASE_URL` già impostate nel terminale) vince sempre sulle impostazioni salvate con `cfai-code config`.
  Controlla con `cfai-code config list` cosa è salvato, e con `echo $env:ANTHROPIC_API_KEY` cosa c'è nell'ambiente.
