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
- `/p.feedback [sujet]` — Collecte les feedbacks depuis Harvestr (+ Slack) et produit une synthese product (problemes, ameliorations, user stories)
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

## Integrations
- Harvestr : API REST directe (rest.harvestr.io/v1) avec token dans .mcp.json
- Notion : creation de pages via MCP (notion-create-pages, notion-update-page)
- Figma : screenshots via MCP (get_metadata, get_screenshot)
- Images : upload via freeimage.host (pas Imgur)
- Slack : #feedbacks-semji (C06TQ2H3J78) — MCP a configurer
