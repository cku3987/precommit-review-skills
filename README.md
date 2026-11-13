# precommit-review-skills

Independent **Git** and **SVN/TortoiseSVN** pre-commit skills in one repository. Install either skill or both; each has its own name and a self-contained `SKILL.md`. Reviews and commit messages are written in Korean.

| Skill | Purpose | Guide |
| --- | --- | --- |
| [git-precommit-review](git-precommit-review/SKILL.md) | Review staged/unstaged changes, preserve partial staging, and commit an approved snapshot | [Git guide](docs/git-precommit-review-guide.md) |
| [svn-precommit-review](svn-precommit-review/SKILL.md) | Review working-copy and property changes, prepare required files, and commit approved paths | [SVN guide](docs/svn-precommit-review-guide.md) |

Both default to review plus safe preparation of clear required new files. Explicit read-only requests make no changes. Actual commits require a commit request and one final confirmation of scope and message. Git push requires an explicit request; a local commit does not authorize publication. No product source edits, broad staging, automatic conflict resolution, or history rewriting.

Agent-created verification-only scripts or test projects are excluded unless explicitly requested as deliverables; test success does not make them required product files. Requested regression tests in existing versioned suites remain eligible.

The repository was previously named `svn-precommit-review`. The SVN skill name and its `svn-precommit-review/SKILL.md` path are unchanged. Releases version the collection; release notes identify which skill changed. No plugin manifest or shared runtime is required for individual skill installation.

## Translations

- [한국어](docs/README.ko.md)
- [日本語](docs/README.ja.md)
- [简体中文](docs/README.zh-CN.md)
- [繁體中文](docs/README.zh-TW.md)
- [Русский](docs/README.ru.md)
- [Español](docs/README.es.md)
- [Français](docs/README.fr.md)
- [Deutsch](docs/README.de.md)

## Git Behavior

- Inspect both staged blobs and working files, including required untracked files, secrets, deletions, renames, binary/mode changes, submodules and test gaps.
- Preserve existing partial staging. Preparing a commit does not silently stage the rest of a partially staged file.
- Keep unrelated staging out of a scoped commit using a temporary index when needed, without resetting the real staging area.
- Use an explicitly UTF-8 message file, verify the exact commit tree and message, and remove owned temporary files after success or failure.
- Preserve hooks and signing. Report hook changes or verification failures without retrying or amending automatically.
- Push only on request, to an explicit destination, without force or tags. Report cached upstream state and remote failures honestly.

## SVN Checks

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

## SVN Commit Behavior

- A bare skill invocation or general review request includes safe preparation of required new files. Explicit review-only, read-only, message-only, and no-change requests make no writes.
- Required source, project, resource, and non-local configuration files are automatically added when tracked references or the reviewed change establish their purpose and scope. Generated, ignored, local-only, ambiguous, excluded, or sensitive files are not added.
- Add is reported as completed only after command success and SVN `A` status verification. Successfully prepared files are not repeated as unresolved missing-source warnings. Independent code risks remain visible but do not prevent safe file preparation.
- Manual TortoiseSVN requests allow this preparation unless read-only was requested, but never trigger a CLI commit.
- If the user asks to include reviewed deletions, the skill may schedule the listed clean versioned local paths. Locally modified, conflicted, switched, external, locked, or ambiguous files remain excluded.
- The skill shows included paths, excluded paths, prepared adds/deletes, remaining risks, and the Korean message before committing.
- A natural commit request such as `커밋해줘` is enough to enter CLI mode; no `CLI` keyword is required. After the final scope-and-message question, a clear reply such as `yes`, `네`, `응`, `진행해`, or `커밋해` authorizes that commit.
- Immediately before committing, the skill repeats status and relevant diffs. Existing final approval is reused across turns unless meaningful scope/content or blocking state changes require a new review.
- Commits use explicit paths only. Broad `svn add`, `svn delete`, and `svn commit` are never used.
- Secret candidates are reported by path and risk type only; values are replaced with `[REDACTED]`.

## Install

Copy the chosen skill folder into your agent's skill directory. Existing user-level Codex installation layout:

```text
%USERPROFILE%\.codex\skills\git-precommit-review\SKILL.md
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

Project-local Codex Agent skill:

```text
<project-root>\.agents\skills\git-precommit-review\SKILL.md
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

For Claude Code, use `.claude/skills/<skill-name>/SKILL.md` in the project or `~/.claude/skills/<skill-name>/SKILL.md` for user-level installation. Keep the two names distinct. Each installable folder contains only its `SKILL.md`.

You can also ask Codex's skill installer:

```text
$skill-installer Install git-precommit-review and svn-precommit-review from https://github.com/cku3987/precommit-review-skills, using their respective repository folders.
```

## Requirements

The matching command-line client (`git` or `svn`) must be available from your shell. The skills themselves do not require Python or a YAML package.

```powershell
git --version
svn --version --quiet
```

TortoiseSVN users can install the command-line client during TortoiseSVN setup, or use another Apache Subversion CLI installation.

## Usage

Git review, read-only review, or commit request:

```text
$git-precommit-review <GIT_REPOSITORY> 변경사항을 검토해줘.
$git-precommit-review <GIT_REPOSITORY> 읽기 전용으로 검토만 해줘.
$git-precommit-review <GIT_REPOSITORY> 변경사항을 커밋해줘.
```

In Claude Code, invoke `/git-precommit-review` or `/svn-precommit-review` instead. The following examples are for SVN:

Default review and required-file preparation:

```text
$svn-precommit-review 현재 SVN working copy 변경사항을 검토해줘.
```

Read-only review (no add, delete, or commit):

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로를 읽기 전용으로 검토만 하고 한국어 커밋 메시지를 작성해줘. svn add/delete/commit은 실행하지 마.
```

Review, prepare clear required files, and request a CLI commit:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 커밋해줘.
```

Include reviewed deletion candidates:

```text
$svn-precommit-review 방금 검토한 삭제 후보도 포함해서 커밋 준비해줘.
```

After Codex shows the scope and message, reply once:

```text
네, 진행해
```

Summaries group changes by intent, not a fixed bullet count. Prior test results are reused only with evidence that they still cover the reviewed content and are labeled as not rerun. The skill does not run builds or tests that would modify the working copy.

## SVN Korean Commit Message Encoding

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

- [Git guide](docs/git-precommit-review-guide.md)
- [SVN guide](docs/svn-precommit-review-guide.md)

## Publishing Note

This public package uses generic paths such as `<GIT_REPOSITORY>`, `<SVN_WORKING_COPY>` and `path/to/changed-file.cs`. Before publishing your own fork, scan Markdown and generated artifacts so local paths or project-specific details do not leak.

## License

MIT
