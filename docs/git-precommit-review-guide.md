# git-precommit-review Guide

The Git skill is independent of `svn-precommit-review`. Both live in the [precommit-review-skills repository](https://github.com/cku3987/precommit-review-skills), with shared releases and separate installation folders. Reviews and commit messages are Korean.

## Install and invoke

Install only `git-precommit-review/SKILL.md`, or install both skill folders. For project-local use, place each in `.agents/skills/<skill-name>/SKILL.md`. Claude Code users can place the same instruction-only folders in `.claude/skills/` and invoke `/git-precommit-review` or `/svn-precommit-review`. No helper scripts or Python YAML runtime are needed by the skills.

```text
$git-precommit-review <GIT_REPOSITORY> 변경사항을 검토해줘.
$git-precommit-review <GIT_REPOSITORY> 읽기 전용으로 검토만 해줘.
$git-precommit-review <GIT_REPOSITORY> 변경사항을 커밋해줘.
```

A general review includes safe staging of clearly required new files, not all modified files. Explicit read-only, review-only or message-only requests perform no writes. A commit request prepares reviewed tracked changes, shows the intended staged content and Korean message, and asks for one final confirmation. `네, 진행해` approves that displayed scope; unchanged approval carries across turns.

## Git-specific behavior

- Staged and unstaged versions are reviewed separately. Existing partial staging is preserved; a full-file add must not silently replace it.
- Ignored, generated, local-only, sensitive and ambiguous new files are excluded. New files needed by tracked build/import references are staged only within the requested scope.
- Agent-created verification-only scripts or test projects stay excluded unless explicitly requested for retention, even if already staged. Existing staging is not silently removed. Requested regression tests in an existing versioned suite remain eligible.
- Existing unrelated staged changes are not automatically approved. When committing only selected staged paths, a temporary index preserves the real staging area and prevents unrelated changes from entering the commit.
- Already-missing tracked files are staged as deletions only on request. The skill does not physically delete files or resolve conflicts.
- Merge/rebase/cherry-pick continuation is a separate task. Submodule working changes are not silently committed in the parent repository.
- Upstream counts may be cached. Remote access failures do not prevent local review or otherwise valid local commits.
- Tests of the working tree do not necessarily validate a partially staged commit. Results identify the actual verified snapshot and any gaps.

## Commit integrity

The final approved staged snapshot is committed with a UTF-8 without BOM message file and `git -c i18n.commitEncoding=UTF-8 commit --cleanup=verbatim --file <message-file>`. No filename arguments, `-a`, or `--only` are used to approximate the staging selection. Hooks and signing remain enabled.

When exclusions require an isolated index, Git constructs it from the recorded staged tree rather than copying an index file. Only that temporary index is adjusted. Afterward the exact commit parent, tree and raw message are verified, and the real staging area is left intact. Hook changes and ambiguous failures are reported, not repaired with amend/reset/retry. Owned temporary message/index files are cleaned up after child processes exit, including failure paths.

Git and SVN do not use the same encoding options: Git uses `i18n.commitEncoding=UTF-8`; SVN uses `--encoding UTF-8`.

## Push is separate

`커밋해줘` does not authorize publishing. Request push explicitly, for example:

```text
$git-precommit-review 변경사항을 커밋하고 origin의 현재 브랜치로 push해줘.
```

The skill identifies the destination and all outgoing commits. It does not hide earlier unpushed commits, push tags, force-push, or automatically pull/rebase after rejection. The report distinguishes local commit success from push success.

## Output

The initial report has six Korean sections: readiness, file summary, unresolved risks, change summary, recommended message, and final scope confirmation. After execution it adds commit OID, verification results, staging preservation, temporary-file cleanup and any requested push result. Follow-up responses omit unchanged sections.

## Git references

- [git commit: index and path behavior](https://git-scm.com/docs/git-commit)
- [git read-tree: constructing an index](https://git-scm.com/docs/git-read-tree)
- [git restore: staged-only restoration](https://git-scm.com/docs/git-restore)
