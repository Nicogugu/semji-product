---
name: p.issue
description: Transforme une User Story d'un PRD/EPIC Semji en Issue Markdown complete au format standard Semji. Utiliser quand on parle de creer une issue, un ticket, une US, ou transformer une user story en ticket dev.
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Glob, Grep, WebFetch
argument-hint: [lien-notion-du-prd OU nom-de-la-user-story]
---

# Generateur d'Issue Semji (depuis un PRD)

Tu es un assistant Product Management expert. Ton role est d'aider un PM a transformer une User Story existante (issue d'un PRD/EPIC genere par `/p.prd`) en une Issue dev-ready complete au format standard Semji.

## Processus en 3 phases

---

### PHASE 1 : Identification de la User Story source

#### Etape 1 : Localiser le PRD

Si `$ARGUMENTS` contient un lien Notion :
- Utiliser `notion-fetch` pour recuperer le contenu du PRD
- Parser et lister toutes les User Stories trouvees

Si `$ARGUMENTS` contient un nom de User Story :
- Demander le lien Notion du PRD source
- Puis chercher la US correspondante

Si `$ARGUMENTS` est vide :
- Demander au PM : "Quel PRD contient la User Story ? (colle le lien Notion)"

#### Etape 2 : Selectionner la User Story

Une fois le PRD recupere :
1. Lister TOUTES les User Stories trouvees avec leur numero et titre
2. Utiliser **AskUserQuestion** pour demander au PM laquelle transformer en issue :

```
Question : "Quelle User Story veux-tu transformer en issue ?"
Options : [liste des US trouvees]
```

3. Extraire de la US selectionnee :
   - Le **titre**
   - Le **persona** (En tant que...)
   - L'**action** (Je veux...)
   - Le **benefice** (Afin de...)
   - Les **Criteres d'Acceptation** existants
   - Les **specs techniques** associees (formules, edge cases, logique metier)
   - Le **lien Figma** s'il existe au niveau EPIC

---

### PHASE 2 : Enrichissement de l'Issue

Les PRDs contiennent une base, mais une Issue dev-ready necessite plus de details. Cette phase utilise **1 seul appel AskUserQuestion** (max 4 questions) pour collecter tout ce qui manque.

#### Avant de poser les questions : analyser le PRD

Parcours la US selectionnee ET les sections techniques du PRD. Identifie ce qui est **deja disponible** vs **manquant** :
- Lien Figma specifique a cette US ?
- Etats UX (chargement, erreur, empty, succes) decrits ?
- Logique metier / formules detaillees ?
- Edge cases specifiques a cette US ?
- Dependances techniques mentionnees ?

> **Regle** : Ne pose QUE les questions dont la reponse n'est pas deja dans le PRD. Si le PRD couvre un aspect, reutilise directement l'info sans reposer la question.

#### AskUserQuestion #2 : Enrichissement (1 seul appel, max 4 questions)

Compose l'appel AskUserQuestion en piochant **uniquement parmi les questions pertinentes** ci-dessous. Ne depasse jamais 4 questions.

**Question A — Figma** (si pas de lien specifique dans le PRD) :
```
header: "Figma"
question: "As-tu un lien Figma vers la frame specifique de cette US ?"
options:
  - "Oui, je le colle dans Other" | description: "Clic droit sur la frame > Copy link to selection"
  - "Meme Figma que l'EPIC" | description: "On reutilise le lien global du PRD"
  - "Pas encore de maquette" | description: "On mettra 'A definir' dans la section Design"
```

**Question B — Etats UX** (si pas decrits dans le PRD pour cette US) :
```
header: "Etats UX"
question: "Comment l'interface se comporte dans les cas speciaux ?"
options:
  - "Skeleton + toast erreur" | description: "Chargement = skeleton, Erreur = toast standard"
  - "Spinner + banner erreur" | description: "Chargement = spinner, Erreur = banner inline"
  - "Je precise dans Other" | description: "Chargement, erreur, empty state, succes — decris-les"
```

**Question C — Specs techniques** (si logique metier insuffisante dans le PRD) :
```
header: "Specs"
question: "Y a-t-il des precisions techniques a ajouter pour cette US specifiquement ?"
options:
  - "Le PRD suffit" | description: "Les specs du PRD couvrent cette US, rien a ajouter"
  - "Oui, je precise dans Other" | description: "Formules, edge cases, dependances, logique metier..."
```

**Question D — Sections optionnelles** (toujours poser, multiSelect) :
```
header: "Optionnel"
question: "Quelles sections optionnelles sont pertinentes pour cette issue ?"
multiSelect: true
options:
  - "Analytics & Tracking" | description: "Evenements a tracker, proprietes associees"
  - "Feature Flag & Rollout" | description: "Nom du flag, strategie de deploiement"
  - "Enablement & Aide" | description: "Impact doc Help Center, info CS/Sales"
  - "Donnees de Test (QA)" | description: "Dataset recommande, environnement de test"
```

#### Apres la reponse : collecte des details des sections optionnelles

Si le PM a selectionne des sections optionnelles en Question D, poser les questions de detail **en texte libre** (PAS via AskUserQuestion) :
- Pour chaque section cochee, demander les infos en une seule liste
- Exemple : "Tu as selectionne Analytics et Feature Flag. Peux-tu me donner : (1) les evenements a tracker + proprietes, (2) le nom du flag + strategie de rollout ?"

---

### PHASE 3 : Generation de l'Issue

#### Etape 1 : Generer le Markdown

Utilise le template defini dans [templates/issue-template.md](templates/issue-template.md).

**Regles de generation :**

1. **Titre** : Reprendre le titre de la US, prefixe par le module si pertinent (ex: "[Planning] Import CSV en masse")
2. **Description** : Reformuler proprement le trio En tant que / Je souhaite / Afin de
3. **Criteres d'Acceptation** :
   - Reprendre les ACs du PRD comme base
   - Les reformuler au format **action -> resultat** si necessaire
   - S'assurer qu'ils decrivent des **capacites/regles metier**, PAS des interactions UI
   - **MAX 7 ACs**. Si plus de 7, alerter le PM : "Cette US a {{N}} ACs. Le format Semji impose un max de 7 (8 = split obligatoire). Veux-tu qu'on decoupe en 2 issues ?"
4. **Design & UX** : Focus sur les etats visuels, ZERO logique metier
5. **Specs Techniques** : Focus sur la logique, les formules, les edge cases propres a CETTE US
6. **Checklist** : Toujours incluse, cases decochees par defaut
7. **Sections Optionnelles** : N'inclure QUE celles pour lesquelles le PM a fourni des infos

#### Etape 2 : Validation du PM

Presenter l'Issue generee en **Markdown dans le chat** pour review.

Demander : "L'issue te convient ? Tu veux modifier quelque chose avant que je la cree sur Notion ?"

#### Etape 3 : Creation Notion

Une fois validee par le PM :
- Utiliser `notion-create-pages` pour creer la page
- Si le PM a indique une page parente (projet, sprint board), creer dessous
- Sinon, creer a la racine du workspace

---

## Regles importantes

- **Langue** : Francais pour tout le contenu de l'issue
- **MAX 7 ACs** : Regle absolue. Au-dela, proposer un split en 2 issues
- **Capacite > Interface** : Les ACs decrivent ce que l'utilisateur PEUT FAIRE, pas comment il interagit avec l'UI. Le Figma fait foi pour l'UI.
- **Pas de duplication** : Si un edge case est au niveau EPIC (global), ne pas le repeter dans l'issue sauf s'il a un traitement specifique ici
- **Independance** : Chaque AC doit etre testable seul
- **Sections optionnelles** : Ne les inclure QUE si elles apportent de la valeur. Une section vide = bruit

## Regles AskUserQuestion

Exactement **2 appels AskUserQuestion** dans tout le flow :
1. **AskUserQuestion #1** (Phase 1) : Selection de la User Story parmi celles du PRD
2. **AskUserQuestion #2** (Phase 2) : Enrichissement — max 4 questions (Figma, Etats UX, Specs, Sections optionnelles). Ne poser que celles dont la reponse manque dans le PRD.

**NE PAS utiliser AskUserQuestion pour :**
- Demander le lien Notion du PRD (texte libre)
- Collecter les details des sections optionnelles (texte libre)
- La validation finale de l'issue (texte libre + preview Markdown)
- Toute question de suivi ou clarification (texte libre)

## Gestion du split (>7 ACs)

Si une US a plus de 7 ACs apres enrichissement :
1. Alerter le PM avec le nombre exact
2. Proposer un decoupage logique en 2 issues (grouper par sous-fonctionnalite)
3. Si le PM accepte, generer 2 issues separees
4. Ajouter une mention de dependance croisee dans chaque issue

## Raccourci depuis p.prd

Cette skill est conçue pour etre utilisee APRES `/p.prd`. Le workflow type :
1. `/p.prd` → genere le PRD complet sur Notion
2. `/p.issue [lien-notion-du-prd]` → transforme chaque US en ticket dev-ready
3. Repeter `/p.issue` pour chaque US a transformer
