# DEPLOYMENT — oraldefrancais.com

Comment le site se déploie, où sont les comptes, quoi faire si quelque chose casse.

> Lis ce document avant de toucher à quoi que ce soit. Les comptes sont chez deux personnes différentes par sécurité.

---

## Ownership

| Quoi | Qui | Compte |
|---|---|---|
| Repo GitHub (code + contenu) | Télia | `github.com/teliaaleena` |
| Domaine `oraldefrancais.com` | Vincent | Cloudflare (compte personnel Vincent) |
| Cloudflare Pages (hébergement) | Vincent | Même compte Cloudflare |
| DNS | Cloudflare (auto, sur compte Vincent) | — |

**Pourquoi ce split :** Télia peut éditer le code/contenu sans toucher au compte de Vincent ; Vincent gère DNS/billing sans toucher à celui de Télia. Transférable plus tard si Télia veut tout reprendre.

---

## Repo

- **URL** : `github.com/teliaaleena/oraldefrancais-com`
- **Visibilité** : public
- **Branche de production** : `main`
- **Stack** : HTML + CSS + JS vanilla. Aucun build, aucun `npm install`, aucun framework. Edit un fichier, push, c'est en ligne.

### Structure du repo

```
oraldefrancais-com/
├── index.html              ← homepage (pop-up onboarding 2 étapes)
├── 404.html                ← page d'erreur (servie auto si URL inconnue)
├── faq/
│   └── index.html          ← page FAQ
├── revision/
│   └── index.html          ← outil de révision (le plus gros fichier ~6700 lignes)
├── content/
│   └── programme-2026.json ← liste des œuvres voie générale + voie techno + FAQ
├── favicon.svg             ← icône onglet navigateur
├── DEPLOYMENT.md           ← ce fichier
├── TODO.md                 ← idées et chantiers ouverts
└── README.md               ← présentation du projet (schemas, helpers, bugs connus)
```

---

## Comment publier une modification

### Option A — Édition rapide via GitHub web UI (recommandé pour Télia)

1. Ouvrir `github.com/teliaaleena/oraldefrancais-com`
2. Cliquer sur le fichier à éditer (par exemple `content/programme-2026.json` pour ajouter une œuvre)
3. Cliquer sur l'icône crayon en haut à droite ("Edit this file")
4. Modifier le texte
5. Descendre tout en bas, écrire un message de commit court (par exemple `add: nouvelle œuvre Baudelaire`)
6. Cliquer **Commit changes**
7. Attendre 30 à 60 secondes
8. Recharger `oraldefrancais.com` — la modification est en ligne

### Option B — Upload de fichier complet (pour des changements plus gros)

1. Sur GitHub, naviguer dans le dossier où le fichier doit aller
2. Cliquer **Add file → Upload files**
3. Glisser-déposer le fichier (il **remplace** s'il existe déjà avec le même nom)
4. Écrire un message de commit
5. **Commit changes**

---

## Modifications qui touchent plusieurs fichiers — règle d'or

Le module **Ambiance** (palettes Pop/Midnight/Sunset/Cyber/Matcha + TryMe avec ses 20 palettes) est volontairement **dupliqué dans 3 fichiers** : `index.html`, `faq/index.html`, `revision/index.html`. Pas de système d'import partagé en HTML pur.

Si tu changes une couleur de thème ou une palette TryMe, tu dois faire le changement dans **les 3 fichiers**, sinon Télia voit "Sakura vif" sur revision et autre chose sur la FAQ.

**Workflow obligatoire :**
1. Modifier le 1er fichier (par ex. `revision/index.html`) → commit → attendre déploiement → vérifier live
2. Si OK, modifier le 2ᵉ fichier → commit → vérifier
3. Pareil pour le 3ᵉ
4. **Ne jamais commit les 3 fichiers en un seul commit** : si l'un casse, les 3 partent en prod en même temps et tout est en panne

Détail du module et de ses helpers : voir `README.md`, section "Module Ambiance — duplication assumée".

---

## Auto-deploy Cloudflare Pages

Le webhook GitHub → Cloudflare est actif depuis le **4 mai 2026** grâce à l'app GitHub `Cloudflare Workers and Pages` installée par Télia sur son compte.

**Ce qui se passe à chaque commit sur `main` :**
1. GitHub notifie Cloudflare Pages
2. Cloudflare clone le repo, copie les fichiers tels quels (pas de build)
3. Le site est en ligne sur `oraldefrancais.com` en 30-60 secondes

**Configuration Cloudflare Pages** (à ne pas modifier sans raison) :
- Framework preset : `None`
- Build command : *(vide)*
- Build output directory : `/`
- Root directory : `/`
- Production branch : `main`

### Page 404

Cloudflare Pages sert **automatiquement** le fichier `404.html` à la racine quand une URL demandée n'existe pas (pas besoin de configuration). Pour modifier l'apparence : éditer `/404.html` directement.

### Redirections

Si tu veux rediriger une URL (ex : `/ancien-chemin/` → `/nouveau/`), créer un fichier `_redirects` à la racine du repo avec :

```
/ancien-chemin/* /nouveau/:splat 301
```

Cloudflare Pages le lit automatiquement. Voir doc : `developers.cloudflare.com/pages/configuration/redirects/`. Pour l'instant, pas de fichier `_redirects` dans le repo — on le créera quand on en aura besoin.

---

## Comportement cross-page

Les 3 pages partagent `localStorage` (même origine `oraldefrancais.com`). Donc :

- Si Télia choisit l'ambiance "Synthwave" sur `/revision/`, elle reste en Synthwave en allant sur `/faq/` et sur `/`
- Le choix est persistant entre les visites (jusqu'à ce qu'elle vide le cache)
- Au premier chargement (avant tout choix), le site détecte `prefers-color-scheme` du système : dark → Midnight, sinon → Pop

Les **textes de la bibliothèque** sont aussi dans le `localStorage` du navigateur de chaque utilisateur — pas de base de données partagée. Pour exporter / réimporter une bibliothèque (changement de téléphone, sauvegarde) : `/revision/` → menu **Outils → Importer / Exporter**.

---

## Si le déploiement ne se fait pas

### Étape 1 — Vérifier sur Cloudflare

1. Vincent se connecte sur `dash.cloudflare.com`
2. Workers & Pages → projet `oraldefrancais-com` → onglet **Deployments**
3. Vérifier le dernier déploiement :
   - Statut **Success** vert : tout va bien, vide le cache navigateur (Ctrl+Shift+R) si tu ne vois pas la modif
   - Statut **Failed** rouge : cliquer dessus pour voir le log
   - **Aucun nouveau déploiement** : le webhook a peut-être sauté

### Étape 2 — Si le webhook a sauté (rare)

1. Sur Cloudflare → projet → **Deployments**
2. Sur la dernière ligne (le déploiement en cours ou le dernier réussi), cliquer le bouton **"..."** à droite
3. Cliquer **Retry deployment**
4. Cloudflare relance le build en récupérant la dernière version du repo
5. Si ça échoue encore, vérifier que l'app GitHub est toujours installée :
   - `github.com/settings/installations` (compte Télia)
   - L'app **"Cloudflare Workers and Pages"** doit apparaître
   - Si elle a disparu, la réinstaller : `https://github.com/apps/cloudflare-workers-and-pages` → Configure → teliaaleena → Only select repositories → `oraldefrancais-com`

### Étape 3 — Si le site répond 404 partout (plus aucune page accessible)

1. Tester directement le sous-domaine Cloudflare : `oraldefrancais-com.pages.dev` (sans le custom domain)
2. Si ça marche → problème DNS sur le custom domain. Cloudflare → DNS → vérifier l'enregistrement `oraldefrancais.com` pointe sur Pages
3. Si même `pages.dev` répond 404 → le déploiement a échoué, voir étape 2

> ⚠️ Attention : un 404 sur **une URL inconnue** (ex. `/blabla`) est normal — c'est `404.html` qui s'affiche. Le problème, c'est si même `/` ou `/revision/` répond 404.

---

## Si je casse une page (mauvais commit)

Pas de panique. Tout est versionné sur GitHub.

1. `github.com/teliaaleena/oraldefrancais-com/commits/main`
2. Trouver le dernier commit qui marchait (avant le commit cassé)
3. Cliquer dessus → bouton **"<>"** (browse files at this point)
4. Naviguer vers le fichier qui posait problème
5. Cliquer crayon → copier tout le contenu
6. Retour sur le fichier dans `main` (la version cassée)
7. Crayon → tout remplacer par le contenu copié
8. Commit `revert: rollback fichier X to commit YYYY`

Le site se redéploie en 30-60s avec l'ancienne version qui marchait.

---

## Règles d'or

- **Ne jamais éditer plusieurs fichiers en même temps** si tu n'es pas sûre. Un fichier à la fois, vérifier que ça marche en ligne, puis fichier suivant.
- **Toujours tester sur mobile** (375px largeur) après une modification visible — Télia révise sur son téléphone.
- **Le contenu vit dans `content/programme-2026.json`** — pas besoin de toucher au HTML pour ajouter une œuvre.
- **Pas de framework, pas de build step, pas de `npm install`.** Si quelqu'un te propose d'ajouter React, Vite, Astro, dis non. C'est volontaire pour que tu puisses maintenir le site dans 6 mois.
- **Module Ambiance dupliqué dans 3 fichiers** : si tu changes une palette, fais-le dans les 3, un commit par fichier (voir section dédiée plus haut).

---

## Contacts d'urgence

- Repo cassé / question de code → demande à ton père Vincent de relancer une session avec Claude
- Domaine / DNS / facturation Cloudflare → Vincent
- Compte GitHub piraté → support GitHub : `support.github.com`
- Compte Cloudflare piraté → support Cloudflare : `support.cloudflare.com`
