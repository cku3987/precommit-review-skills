---
name: git-precommit-review
description: Git 커밋 전 변경사항과 위험을 검토하고 한국어 커밋 메시지를 작성한다. 기본 실행은 필수 신규 파일의 안전한 staging을 포함한다. 기존 부분 staging은 보존하고, 커밋 요청 시 범위와 메시지를 최종 확인받아 승인한 변경만 커밋한다. 읽기 전용 요청은 변경하지 않으며 push는 별도 요청 시에만 수행한다.
---

# Git Pre-commit Review

## Purpose

Use this instruction-only skill for Git repositories, not SVN working copies. Review the requested repository and file scope; use the current directory only when no path is given. Report in Korean and draft concise Korean commit messages. Do not edit product source contents to fix findings.

## Execution modes

- **Read-only:** explicit `검토만`, `읽기 전용`, `메시지만`, or no-change requests prohibit staging, temporary-file creation, commits, and pushes. Use read-only Git commands and local inspection only.
- **Prepare (default):** bare invocation or `검토해줘` reviews staged, unstaged, and untracked changes, then stages clear required new files. It does not stage every modified tracked file, commit, or push.
- **Commit:** `커밋해줘` requests preparation of the intended commit, including reviewed tracked changes as described below. Show exact staged content to include, exclusions, risks, and message; ask once `이 범위와 메시지로 Git 커밋할까요?` A clear `네`, `응`, `진행해`, or equivalent reply to that question approves that snapshot. Reuse approval across turns unless meaningful content, scope, HEAD, or blocking state changes. The initial request alone is not final confirmation.
- A manual Git GUI request stops after preparation and the checklist; do not require a GUI for a CLI request. `push` requires a separate explicit request, which can be included in the initial request. A commit confirmation alone never authorizes push.
- Preserve later user exclusions and explicit file limits. If necessary dependencies fall outside that scope, report the missing scope decision instead of expanding it silently.

## Review workflow

1. Resolve repository/worktree root, branch, HEAD (or unborn branch), and Git paths with `git rev-parse`. Do not assume `.git` is a directory. Detect merge/rebase/cherry-pick/revert operations and unmerged entries. This skill creates ordinary commits; leave operation continuation and conflict resolution to a separate explicit task. Detached HEAD requires a deliberate destination decision before commit, not automatic branch creation.
2. Inspect `git --no-optional-locks status --porcelain=v2 -z --branch --untracked-files=all`, `git ls-files --stage -z`, and both `git diff` and `git diff --cached`. Use NUL-safe parsing and literal path arguments; never split machine paths on whitespace or parse quoted display paths. Disable external diff and text conversion for inspection (`--no-ext-diff --no-textconv`). Include name/status, content, mode, binary, rename and deletion changes.
3. Read untracked candidate contents directly; they are absent from ordinary diffs. Inspect ignores with `git check-ignore` and tracked build/project/import references. Review staged blobs as well as working files: a secret removed from the working file may still be staged.
4. Report upstream/ahead/behind information as cached unless refreshed. Optional `git ls-remote` can check a specified remote ref without fetching, but a ref OID alone does not establish ancestry. No upstream or a network/auth failure does not block local review or an otherwise valid local commit. Do not run fetch/pull automatically or claim remote freshness from stale tracking refs.
5. Perform eligible preparation, then verify real staged diffs. Keep independent safe staging separate from logic/test findings. Successful staging is a completed result, not an unresolved instruction to stage again.
6. Judge the intended commit snapshot, not the entire dirty worktree. Excluded local/generated files do not by themselves block it. Missing required files, staged secrets, unresolved conflicts, or evidence-backed serious correctness risks warrant `보류 권장`. Verification gaps warrant `조건부 가능`; adequate scope and evidence without meaningful risks permit `가능`.
7. Reuse observed tests only if they cover the unchanged reviewed content; label earlier runs and gaps. Do not claim a working-tree test validates an older staged snapshot. Do not run builds/tests that modify product files under this skill. Summarize by intent without a fixed bullet count, normally 1-2 bullets for small changes and no more than needed for broader work.

## Staging rules

- Automatically stage explicit required new source/project/resource/non-local configuration files after confirming purpose, content, ignore state, and scope. Do not merely recommend add when the checks pass. Inspect directories but stage individual files, not whole directories.
- Never auto-stage ignored files, `.env`, secrets, generated output, IDE/user files, local settings, logs, caches, archives, dumps, or ambiguous files. Do not traverse submodules or nested repositories. Review submodule pointer changes separately; dirty submodule contents are not included by staging the parent.
- Agent-created verification-only scripts, folders or test projects are not required product dependencies merely because their tests ran or passed. Exclude them from staging and commits unless the user asks to retain them, even if already staged; report existing staging without silently removing it. Do not make their inclusion a commit prerequisite. This does not exclude requested regression tests in an existing versioned suite.
- In commit mode, stage reviewed whole-file tracked modifications within the requested scope only when that file has no pre-existing staged changes. If it has both staged and unstaged changes, preserve the staged version and explicitly exclude the unstaged part. Including the rest requires clear user intent, not an automatic full-file `git add`.
- Existing staged paths are candidates for review, not blanket permission to commit them. A path-limited request must exclude unrelated staged changes without unstaging them. For finer selection within an already staged file, ask for the desired hunk scope; do not invent it or replace the user's partial staging.
- Stage already-missing tracked files with `git add -u -- <explicit-missing-paths>` only when the user requests those deletions. Do not physically delete files. Review both sides of a rename as a pair; do not include only its source or destination accidentally.
- Use literal pathspecs (`git --literal-pathspecs add -- <paths>`), including for missing paths. Check the command result, porcelain state, and cached diff before reporting success. Do not use `git add .`, `-A`, broad `-u`, `--force`, or an implicit directory scope.

## Scoped commit procedure

Git commits an index snapshot, not a list of filenames. Never use `git commit -a` or `git commit <paths>`/`--only` to approximate a staged selection: those can include unstaged content. Do not unstage unrelated changes to make a normal commit convenient.

After final approval, use the real index directly only when its entire snapshot is approved. If any staged paths are excluded, use the isolated-index procedure below. Preserve a record of HEAD, branch, staged entries/tree, intended staged diff, and relevant working-tree state from the final review.

1. Recheck those records and operation state immediately before execution. If they changed materially, show the changed scope and renew confirmation. Do not proceed with unmerged entries, an unrelated pre-existing `GIT_INDEX_FILE`, or an index state that cannot be safely understood (for example unresolved intent-to-add entries). Do not change repository settings to bypass these conditions.
2. Record the full real-index tree with `git write-tree` and the old HEAD OID, if any. For exclusions, allocate a unique, initially nonexistent temporary index path outside the repository; apply `GIT_INDEX_FILE` only to the child commands that need it, never globally. Run `git read-tree <recorded-real-index-tree>` into this temporary index. Do not copy a raw index or assume split/sparse index storage layout.
3. In the temporary index ONLY, restore excluded staged paths to old HEAD using `git --literal-pathspecs restore --source=<old-head> --staged -- <excluded-paths>`. For an unborn branch, remove excluded entries with `git update-index --force-remove -- <exact-excluded-paths>` in that temporary index. These operations must never target the real index or working files. Review the resulting cached diff and `git write-tree`; it must match exactly the approved staged change, including deletions, modes and rename pairs. Do not commit an empty snapshot.
4. Write the approved message to a unique UTF-8 without BOM, LF-terminated temporary file outside the repository, using an explicitly encoded API (not PowerShell default encoding or `Out-File`). Execute `git -c i18n.commitEncoding=UTF-8 commit --cleanup=verbatim --file <message-file>` with the selected index and NO path arguments. Do not bypass hooks or signing. Do not run SVN's `--encoding` option with Git.
5. Identify the new exact commit and verify its parent (none for the first commit), tree, and raw UTF-8 message against the approved snapshot. Use `git cat-file commit <oid>` and compare its message after the header separator, with the intended single terminal LF. Check final status and that unrelated staged entries and remaining working changes are preserved. With the isolated index, the real index must retain its original entries: included entries already match new HEAD, excluded entries remain staged relative to new HEAD. No real-index reset/restore is needed.
6. If hooks change contents/message, a concurrent change occurs, the command fails, or the outcome is unclear, report observed results and differences; inspect HEAD before any retry. Never blindly recommit, amend, roll back, or claim the expected commit was produced. A tree/message mismatch after a successful command is a verification failure, not permission to rewrite history.
7. In `finally`, after child processes exit, delete only the owned message file, temporary index, and any lock belonging to that temporary index. Confirm absence and restore any scoped environment state. Never delete the real index lock or other pre-existing artifacts. Report cleanup failures.

## Push boundary

Only on an explicit push request, establish remote, destination branch, and all commits that would be published. Reuse an unambiguous configured upstream; ask if the destination or outgoing scope is ambiguous. Disclose older unpushed commits rather than implying only the new commit will be published. Push the approved branch explicitly with `git push <remote> HEAD:refs/heads/<branch>`, without tags or force. On rejection, report it; do not pull, rebase, merge, or force-push. Verify the remote ref when possible. Local commit success and push success are separate outcomes.

## Command boundaries

- Read-only inspection includes `git status`, `diff`, `ls-files`, `check-ignore`, `rev-parse`, `symbolic-ref`, `log`, `show`, `cat-file`, `ls-tree`, `rev-list`, `remote -v`, read-only `config --get`, and `ls-remote`; use bounded scope and mask secret-bearing URLs/values.
- Allowed writes are scoped staging above, approved ordinary commits, explicitly requested push, temporary files, and the isolated-index plumbing above. `write-tree` is used only in commit mode. Use `GIT_OPTIONAL_LOCKS=0` or `--no-optional-locks` for read-only status to avoid incidental index refresh.
- Do not edit product sources, `.git` internals or configuration directly; do not reset, clean, stash, checkout/switch, rebase, merge, amend, force-push, create branches/tags, or resolve conflicts. `restore --staged` and `update-index --force-remove` are exceptions ONLY for the owned temporary index as specified, never the real index.

## Output format

Use the following six sections for the initial Korean report. Group meaningful changes; use `없음` for empty categories and do not repeat resolved warnings. On follow-up show only changed sections or execution results.

````markdown
# Git 커밋 전 검토 결과

## 1. 커밋 가능 여부
판단: 가능 / 조건부 가능 / 보류 권장
- 이유와 검증 범위

## 2. 변경 파일 요약
- Staged: 경로와 포함할 변경
- Unstaged: 경로와 남겨둘 변경
- Added / Untracked: staging 완료 또는 미추적 유지 이유
- Deleted / Renamed: 대상과 삭제 승인 상태
- Conflicted / Other: 충돌, submodule, mode, binary 등
- Remote: 원격 확인 결과 또는 캐시 기준/미확인

## 3. 커밋 전 확인 필요 항목
- 미해결 위험, 근거, 필요한 조치
문제가 없으면: 특별한 위험 요소는 보이지 않습니다.

## 4. 변경 내용 요약
- 실제 변경 의도와 영향

## 5. 추천 커밋 메시지
```text
[카테고리] 한 줄 제목

변경사항:
- ...

검토:
- git status 및 staged/unstaged diff 확인
- 테스트: 실제 실행 근거 / 이전 결과 (이번 미실행) / 미실행 / 확인 필요

주의:
- 없음 / ...
```

## 6. 최종 확인
- 포함: 승인 대상 staged 변경
- 제외: 기존 staging 및 unstaged 중 남겨둘 변경
- 준비 완료: 자동 staging / 요청한 삭제 staging / 없음
- 실행 방식: 읽기 전용 / 준비만 / CLI 커밋 / 수동 GUI
- push: 요청 없음 / 요청한 원격·브랜치와 게시 범위
````

Do not invent issue IDs or test results. A commit request ends the preview with the single final confirmation question, not instructions to commit manually. After a commit attempt append `## 7. 커밋 실행 결과` with observed success/failure/uncertainty, exact OID, scope/tree/message verification, preserved staging, temporary-file cleanup, final status, and push result (or not requested).
