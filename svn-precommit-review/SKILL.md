---
name: svn-precommit-review
description: SVN/TortoiseSVN 커밋 직전에 사용. svn status와 svn diff를 검토하고, 위험 파일·충돌·누락 파일·민감정보 가능성을 점검한 뒤 한국어 커밋 메시지 초안과 커밋 전 최종 확인 요청을 작성한다. 기본 검토는 읽기 전용이며, 사용자가 커밋 준비/자동 add/삭제 예약/커밋을 명시적으로 요청한 경우 빌드가 깨질 수 있는 필수 신규 파일은 안전 기준 통과 시 svn add할 수 있고, 검토 후 재요청된 명시 경로만 안전 기준 통과 시 svn delete할 수 있다. svn commit은 최종 승인 후 UTF-8 메시지 파일과 --encoding UTF-8 방식으로만 허용한다.
---

# SVN Pre-commit Review

## Purpose

Use this instruction-only skill immediately before a human commits with TortoiseSVN.

Review the current SVN working copy, judge whether the changes are ready to commit, identify risky files or missing actions, draft a Korean commit message, and present a final human confirmation request before commit.

Default review mode is read-only. If the user explicitly asks to prepare the commit, auto-add required files, schedule deletion, or commit after review, perform only the safe `svn add`, `svn delete`, and `svn commit` workflows described below. Do not modify code, project files, SVN metadata, or working-copy state except for safe `svn add` of required build files, safe `svn delete` of approved versioned paths, the approved `svn commit` itself, and a temporary UTF-8 commit-message file created outside the working copy.

## Strict safety rules

- Treat the working copy as read-only unless the user explicitly asks to prepare the commit, auto-add required build files, schedule deletion, or commit.
- Do not edit, create, delete, move, format, normalize, generate, or rewrite project files except through the conditional safe `svn delete` workflow described in this skill.
- Do not run commands that change SVN state or filesystem content except the conditional safe `svn add`, `svn delete`, and `svn commit` workflows described in this skill.
- Do not modify anything under `.svn`.
- Do not automatically resolve conflicts.
- Do not add all unversioned files. Add only narrow, explicit file paths that satisfy the safe-add criteria below.
- Do not automatically revert missing or deleted files.
- If a command requires network access or authentication and fails, continue the review and report the failure clearly.
- If verification is incomplete, say so explicitly. Never claim that tests or remote freshness were verified unless the command output confirms it.
- Do not commit just because the review says `가능` or `조건부 가능`.
- Run `svn add` only after all of these are true:
  - the user explicitly asked this skill to prepare the commit, auto-add required files, or commit
  - the file is unversioned and inside the intended SVN working copy
  - the file is not ignored by SVN and is not under `.svn`
  - the file is not a build output, generated bundle, IDE/user file, log, temp/cache file, local-only config, archive, dump, backup, screenshot, or other commit-excluded candidate
  - the file is not `.env` and does not look like it contains tokens, passwords, API keys, private keys, connection strings, or other secrets
  - the file is likely required to avoid a broken build or runtime because it is referenced by a tracked solution/project/build/config file, belongs to a newly referenced project, or is a source/config/resource file in a tracked build path that the project will compile or load
  - the add target is an explicit file path; do not add a whole directory unless every child has been inspected and passes these criteria
  - if the classification is ambiguous, do not add it; report it as "수동 확인 후 add 필요"
- Run `svn delete` only after all of these are true:
  - the skill has already reviewed `svn status` and relevant diffs
  - the user explicitly re-requested deletion or deletion scheduling for exact paths after seeing the review, or explicitly asked for commit preparation that includes those exact delete targets
  - the path is versioned, inside the intended SVN working copy, and not under `.svn`
  - the path is not conflicted, out-of-date, remotely changed, or otherwise unclear in `svn status -u`
  - the deletion intent is clear and the output has explained the likely build/runtime impact
  - the delete target is an explicit file path; do not delete via `.` or `*`
  - do not delete a directory unless every child has been inspected and the user explicitly approved the whole directory deletion
  - if the file has local modifications, delete it only when the user explicitly approved discarding/scheduling that modified file for deletion
  - if the file is already missing (`!`), do not use `--force` unless the user explicitly approved scheduling that already-missing exact path for deletion
  - if the classification is ambiguous, do not delete it; report it as "삭제 예약 보류, 수동 확인 필요"
- Run `svn commit` only after all of these are true:
  - the user explicitly asks this skill to commit
  - the review has shown included targets, excluded targets, remaining risks, and the exact commit message
  - the user gives final approval for those exact targets and message
  - conflicted files and likely secret leaks are absent
  - `svn status` is rechecked immediately before commit
- Commit only explicit approved target paths. Do not run a broad working-copy commit implicitly.
- For Korean commit messages, use only `svn commit --file <message-file> --encoding UTF-8 <approved-paths>`. Never use `svn commit -m "한글..."`.
- Warn the user not to use PowerShell `svn commit -m "한글..."` for Korean commit messages because console encoding can corrupt the stored `svn:log`. Prefer pasting the message into TortoiseSVN. If a separate, explicitly approved CLI commit workflow is used, require a UTF-8 message file plus `svn commit --file <message-file> --encoding UTF-8`; `--file` alone is not sufficient for this environment.
- In commit mode, create any temporary UTF-8 commit message file outside the SVN working copy when possible, delete it after the commit attempt, and verify it does not remain as an unversioned `?` file.
- Output the final review result in Korean.

## Allowed commands

Allowed SVN commands:

- `svn status`
- `svn status -u`
- `svn diff`
- `svn diff --summarize`
- `svn info`
- `svn log --limit N`
- `svn log --xml --limit N`
- Conditional commit only after explicit user request and final approval: `svn commit --file <message-file> --encoding UTF-8 <approved-paths>`
- Conditional safe add only after explicit user request to prepare/auto-add/commit: `svn add <safe-build-required-paths>`
- Conditional safe delete only after review and explicit user re-request/approval: `svn delete <safe-delete-approved-paths>`
- Conditional safe delete for already-missing approved paths only when explicitly approved: `svn delete --force <approved-missing-paths>`

Read-only local inspection commands may be used only when needed to understand changed or unversioned files, such as viewing file contents or searching for conflict markers and likely secrets. These commands must not write, format, delete, move, or stage anything.

In commit mode only, local file commands may create and delete one temporary UTF-8 commit-message file outside the SVN working copy. Do not create project files for this purpose.

## Forbidden commands

Never run these SVN commands:

- `svn commit -m ...`
- `svn commit --file <message-file>` without `--encoding UTF-8`
- `svn commit` without explicit approved target paths
- broad `svn add`
- `svn add .`
- `svn add *`
- `svn add <directory>` unless every child has been inspected and passes the safe-add criteria
- `svn add` for generated, local-only, sensitive, ambiguous, or commit-excluded files
- `svn update`
- `svn revert`
- `svn cleanup`
- `svn resolve`
- `svn resolved`
- broad `svn delete`
- `svn delete .`
- `svn delete *`
- `svn delete <directory>` unless every child has been inspected and the user explicitly approved the whole directory deletion
- `svn delete --force` except for already-missing exact paths that the user explicitly approved after review
- `svn remove`
- `svn rename`
- `svn move`
- `svn lock`
- `svn unlock`

Also forbidden:

- Any command that edits code or file contents.
- Any command that creates, deletes, renames, moves, or rewrites project files outside the safe-delete workflow.
- Any temporary commit-message file inside the working copy unless no outside location is available and the user explicitly approved that path.
- Any command that changes `.svn` internals.
- Any formatter, code generator, migration, installer, or build step that rewrites tracked files.
- Any automatic conflict resolution.
- Any automatic handling of unversioned files outside the safe-add criteria.

## Review workflow

1. Confirm SVN context with `svn info` when useful.
2. Run `svn status` to identify local changes.
3. Run `svn status -u` when possible to check remote freshness.
   - If it fails because of network, server, certificate, or authentication issues, do not stop.
   - Report it as "원격 최신 여부 확인 실패" and explain the observed error briefly.
4. Run `svn diff --summarize` when possible to get a compact changed-file summary.
   - If it fails, continue with `svn status` and `svn diff` output.
5. Run `svn diff` to inspect actual content changes.
6. Classify files by SVN state:
   - Modified
   - Added / Unversioned
   - Deleted / Missing
   - Conflicted
   - Out-of-date / Remote-changed
   - Other
7. Inspect meaningful changed content for risks:
   - build artifacts or generated output
   - IDE or user-specific files
   - logs, temporary files, cache files
   - local configuration files
   - `.env`, tokens, passwords, API keys, private URLs, credentials, or other secrets
   - conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`
   - unversioned files that may need manual `svn add`
   - deleted or missing files that may have been removed accidentally
   - insufficient testing or verification
8. Evaluate unversioned files for safe-add handling:
   - If the user did not ask for commit preparation, auto-add, or commit, do not run `svn add`; only report candidates.
   - If the user did ask for commit preparation, auto-add, or commit, add only files that pass every safe-add criterion.
   - Prefer explicit file paths. Avoid directory adds unless the entire directory tree has been inspected and every child is safe.
   - After any `svn add`, run `svn status` again and report the added paths.
   - If any candidate is ambiguous, generated, local-only, sensitive-looking, or not clearly build-required, do not add it; report the reason.
9. Evaluate versioned deletion candidates for safe-delete handling:
   - If the user did not explicitly re-request deletion or commit preparation with exact delete targets, do not run `svn delete`; only report candidates.
   - If the user did re-request deletion, delete only exact paths that pass every safe-delete criterion.
   - Prefer explicit file paths. Avoid directory deletion unless the whole tree has been inspected and explicitly approved.
   - After any `svn delete`, run `svn status` again and report the scheduled-delete paths.
   - If any candidate is ambiguous, conflicted, remotely changed, locally modified without explicit approval, or likely to break build/runtime unexpectedly, do not delete it; report the reason.
10. Infer the change intent from file paths, diff content, and nearby context. Do not invent issue numbers or requirements.
11. Draft a Korean commit message that reflects only verified changes.
12. Finish with a short TortoiseSVN checklist and a final confirmation request for the human committer.
13. The final confirmation request must show:
   - files or groups that should be included
   - files already added with safe `svn add`
   - files already scheduled for deletion with safe `svn delete`
   - files or groups that should be excluded
   - unresolved risks or required manual actions
   - the recommended commit message
   - a clear Korean confirmation question asking whether the human approves committing with those exact targets and message
   - cleanup reminder for any temporary UTF-8 commit message file, especially if it was created inside the working copy
14. If the user does not explicitly approve committing, stop after the review and confirmation request.
15. If the user explicitly approves committing:
   - run `svn status` again and compare it with the approved target list
   - refuse to commit if new unapproved changes, conflicts, likely secrets, or unapproved/unclear missing/deleted files appear
   - create a temporary UTF-8 commit-message file outside the working copy when possible
   - run `svn commit --file <message-file> --encoding UTF-8 <approved-paths>` with only approved paths
   - delete the temporary message file whether the commit succeeds or fails
   - run `svn log --xml --limit 1` after a successful commit and verify the Korean `svn:log` message is not broken
   - report the committed revision, committed paths, message verification result, and cleanup result in Korean

## Risk checklist

Check and report only meaningful risks:

- Commit-excluded candidates:
  - build outputs such as `bin/`, `obj/`, `dist/`, `build/`, `target/`, generated bundles, compiled binaries
  - IDE or user files such as `.idea/`, `.vscode/`, `*.suo`, `*.user`, `*.iml`
  - logs, temp files, caches, screenshots, local exports
  - local-only config files
- Sensitive information:
  - `.env` files
  - tokens, passwords, API keys, access keys, private keys
  - connection strings or credentials
  - secrets newly added or changed in diffs
- SVN state risks:
  - conflicted files
  - missing files marked with `!`
  - deleted files marked with `D`
  - unversioned files marked with `?` that look required by the change but did not pass safe-add criteria
  - files added automatically with safe `svn add` that still need human diff review before commit
  - files scheduled for deletion with safe `svn delete` that still need human confirmation before commit
  - deleted or missing files that were not explicitly approved for deletion
  - remote-changed or out-of-date files from `svn status -u`
- Content risks:
  - conflict markers `<<<<<<<`, `=======`, `>>>>>>>`
  - suspicious large generated diffs
  - broad formatting-only changes mixed with logic changes
  - risky logic changes without tests
- Verification risks:
  - tests not run
  - only partial verification
  - remote freshness not checked
- Commit-message encoding risks:
  - Korean commit messages passed through PowerShell `svn commit -m "..."` may be stored broken in `svn:log`
  - in this environment, `svn commit --file <message-file>` without `--encoding UTF-8` can still store broken Korean in `svn:log`
  - already-committed broken `svn:log` messages require server-side revision property changes and may be blocked without a `pre-revprop-change` hook
  - temporary commit message files can become accidental unversioned files if created inside the working copy

## Output format

Always write the final result in Korean using this exact structure:

````markdown
# SVN 커밋 전 검토 결과

## 1. 커밋 가능 여부

판단: 가능 / 조건부 가능 / 보류 권장

이유:
- ...

## 2. 변경 파일 요약

### Modified
- `path/to/file`: 무엇이 바뀐 것으로 보이는지 간단히 설명

### Added / Unversioned
- `path/to/file`: 새 파일 또는 unversioned 파일의 의미, 자동 `svn add` 수행 여부, 수동 확인 필요 여부

### Deleted / Missing
- `path/to/file`: 삭제 또는 missing 상태가 의도된 것으로 보이는지, 자동 `svn delete` 수행 여부, 수동 확인 필요 여부 설명

### Conflicted
- `path/to/file`: 충돌 상태 또는 충돌 마커 여부

### Out-of-date / Remote-changed
- `path/to/file`: `svn status -u` 기준 원격 변경 또는 최신 아님 여부

### Other
- `path/to/file`: 위 분류에 들어가지 않는 상태 설명

## 3. 커밋 전 확인 필요 항목

- 커밋 제외 권장 파일: ...
- 자동 `svn add` 완료 파일: ...
- 승인 후 `svn delete` 완료 파일: ...
- 수동 `svn add` 필요 가능 파일: ...
- 수동 삭제 확인 필요 파일: ...
- revert 검토 필요 파일: ...
- 충돌 해결 필요 파일: ...
- 테스트 미실행/검증 부족: ...
- 민감정보 가능성: ...
- 로직상 위험한 변경: ...

문제가 없으면:
특별한 위험 요소는 보이지 않습니다.

## 4. 변경 내용 요약

실제 변경 의도를 변경 규모에 맞게 요약합니다.
단순 변경이면 1~2개 bullet도 허용하고, 일반적인 변경은 3~7개 bullet로 작성합니다.
줄 수를 채우기 위해 의미 없는 항목을 만들지 말고, 큰 변경은 관련 항목을 묶어서 7개를 넘기지 않습니다.
세부 파일명이나 줄 단위 변경을 모두 풀어쓰지 말고, 커밋 판단에 필요한 의미 단위로 묶어 요약합니다.
파일별 상세 설명은 `## 2. 변경 파일 요약`에 두고, 이 섹션은 변경 의도와 영향 중심으로 씁니다.

- ...
- ...
- ...

## 5. 추천 커밋 메시지

```text
[카테고리] 한 줄 제목

변경사항:
- ...
- ...
- ...

검토:
- svn status 확인
- svn diff 확인
- 테스트: 실행함 / 미실행 / 확인 필요

주의:
- ...
```

## 6. TortoiseSVN에서 할 일

- Check for Modifications 열기
- 파일별 Diff 확인
- 새 파일 중 필요한 것만 Add
- 자동 `svn add`된 파일도 Diff로 재확인
- `svn delete` 예약된 파일이 의도된 삭제인지 재확인
- 로컬/생성 파일 제외
- 충돌 또는 missing 파일 확인
- 추천 커밋 메시지를 TortoiseSVN Commit 창에 붙여넣기
- PowerShell `svn commit -m "한글..."` 사용 금지
- CLI 커밋은 별도 승인된 경우에만 `svn commit --file <메시지파일> --encoding UTF-8` 형식으로 수행
- CLI 커밋용 UTF-8 메시지 파일을 만들었다면 사용 후 삭제하고 `svn status`에 `?` 파일로 남지 않았는지 확인
- 커밋 후 `svn log --xml --limit 1`로 저장소에 저장된 한글 메시지가 깨지지 않았는지 확인
- 최종 확인 후 Commit

최종 확인 요청:
- 포함할 항목: ...
- 자동 `svn add` 완료 항목: ...
- `svn delete` 예약 완료 항목: ...
- 제외할 항목: ...
- 남은 확인 사항: ...
- 사용 후 정리할 임시 파일: ...
- 사용할 커밋 메시지: 위 추천 커밋 메시지
- 이 포함/제외 대상과 커밋 메시지로 TortoiseSVN에서 커밋해도 되는지 최종 확인해 주세요.
````

If this skill performs the commit after explicit user approval, append this Korean section after the review result:

````markdown
## 7. 커밋 실행 결과

- 실행 명령: `svn commit --file <메시지파일> --encoding UTF-8 <승인된 경로>`
- 사전 `svn add` 결과: 자동 add 완료 / 자동 add 없음 / 일부 add 보류
- 사전 `svn delete` 결과: 삭제 예약 완료 / 삭제 예약 없음 / 일부 삭제 보류
- 커밋 리비전: r...
- 커밋 대상: ...
- 메시지 인코딩 확인: `svn log --xml --limit 1` 기준 정상 / 깨짐 의심 / 확인 실패
- 임시 메시지 파일 정리: 삭제 완료 / 삭제 실패 / 해당 없음
- 남은 확인 사항: ...
````

Judgment guidance:

- Use `가능` only when no meaningful risk is found, required files are tracked, conflicts are absent, and verification is sufficient or clearly acceptable for the change.
- Use `조건부 가능` when the commit appears reasonable but the human must first confirm specific items such as unversioned files, skipped tests, remote freshness failure, or intentional deletion.
- Use `보류 권장` when there are conflicts, likely secrets, accidental generated/local files, suspicious missing files, out-of-date risk, or insufficient understanding of risky changes.

Commit message guidance:

- Choose a concise Korean category such as `[수정]`, `[개선]`, `[추가]`, `[정리]`, `[문서]`, `[설정]`, or `[테스트]`.
- Include an issue number only when it is clearly visible in user input, branch name, file name, or diff content.
- Do not invent issue numbers, ticket IDs, test results, or motivations.
- Keep the message specific to the reviewed diff.
- In the `변경사항:` block, use as many bullets as the diff genuinely needs. Do not pad to a default count. For small changes, 1-2 bullets are acceptable; for broad changes, group related edits and keep the list concise.
- Do not list every file or line-level edit in the commit message unless each item has distinct meaning. Group related edits by intent, component, or behavior.

## TortoiseSVN final checklist

In section 6, remind the human to:

- Open TortoiseSVN Check for Modifications.
- Review each file diff manually.
- Add only required new files.
- Recheck files added automatically by this skill before committing.
- Recheck files scheduled for deletion by this skill before committing.
- Exclude local, generated, IDE, log, temp, and secret files.
- Confirm conflicted, deleted, and missing files.
- Confirm remote freshness if `svn status -u` failed or showed remote changes.
- Run or confirm relevant tests when needed.
- Paste or adapt the recommended Korean commit message.
- Do not use PowerShell `svn commit -m "Korean message"` for Korean commit messages.
- Prefer pasting the Korean message into the TortoiseSVN Commit window. If a CLI commit is handled by a separate explicitly approved workflow, use a UTF-8 message file with `svn commit --file <message-file> --encoding UTF-8`. Do not suggest `--file` without `--encoding UTF-8` in this environment.
- If a UTF-8 commit message file was created for a separate CLI workflow, remove it after use and confirm it does not appear as an unversioned file in `svn status`. Prefer creating that file outside the SVN working copy.
- After commit, verify the stored message with `svn log --xml --limit 1`; ordinary `svn log` output can look correct even when the repository `svn:log` value is broken.
- Ask for final human confirmation before commit, explicitly listing included targets, excluded targets, remaining risks, and the exact commit message.
- If the user asked this skill to prepare the commit or auto-add required files, verify any safe `svn add` result in TortoiseSVN before committing.
- If the user re-requested deletion or asked this skill to prepare exact delete targets, verify any safe `svn delete` result in TortoiseSVN before committing.
- If the user wants this skill to commit, commit only after all listed risks are resolved or intentionally accepted and the human gives final approval for the exact targets and message.
- If the user does not explicitly request this skill to commit, stop at the TortoiseSVN checklist and final confirmation request.
