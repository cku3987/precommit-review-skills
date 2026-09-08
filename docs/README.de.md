# svn-precommit-review Anleitung

`svn-precommit-review` ist ein Codex-Agent-Skill zum Prüfen einer SVN/TortoiseSVN-Arbeitskopie vor dem Commit und zum Erstellen einer koreanischen Commit-Nachricht. Standardmäßig prüft er die Änderungen und führt automatisch `svn add` für erforderliche neue Dateien aus, die die Sicherheitskriterien erfüllen. Der eigentliche Commit erfordert einen Benutzerauftrag und eine einzige finale Bestätigung von Umfang und Nachricht; nur der genehmigte Umfang wird committet.

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

Standardprüfung und Vorbereitung erforderlicher Dateien (auch bei einer allgemeinen Bitte wie „prüfe“ oder `검토해줘`):

```text
$svn-precommit-review
```

Schreibgeschützte Prüfung:

```text
$svn-precommit-review Prüfe die aktuellen SVN-Änderungen nur lesend. Führe svn add/delete/commit nicht aus.
```

Prüfung und Commit anfordern:

```text
$svn-precommit-review Prüfe die Änderungen in <SVN_WORKING_COPY> und committe sie.
```

Bei einem Commit-Auftrag prüfen Sie eingeschlossene und ausgeschlossene Pfade, vorbereitete add/delete-Aktionen, verbleibende Risiken und die Nachricht und antworten einmal auf die finale Frage. „Committe die Änderungen“ oder `커밋해줘` genügt; das Wort `CLI` ist nicht erforderlich.

```text
네, 진행해
```

## Verhalten

- Prüft `svn status --xml`, den Remote-Status, Inhaltsdifferenzen und Property-Differenzen.
- Prüft generierte Dateien, IDE-/Benutzerdateien, Logs, lokale Konfiguration, sensible Daten, Konflikte, fehlende Dateien und fehlende Tests.
- Bei der Standardprüfung führt der Skill zwingend automatisch `svn add` für unversionierte Quell-, Projekt-, Ressourcen- und nicht rein lokale Konfigurationsdateien aus, deren Notwendigkeit durch Referenzen in versionierten Dateien oder geprüfte Änderungen belegt ist. Sind die Kriterien erfüllt, genügt eine Empfehlung nicht.
- Ausdrückliche Wünsche nach schreibgeschützter Prüfung, ausschließlich Review, ausschließlich Nachricht oder keinerlei Änderungen erlauben keine Schreibzugriffe. Eine allgemeine Prüfbitte umfasst die Standardvorbereitung.
- Ein Hinzufügen gilt erst nach erfolgreichem add-Befehl und bestätigtem Status `A` in `svn status` als abgeschlossen. Abgeschlossene Ergänzungen stehen in der Dateiübersicht, nicht unter offenen Warnungen.
- Ein unabhängiges, sicheres add kann trotz eines verbleibenden Coderisikos erfolgen. Das Risiko wird weiter gemeldet; add wird nicht als Behebung des Codeproblems dargestellt.
- `bin`, `obj`, generierte Dateien, IDE-/Benutzerdateien, Logs, temporäre Dateien, Caches, lokale Konfiguration, Archive, Dumps, Sicherungen, Screenshots, ignorierte Dateien, `.env`, Dateien mit möglichen Geheimnissen und mehrdeutige Dateien werden nicht hinzugefügt. Verzeichnisse werden vorher vollständig rekursiv geprüft; ist nicht der gesamte Inhalt sicher, werden nur konkrete sichere Pfade hinzugefügt.
- Wenn der Benutzer geprüfte Löschkandidaten einschließen möchte, werden nur saubere versioned lokale Pfade zur Löschung vorgemerkt.
- Lokal geänderte, konfliktbehaftete, switched, external, locked oder mehrdeutige Pfade werden nicht gelöscht.
- Der eigentliche Commit erfordert einen Auftrag und die finale Genehmigung des genauen Umfangs und der Nachricht. Eine frühere Genehmigung desselben Umfangs und Inhalts bleibt gültig; unmittelbar vor dem Commit werden Status sowie Inhalts- und Property-Diffs erneut geprüft. Nur bei wesentlichen Änderungen an Umfang oder Inhalt oder einem geänderten blockierenden Zustand wird erneut geprüft und nachgefragt.
- Bei ausdrücklich gewähltem manuellem TortoiseSVN-Commit endet der Ablauf mit Prüfung und Checkliste, ohne CLI-Commit oder entsprechende Rückfrage. Die sichere Standardvorbereitung erfolgt weiterhin, sofern keine schreibgeschützte Prüfung verlangt wurde.
- Es werden keine breit gefassten `svn add`, `svn delete` oder `svn commit` verwendet.

Hinweise während der Ausführung, Prüfung, Commit-Nachricht und Ergebnisse sind ausschließlich auf Koreanisch. Zusammenfassungen werden nach Änderungsabsicht gruppiert, ohne feste Anzahl von Punkten oder überlange Details. Frühere Tests werden nur mit tatsächlichen Belegen für den unveränderten geprüften Inhalt übernommen und als frühere, nicht erneut ausgeführte Ergebnisse gekennzeichnet.

## Koreanische Commit-Nachrichten

Für CLI-Commits wird außerhalb der Arbeitskopie eine eindeutige temporäre Nachrichtendatei in UTF-8 ohne BOM mit LF-Zeilenenden erstellt; verwendet werden `--file` und `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Die temporäre Datei wird nach jedem Versuch gelöscht und ihre Abwesenheit geprüft. Wenn der Repositoryzugriff es zulässt, wird die XML-Log-Nachricht der aus der Commit-Ausgabe ermittelten tatsächlichen Revision Zeichen für Zeichen mit dem Original verglichen. Ist der gespeicherte koreanische Text beschädigt, meldet der Skill einen Fehler bei der Kodierungsprüfung und wiederholt den Commit nicht und ändert keine Revision Properties.
