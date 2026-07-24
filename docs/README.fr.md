# Guide svn-precommit-review

`svn-precommit-review` est une Codex Agent Skill qui vérifie les changements SVN/TortoiseSVN avant commit et prépare un brouillon de message de commit en coréen. Par défaut, la revue est en lecture seule; si l’utilisateur demande explicitement une préparation de commit ou un auto-add, elle peut exécuter `svn add` uniquement pour les nouveaux fichiers nécessaires qui pourraient casser le build/runtime et qui passent les contrôles de sécurité. Après la revue, `svn delete` n’est autorisé que pour des chemins versionnés exacts dont la suppression est redemandée explicitement et qui passent les contrôles de sécurité.

## Installation

Au niveau utilisateur:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

Au niveau projet:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copiez `svn-precommit-review/SKILL.md` depuis ce dépôt vers l’emplacement souhaité.

## Utilisation

```text
$svn-precommit-review Vérifie les changements SVN actuels avant commit et prépare un message de commit en coréen.
```

Pour une working copy précise:

```text
$svn-precommit-review Vérifie svn status et svn diff dans <SVN_WORKING_COPY> et donne une recommandation de commit.
```

Safe add des fichiers nécessaires:

```text
$svn-precommit-review Vérifie les changements dans <SVN_WORKING_COPY> et ajoute avec svn add uniquement les nouveaux fichiers nécessaires s'ils peuvent casser le build et passent les contrôles de sécurité.
```

## Contrôles effectués

- revue structurée avec `svn status --xml`, `svn status -u --xml`, diff de contenu et de propriétés
- artefacts de build, fichiers IDE, logs, fichiers temporaires, configuration locale
- risques liés à `.env`, tokens, mots de passe, API keys et autres secrets; les valeurs sont toujours masquées par `[REDACTED]`
- conflits de texte, propriétés et arbre, ainsi que les états replaced, obstructed, switched, external, nested working copy et lock
- `svn add` sécurisé pour les fichiers nécessaires au build uniquement sur demande explicite
- `svn delete` sécurisé uniquement pour les chemins locaux exacts et non modifiés; jamais pour un fichier modifié localement ni une URL du dépôt
- snapshot SHA-256 de l’état, des diffs, des propriétés, des cibles et du contenu des fichiers avant validation puis juste avant commit
- commits par fichier par défaut; une cible répertoire exige la revue de tous les descendants et frontières SVN
- les catégories vides sont indiquées brièvement par `- 없음`
- en mode de commit manuel TortoiseSVN, Codex n’exécute pas `svn commit` et ne demande pas d’autorisation d’exécution CLI
- le commit CLI par Codex est exécuté uniquement après une demande explicite et une réponse affirmative claire à la question directe d’exécution, après vérification des cibles et du message

Les réponses brèves et claires comme `oui`, `yes` ou `continue` autorisent le commit CLI uniquement si le mode CLI a déjà été demandé explicitement et si Codex vient de poser la question directe d’exécution. Une simple demande de revue se termine sans cette question.

## Encodage des messages coréens

Évitez ceci dans PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Pour un commit CLI, utilisez un fichier UTF-8 sans BOM avec fins de ligne LF et `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Lisez la révision dans le résultat du commit et vérifiez précisément cette révision:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

Comparez exactement le `<msg>` XML au message original. `--limit 1` n’est pas une preuve valable en présence de commits simultanés.

## Détails

Guide anglais par défaut: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
