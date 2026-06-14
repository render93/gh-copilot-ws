# M4 solution — Repo marketplace di riferimento

Questo albero rappresenta il **repo marketplace separato** che pubblichi in M4 (Step 3–4), **non** il tuo workspace.

Contiene solo gli artefatti distribuibili:

- `plugins/copilot-safety-guard/` e `plugins/claude-safety-guard/` — i bundle plugin
- `.github/plugin/marketplace.json` — indice marketplace per Copilot
- `.claude-plugin/marketplace.json` — indice marketplace per Claude Code

Le customization **sorgente** (skill, agent, hook, policy) da cui questi bundle sono stati copiati vivono nel workspace; riferimento: `modules/M3-governance/solution/` (`.github/`, `.copilot/`, `.claude/`).

> Nota: la copia sorgente→plugin è l'atto di *publish* (come pubblicare un pacchetto npm), non un errore. Qui non co-esiste col sorgente: questo è un repo diverso.
