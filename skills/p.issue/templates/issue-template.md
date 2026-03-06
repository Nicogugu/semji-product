# Template Issue Semji

Utilise ce format exact pour generer l'Issue. Remplace les {{VARIABLES}} par les informations collectees ou derivees de la User Story source.

---

## Format de l'Issue

```
# {{TITRE_ISSUE}}

## Description

**En tant que :** {{PERSONA}}
**Je souhaite :** {{FONCTIONNALITE}}
**Afin de :** {{VALEUR_AJOUTEE}}

---

## Criteres d'Acceptation (AC)

> _**Instructions :** C'est ici que l'on definit quand la tache est "terminee". Sert de base pour la demo._
> _- **Capacite vs Interface :** Ne decrivez pas le "Comment" (ex: "cliquer sur le bouton bleu"), mais la **capacite offerte** ou la **regle metier** (ex: "Possibilite de supprimer un prompt"). Le prototype Figma fait foi pour l'UI._
> _- **Independance :** Un AC doit etre testable independamment des autres dans la mesure du possible._
> _- **Taille :** A partir de **8 AC**, le decoupage en deux US distinctes est **obligatoire**._

{{POUR_CHAQUE_AC}}
- [ ] **AC {{NUMERO}} :** {{ACTION_UTILISATEUR}} -> {{RESULTAT_ATTENDU}}
{{/POUR_CHAQUE_AC}}

---

## Design & UX

> _- **Lien Figma** : Clic droit sur la frame specifique dans Figma -> "Copy link to selection"._
> _- **Etats Cles** : Decrivez le comportement visuel attendu pour chaque etat._
> _- **Zero Logique** : Ne repetez pas ici les regles de calcul. Concentrez-vous sur l'experience utilisateur._

- **Maquette Figma :** {{LIEN_FIGMA}}
- **Etats Cles :**
    - **Chargement :** {{ETAT_CHARGEMENT}}
    - **Erreur :** {{ETAT_ERREUR}}
    - **Empty State :** {{ETAT_VIDE}}
    - **Succes :** {{ETAT_SUCCES}}

---

## Specifications Techniques

> _- **Focus Logique** : Detaillez le "cerveau" de l'US. Comment la donnee est-elle traitee avant d'etre affichee ?_
> _- **Precision Mathematique** : Utilisez des formules claires pour eviter les erreurs d'interpretation._
> _- **Anticipation** : Ne listez que les cas limites propres a cette US._

- **Logique Metier :** {{LOGIQUE_METIER}}
- **Calculs / Formules :** {{CALCULS_FORMULES}}
- **Cas Limites (Edge Cases) :**
{{POUR_CHAQUE_EDGE_CASE}}
    - [ ] {{DESCRIPTION_EDGE_CASE}}
{{/POUR_CHAQUE_EDGE_CASE}}
- **Dependances :** {{DEPENDANCES}}

---

## Checklist

_Si une case n'est pas cochee, le ticket ne doit pas passer en "Todo"._

- [ ] **Small :** Le ticket est realisable en un seul cycle.
- [ ] **Independent :** Les dependances sont identifiees et levees.
- [ ] **Clear :** Les AC permettent de faire l'auto-QA sans poser de questions.

---

## Optionnel

{{SI_ANALYTICS}}
- **Analytics & Tracking :**
    - **Evenements a tracker :** {{EVENEMENTS_TRACKING}}
    - **Proprietes associees :** {{PROPRIETES_TRACKING}}
{{/SI_ANALYTICS}}

{{SI_FEATURE_FLAG}}
- **Feature Flag & Rollout :**
    - **Nom du flag :** {{NOM_FLAG}}
    - **Strategie :** {{STRATEGIE_ROLLOUT}}
{{/SI_FEATURE_FLAG}}

{{SI_ENABLEMENT}}
- **Enablement & Aide :**
    - **Impact Documentation :** {{IMPACT_DOC}}
    - **Info CS/Sales :** {{INFO_CS_SALES}}
{{/SI_ENABLEMENT}}

{{SI_QA}}
- **Donnees de Test (QA) :**
    - **Dataset recommande :** {{DATASET_QA}}
    - **Environnement :** {{ENVIRONNEMENT_QA}}
{{/SI_QA}}

{{SI_REFERENCES}}
- **References Externes :**
    - **Contexte :** {{LIENS_REFERENCES}}
{{/SI_REFERENCES}}
```

## Notes sur le formatage Notion

Quand tu generes la page Notion via l'outil `notion-create-pages`, utilise le format Notion Markdown :
- Les titres avec `#`, `##`, `###`
- Les checkboxes avec `- [ ]` et `- [x]`
- Les listes a puces avec `-`
- Le texte en gras avec `**texte**`
- Les separateurs avec `---`
- Les citations avec `>`
