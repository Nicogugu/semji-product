# Semji Product Tools

Plugin Claude Code pour l'equipe Product Semji. 4 skills pour accelerer le workflow PM :

| Skill | Ce que ca fait |
|-------|---------------|
| `/p.feedback [sujet]` | Collecte les feedbacks clients depuis Harvestr et produit une synthese product |
| `/p.prd [feature]` | Guide une interview PM et genere une EPIC complete sur Notion |
| `/p.issue [lien-notion]` | Transforme une User Story du PRD en issue dev-ready |
| `/p.wireframe [description]` | Genere des wireframes a partir des screenshots Semji |

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

Le plugin a besoin de 2 cles API pour fonctionner. Elles ne sont pas dans le repo (securite).

**Etape 1** : Copier le fichier exemple

Dans le dossier ou tu travailles avec Claude Code, copier `.mcp.json.example` en `.mcp.json` :
```
cp .mcp.json.example .mcp.json
```

**Etape 2** : Remplir les tokens dans `.mcp.json`

Ouvrir `.mcp.json` avec un editeur de texte et remplacer :

- `<your-harvestr-token>` → demander le token a Nico
- `<your-gemini-api-key>` → demander la cle a Nico (ou en creer une sur https://aistudio.google.com/apikey)

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
```
/semji-product:p.wireframe modale d'ajout de prompts GEO depuis le listing
```

> Les wireframes utilisent les screenshots dans le dossier `semji-screenshots/` comme reference visuelle. Ajouter des screenshots de l'app Semji dans ce dossier pour de meilleurs resultats.

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
```

---

## FAQ

**Je n'ai pas le bon token Harvestr**
→ Demander a Nico. Le token est celui de l'app privee Harvestr.

**Les wireframes ne se generent pas**
→ Verifier que `pip install google-genai` est fait et que la cle Gemini est dans `.mcp.json`.

**Claude ne trouve pas le plugin**
→ Relancer Claude Code apres l'installation : fermer et rouvrir le terminal.

**Je veux utiliser le plugin dans un autre projet**
→ Le plugin est installe globalement. Il suffit de copier `.mcp.json` dans le nouveau projet et d'ajouter un dossier `semji-screenshots/` avec des captures de l'app.
