# M4 De-duplicazione via repo-marketplace separato — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Trasformare `modules/M4-distribuzione/solution/` nella fotografia di un repo-marketplace separato (solo plugin + indici), eliminando la co-locazione sorgente↔plugin, e allineare i doc.

**Architecture:** Refactor di sola documentazione/struttura. Nessun codice applicativo cambia. Si rimuovono da `M4/solution/` i sorgenti (customization root) e l'app code, lasciando i due bundle plugin e i due `marketplace.json`; si riallinea il README di M4 al modello "repo separato"; si aggiornano `AGENTS.md` e il design doc; si lascia una nota di rimando sul vecchio plan doc.

**Tech Stack:** Markdown, JSON, struttura di filesystem, git. Verifiche via `find`/`grep`.

**Spec:** `docs/superpowers/specs/2026-06-14-m4-dedup-marketplace-repo-design.md`

**Branch:** `m4-dedup-marketplace-repo` (già creato e con lo spec committato).

---

### Task 1: Restructure `M4/solution/` nel repo-marketplace

**Files:**
- Delete: `modules/M4-distribuzione/solution/.github/skills/`
- Delete: `modules/M4-distribuzione/solution/.github/agents/`
- Delete: `modules/M4-distribuzione/solution/.github/hooks/`
- Delete: `modules/M4-distribuzione/solution/.copilot/`
- Delete: `modules/M4-distribuzione/solution/.claude/` (NON `.claude-plugin/`)
- Delete: `modules/M4-distribuzione/solution/dotnet/`, `.../python/`, `.../typescript/`
- Keep: `modules/M4-distribuzione/solution/plugins/`, `.../.github/plugin/marketplace.json`, `.../.claude-plugin/marketplace.json`
- Create: `modules/M4-distribuzione/solution/README.md`

- [ ] **Step 1: Snapshot dello stato attuale (per confronto)**

Run:
```bash
find modules/M4-distribuzione/solution -type f -not -path '*/node_modules/*' | sort
```
Expected: elenco che include `.copilot/...`, `.claude/...`, `.github/{skills,agents,hooks}/...`, `dotnet/python/typescript/...`, oltre a `plugins/...` e i due `marketplace.json`. (Conferma che i sorgenti sono ancora co-locati.)

- [ ] **Step 2: Rimuovi sorgenti e app code dal disco**

Run:
```bash
rm -rf modules/M4-distribuzione/solution/.github/skills \
       modules/M4-distribuzione/solution/.github/agents \
       modules/M4-distribuzione/solution/.github/hooks \
       modules/M4-distribuzione/solution/.copilot \
       modules/M4-distribuzione/solution/.claude \
       modules/M4-distribuzione/solution/dotnet \
       modules/M4-distribuzione/solution/python \
       modules/M4-distribuzione/solution/typescript
```

- [ ] **Step 3: Crea il README di cartella**

Write to `modules/M4-distribuzione/solution/README.md`:
```markdown
# M4 solution — Repo marketplace di riferimento

Questo albero rappresenta il **repo marketplace separato** che pubblichi in M4 (Step 3–4), **non** il tuo workspace.

Contiene solo gli artefatti distribuibili:

- `plugins/copilot-safety-guard/` e `plugins/claude-safety-guard/` — i bundle plugin
- `.github/plugin/marketplace.json` — indice marketplace per Copilot
- `.claude-plugin/marketplace.json` — indice marketplace per Claude Code

Le customization **sorgente** (skill, agent, hook, policy) da cui questi bundle sono stati copiati vivono nel workspace; riferimento: `modules/M3-governance/solution/` (`.github/`, `.copilot/`, `.claude/`).

> Nota: la copia sorgente→plugin è l'atto di *publish* (come pubblicare un pacchetto npm), non un errore. Qui non co-esiste col sorgente: questo è un repo diverso.
```

- [ ] **Step 4: Stage delle modifiche (cancellazioni + nuovo file)**

Run:
```bash
git add -A modules/M4-distribuzione/solution/
```

- [ ] **Step 5: Verifica struttura finale**

Run:
```bash
find modules/M4-distribuzione/solution -type f -not -path '*/node_modules/*' | sort
```
Expected: SOLO questi path (più i file interni ai bundle plugin):
- `modules/M4-distribuzione/solution/README.md`
- `modules/M4-distribuzione/solution/.claude-plugin/marketplace.json`
- `modules/M4-distribuzione/solution/.github/plugin/marketplace.json`
- `modules/M4-distribuzione/solution/plugins/claude-safety-guard/...`
- `modules/M4-distribuzione/solution/plugins/copilot-safety-guard/...`

Nessun `dotnet/`, `python/`, `typescript/`, `.copilot/`, `.claude/`, `.github/skills`, `.github/agents`, `.github/hooks`.

- [ ] **Step 6: Verifica no-duplicazione**

Run:
```bash
find modules/M4-distribuzione/solution -name policy.yml -not -path '*/node_modules/*'
find modules/M4-distribuzione/solution -name 'SKILL.md' -not -path '*/node_modules/*'
```
Expected:
- `policy.yml` → solo `plugins/copilot-safety-guard/policy.yml` e `plugins/claude-safety-guard/policy.yml`
- `SKILL.md` → solo i due dentro `plugins/.../skills/endpoint-creator/SKILL.md`

(Nessuna copia fuori da `plugins/`.)

- [ ] **Step 7: Commit**

```bash
git commit -m "$(cat <<'EOF'
refactor(M4): solution/ become standalone marketplace repo

Remove co-located source customizations (.github/{skills,agents,hooks},
.copilot/, .claude/) and app code (dotnet/python/typescript) from
M4/solution/. Keep only the distributable artifacts: plugin bundles and
the two marketplace.json indices. Source lives in M3 solution.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: Reframe degli Step 3–4 nel README di M4

**Files:**
- Modify: `modules/M4-distribuzione/README.md`

- [ ] **Step 1: Leggi il README corrente**

Run: `Read modules/M4-distribuzione/README.md` per individuare le sezioni `### Step 3 - Crea il tuo plugin` e `### Step 4 - Pubblica il tuo plugin`.

- [ ] **Step 2: Sostituisci la sezione Step 3**

Sostituisci il blocco che va dall'heading `### Step 3 - Crea il tuo plugin` fino a (escluso) `### Step 4 - ...`, comprese le righe iniziali e l'albero, con il testo seguente. **Mantieni invariati** i due blocchi esplicativi già presenti su `plugin.json` (il JSON del manifest + il paragrafo "Il manifest quindi non punta...") e su `hooks.json` (il JSON + paragrafo), e la nota su `policy.yml`: vanno riposizionati dopo l'albero qui sotto, senza modifiche al loro contenuto. Riallinea solo l'heading, l'intro, il box e l'albero:

````markdown
### Step 3 - Crea il tuo repo-marketplace e assembla il plugin

> **Workspace vs marketplace.** Il tuo workspace (le customization costruite in M1–M3: `.github/skills`, `.github/agents`, `.copilot/`) è il **sorgente**. Il marketplace è un **repo separato** che ne pubblica una copia curata. La copia è l'atto di *publish*, non un link — come quando pubblichi un pacchetto npm: il pacchetto è uno snapshot del tuo sorgente, in un posto diverso. Per questo qui crei un repo nuovo invece di mettere il plugin accanto al sorgente.

Crea un **nuovo repository** (separato dal workspace del workshop) e al suo interno assembla il plugin `copilot-safety-guard`, copiando il sottoinsieme curato dal tuo workspace:

```
<nuovo-repo-marketplace>/
├── .github/plugin/marketplace.json         (indice del marketplace - aggiunto allo Step 4)
└── plugins/
    └── copilot-safety-guard/
        ├── plugin.json                      (manifest del bundle)
        ├── hooks.json                       (registra l'hook preToolUse => script)
        ├── policy.yml                       (regole di blocco - copiata dal workspace: .copilot/policy.yml)
        ├── skills/
        │   └── endpoint-creator/SKILL.md    (copiata dal workspace: .github/skills/endpoint-creator/SKILL.md)
        ├── agents/
        │   └── code-reviewer.agent.md       (copiato dal workspace: .github/agents/code-reviewer.agent.md)
        └── scripts/
            ├── pre-tool-use.sh              (copiato dal workspace: .copilot/hooks/pre-tool-use.sh)
            └── pre-tool-use.ps1             (copiato dal workspace: .copilot/hooks/pre-tool-use.ps1)
```
````

(Dopo questo albero seguono, invariati, i blocchi esistenti su `plugin.json`, `hooks.json` e la nota `policy.yml`.)

- [ ] **Step 3: Sostituisci la sezione Step 4**

Sostituisci il blocco dall'heading `### Step 4 - Pubblica il tuo plugin` fino a (escluso) `## Wrap`, mantenendo invariato il JSON di esempio di `marketplace.json` e il paragrafo che spiega `pluginRoot`/`source`, ma riallineando heading, lista di pubblicazione e frase di chiusura così:

````markdown
### Step 4 - Pubblica e installa

Il tuo repo-marketplace diventa installabile aggiungendo l'indice `.github/plugin/marketplace.json` che elenca i plugin del repo:

```json
{
  "name": "my-workshop-marketplace",
  "owner": { "name": "Your Name", "email": "you@example.com" },
  "metadata": {
    "description": "Il mio marketplace del workshop",
    "version": "0.1.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "copilot-safety-guard",
      "description": "endpoint-creator skill + code-reviewer subagent + safety preToolUse hook.",
      "version": "0.1.0",
      "source": "copilot-safety-guard"
    }
  ]
}
```

`pluginRoot` indica la cartella che contiene i plugin (`./plugins`); `source` è il nome della sottocartella del plugin sotto `pluginRoot`. Poi:

1. `git init && git add . && git commit && git push` del repo-marketplace su un nuovo repository GitHub (con `plugins/copilot-safety-guard/` e `.github/plugin/marketplace.json` al root del repo).
2. Aggiungi l'URL del tuo repo a `chat.plugins.marketplaces`.
3. `@agentPlugins` => compare `copilot-safety-guard` => **Install**.

Questo chiude il cerchio: la copia che hai fatto vive in un **repo separato** dal workspace (niente sorgente e plugin nello stesso albero), e da lì è installabile esattamente come `dev-guardian` nello Step 1.
````

- [ ] **Step 4: Riallinea i blocchi 🔵 Claude Code degli Step 3–4**

Nel `<details>` 🔵 dello Step 3, cambia l'incipit "Crea `plugins/claude-safety-guard/` al root del workspace:" in "Crea `plugins/claude-safety-guard/` nel tuo repo-marketplace separato (stesso repo dello Step 3 Copilot, struttura Claude):" e nell'albero sostituisci ogni nota "copiata da `.claude/...`" con "copiata dal workspace: `.claude/...`". Lascia invariati i JSON di `plugin.json` e `hooks/hooks.json`. Nel `<details>` 🔵 dello Step 4 lascia il contenuto com'è (descrive già `.claude-plugin/marketplace.json` e "crea un nuovo repository Git"): verifica solo che la frase di chiusura non implichi co-locazione col workspace.

- [ ] **Step 5: Verifica che non resti testo di co-locazione errato**

Run:
```bash
grep -n "Crea un nuovo repository con questa struttura" modules/M4-distribuzione/README.md
grep -n "plugins/copilot-safety-guard/$" modules/M4-distribuzione/README.md
grep -n "copiata da \.copilot/policy.yml" modules/M4-distribuzione/README.md
```
Expected: nessun match per le prime due (testo vecchio rimosso). La terza può non avere match (è stata riformulata in "copiata dal workspace: .copilot/policy.yml").

- [ ] **Step 6: Verifica che i puntatori "Problemi?" risolvano**

Run:
```bash
ls modules/M4-distribuzione/solution/plugins/copilot-safety-guard/ \
   modules/M4-distribuzione/solution/.github/plugin/marketplace.json
```
Expected: entrambi esistono (la sezione "Problemi?" del README continua a puntare a path validi).

- [ ] **Step 7: Commit**

```bash
git add modules/M4-distribuzione/README.md
git commit -m "$(cat <<'EOF'
docs(M4): reframe README steps 3-4 around a separate marketplace repo

Step 3 now creates the plugin inside a separate marketplace repo (not
co-located with workspace source); add workspace-vs-marketplace box;
tree notes point to the workspace as source. Step 4 = publish & install.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: Aggiorna `AGENTS.md`

**Files:**
- Modify: `AGENTS.md`

- [ ] **Step 1: Aggiorna la riga M4 nell'elenco delle customizations**

Trova la riga:
```
- `plugins/copilot-safety-guard/` (creato in M4 - impacchetta un sottoinsieme curato: skill + code-reviewer + safety hook)
```
e sostituiscila con:
```
- `plugins/copilot-safety-guard/` + `.github/plugin/marketplace.json` (creati in M4 - il plugin impacchetta un sottoinsieme curato (skill + code-reviewer + safety hook) e viene pubblicato in un **repo marketplace separato**, non nel workspace)
```

- [ ] **Step 2: Aggiorna "Dove trovare cosa"**

Trova la riga:
```
- `modules/Mn/solution/.github/`, `modules/Mn/solution/.copilot/` - customizations cumulative (skill, agent, hook, policy) come reference da copiare al root del repo se ci si blocca.
```
e aggiungi **subito sotto** una nuova riga:
```
- `modules/M4-distribuzione/solution/` - eccezione: è la fotografia di un **repo marketplace separato** (solo `plugins/` + i due `marketplace.json`), non un workspace. I sorgenti dei bundle M4 sono in `modules/M3-governance/solution/`.
```

- [ ] **Step 3: Verifica**

Run:
```bash
grep -n "repo marketplace separato" AGENTS.md
```
Expected: due match (riga M4 + "Dove trovare cosa").

- [ ] **Step 4: Commit**

```bash
git add AGENTS.md
git commit -m "$(cat <<'EOF'
docs: AGENTS.md describes M4 solution as a separate marketplace repo

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: Aggiorna la sezione M4 del design doc del workshop

**Files:**
- Modify: `docs/superpowers/specs/2026-05-24-workshop-agent-strikes-back-design.md` (sezione "M4 — Distribuzione", righe ~234–266)

- [ ] **Step 1: Aggiorna le righe sullo "Step 2 — Impacchettare il proprio plugin"**

Trova la riga (~259):
```
- Prendere skill (M1) + subagent (M2) + hook safety (M3) e impacchettarli come **plugin locale** in `plugins/copilot-safety-guard/`.
```
e sostituiscila con:
```
- Prendere skill (M1) + subagent (M2) + hook safety (M3) e impacchettarli come plugin `copilot-safety-guard` **in un repo marketplace separato** (la copia sorgente→plugin è l'atto di publish; sorgente nel workspace, bundle nel repo marketplace).
```

- [ ] **Step 2: Aggiorna la riga "Output portabile"**

Trova la riga (~266):
```
**Output portabile**: `plugins/copilot-safety-guard/` con manifest e tutti i componenti, pronto in teoria da pubblicare.
```
e sostituiscila con:
```
**Output portabile**: un repo marketplace separato con `plugins/copilot-safety-guard/` (manifest + componenti) e `.github/plugin/marketplace.json`, installabile come quello dello Step 1.
```

- [ ] **Step 3: Verifica**

Run:
```bash
grep -n "plugin locale" docs/superpowers/specs/2026-05-24-workshop-agent-strikes-back-design.md
```
Expected: nessun match.

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/specs/2026-05-24-workshop-agent-strikes-back-design.md
git commit -m "$(cat <<'EOF'
docs: align workshop design doc M4 section with separate-repo model

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 5: Nota di rimando sul plan doc storico + verifica README principale

**Files:**
- Modify: `docs/superpowers/plans/2026-05-24-workshop-agent-strikes-back.md` (Phase 6, riga ~1920)
- Verify (no change attesa): `README.md`

- [ ] **Step 1: Aggiungi nota di rimando alla Phase 6 del plan doc**

Trova l'heading (~1920):
```
## Phase 6 — Module M4 "Distribuzione" (Plugins & Marketplace)
```
e inserisci **subito sotto** la riga:
```
> ⚠️ Struttura superata: M4 è stato refactorato in `docs/superpowers/specs/2026-06-14-m4-dedup-marketplace-repo-design.md` (la solution di M4 è ora un repo marketplace separato, senza app code né customization co-locate). I task qui sotto riflettono il build originale.
```

- [ ] **Step 2: Verifica il README principale**

Run:
```bash
grep -n -iE 'plugin|marketplace' README.md
```
Expected: i match riguardano l'installazione di customizations/plugin al root del workspace (es. righe ~151, 153) — concetti ancora corretti per M1–M3. Nessuna affermazione che dica che il plugin M4 vive nello stesso albero della solution. **Se** trovi una frase che contraddice il modello a repo separato, correggila in linea; altrimenti nessuna modifica.

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/plans/2026-05-24-workshop-agent-strikes-back.md README.md
git commit -m "$(cat <<'EOF'
docs: mark historical M4 build plan as superseded; verify main README

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

(Se il README principale non è cambiato, `git add README.md` è un no-op innocuo.)

---

### Task 6: Verifica finale contro i criteri di accettazione

**Files:** nessuno (solo verifica).

- [ ] **Step 1: Criterio 1 — nessun file di customization duplicato in M4/solution**

Run:
```bash
find modules/M4-distribuzione/solution -not -path '*/node_modules/*' \
  \( -name 'policy.yml' -o -name 'SKILL.md' -o -name 'code-reviewer.*' -o -name 'pre-tool-use.*' \) | sort
```
Expected: ogni risultato è **dentro** `plugins/...`. Nessuna copia al di fuori.

- [ ] **Step 2: Criterio 2 — struttura della solution**

Run:
```bash
ls -A modules/M4-distribuzione/solution
```
Expected: solo `.claude-plugin`, `.github`, `plugins`, `README.md`. (Nessun `.copilot`, `.claude`, `dotnet`, `python`, `typescript`.)

- [ ] **Step 3: Criterio 4 — nessun riferimento residuo alla vecchia struttura**

Run:
```bash
grep -rn -iE 'plugin locale|Crea un nuovo repository con questa struttura' \
  AGENTS.md README.md modules/M4-distribuzione/README.md \
  docs/superpowers/specs/2026-05-24-workshop-agent-strikes-back-design.md
```
Expected: nessun match.

- [ ] **Step 4: Criterio 5 — i puntatori "se ti blocchi" risolvono**

Run:
```bash
test -d modules/M4-distribuzione/solution/plugins/copilot-safety-guard \
  && test -f modules/M4-distribuzione/solution/.github/plugin/marketplace.json \
  && echo OK
```
Expected: `OK`.

- [ ] **Step 5: Report finale**

Riassumi: criteri 1–5 verificati, elenco commit del branch (`git log --oneline main..HEAD`).

---

## Self-Review

**Spec coverage:**
- §Design 1 (restructure solution) → Task 1. ✓
- §Design 2 (README flow Step 3–4 + box + albero + 🔵) → Task 2. ✓
- §Design 3 riga AGENTS.md → Task 3; design doc → Task 4; plan doc → Task 5; main README → Task 5 Step 2. ✓
- §Criteri di accettazione 1–5 → Task 6. ✓

**Placeholder scan:** Nessun TBD/TODO; ogni edit riporta il testo nuovo verbatim o l'istruzione esatta di sostituzione con anchor citato. ✓

**Type/anchor consistency:** I nomi dei path e gli anchor (`### Step 3 - Crea il tuo plugin`, `plugin locale`, `Crea un nuovo repository con questa struttura`) sono usati coerentemente tra i task di edit (Task 2–4) e i grep di verifica (Task 5–6). ✓

**Nota residuo:** i due `policy.yml` dual-runtime in `plugins/{copilot,claude}-safety-guard/` restano per design (spec §Residuo accettato) — non sono un fallimento del criterio 1, che riguarda copie **fuori** da `plugins/`.
