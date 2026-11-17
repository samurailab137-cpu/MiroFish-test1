# MiroFish — note per le sessioni Claude Code

Questo fork di MiroFish serve a testare lanci di prodotti/servizi e creatività pubblicitarie
simulando la reazione del pubblico. Branch di lavoro: `samurailab/inspiring-volta-c31jnh`.

## Configurazione LLM e Zep

Il backend legge queste variabili d'ambiente (file `.env` alla radice oppure ambiente):

```
LLM_API_KEY=<chiave Anthropic sk-ant-…, legata a un workspace>
LLM_BASE_URL=https://api.anthropic.com/v1/
LLM_MODEL_NAME=claude-haiku-5-5
ZEP_API_KEY=<chiave Zep Cloud>
```

- La chiave Anthropic deve appartenere a un workspace; una chiave personale multi-workspace
  viene rifiutata ("must include the anthropic-workspace-id header").
- Le chiavi non vanno mai committate né scritte in chat. Nelle sessioni cloud vanno nelle
  variabili d'ambiente dell'ambiente cloud; in Codespaces nei Codespaces secrets.
- Verifica rapida della chiave:
  `curl -s https://api.anthropic.com/v1/chat/completions -H "Authorization: Bearer $LLM_API_KEY" -H "content-type: application/json" -d "{\"model\":\"$LLM_MODEL_NAME\",\"max_tokens\":50,\"messages\":[{\"role\":\"user\",\"content\":\"ciao\"}]}"`

## Avvio

```
npm run setup:all   # dipendenze root + frontend + backend (uv)
npm run dev         # frontend :3000, backend :5001
```

Il backend si ferma subito se mancano `LLM_API_KEY` o `ZEP_API_KEY`.
Codespaces: `.devcontainer/` installa tutto e apre la porta 3000.
Fermare sempre l'app (Ctrl+C) prima di `git pull`: Vite ricarica la pagina.

## Modifiche fatte su questo branch (rispetto all'upstream)

- Supporto Claude via endpoint OpenAI-compatibile di Anthropic
  (`backend/app/utils/openai_chat_compat.py`): per i modelli `claude-*` niente
  `temperature` né `response_format`, `max_tokens` sempre presente (default 16000);
  gli script OASIS passano `max_tokens=4096`.
- Simulazione: lo stato COMPLETED viene pubblicato quando entrambe le piattaforme hanno
  finito e Zep ha ingerito tutto, anche se il processo resta vivo per le interviste
  (prima il pulsante "Generate Report" non si attivava mai).
- Uno stato "running" rimasto orfano dopo un riavvio del backend non blocca più il riavvio;
  `paused`/`stopping` contano come "preparato".
- Lo Step 3 si riaggancia alla simulazione esistente invece di riavviarla a ogni
  ricaricamento della pagina.
- Attesa batch Zep: timeout calcolato sull'assenza di progressi (10 min), tetto 60 min.
- Log del backend con il messaggio d'errore del provider LLM in caso di fallimento.

Test: `cd backend && uv run pytest -q tests`.

## Flusso API (per usare MiroFish senza interfaccia)

1. `POST /api/graph/ontology/generate` (multipart: file + richiesta di simulazione) → project_id
2. `POST /api/graph/build` → task_id; polling `GET /api/graph/task/<task_id>`
3. `POST /api/simulation/create`, poi `POST /api/simulation/prepare` + `/prepare/status`
4. `POST /api/simulation/start` (`platform: parallel`, `max_rounds`: 15–20 per i test)
   → polling `GET /api/simulation/<id>/run-status` fino a `completed`
5. `POST /api/report/generate` + `/generate/status` → `GET /api/report/<report_id>`

Controllare i parametri esatti nei file `backend/app/api/*.py` prima di scrivere un client.

## Lavoro in corso: campagna MEET YOUR FASCIA (Fascia Viva, Scarlett Canevari)

Lezione online gratuita su fascia, mandibola/cervicale e alcune forme di acufene.
File in `campagne/meet-your-fascia/`:
- `01-varianti-originali.md`: 3 script video + 3 copy forniti dal cliente (documento da
  caricare in MiroFish come Reality Seed).
- `02-varianti-conformi-meta.md`: versioni riscritte per la policy Meta sugli attributi
  personali (niente "ACUFENE?" rivolto a chi guarda); completare [DATA] [ORA] [DURATA].

Prossimi passi:
1. Simulazione MiroFish sulle varianti (15–20 turni), prompt suggerito:
   "Simula come reagisce il pubblico italiano che soffre di acufene (40–70 anni, già visto da
   otorino o audioprotesista) e i professionisti sanitari che ne parlano online, quando vedono
   le 3 varianti di annuncio Meta per MEET YOUR FASCIA. Per ogni variante: chi si iscriverebbe
   e perché, quali obiezioni emergono, quali frasi generano scetticismo o accuse di promesse
   sanitarie, quale variante genera più interesse. Concludi con una classifica e le modifiche
   consigliate."
2. Test A/B su Meta, variante 1 vs 3 (versioni conformi), account pubblicitario
   "Scarlet Canevari Yoga", pagina "Rilasciomiofascialescarlet", obiettivo Contatti (modulo).
   Riferimento: campagna "Lead - fasciaviva - Ottobre 2026" (pubblico professionisti),
   ~1,41 € per contatto. Proposta: Italia 40–70, Advantage+, 10–15 €/giorno per variante,
   7 giorni, tutto creato IN PAUSA.
   In attesa dal cliente: i 2 video registrati, data/ora/durata dell'evento, modulo Meta o
   landing page, conferma di budget e periodo. Nessuna campagna va attivata senza un ok esplicito.
