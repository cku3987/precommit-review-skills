# Руководство svn-precommit-review

`svn-precommit-review` — это Codex Agent Skill для проверки изменений SVN/TortoiseSVN перед коммитом и подготовки черновика сообщения коммита на корейском языке. По умолчанию проверка только читает данные; если пользователь явно просит подготовить коммит или auto-add, навык может выполнить `svn add` только для обязательных новых файлов, без которых сборка или запуск могут сломаться, и только после проверок безопасности. После проверки `svn delete` разрешен только для точных versioned-путей, которые пользователь повторно запросил удалить и которые прошли проверки безопасности.

## Установка

На уровне пользователя:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

На уровне проекта:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Скопируйте `svn-precommit-review/SKILL.md` из этого репозитория в нужное место.

## Использование

```text
$svn-precommit-review Проверь текущие изменения SVN перед коммитом и подготовь корейское сообщение коммита.
```

Для конкретной working copy:

```text
$svn-precommit-review Проверь svn status и svn diff в <SVN_WORKING_COPY> и дай рекомендацию по коммиту.
```

Safe add для обязательных новых файлов:

```text
$svn-precommit-review Проверь изменения в <SVN_WORKING_COPY> и добавь через svn add только обязательные новые файлы, если без них может сломаться сборка и они проходят проверки безопасности.
```

## Что проверяется

- структурная проверка через `svn status --xml`, `svn status -u --xml`, diff содержимого и свойств
- build outputs, IDE files, logs, temporary files, local settings
- риски `.env`, токенов, паролей, API-ключей и других секретов; значения всегда маскируются как `[REDACTED]`
- конфликты текста, свойств и дерева, а также состояния replaced, obstructed, switched, external, nested working copy и lock
- safe `svn add` for required build files only after explicit user request
- безопасный `svn delete` только для точных неизмененных локальных путей; не для локально измененных файлов или URL репозитория
- SHA-256 snapshot статуса, diff, свойств, целей и содержимого файлов перед подтверждением и непосредственно перед коммитом
- по умолчанию коммит отдельных файлов; каталог разрешен только после проверки всех потомков и границ SVN
- пустые категории вывода кратко обозначаются как `- 없음`
- в ручном режиме TortoiseSVN Codex не выполняет `svn commit` и не запрашивает разрешение на запуск CLI
- CLI-коммит Codex выполняется только после явного запроса и четкого утвердительного ответа на прямой вопрос о выполнении после проверки целей и сообщения

Короткие четкие ответы вроде `да`, `yes` или `продолжай` разрешают CLI-коммит только тогда, когда режим CLI уже был явно запрошен и Codex только что задал прямой вопрос о выполнении. Запрос только на проверку завершается без такого вопроса.

## Korean Commit Message Encoding

Avoid this in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Для CLI-коммита используйте UTF-8 без BOM, переводы строк LF и `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Получите номер ревизии из результата коммита и проверьте именно эту ревизию:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

Сравните XML `<msg>` с исходным сообщением точно. `--limit 1` не является надежной проверкой при параллельных коммитах.

## Details

Default English guide: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
