# DEPLOYMENT.md

Document de référence pour le déploiement et la maintenance d'**oraldefrancais.com**.

> **Si tu lis ça parce que quelque chose est cassé** : va directement à [Dépannage](#dépannage) en bas du fichier.

---

## TL;DR

- Le site vit sur https://oraldefrancais.com
- Pour modifier : édite un fichier sur GitHub (web UI) ou en local (git CLI), push sur `main`, attends 30-60s
- Si cassé : va dans Cloudflare → Pages → `oraldefrancais-com` → Deployments → rollback à la version précédente

---

## Vue d'ensemble

Site statique (HTML + CSS + JavaScript pur, pas de build) hébergé gratuitement sur Cloudflare Pages, avec déploiement automatique depuis GitHub.

```
        ┌──────────────────────────┐
        │  Édition d'un fichier    │
        │  sur GitHub (web UI)     │
        │  ou en local (git CLI)   │
        └────────────┬─────────────┘
                     │ push to main
                     ▼
        ┌──────────────────────────┐
        │  GitHub repo             │
        │  teliaaleena/            │
        │  oraldefrancais-com      │
        └────────────┬─────────────┘
                     │ webhook
                     ▼
        ┌──────────────────────────┐
        │  Cloudflare Pages        │
        │  (compte Vincent)        │
        │  oraldefrancais-com      │
        │  .pages.dev              │
        └────────────┬─────────────┘
                     │ DNS CNAME
                     ▼
        ┌──────────────────────────┐
        │  oraldefrancais.com      │
        │  (live, ~30-60s après    │
        │   le push)               │
        └──────────────────────────┘
```

---

## Comptes et propriété

| Élément | Propriétaire | Compte |
|---|---|---|
| Code source | Télia | github.com/teliaaleena |
| Repo | Télia | github.com/teliaaleena/oraldefrancais-com (public) |
| Cloudflare Pages projet | Vincent | dash.cloudflare.com |
| Domaine `oraldefrancais.com` | Vincent | dash.cloudflare.com (Registrar + DNS) |

**Pourquoi ce split** : Télia peut éditer le contenu sans toucher au compte Cloudflare. Vincent gère DNS et facturation domaine sans toucher au compte GitHub. Si plus tard Télia veut tout récupérer, le repo lui appartient déjà ; il suffit de transférer le projet Pages et le domaine sur son propre compte Cloudflare.

---

## Configuration Cloudflare Pages

Ces réglages sont fixés. Si quelqu'un les modifie, le site peut casser.

| Paramètre | Valeur |
|---|---|
| Project name | `oraldefrancais-com` |
| Repository | `teliaaleena/oraldefrancais-com` |
| Production branch | `main` |
| Framework preset | `None` |
| Build command | *(vide)* |
| Build output directory | `/` |
| Root directory | `/` |

**Custom domains attachés** :
- `oraldefrancais.com` (apex, primary)
- `www.oraldefrancais.com` (redirect to apex, optionnel)

**DNS** (côté Cloudflare DNS, zone `oraldefrancais.com`) :
- Type `CNAME`, Name `@`, Target `oraldefrancais-com.pages.dev`, Proxied (orange cloud)

---

## Comment modifier le site

### Cas 1 — petite modif via GitHub web UI (le plus simple)

1. Va sur `github.com/teliaaleena/oraldefrancais-com`
2. Clique le fichier à modifier (ex. `index.html` à la racine, ou `revision/index.html`)
3. Bouton **crayon ✏️** en haut à droite du fichier
4. Modifie dans l'éditeur
5. En bas, dans "Commit changes" : écris un message court (ex. "Corrige typo intro")
6. Bouton vert **Commit changes**
7. Attends ~30-60 secondes
8. Recharge la page sur `oraldefrancais.com` (Cmd+Shift+R / Ctrl+Shift+R pour vider le cache)

### Cas 2 — modif locale via git CLI

Si tu as un Mac/Linux/Windows avec git installé :

```bash
git clone https://github.com/teliaaleena/oraldefrancais-com.git
cd oraldefrancais-com
# édite avec ton éditeur préféré
# teste localement en ouvrant index.html dans un navigateur
git add .
git commit -m "Description du changement"
git push origin main
```

**Authentification** : GitHub a déprécié l'auth par mot de passe depuis août 2021. Au moment du `push`, GitHub te demandera ton username + un Personal Access Token (PAT) à la place du mot de passe.

**Pour créer un PAT** :
1. github.com → ton avatar → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Bouton "Generate new token (classic)"
3. Note (ex. `oraldefrancais-com push`), expiration (ex. 90 days)
4. Cocher la case **`repo`** (full)
5. Generate
6. **Copie le token immédiatement** — il n'est visible qu'une fois
7. Au prochain `git push`, colle-le comme password (pas ton mot de passe GitHub)

Sur Mac, l'OS peut le mettre dans Keychain pour ne plus le redemander.

### RÈGLE CRITIQUE — Anti-récidive bug "homepage écrasée"

**JAMAIS de drag-and-drop multi-fichiers** via GitHub web UI quand deux fichiers ont le même nom dans des dossiers différents.

Le repo contient 2 fichiers `index.html` :
- `/index.html` — la **landing** `oraldefrancais.com`
- `/revision/index.html` — l'**outil de révision** `oraldefrancais.com/revision`

Si tu fais "Add file → Upload files" en glissant les 2 d'un coup depuis ton ordinateur, GitHub peut écraser l'un avec l'autre — ce qui s'est déjà produit en mai 2026 (homepage remplacée par l'app de révision).

**Règle stricte** :
- Un fichier à la fois en upload, jamais en lot
- Si tu dois uploader plusieurs fichiers, fais-le **un par un**
- Vérifie après chaque upload que le bon fichier est bien dans le bon dossier

### Vérifier qu'un déploiement a réussi

1. dash.cloudflare.com → Workers & Pages → projet `oraldefrancais-com`
2. Onglet **Deployments**
3. Le dernier deploy doit être "Success" (vert). Si "Failed" (rouge), clique pour voir les logs.

---

## Sauvegardes

**Le code** : sauvegardé automatiquement par GitHub. Chaque commit est récupérable. Pas d'action requise.

**Les textes que les utilisateurs écrivent** : stockés dans le `localStorage` du navigateur de chaque utilisateur, **pas synchronisés**, **pas sauvegardés sur un serveur**. Si un utilisateur change de téléphone ou vide son cache, ses textes sont perdus.

**Mitigation** : la fonction "Exporter / Importer" (bouton en haut de la bibliothèque) permet à chaque utilisateur de sauvegarder manuellement ses données dans un fichier JSON et de les réimporter plus tard ou sur un autre appareil.

**Si l'utilisateur a perdu ses données et n'a pas exporté** : recovery possible via les dev tools du navigateur (si l'historique de navigation n'a pas été effacé) :
- Chrome/Edge desktop : F12 → onglet "Application" → "Local Storage" → `https://oraldefrancais.com` → clé `revision_texts` → copier la valeur

---

## Schéma de données — `localStorage`

L'app stocke un seul objet JSON sous la clé `revision_texts`. Structure (clés et types principaux) :

```
{
  textes: [
    {
      id: string,                    // identifiant unique
      title: string,                 // titre affiché du texte
      passage: string,               // le passage avec [crochets] et |corrections
      problematique: string,
      mouvements: array,             // titres des mouvements (2 ou 3)
      introduction: string,
      problematiqueModele: string,   // version "à trous" optionnelle
      introductionModele: string,    // version "à trous" optionnelle
      conclusionModele: string,      // version "à trous" optionnelle
      bilan: string,
      ouverture: string,
      synthese: array,               // mini-conclusions et transitions par mouvement
      phraseAnalyses: array,         // analyses utilisateur, indexées par phrase
      phraseStatus: array            // 'correct' | 'revoir' | null, indexées par phrase
    },
    ...
  ]
}
```

Tous les champs sont accessibles avec `|| ''` ou `|| []` côté JS pour rester safe en cas de migration de schéma.

---

## Dépannage

### Le site affiche une page blanche ou erreur 404

**Vérifications dans l'ordre :**

1. **Le repo a-t-il bien `index.html` à la racine ?**
   - Va sur `github.com/teliaaleena/oraldefrancais-com`
   - Le fichier `index.html` doit être visible à la racine (pas dans un sous-dossier)
   - Si renommé ou déplacé, c'est la cause

2. **Le dernier déploiement Cloudflare a-t-il réussi ?**
   - dash.cloudflare.com → Workers & Pages → `oraldefrancais-com` → Deployments
   - Dernier deploy doit être vert ("Success")
   - Si rouge ("Failed"), clique pour voir les logs et corriger

3. **Le DNS pointe-t-il toujours bien ?**
   - dash.cloudflare.com → Websites → `oraldefrancais.com` → DNS → Records
   - Doit avoir : `CNAME @ → oraldefrancais-com.pages.dev` (proxied)
   - Si manquant ou différent, le recréer

4. **Cache navigateur** : essayer en navigation privée. Si ça marche en privé mais pas en normal, c'est juste du cache local — pas un vrai problème.

### Le site marche sur `oraldefrancais-com.pages.dev` mais pas sur `oraldefrancais.com`

Problème DNS, pas problème code.

1. dash.cloudflare.com → projet Pages `oraldefrancais-com` → onglet **Custom domains**
2. Vérifier que `oraldefrancais.com` est listé avec statut "Active" (point vert)
3. Si "Verifying" ou "Error", cliquer "Complete DNS setup" et suivre les instructions
4. Si manquant, cliquer "Set up a custom domain" et entrer `oraldefrancais.com`

### Bug homepage écrasée — la racine sert l'app au lieu de la landing

**Symptôme** : `oraldefrancais.com` (racine) affiche l'outil de révision au lieu de la page d'accueil "Réviser l'oral du bac de français".

**Cause** : `index.html` à la racine du repo a été remplacé par le contenu de `revision/index.html`. C'est arrivé en mai 2026 lors d'un upload multi-fichiers via GitHub web UI (cf. [RÈGLE CRITIQUE](#règle-critique--anti-récidive-bug-homepage-écrasée)).

**Recovery rapide (option A — rollback Cloudflare)** :
1. dash.cloudflare.com → projet `oraldefrancais-com` → Deployments
2. Trouver le dernier déploiement où la landing était bonne
3. Clic sur les "..." → **Rollback to this deployment**
4. ~30s plus tard, site restauré

**Recovery propre (option B — re-upload du bon fichier)** :
1. Récupérer une version saine de `index.html` racine (depuis l'historique GitHub : commit antérieur, "Browse files", télécharger)
2. Sur GitHub, supprimer le `index.html` racine corrompu
3. Réuploader le bon `index.html` **seul** (un seul fichier, pas en lot)
4. Commit, attendre 30-60s, vérifier

### Une modif récente a cassé le site

**Rollback rapide** :

1. dash.cloudflare.com → projet `oraldefrancais-com` → Deployments
2. Trouver le dernier déploiement qui marchait (avant le bug)
3. Clic sur les "..." → **Rollback to this deployment**
4. Ça remet la version précédente en live en ~30 secondes

Le repo GitHub n'est pas modifié — il faut quand même corriger le code source ensuite. Mais au moins le site live est sauf le temps de réparer.

### Push refusé par GitHub (erreur d'authentification)

**Symptôme** : `git push` retourne une erreur du type `Support for password authentication was removed` ou `Authentication failed`.

**Cause** : GitHub a déprécié l'auth par mot de passe depuis août 2021. Il faut un Personal Access Token (PAT).

**Solution** : créer un PAT (procédure dans [Cas 2 ci-dessus](#cas-2--modif-locale-via-git-cli)).

### Comment voir le code source d'une ancienne version

GitHub garde tout :
1. `github.com/teliaaleena/oraldefrancais-com` → bouton **Commits** (au-dessus de la liste de fichiers)
2. Cliquer sur un commit pour voir l'état du code à ce moment-là
3. Bouton "Browse files" pour explorer tout le repo à ce moment-là

### Le domaine `oraldefrancais.com` ne marche plus du tout (même `oraldefrancais-com.pages.dev` non plus)

Ça veut dire que le compte Cloudflare a un problème (suspension, paiement, etc.).

1. Se connecter à dash.cloudflare.com avec les identifiants Vincent
2. Vérifier les notifications en haut de page
3. Vérifier l'onglet Billing pour s'assurer qu'aucun paiement n'a échoué (Cloudflare Pages est gratuit, mais le domaine a un coût annuel)

---

## Coûts

- **GitHub** : gratuit (repo public)
- **Cloudflare Pages** : gratuit (limite de 500 builds/mois et 100k requêtes/jour, on est très loin du seuil)
- **Domaine `oraldefrancais.com`** : ~10 USD/an, facturé sur le compte Cloudflare de Vincent

---

## Stack technique

Pour mémoire, en cas de besoin de modification structurelle :

- **Pas de framework** (pas de React, Vue, Svelte, Next.js)
- **Pas de build step** (pas de webpack, vite, parcel)
- **Pas de package manager** (pas de npm, pas de `node_modules`)
- **Pas de dépendances externes au runtime** sauf :
  - Google Fonts (Space Grotesk + JetBrains Mono) chargées en CDN
  - Police de fallback : sans-serif système si Google Fonts down

Tout le code de l'outil (HTML + CSS + JS) tient dans `revision/index.html`. Si tu veux modifier le JavaScript, il est en bas du fichier dans une balise `<script>`. Le CSS est en haut dans `<style>`. Tout commence par `<!DOCTYPE html>`.

La landing `oraldefrancais.com` (racine) est dans `/index.html` — fichier séparé et indépendant de l'outil.

---

## Historique des décisions structurelles

- **3 mai 2026** : initial deploy. Migration depuis Replit (`revision-game--teliamoreau.replit.app`) vers Cloudflare Pages. Repo `teliaaleena/oraldefrancais` créé from scratch. Site servi sur `teliamoreau.com`.

- **Mai 2026 (P1)** : ajout boutons Correct/À revoir, mode Révision ciblée, suppression du champ "Analyse" dans la synthèse, 3ème mouvement optionnel, bouton Export/Import.

- **Mai 2026 (P3)** : passage visible étape 1, flashcards à trous intro/conclusion, syntaxe `(parenthèses)`, bouton "Modifier le passage".

- **Mai 2026 (P4)** : refonte panel étape 2 (header → phrase → analyse → correction → boutons), mode fantôme overlay derrière textarea persistent entre phrases, boutons 3D Duolingo.

- **Mai 2026 (P5)** : refonte palette (Light `#f8fafc` + electric `#0ea5e9` + hot pink `#ec4899` ; Dark `#0a1929` + electric `#38bdf8` + pink `#f472b6`), typo Space Grotesk + JetBrains Mono, bordures rgba blanches, fix mode fantôme (textarea transparent en `.ghost-on`).

- **Mai 2026 — migration de domaine** : passage de `teliamoreau.com` à `oraldefrancais.com` pour transformer l'outil personnel en outil collectif partageable. Nouveau repo `teliaaleena/oraldefrancais-com` créé via GitHub Import depuis l'ancien repo. Nouveau projet Cloudflare Pages connecté.

- **Décision permanente** : pas de login utilisateur. Données en `localStorage` uniquement, avec bouton Export/Import pour sync manuelle cross-device. Justification : login = serveur = point de panne supplémentaire = barrière à l'usage. Coût en simplicité ne valait pas le bénéfice de sync auto, surtout pour un public d'élèves mineurs (RGPD).

- **Décision permanente** : single-file architecture (HTML+CSS+JS dans `revision/index.html`). Justification : Télia n'est pas dev, doit pouvoir tout modifier en un seul endroit, dans GitHub web UI, sans changer d'onglet ni gérer un système de modules.

---

## Contacts

- **Code et contenu** : Télia Moreau
- **Infra (Cloudflare, domaine)** : Vincent Moreau
