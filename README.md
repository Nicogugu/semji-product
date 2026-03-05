# Semji Product Tools

Plugin Claude Code pour l'equipe Product Semji. 5 skills pour accelerer le workflow PM :

| Skill | Ce que ca fait |
|-------|---------------|
| `/p.feedback [sujet]` | Collecte les feedbacks clients depuis Harvestr et produit une synthese product |
| `/p.prd [feature]` | Guide une interview PM et genere une EPIC complete sur Notion |
| `/p.issue [lien-notion]` | Transforme une User Story du PRD en issue dev-ready |
| `/p.wireframe [description ou lien Slack]` | Genere des wireframes a partir des screenshots Semji. Accepte un brief precis OU un besoin vague (lien Slack, idee brute) — imagine la feature en mode Product/UX avant de wireframer |
| `/p.gitlab [action]` | Cree, lit, modifie et recherche des issues sur le GitLab Semji |

---

## Installation

### 1. Installer Claude Code

Si ce n'est pas deja fait, installer Claude Code : https://docs.anthropic.com/en/docs/claude-code/overview

Verifier que ca marche :
```
claude --version
```

### 2. Ajouter le plugin

Ouvrir un terminal et lancer ces 2 commandes :

```
claude marketplace add https://github.com/Nicogugu/semji-product.git
claude plugin install semji-product
```

### 3. Configurer les tokens

Le plugin a besoin de 3 cles API pour fonctionner. Elles ne sont pas dans le repo (securite).

**Etape 1** : Copier le fichier exemple

Dans le dossier ou tu travailles avec Claude Code, copier `.mcp.json.example` en `.mcp.json` :
```
cp .mcp.json.example .mcp.json
```

**Etape 2** : Remplir les tokens dans `.mcp.json`

Ouvrir `.mcp.json` avec un editeur de texte et remplacer :

- `<your-harvestr-token>` → demander le token a Nico
- `<your-gemini-api-key>` → demander la cle a Nico (ou en creer une sur https://aistudio.google.com/apikey)
- `<your-gitlab-personal-access-token>` → creer un token sur https://gitlab.rvip.fr/-/user_settings/personal_access_tokens (scope `api`)

### 4. Installer la dependance Python (wireframes uniquement)

Necessaire uniquement si tu utilises `/p.wireframe` :
```
pip install google-genai
```

---

## Utilisation

Lancer Claude Code dans n'importe quel projet :
```
claude
```

Puis taper la commande du skill voulu :

### Collecter des feedbacks
```
/semji-product:p.feedback webhooks
/semji-product:p.feedback content score
```

### Creer un PRD
```
/semji-product:p.prd Webhook v2
```

### Creer une issue depuis un PRD
```
/semji-product:p.issue https://notion.so/semji/mon-prd-123
```

### Generer des wireframes

Avec un brief precis :
```
/semji-product:p.wireframe modale d'ajout de prompts GEO depuis le listing
```

Avec un besoin vague ou un lien Slack (mode Product Imagination) :
```
/semji-product:p.wireframe https://semji.slack.com/archives/C06TQ2H3J78/p1772703414552459
/semji-product:p.wireframe on voudrait permettre aux users de comparer leurs articles
```

> En mode Product Imagination, le skill lit le message source, imagine la feature (probleme, solution UX, user flow, ecrans, edge cases), valide avec le PM, puis genere les wireframes.
>
> Les wireframes utilisent les screenshots dans le dossier `semji-screenshots/` comme reference visuelle. Ajouter des screenshots de l'app Semji dans ce dossier pour de meilleurs resultats.

### Gerer les issues GitLab
```
/semji-product:p.gitlab chercher les issues Bug ouvertes
/semji-product:p.gitlab lire #11529
/semji-product:p.gitlab creer une issue pour le bug du lien casse
/semji-product:p.gitlab modifier #11550 — passer en P2
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

## FAQ

**Je n'ai pas le bon token Harvestr**
→ Demander a Nico. Le token est celui de l'app privee Harvestr.

**Les wireframes ne se generent pas**
→ Verifier que `pip install google-genai` est fait et que la cle Gemini est dans `.mcp.json`.

**Les commandes GitLab ne marchent pas**
→ Verifier que le token GitLab est bien dans `.mcp.json` et qu'il a le scope `api`. Le creer ici : https://gitlab.rvip.fr/-/user_settings/personal_access_tokens

**Claude ne trouve pas le plugin**
→ Relancer Claude Code apres l'installation : fermer et rouvrir le terminal.

**Je veux utiliser le plugin dans un autre projet**
→ Le plugin est installe globalement. Il suffit de copier `.mcp.json` dans le nouveau projet et d'ajouter un dossier `semji-screenshots/` avec des captures de l'app.
