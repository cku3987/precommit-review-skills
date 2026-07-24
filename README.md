# svn-precommit-review

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes before commit.

It inspects structured `svn status` and `svn diff` data, checks common commit risks, drafts a Korean commit message, and asks for final human confirmation before any CLI commit. By default the review is read-only. If the user explicitly asks for commit preparation or auto-add, it may run narrow `svn add` only for required new files that would otherwise break build/runtime and pass the safety checks. If the user re-requests deletion for exact reviewed paths, it may run safe `svn delete` only for clean approved versioned files. Manual TortoiseSVN mode never runs `svn commit` or asks for CLI execution approval. Codex CLI commit mode is separate and requires an explicit request for Codex to commit followed by a clear affirmative answer to the direct execution question shown with the exact targets and message.

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
- Text, property, and tree conflicts; replaced, obstructed, switched, external, nested-working-copy, and lock states
- Property changes including `svn:externals`, `svn:ignore`, and `svn:eol-style`
- Test/build verification gaps

## Commit Safety

- `svn status --xml` and `svn status -u --xml` are parsed structurally.
- Before asking for CLI approval, the skill fingerprints status, target diffs, property diffs, target paths, and approved file bytes with SHA-256. It repeats the snapshot immediately before commit and stops if anything changed; no message file is created before this comparison passes.
- Locally modified files are never auto-deleted, and `svn delete` never receives a repository URL.
- Commits prefer explicit file paths. Directory scope is accepted only after every changed descendant is reviewed and externals, nested working copies, switched paths, conflicts, and obstructions are absent.
- Secret candidates are reported by path and risk type only; values are replaced with `[REDACTED]`.
- The committed revision is parsed from the commit result and verified with `svn log --xml -r <revision> <repository-root-url>`. The skill never uses the latest log entry as proof.

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

After Codex shows the exact targets and message and asks whether to execute the CLI commit:

```text
응, 진행해.
```

A clear affirmative reply such as `yes`, `네`, `응`, or `진행해` authorizes the commit only when Codex CLI commit mode was already explicitly requested and Codex has just asked the direct execution question. For a review-only or manual TortoiseSVN request, Codex stops after the checklist without asking that question.

## Korean Commit Message Encoding

Avoid this in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

For CLI commits, use a BOM-free UTF-8 message file with LF line endings and explicitly pass `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

After commit, parse the committed revision from that commit command, obtain the repository root URL, and verify that exact revision:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

Compare the XML `<msg>` value with the original message exactly. Plain `svn log` can look correct even when the stored `svn:log` value is broken by console encoding, while `--limit 1` may select another user's concurrent commit.

## Documentation

- [English Markdown guide](docs/svn-precommit-review-guide.md)

## Publishing Note

This public package uses generic paths such as `<SVN_WORKING_COPY>` and `path/to/changed-file.cs`. Before publishing your own fork, scan Markdown and any generated artifacts you add so local paths or project-specific details do not leak.

## License

MIT
