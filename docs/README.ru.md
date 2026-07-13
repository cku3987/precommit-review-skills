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

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- build outputs, IDE files, logs, temporary files, local settings
- `.env`, tokens, passwords, API keys, and other sensitive data risks
- conflict markers and missing/deleted files
- safe `svn add` for required build files only after explicit user request
- safe `svn delete` only for exact reviewed paths after explicit re-request
- guarded CLI commit only after explicit user request and final approval

## Korean Commit Message Encoding

Avoid this in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Use a UTF-8 message file and `--encoding UTF-8` for CLI commits:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Verify the stored log with:

```powershell
svn log --xml --limit 1
```

## Details

Default English guide: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
