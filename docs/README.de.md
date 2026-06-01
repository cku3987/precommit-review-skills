# svn-precommit-review Anleitung

`svn-precommit-review` ist eine Codex Agent Skill zum Prüfen von SVN/TortoiseSVN-Änderungen vor dem Commit und zum Erstellen eines koreanischen Commit-Message-Entwurfs.

## Installation

Benutzerweit:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

Projektlokal:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Kopieren Sie `svn-precommit-review/SKILL.md` aus diesem Repository an den gewünschten Ort.

## Verwendung

```text
$svn-precommit-review Prüfe die aktuellen SVN-Änderungen vor dem Commit und erstelle eine koreanische Commit Message.
```

Für eine bestimmte Working Copy:

```text
$svn-precommit-review Prüfe svn status und svn diff in <SVN_WORKING_COPY> und gib eine Commit-Empfehlung.
```

## Prüfungen

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- Build-Artefakte, IDE-Dateien, Logs, temporäre Dateien, lokale Einstellungen
- Risiken durch `.env`, Tokens, Passwörter, API Keys und andere Secrets
- Konfliktmarker und missing/deleted Dateien
- geschützter CLI-Commit nur nach ausdrücklicher Anfrage und finaler Freigabe

## Encoding koreanischer Commit Messages

Vermeiden Sie dies in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Für CLI-Commits verwenden Sie eine UTF-8-Datei und `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Prüfen Sie den gespeicherten Log mit:

```powershell
svn log --xml --limit 1
```

## Details

Englische Standarddokumentation: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
