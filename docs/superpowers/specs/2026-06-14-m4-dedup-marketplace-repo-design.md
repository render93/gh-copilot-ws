# M4 — De-duplicazione via repo-marketplace separato

> Spec di refactoring del modulo M4 "Distribuzione". Data: 2026-06-14.
> Non cambia i concetti insegnati (plugin + marketplace): cambia **dove** vivono gli artefatti per eliminare la duplicazione sorgente↔plugin.

## Problema

La `solution/` di M4 oggi co-loca, nello stesso albero, tre cose:

1. le **customization sorgente** al root (`.github/{skills,agents,hooks}`, `.copilot/{hooks,context,policy.yml}`, `.claude/{skills,agents,hooks,policy.yml,settings.json}`) — byte-identiche allo stato finale di M3;
2. la loro **copia** dentro `plugins/copilot-safety-guard/` e `plugins/claude-safety-guard/`;
3. gli **indici marketplace** (`.github/plugin/marketplace.json`, `.claude-plugin/marketplace.json`);

più il **codice app** (`dotnet/`, `python/`, `typescript/`) che a M4 non serve.

Conseguenza visibile: file come `policy.yml` (e ogni skill/agent/hook) compaiono **due volte** nello stesso albero. Per un partecipante è confondente: non si capisce quale sia la fonte di verità, e sembra un errore invece che un atto di packaging.

## Causa

La duplicazione di **contenuto** tra "le mie customization" e "il mio plugin pubblicato" è **inerente al modello plugin**: un plugin pubblicato è uno snapshot self-contained delle customization (come un pacchetto npm pubblicato duplica il sorgente). Non è eliminabile a livello di contenuto senza introdurre un build-step / single-source-of-truth.

Il problema reale quindi **non è la copia**, ma la **co-locazione**: sorgente e artefatto pubblicato vivono nello stesso albero/repo, mentre nella realtà stanno in **repo distinti** (il repo di progetto vs. il repo marketplace).

## Decisione

**Separazione fisica.** La copia sorgente→plugin è ammessa perché è l'atto di publish; si elimina la **co-locazione**. M4 produce e mostra il marketplace come **repo separato**. La `solution/` di M4 smette di tenere root-customizations + app code accanto al plugin.

Scartata (consapevolmente) l'alternativa "single source of truth + build script" (plugin generato da uno script di package, cartella plugin come artefatto): più corretta da ingegneri ma aggiunge uno step di build al workshop di 90 min e cambia il flusso hands-on. Fuori scope per questo refactor.

## Design

### 1. `modules/M4-distribuzione/solution/` diventa SOLO il repo-marketplace

La `solution/` di M4 non rappresenta più "workspace + plugin", ma **la fotografia del repo marketplace separato del partecipante**.

**Prima:**

```
M4/solution/
├── .github/{skills,agents,hooks}/…        ← sorgente (copia di M3)
├── .copilot/{hooks,context,policy.yml}    ← sorgente (copia di M3)
├── .claude/{skills,agents,hooks,policy…}  ← sorgente (copia di M3)
├── dotnet/ python/ typescript/            ← app code (copia di M3)
├── .github/plugin/marketplace.json
├── .claude-plugin/marketplace.json
└── plugins/{copilot,claude}-safety-guard/ ← copia degli stessi sorgenti
```

**Dopo:**

```
M4/solution/                               ← "questo albero = il TUO repo marketplace"
├── README.md                              (1 riga: cos'è questo repo)
├── .github/plugin/marketplace.json        (indice Copilot)
├── .claude-plugin/marketplace.json        (indice Claude)
└── plugins/
    ├── copilot-safety-guard/              (bundle completo, invariato)
    └── claude-safety-guard/               (bundle completo, invariato)
```

**Rimosso da M4/solution/:**
- `.github/skills/`, `.github/agents/`, `.github/hooks/`
- `.copilot/` (intera: `context/`, `hooks/`, `policy.yml`)
- customization `.claude/`: `agents/`, `context/`, `hooks/`, `policy.yml`, `settings.json`, `skills/`
- `dotnet/`, `python/`, `typescript/`

**Mantenuto:**
- `plugins/copilot-safety-guard/` e `plugins/claude-safety-guard/` (i bundle, invariati nel contenuto)
- `.github/plugin/marketplace.json` (indice Copilot)
- `.claude-plugin/marketplace.json` (indice Claude — attenzione: `.claude-plugin/` ≠ `.claude/`, è l'indice, non le customization)
- nuovo `README.md` di cartella

**Effetto:** dentro l'albero di M4 non esiste più una copia delle customization accanto al plugin. `policy.yml` compare una sola volta per bundle. La sorgente da cui copiare è **M3 solution** (o il workspace del partecipante). La copia sorgente→plugin esiste ancora ma **tra due repo diversi** (M3 = workspace, M4 = marketplace): è la separazione fisica voluta.

**Residuo accettato:** `copilot-safety-guard/policy.yml` e `claude-safety-guard/policy.yml` restano due file con contenuto uguale, ma sono due bundle per due runtime diversi (Copilot / Claude) nello stesso marketplace — natura dual-runtime del repo, non la duplicazione confusa di prima. Non si interviene.

### 2. Flusso hands-on del README di M4

Step 1–2 (installa + ispeziona un plugin reale dal marketplace `render93/gh-copilot-dev-days-2026`) **invariati**.

Step 3–4 riallineati alla separazione fisica:

| | Oggi | Dopo |
|---|---|---|
| **Step 3** | "Crea il tuo plugin" → `plugins/copilot-safety-guard/` (ambiguo: al root del workspace → sorgente + plugin nello stesso albero) | **"Crea il tuo repo-marketplace e assembla il plugin"** → repo/cartella **nuova, fuori dal workspace**; dentro `plugins/copilot-safety-guard/` + `marketplace.json`, copiando il sottoinsieme curato **dal workspace** |
| **Step 4** | "Pubblica" | **"Pubblica e installa"** → push su GitHub, registra l'URL in `chat.plugins.marketplaces`, `@agentPlugins` → Install |

Aggiunte al README:
- **Box concettuale:** *"Il tuo workspace (M1–M3) è il sorgente. Il marketplace è un repo separato che ne pubblica una copia curata. La copia è l'atto di publish, non un link — come pubblicare un pacchetto npm."*
- **Albero Step 3 aggiornato:** le note "copiata da `.copilot/policy.yml`" diventano "copiata dal tuo workspace (riferimento: M3 solution)", perché M4 solution non contiene più quei sorgenti.
- I blocchi `🔵 Claude Code` degli Step 3–4 riallineati allo stesso modello (repo separato, indice `.claude-plugin/marketplace.json`).
- Sezione "Problemi?": i puntatori `solution/plugins/...` e `solution/.github/plugin/marketplace.json` restano validi (verificato).

### 3. Allineamento file collegati

| File | Intervento |
|---|---|
| `modules/M4-distribuzione/solution/` | Restructure §1 (rimozione sorgenti + app code; aggiunta `README.md`) |
| `modules/M4-distribuzione/README.md` | Reframe Step 3–4 + box + albero + blocchi 🔵 (§2) |
| `AGENTS.md` | Riga M4 e "Dove trovare cosa": M4 solution = repo marketplace (plugin + indici); i sorgenti vivono in M1–M3 solution; esplicitare il modello "repo separato" |
| `docs/superpowers/specs/2026-05-24-…-design.md` | Sezione M4 (righe ~234–266): da "plugin locale in `plugins/…`" a "repo marketplace separato" |
| `docs/superpowers/plans/2026-05-24-…md` | Grep dei passi M4; allineare dove descrivono la vecchia struttura |
| `README.md` (main) | Solo verifica; modifica solo se contraddice il modello a repo separato (atteso: minimo/nullo) |

## Fuori scope

- La duplicazione **cross-modulo** (ogni `Mn/solution/` ricontiene lo stato cumulativo dei moduli precedenti). È un pattern preesistente del workshop, non l'oggetto di questo refactor.
- Build-step / single-source-of-truth per generare i plugin (alternativa scartata).
- Modifiche al contenuto dei bundle plugin (skill, agent, hook, policy) — restano invariati.
- Il marketplace esterno `render93/gh-copilot-dev-days-2026` (showcase plugin di Step 1) — resta com'è.

## Criteri di accettazione

1. In `M4/solution/` nessun file di customization compare due volte: `policy.yml`, `SKILL.md`, `code-reviewer.*`, hook script esistono **solo** dentro `plugins/...`.
2. `M4/solution/` non contiene `.copilot/`, né le customization `.claude/`, né `.github/{skills,agents,hooks}`, né app code; contiene `plugins/`, i due `marketplace.json` e un `README.md`.
3. Il README di M4 presenta gli Step 3–4 come "crea repo-marketplace separato → pubblica", con il box concettuale e l'albero che punta a M3 solution come sorgente.
4. `AGENTS.md`, design doc e plan doc descrivono M4 in modo coerente con la nuova struttura; nessun riferimento residuo a "plugin co-locato al root del workspace" o a `M4/solution` come workspace completo.
5. I puntatori "se ti blocchi" del README risolvono a path esistenti.
