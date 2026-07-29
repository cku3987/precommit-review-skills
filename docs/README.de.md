# svn-precommit-review Anleitung

`svn-precommit-review` ist ein Codex-Agent-Skill zum Prüfen einer SVN/TortoiseSVN-Arbeitskopie vor dem Commit und zum Erstellen einer koreanischen Commit-Nachricht. Standardmäßig arbeitet er schreibgeschützt. Bei einer ausdrücklichen Anfrage für einen Codex-CLI-Commit kann er erforderliche neue Quelldateien sicher adden und nach einer einzigen finalen Bestätigung nur den genehmigten Umfang committen.

## Installation

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

oder:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Kopieren Sie `svn-precommit-review/SKILL.md` aus diesem Repository an den gewünschten Ort.

## Verwendung

Schreibgeschützte Prüfung:

```text
$svn-precommit-review Prüfe die aktuellen SVN-Änderungen vor dem Commit und erstelle eine koreanische Commit-Nachricht.
```

CLI-Commit anfordern:

```text
$svn-precommit-review Prüfe die Änderungen in <SVN_WORKING_COPY>, add die erforderlichen neuen Quelldateien und committe nach der finalen Bestätigung per CLI nur den genehmigten Umfang.
```

Nach Anzeige von Umfang und Nachricht genügt eine eindeutige Antwort.

```text
네, 진행해
```

## Verhalten

- Prüft `svn status --xml`, den Remote-Status, Inhaltsdifferenzen und Property-Differenzen.
- Prüft generierte Dateien, IDE-/Benutzerdateien, Logs, lokale Konfiguration, sensible Daten, Konflikte, fehlende Dateien und fehlende Tests.
- Bei einer CLI-Commit-Anfrage kann `svn add` für eindeutig erforderliche neue Quellen, Projektdateien, Ressourcen und Konfiguration ausgeführt werden.
- `bin`, `obj`, Logs, Caches, Archive, `.env`, Dateien mit möglichen Geheimnissen, generierte und mehrdeutige Dateien werden nicht hinzugefügt.
- Wenn der Benutzer geprüfte Löschkandidaten einschließen möchte, werden nur saubere versioned lokale Pfade zur Löschung vorgemerkt.
- Lokal geänderte, konfliktbehaftete, switched, external, locked oder mehrdeutige Pfade werden nicht gelöscht.
- Unmittelbar vor dem Commit werden Status und Ziel-Diffs erneut geprüft; nur bei geändertem genehmigtem Umfang, wesentlichem Inhalt oder einem blockierenden SVN-Zustand wird angehalten und erneut geprüft.
- Es werden keine breit gefassten `svn add`, `svn delete` oder `svn commit` verwendet.

## Koreanische Commit-Nachrichten

Für CLI-Commits wird eine temporäre UTF-8-Datei ohne BOM mit `--encoding UTF-8` verwendet:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Die temporäre Datei wird nach jedem Versuch gelöscht. Wenn der Repositoryzugriff es zulässt, wird die XML-Nachricht der tatsächlich erstellten Revision mit dem Original verglichen. Ist der gespeicherte koreanische Text beschädigt, meldet der Skill einen Fehler bei der Kodierungsprüfung und wiederholt den Commit nicht und ändert keine Revision Properties.
