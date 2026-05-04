# oraldefrancais.com

Outil de révision pour le bac de français (EAF), construit par et pour des élèves.

Site en ligne : [oraldefrancais.com](https://oraldefrancais.com)

---

## C'est quoi

Un outil web pour préparer l'oral du bac de français. Tu importes les œuvres étudiées en classe (depuis le programme officiel ou en texte libre), et tu travailles tes plans détaillés phrase par phrase.

Pas d'inscription. Pas de tracking. Tout reste sur ton téléphone ou ton ordi.

## Comment ça marche

1. Choisis ton ambiance (5 palettes ou TryMe pour une surprise)
2. Renseigne ton prénom et la date de ton oral pour le compte à rebours
3. Importe une œuvre (au programme officiel ou en texte libre)
4. Pour chaque phrase, écris ton analyse — l'outil te guide

## Stack

HTML, CSS et JavaScript vanilla. Aucun framework, aucun build, aucune dépendance.

```
.
├── index.html              homepage + onboarding
├── faq/index.html          questions fréquentes
├── revision/index.html     l'outil de révision
├── content/
│   └── programme-2026.json contenu (œuvres au programme + FAQ)
└── favicon.svg
```

Le contenu (`programme-2026.json`) est éditable directement depuis l'interface web GitHub.

## Déploiement

Cloudflare Pages, auto-deploy sur push vers `main`. Voir [DEPLOYMENT.md](./DEPLOYMENT.md).

## État du projet

Pas un produit commercial. Outil personnel pour préparer un oral, partagé en open source au cas où ça aide quelqu'un d'autre.

Les améliorations en cours et celles arrêtées : [TODO.md](./TODO.md).

## Licence

Code sous MIT (fais ce que tu veux). Le contenu littéraire (œuvres, analyses ajoutées par les élèves) reste la propriété de leurs auteurs respectifs.
