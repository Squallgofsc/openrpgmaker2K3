# Open RPG Maker — Fork Qt5/SDL2 (non-officiel) / Unofficial Qt5/SDL2 Fork

> 🇫🇷 Ceci est un fork **non-officiel** d'Open RPG Maker 0.2.3, remis à jour
> pour fonctionner sur les systèmes modernes. Il n'est pas affilié à
> l'auteur d'origine.
>
> 🇬🇧 This is an **unofficial** fork of Open RPG Maker 0.2.3, updated to run
> on modern systems. It is not affiliated with the original author.

---

## 🇫🇷 Français

### Qu'est-ce que c'est ?

[Open RPG Maker](https://sourceforge.net/projects/openrpgmaker/) est un
éditeur de jeux de rôle gratuit et open source, créé par **Justin
"Tuxinator" Davis**, reprenant les fonctionnalités des éditeurs RPG Maker
2000/2003. Le projet d'origine, dont la dernière version (0.2.3) date de
2014, ne compilait et ne fonctionnait plus sur les systèmes Linux
modernes (Qt4 n'est plus disponible dans les dépôts des distributions
récentes) ni via son moteur audio d'origine (FMOD, propriétaire).

Ce fork corrige ces problèmes pour permettre à quiconque de continuer à
faire tourner, tester et améliorer ce logiciel.

### Attribution et licence

Ce projet reste sous licence **GNU General Public License** (GPLv2 ou
ultérieure, telle que publiée par l'auteur d'origine). Tout le mérite du
travail de conception original revient à **Justin "Tuxinator" Davis**
(tuxdavis@gmail.com) et aux contributeurs ayant participé à la création
de tous les visuels et graphiques : krkrbs420, Andrés Montalvo alias
Ingfrith et Gambit. Ce fork n'est ni approuvé ni supervisé par l'auteur
d'origine — c'est un travail de maintenance communautaire, publié dans
le même esprit de partage que le projet initial.

### Historique des corrections apportées par ce fork

**Portage Qt4 → Qt5**
- Modules Qt manquants (`webkitwidgets`, `widgets`, `printsupport`) ajoutés
  aux fichiers `.pro`
- `QHeaderView::setResizeMode` renommé en `setSectionResizeMode` (17
  fichiers)
- Includes Qt explicites ajoutés là où Qt4 les incluait implicitement
  (`QRegExpValidator`, `QMimeData`, `QDrag`)
- Système de plugins Qt4 obsolète (`Q_EXPORT_PLUGIN2`) migré vers le
  système Qt5 (`Q_PLUGIN_METADATA`) dans les 7 plugins d'import

**Remplacement du moteur audio FMOD (propriétaire) par SDL2_mixer (zlib)**
- Nouveau `AudioManager` réécrit intégralement sur SDL2/SDL2_mixer,
  interface publique identique à l'originale (aucun appelant à modifier)
- Le dossier `fmodex/` (bibliothèque et en-têtes propriétaires) a été
  entièrement retiré du dépôt

**Bugs corrigés (trouvés lors de tests fonctionnels approfondis)**
- **Crash mémoire** à l'import de tileset RM2K3 (`importrm2k3tileset.cpp`)
  : boucle de 32 itérations écrivant hors limites d'un tableau de 24
  cases (`NUM_PHASES`), corrompant le tas
- **Désynchronisation d'interface** dans la base de données "Items" :
  after fermeture/réouverture, le panneau affiché ne correspondait plus
  au type d'objet sélectionné (dépendance incorrecte à un signal Qt qui
  ne se déclenche pas toujours)
- **Crash** à l'ouverture de l'onglet "Classes" de la base de données :
  même bug de boucle 32 vs 24 que le tileset, dans un fichier différent
  (`database_classes.cpp`, 6 occurrences)
- **Échec silencieux** de l'import "Monster" sur des images en vraie
  couleur : appel à `QImage::pixelIndex()` sans vérifier que l'image est
  bien en mode palette
- **Affichage incohérent** de la durée/position dans l'aperçu musical
  (`Tools > Listen to Music`) quand la vitesse de lecture n'est pas 100%
- **Conditionnelle de compilation multiplateforme** jamais finalisée par
  l'auteur d'origine (préfixe/suffixe de bibliothèques dynamiques codé en
  dur pour Linux) — corrigée sur la base de preuves tirées des binaires
  officiels Linux/Windows/Mac de la 0.2.3

Le détail complet de chaque correction (avec preuves, code avant/après,
méthode de diagnostic) est disponible dans l'historique du dépôt et dans
`UNIMPLEMENTED_FEATURES.md`.

### Compiler depuis les sources

#### Linux

Dépendances (exemple pour Ubuntu/Debian) :
```sh
sudo apt install build-essential qtbase5-dev qt5-qmake qtbase5-dev-tools \
                  qttools5-dev-tools libqt5webkit5-dev \
                  libsdl2-dev libsdl2-mixer-dev
```

Compilation :
```sh
cd openrpgmaker-0.2.3-source
qmake OpenRPGMaker.pro
make
```
L'exécutable `OpenRPGMaker` est généré à la racine du projet.

#### Windows

Voir le guide dédié **`BUILD_WINDOWS.md`**, fourni avec ce dépôt, qui
détaille pas à pas l'installation des outils et la compilation.

#### macOS

```sh
brew install qt@5 sdl2 sdl2_mixer
cd openrpgmaker-0.2.3-source
/usr/local/opt/qt@5/bin/qmake OpenRPGMaker.pro
make
```
Un exécutable macOS n'a pas pu être produit ni testé par ce fork (aucun
Mac disponible pour la maintenance) — la conditionnelle de compilation a
été préparée sur la base de preuves tirées du binaire Mac officiel
d'origine, mais reste **non testée en conditions réelles**. Les retours
de compilation sur Mac sont les bienvenus via une issue GitHub.

### Feuille de route

Ce qui suit recense tout ce qui reste non implémenté ou incomplet,
découvert au fil des tests et qui sera rajouté au fil des mises à jour à
venir :

- **Moteur de jeu** (`src/engine`) : jamais implémenté par l'auteur
  d'origine, à aucune version connue du projet. L'éditeur permet de
  construire un jeu complet, mais aucun moyen de le lancer/tester
  n'existe actuellement.
- **Database > Common Events** et **Database > Monster Groups** : jamais
  implémentés ("Not Yet Implemented" affiché à l'écran).
- **Menu Project > Create Game Disk** : jamais implémenté (fonction vide
  dans le code).
- **Import externe** : plusieurs plugins échouent silencieusement (sans
  message d'erreur) sur des images ne respectant pas exactement le
  format RM2K3 classique attendu — à améliorer avec un vrai retour
  utilisateur.
- **Fonctionnalités des générations RPG Maker plus récentes** (VX, VX
  Ace, MV, MZ) : création de 2 projets basés sur ce fork et visant à
  reproduire les fonctionnalités des générations plus récentes de RPG
  Maker. Piste lointaine, non engagée.
- **Traduction française de l'interface** + sélecteur de langue : faisable
  et bien cadré (l'interface est déjà structurée pour l'internationalisation
  Qt), non encore réalisé.

---

## 🇬🇧 English

### What is this?

[Open RPG Maker](https://sourceforge.net/projects/openrpgmaker/) is a
free and open-source RPG editor created by **Justin "Tuxinator" Davis**,
reproducing the features of the RPG Maker 2000/2003 editors. The
original project's last release (0.2.3) dates back to 2014 and no longer
built or ran on modern Linux systems (Qt4 is no longer available in
current distribution repositories), nor did its original audio backend
(FMOD, proprietary).

This fork fixes these issues so that anyone can keep running, testing,
and improving this software.

### Attribution and license

This project remains under the **GNU General Public License** (GPLv2 or
later, as published by the original author). All credit for the original
design work belongs to **Justin "Tuxinator" Davis** (tuxdavis@gmail.com)
and the contributors who took part in creating all the visuals and
graphics: krkrbs420, Andrés Montalvo aka Ingfrith, and Gambit. This fork
is neither endorsed nor supervised by the original author — it is a
community maintenance effort, published in the same spirit of sharing as
the original project.

### History of corrections made by this fork

**Qt4 → Qt5 port**
- Missing Qt modules (`webkitwidgets`, `widgets`, `printsupport`) added to
  the `.pro` files
- `QHeaderView::setResizeMode` renamed to `setSectionResizeMode` (17
  files)
- Explicit Qt includes added where Qt4 included them implicitly
  (`QRegExpValidator`, `QMimeData`, `QDrag`)
- Obsolete Qt4 plugin system (`Q_EXPORT_PLUGIN2`) migrated to the Qt5
  system (`Q_PLUGIN_METADATA`) across the 7 import plugins

**Replaced the proprietary FMOD audio engine with SDL2_mixer (zlib
license)**
- New `AudioManager` fully rewritten on top of SDL2/SDL2_mixer, with the
  exact same public interface as the original (no caller code changed)
- The `fmodex/` directory (proprietary library and headers) has been
  entirely removed from the repository

**Bugs fixed (found through extensive functional testing)**
- **Memory corruption crash** when importing an RM2K3 tileset
  (`importrm2k3tileset.cpp`): a 32-iteration loop writing past the bounds
  of a 24-slot array (`NUM_PHASES`), corrupting the heap
- **UI desync** in the "Items" database: after closing and reopening the
  dialog, the displayed panel no longer matched the selected item's type
  (incorrect reliance on a Qt signal that doesn't always fire)
- **Crash** when opening the "Classes" database tab: the same 32-vs-24
  loop bug as the tileset importer, in a different file
  (`database_classes.cpp`, 6 occurrences)
- **Silent failure** of the "Monster" importer on truecolor images: a
  call to `QImage::pixelIndex()` without checking that the source image
  is actually in palette mode
- **Inconsistent duration/position display** in the music preview
  (`Tools > Listen to Music`) when playback speed isn't 100%
- **Cross-platform build conditional** never finished by the original
  author (dynamic library prefix/suffix hardcoded for Linux) — fixed
  based on evidence extracted from the official 0.2.3 Linux/Windows/Mac
  binaries

Full details of each fix (with evidence, before/after code, diagnostic
method) are available in the repository history and in
`UNIMPLEMENTED_FEATURES.md`.

### Building from source

#### Linux

Dependencies (Ubuntu/Debian example):
```sh
sudo apt install build-essential qtbase5-dev qt5-qmake qtbase5-dev-tools \
                  qttools5-dev-tools libqt5webkit5-dev \
                  libsdl2-dev libsdl2-mixer-dev
```

Build:
```sh
cd openrpgmaker-0.2.3-source
qmake OpenRPGMaker.pro
make
```
The `OpenRPGMaker` executable is produced at the project root.

#### Windows

See the dedicated **`BUILD_WINDOWS.md`** guide included in this
repository, which walks through tool installation and compilation step
by step.

#### macOS

```sh
brew install qt@5 sdl2 sdl2_mixer
cd openrpgmaker-0.2.3-source
/usr/local/opt/qt@5/bin/qmake OpenRPGMaker.pro
make
```
A macOS executable could not be produced or tested by this fork (no Mac
available for maintenance) — the build conditional was prepared based on
evidence from the official Mac binary, but remains **untested under real
conditions**. Build reports from macOS users are welcome via a GitHub
issue.

### Roadmap

The following tracks everything still unimplemented or incomplete,
discovered through testing, and will keep being added to as future
updates are made:

- **Game engine** (`src/engine`): never implemented by the original
  author, at any known version of the project. The editor lets you build
  a complete game, but there is currently no way to run or test it.
- **Database > Common Events** and **Database > Monster Groups**: never
  implemented ("Not Yet Implemented" shown on screen).
- **Project menu > Create Game Disk**: never implemented (empty function
  body in the code).
- **External import**: several plugins fail silently (no error message)
  on images that don't exactly match the expected classic RM2K3
  format/dimensions — needs proper user feedback.
- **Features from newer RPG Maker generations** (VX, VX Ace, MV, MZ):
  creation of 2 projects based on this fork, aiming to reproduce the
  features of newer RPG Maker generations. A distant possibility, not
  currently pursued.
- **French UI translation** + language selector: feasible and
  well-scoped (the UI is already structured for Qt internationalization),
  not yet implemented.

---

## Crédits / Credits

- **Auteur d'origine / Original author :** Justin "Tuxinator" Davis
- **Graphismes / Graphics :** krkrbs420, Andrés Montalvo (Ingfrith),
  Gambit
- **Icônes / Icons :** thème CrystalSVG de KDE / KDE's CrystalSVG icon
  theme
- **Fork Qt5/SDL2 :** voir l'historique Git / see Git history
- **Corrections et modifications de code de ce fork / This fork's code
  changes and fixes :** rédigées par Claude (Anthropic) / written by
  Claude (Anthropic)

Licence / License : **GNU GPL v2 or later** — voir `COPYING`.
