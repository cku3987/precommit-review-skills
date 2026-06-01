---
name: svn-precommit-review
description: SVN/TortoiseSVN 커밋 직전에 사용. svn status와 svn diff를 검토하고, 위험 파일·충돌·누락 파일·민감정보 가능성을 점검한 뒤 한국어 커밋 메시지 초안과 커밋 전 최종 확인 요청을 작성한다. 기본은 읽기 전용이며, 사용자가 명시적으로 커밋을 요청하고 최종 승인한 경우에만 UTF-8 메시지 파일과 --encoding UTF-8 방식의 svn commit을 허용한다.
---

# SVN Pre-commit Review

## Purpose

Use this instruction-only skill immediately before a human commits with TortoiseSVN.

Review the current SVN working copy, judge whether the changes are ready to commit, identify risky files or missing actions, draft a Korean commit message, and present a final human confirmation request before commit.

Default mode is read-only. If the user explicitly asks to commit after the review and gives final approval, perform only the safe commit workflow described below. Do not modify code, project files, SVN metadata, or working-copy state except for the approved `svn commit` itself and a temporary UTF-8 commit-message file created outside the working copy.

## Strict safety rules

- Treat the working copy as read-only until the user explicitly asks for a commit and approves the exact targets and message.
- Do not edit, create, delete, move, format, normalize, generate, or rewrite project files.
- Do not run commands that change SVN state or filesystem content except the conditional `svn commit` workflow described in this skill.
- Do not modify anything under `.svn`.
- Do not automatically resolve conflicts.
- Do not automatically add unversioned files.
- Do not automatically revert missing or deleted files.
- If a command requires network access or authentication and fails, continue the review and report the failure clearly.
- If verification is incomplete, say so explicitly. Never claim that tests or remote freshness were verified unless the command output confirms it.
- Do not commit just because the review says `가능` or `조건부 가능`.
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

Read-only local inspection commands may be used only when needed to understand changed or unversioned files, such as viewing file contents or searching for conflict markers and likely secrets. These commands must not write, format, delete, move, or stage anything.

In commit mode only, local file commands may create and delete one temporary UTF-8 commit-message file outside the SVN working copy. Do not create project files for this purpose.

## Forbidden commands

Never run these SVN commands:

- `svn commit -m ...`
- `svn commit --file <message-file>` without `--encoding UTF-8`
- `svn commit` without explicit approved target paths
- `svn add`
- `svn update`
- `svn revert`
- `svn cleanup`
- `svn resolve`
- `svn resolved`
- `svn delete`
- `svn remove`
- `svn rename`
- `svn move`
- `svn lock`
- `svn unlock`

Also forbidden:

- Any command that edits code or file contents.
- Any command that creates, deletes, renames, moves, or rewrites project files.
- Any temporary commit-message file inside the working copy unless no outside location is available and the user explicitly approved that path.
- Any command that changes `.svn` internals.
- Any formatter, code generator, migration, installer, or build step that rewrites tracked files.
- Any automatic conflict resolution.
- Any automatic handling of unversioned files.

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
8. Infer the change intent from file paths, diff content, and nearby context. Do not invent issue numbers or requirements.
9. Draft a Korean commit message that reflects only verified changes.
10. Finish with a short TortoiseSVN checklist and a final confirmation request for the human committer.
11. The final confirmation request must show:
   - files or groups that should be included
   - files or groups that should be excluded
   - unresolved risks or required manual actions
   - the recommended commit message
   - a clear Korean confirmation question asking whether the human approves committing with those exact targets and message
   - cleanup reminder for any temporary UTF-8 commit message file, especially if it was created inside the working copy
12. If the user does not explicitly approve committing, stop after the review and confirmation request.
13. If the user explicitly approves committing:
   - run `svn status` again and compare it with the approved target list
   - refuse to commit if new unapproved changes, conflicts, likely secrets, or unclear missing/deleted files appear
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
  - unversioned files marked with `?` that look required by the change
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
- `path/to/file`: 새 파일 또는 unversioned 파일의 의미와 수동 `svn add` 필요 여부

### Deleted / Missing
- `path/to/file`: 삭제 또는 missing 상태가 의도된 것으로 보이는지 설명

### Conflicted
- `path/to/file`: 충돌 상태 또는 충돌 마커 여부

### Out-of-date / Remote-changed
- `path/to/file`: `svn status -u` 기준 원격 변경 또는 최신 아님 여부

### Other
- `path/to/file`: 위 분류에 들어가지 않는 상태 설명

## 3. 커밋 전 확인 필요 항목

- 커밋 제외 권장 파일: ...
- 수동 `svn add` 필요 가능 파일: ...
- revert 검토 필요 파일: ...
- 충돌 해결 필요 파일: ...
- 테스트 미실행/검증 부족: ...
- 민감정보 가능성: ...
- 로직상 위험한 변경: ...

문제가 없으면:
특별한 위험 요소는 보이지 않습니다.

## 4. 변경 내용 요약

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

## TortoiseSVN final checklist

In section 6, remind the human to:

- Open TortoiseSVN Check for Modifications.
- Review each file diff manually.
- Add only required new files.
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
- If the user wants this skill to commit, commit only after all listed risks are resolved or intentionally accepted and the human gives final approval for the exact targets and message.
- If the user does not explicitly request this skill to commit, stop at the TortoiseSVN checklist and final confirmation request.
