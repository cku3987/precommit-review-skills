# svn-precommit-review Guide

## Overview

`svn-precommit-review` is a Codex Agent Skill for reviewing SVN/TortoiseSVN working-copy changes immediately before commit. Its review result and commit message are always written in Korean.

The default is a read-only review. When the user explicitly requests commit preparation or a Codex CLI commit, the skill can:

- add clear required source, project, resource, and configuration files
- schedule reviewed clean deletion candidates when the user asks to include them
- show the exact scope, exclusions, risks, and message
- execute a narrow CLI commit after one final affirmative reply

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

Review only:

```text
$svn-precommit-review 현재 SVN 변경사항을 커밋 전에 검토하고 한국어 커밋 메시지를 작성해줘.
```

Review a specific working copy:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로의 변경사항을 검토해줘.
```

Prepare clear required files and request a CLI commit:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고 필요한 신규 소스 파일은 add한 뒤, 최종 확인을 받으면 승인 범위만 CLI로 커밋해줘.
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

When the user requests commit preparation, auto-add, or a CLI commit, the skill may add explicit unversioned files only when they are clearly intended source, project, resource, or required configuration files in the reviewed change.

It excludes generated outputs, local settings, IDE state, logs, archives, caches, screenshots, `.env` files, and likely secret-bearing or ambiguous files. A directory is inspected recursively before it is added.

### Safe delete

When the user asks to include reviewed deletions, the skill may schedule explicit clean versioned local paths with clear deletion intent. A reply such as `삭제도 포함해줘` is sufficient for the reviewed candidates.

It never deletes a repository URL, a broad path, a locally modified path, or a conflicted, switched, external, locked, or ambiguous target.

### Final confirmation

For an explicit CLI commit request, the skill shows the final included paths, exclusions, prepared adds/deletes, risks, and Korean message. It then asks one final question.

A clear answer such as `예`, `네`, `응`, `진행`, `진행해`, `커밋해`, or `커밋해줘` authorizes the commit. Immediately before execution, the skill reruns status and target diffs. It stops only if the approved scope, meaningful content, or a blocking SVN state changed.

## Korean Commit Message Encoding

The skill writes the Korean message to a unique temporary file outside the working copy using BOM-free UTF-8 and LF line endings. It commits only with:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

The temporary file is removed after every attempt. The skill parses the committed revision from the commit output and, when repository access permits, checks the exact revision through:

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
