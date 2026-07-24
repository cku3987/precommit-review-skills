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

- strukturierte Prüfung mit `svn status --xml`, `svn status -u --xml`, Inhalts- und Property-Diffs
- Build-Artefakte, IDE-Dateien, Logs, temporäre Dateien, lokale Einstellungen
- Risiken durch `.env`, Tokens, Passwörter, API Keys und andere Secrets; Werte werden immer als `[REDACTED]` maskiert
- Text-, Property- und Tree-Konflikte sowie replaced, obstructed, switched, external, nested working copy und Lock-Zustände
- sicheres `svn add` für notwendige Build-Dateien nur nach ausdrücklicher Anfrage
- sicheres `svn delete` nur für exakt geprüfte, unveränderte lokale Pfade; keine lokal geänderten Dateien oder Repository-URLs
- SHA-256-Snapshot von Status, Diffs, Properties, Zielpfaden und Dateiinhalten vor Freigabe und erneut direkt vor dem Commit
- standardmäßig dateibasierte Commits; Verzeichnisziele nur nach vollständiger Prüfung aller Nachkommen und SVN-Grenzen
- leere Ausgabekategorien werden kurz mit `- 없음` angegeben
- im manuellen TortoiseSVN-Modus führt Codex kein `svn commit` aus und fragt nicht nach einer CLI-Ausführungsfreigabe
- ein CLI-Commit durch Codex wird nur nach ausdrücklicher Anfrage und einer eindeutigen Zustimmung auf die direkte Ausführungsfrage nach Prüfung von Zielen und Nachricht ausgeführt

Kurze eindeutige Antworten wie `ja`, `yes` oder `weiter` autorisieren den CLI-Commit nur, wenn der CLI-Modus bereits ausdrücklich angefordert wurde und Codex gerade die direkte Ausführungsfrage gestellt hat. Eine reine Prüfung endet ohne diese Frage.

## Encoding koreanischer Commit Messages

Vermeiden Sie dies in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Für CLI-Commits verwenden Sie eine UTF-8-Datei ohne BOM mit LF-Zeilenenden und `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Lesen Sie die Revision aus dem Commit-Ergebnis und prüfen Sie genau diese Revision:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

Vergleichen Sie XML `<msg>` exakt mit der Originalnachricht. `--limit 1` ist wegen möglicher paralleler Commits kein gültiger Nachweis.

## Details

Englische Standarddokumentation: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
