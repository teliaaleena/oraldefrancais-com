# TODO — oraldefrancais.com

État au **4 mai 2026**, après livraison Bundle 9.

Document interne. Ce qui est fait, ce qui reste à faire, ce qui a été décidé de ne pas faire.

---

## Faits récemment (Bundle 9 — 4 mai 2026)

- Logo : carré rose remplacé par wordmark `oraldefrancais.com` avec "oral" surligné rose rotated/skewed (Direction C)
- Favicon : nouveau "o" italique blanc dans carré rose rotated -6°
- Palette `TryMe` ajoutée : 6e option qui tire 1 palette parmi 20 combos curés à chaque clic
- Bouton "Re-tirer" topbar revision visible quand TryMe est actif
- Variables CSS canon ajoutées (`--t-fast/base/slow`, `--ease-spring`)
- 47 transitions hardcoded migrées vers les variables canon
- Box-shadow modal Roulette tinté avec `ink` au lieu de noir générique
- Lisibilité sous-titres swatches améliorée (11px + ink-2)

---

## Chantiers ouverts (par priorité)

### P1 — Contenu

- [ ] **Ajouter les textes complets aux 12 œuvres voie générale** dans un fichier `content/textes-shared.json` (ou intégré à `programme-2026.json`). Aujourd'hui, quand Télia importe une œuvre depuis le programme, elle reçoit une carte vide à remplir. Les textes complets permettraient un import "prêt à réviser".
- [ ] Vérifier l'orthographe des œuvres et auteurs dans `programme-2026.json` (parcours Eduscol officiel)

### P2 — Modes de révision

- [ ] **Mode P6 oral** : entraînement chronométré 8 min par texte (timer visible, micro pour s'enregistrer)
- [ ] **Mode P7 entretien** : questions BCIS / Eduscol affichées en flashcards avec spaced repetition légère
- [ ] **Mode P8 sprint** : 3 phrases au hasard parmi tous les textes importés, en 90 secondes

### P3 — UX / accessibilité

- [ ] Audit lecteur d'écran (VoiceOver iOS) — Télia ne l'utilise pas mais d'autres élèves pourraient
- [ ] Mode impression CSS pour exporter une fiche révision en PDF propre
- [ ] PWA : manifest + service worker pour permettre install sur écran d'accueil iOS et offline-first
  - Décision en attente sur le timing — à voir après les exams 2026 ?

### P4 — Site personnel Télia

- [ ] **Refonte `teliamoreau.com`** en page perso simple (bio, projets, lien vers oraldefrancais.com)
- [ ] Décider du sort du repo `github.com/teliaaleena/oraldefrancais` (l'ancien) qui sert encore `teliamoreau.com` — soit 404 propre, soit redirect 301 vers `oraldefrancais.com/revision/`

### P5 — Polish mineur

- [ ] Sur les cards seedées dans la bibliothèque (depuis le programme), décider si on affiche aussi auteur + parcours en plus du titre
- [ ] Empty state library plus riche quand 0 texte importé (CTAs visuels Au programme / Texte libre directement dans la zone vide)

---

## Décisions arrêtées (ne pas re-litiger)

- **Pas de framework.** HTML/CSS/JS vanilla, point. Aucune migration vers React, Vue, Astro, Vite, Next, etc.
- **Pas de build step.** Edit un fichier, push, c'est en ligne. Le confort dev d'un bundler ne vaut pas le coût de maintenance pour des non-coders.
- **Pas de tracking, pas d'analytics, pas de cookies, pas de comptes.** Site statique privé-par-défaut. Le contenu vit en `localStorage` côté client.
- **Pas de palette DIY** (color picker libre). Remplacé par TryMe (20 combos curés tirés au hasard) — limite le risque combo illisible.
- **Pas d'emoji** nulle part dans l'UI, le code, les commits.
- **Logo : Direction C (surligneur).** Le logo image gaming/néon générée par DALL-E proposée le 3 mai a été rejetée (gaming, néons, drapeau français, Tour Eiffel, 6 promesses, non-reproductible, codé masculin) — voir conversation Bundle 8/9 pour le détail.
- **Pop-up onboarding 2 étapes** sur la homepage (Ambiance + Identité). Voie + Œuvres déplacés dans `/revision/`.
- **Force Pop par défaut au 1er chargement homepage**, ignore `prefers-color-scheme: dark`. Override seulement si choix explicite mémorisé.

---

## Bugs connus à surveiller

Aucun bug ouvert connu après Bundle 9. Les bugs corrigés au fil des bundles :

- [Bundle 6→7] Modal "Nouveau texte" très moche (SVG énormes, eyebrows all-caps) → refonte sur pattern `.nt-*`
- [Bundle 8] `@media (max-width: 600px)` ouvert depuis Bundle 5 jamais refermé, CSS suivant imbriqué dans media query → corrigé
- [Bundle 9] `ONB_TRYME_PALETTES` en TDZ référencé par `onbInit` IIFE avant déclaration → corrigé via `Promise.resolve().then()` pour différer
- [Bundle 9] Webhook GitHub → Cloudflare cassé tant que app `Cloudflare Workers and Pages` non installée → résolu le 4 mai 2026

---

## Notes de schéma `content/programme-2026.json`

Structure attendue par `revision/index.html` :

```json
{
  "voies": {
    "generale": {
      "label": "Voie générale",
      "oeuvres": [
        {
          "id": "musset-badine",
          "objet": "theatre",
          "titre": "On ne badine pas avec l'amour",
          "auteur": "Alfred de Musset",
          "parcours": "Les jeux du cœur et de la parole"
        }
      ]
    },
    "techno": { ... }
  },
  "objets_etude": [
    { "id": "theatre", "label": "Théâtre" },
    { "id": "poesie",  "label": "Poésie" },
    ...
  ]
}
```

**Pour ajouter une œuvre :**
1. Trouver le bon `objet` (théâtre, poésie, roman, littérature d'idées)
2. Ajouter un objet dans le tableau `oeuvres` de la bonne voie
3. `id` doit être unique, en kebab-case sans accents : `auteur-titre-court`
4. Commit, attendre 60s, recharger `oraldefrancais.com/revision/` → "Nouveau texte" → "Au programme" → l'œuvre apparaît
