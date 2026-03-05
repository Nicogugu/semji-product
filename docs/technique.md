# Details techniques — Plugin Semji Product

## Architecture MCP

### Harvestr MCP (local, FastMCP)
- Code : `.harvestr-mcp/src/harvestr_mcp/server.py`
- Dependances : `mcp[cli]>=1.0.0`, `httpx>=0.27.0`, `pydantic>=2.0.0`
- Lancement : `uv run --directory .harvestr-mcp fastmcp run src/harvestr_mcp/server.py:mcp`
- API : `https://rest.harvestr.io/v1/`
- Reponse messages : `{ code, messageStatus, pageInfos: { next_offset, per_page }, messages: [...] }`
- Reponse discoveries : meme structure avec `discoveries`
- Pagination : `limit` (max 100, retourne toujours 25 par page), `offset`
- Filtres messages API : `created_before`, `created_after`, `updated_before`, `updated_after`, `channel`, `requesterId`, `hasFeedback`, `customInboxId`
- Channels valides : CHROME_WIDGET, FORM, FRESHDESK, HUBSPOT, INTERCOM, MAIL, NOTE, SALESFORCE, SHEET, SLACK, ZENDESK, MS_TEAMS, PUBLIC_API, HFC, AIRCALL, CLAAP, GONG, MODJO, PRAIZ

### n8n-feedbacks MCP (Streamable HTTP)
- Endpoint : URL dans `.mcp.json` (fait office de token, gitignored)
- Protocole : JSON-RPC 2.0 over SSE (text/event-stream)
- Session : header `mcp-session-id` retourne a l'initialize
- Outils :
  1. `Execute_a_SQL_query_in_Postgres` — param `postgr_sql_query` (string SQL)
  2. `RAG_classique_50_items_` — param `input` (string recherche)
  3. `RAG_100_items_Rerank` — param `input` (string recherche)

### Base Supabase PostgreSQL
- Table `documents` — schema reel :
  - `id` bigint PK (autoincrement)
  - `content` text — corps du feedback
  - `metadata` jsonb — contient : `label`, `requester`, `via`, `created_at`, `loc`, `source`, `blobType`
  - `embedding` vector — NE JAMAIS selectionner
- Acces aux champs metadata en SQL : `metadata->>'label'`, `(metadata->>'created_at')::date`
- ~1400 documents au 2026-03-05, ~84/jour en moyenne recemment

### Difference entre les deux sources
- Harvestr = feedbacks qualitatifs envoyes via Slack #feedback-semji, valides par le Product
  - Contient des discoveries avec ARR (champ custom)
  - Petit volume (~300 messages + ~25 discoveries)
- n8n RAG/SQL = feedbacks extraits automatiquement des calls clients (Fireflies, Gong, etc.)
  - Plus gros volume (~1400 documents)
  - Non review par les equipes Product
  - Plus de labels automatiques

## Fichiers modifies (v1.2.0)

| Fichier | Action |
|---------|--------|
| `skills/p.feedback/SKILL.md` | Reecrit completement |
| `plugins/semji-product/skills/p.feedback/SKILL.md` | Copie synchronisee |
| `plugins/semji-product/.claude-plugin/plugin.json` | Version 1.1.0 → 1.2.0 |
| `.mcp.json.example` | n8n-feedbacks : stdio → url |
| `plugins/semji-product/.mcp.json.example` | idem |
| `.harvestr-mcp/` (gitignore) | Serveur MCP + filtres date ajoutes |
| `CLAUDE.md` | Doc technique complete |

## Problemes rencontres et solutions
- `uv` pas dans PATH bash → installe via `pip install uv`, binaire a `C:\...\Python314\Scripts\uv.exe`
- Encodage UTF-8 dans terminal bash Windows → `PYTHONIOENCODING=utf-8`
- Schema base documente incorrectement (colonnes dediees vs metadata jsonb) → corrige apres inspection SQL
- `.harvestr-mcp/` gitignore → commit sans ce dossier, setup local pour chaque PM
