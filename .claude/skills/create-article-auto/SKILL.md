# Skill : article evergreen quotidien de Top Activités (full auto)

Produit **automatiquement** un article evergreen bilingue FR + EN à partir du mot-clé le plus ancien encore `todo` dans `roadmap.yaml`, et le publie sur GitHub. Aucun input humain, aucun point d'arrêt.

Adaptée de la skill du même nom des autres blogs du réseau, recalée sur les spécificités de Top Activités le 2026-09-14. **Elle tourne en local sur le Mac** (LaunchAgent `com.datashake.topactivites.quotidien`), pas dans le sandbox cloud, donc Hugo est déjà installé et le réseau n'est pas filtré.

## Cadence

**Un article par jour**, arbitrage de Charlie du 2026-09-14 (le budget du projet ne permet pas d'étaler). Cette skill **ignore** la règle indicative des 4 articles par semaine du réseau, qui ne vaut que pour les skills interactives. Un seul article par exécution, jamais deux.

## Étape 0, sélection de l'entrée

```bash
cd ~/pbn-repos/top-activites && git pull --rebase origin main
```

Lire `roadmap.yaml`, filtrer `status == todo` et `scheduled_date <= today`, trier par `scheduled_date` croissante, prendre la première. Aucune entrée éligible, logger et sortir en 0. Une entrée `failed` n'est jamais reprise automatiquement.

## Étape 1, SERP

Clé dans le `.env` du workspace, jamais dans le repo (public) :

```bash
CRAZYSERP_API_KEY=$(grep '^CRAZYSERP_API_KEY=' "$WORKSPACE/.claude/secrets/.env" | cut -d= -f2-)
curl -s --max-time 240 -G "https://crazyserp.com/api/search" \
  --data-urlencode "q=<kw>" --data-urlencode "gl=fr" --data-urlencode "hl=fr" \
  --data-urlencode "location=France" --data-urlencode "page=1" \
  -H "Authorization: Bearer $CRAZYSERP_API_KEY" -o /tmp/serp.json
```

`location=France` et rien d'autre, jamais `Paris,France` qui résout vers Paris en Ontario. `--max-time 240` obligatoire, CrazySERP scrape en direct. Un seul appel par article.

En extraire l'intention, les angles récurrents du top 10, les People Also Ask (source des questions de FAQ, **toujours reformulées**), les recherches associées, l'AI Overview (couvrir ses sous-questions dans les Hn, **ne jamais recopier son texte**) et le volume.

Repli en cascade, sans jamais échouer sur cette étape : CrazySERP, puis `WebSearch` (3 recherches max, titres et snippets seulement), puis mode dégradé sur le seul mot-clé. On publie dans tous les cas. Noter le mode retenu dans le message de commit.

## Étape 2, format de l'article

**En automatique, toujours `format: "guide"`.** Jamais `format: "top"`.

Un article "top X adresses" impose de relever prix, capacité et horaires sur le site officiel de chaque établissement, ce qu'un run non supervisé ne fait pas de façon fiable. Le réseau a déjà produit des concurrents inventés et des chiffres faux de cette manière. Les "top X" restent produits à la main par un consultant.

Le guide traite le sujet par occasion, par format, par budget ou par contrainte, sans liste d'adresses. C'est aussi le format le plus transposable, donc le plus cité par les moteurs IA.

## Étape 3, où écrire

Le FR vit à la racine de `content/`, l'EN sous `content/en/`. Le fichier va **dans le dossier du hub**, jamais dans un `/blog/` qui n'existe pas ici.

| `category` de la roadmap | Dossier | Catégorie EN | Auteur |
|---|---|---|---|
| Team building et séminaires | `team-building-seminaires/` | Team building and seminars | Sarah Nguyen |
| Bars et lieux ludiques | `bars-lieux-ludiques/` | Fun bars and venues | Thomas Bérard |
| Afterwork | `afterwork/` | Afterwork | Thomas Bérard |
| EVG | `evg/` | Bachelor party | Thomas Bérard |
| EVJF | `evjf/` | Bachelorette party | Thomas Bérard |
| Activités insolites | `activites-insolites/` | Unusual activities | Ursule Degarde |
| Anniversaires | `anniversaires/` | Birthdays | Ursule Degarde |

Ces libellés EN sont canoniques, ne jamais en inventer une variante, la taxonomie EN se fragmente sinon. Aucun `&` dans une catégorie ni dans un Hn.

## Étape 4, front matter

```yaml
---
title: "[Titre FR avec le kw, et {annee} si le sujet est daté]"
seoTitle: "[<= 60 caractères, kw dans le premier tiers, sans {annee}]"
description: "[<= 155 caractères, contient le kw, factuelle, pas de call to action]"
date: [YYYY-MM-DD]
lastmod: [YYYY-MM-DD]
author: "[Nom Prénom]"
authors: ["[Nom Prénom]"]
categories: ["[libellé exact du tableau]"]
tags: ["4 à 5 tags", "Paris"]
villes: ["Paris"]
translationKey: "[slug FR, identique FR et EN]"
format: "guide"
tldr:
  - "[3 puces, une idée forte par puce, un passage en gras par puce]"
faq:
  - question: "[4 à 6 questions issues des PAA, reformulées]"
    answer: "[3 à 5 phrases, réponse autonome et factuelle]"
draft: false
---
```

`{annee}` est résolu par Hugo dans le title. Dans le corps, l'année s'écrit `{{< year >}}`. Une URL ne porte jamais l'année, un article saisonnier est une page permanente mise à jour, pas une republication.

## Étape 5, rédaction FR

Charte de style du média, détaillée dans `LIGNE-EDITORIALE.md`, résumé opérationnel :

- Voix du pote qui connaît les bons plans de Paris, **tutoiement**, punchy et complice, mais toujours utile et crédible. Humour au service de l'info, jamais sur les faits. Positif, jamais vache.
- Chaque affirmation utile porte un chiffre, prix, durée, nombre de personnes, quartier, délai de réservation.
- 1 200 à 1 800 mots, 4 à 7 H2 explicites et auto-suffisants, paragraphes de 3 à 5 phrases.
- Un tableau comparatif dès que le sujet compare des formats, des occasions ou des budgets.
- Accents français complets. Pas de tiret cadratin ni demi-cadratin. Pas de deux-points en milieu de phrase, seulement pour introduire une liste. Pas de séparateur horizontal. Pas d'emoji.
- **Ne jamais recopier la FAQ dans le corps.** Depuis le 15/09, `single.html` rend le bloc FAQ en accordéon à partir du front matter, et `seo-head.html` en tire le JSON-LD FAQPage. Un H2 "Questions fréquentes" écrit à la main afficherait donc la FAQ deux fois. Historique à connaître, le thème ne rendait rien avant cette date, le run du 14/09 recopiait la FAQ dans le corps et la correction du même jour l'a retirée, ce qui a laissé trois articles avec un balisage sans contenu visible jusqu'à la correction du thème.
- Dernier H2 avant les sources, un paragraphe de maillage vers deux hubs pour élargir hors du sujet saisonnier.
- **Le tutoiement vaut aussi dans la FAQ et dans les tldr du front matter.** Le premier run avait tutoyé le corps et vouvoyé la FAQ ("comptez", "vérifiez"), ça se voit tout de suite. Relire les deux blocs du front matter avant de committer.

## Étape 6, faits et citations clients

**Aucun nom d'établissement, prix, capacité ni horaire qui ne soit pas vérifié.** En cas de doute, écrire le format générique sans nommer d'adresse. Un guide sans nom propre reste publiable, un guide avec une adresse inventée ne l'est pas.

Tout article sur un anniversaire va dans le hub Anniversaires (`anniversaires/`) et renvoie vers le pilier `/anniversaires/ou-feter-son-anniversaire-paris/` (EN `/en/anniversaires/where-to-celebrate-birthday-paris/`).

Deux fiches sont figées et vérifiées sur les sites officiels, réutilisables telles quelles quand le sujet s'y prête, sans jamais les forcer :

- **Bomb Squad Paris**, action game, boulevard de Sébastopol dans le 3e. Une heure de jeu, équipes de 2 à 6 joueurs, départs échelonnés permettant jusqu'à 60 participants sur deux heures, tarif dégressif de 45 euros par joueur à deux jusqu'à 28 euros à six, accessible dès 7 ans avec un adulte jusqu'à 15 ans.
- **PAN Bar Paris** (toujours écrit « PAN Bar Paris », jamais « PAN Bar » seul, les moteurs confondent avec d'autres villes), bar à tir, 6 rue de Paradis, Paris 10e. À partir de 16 euros par personne, box de 2 à 10 personnes, jusqu'à 66 joueurs en multi-box, privatisation des 400 m² à partir de 30 personnes, fermé le lundi. Jeu accessible dès 16 ans (accompagné d'un adulte pour les mineurs), bar réservé aux majeurs, vérifié le 23/09/2026. Pas de carte cadeau sur le site à cette date.

Le média est neutre et indépendant. Citer aussi les concurrents quand ils sont pertinents, ne jamais présenter une adresse comme la seule réponse, aucun contenu sponsorisé.

## Étape 7, maillage et sources

- Minimum 3 liens internes, **intra-langue uniquement**, insérés dans le corps de façon contextuelle. Au moins un vers un hub, au moins un vers un article existant du même univers. Ancres en langue naturelle contenant le mot-clé de la cible.
- Les liens externes vont **uniquement** dans le bloc final "Sources et liens utiles", jamais dans le corps. Le bloc s'ouvre par la phrase de relevé, "Tarifs, capacités et horaires relevés sur les sites officiels des établissements en [mois] {{< year >}}."

## Étape 8, version EN

Traduction fidèle du FR, même `translationKey`, slug EN traduit et non translittéré, catégorie et tags en EN, FAQ et tldr traduits, même auteur. Le fichier va dans `content/en/[même dossier de hub]/`.

## Étape 9, llms.txt

Ajouter les deux URLs dans `static/llms.txt`, à la fin de la liste "Articles de référence (FR)" et de son équivalent EN, au format `- Titre court : URL`.

## Étape 10, build et vérification

```bash
hugo --quiet
ls public/[hub]/[slug-fr]/index.html public/en/[hub]/[slug-en]/index.html
```

Une page absente du build est un échec, **même si `hugo` rend 0**. C'est le symptôme d'un fichier écrit hors du `contentDir` de sa langue, l'article part alors en 404 en production sans aucune erreur visible.

## Étape 11, roadmap, commit, push

Passer l'entrée en `status: done`, remplir `published_date`, `published_url_fr`, `published_url_en`, puis :

```bash
git add -A && git commit -m "Auto: [Titre FR] (FR + EN)" && git pull --rebase origin main && git push origin main
```

Push rejeté, retenter jusqu'à 3 fois en `pull --rebase` puis `push`. Toujours KO, marquer `failed` avec l'erreur, committer la roadmap seule, sortir en non-zéro.

## Échec, à n'importe quelle étape

Logger l'étape et la cause, passer l'entrée en `status: failed` avec le message dans `error`, **supprimer les fichiers d'article partiellement écrits**, committer la roadmap seule avec le message `Auto: roadmap update (failed)`, pousser, sortir en non-zéro. Une entrée `failed` attend une correction humaine.

## Dernière ligne obligatoire

Terminer la réponse par une ligne seule, sans gras ni ponctuation autour :

```
ARTICLE_PUBLIE=oui|non|rien-a-faire
```

`oui` uniquement si le push a réellement réussi, `rien-a-faire` si aucune entrée n'était éligible. Le script de lancement se sert de cette ligne, il alerte si elle manque.
