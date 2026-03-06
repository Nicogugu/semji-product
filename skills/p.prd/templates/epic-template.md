# Template EPIC Semji

Utilise ce format exact pour generer l'EPIC. Remplace les {{VARIABLES}} par les informations collectees.

---

## Format de l'EPIC

```
# 1. Contexte & Valeur Business

**Objectif** : {{OBJECTIF_PRINCIPAL}}

**Valeur Business** :
- **Acquisition** : {{VALEUR_ACQUISITION}}
- **Retention** : {{VALEUR_RETENTION}}
- **Competitivite** : {{VALEUR_COMPETITIVITE}}

---

# 2. Objectifs & Succes

## Success Metrics

L'epique sera consideree comme reussie si :
{{POUR_CHAQUE_KPI}}
- **{{NOM_KPI}}** : {{CIBLE_CHIFFREE}}
{{/POUR_CHAQUE_KPI}}

## Permissions & Roles

- **Admin/Editor** : {{PERMISSIONS_ADMIN}}
- **Viewer** : {{PERMISSIONS_VIEWER}}

---

# 3. Experience Utilisateur (Design & UX)

**Reference Figma** : {{LIEN_FIGMA_OU_A_DEFINIR}}

## Onboarding & Etats de l'interface

- **Empty State** : {{DESCRIPTION_EMPTY_STATE}}
- **Etat "Processing"** : {{DESCRIPTION_ETAT_CHARGEMENT}}
- **Bandeau d'erreur** : {{DESCRIPTION_ETAT_ERREUR}}

---

# 4. Specifications Fonctionnelles (User Stories)

{{POUR_CHAQUE_MODULE}}

## {{LETTRE}}. {{NOM_MODULE}} ({{DESCRIPTION_MODULE}})

{{POUR_CHAQUE_US_DU_MODULE}}

### US {{NUMERO}} : {{TITRE_US}}

**En tant qu'** {{ROLE}}, **je veux** {{ACTION}} **afin de** {{BENEFICE}}.

{{POUR_CHAQUE_AC}}
- **AC {{NUMERO_AC}}** : {{DESCRIPTION_CRITERE_ACCEPTANCE}}
{{/POUR_CHAQUE_AC}}

{{/POUR_CHAQUE_US_DU_MODULE}}

{{/POUR_CHAQUE_MODULE}}

---

# 5. Specifications Techniques & Data

## Logique des KPIs

{{POUR_CHAQUE_FORMULE}}
- **{{NOM_KPI}}** :
  {{FORMULE_LATEX_OU_TEXTE}}
{{/POUR_CHAQUE_FORMULE}}

## Logique Metier & Data

{{DESCRIPTION_LOGIQUE_METIER}}
- **Deltas** : {{LOGIQUE_DELTAS}}
- **Ingestion** : {{SOURCE_DONNEES_ET_FREQUENCE}}
{{AUTRES_ELEMENTS_LOGIQUE_METIER}}

## Robustesse & Edge Cases

{{POUR_CHAQUE_EDGE_CASE}}
- **{{NOM_EDGE_CASE}}** : {{DESCRIPTION_ET_GESTION}}
{{/POUR_CHAQUE_EDGE_CASE}}

## Exigences Non Fonctionnelles (NFR)

### Performance
{{EXIGENCES_PERFORMANCE}}

### Scalabilite
{{EXIGENCES_SCALABILITE}}

### Securite & Confidentialite
{{EXIGENCES_SECURITE}}

### Utilisabilite (UX Quality)
{{EXIGENCES_UX}}

---

# 6. Perimetre (Scope)

## Out of Scope (V1)

{{POUR_CHAQUE_ELEMENT_OUT}}
- {{ELEMENT_OUT_OF_SCOPE}}
{{/POUR_CHAQUE_ELEMENT_OUT}}

---

# 7. Definition of Ready (DoR)

Cette section sert de checklist finale avant de passer l'epique en statut "Ready for Dev".

### Vision
- [ ] Objectif clair et Business Value definis.
- [ ] KPIs de succes et User Personas identifies.

### Design
- [ ] Mockups Figma complets (Inclus Empty states & Erreurs).
- [ ] Parcours utilisateur revu et valide.

### Fonctionnel
- [ ] User Stories et Criteres d'Acceptation (AC) rediges.
- [ ] Scope (In/Out) clairement delimite.

### Technique
- [ ] Faisabilite confirmee par le Lead Tech.
- [ ] Logiques de calcul et Edge Cases documentes.
- [ ] Exigences Non Fonctionnelles documentees.

### Validation
- [ ] Epique relue et validee par PM, Lead Tech et Designer.
```

## Notes sur le formatage Notion

Quand tu generes la page Notion via l'outil `notion-create-pages`, utilise le format Notion Markdown :
- Les titres avec `#`, `##`, `###`
- Les checkboxes avec `- [ ]` et `- [x]`
- Les listes a puces avec `-`
- Le texte en gras avec `**texte**`
- Les formules mathematiques avec `$$` si supporte
- Les separateurs avec `---`
- Les citations avec `>`
