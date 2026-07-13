# svn-precommit-review Guide

## 1. Overview

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes immediately before commit.

The skill is designed to:

- inspect current working-copy changes with `svn status` and `svn diff`
- classify commit readiness as `가능`, `조건부 가능`, or `보류 권장`
- detect build outputs, IDE/user-specific files, logs, temporary files, local settings, sensitive information, conflict markers, and suspicious missing files
- safely `svn add` required new files that would otherwise break build/runtime when the user explicitly asks for commit preparation or auto-add
- safely `svn delete` exact reviewed versioned files only when the user re-requests deletion or explicitly includes those delete targets in commit preparation
- draft a Korean commit message
- ask for final human confirmation of included targets, excluded targets, remaining risks, and the exact commit message
- run a guarded CLI commit only when the user explicitly requests it and approves the exact targets and message

The default behavior is read-only review. The skill does not run broad `svn add`, broad `svn delete`, `svn update`, `svn revert`, or `svn resolve`. If the user explicitly asks for commit preparation, auto-add, or commit, the skill may run `svn add` only for required new files that pass the safe-add criteria. If the user re-requests deletion after review, the skill may run `svn delete` only for exact approved versioned paths that pass the safe-delete criteria.

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

Review and safely add required build files:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고, 빌드가 깨질 수 있는 필수 신규 파일은 안전 기준을 통과하면 svn add까지 해줘.
```

After review, safely schedule exact reviewed files for deletion:

```text
$svn-precommit-review 방금 검토한 path/to/obsolete-file.cs 삭제가 의도된 것이 맞으니, 안전 기준을 통과하면 svn delete로 삭제 예약해줘.
```

The skill checks:

- `svn info`
- `svn status`
- `svn status -u`
- `svn diff --summarize`
- `svn diff`
- conflict markers and likely sensitive information when relevant

If `svn status -u` fails because of network or authentication problems, the skill should continue and report remote freshness as not verified.

## 5. Safe Add Mode

Safe add mode is enabled only when the user explicitly asks for commit preparation, auto-add, or commit. It is not used for a review-only request.

The skill may run `svn add <path>` only when all of these are true:

- the file is unversioned and inside the intended SVN working copy
- the file is not ignored by SVN and is not under `.svn`
- the file is not generated output, IDE/user-specific state, logs, temp/cache files, local-only config, archives, backups, screenshots, or other commit-excluded material
- the file is not `.env` and does not look like it contains tokens, passwords, API keys, private keys, connection strings, or other secrets
- the file is likely required to avoid a broken build or runtime because it is referenced by tracked solution/project/build/config files, belongs to a newly referenced project, or is a source/config/resource file in a tracked build path
- the command targets explicit file paths; broad `svn add .`, `svn add *`, and uninspected directory adds are forbidden

If the file is ambiguous, the skill must not add it. It should report the file as `수동 확인 후 add 필요`.

## 6. Safe Delete Mode

Safe delete mode is enabled only after review when the user explicitly re-requests deletion for exact paths, or when the user explicitly asks for commit preparation with exact delete targets. It is not used for a review-only request.

The skill may run `svn delete <path>` only when all of these are true:

- the path is versioned and inside the intended SVN working copy
- the path is not under `.svn`
- the path is not conflicted, out-of-date, or remotely changed
- the deletion intent is clear and the likely build/runtime impact has been reported
- the command targets explicit file paths; broad `svn delete .`, `svn delete *`, and uninspected directory deletes are forbidden
- if the file has local modifications, the user explicitly approved scheduling that modified file for deletion
- if the file is already missing (`!`), `svn delete --force` is allowed only for that exact approved missing path

If the file is ambiguous, the skill must not delete it. It should report the file as `삭제 예약 보류, 수동 확인 필요`.

## 7. Output Format

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

## 8. Readiness Judgment

`가능`:

- no meaningful risk is visible
- no conflicted files are present
- required files are tracked or the commit target is explicit
- no likely sensitive information is visible
- verification is sufficient for the size and risk of the change

`조건부 가능`:

- the commit appears reasonable, but a human must confirm specific items
- examples: unversioned files that need manual confirmation, skipped tests, remote freshness not verified, or intentional deletions

`보류 권장`:

- conflicts, likely secrets, accidental generated files, suspicious missing files, or out-of-date risk are present
- the change intent cannot be understood with enough confidence
- committing may break build, runtime, or operations

## 9. Guarded Commit Mode

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

Immediately before commit, the skill must run `svn status` again. If new unapproved changes, conflicts, likely secrets, or unapproved/unclear missing/deleted files appear, it must refuse to commit. Files added by safe add mode and files scheduled by safe delete mode still require human diff review before commit.

## 10. Korean Commit Message Encoding

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

## 11. Temporary Message File Cleanup

When a UTF-8 commit message file is used:

- create it outside the SVN working copy when possible
- delete it whether the commit succeeds or fails
- confirm it does not remain as an unversioned `?` file in `svn status`
- never accidentally `svn add` a temporary message file

The skill may create a temporary message file only in guarded commit mode. It must not create or modify project files.

## 12. Already-Broken Commit Messages

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

## 13. Prompt Examples

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

Review and safe-add required files:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고, 빌드가 깨질 수 있는 필수 신규 파일은 안전 기준을 통과하면 svn add까지 해줘.
```

Schedule exact reviewed deletion:

```text
$svn-precommit-review 방금 검토한 path/to/obsolete-file.cs 삭제가 의도된 것이 맞으니, 안전 기준을 통과하면 svn delete로 삭제 예약해줘.
```

Approve exact targets after review:

```text
포함 대상은 path/to/changed-file.cs만입니다.
제외 대상은 build outputs, IDE files, logs, temp files입니다.
추천 커밋 메시지로 커밋을 진행해줘.
```

## 14. Safety Notes

- The skill does not run broad `svn add`.
- Safe `svn add` is allowed only when the user explicitly asks for commit preparation, auto-add, or commit.
- The skill does not run broad `svn delete`.
- Safe `svn delete` is allowed only after review and explicit re-request for exact versioned paths.
- Ambiguous unversioned files must be manually selected by a human.
- Missing or deleted files must be confirmed as intentional before deletion scheduling or commit.
- Do not commit if conflicts are present.
- Do not commit if likely sensitive information is present.
- Commit only explicitly approved target paths.
- Avoid broad working-copy commits.
- For Korean CLI commit messages, use `--file <message-file> --encoding UTF-8`.
