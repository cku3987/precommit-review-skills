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

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- artefacts de build, fichiers IDE, logs, fichiers temporaires, configuration locale
- risques liés à `.env`, tokens, mots de passe, API keys et autres secrets
- marqueurs de conflit et fichiers missing/deleted
- `svn add` sécurisé pour les fichiers nécessaires au build uniquement sur demande explicite
- `svn delete` sécurisé uniquement pour les chemins exacts revus après nouvelle demande explicite
- commit CLI protégé uniquement après demande explicite et validation finale

## Encodage des messages coréens

Évitez ceci dans PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Pour un commit CLI, utilisez un fichier UTF-8 et `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Vérifiez le log enregistré avec:

```powershell
svn log --xml --limit 1
```

## Détails

Guide anglais par défaut: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
