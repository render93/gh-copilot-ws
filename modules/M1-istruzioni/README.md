# Modulo M1 - Istruzioni · AGENTS.md + Skills

> Obiettivo: capire come si descrive un repo in un file AGENTS.md e come si crea una Skill. Vedere in pratica la differenza tra "regole sempre attive" e "know-how caricato quando serve".

> 🔵 **Claude Code?** Questo modulo è identico. `AGENTS.md` è già cross-tool: al root del repo c'è `CLAUDE.md` che lo importa (`@AGENTS.md`), quindi Claude Code lavora con lo **stesso contratto**. La Skill che crei nello Step 2 vale per entrambi gli strumenti: cambia solo la cartella (`.claude/skills/` invece di `.github/skills/`). Sotto ogni passo hands-on trovi un blocco 🔵 con l'equivalente esatto.

## Teoria

### AGENTS.md

`AGENTS.md` è lo standard `de facto` cross-tool (GitHub Copilot, Claude Code, e altri coding agent) per descrivere il *contratto* del repository all'agente. È un file Markdown che viene iniettato come parte del system prompt in tutte le sessioni.

**Cosa includere**:
- **Regole architetturali**: stack tecnologico, layering, decisioni di design che non si vogliono violare.
- **Convenzioni di codice**: naming, validation, error handling, struttura dei test.
- **Vincoli non negoziabili**: cose che l'agente non deve mai fare ("non modificare X", "esegui sempre Y prima di commit").
- **Punti di ingresso**: dove cercare cosa nel repository.

**Cosa NON includere**:
- Documentazione esaustiva del progetto.
- Esempi lunghi di codice (vanno in skill o in commenti del codice).
- Storia delle decisioni o changelog (lo dice `git log`).
- Informazioni che cambiano frequentemente (ad ogni modifica si paga la regression del prompt se non costantemente aggiornato).

**Vincolo pratico di lunghezza**: meno di 200 righe è consigliato, hard-cap sui 500. Sopra questa soglia: il costo in token per prompt diventa percepibile, l'attenzione dell'agente si distribuisce, ed è il segnale che alcune regole vanno spostate in una Skill.

### Skill

Una skill è uno standard e contiene un insieme di risorse e istruzioni che forniscono al coding agent know-how procedurale su *come* fare qualcosa di specifico. La skill è caricata dall'agente solo quando necessario, in base alla sua `description`. Questo permette di mantenere il prompt leggero e focalizzato, pagando il costo in token solo quando si ha effettivamente bisogno di quel know-how.

La struttura è la seguente:

```
skills/<nome-skill>/
├── SKILL.md          <= frontmatter YAML + corpo istruzioni
├── scripts/          <= (opzionale) script eseguibili
└── resources/        <= (opzionale) template, esempi, schema
```

Il file `SKILL.md` ha un frontmatter YAML con due campi obbligatori:

```yaml
---
name: endpoint-creator
description: Use when creating a new REST endpoint in this repo. Covers validation, code location, status codes, test structure, and naming.
---
```

- `name`: identificatore tecnico in kebab-case.
- `description`: la frase con cui l'agente decide **se e quando** caricare la skill. Una buona `description` produce auto-loading affidabile; una vaga rischia di non essere caricata quando serve o, al contrario, di essere caricata in contesti non rilevanti.

### Differenza pratica AGENTS.md vs Skill

| | AGENTS.md | Skill |
|---|---|---|
| Caricamento | sempre, in ogni prompt | on-demand, quando l'agente lo decide |
| Costo token | costante | pagato solo se serve |
| Granularità | regole macro del repo | procedura specifica (es. "creare un endpoint") |
| Numero per repo | uno (al root) | molte |

## Hands-on

Scegli il tuo linguaggio in `starters/dotnet`, `starters/typescript`, `starters/python` e segui i passi.

### Step 1 - Verifica che AGENTS.md guidi l'agente

Lo starter ha già un `AGENTS.md` nella root che descrive: stack, struttura della Task API, convenzioni di naming, validazione, status code, vincolo "ogni endpoint ha un test".

In Copilot Chat (modalità **Agent**) chiedi:
> Nel progetto `@modules/M1-istruzioni/starters/<linguaggio>` aggiungi un endpoint `PUT /tasks/:id` che sostituisce completamente un task esistente. Il body deve contenere `title` e `status`, e l'endpoint restituisce il task aggiornato.

Osserva nella risposta dell'agente:
- la posizione del nuovo endpoint (allineata alle convenzioni in AGENTS.md);
- la **validazione del body** (entrambi i campi richiesti - `PUT` ha semantica *replace*, quindi tutti i campi del payload sono obbligatori);
- gli status code di risposta (`200` se il task esiste ed è aggiornato, `404` se l'id non esiste, `400` se il body è invalido);
- il test generato accanto al codice di produzione.

**Verifica**: l'agente segue il contratto della repo *senza che tu glielo abbia detto in chat*. È AGENTS.md che lo ha guidato.

<details>
<summary>🔵 <b>Claude Code — Step 1</b></summary>

Identico. Apri il pannello **Claude Code** in VS Code e, in una nuova conversazione, incolla **lo stesso prompt** presente qui sopra. Claude Code carica `CLAUDE.md` (che importa `AGENTS.md`) all'avvio della sessione, quindi segue lo stesso contratto di repo: posizione dell'endpoint, validazione, status code, test accanto al codice. La verifica è la stessa — è `AGENTS.md` (via `CLAUDE.md`) a guidare l'agente, senza istruzioni in chat.

</details>

### Step 2 - Crea una Skill e osserva l'auto-loading

Crea il file `.github/skills/endpoint-creator/SKILL.md` **nella root del workspace** (la root del repo che hai forkato), con questo contenuto:

> **Nota**: VS Code Copilot Chat cerca le skill in `.github/skills/` al workspace root — ecco perché va lì, non dentro la cartella della starter.

```yaml
---
name: endpoint-creator
description: Use when creating a new REST endpoint in this repo. Covers validation, code location, status codes, test structure, and naming.
---

# How to create a REST endpoint in this repo

## 1. Code location
- .NET: `Tasks/TasksEndpoints.cs` (inside `MapTasks`)
- TypeScript: `src/tasks/routes.ts` (inside `tasksRoutes()`)
- Python: `app/main.py` (function decorated with `@app.<verb>`)

## 2. Input validation
- Invalid input => 400 with body `{ "error": "<short message>" }`
- Python: pydantic.BaseModel; TS: type-narrowing; .NET: record + manual checks

## 3. Status codes
- 200 GET · 201 POST + Location header · 200 PUT (replace) · 200 PATCH · 204 No Content (DELETE) · 404 not found · 400 validation

## 4. Tests
Every endpoint ships with one happy path test and one error case test.

## 5. Naming
Kebab-case paths (e.g. `/tasks/by-status`). Typed path params (`{id:int}` in .NET).
```

Ora apri una nuova chat e chiedi all'agente, **senza nominare la skill**:
> Nel progetto `@modules/M1-istruzioni/starters/<linguaggio>` aggiungi un endpoint `DELETE /tasks/:id` che cancella un task esistente.

Nota: l'agente carica autonomamente la skill `endpoint-creator` perché la `description` del frontmatter dichiara *"Use when creating a new REST endpoint"*, e il prompt soddisfa quel trigger. Vedrai nella chat un riferimento esplicito relativo al caricamento della skill.

La skill può anche essere chiamata esplicitamente, ad esempio con:
> Usa la skill /endpoint-creator per aggiungere un endpoint `DELETE /tasks/:id` che cancella un task esistente.

<details>
<summary>🔵 <b>Claude Code — Step 2</b></summary>

Stesso identico contenuto di `SKILL.md`, solo un'altra cartella: crea il file in **`.claude/skills/endpoint-creator/SKILL.md`** al root del workspace (Claude Code carica le skill da `.claude/skills/`). Il frontmatter (`name` + `description`) e il corpo sono **identici** a quelli mostrati sopra — copiali tali e quali.

Poi apri una nuova conversazione in Claude Code e usa **lo stesso prompt** (`DELETE /tasks/:id`) **senza nominare la skill**: Claude la auto-carica perché la `description` matcha il task. Per invocarla esplicitamente digita `/endpoint-creator` nel prompt (le skill compaiono come slash command quando digiti `/`).

Reference: `modules/M1-istruzioni/solution/.claude/skills/endpoint-creator/SKILL.md`.

</details>

## Wrap

- AGENTS.md = identità del repo, regole sempre vere, paghi token sempre.
- Skill = know-how procedurale, caricato on-demand quando il `description` matcha il task.
- Triage: la regola serve "sempre"? => AGENTS.md. Serve "a volte"? => Skill.

## Problemi?

Se ti blocchi: `solution/.github/skills/endpoint-creator/SKILL.md` oppure 🔵 `solution/.claude/skills/endpoint-creator/SKILL.md` contengono la versione di riferimento da copiare nella root del repo.

➡️ Prossimo modulo: [`../M2-capacita/README.md`](../M2-capacita/README.md)
