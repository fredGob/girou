# Plan de jeu — Équipe Une OGFC

## Objet du projet

Outil de présentation du **plan de jeu de l'équipe une** de l'Olympique Girou Football Club
(OGFC), saison 2026-2027 (Football R3 Occitanie, objectif montée en R2). L'utilisateur est
**adjoint** de cette équipe. Le contenu (objectifs, identité, valeurs, tactique 4-5-1,
organisation) provient de `contenu_presentation_saison.md`, qui est la **source de vérité** :
toute évolution du plan de jeu se fait d'abord dans ce fichier markdown, puis se répercute dans
le livrable HTML.

## Fichiers

| Fichier | Rôle |
|---|---|
| `contenu_presentation_saison.md` | Source de vérité — contenu brut du plan de jeu (équipe une) |
| `projet_de_jeu.html` | **Livrable principal** — présentation de consultation (voir ci-dessous) |
| `projet_de_jeu.pdf`, `projet_de_jeu_clair.pdf` | Export PDF (thème sombre / clair) de `projet_de_jeu.html`, généré via `export_pdf/` (voir ci-dessous) |
| `export_pdf/` | Script de génération des PDF ci-dessus (Node + Puppeteer) |
| `fiches_de_poste/` | **Livrable secondaire** — 10 fiches de poste PDF, une par poste du 4-5-1 (voir ci-dessous) |
| `fiches_de_poste/GUIDE_SCHEMAS.md` | Guide pour modifier les schémas tactiques (coordonnées, repère, miroir droite/gauche) |
| `exemple_fiche_poste_latéral_droit.md` | Ébauche d'origine (générée par Mistral) ayant servi de trame aux fiches de poste |
| `outils/telecharger_video.html` | Générateur de commande `yt-dlp` (téléchargement + découpe de vidéos) |
| `outils/sources_video_tactique.md` | Sources de vidéos tactiques par poste (chaînes, plateformes) — trace de recherche |
| `outils/banque_video_par_poste.md` | Grille de rangement des clips de formation, calée sur les 10 fiches de poste |
| `brand/` | Logos et maillots OGFC référencés par le HTML |
| `equipe_reserve_saison.md`, `equipe_reserve_saison.html`, `equipe_reserve_saison_slides.html` | **Archive** — plan de jeu de l'ancienne équipe réserve (saison 2024-25), remplacé par ce projet. Conservés pour mémoire, non maintenus. |

## Trame narrative (ordre voulu par l'utilisateur)

L'ordre des sections raconte une histoire et **ne doit pas être réarrangé sans accord** :

```
Couverture → Sommaire (diapo « ce qu'on va voir »)
  01 Objectifs            → où on va (montée R2, jalons chiffrés, mentalité)
  02 Règles de l'équipe   → comment on fonctionne (cadre du groupe, référents)
  03 Valeurs              → ce qu'on défend (identité « Différents », 3 valeurs, interdits)
  04 Tactique             → comment on joue (4-5-1, défense, construction, finition,
                             coups de pied arrêtés)
  Clôture
```

Les **coups de pied arrêtés** (6 mètres, touches, corners) sont une slide unique, sous-section
de Tactique (paginée à la suite, même eyebrow « 04 · Tactique »), et non une section 05
autonome : elle n'a pas d'entrée propre dans le sommaire (`data-toc-group`), on y arrive en
continuant à naviguer après la tactique.

Les **règles de l'équipe** ne sont pas figées : le markdown source n'en contient que trois
(ponctualité, communication, engagement) et la slide porte une note « à définir ensemble ».
C'est un chantier ouvert.

## Principe de `projet_de_jeu.html`

Application web autonome (un seul fichier, aucune dépendance hors polices Google), en format
**hybride** : un diaporama linéaire (navigation séquentielle, comme une présentation classique)
doublé d'un sommaire de navigation directe.

- Chaque écran est une **slide** plein écran (`section.slide`), une seule active à la fois,
  avec transition fondu/glissement.
- Navigation séquentielle par flèches clavier / molette de boutons prev-next / swipe (mobile),
  comme un diaporama qu'on présente à l'oral.
- Un bouton **Sommaire** (`M` ou clic) ouvre une vue d'ensemble des grandes sections
  (Objectifs, Identité, Tactique, Coups de pied arrêtés, Organisation) pour sauter directement
  à l'une d'elles — utile pour l'usage en outil de consultation entre les séances.
- **Routing par ancre** : chaque slide a un `data-id` unique (`#objectifs-chiffres`,
  `#systeme`…), reflété dans le hash d'URL. Les boutons précédent/suivant du navigateur
  fonctionnent et chaque écran est partageable en favori.
- Logique de navigation en JS vanilla en bas du fichier (fonction `go(n)`).
- **Images embarquées (obligatoire)** : les navigateurs de l'utilisateur sont installés en Flatpak
  et n'ont pas accès à `~/Documents`. À l'ouverture par double-clic, le portail de documents ne
  leur expose **que le fichier cliqué** — tout `src="brand/…"` ou `src="villes_poule.png"` échoue
  silencieusement (les logos de slide ont `alt=""`, donc l'absence est invisible). Le bloc
  « IMAGES EMBARQUÉES » en fin de fichier réinjecte donc toutes les images en data URI, depuis un
  dictionnaire `EMBED` à une clé par image (`logo-sombre`, `logo-clair`, `carte-poule`,
  `tactique`) ; les `src` de fichiers restent en repli pour l'export PDF.
  **Toute image ajoutée au deck doit être ajoutée à ce bloc**, sinon elle ne s'affichera pas chez
  l'utilisateur alors qu'elle s'affiche en rendu Puppeteer : poser
  `<img data-embed="ma-cle" src="mon_fichier.png">` dans la slide, puis ajouter la clé au
  dictionnaire. Bloc régénérable : redimensionner (photos/captures en JPEG ~1400 px, images à fond
  transparent ou à aplats en PNG) puis réencoder en base64.
  Test de non-régression : copier **le seul** `projet_de_jeu.html` dans un dossier vide et l'ouvrir.

### Export PDF (`export_pdf/`)

`projet_de_jeu.html` reste la seule source éditée à la main. Les PDF (`projet_de_jeu.pdf` en
thème sombre, `projet_de_jeu_clair.pdf` en thème clair) en sont un **export dérivé**, une page
par slide (1280×720) — jamais édités directement, à régénérer après toute modification du HTML.

Un bloc `@media print` dans `projet_de_jeu.html` aplatit les slides (empilées en flux normal
avec saut de page) au lieu du mode diaporama JS ; `export_pdf/make_pdf.js` (Node + Puppeteer,
déjà installé dans `export_pdf/node_modules/`) charge la page, force le thème demandé via la
fonction JS `applyTheme()`, puis imprime en PDF.

```bash
cd export_pdf
node make_pdf.js ../projet_de_jeu.html ../projet_de_jeu.pdf dark
node make_pdf.js ../projet_de_jeu.html ../projet_de_jeu_clair.pdf light
```

Si `export_pdf/node_modules/` a été supprimé ou sur une nouvelle machine : `npm install
puppeteer` dans `export_pdf/` (télécharge son propre Chromium, ~300 Mo — à exclure d'un futur
`.gitignore`). Contrôle visuel recommandé après régénération : `pdftoppm -png` puis relecture
des images.

## Principe de `fiches_de_poste/`

Dix fiches PDF A4 portrait de **6 pages**, une par poste du 4-5-1 (gardien, latéral droit,
défenseur central — fiche commune aux deux —, latéral gauche, milieu droit, milieu gauche,
milieu défensif, milieu relayeur, milieu offensif, attaquant axial), destinées à être
partagées individuellement aux joueurs concernés.

Structure de chaque fiche : couverture bleu OGFC pleine page (numéro, poste, schéma du 4-5-1
avec la position et la course du joueur, phrase-clé, 3 missions), puis 5 pages sur fond blanc
— Défense, Attaque, Mouvements attendus, Coups de pied arrêtés, Exigences. Le fond clair des
pages de contenu est **volontaire** et n'est pas une entorse à la règle du diaporama : c'est un
document imprimable.

Le contenu est **dérivé des principes collectifs** de `contenu_presentation_saison.md` (règle
du +1, bloc médian, passe et va, dézonage/compensation, joueur axial, centres 1re/2e intention,
premier poteau) — il n'existe nulle part ailleurs et reste soumis à validation.

### Chaîne de production

```
_postes.py   contenu des 10 fiches (source de vérité)
_style.css   feuille de style validée, partagée par les 10 fiches
_build.py    gabarit + schémas SVG → NN_poste.md, fiche_poste.html, fiche_poste.pdf
```

```bash
cd fiches_de_poste
uv venv .venv && VIRTUAL_ENV=$PWD/.venv uv pip install weasyprint   # à refaire par session
WEASY=$PWD/.venv/bin/weasyprint python3 _build.py        # les 10
WEASY=$PWD/.venv/bin/weasyprint python3 _build.py 02 05  # au choix
```

Ne jamais éditer les `.md` ni les `.html` générés : ils sont réécrits à chaque build. Tout
passe par `_postes.py` (contenu et coordonnées des schémas) ou `_style.css` (mise en forme).

**Modifier le contenu texte** — directement dans `_postes.py`. Chaque poste suit la même
structure (`lede`/`missions` pour la couverture, puis `defense`, `attaque`, `mouvements`, `cpa`,
`exigences`). Les latéraux (`_lateral()`) et les milieux de couloir (`_milieu_couloir()`) sont
écrits une seule fois pour les deux côtés, via des jetons (`{C}`, `{A}`, `{LAT}`, `{MC}`) —
une modif s'y répercute sur les deux fiches du couple.

**Modifier un schéma tactique** — voir `fiches_de_poste/GUIDE_SCHEMAS.md`. En bref : les
schémas ne sont pas des fichiers image, ce sont des SVG générés depuis des coordonnées de
terrain (`you`, `amis`, `advs`, `ball`, `arrow`, `ligne`, `xspan`) dans les mêmes blocs
`schema=dict(...)` / `mouvements=dict(..., situations=[...])` de `_postes.py`. Tous les
schémas sont décrits dans un repère unique : `x = 0` notre but, `x = 100` leur but ; `y = 0`
notre gauche, `y = 100` notre droite. Les fonctions de rendu gèrent l'orientation (terrain
horizontal : notre droite est **en bas** ; terrain vertical : notre droite est **à droite**).
Ne jamais écrire de coordonnées SVG en dur ni de valeur numérique brute à la place de `f(...)`
dans les fonctions à deux côtés — c'est comme ça qu'on inverse droite et gauche sans le voir.

Le HTML porte à la fois un rendu **impression** (boîtes `@page` pour l'en-tête, le pied et le
numéro) et un rendu **écran** sous `@media screen`, qui rejoue ces éléments et le fond de la
couverture — WeasyPrint ignore `@media screen`, le navigateur ignore les boîtes `@page`.
Contrôle visuel obligatoire avant livraison : `pdftoppm -png` puis relecture des images.

## Identité visuelle (skill girou-brand)

Le rendu applique la charte OGFC via le skill `girou-brand` (`.claude/skills/girou-brand/`) :
bleu OGFC `#183C6E` et dérivés, vert terrain / rouge / jaune / orange pour les accents,
typographie Montserrat (titres, jusqu'à la graisse 900/Black pour les temps forts) + Open Sans
(corps). Logos dans `brand/`.

### Contraintes de design (préférences utilisateur — à respecter)

- **Deux thèmes, un fond cohérent dans chacun** : le livrable propose un thème **sombre**
  (dégradé bleu OGFC → bleu foncé, thème par défaut) et un thème **clair** (blanc → gris clair
  OGFC), permutables par le bouton de la barre supérieure ou la touche `T`, avec mémorisation
  dans `localStorage`. À l'intérieur d'un thème, le fond reste **uniforme sur toutes les
  slides** : on n'alterne jamais slide claire / slide sombre dans un même déroulé.
  Le logo suit le thème (`Nouveau logo blanc.PNG` en sombre, `Nouveau logo bleu.PNG` en clair) :
  toute image de logo ajoutée doit porter l'attribut `data-logo` pour être permutée par le JS.
  Techniquement, toutes les couleurs passent par des **jetons CSS** (`--fg`, `--fg-soft`,
  `--rule`, `--stroke`…) redéfinis sous `:root[data-theme="light"]` — ne jamais réintroduire de
  couleur de texte ou de fond en dur dans une règle de composant.
- **Pas d'orange sur les titres** : l'orange `#E65100` est jugé trop flashy en typographie de
  titre. Il reste acceptable en **couleur secondaire** (filets fins, flèches, marqueurs de
  liste, barre de progression, petits symboles). Les titres sont **blancs** ; le mot d'emphase
  se distingue par un **contour** (`-webkit-text-stroke` blanc translucide), pas par la couleur.
  Pour les libellés et eyebrows, utiliser le tint clair du bleu `--pale:#A9C4E4`.
- **Occuper toute la largeur** : ne pas laisser le contenu tassé dans une colonne à gauche avec
  du vide à droite. Chaque slide utilise des grilles éditoriales pleine largeur (`.grid-2`,
  `.grid-3`, `.grid-4`, séparateurs verticaux neutres) et un en-tête `.head.split` qui place le
  titre à gauche et une accroche à droite.
- **Chiffres classiques** : les numéros (numéros de colonne, sommaire, R3/R2) sont en
  **aplat plein**, jamais en contour/typographie évidée (`-webkit-text-stroke`) — traitement
  jugé trop stylisé. Même règle pour le mot d'emphase des titres, qui se distingue par la
  **couleur** (`--label`, bleu clair de marque) et non par un contour. Pas de gros numéro
  décoratif en filigrane de fond.
- **Pas de numérotation de page** : aucun compteur « n / total ». Seuls repères de progression :
  la barre orange en haut et le nom de la section en bas à gauche.
- **Aucun liseré d'accent** sur les blocs de contenu (ni `border-top` ni `border-left` coloré) :
  jugé « template IA ». La couleur de marque s'exprime via les fonds translucides, les
  numéros fantômes (`.gn`), et la typographie.
- **Éviter l'esthétique « grille de cartes arrondies »** générique (3-4 cards identiques côte à
  côte façon slide corporate) : jugée trop « template IA » lors de la refonte 2026. Préférer une
  direction éditoriale/sport — grands mots typographiques (Montserrat Black), listes à numéros
  fantômes, bandeaux pleine largeur, statistiques en gros chiffres — avec un seul motif
  décoratif récurrent (ligne diagonale façon ligne de touche, watermark numéroté) plutôt que des
  cartes répétées sur chaque écran.
- Tailles en `vh`/`vw` pour conserver un format « plein écran » adaptatif.
