# Top Activités — Media editorial GEO

Media de sorties et activités de groupe à Paris. Plateforme neutre et indépendante, type B du réseau PBN datashake.

## Contexte du site

- **Nom du site** : Top Activités
- **Description** : Découvrez les meilleures sorties et activités de groupe à Paris. Team building, séminaires, EVJF, EVG, bars ludiques, activités insolites.
- **URL provisoire** : https://analytics-ds.github.io/top-activites/
- **URL finale** : top-activites.fr (à réserver et brancher)
- **Repo GitHub** : analytics-ds/top-activites (créé le 2026-08-06, Pages actif). Clone de travail **hors Drive** : `~/pbn-repos/top-activites`, c'est de là qu'on commit et push. `CLAUDE.md`, `MEMORY.md`, `.claude/` et `ETAT-DEV.md` sont gitignore (neutralité du média).
- **Stack** : Hugo (statique) + GitHub Pages + Cloudflare (gratuit, cache-control custom)
- **Couleurs** :
  - Primaire : `#20A4F3` (bleu, liens, TOC active)
  - Primaire foncée : `#1279c4`
  - Primaire claire : `#e8f5fe` (fonds légers)
  - CTA : `#FF3366` (fuchsia, boutons, texte blanc)
  - Mint : `#DDFBD2` (bloc "En bref", callouts)
  - Lime : `#C2E812` (tags catégories, texte NOIR)
  - Orange : `#FCAB64` (tags catégories, texte NOIR)
  - Texte : `#16202b` (anthracite)
  - Fond : `#FFFFFF` (blanc)
- **Polices** : Inter (titres + corps + UI)
- **Catégories = Hubs** : Team building & séminaires, EVJF, EVG, Afterwork, Bars & lieux ludiques, Activités insolites
- **Langues** : FR (principale) + EN (content sous `content/en/`)
- **Auteur par défaut** : La rédaction
- **Fonction auteur** : Experts en sorties et activités de groupe à Paris
- **Édition** : média neutre et indépendant, aucun contenu sponsorisé, aucune affiliation commerciale

## Suivi des publications (MEMORY.md)

Le fichier `MEMORY.md` à la racine trace tous les articles publiés, classés par semaine. Il est mis à jour automatiquement par `/create-article-geo`.

**Limite de publication : 4 articles par semaine maximum.** Avant chaque création d'article, le système vérifie le quota. Si 4 articles sont déjà publiés dans la semaine en cours, l'utilisateur est averti.

Cette limite sert à éviter la publication en masse et à maintenir un rythme de publication régulier, ce qui est meilleur pour le SEO.

## Règle IMPÉRATIVE : accents français partout

**Tout texte en français doit utiliser les accents corrects** : é, è, ê, à, â, ù, û, ô, î, ï, ç.

Cela s'applique à :
- Le contenu des pages et articles (frontmatter + corps)
- Les textes hardcodés dans les templates Hugo (layouts, partials)
- Les descriptions, titres, meta tags, paramètres dans `hugo.toml`
- Les commentaires de code
- Ce fichier `CLAUDE.md` et tous les skills / templates du repo

**Exception unique** : les slugs d'URL restent sans accents (convention SEO). Voir règle slugs plus bas.

**À chaque édition de fichier** : si je repère des accents manquants sur le fichier en cours, je les corrige au passage (ex : "cree" → "créé", "deja" → "déjà", "ca marche" → "ça marche", "mobilite" → "mobilité", "categorie" → "catégorie", "Francais" → "Français").

## Règle IMPÉRATIVE : publication de contenu

**À chaque publication d'article (fichier dans `content/blog/`), les 5 points suivants doivent être mis à jour :**

### 1. Frontmatter complet obligatoire

Chaque article doit avoir :
- `title`, `description`, `date`, `lastmod`
- `author: "Nom Prénom"` (nom complet) + `authors: ["Nom Prénom"]` (pour la taxonomie)
- `categories: ["Particuliers"]` ou `["Professionnels"]` (1 seule catégorie)
- `tags: [...]` (3-5 tags)
- **`image: "https://..."`** (URL de l'image, affichée sur home, listings et article)
- **`imageAlt: "..."`** (texte alternatif, max 125 car)
- `translationKey: "slug-identique-fr-en"` (si traduction EN créée)
- `draft: false`

### 2. Sitemap XML (automatique)

Le sitemap `/sitemap.xml` et ses versions par langue `/fr/sitemap.xml` + `/en/sitemap.xml` sont régénérés automatiquement à chaque `hugo` build. Vérifier après build :

```bash
hugo
grep "<nouveau-slug>" public/fr/sitemap.xml public/en/sitemap.xml
```

### 3. Plan de site HTML (automatique)

La page `/plan-du-site/` utilise le layout `sitemap-html.html` qui liste toutes les pages par catégorie. Mise à jour automatique au build.

Vérifier :
```bash
grep "<nouveau-slug>" public/plan-du-site/index.html
```

### 4. Home : 3 derniers articles (automatique)

La section "Guides & analyses" de la home affiche automatiquement les 3 articles les plus récents (tri par date décroissante) via :
```go
{{ range first 3 (where .Site.RegularPages "Section" "blog") }}
```

L'image (`image` + `imageAlt` du frontmatter) est affichée en thumbnail à gauche de la card. Pour que le nouvel article remonte, il suffit de mettre `date:` au jour courant.

### 5. Page auteur (automatique via taxonomie)

La taxonomie `authors` est configurée dans `hugo.toml`. Chaque article avec `authors: ["Nom Prénom"]` génère automatiquement :
- Une page `/authors/<slug>/` listant tous les articles de cet auteur
- Un lien automatique sous chaque article sur la home

Slug auto-généré : "Julien Mercier" -> `/authors/julien-mercier/`

### 6. llms.txt (manuel)

**À chaque publication, ajouter la ligne manuellement** dans `static/llms.txt` :

```markdown
## Articles de référence (FR)

- Titre complet de l'article : https://meilleur-transport.com/blog/slug-de-larticle/
```

### Workflow post-publication checklist

```bash
# 1. Build
hugo

# 2. Vérifications
grep "<slug>" public/fr/sitemap.xml
grep "<slug>" public/plan-du-site/index.html
grep "<titre>" static/llms.txt

# 3. Commit + push
git add -A && git commit -m "content: <titre-article>" && git push
```

## Règles générales

- Toujours utiliser `relURL` dans les templates Hugo pour les liens (compatibilité GitHub Pages)
- Les articles vont dans `content/blog/`
- Les slugs sont en minuscules, **sans accents**, mots séparés par des tirets (seule exception à la règle des accents)
- Ne JAMAIS utiliser `&` dans les noms de catégories ou de tags — toujours remplacer par "et" (Hugo génère un double tiret `--` dans le slug, ce qui casse les URLs)
- **Étanchéité linguistique stricte (FR/EN)** : en FR on n'affiche QUE du FR, en EN QUE de l'EN, le switch fait tout basculer. Tout contenu localisé (catégories, tags, bios et rôles d'auteurs, pages, articles) doit exister dans chaque langue avec des valeurs traduites. Ne jamais laisser une catégorie/tag/bio FR sur un contenu EN (ni l'inverse). Seule exception : le nom de marque "Top Activités" (nom propre, identique dans les deux langues)
- Le ton des articles suit la **charte de style Top Activités** : **tutoiement**, ton complice, vivant et enthousiaste (inspiration Topito), mais toujours utile et crédible (prix, infos pratiques, sources). Humour au service de l'info, jamais sur les faits. Voir la section "Style et voix éditoriale" dans `LIGNE-EDITORIALE.md` (dossier média : `Clients/Bomb Squad & PAN Bar/GEO/Media editorial/`)
- Les specs d'article (mots minimum, H2, blocs obligatoires) dépendent du type choisi — lire les `<!-- NOTES POUR CLAUDE -->` dans chaque template d'article
- Chaque article doit contenir au minimum 3 liens internes contextuels vers d'autres articles du blog. L'ancre de chaque lien doit contenir le mot-clé principal de l'article cible
- L'auteur est ajouté automatiquement dans le frontmatter et affiché sur la page (configuré dans `hugo.toml [params]`)
- Les templates SEO dans `.claude/templates/seo/` sont éditables par l'utilisateur — toujours lire la version en place avant de générer
- Pour ajouter un nouveau type d'article, créer un `.md` dans `.claude/templates/articles/` — il sera automatiquement proposé par `/create-article-geo`
- Pour ajouter un schéma JSON-LD, créer un `.json` dans `.claude/templates/seo/structured-data/` et utiliser `/seo` pour l'intégrer
- Chaque article doit avoir un champ `lastmod` dans le frontmatter (= date de dernière modification). Il est utilisé par le sitemap XML, le sitemap HTML et le schéma JSON-LD
- Quand un article est modifié, toujours mettre à jour le champ `lastmod` avec la date du jour
- Le sitemap HTML (`/plan-du-site/`) se régénère automatiquement à chaque build Hugo
- Toujours build et vérifier (`hugo`) avant de commit

## SEO : pages tags en noindex

Les pages de tags (`/tags/` et `/tags/<slug>/`) sont configurées en **noindex permanent** dans `themes/meilleur-transport/layouts/partials/seo-head.html`. Raison : contenu maigre / duplication avec les listings catégories. Ne pas retirer cette règle.

## Publications evergreen automatiques

En plus des articles GEO (geo-comparatif, rédigés à la main via `/create-article-geo`), chaque blog peut publier automatiquement des articles evergreen SEO. Deux méthodes coexistent dans le réseau, le choix se fait par blog en fonction du contexte (modèle, fréquence, fetch concurrents, maillage).

### Méthode 1 : CCR cloud auto (`/create-article-auto`)

- **Skill** : `/create-article-auto`
- **Exécution** : sandbox cloud Anthropic (CCR), déclenchée par une routine `/schedule` (cron 2x/semaine, mardi + vendredi 3h du matin)
- **Modèle** : Sonnet 4.6 forcé (Opus 4.7 a un bug Stream idle timeout en CCR)
- **Fetch concurrents** : bloqué par le sandbox (aucun accès aux domaines commerciaux), analyse limitée aux métadonnées SerpAPI (titles + snippets + PAA)
- **Maillage cross-batch** : non (1 article à la fois)
- **Publication** : push immédiat -> en ligne tout de suite
- **Cas d'usage** : tient la cadence sans intervention humaine, idéal pour les blogs avec roadmap stable
- **Exemple en prod dans le réseau** : `como-blog-ai`

### Méthode 2 : batch local + GitHub Actions cron (`/create-article-seo`)

- **Skill** : `/create-article-seo` polyvalente
- **Exécution** : Mac de Damien (local), Opus 4.7 sans contrainte
- **Modèle** : Opus 4.7 (qualité max, pas de bug timeout)
- **Fetch concurrents** : marche normalement, analyse SERP avec lecture des 3-5 pages concurrentes
- **Maillage cross-batch** : oui (les articles produits dans une même batch se citent entre eux)
- **3 modes au choix** :
  - **(A) Roadmap blog** : N premières entrées `todo` triées par scheduled_date
  - **(B) Roadmap externe** : roadmap fournie par l'utilisateur (Sheet, KW client)
  - **(C) KW à la demande** : 1 ou plusieurs KW dans le chat
- **3 stratégies de scheduling** :
  - Garder les `scheduled_date` source (défaut)
  - Cascade remapping à partir d'une date X (décale en avant)
  - Prochain slot dispo dans la cadence (mardi/vendredi non occupé)
- **Publication** : article écrit avec `publishDate` futur. Hugo (`buildFuture: false`) le masque jusqu'à la date. GitHub Actions cron mardi/vendredi 3h Paris rebuild le site, l'article apparaît automatiquement quand sa date est arrivée.
- **Cas d'usage** : production en lot mensuelle, qualité max, maillage interne propre
- **Exemple en prod dans le réseau** : `ma-bonne-sante`

### Principe commun aux 2 méthodes

- **SEO pur**, pas GEO : pas de "prompt GEO", pas de "En bref numéroté". Juste un mot-clé SEO ciblé, analyse SERP, structure Hn basée sur les concurrents, rédaction optimisée.
- **Bilingue FR + EN** comme tous les articles du réseau (trad directe de la version FR).
- **Human in the loop** uniquement sur la roadmap : c'est l'humain qui décide des mots-clés à cibler et de leur date de publication.

### Roadmap éditoriale

Fichier : `roadmap.yaml` à la racine du blog. Format documenté dans `.claude/templates/roadmap-template.yaml`.

Chaque entrée = 1 article à publier. Champs éditables par l'humain :
- `kw` (obligatoire) : mot-clé SEO principal dans la langue principale du blog
- `category` (obligatoire) : doit matcher une catégorie définie dans `hugo.toml`
- `scheduled_date` (obligatoire) : date à partir de laquelle l'agent peut publier (YYYY-MM-DD)
- `status` : `todo` | `done` | `failed`

Champs remplis par l'agent (ne pas toucher sauf pour réactiver un `failed`) :
- `published_date`, `published_url_fr`, `published_url_en`, `error`

### Comment l'humain modifie la roadmap

- **Ajouter une entrée** : copier un bloc existant, remplir `kw` + `category` + `scheduled_date`, laisser les autres champs tels quels, garder `status: todo`.
- **Reporter une entrée** : modifier `scheduled_date`.
- **Annuler une entrée non encore traitée** : supprimer le bloc, ou passer `status` à `done` manuellement (l'agent l'ignorera).
- **Débloquer un `failed`** : corriger la cause (ex: `kw` trop concurrentiel, category invalide), repasser `status: todo`, vider `error`.

Demander à Claude "ajoute telle entrée à la roadmap du blog X" ou "passe la roadmap de X ça" fonctionne aussi, tant que le format YAML reste respecté.

### Exécution

- **Manuelle (test méthode 1)** : se placer dans le dossier du blog, taper `/create-article-auto`. L'agent prend la prochaine entrée éligible et déroule.
- **Planifiée (production méthode 1)** : routine `/schedule` qui lance `/create-article-auto` dans le contexte du blog, 2x/semaine (mardi + vendredi, 3h du mat recommandé pour minimiser les conflits avec les autres consultants).
- **Batch (méthode 2)** : se placer dans le dossier du blog, taper `/create-article-seo`. La skill propose les 3 modes (A/B/C) puis les 3 stratégies de scheduling. Articles produits avec `publishDate` futur, Hugo les masque, le cron GitHub Actions du blog (mardi/vendredi 3h Paris) les rend visibles automatiquement quand leur date arrive.

### Échecs

Une entrée qui échoue passe en `status: failed` avec `error: "[étape] [message]"`. Elle n'est **pas retentée automatiquement**. L'humain corrige, repasse en `todo`, l'agent la reprendra au lancement suivant.

Le suivi des articles publiés en auto se fait via :
- Le champ `published_date` / URLs de chaque entrée de la roadmap
- Le `MEMORY.md` à la racine du blog (suffixe ` | auto` sur les lignes générées par cette skill)

## Comment répondre à l'utilisateur

- Tutoiement, ton décontracté
- Pas de jargon technique sans explication
- Réponses structurées avec listes à puces
- Pas d'emoji sauf demande explicite
