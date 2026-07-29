# svn-precommit-review

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes before commit.

It inspects structured `svn status` and `svn diff` data, checks common commit risks, and drafts a Korean commit message. The default is read-only review. When the user requests a Codex CLI commit, the skill can prepare clear required new source files, show the exact scope and message, and commit after one final affirmative reply. It never performs a broad working-copy commit.

## Translations

- [한국어](docs/README.ko.md)
- [日本語](docs/README.ja.md)
- [简体中文](docs/README.zh-CN.md)
- [繁體中文](docs/README.zh-TW.md)
- [Русский](docs/README.ru.md)
- [Español](docs/README.es.md)
- [Français](docs/README.fr.md)
- [Deutsch](docs/README.de.md)

## What It Checks

- Build outputs and generated files
- IDE or user-specific files
- Logs, temporary files, cache files, and local configuration
- `.env`, tokens, passwords, API keys, and other secret-like changes
- Conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`
- Unversioned files that should be added for the reviewed change to build or run
- Deleted or missing files that may be accidental
- Remote freshness via `svn status -u` when available
- Text, property, and tree conflicts; replaced, obstructed, switched, external, nested-working-copy, and lock states
- Property changes including `svn:externals`, `svn:ignore`, and `svn:eol-style`
- Test or build verification gaps

## Commit Behavior

- Review-only and manual TortoiseSVN requests remain read-only.
- For an explicit Codex CLI commit request, the skill may add clear required source, project, resource, and configuration files. Generated, local-only, ambiguous, or sensitive files remain excluded.
- If the user asks to include reviewed deletions, the skill may schedule the listed clean versioned local paths. Locally modified, conflicted, switched, external, locked, or ambiguous files remain excluded.
- The skill shows included paths, excluded paths, prepared adds/deletes, remaining risks, and the Korean message before committing.
- After the final question, a clear reply such as `yes`, `네`, `응`, `진행해`, or `커밋해` authorizes the explicit CLI commit.
- Immediately before committing, the skill repeats status and relevant diffs. It stops only if the approved scope, meaningful content, or a blocking SVN state changed.
- Commits use explicit paths only. Broad `svn add`, `svn delete`, and `svn commit` are never used.
- Secret candidates are reported by path and risk type only; values are replaced with `[REDACTED]`.

## Install

User-level Codex skill:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

Project-local Codex Agent skill:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copy [`svn-precommit-review/SKILL.md`](svn-precommit-review/SKILL.md) to one of those locations.

## Requirements

The `svn` command-line client must be available from your shell.

```powershell
svn --version --quiet
svn status
```

TortoiseSVN users can install the command-line client during TortoiseSVN setup, or use another Apache Subversion CLI installation.

## Usage

Review only:

```text
$svn-precommit-review 현재 SVN working copy 변경사항을 커밋 전에 검토하고, 최종 결과는 한국어로 작성해줘.
```

Review a specific working copy:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로에서 svn status와 svn diff를 확인하고 커밋 가능 여부와 추천 커밋 메시지를 작성해줘.
```

Review, prepare clear required files, and request a CLI commit:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고 필요한 신규 소스 파일은 add한 뒤, 최종 확인을 받으면 승인 범위만 CLI로 커밋해줘.
```

Include reviewed deletion candidates:

```text
$svn-precommit-review 방금 검토한 삭제 후보도 포함해서 커밋 준비해줘.
```

After Codex shows the scope and message, reply once:

```text
네, 진행해
```

## Korean Commit Message Encoding

For CLI commits, the skill writes the exact message to a BOM-free UTF-8 temporary file with LF line endings, then runs:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

It removes the temporary file after the attempt and, when repository access permits, verifies the exact committed revision with:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

If the stored Korean message is broken, the commit is reported as completed with an encoding-verification failure. The skill does not retry the commit or modify revision properties.

## Documentation

- [English Markdown guide](docs/svn-precommit-review-guide.md)

## Publishing Note

This public package uses generic paths such as `<SVN_WORKING_COPY>` and `path/to/changed-file.cs`. Before publishing your own fork, scan Markdown and generated artifacts so local paths or project-specific details do not leak.

## License

MIT
