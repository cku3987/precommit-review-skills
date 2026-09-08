# svn-precommit-review Guide

## Overview

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes immediately before commit. Its review result and commit message are always written in Korean.

The default is review plus safe preparation of required new files. Explicit read-only, review-only, message-only, or no-change requests make no writes. The workflow is:

- automatically add clear required source, project, resource, and non-local configuration files, then verify their SVN `A` state
- schedule reviewed clean deletion candidates when the user asks to include them
- show the exact scope, exclusions, risks, and message
- execute a narrow CLI commit only when a commit was requested and the final scope and message received one affirmative reply

It never uses broad working-copy mutations such as `svn add .`, `svn delete .`, or `svn commit .`.

## Installation

Install the skill at one of these locations:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copy `svn-precommit-review/SKILL.md` from this repository to the chosen location.

## Requirements

The `svn` command-line client must be available in the shell used by Codex.

```powershell
svn --version --quiet
svn status
```

TortoiseSVN users can install the command-line client during TortoiseSVN setup, or install Apache Subversion separately.

## Usage

Default review and required-file preparation:

```text
$svn-precommit-review 현재 SVN 변경사항을 검토해줘.
```

Read-only review of a specific working copy:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로를 읽기 전용으로 검토만 해줘. svn add/delete/commit은 실행하지 마.
```

Prepare clear required files and request a CLI commit:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 커밋해줘.
```

Include reviewed deletion candidates:

```text
$svn-precommit-review 방금 검토한 삭제 후보도 포함해서 커밋 준비해줘.
```

When the skill shows the final included paths and Korean message, answer once with a clear affirmative reply:

```text
네, 진행해
```

## Review Coverage

The skill uses `svn status --xml` as the primary state scan and attempts `svn status -u --xml` for remote freshness. It also reviews summary, content, and property diffs.

It checks for:

- generated files, build output, IDE/user files, logs, caches, local configuration, archives, and backups
- `.env`, credentials, tokens, passwords, API keys, private keys, and connection strings
- conflict markers and text/property/tree conflicts
- unversioned, added, missing, deleted, replaced, obstructed, switched, external, nested working-copy, and lock states
- relevant property changes, including `svn:externals`, `svn:ignore`, `svn:eol-style`, `svn:mergeinfo`, `svn:mime-type`, `svn:needs-lock`, and `svn:executable`
- test and build verification gaps

Remote-check failures caused by network, authentication, certificate, or server errors do not stop a review. They are reported as an unverified remote state.

## Preparation and Commit Rules

### Safe add

Default preparation and CLI modes inspect unversioned files, including content absent from `svn diff`. Tracked project/build references, imports, or the reviewed change must establish that the source, project, resource, or non-local configuration file is required and within scope. Eligible files are added without a separate add question, not merely listed as a recommendation.

Generated outputs, ignored files, local settings, IDE state, logs, archives, caches, screenshots, `.env` files, and likely secret-bearing or ambiguous files stay excluded. User exclusions and explicit file limits take precedence. A directory is inspected recursively before addition; mixed directories use `svn add --depth empty <parent>` for the necessary safe parents, followed by explicit safe files. The skill does not recursively add their generated or sensitive siblings.

Completion requires command success and a subsequent `svn status --xml` result of `added` (`A`), plus review of resulting content and properties. A successful addition appears in the changed-file summary. Failed or unverified additions remain unresolved, with the actual error. Independent logic or test risks do not prevent safe add, and add does not count as fixing those risks or proving a build.

For example, when a tracked project references an eligible new source file, the result should record the verified add in the file summary instead of asking the user to add it again. In explicit read-only mode, that same file stays untracked and is correctly reported as unresolved.

### Safe delete

When the user asks to include reviewed deletions, the skill may schedule explicit clean versioned local paths with clear deletion intent. A reply such as `삭제도 포함해줘` is sufficient for the reviewed candidates.

It never deletes a repository URL, a broad path, a locally modified path, or a conflicted, switched, external, locked, or ambiguous target.

### Final confirmation

Natural commit requests such as `커밋해줘` enter CLI mode without requiring the word `CLI`. Bare invocation and preparation-only requests stop after review; explicit manual TortoiseSVN requests do not trigger a CLI commit. In CLI mode, the skill shows final included paths, exclusions, prepared adds/deletes, risks, and the exact Korean message, then asks one final question.

A clear answer such as `예`, `네`, `응`, `진행`, `진행해`, `커밋해`, or `커밋해줘` to that question authorizes the shown scope and message across turns. Immediately before execution, the skill reruns status and relevant target diffs. Meaningful scope/content changes or new commit blockers require re-review; unchanged approval is not requested again. The initial commit request alone is not final confirmation.

### Review and test evidence

Report only unresolved, meaningful risks in the pre-commit action section. Keep verified preparation results in the file summary and group summaries by intent without a fixed bullet count. On follow-up, report the changed review or execution outcome rather than repeating unchanged sections.

Judge readiness for the intended commit scope after preparation. Missing required build files or evidence-backed serious logic risks warrant recommending a hold. Files verified as excluded, including local secret candidates, do not by themselves block an otherwise valid commit; directory targets must not include them.

Prior tests or builds may be cited only with observed evidence that the relevant reviewed content is unchanged. State that the result came from an earlier run and was not rerun. Otherwise report the verification gap; the skill does not run builds/tests that would modify the working copy.

## Korean Commit Message Encoding

The skill writes the Korean message to a unique temporary file outside the working copy using BOM-free UTF-8 and LF line endings. It commits only with:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

The temporary file is removed in a `finally` cleanup path after success, failure, or cancellation, including errors before execution; its absence is checked. The skill parses the committed revision from the commit output and, when repository access permits, checks the exact revision through:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

If the stored message does not exactly match the original Korean message, the skill reports an encoding-verification failure. It does not retry the commit or modify revision properties.

## Output

The review result is always Korean and includes:

1. Commit readiness: `가능`, `조건부 가능`, or `보류 권장`
2. Changed-file summary by SVN state
3. Meaningful pre-commit risks
4. Change-intent summary
5. Recommended Korean commit message
6. TortoiseSVN final checklist

For a CLI commit attempt, it appends the execution result, parsed revision, message-verification status, temporary-file cleanup result, and final SVN status.
