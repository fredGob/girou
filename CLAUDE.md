# Plan de jeu — Équipe Une OGFC

## Objet du projet

Outil de présentation du **plan de jeu de l'équipe une** de l'Olympique Girou Football Club
(OGFC), saison 2026-2027 (Football R3 Occitanie, objectif montée en R2). L'utilisateur est
**adjoint** de cette équipe.

Le projet a été **recentré en 2026** sur une trame plus resserrée en **deux parties** — « La
saison » (où on va, comment on fonctionne) puis « Identité » (l'état d'esprit à mettre en
place) — et ne couvre plus le détail tactique (système 4-5-1, phases de jeu, coups de pied
arrêtés, fiches de poste), retiré de cette version. Ces éléments restent consultables dans
l'historique git si besoin.

Il n'y a plus de source markdown séparée : `projet_de_jeu.html` est désormais **édité
directement à la main** et fait foi.

## Fichiers

| Fichier | Rôle |
|---|---|
| `projet_de_jeu.html` | **Livrable** — présentation de consultation, éditée à la main (voir ci-dessous) |
| `tableau-anime.html` | Outil autonome — éditeur de tableau tactique animé (indépendant du deck) |
| `tactique.png` | Schéma 4-5-1 (asset conservé ; plus référencé par le deck actuel) |
| `brand/` | Logos OGFC référencés par le HTML (absents du dépôt : embarqués en data URI, voir plus bas) |
| `README.md` | — |

## Trame narrative (ordre voulu par l'utilisateur)

L'ordre des slides raconte une histoire et **ne doit pas être réarrangé sans accord** :

```
Couverture → Sommaire (diapo « au programme », 2 entrées)
  01 · La saison   → objectif principal (montée R2), objectifs annexes (esprit club),
                     organisation (règles, horaires), adversaires (la poule)
  02 · Identité    → l'exemple par les leaders, montrer du caractère,
                     se responsabiliser, créer une nouvelle culture
  Clôture (« Différents »)
```

Le **sommaire** existe en deux formes qui doivent rester cohérentes entre elles : l'agenda
statique de la slide « Au programme » et le sommaire déroulant (touche `M`) construit depuis les
attributs `data-toc-group` / `data-toc-title` / `data-toc-desc`. Les deux n'affichent que les
**deux parties** (01 La saison, 02 Identité) : seules la première slide de chaque partie porte
un `data-toc-group`. La clôture n'a pas d'entrée de sommaire, on y arrive en continuant à
naviguer.

Les **règles de l'équipe** (slide « Organisation ») ne sont pas figées : c'est un chantier
ouvert, à compléter avec le groupe.

## Principe de `projet_de_jeu.html`

Application web autonome (un seul fichier, aucune dépendance hors polices Google), en format
**hybride** : un diaporama linéaire (navigation séquentielle, comme une présentation classique)
doublé d'un sommaire de navigation directe.

- Chaque écran est une **slide** plein écran (`section.slide`), une seule active à la fois,
  avec transition fondu/glissement.
- Navigation séquentielle par flèches clavier / boutons prev-next / swipe (mobile), comme un
  diaporama qu'on présente à l'oral.
- Un bouton **Sommaire** (`M` ou clic) ouvre une vue d'ensemble des deux parties pour sauter
  directement à l'une d'elles — utile pour l'usage en outil de consultation entre les séances.
- **Routing par ancre** : chaque slide a un `data-id` **unique** (`#saison`, `#valeurs-leaders`…),
  reflété dans le hash d'URL. Les boutons précédent/suivant du navigateur fonctionnent et chaque
  écran est partageable en favori. ⚠️ Ne jamais dupliquer un `data-id` : deux slides partageant
  la même ancre cassent le rechargement et la navigation arrière (on retombe toujours sur la
  première).
- Logique de navigation en JS vanilla en bas du fichier (fonction `go(n)`).
- **Images embarquées (obligatoire)** : les navigateurs de l'utilisateur sont installés en Flatpak
  et n'ont pas accès à `~/Documents`. À l'ouverture par double-clic, le portail de documents ne
  leur expose **que le fichier cliqué** — tout `src="brand/…"` ou `src="villes_poule.png"` échoue
  silencieusement (les logos de slide ont `alt=""`, donc l'absence est invisible). Le bloc
  « IMAGES EMBARQUÉES » en fin de fichier réinjecte donc toutes les images en data URI, depuis un
  dictionnaire `EMBED` à une clé par image (`logo-sombre`, `logo-clair`, `carte-poule`) ; les
  `src` de fichiers restent en repli.
  **Toute image ajoutée au deck doit être ajoutée à ce bloc**, sinon elle ne s'affichera pas chez
  l'utilisateur alors qu'elle s'affiche en rendu navigateur : poser
  `<img data-embed="ma-cle" src="mon_fichier.png">` dans la slide, puis ajouter la clé au
  dictionnaire. Inversement, retirer la clé quand l'image n'est plus utilisée (poids mort).
  Bloc régénérable : redimensionner (photos/captures en JPEG ~1400 px, images à fond transparent
  ou à aplats en PNG) puis réencoder en base64.
  Test de non-régression : copier **le seul** `projet_de_jeu.html` dans un dossier vide et
  l'ouvrir.

## Identité visuelle (skill girou-brand)

Le rendu applique la charte OGFC via le skill `girou-brand` (`.claude/skills/girou-brand/`) :
bleu OGFC `#183C6E` et dérivés, vert terrain / rouge / jaune / orange pour les accents,
typographie Montserrat (titres, jusqu'à la graisse 900/Black pour les temps forts) + Open Sans
(corps). Logos référencés depuis `brand/` (et embarqués en data URI, voir plus haut).

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
