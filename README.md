# Oral de français

Outil libre de révision pour l'EAF (oral du bac de français).  

**Site live :** https://oraldefrancais.com

## Comment ça marche

1. **Colle ton passage** avec des `[crochets]` autour des phrases que tu veux analyser. Tu peux ajouter une correction modèle après une barre `|` :
   `[meilleur des temps | antithèse, hyperbole, registre épique]`

2. **Construis ton plan** en 3 étapes guidées :
   - **Cadre** : problématique, mouvements, introduction
   - **Phrases** : analyse phrase par phrase, auto-évaluation Correct / À revoir
   - **Synthèse** : mini-conclusions, transitions, conclusion, ouverture

3. **Entraîne-toi** : simulation orale chronométrée (5/8/10/12 min), roulette des textes, mode fantôme pour mémoriser.

## Fonctionnalités

- Analyse phrase par phrase avec correction modèle masquable
- Auto-évaluation `Correct` / `À revoir` sur chaque phrase
- Mode Révision ciblée pour reprendre uniquement les phrases marquées
- Flashcards à trous pour mémoriser introduction et conclusion (syntaxe `(parenthèses)`)
- Mode fantôme : la correction modèle s'affiche derrière ton texte pour t'aider à mémoriser
- Simulation orale avec chronomètre et masquage des notes
- Roulette des textes pour le tirage au sort
- Export / Import JSON pour sauvegarder ou changer d'appareil
- Mode clair / mode sombre
- Raccourcis clavier : `C` correct, `R` à revoir, `Enter` phrase suivante

## Données et confidentialité

Tous les textes que tu écris sont stockés dans le `localStorage` de ton navigateur, **sur ton appareil uniquement**.

- Pas de compte
- Pas de tracking
- Pas de cookies
- Pas de serveur qui collecte tes données

**Pour sauvegarder ou changer d'appareil** : utilise les boutons `Exporter` et `Importer` en haut de la bibliothèque. L'export te donne un fichier JSON que tu peux réimporter plus tard, ou sur un autre téléphone.

**Attention** : si tu vides le cache de ton navigateur sans avoir exporté, tes textes sont perdus.

## Stack

HTML + CSS + JavaScript pur. Pas de framework, pas de build, pas de dépendances `npm`.

Pour modifier le site : ouvrir `index.html` ou `revision/index.html`, modifier, sauver, push sur GitHub. Cloudflare Pages redéploie automatiquement en ~30-60 secondes.

Polices chargées via Google Fonts : Space Grotesk (texte) + JetBrains Mono (mono).

## Structure du repo

```
oraldefrancais-com/
├── index.html          Landing oraldefrancais.com (présentation + lien vers /revision)
├── revision/
│   └── index.html      L'outil de révision complet (HTML + CSS + JS dans un seul fichier)
├── 404.html            Page d'erreur
├── favicon.svg         Icône du site
├── og-image.svg        Image Open Graph (partage social)
├── README.md           Ce fichier
└── DEPLOYMENT.md       Documentation déploiement et maintenance
```

## Pour contribuer

Tu es élève, prof, parent, ou simplement quelqu'un qui veut aider ? Les contributions sont les bienvenues.

**Pour signaler un bug ou suggérer une amélioration** :
- Ouvre une issue sur https://github.com/teliaaleena/oraldefrancais-com/issues
- Décris ce qui se passe, ce que tu attendais, et sur quel appareil tu es

**Pour proposer une modification de code** :
- Fork le repo, crée une branche, fais ta modif, ouvre une Pull Request
- Garde la stack : pas de framework, pas de build step, pas de dépendance `npm`
- Teste sur mobile (375px) et desktop (1280px) avant de soumettre

**Pour proposer du contenu** (textes du programme, analyses modèles) :
- Les contributions de contenu sont prévues mais l'infrastructure n'est pas encore en place. Reviens dans quelques semaines, ou ouvre une issue pour discuter.

## Déploiement

Voir `DEPLOYMENT.md` pour la procédure complète, le dépannage et l'historique des décisions.

---

Créé par Télia Moreau · Phuket · 2026 · Code source ouvert
