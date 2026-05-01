# Audit du thème Dawn v15.4.1 — Personnalisation Premium

> **Branche :** `premium/phase-0`  
> **Date :** 2026-05-01  
> **Scope :** Documentation uniquement — aucune modification fonctionnelle

---

## 1. Résumé de l'audit

Dawn 15.4.1 est le thème de référence de Shopify Online Store 2.0 (OS 2.0). Il repose sur une architecture basée sur les **sections et blocs JSON**, les **Web Components natifs** (Custom Elements), et le pattern **pub/sub** interne pour la communication entre composants.

Points clés :

- **Architecture OS 2.0** : sections entièrement paramétrables via `config/settings_schema.json` et les fichiers `templates/*.json`.
- **JavaScript modulaire** : chaque fonctionnalité (panier, filtres, recherche, média différé…) est encapsulée dans un Custom Element autonome.
- **CSS BEM + utilitaires** : les styles sont organisés en composants (`component-*.css`) et en fichiers de base (`base.css`, `section-*.css`).
- **Performances** : utilisation du lazy loading, du `IntersectionObserver` et des animations configurables via `performance.js`.
- **Internationalisation** : 100 % des chaînes passent par `locales/*.json`.

---

## 2. Fichiers critiques à ne pas modifier brutalement

Ces fichiers constituent le cœur du thème. Toute modification directe risque de casser les mises à jour futures et l'intégrité du thème.

| Fichier | Rôle | Risque |
|---|---|---|
| `layout/theme.liquid` | Squelette HTML global, chargement des assets | Casse totale du thème si mal modifié |
| `assets/pubsub.js` | Bus d'événements inter-composants | Rompt toutes les communications (panier, filtres, recherche) |
| `assets/global.js` | Utilitaires partagés, helpers DOM | Casse les composants dépendants |
| `assets/constants.js` | Constantes globales (routes, events) | Incohérences silencieuses entre composants |
| `snippets/card-product.liquid` | Rendu des cartes produit | Régression sur toutes les collections et featured products |
| `snippets/facets.liquid` | Filtres de collection | Casse le système de filtrage URL-based |
| `config/settings_schema.json` | Schéma de personnalisation éditeur | Perte ou corruption des settings existants |
| `templates/*.json` | Structure des pages (blocs/sections) | Perte de la mise en page des pages |
| `locales/*.json` | Traductions | Erreurs d'affichage si clés supprimées |

---

## 3. Zones sûres de personnalisation

Ces zones permettent d'ajouter des fonctionnalités sans toucher au cœur du thème.

### 3.1 Fichiers nouveaux (approche append-only)
- Créer de **nouveaux snippets** avec le préfixe `custom_` ou `premium_` (ex : `snippets/premium_badge.liquid`).
- Créer de **nouvelles sections** avec le préfixe `custom_` ou `premium_` (ex : `sections/premium_hero-video.liquid`).
- Créer de **nouveaux assets JS/CSS** avec le préfixe `custom-` ou `premium-` (ex : `assets/premium-animations.css`).

### 3.2 Injection via `theme.liquid` (limitée et documentée)
- Ajouter des `{% render %}` de nouveaux snippets dans les zones de head/body uniquement **via commentaires balisés** :
  ```liquid
  {%- comment -%}PREMIUM INJECTION START{%- endcomment -%}
  {%- render 'premium_gtm-head' -%}
  {%- comment -%}PREMIUM INJECTION END{%- endcomment -%}
  ```

### 3.3 CSS Override sécurisé
- Créer `assets/premium-overrides.css` et le charger **après** `base.css` via un snippet dédié.
- Utiliser des sélecteurs CSS suffisamment spécifiques pour ne pas polluer le scope global.

### 3.4 Settings additionnels
- Ajouter de **nouveaux blocs** dans `config/settings_schema.json` **sans renommer ni supprimer** les entrées existantes.

---

## 4. Risques OS 2.0

| Risque | Description | Mitigation |
|---|---|---|
| **Conflit de settings** | Renommer ou supprimer un setting existant efface la valeur stockée en base pour tous les marchands | Ne jamais renommer, uniquement ajouter |
| **Casse des mises à jour Dawn** | Modifier directement les fichiers natifs rend impossible le merge des futures versions | Travailler uniquement sur des fichiers préfixés `premium_`/`custom_` |
| **Custom Elements en conflit** | Définir deux fois un `customElements.define('x', ...)` avec le même tag lève une erreur fatale | Vérifier l'existence avant définition : `if (!customElements.get('tag'))` |
| **Event pub/sub cassé** | Publier sur un canal existant avec un payload différent casse les abonnés natifs | Créer des canaux événementiels propres (ex : `premium:cart:updated`) |
| **Performance régression** | Charger des scripts synchrones dans `<head>` bloque le rendu | Toujours utiliser `defer` ou `type="module"` |
| **Hydratation côté client** | Les sections dynamiques utilisent `requestAnimationFrame` et `IntersectionObserver` — les surcharger casse l'expérience | Ne jamais redéfinir ces observateurs globalement |

---

## 5. Roadmap premium recommandée en phases

### Phase 0 — Documentation (actuelle)
- [ ] Audit complet du thème (ce fichier)
- [ ] Cartographie des zones de personnalisation
- [ ] Définition des règles de développement

### Phase 1 — Infrastructure CSS/JS Premium
- [ ] Création de `assets/premium-base.css` (variables CSS custom, reset premium)
- [ ] Création de `assets/premium-components.css` (composants visuels)
- [ ] Création de `snippets/premium_styles.liquid` (injection CSS conditionnelle)
- [ ] Création de `snippets/premium_scripts.liquid` (injection JS conditionnelle)
- [ ] Ajout des settings premium dans `config/settings_schema.json` (section dédiée)

### Phase 2 — Composants visuels
- [ ] Badge produit premium (`snippets/premium_badge.liquid`)
- [ ] Hero vidéo (`sections/premium_hero-video.liquid`)
- [ ] Bandeau promo animé (`sections/premium_promo-bar.liquid`)
- [ ] Grille produits enrichie (overlay hover, quick add amélioré)

### Phase 3 — Fonctionnalités e-commerce avancées
- [ ] Sélecteur de variantes enrichi (swatches couleur/taille)
- [ ] Panier slide-over amélioré (upsell, cross-sell)
- [ ] Système de wishlist (Custom Element + localStorage)
- [ ] Compteur stock urgence (`premium_stock-counter.liquid`)

### Phase 4 — Performance & SEO
- [ ] Optimisation des images (lazy-load avancé, formats modernes)
- [ ] Structured data enrichi (JSON-LD produits, fil d'Ariane)
- [ ] Critical CSS inline pour above-the-fold
- [ ] Préchargement des pages de collection

### Phase 5 — Analytics & Personnalisation
- [ ] Intégration GTM/GA4 via snippets dédiés
- [ ] A/B testing infrastructure (feature flags Liquid)
- [ ] Recommandations produits dynamiques

---

## 6. Règles de développement pour les prochaines phases

Ces règles sont **obligatoires** pour toutes les phases suivantes et doivent être respectées par tous les contributeurs.

### 6.1 Approche append-only
- **Ne jamais supprimer** de fichiers Dawn existants.
- **Ne jamais renommer** de fichiers Dawn existants.
- Toute nouvelle fonctionnalité est ajoutée dans de **nouveaux fichiers** ou via des **injections balisées** dans les fichiers existants.

### 6.2 Préfixage obligatoire
- Tout nouveau fichier Liquid, JS ou CSS doit être préfixé `custom_` ou `premium_` :
  - `snippets/premium_badge.liquid` ✅
  - `assets/premium-animations.js` ✅
  - `snippets/badge.liquid` ❌
  - `assets/animations.js` ❌

### 6.3 Settings schema
- **Ne jamais renommer** un setting existant dans `config/settings_schema.json`.
- **Ne jamais supprimer** un setting existant.
- Ajouter les nouveaux settings dans une **section dédiée** clairement identifiée `"name": "Premium"`.

### 6.4 Fichiers Dawn intouchables (modification directe interdite)
Les fichiers suivants ne doivent **jamais** être modifiés directement :

```
assets/pubsub.js
assets/constants.js
assets/global.js
snippets/card-product.liquid
snippets/facets.liquid
```

Si une modification est impérativement nécessaire sur ces fichiers, elle doit :
1. Faire l'objet d'une **issue documentée** avec justification.
2. Être approuvée en revue de code.
3. Être isolée dans un commit séparé avec le message `feat(breaking): modify Dawn core file <nom_fichier>`.

### 6.5 JavaScript Premium
- Toujours vérifier l'existence d'un Custom Element avant de le définir :
  ```js
  if (!customElements.get('premium-badge')) {
    customElements.define('premium-badge', PremiumBadge);
  }
  ```
- Utiliser des canaux pub/sub dédiés avec le namespace `premium:` :
  ```js
  publish('premium:wishlist:updated', { productId });
  ```
- Charger tous les scripts avec `defer` ou `type="module"`.

### 6.6 Versionning et branches
- Chaque phase correspond à une branche dédiée : `premium/phase-1`, `premium/phase-2`, etc.
- Les PR sont toujours ouvertes vers `main`.
- Les commits suivent la convention **Conventional Commits** :
  - `feat(premium): add badge snippet`
  - `style(premium): add base CSS variables`
  - `docs(premium): update AUDIT.md`

---

*Document maintenu par l'équipe premium. Dernière mise à jour : 2026-05-01.*
