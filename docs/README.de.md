# svn-precommit-review Anleitung

`svn-precommit-review` ist eine Codex Agent Skill zum Prüfen von SVN/TortoiseSVN-Änderungen vor dem Commit und zum Erstellen eines koreanischen Commit-Message-Entwurfs. Standardmäßig ist die Prüfung read-only; wenn der Benutzer Commit-Vorbereitung oder auto-add ausdrücklich anfordert, darf der Skill `svn add` nur für notwendige neue Dateien ausführen, die sonst Build/Runtime brechen könnten und die Sicherheitskriterien erfüllen. Nach der Prüfung ist `svn delete` nur für exakte versionierte Pfade erlaubt, deren Löschung erneut ausdrücklich angefordert wurde und die die Sicherheitskriterien erfüllen.

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

Safe add notwendiger neuer Dateien:

```text
$svn-precommit-review Prüfe die Änderungen in <SVN_WORKING_COPY> und führe svn add nur für notwendige neue Dateien aus, wenn sie den Build brechen könnten und die Sicherheitskriterien erfüllen.
```

## Prüfungen

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- Build-Artefakte, IDE-Dateien, Logs, temporäre Dateien, lokale Einstellungen
- Risiken durch `.env`, Tokens, Passwörter, API Keys und andere Secrets
- Konfliktmarker und missing/deleted Dateien
- sicheres `svn add` für notwendige Build-Dateien nur nach ausdrücklicher Anfrage
- sicheres `svn delete` nur für exakt geprüfte Pfade nach erneuter ausdrücklicher Anfrage
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
