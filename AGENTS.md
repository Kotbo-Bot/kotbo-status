# AGENTS.md — kotbo-status

Guide pour tout agent (ou humain) qui intervient sur ce dépôt.

## Ce qu'est ce projet

Page de statut (status page) publique de **Kotbo**, propulsée par
[**Upptime**](https://github.com/upptime/upptime) — un moniteur d'uptime
open-source **sans serveur**, entièrement piloté par GitHub Actions.

- Site public : <https://status.kotbo.fr>
- Dépôt : `Kotbo-Bot/kotbo-status`
- Services surveillés (définis dans `.upptimerc.yml`) :
  - **API Kotbo** — `https://api.kotbo.fr/health`
  - **Bot Discord** — `https://api.kotbo.fr/health/discord`
  - **Base de données** — `https://api.kotbo.fr/health/db`
  - **Redis** — `https://api.kotbo.fr/health/redis`
  - **Dashboard** — `https://dash.kotbo.fr`

## Comment ça marche (important)

Il n'y a **aucun code applicatif ni `package.json`** dans ce dépôt. Tout est
généré par des GitHub Actions qui appellent l'action `upptime/uptime-monitor`.

- **`uptime.yml`** (toutes les ~5 min) : ping chaque URL, enregistre le résultat
  dans `history/` (un fichier `.yml` par service = commits + issues d'incident).
- **`response-time.yml` / `graphs.yml`** : calculent les temps de réponse et
  régénèrent les PNG dans `graphs/` et les JSON dans `api/`.
- **`summary.yml`** : régénère `history/summary.json` (consommé par la page) et
  le tableau dans `README.md`.
- **`site.yml`** : **génère la status page** (app Svelte/Sapper `@upptime/status-page`)
  et la publie sur la branche `gh-pages` → servie sur `status.kotbo.fr`.
- **`update-template.yml`** (hebdo) : met à jour les workflows depuis le template
  Upptime amont. ⇒ **Ne jamais éditer les fichiers `.github/workflows/*` à la main**,
  ils sont écrasés. Toute config passe par `.upptimerc.yml`.

Le HTML/CSS de la page **n'existe pas dans ce dépôt** : il est buildé à distance
au moment du `site.yml`. On ne peut donc pas le lancer localement tel quel.

## Le seul fichier à éditer : `.upptimerc.yml`

C'est le point d'entrée de toute la configuration. Champs clés sous
`status-website` :

| Champ | Rôle |
|---|---|
| `name`, `introTitle`, `introMessage` | Titres/textes d'intro (supportent le markdown via snarkdown, et le HTML brut passe tel quel) |
| `navbar` | Liens de la barre de navigation |
| `theme` | Thème de base (`light`, `dark`, `night`, `ocean`) |
| `css` | **CSS inline injecté** (chargé après `global.css` et le thème → priorité maximale) |
| `customHeadHtml` | HTML injecté dans `<head>` (ex. `<link>` Google Fonts) |
| `customBodyHtml` / `customFootHtml` | HTML en début/fin de `<body>` |
| `i18n` | **Traduction de toute l'UII** (voir ci-dessous) |
| `cname` | Domaine perso (`status.kotbo.fr`) |

### Traduction (i18n)

Les libellés de l'app (« Live Status », « Past Incidents », « Overall
uptime »…) sont **codés en dur en anglais** dans les composants Svelte : ils ne
se traduisent **pas** via `name`/`introTitle`, uniquement via le bloc
`status-website.i18n`. Placeholders à conserver dans les valeurs :

- `overallUptime` → `$UPTIME` · `averageResponseTime` → `$TIME`
- `pastIncidentsResolved` → `$MINUTES`, `$POSTS` · `activeIncidentSummary` → `$DATE`, `$POSTS`
- `incidentReport` → `$NUMBER` · `footer` → `$REPO` (rendu markdown)
- `locale` → mettre `fr` (formatage des dates)

### Structure DOM ciblée par la CSS (repère pour le style)

```
nav > .container (logo + ul>li>a)
main.container
  header > h1 + p.lead                     ← intro
  article.up:not(.link)                    ← bannière « tous les services opérationnels »
  div.f.changed > h2 + form.f.r (labels)   ← titre « État en direct » + sélecteur 24h/7j/…
  section.live-status > article.up.link.graph
     h4 (img.icon + a) + div (.data) …     ← carte de service
  section > h2 + h3(date) + article.down.link  ← incidents passés (style post-it)
footer > p
```

## Style visuel

La page reprend l'identité **« whiteboard / post-it »** de <https://kotbo.fr> :

- Fond `#f8f9fa` avec **grille de points** (dot grid).
- Polices : **Manrope** (titres, 900), **Inter** (corps), **Caveat** (manuscrit / post-it).
- Palette : primaire `#625fff` ; post-it `#ffb7b2` (rose), `#fdfd96` (jaune),
  `#aec6cf` (bleu) ; marqueur `#ff4d4d` / `#4d79ff` ; up `#00bb7f`, down `#ff2357`.
- Cartes de service = fiches blanches arrondies ombrées ; incidents = **post-it**
  légèrement inclinés avec scotch.

Tout ce style vit dans le bloc `css:` (et `customHeadHtml:` pour les fonts) de
`.upptimerc.yml`.

## Prévisualiser un changement de style localement

Le build réel étant distant, on valide la CSS via un preview qui reproduit le
DOM Upptime avec les vraies données (`history/summary.json`, `graphs/`). Voir le
dossier scratchpad de session ou recréer un `preview.html` reprenant la
structure DOM ci-dessus + le bloc `css:`. Rendu/screenshot possible avec
`chromium --headless --screenshot`.

## Workflow de contribution

1. Modifier **uniquement** `.upptimerc.yml` (jamais `.github/workflows/*`, `api/`,
   `graphs/`, `history/` — ce sont des artefacts régénérés).
2. Commit + push sur `main` (ne commit/push que si demandé).
3. Le workflow `site.yml` se déclenche (push sur `assets/**`, cron quotidien, ou
   `workflow_dispatch`) et redéploie `status.kotbo.fr`. Pour forcer :
   déclencher « Static Site CI » manuellement dans l'onglet Actions.
