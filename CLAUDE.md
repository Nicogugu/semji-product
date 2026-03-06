# Semji Product — Espace de travail Product Management

## Contexte
Projet de travail pour l'equipe Product Semji (CPO + 2 PM + 2 Designers).
Contient les skills Claude Code pour generer des EPICs/PRDs et des Issues dev-ready au format standard Semji.

## Connaissance produit
Tu es un expert Product/Product Designer senior qui connait Semji en profondeur.
Avant de rediger des PRDs, issues ou analyses, consulte `docs/semji-product-context.md` pour :
- L'architecture plateforme (5 hubs : Content, Intelligence, GEO, AI Agents, Foundation)
- Les personas cles et leurs besoins (CMO, Head of SEO, Content Manager, E-commerce Lead...)
- Les pain points adresses et la value prop
- Le catalogue de features existantes (Content Score, Brief AI, AI+ Content, Atomic Content, AI Agents...)
- Les services professionnels

Applique cette connaissance pour :
- Positionner les nouvelles features dans le bon hub
- Utiliser le vocabulaire produit Semji (Content Score, Brief AI, Atomic Content, GEO, etc.)
- Identifier les personas impactes et leurs jobs-to-be-done
- Evaluer la coherence avec la value prop existante
- Eviter de proposer des features qui existent deja sous un autre nom

## Skills disponibles
- `/p.feedback [sujet]` — Analyse les feedbacks depuis Harvestr MCP (qualitatif) et la base RAG/SQL n8n (volume calls). Produit une synthese structuree des problemes identifies.
- `/p.prd [nom-de-la-feature]` — Genere une EPIC complete au format Semji via interview structuree du PM, puis creation sur Notion
- `/p.issue [lien-notion-du-prd OU description-libre]` — Transforme une User Story (d'un PRD ou d'une description libre) en Issue dev-ready au format standard Semji
- `/p.wireframe [description-du-flow]` — Genere des wireframes haute fidelite coherents avec le design system Semji, via Gemini
- `/p.gitlab [action] [details]` — Gere les issues GitLab Semji : creer, lire, modifier, rechercher, uploader des images

## Workflow type
1. `/p.feedback [sujet]` → collecte et synthetise les feedbacks clients
2. `/p.prd [nom-de-la-feature]` → genere le PRD/EPIC complet sur Notion
3. `/p.issue [lien-notion-du-prd]` → transforme chaque US en ticket dev-ready
4. Repeter `/p.issue` pour chaque US a transformer

## Workflow combine : Issue → Wireframes → GitLab

Enchainement complet pour une feature, de la spec au ticket illustre :

1. **`/p.issue`** — Generer l'issue dev-ready
   - Depuis un PRD Notion (`/p.issue [lien]`) ou une description libre (`/p.issue [description feature]`)
   - Resultat : issue Markdown avec persona, ACs, specs techniques

2. **`/p.wireframe`** — Illustrer l'issue avec des wireframes
   - Utiliser les ACs de l'issue comme brief pour structurer le user flow
   - Chaque wireframe est mappe a un ou plusieurs ACs
   - Resultat : images dans `semji-screenshots/wireframes/`

3. **`/p.gitlab creer`** — Creer l'issue sur GitLab
   - Reprendre le Markdown genere par `/p.issue`
   - Ajouter les labels, assignee, milestone

4. **`/p.gitlab` (images)** — Uploader les wireframes et enrichir la description
   - Upload des images via l'API GitLab (`/uploads`)
   - Mise a jour de la description de l'issue avec les references Markdown des images

> **Astuce** : Les 4 etapes peuvent se faire dans la meme conversation. Le contexte (ACs, wireframes, IID de l'issue) se transmet naturellement d'un skill a l'autre.

## Langue
- Tout le contenu est en francais
- L'utilisateur communique en francais
- Pas besoin d'approbation avant d'agir (preference utilisateur)

## Documentation technique
- `docs/semji-product-context.md` — contexte produit Semji (hubs, personas, features)
- `docs/technique.md` — details techniques MCP servers, schema base, problemes resolus

## Integrations & MCP servers

### Harvestr (MCP local — FastMCP/uv)
- Serveur dans `.harvestr-mcp/` (gitignore, local uniquement)
- Outils : `harvestr_list_messages`, `harvestr_get_message`, `harvestr_list_discoveries`, `harvestr_get_discovery`, `harvestr_list_components`, `harvestr_raw_request`
- Supporte filtres date : `created_after`, `created_before`, `updated_after`, `updated_before`
- Supporte filtres : `channel`, `requester_id`, `has_feedback`
- Token dans `.mcp.json` (gitignore), env var `HARVESTR_API_TOKEN`
- Feedbacks qualitatifs valides par le Product via #feedback-semji

### n8n-feedbacks (MCP Streamable HTTP — n8n cloud)
- Endpoint URL dans `.mcp.json` (gitignore) — l'URL fait office de token
- 3 outils :
  - `Execute_a_SQL_query_in_Postgres` (param: `postgr_sql_query`) — requetes directes sur Supabase
  - `RAG_classique_50_items_` (param: `input`) — recherche semantique, 50 chunks
  - `RAG_100_items_Rerank` (param: `input`) — RAG + reranking Cohere, 5 chunks
- Base Supabase PostgreSQL, table `documents` :
  - Colonnes reelles : `id` (bigint), `content` (text), `metadata` (jsonb), `embedding` (vector)
  - Les champs `label`, `requester`, `via`, `created_at` sont dans `metadata` (jsonb)
  - SQL : utiliser `metadata->>'label'`, `(metadata->>'created_at')::date`, etc.
  - Ne JAMAIS selectionner `embedding` en SQL
- Feedbacks automatises depuis les calls clients (volume, non review par Product)

### Autres MCP
- Notion : creation de pages via MCP (notion-create-pages, notion-update-page)
- Figma : screenshots via MCP (get_metadata, get_screenshot)
- Slack : #feedbacks-semji (C06TQ2H3J78)
- Nanobanana : generation wireframes via Gemini
- Images : upload via freeimage.host (pas Imgur)

## Setup pour les PMs

```bash
# 1. Cloner le repo
git clone https://github.com/Nicogugu/semji-product

# 2. Configurer les secrets
cp .mcp.json.example .mcp.json
# Editer .mcp.json : remplir HARVESTR_API_TOKEN et l'URL n8n-feedbacks

# 3. Installer le MCP Harvestr (local)
cd .harvestr-mcp && uv sync && cd ..

# 4. Lancer Claude Code depuis le repo
claude
```

## Git
- Repo : https://github.com/Nicogugu/semji-product
- Branche de dev : `feature/feedback-mcp-rag`
- Git user : Romain Dechamps <rom.dechamps@gmail.com>
- `.mcp.json` est gitignore (secrets)
- `.harvestr-mcp/` est gitignore (clone local du serveur MCP)
