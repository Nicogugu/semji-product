# Semji Product Tools

Plugin Claude Code pour l'equipe Product Semji. 5 skills pour accelerer le workflow PM :

| Skill | Ce que ca fait |
|-------|---------------|
| `/p.feedback [sujet]` | Analyse les feedbacks depuis Harvestr (qualitatif) et la base de calls (RAG/SQL). Produit une synthese structuree des problemes. |
| `/p.prd [feature]` | Guide une interview PM et genere une EPIC complete sur Notion |
| `/p.issue [lien-notion]` | Transforme une User Story du PRD en issue dev-ready |
| `/p.wireframe [description]` | Genere des wireframes a partir des screenshots Semji |
| `/p.gitlab [action]` | Cree, lit, modifie et recherche des issues sur le GitLab Semji |

---

## Installation

### 1. Prerequis

- **Claude Code** : https://docs.anthropic.com/en/docs/claude-code/overview

Verifier :
```bash
claude --version
```

### 2. Ajouter le plugin

```bash
claude marketplace add https://github.com/Nicogugu/semji-product.git
claude plugin install semji-product
```

### 3. Configurer les secrets

Le plugin utilise plusieurs services qui necessitent des tokens. Aucun secret n'est commite dans le repo.

**Etape 1** : Copier le fichier exemple
```bash
cp .mcp.json.example .mcp.json
```

**Etape 2** : Remplir les tokens dans `.mcp.json`

| Champ | Ou le trouver | Utilise par |
|-------|--------------|-------------|
| URL `harvestr` | Demander a Nico | `/p.feedback` (feedbacks qualitatifs Harvestr) |
| URL `n8n-feedbacks` | Demander a Nico | `/p.feedback` (feedbacks calls RAG/SQL) |
| `GEMINI_API_KEY` | https://aistudio.google.com/apikey | `/p.wireframe` |
| `GITLAB_TOKEN` | https://gitlab.rvip.fr/-/user_settings/personal_access_tokens (scope `api`) | `/p.gitlab` |

> **Important** : Le fichier `.mcp.json` contient des secrets. Ne jamais le commiter (il est dans `.gitignore`).

### 4. Dependance Python (wireframes uniquement)

Necessaire uniquement pour `/p.wireframe` :
```bash
pip install google-genai
```

---

## Utilisation

Lancer Claude Code dans n'importe quel projet :
```
claude
```

Puis taper la commande du skill voulu :

### Analyser les feedbacks
```
/semji-product:p.feedback content score
/semji-product:p.feedback GEO
```
> Combine automatiquement les feedbacks Harvestr (valides Product) et les feedbacks de calls (volume RAG/SQL).

### Creer un PRD
```
/semji-product:p.prd Webhook v2
```

### Creer une issue depuis un PRD
```
/semji-product:p.issue https://notion.so/semji/mon-prd-123
```

### Generer des wireframes
```
/semji-product:p.wireframe modale d'ajout de prompts GEO depuis le listing
```

> Les wireframes utilisent les screenshots dans le dossier `semji-screenshots/` comme reference visuelle. Ajouter des screenshots de l'app Semji dans ce dossier pour de meilleurs resultats.

### Gerer les issues GitLab
```
/semji-product:p.gitlab chercher les issues Bug ouvertes
/semji-product:p.gitlab lire #11529
/semji-product:p.gitlab creer une issue pour le bug du lien casse
```

---

## Workflow type

```
/p.feedback [sujet]     →  comprendre les besoins clients
        ↓
/p.prd [feature]        →  creer le PRD sur Notion
        ↓
/p.wireframe [ecran]    →  generer les maquettes
        ↓
/p.issue [lien-notion]  →  creer les tickets dev (repeter par US)
        ↓
/p.gitlab creer [issue] →  pousser les issues sur GitLab
```

---

## Architecture MCP

Le plugin `/p.feedback` utilise 2 sources de donnees via MCP :

| Source | Type | Donnees | Config |
|--------|------|---------|--------|
| **Harvestr** | MCP Streamable HTTP (n8n cloud) | Feedbacks qualitatifs valides par le Product via #feedback-semji | URL endpoint dans `.mcp.json` |
| **n8n-feedbacks** | MCP Streamable HTTP (n8n cloud) | Feedbacks extraits des calls clients (RAG + SQL sur Supabase) | URL endpoint dans `.mcp.json` |

Pour plus de details techniques : voir `docs/technique.md`.

---

## FAQ

**Claude ne trouve pas le plugin**
→ Relancer Claude Code apres l'installation : fermer et rouvrir le terminal.

**Les feedbacks Harvestr ne remontent pas**
→ Verifier que l'URL Harvestr est bien renseignee dans `.mcp.json`. L'URL fait office de token d'acces.

**Les feedbacks RAG/SQL ne fonctionnent pas**
→ Verifier que l'URL n8n-feedbacks est bien renseignee dans `.mcp.json`. L'URL fait office de token d'acces.

**Les wireframes ne se generent pas**
→ Verifier que `pip install google-genai` est fait et que la cle Gemini est dans `.mcp.json`.

**Les commandes GitLab ne marchent pas**
→ Verifier que le token GitLab est bien dans `.mcp.json` et qu'il a le scope `api`.

**Je veux utiliser le plugin dans un autre projet**
→ Le plugin est installe globalement. Copier `.mcp.json` dans le nouveau projet et ajouter un dossier `semji-screenshots/` avec des captures de l'app.
