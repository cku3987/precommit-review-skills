# svn-precommit-review

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes before commit.

It inspects `svn status` and `svn diff`, checks common commit risks, drafts a Korean commit message, and asks for final human confirmation before any commit. By default the review is read-only. If the user explicitly asks for commit preparation or auto-add, it may run narrow `svn add` only for required new files that would otherwise break build/runtime and pass the safety checks. If the user re-requests deletion for exact reviewed paths, it may run safe `svn delete` only for approved versioned files. It may run `svn commit` only when the user explicitly requests it and approves the exact target paths and message.

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
- Logs, temporary files, cache files, and local config
- `.env`, tokens, passwords, API keys, and other secret-like changes
- Conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`
- Unversioned files that may need manual `svn add` or safe auto-add
- Deleted or missing files that may be accidental, or exact reviewed files that may be safely scheduled for deletion after re-request
- Remote freshness via `svn status -u` when available
- Test/build verification gaps

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

Examples:

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

Review and then request a guarded commit:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토한 뒤, 문제가 없으면 커밋 전 최종 확인을 받고 승인된 파일만 커밋해줘.
```

Review and safely add required build files:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고, 빌드가 깨질 수 있는 필수 신규 파일은 안전 기준을 통과하면 svn add까지 해줘.
```

Review first, then safely schedule exact reviewed files for deletion:

```text
$svn-precommit-review 방금 검토한 path/to/obsolete-file.cs 삭제가 의도된 것이 맞으니, 안전 기준을 통과하면 svn delete로 삭제 예약해줘.
```

Approve exact targets after review:

```text
포함 대상은 path/to/changed-file.cs만입니다.
제외 대상은 build outputs, IDE files, logs, temp files입니다.
추천 커밋 메시지로 커밋을 진행해줘.
```

## Korean Commit Message Encoding

Avoid this in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

For CLI commits, use a UTF-8 message file and explicitly pass `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

After commit, verify the stored log message with XML output:

```powershell
svn log --xml --limit 1
```

Plain `svn log` can look correct even when the stored `svn:log` value is broken by console encoding.

## Documentation

- [English Markdown guide](docs/svn-precommit-review-guide.md)

## Publishing Note

This public package uses generic paths such as `<SVN_WORKING_COPY>` and `path/to/changed-file.cs`. Before publishing your own fork, scan Markdown and any generated artifacts you add so local paths or project-specific details do not leak.

## License

MIT
