[README.md](https://github.com/user-attachments/files/27384911/README.md)
# oraldefrancais.com — site

Site statique de révision de l'oral de français (EAF). 3 pages + 1 page d'erreur :

- **`/` (`index.html` à la racine)** — Landing + onboarding modal (1ʳᵉ visite).
- **`/faq/`** — FAQ et programme bac de français 2026.
- **`/revision/`** — Outil de révision (cœur de l'app).
- **`/404.html`** — Page d'erreur (servie auto par Cloudflare Pages quand l'URL n'existe pas).

Pas de framework, pas de build step. HTML + CSS + JS vanilla. Cloudflare Pages auto-deploy au push sur `main`.

## Structure

```
/
├── index.html               ← landing + onboarding
├── faq/
│   └── index.html           ← FAQ + programme
├── revision/
│   └── index.html           ← outil de révision (cœur ~6700 lignes)
├── 404.html                 ← page d'erreur (servie par Cloudflare Pages)
├── content/
│   └── programme-2026.json  ← données du programme bac (lu par /faq et /revision)
├── favicon.svg              ← icône du site
├── og-image.svg             ← image partage social (Open Graph)
├── README.md                ← ce fichier
├── DEPLOYMENT.md            ← procédure de déploiement et dépannage
└── TODO.md                  ← chantiers ouverts et décisions arrêtées
```

Pas de monorepo, pas de workspaces, pas de package.json. Si tu peux ouvrir un fichier dans GitHub web UI et le modifier, tu peux maintenir le site.

## Comment ajouter / modifier un texte

Les textes (Le Cid, L'Albatros, etc.) sont stockés dans le `localStorage` de chaque navigateur — il n'y a **pas de base de données partagée**. Chaque utilisateur a sa propre bibliothèque locale.

L'app expose dans Outils > Importer / Exporter une fonction d'export JSON pour sauvegarder la bibliothèque, et d'import pour la restaurer (ou la partager entre 2 appareils).

### Schema d'un texte

```json
{
  "id": 1730000000000,
  "title": "L'Albatros",
  "passage": "Souvent, pour s'amuser...",
  "problematique": "...",
  "introduction": "...",
  "bilan": "...",
  "ouverture": "...",
  "synthese": ["...", "...", "..."],
  "phraseAnalyses": ["...", "..."],
  "phraseStatus": [0, 1, 2],
  "mouvements": "I. Titre\nII. Titre",
  "mouvementsData": [
    { "numero": 1, "titre": "Élévation", "startOffset": 0, "endOffset": 145 },
    { "numero": 2, "titre": "Chute", "startOffset": 145, "endOffset": 320 }
  ],
  "problematiqueModele": "...",
  "introductionModele": "...",
  "conclusionModele": "..."
}
```

`phraseStatus` : 0 = à faire, 1 = à revoir, 2 = fait.
`mouvementsData` : nouveau format avec offsets exacts dans le passage. `mouvements` : ancien format (texte libre), gardé pour compat.

### Format des phrases-clés dans le passage

Pour marquer un mot ou groupe de mots comme **phrase-clé** (rendu en surligné dans l'UI) :

```
[mot ou groupe à mémoriser|correction (synonyme)]
```

- `[mot]` (sans `|`) : juste un surlignage
- `[mot|correction]` : surlignage + correction affichée au clic dans la flashcard
- `(mot entre parenthèses)` : trous à compléter dans la flashcard à trous

Exemple :
```
Charles Baudelaire publie [L'Albatros|son poème emblématique] en (1859), dans (Les Fleurs du Mal).
```

## Module Ambiance — duplication assumée

Les 3 pages partagent un **menu Ambiance** dans le header (5 ambiances fixes : Pop, Midnight, Sunset, Cyber, Matcha + 1 surprise : **TryMe** qui tire l'une des 20 palettes nouvelles à chaque clic).

**⚠️ Le code Ambiance complet (palettes + helpers) est dupliqué dans les 4 fichiers HTML** parce qu'on n'a pas de système d'import partagé en HTML pur. Si tu modifies une palette ou ajoutes/supprimes une palette, tu dois faire le changement dans **4 endroits** :

| Fichier | Cherche cette ligne |
|---|---|
| `index.html` (landing) | `const TRYME_PALETTES = [` |
| `faq/index.html` | `const TRYME_PALETTES = [` |
| `revision/index.html` | `const TRYME_PALETTES = [` |
| `404.html` | `const TRYME_PALETTES = [` |

Le bloc fait ~22 lignes (20 palettes + commentaires). Les 4 doivent être **strictement identiques** sinon Télia voit "Sakura vif" sur l'une et autre chose sur l'autre.

Cherche aussi le bloc commentaire `AMBIANCE & TRYME — bloc partagé` dans chaque fichier : il marque le début et la fin du bloc à synchroniser.

### Helpers JS partagés (à garder synchronisés)

Dans le bloc `AMBIANCE & TRYME — bloc partagé` :

| Fonction | Rôle |
|---|---|
| `AMBIANCE_VALUES`, `AMBIANCE_LEGACY` | Liste des thèmes valides + migration "light"→"pop" |
| `TRYME_PALETTES` | Les 20 palettes (objet par palette, 13 tokens) |
| `pickTryMePaletteIdx()` | Tire un index aléatoire en évitant le précédent |
| `applyTryMePalette(idx)` | Applique les 13 tokens d'une palette aux variables CSS root |
| `clearTryMePalette()` | Retire les overrides → retour ambiance fixe |
| `rerollTryMe()` | Tire + applique une nouvelle palette |
| `setTheme(theme)` | Active une ambiance (fixe ou tryme) + persiste |
| `updateAmbianceButtons(theme)` | Sync l'état visuel actif des boutons popover |
| `updateThemeColorMeta(theme)` | Sync `<meta theme-color>` (barre statut iOS/Android) |
| `toggleAmbiancePopover()` / `closeAmbiancePopover()` | Ouvre/ferme le popover |

### Schema d'une palette

```js
{
  label: 'Sakura vif',
  cream: '#fdf2f8',
  cream2: '#fbcfe8',
  cream3: '#f9a8d4',
  white: '#fffafd',
  whiteLine: '#f7c2dc',
  ink: '#500724',
  ink2: '#831843',
  ink3: '#9d174d',
  electric: '#be185d',
  pink: '#db2777'
}
```

- `cream` : fond principal
- `cream2` : cards / panneaux
- `cream3` : bordures / séparateurs
- `white` : cards blanches / inputs
- `whiteLine` : bordures cards
- `ink` : texte principal (le plus contrasté)
- `ink2` : texte secondaire
- `ink3` : texte tertiaire (hints)
- `electric` : titres, key phrases (couleur "sérieuse")
- `pink` : CTA, progressions (couleur "vive")

**Règle contraste** : ratio ink/cream doit être ≥ 4.5:1 (WCAG AA). Pour vérifier : https://webaim.org/resources/contrastchecker/

### Si tu veux ajouter une 21ᵉ palette

1. Ouvre les 4 fichiers HTML ci-dessus
2. Cherche `const TRYME_PALETTES = [`
3. Ajoute la nouvelle palette **dans les 4 fichiers** avec exactement le même contenu (copier-coller)
4. Vérifie le contraste ink/cream ≥ 4.5
5. Commit + push → auto-deploy Cloudflare

### Si tu veux changer les 5 ambiances fixes

Elles sont définies en CSS via `[data-theme="pop|midnight|sunset|cyber|matcha"]` dans chaque `<style>` des 4 fichiers. Même règle : modifier les 4 fichiers en parallèle.

### Comportement par défaut au premier chargement

Si l'utilisateur n'a jamais choisi d'ambiance (`localStorage.theme` vide) :
- Si son OS est en mode sombre (`prefers-color-scheme: dark`) → Midnight
- Sinon → Pop

Dès qu'il clique une ambiance, son choix est sauvé dans `localStorage.theme` et la détection système ne reprend plus la main. Le choix persiste sur toutes les pages du site (même origin = même `localStorage`), 404 incluse.

## Bugs connus & réparations

### "Mon texte s'affiche un mot par ligne"

Cause : copier-coller depuis un PDF avec colonne étroite. Chaque mot est sur sa propre ligne dans le passage source.

Réparation auto : sur `/revision/`, étape 1, une notice jaune "Ce passage est mal mis en forme" apparaît au-dessus du passage cassé avec un bouton "Réparer le passage" qui :
- Joint toutes les lignes en une
- Nettoie les espaces avant ponctuation française
- Reset les mouvements posés (les offsets ne sont plus valides après normalisation)

La même normalisation est appliquée automatiquement à chaque save.

Heuristique de détection : si moyenne mots/ligne < 1.5 sur ≥ 5 lignes → considéré cassé.

## Déploiement

Push sur `main` → Cloudflare Pages déploie automatiquement en 30-60s.

- Repo : `github.com/teliaaleena/oraldefrancais-com`
- Cloudflare Pages settings : framework=None, build command=(empty), output=`/`, root=`/`, branch=`main`
- Domaine : `oraldefrancais.com` (DNS Cloudflare, compte Vincent)

Voir `DEPLOYMENT.md` pour les détails complets de setup (à créer si besoin).

## Workflow recommandé pour modifications

Pour une modif simple (texte, couleur, copy) :
1. GitHub web UI → fichier concerné → Edit (icône crayon)
2. Modifier en place
3. "Commit changes" en bas de page → message de commit court
4. Cloudflare Pages déploie auto en 30-60s
5. Hard refresh sur le navigateur (Cmd+Shift+R Mac / Ctrl+Shift+R PC) pour voir

Pour une modif qui touche le module Ambiance (palette, helpers, ambiance fixe) :
- Faire les 4 commits l'un après l'autre (index, faq, revision, 404), vérifier après chaque que le site marche encore
- Ne **jamais** faire un seul commit qui modifie les 4 fichiers en même temps : si l'un casse, les 4 sont déployés ensemble et tout est en panne
