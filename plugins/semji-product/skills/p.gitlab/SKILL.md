---
name: p.gitlab
description: Gere les issues GitLab Semji via API REST. Utiliser quand on parle de creer une issue gitlab, lire, modifier, rechercher des issues, uploader des images, ou interagir avec le backlog GitLab.
disable-model-invocation: false
user-invocable: true
allowed-tools: Bash, Read, Grep, AskUserQuestion
argument-hint: [action] [details] — ex: "creer une issue pour le bug X" ou "chercher les issues P0 ouvertes"
---

# GitLab Issues Manager — Semji

Tu es un assistant qui gere les issues du projet GitLab Semji (`semji/semji`, project ID 221) via l'API REST GitLab. Tu connais l'app Semji en profondeur (consulte `${CLAUDE_SKILL_DIR}/semji-product-context.md` pour le contexte produit complet).

## Configuration

- **GitLab URL** : `https://gitlab.rvip.fr`
- **Project ID** : `221` (semji/semji)
- **Auth** : Token lu depuis la variable d'environnement `GITLAB_TOKEN`, ou depuis `.mcp.json` (champ `gitlab.env.GITLAB_TOKEN`), ou demander a l'utilisateur.

Header d'authentification pour tous les appels :
```
--header "PRIVATE-TOKEN: $TOKEN"
```

## Processus

### Etape 1 : Determiner l'action

Analyser `$ARGUMENTS` pour identifier l'intention :

| Intention detectee | Action |
|---|---|
| "creer", "ajouter", "nouvelle issue" | → **CREER** |
| "lire", "voir", "afficher", "#1234" | → **LIRE** |
| "modifier", "update", "changer", "assigner" | → **MODIFIER** |
| "chercher", "rechercher", "lister", "trouver" | → **RECHERCHER** |
| "image", "upload", "wireframe", "screenshot", "ajouter image" | → **AJOUTER DES IMAGES** |

Si ambigue, utiliser **AskUserQuestion** :
```
header: "Action"
question: "Que veux-tu faire sur GitLab ?"
options:
  - "Creer une issue"
  - "Lire une issue existante"
  - "Modifier une issue"
  - "Rechercher des issues"
```

### Etape 2 : Recuperer le token

```bash
# Priorite 1 : variable d'environnement
echo $GITLAB_TOKEN

# Priorite 2 : .mcp.json
cat .mcp.json | python -c "import json,sys; print(json.load(sys.stdin)['mcpServers']['gitlab']['env']['GITLAB_TOKEN'])"
```

Si aucun token trouve, demander a l'utilisateur.

---

## CREER une issue

### Collecte des infos (1 seul AskUserQuestion, max 4 questions)

Analyser `$ARGUMENTS` pour pre-remplir ce qui est deja fourni. Ne poser QUE les questions manquantes.

**Question A — Titre** (si pas dans les arguments) :
```
header: "Titre"
question: "Quel titre pour l'issue ?"
options:
  - "Je decris dans Other"
```

**Question B — Labels** (toujours poser) :
```
header: "Labels"
question: "Quels labels ?"
multiSelect: true
options:
  - "Bug" | description: "Bug confirme"
  - "Dev/To Do" | description: "Pret pour dev"
  - "P0" | description: "Priorite critique"
  - "P1" | description: "Priorite haute"
```
> L'utilisateur peut aussi ecrire des labels custom dans Other. Labels disponibles : voir section Reference Labels.

**Question C — Assignee** (optionnel) :
```
header: "Assignee"
question: "Assigner a quelqu'un ?"
options:
  - "Personne pour l'instant"
  - "Moi (nicolas.nguyen)" | description: "User ID 29"
  - "Je precise dans Other" | description: "Username GitLab"
```

**Question D — Milestone** (optionnel) :
```
header: "Milestone"
question: "Quel milestone ?"
options:
  - "Aucun"
  - "20.4.2" | description: "Milestone ID 1706"
  - "20.5.0" | description: "Milestone ID 1705"
  - "next" | description: "Milestone ID 1295"
```

### Appel API

```bash
curl -s --request POST \
  --header "PRIVATE-TOKEN: $TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "title": "TITRE",
    "description": "DESCRIPTION en Markdown",
    "labels": "label1,label2",
    "assignee_ids": [USER_ID],
    "milestone_id": MILESTONE_ID
  }' \
  "https://gitlab.rvip.fr/api/v4/projects/221/issues"
```

Apres creation, afficher :
- Numero de l'issue `#IID`
- Lien direct : `https://gitlab.rvip.fr/semji/semji/-/issues/IID`
- Labels et assignee confirmes

---

## LIRE une issue

Si `$ARGUMENTS` contient un numero (#1234 ou 1234) :

```bash
curl -s --header "PRIVATE-TOKEN: $TOKEN" \
  "https://gitlab.rvip.fr/api/v4/projects/221/issues/IID" | python -m json.tool
```

Presenter de facon lisible :
- **Titre** et **Etat** (ouvert/ferme)
- **Description** (Markdown formate)
- **Labels**, **Milestone**, **Assignee**
- **Dates** (creation, derniere modif)
- **Lien** direct vers GitLab

Si pas de numero, demander : "Quelle issue veux-tu consulter ? (numero ou titre)"

---

## MODIFIER une issue

### Identifier l'issue

Si `$ARGUMENTS` contient un numero, l'utiliser. Sinon demander.

### Collecter les modifications (AskUserQuestion)

```
header: "Modifier"
question: "Que veux-tu modifier sur #IID ?"
multiSelect: true
options:
  - "Titre / Description"
  - "Labels"
  - "Assignee"
  - "Milestone / Etat (open/close)"
```

Pour chaque selection, demander les nouvelles valeurs en texte libre.

### Appel API

```bash
curl -s --request PUT \
  --header "PRIVATE-TOKEN: $TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "title": "nouveau titre",
    "description": "nouvelle description",
    "labels": "label1,label2",
    "assignee_ids": [USER_ID],
    "milestone_id": MILESTONE_ID,
    "state_event": "close"
  }' \
  "https://gitlab.rvip.fr/api/v4/projects/221/issues/IID"
```

> Ne passer que les champs a modifier. `state_event` : `close` ou `reopen`.

---

## RECHERCHER des issues

### Construire la requete

Analyser `$ARGUMENTS` pour extraire les filtres. Parametres API disponibles :

| Parametre | Usage | Exemple |
|---|---|---|
| `state` | opened, closed, all | `state=opened` |
| `labels` | Filtrer par labels (virgule) | `labels=Bug,P0` |
| `search` | Recherche texte libre | `search=editor` |
| `assignee_username` | Filtrer par assignee | `assignee_username=nicolas.nguyen` |
| `milestone` | Filtrer par milestone | `milestone=20.5.0` |
| `order_by` | Tri : created_at, updated_at, priority | `order_by=created_at` |
| `sort` | asc ou desc | `sort=desc` |
| `per_page` | Nombre de resultats (max 100) | `per_page=20` |

### Appel API

```bash
curl -s --header "PRIVATE-TOKEN: $TOKEN" \
  "https://gitlab.rvip.fr/api/v4/projects/221/issues?state=opened&labels=Bug&per_page=20&sort=desc&order_by=created_at"
```

### Affichage

Presenter sous forme de tableau :

```
| #    | Titre                          | Labels         | Assignee | Date       |
|------|--------------------------------|----------------|----------|------------|
| 1234 | Bug: crash editor              | Bug, P0        | alice    | 2026-03-01 |
```

Si plus de 20 resultats, indiquer le total et proposer de paginer.

---

## AJOUTER DES IMAGES a une issue

Workflow en 2 etapes pour uploader des images (wireframes, screenshots, schemas) et les integrer dans la description d'une issue existante.

### Etape 1 : Uploader les images

Pour chaque image a ajouter, utiliser l'endpoint upload :

```bash
curl -s --header "PRIVATE-TOKEN: $TOKEN" \
  --form "file=@chemin/vers/image.png" \
  "https://gitlab.rvip.fr/api/v4/projects/221/uploads"
```

La reponse contient la reference Markdown a conserver :
```json
{
  "alt": "image",
  "url": "/uploads/HASH/image.png",
  "full_path": "/semji/semji/uploads/HASH/image.png",
  "markdown": "![image](/uploads/HASH/image.png)"
}
```

**Collecter toutes les references `markdown`** de chaque upload avant de passer a l'etape 2.

> **Astuce** : Uploader plusieurs images en boucle bash :
> ```bash
> for img in wireframe-step1.png wireframe-step2.png; do
>   curl -s --header "PRIVATE-TOKEN: $TOKEN" \
>     --form "file=@$img" \
>     "https://gitlab.rvip.fr/api/v4/projects/221/uploads"
> done
> ```

### Etape 2 : Mettre a jour la description de l'issue

Construire la nouvelle description en integrant les references Markdown des images, puis mettre a jour l'issue :

```bash
curl -s --request PUT \
  --header "PRIVATE-TOKEN: $TOKEN" \
  --header "Content-Type: application/json" \
  --data @description.json \
  "https://gitlab.rvip.fr/api/v4/projects/221/issues/IID"
```

> **Astuce : fichier JSON temporaire** — Pour les descriptions longues contenant du Markdown avec des backticks, retours a la ligne, et caracteres speciaux, ecrire le JSON dans un fichier temporaire (`description.json`) puis le passer avec `--data @fichier.json`. Cela evite tous les problemes d'echappement bash. Penser a supprimer le fichier apres.

### Regles
- **Uploader AVANT de modifier** : Toujours uploader toutes les images d'abord, puis faire un seul PUT pour la description
- **Nommage** : Utiliser des noms descriptifs (pas `image1.png` mais `webhook-step1-dropdown.png`)
- **Ordre** : Inserer les images dans l'ordre logique du user flow
- **Sections** : Grouper les wireframes sous un heading `### Wireframes du flow` dans la section Design & UX

---

## Reference Labels (les plus courants)

### Workflow Dev
`Dev/To Do`, `Dev/Doing`, `Dev/To Review`, `Dev/To brainstorm`

### Priorite
`P0` (critique), `P1` (haute), `P2` (moyenne), `P3` (basse), `P4`

### Type
`Bug`, `Bug/Confirmed`, `Bug/Production`, `Bug/Regression`, `EPIC`, `UX/UI`, `Clean`, `Security`, `Perfs`

### Fonctionnel
`Functional/AI+ Content`, `Functional/Agents`, `Functional/Benchmark GEO`, `Functional/Editor`, `Functional/GEO`, `Functional/Intelligence Hub`, `Functional/Planning`, `Functional/Chrome Extension`

### Product
`Product/Specs To Do`, `Product/Specs Doing`, `Product/Specs Ready`, `Product/Specs To Review`, `Product/Mockups To Do`, `Product/Mockups Doing`, `Product/Needs CPO Review`

### QA
`QA/To Test`, `QA/Testing`, `QA/Validated`, `QA/Not Validated`

### Squads
`Squad/Productivity`, `Squad/Visibility`

---

## Regles

- **Langue** : Francais pour tout le contenu des issues
- **Pas de suppression** : Ce skill ne permet PAS de supprimer des issues
- **Confirmation avant creation/modification** : Toujours montrer un resume de ce qui va etre envoye a l'API et attendre validation
- **Token securise** : Ne jamais afficher le token dans les outputs
- **Encodage** : Echapper les caracteres speciaux dans les JSON (guillemets, retours a la ligne)
- **Markdown** : Les descriptions supportent le Markdown GitLab (checkboxes `- [ ]`, mentions `@user`, references `#issue`)
