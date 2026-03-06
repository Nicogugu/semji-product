# Details techniques — Plugin Semji Product

## Architecture MCP

### Harvestr MCP (Streamable HTTP — n8n cloud)
- Endpoint : URL dans `.mcp.json` (fait office de token, gitignored)
- Protocole : JSON-RPC 2.0 over SSE (text/event-stream)
- Backend : workflow n8n qui proxifie l'API Harvestr `https://rest.harvestr.io/v1/`
- Outils :
  1. `List_Messages` — params : `limit`, `offset`, `channel`, `requester_id`, `has_feedback`, `created_after`, `created_before`, `updated_after`, `updated_before`
  2. `Get_Message` — param : `message_id`
  3. `List_Discoveries` — params : `limit`, `offset`, `component_id`, `created_after`, `created_before`, `updated_after`, `updated_before`
  4. `Get_Discovery` — param : `discovery_id`
  5. `List_Components` — params : `limit`, `offset`
  6. `Raw_Request` — params : `method`, `endpoint`
- Pagination : `limit` (max 100, retourne toujours 25 par page), `offset`
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
| `plugins/semji-product/skills/p.feedback/SKILL.md` | Reecrit completement (RAG + SQL + Harvestr) |
| `plugins/semji-product/.claude-plugin/plugin.json` | Version 1.1.0 → 1.2.0 |
| `.mcp.json.example` | n8n-feedbacks : stdio → url |
| `.mcp.json` | Harvestr : local FastMCP → URL n8n cloud |
| `.gitignore` | Supprime `.harvestr-mcp/` (dossier supprime) |
| `CLAUDE.md` | Doc technique complete |

## Fichiers modifies (v1.3.0)

| Fichier | Action |
|---------|--------|
| Structure | Supprime `skills/` (racine) et `.claude/skills/` : doublons du plugin. Source unique = `plugins/semji-product/skills/` |
| `.claude/settings.local.json` | Supprime (permissions stale, pas necessaire pour le plugin) |
| `plugins/semji-product/docs/` | Supprime (orphelin, non reference par les skills) |
| `plugins/semji-product/.mcp.json.example` | Supprime (doublon du `.mcp.json.example` racine) |
| `semji-product-context.md` | Co-localise dans les 5 skills via `${CLAUDE_SKILL_DIR}/semji-product-context.md`. Copie master dans `docs/` |
| Tous les SKILL.md | Ajout reference `${CLAUDE_SKILL_DIR}/semji-product-context.md` pour acces au contexte produit |
| `plugins/semji-product/.claude-plugin/plugin.json` | Version 1.2.0 → 1.3.0 |

## Problemes rencontres et solutions
- Schema base documente incorrectement (colonnes dediees vs metadata jsonb) → corrige apres inspection SQL
- Migration Harvestr local → n8n cloud : supprime la dependance a Python/uv, simplifie le setup pour les PMs
- Structure 3 copies (root skills/, .claude/skills/, plugin skills/) → nettoyee en source unique `plugins/semji-product/skills/`
