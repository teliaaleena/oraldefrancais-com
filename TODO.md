# TODO.md

Liste des actions ouvertes sur le projet `oraldefrancais.com`. Les entrées les plus urgentes sont en haut.

---

## En attente — action requise

### [P0] Télia doit installer l'app GitHub "Cloudflare Workers and Pages" sur son compte

**Pourquoi** : sans cette install, les commits sur le repo ne déclenchent pas de redéploiement automatique sur Cloudflare. Chaque modification nécessiterait un trigger manuel. Le bandeau bleu "disconnected from your Git account" sur Cloudflare reste affiché tant que ce n'est pas fait.

**Procédure** (Télia, depuis sa session GitHub `teliaaleena`) :
1. Aller sur https://github.com/apps/cloudflare-workers-and-pages
2. Cliquer **Configure** (ou **Install**)
3. Choisir le compte **`teliaaleena`** si plusieurs apparaissent
4. Dans **Repository access** : "Only select repositories" → cocher **`oraldefrancais-com`**
5. Cliquer **Install** / **Save**

**Vérification** :
- Le bandeau bleu sur Cloudflare doit disparaître après refresh de la page Settings
- Test final : éditer `README.md` sur GitHub, ajouter un espace, commit. Un nouveau deployment doit apparaître automatiquement dans Cloudflare en ~30 sec.

---

## En attente — décisions à prendre

### [P1] Refonte de `teliamoreau.com` en page perso de Télia

Site personnel hébergé sur l'ancien repo `teliaaleena/oraldefrancais` + ancien projet Cloudflare Pages. Aujourd'hui sert encore l'ancienne landing "Télia Moreau" qui était la landing de l'outil avant migration.

À décider avant de démarrer :
- Photo de Télia ou pas ?
- Projets futurs visibles ou page minimaliste ?
- Lien vers `oraldefrancais.com` en CTA principal ou en mention discrète ?
- Citations Proust gardées ou autres choisies ?

**Bloque** : démarrage du chantier "page perso Télia".

### [P1] Liste des textes du programme EAF de Télia (BCIS Phuket Première)

Pour démarrer la fonctionnalité "bibliothèque commune" partageable avec la classe, il faut le contenu : ~15-22 textes du programme avec pour chaque texte :
- Passage intégral
- Problématique modèle
- 2-3 mouvements (titres + analyse)
- Introduction modèle
- Conclusion modèle
- Flashcards à trous

**Bloque** : démarrage du code de la bibliothèque commune.

### [P2] Sort du dossier `revision/` dans l'ancien repo `oraldefrancais` une fois `teliamoreau.com` refondu en page perso

Options :
- Le supprimer (404 sur `teliamoreau.com/revision`)
- Garder une redirection 301 vers `oraldefrancais.com/revision`

### [P2] Timing de la PWA (Progressive Web App)

Télia a coché PWA dans ses choix de features. À implémenter avant ou après les autres features Bloc 1/2 ?

---

## Backlog technique

- Tester nouvelle landing sur iPhone SE (375px) — vérifier que le H1 long ne casse pas le layout
- Tester mode fantôme sur le revision tool en live
- Créer un PAT GitHub côté Vincent si jamais on veut pousser en CLI plus tard
