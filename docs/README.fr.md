# Guide svn-precommit-review

`svn-precommit-review` est une compétence Codex Agent qui vérifie une copie de travail SVN/TortoiseSVN avant le commit et rédige un message de commit en coréen. Par défaut, elle est en lecture seule. Lorsqu'un utilisateur demande explicitement un commit Codex CLI, elle peut add les nouveaux fichiers source nécessaires, puis ne committer que le périmètre approuvé après une seule confirmation finale.

## Installation

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

ou :

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copiez `svn-precommit-review/SKILL.md` de ce dépôt vers l'emplacement choisi.

## Utilisation

Vérification en lecture seule :

```text
$svn-precommit-review Vérifie les changements SVN actuels avant le commit et crée un message de commit en coréen.
```

Demander un commit CLI :

```text
$svn-precommit-review Vérifie les changements de <SVN_WORKING_COPY>, add les nouveaux fichiers source nécessaires et, après confirmation finale, commit par CLI uniquement le périmètre approuvé.
```

Après l'affichage du périmètre et du message, donnez une réponse claire une seule fois.

```text
네, 진행해
```

## Comportement

- Vérifie `svn status --xml`, l'état distant, les diffs de contenu et de propriétés.
- Vérifie les fichiers générés, les fichiers IDE/utilisateur, les journaux, la configuration locale, les données sensibles, les conflits, les fichiers manquants et les lacunes de tests.
- Lors d'une demande de commit CLI, peut lancer `svn add` pour des fichiers source, projets, ressources et configurations nécessaires dont l'intention est claire.
- N'ajoute pas `bin`, `obj`, journaux, caches, archives, `.env`, fichiers susceptibles de contenir des secrets, fichiers générés ou ambigus.
- Si l'utilisateur demande d'inclure les candidats à la suppression examinés, seule la suppression de chemins locaux versioned propres est planifiée.
- Ne supprime pas de chemins modifiés localement, en conflit, switched, external, locked ou ambigus.
- Avant le commit, vérifie à nouveau l'état et les diffs ciblés ; elle s'arrête et recommence l'examen uniquement si le périmètre approuvé, le contenu significatif ou un état SVN bloquant a changé.
- N'utilise pas de `svn add`, `svn delete` ni `svn commit` à large périmètre.

## Messages de commit en coréen

Pour les commits CLI, utilise un fichier temporaire UTF-8 sans BOM et `--encoding UTF-8` :

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Le fichier temporaire est supprimé après chaque tentative. Lorsque l'accès au dépôt le permet, le message XML de la révision réellement créée est comparé à l'original. Si le coréen stocké est endommagé, la compétence signale un échec de vérification d'encodage et ne réessaie pas le commit ni ne modifie les revision properties.
