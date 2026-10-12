# Compiler Open RPG Maker (fork Qt5/SDL2) sur Windows — Guide détaillé

Ce guide décrit, étape par étape, comment compiler un exécutable Windows
à partir des sources de ce fork. Il est écrit pour quelqu'un qui n'a
jamais compilé de projet Qt sous Windows.

## Pourquoi MSYS2, et pas l'installeur Qt officiel ?

Ce projet utilise le module **QtWebKit**, qui a été **retiré des
distributions officielles de Qt à partir de la version 5.6** (2016).
L'installeur Qt officiel (Qt Online Installer) ne propose donc plus ce
module pour aucune version de Qt5 raisonnablement récente.

La solution retenue ici est **MSYS2**, un environnement qui fournit des
paquets Qt5 pré-compilés incluant un portage communautaire actif de
QtWebKit (paquet `mingw-w64-x86_64-qtwebkit`, toujours maintenu à ce
jour). C'est la méthode la plus fiable pour obtenir un environnement de
compilation complet et cohérent sans avoir à recompiler QtWebKit
soi-même depuis les sources (ce qui prendrait plusieurs heures et
nécessiterait de nombreux outils annexes : Perl, Python, Ruby, gperf,
bison, flex, ICU...).

## Étape 1 — Installer MSYS2

1. Télécharger l'installeur officiel depuis **https://www.msys2.org/**
   (fichier `msys2-x86_64-<date>.exe`).
2. Lancer l'installeur, laisser le dossier d'installation par défaut
   (`C:\msys64`), terminer l'installation.
3. À la fin de l'installation, une fenêtre de terminal MSYS2 s'ouvre
   automatiquement. Fermer cette fenêtre.
4. Depuis le menu Démarrer de Windows, chercher et ouvrir
   **"MSYS2 MinGW x64"** (⚠️ pas "MSYS2 CLANG64", ni "MSYS2 UCRT64", ni
   "MSYS2 MSYS" tout court — bien choisir précisément **"MSYS2 MinGW
   x64"**, aussi affiché `MINGW64`, c'est un terminal différent des
   autres icônes installées). On peut vérifier qu'on est au bon endroit
   à tout moment avec :
   ```sh
   echo $MSYSTEM
   ```
   qui doit afficher `MINGW64`.

## Étape 2 — Mettre à jour MSYS2

Dans le terminal **"MSYS2 MinGW x64"** qui vient de s'ouvrir, taper :

```sh
pacman -Syu
```

Répondre `O` (oui) si demandé. Le terminal va probablement se fermer
tout seul à la fin de cette étape (normal, une mise à jour du cœur du
système l'exige). **Rouvrir "MSYS2 MinGW x64"** depuis le menu Démarrer,
puis relancer une seconde fois :

```sh
pacman -Syu
```

Répéter jusqu'à ce que la commande indique qu'il n'y a plus rien à
mettre à jour ("there is nothing to do").

## Étape 3 — Installer les outils de compilation et les dépendances

Toujours dans **"MSYS2 MinGW x64"**, installer en une seule commande
tout ce qui est nécessaire :

```sh
pacman -S --needed base-devel mingw-w64-x86_64-toolchain \
  mingw-w64-x86_64-qt5-base mingw-w64-x86_64-qt5-tools \
  mingw-w64-x86_64-qt5-multimedia mingw-w64-x86_64-qtwebkit \
  mingw-w64-x86_64-SDL2 mingw-w64-x86_64-SDL2_mixer \
  mingw-w64-x86_64-make
```

- Répondre `O` (oui) à toutes les questions de confirmation
  (sélectionner "all" si un choix de paquets multiples est proposé).
- Cette installation télécharge plusieurs centaines de Mo — elle peut
  prendre entre 10 et 30 minutes selon la connexion.
- Si une commande échoue avec une erreur de miroir/serveur, relancer
  simplement la même commande (les serveurs MSYS2 sont parfois
  temporairement indisponibles).
- Pacman installe automatiquement toutes les dépendances nécessaires
  (dont les bibliothèques de décodage audio utilisées par
  SDL2_mixer — FLAC, mpg123, Opus, Vorbis — inutile de les demander
  séparément).

## Étape 4 — Récupérer les sources du projet

Deux méthodes possibles :

**Méthode A — via Git (recommandée si vous comptez suivre les mises à
jour)**
```sh
pacman -S --needed git
cd /c/
git clone https://github.com/<votre-compte>/<nom-du-depot>.git openrpgmaker-src
cd openrpgmaker-src
```

**Méthode B — depuis l'archive téléchargée sur GitHub**
1. Télécharger `openrpgmaker-qt5-fork-0.2.3-source.zip` depuis la page
   des releases GitHub du projet.
2. L'extraire, par exemple dans `C:\openrpgmaker-src`.
3. Dans le terminal MSYS2 :
```sh
cd /c/openrpgmaker-src/openrpgmaker-0.2.3-source
```

⚠️ **Important sur les chemins** : dans le terminal MSYS2, un chemin
Windows comme `C:\openrpgmaker-src` s'écrit `/c/openrpgmaker-src`
(lettre de lecteur en minuscule, précédée d'un slash, puis des slashs
au lieu des antislashs).

⚠️ **Éviter OneDrive** : si votre dossier utilisateur Windows est
synchronisé avec OneDrive, ne placez pas les sources dans un dossier
synchronisé (ex. `Bureau`, `Documents`). La synchronisation peut
verrouiller des fichiers pendant la compilation et provoquer des
erreurs difficiles à diagnostiquer. Préférez un chemin simple comme
`C:\openrpgmaker-src`.

## Étape 5 — Compiler

Toujours dans le même terminal, en étant bien positionné dans le dossier
qui contient `OpenRPGMaker.pro` :

```sh
qmake-qt5 OpenRPGMaker.pro
mingw32-make
```

(Le paquet MSYS2 installe l'exécutable sous le nom `qmake-qt5` et non
`qmake`, pour éviter tout conflit avec une éventuelle installation Qt6
en parallèle.)

- La compilation complète (éditeur + 7 plugins d'import) prend
  généralement entre 10 et 25 minutes selon la machine.
- En cas d'erreur de compilation, copier le message d'erreur complet
  affiché dans le terminal — c'est ce qu'il faudra fournir pour obtenir
  de l'aide (ouvrir une issue sur le dépôt GitHub du projet).
- À la fin, l'exécutable `OpenRPGMaker.exe` est créé à la racine du
  projet (là où se trouve `OpenRPGMaker.pro`).

**Erreurs de compilation connues et déjà corrigées dans ce fork** : les
versions de ce dépôt intègrent déjà les correctifs nécessaires pour
compiler proprement avec MSYS2 (désactivation des macros `UNICODE`
incompatibles avec la bibliothèque XML embarquée, dans tous les
fichiers `.pro` du projet et des plugins). Si vous partez d'une version
plus ancienne du code et rencontrez une erreur du type
`cannot convert 'const char [...]' to 'const wchar_t*'`, voir la section
"Problèmes fréquents" en fin de guide.

## Étape 6 — Rendre l'exécutable autonome (déploiement)

`OpenRPGMaker.exe`, tel que compilé, dépend de nombreuses DLL qui se
trouvent dans l'environnement MSYS2 mais ne sont pas installées sur un
Windows "normal". Pour pouvoir distribuer/exécuter le programme en
dehors du terminal MSYS2 (double-clic normal depuis l'explorateur
Windows), il faut rassembler ces DLL à côté de l'exécutable.

**Cette étape a été testée en conditions réelles et la méthode
ci-dessous couvre correctement toute la chaîne de dépendances**
(y compris les dépendances indirectes de SDL2_mixer pour la lecture de
divers formats audio, qui ne sont pas toujours détectées en une seule
passe).

1. Créer un dossier de distribution et y copier l'exécutable et les
   données du projet :
   ```sh
   mkdir -p /c/openrpgmaker-dist
   cp OpenRPGMaker.exe /c/openrpgmaker-dist/
   cp -r data /c/openrpgmaker-dist/
   cd /c/openrpgmaker-dist
   ```

2. Copier les plugins Qt indispensables au démarrage de l'application
   (sans le plugin "platforms", l'application ne peut pas s'afficher du
   tout, même si toutes les DLL sont présentes) :
   ```sh
   mkdir -p platforms imageformats
   cp /mingw64/share/qt5/plugins/platforms/qwindows.dll platforms/
   cp /mingw64/share/qt5/plugins/imageformats/*.dll imageformats/
   ```

3. Copier automatiquement **toutes** les DLL nécessaires, en résolvant
   la chaîne complète de dépendances (l'exécutable, les plugins
   d'import, et les DLL déjà copiées peuvent eux-mêmes dépendre
   d'autres DLL) :
   ```sh
   prev_count=-1
   count=0
   while [ "$count" -ne "$prev_count" ]; do
     prev_count=$count
     for f in *.exe *.dll data/importers64/*.dll platforms/*.dll imageformats/*.dll; do
       [ -f "$f" ] && ldd "$f" 2>/dev/null
     done | grep "=> /mingw64/bin" | awk '{print $3}' | sort -u | xargs -I{} cp -n {} /c/openrpgmaker-dist/
     count=$(ls *.dll 2>/dev/null | wc -l)
   done
   echo "Terminé : $count DLL au total à la racine du dossier."
   ```
   Cette boucle relance l'analyse tant que de nouvelles DLL sont
   trouvées — elle s'arrête automatiquement une fois que plus rien de
   nouveau n'apparaît, quelle que soit la profondeur de la chaîne de
   dépendances.

4. **Tester** : depuis l'explorateur Windows (pas le terminal MSYS2),
   double-cliquer sur `OpenRPGMaker.exe` dans `C:\openrpgmaker-dist`.
   L'application doit se lancer et s'afficher normalement.

   Si un message signale encore une DLL manquante (rare après l'étape
   3, mais possible si un nouveau composant est ajouté au projet plus
   tard), noter son nom exact et relancer simplement la boucle de
   l'étape 3 — si la DLL existe bien dans `/mingw64/bin`, elle sera
   trouvée et copiée automatiquement dès qu'un fichier déjà présent
   dans le dossier en dépend explicitement.

## Problèmes fréquents

**`qmake: command not found`**
→ Vous n'êtes pas dans le terminal **"MSYS2 MinGW x64"** (vérifier avec
`echo $MSYSTEM`, qui doit afficher `MINGW64`), ou l'étape 3 a échoué.
Relancer l'étape 3. Utiliser `qmake-qt5` et non `qmake` (voir étape 5).

**Erreur mentionnant `QtWebKit` ou `QtWebKitWidgets` introuvable**
→ Le paquet `mingw-w64-x86_64-qtwebkit` n'a pas été installé
correctement. Relancer :
```sh
pacman -S mingw-w64-x86_64-qtwebkit
```

**Erreur de compilation `cannot convert 'const char [...]' to 'const
wchar_t*'` dans `xmlParser.h` ou un fichier l'utilisant**
→ Qt/MinGW active par défaut les macros `UNICODE`/`_UNICODE` pour les
builds Windows, ce qui bascule la bibliothèque XML embarquée du projet
en mode "grands caractères" (`wchar_t`), incompatible avec le reste du
code qui utilise des chaînes classiques partout. Ce fork désactive déjà
ces macros dans tous les fichiers `.pro` du projet
(`DEFINES -= UNICODE _UNICODE` dans un bloc `win32 { }`). Si l'erreur
apparaît malgré tout, vérifier que cette ligne est bien présente dans
`src/editor/editor.pro` **et** dans les 8 fichiers `.pro` du dossier
`src/editor/importers/*/`, puis supprimer tous les `Makefile` existants
et relancer `qmake-qt5` + `mingw32-make` depuis zéro.

**L'exécutable se lance dans le terminal MSYS2 mais pas depuis
l'explorateur Windows ("DLL introuvable")**
→ Il manque des DLL à côté de l'exécutable : reprendre l'étape 6 dans
son intégralité, en particulier la boucle de l'étape 6.3 qui couvre
l'ensemble de la chaîne de dépendances.

**"This application failed to start because no Qt platform plugin
could be initialized"**
→ Il manque le dossier `platforms/` avec `qwindows.dll` à côté de
l'exécutable (étape 6.2). C'est une erreur différente d'une DLL
manquante : même avec toutes les DLL présentes, Qt a besoin de ce
plugin pour savoir comment afficher une fenêtre sous Windows.

**La compilation est très lente ou semble bloquée**
→ Normal pour la première compilation complète (des centaines de
fichiers). Patienter ; si le terminal n'affiche vraiment plus rien
pendant plus de 10 minutes sans aucune activité disque, interrompre avec
`Ctrl+C` et relancer `mingw32-make` (la compilation reprendra là où
elle s'était arrêtée).

## Pour les mises à jour futures du projet

Si le code source évolue (nouvelles corrections publiées sur GitHub) :
```sh
cd /c/openrpgmaker-src   # ou l'emplacement choisi à l'étape 4
git pull
qmake-qt5 OpenRPGMaker.pro
mingw32-make
```
Puis recopier le nouvel `OpenRPGMaker.exe` par-dessus celui du dossier
de distribution (étape 6, point 1) — inutile de refaire les étapes 6.2
et 6.3 si les dépendances (DLL) n'ont pas changé entre-temps.
