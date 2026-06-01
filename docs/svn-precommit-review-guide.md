# svn-precommit-review Guide

## 1. Overview

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes immediately before commit.

The skill is designed to:

- inspect current working-copy changes with `svn status` and `svn diff`
- classify commit readiness as `가능`, `조건부 가능`, or `보류 권장`
- detect build outputs, IDE/user-specific files, logs, temporary files, local settings, sensitive information, conflict markers, and suspicious missing files
- draft a Korean commit message
- ask for final human confirmation of included targets, excluded targets, remaining risks, and the exact commit message
- run a guarded CLI commit only when the user explicitly requests it and approves the exact targets and message

The default behavior is read-only review. The skill does not automatically run `svn add`, `svn update`, `svn revert`, or `svn resolve`.

## 2. Installation

User-level Codex skill location:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

Project-local skill location:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copy `svn-precommit-review/SKILL.md` from this repository to either location.

## 3. Requirements

The SVN command-line client must be available from the shell.

```powershell
svn --version --quiet
svn status
```

TortoiseSVN users can install the command-line client during TortoiseSVN setup, or use another Apache Subversion CLI installation.

For a complete pre-commit review, provide the SVN working-copy root path.

## 4. Review-Only Usage

Review the current working copy:

```text
$svn-precommit-review 현재 SVN 변경사항을 커밋 전에 검토하고 한국어 커밋 메시지를 작성해줘.
```

Review a specific working copy:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로의 SVN 변경사항을 커밋 전에 검토해줘.
```

The skill checks:

- `svn info`
- `svn status`
- `svn status -u`
- `svn diff --summarize`
- `svn diff`
- conflict markers and likely sensitive information when relevant

If `svn status -u` fails because of network or authentication problems, the skill should continue and report remote freshness as not verified.

## 5. Output Format

The final review output is intentionally Korean-first because the target workflow is Korean SVN/TortoiseSVN commit review.

```markdown
# SVN 커밋 전 검토 결과

## 1. 커밋 가능 여부

판단: 가능 / 조건부 가능 / 보류 권장

## 2. 변경 파일 요약

### Modified
### Added / Unversioned
### Deleted / Missing
### Conflicted
### Out-of-date / Remote-changed
### Other

## 3. 커밋 전 확인 필요 항목

## 4. 변경 내용 요약

## 5. 추천 커밋 메시지

## 6. TortoiseSVN에서 할 일
```

If the user explicitly asks the skill to commit and approves the exact targets and message, the skill appends:

```markdown
## 7. 커밋 실행 결과
```

## 6. Readiness Judgment

`가능`:

- no meaningful risk is visible
- no conflicted files are present
- required files are tracked or the commit target is explicit
- no likely sensitive information is visible
- verification is sufficient for the size and risk of the change

`조건부 가능`:

- the commit appears reasonable, but a human must confirm specific items
- examples: unversioned files that may need `svn add`, skipped tests, remote freshness not verified, or intentional deletions

`보류 권장`:

- conflicts, likely secrets, accidental generated files, suspicious missing files, or out-of-date risk are present
- the change intent cannot be understood with enough confidence
- committing may break build, runtime, or operations

## 7. Guarded Commit Mode

The skill does not commit automatically after review.

To commit, the user must explicitly ask for it and approve:

- included target paths
- excluded paths or file groups
- remaining risks
- exact commit message

Example review request:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토해줘.
```

Example approval after review:

```text
포함 대상은 path/to/changed-file.cs만입니다.
제외 대상은 build outputs, IDE files, logs, temp files입니다.
추천 커밋 메시지를 사용해서 커밋해줘.
```

Immediately before commit, the skill must run `svn status` again. If new unapproved changes, conflicts, likely secrets, or unclear missing/deleted files appear, it must refuse to commit.

## 8. Korean Commit Message Encoding

Avoid this in PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

For CLI commits, use a UTF-8 message file and explicitly pass `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Using `--file` without `--encoding UTF-8` may still store a broken Korean `svn:log` value in some environments.

After commit, verify the stored log message with XML output:

```powershell
svn log --xml --limit 1
```

Plain `svn log` can look correct because of console encoding even when the repository `svn:log` value is broken.

## 9. Temporary Message File Cleanup

When a UTF-8 commit message file is used:

- create it outside the SVN working copy when possible
- delete it whether the commit succeeds or fails
- confirm it does not remain as an unversioned `?` file in `svn status`
- never accidentally `svn add` a temporary message file

The skill may create a temporary message file only in guarded commit mode. It must not create or modify project files.

## 10. Already-Broken Commit Messages

If a committed `svn:log` message is already broken, fixing it requires a revision property change:

```powershell
svn propset --revprop svn:log -r <revision> --file <message-file> <repository-url>
```

Some servers block revision property changes unless a `pre-revprop-change` hook is configured. In that case, a client cannot repair the message alone.

Typical error:

```text
Repository has not been enabled to accept revision propchanges
```

Ask the repository administrator to enable the appropriate `pre-revprop-change` hook if message repair is required.

## 11. Prompt Examples

Review only:

```text
$svn-precommit-review 현재 SVN working copy 변경사항을 커밋 전에 검토하고, 최종 결과는 한국어로 작성해줘.
```

Review a specific path:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로에서 svn status와 svn diff를 확인하고 커밋 가능 여부와 추천 커밋 메시지를 작성해줘.
```

Review and prepare a guarded commit:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토한 뒤, 문제가 없으면 커밋 전 최종 확인을 받고 승인된 파일만 커밋해줘.
```

Approve exact targets after review:

```text
포함 대상은 path/to/changed-file.cs만입니다.
제외 대상은 build outputs, IDE files, logs, temp files입니다.
추천 커밋 메시지로 커밋을 진행해줘.
```

## 12. Safety Notes

- The skill does not automatically run `svn add`.
- Unversioned files must be manually selected by a human.
- Missing or deleted files must be confirmed as intentional.
- Do not commit if conflicts are present.
- Do not commit if likely sensitive information is present.
- Commit only explicitly approved target paths.
- Avoid broad working-copy commits.
- For Korean CLI commit messages, use `--file <message-file> --encoding UTF-8`.
