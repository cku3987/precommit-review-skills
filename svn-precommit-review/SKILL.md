---
name: svn-precommit-review
description: SVN/TortoiseSVN 커밋 직전에 사용. svn status와 svn diff를 구조적으로 검토하고 위험 파일, 충돌, 누락 파일, property 변경, 민감정보 가능성을 점검한 뒤 한국어 커밋 메시지를 작성한다. 기본은 읽기 전용이다. 사용자가 CLI 커밋을 요청하면 필요한 신규 소스 파일을 안전하게 add하고, 검토 결과 뒤 한 번의 최종 확인으로 명시 범위만 커밋한다.
---

# SVN Pre-commit Review

## Purpose

Use this instruction-only skill immediately before an SVN/TortoiseSVN commit.

Review the user-specified working copy, explain meaningful risks, draft a concise Korean commit message, and help prepare or execute a narrowly scoped CLI commit when the user asks. Use the current directory only when no path is supplied. Never silently switch to another working copy.

## Strict safety rules

- Always write review and commit results in Korean.
- Default to read-only review. Do not run a write command unless the user asks for commit preparation, add/delete preparation, or a Codex CLI commit.
- Keep project files and `.svn` contents unchanged. The only allowed mutations are the explicitly scoped `svn add`, `svn delete`, and `svn commit` commands below.
- Never expose a secret value. Report only the path, optional line number, and risk type; replace values with `[REDACTED]`.
- Continue when `svn status -u` fails due to network, certificate, authentication, or server errors. Report that remote freshness was not verified.
- Never infer a successful check from missing, truncated, or failed output.

### Commit flow

1. For review-only or TortoiseSVN requests, stop after the Korean review and final checklist. Do not ask a CLI-commit question.
2. For a request such as `CLI로 커밋해줘`, inspect the change, automatically prepare clear required files when allowed, and show the exact included scope, exclusions, remaining risks, and message.
3. Ask one short final question: `이 범위와 메시지로 CLI 커밋할까요?`
4. After that question, a clear reply such as `예`, `네`, `응`, `진행`, `진행해`, `커밋해`, or `커밋해줘` authorizes the CLI commit.
5. Immediately before committing, rerun status and relevant diffs. Stop and re-review only when the approved scope, meaningful content, or blocking SVN state changed.

### Add and delete preparation

- When the user asks for commit, commit preparation, or auto-add, automatically run `svn add` for an explicit unversioned source, project, resource, or required configuration file when its purpose is clear from the reviewed change or tracked references.
- Do not add build output, generated bundles, IDE/user files, logs, caches, local settings, archives, dumps, backups, screenshots, `.env`, or likely secret-bearing files.
- Inspect a directory recursively before adding it. Add it only when every child is clearly intended and safe; otherwise add only the safe explicit children and list the rest as excluded.
- When the user asks to include reviewed deletions, allow `svn delete` or `svn delete --force` for explicitly listed clean versioned local candidates with clear deletion intent. A reply such as `삭제도 포함해줘` may approve the reviewed deletion candidates; do not require the user to repeat every path.
- Never schedule deletion for a locally modified, conflicted, switched, external, locked, or ambiguous path. Never delete a repository URL, use `--keep-local`, or use a broad path.

### Korean commit message and cleanup

- For Korean CLI messages, use a unique temporary file outside the working copy, written as UTF-8 without BOM and normalized to LF. Do not use PowerShell `-m`, `Out-File`, or default `Set-Content`.
- Commit only with `svn commit --file <message-file> --encoding UTF-8 <explicit-approved-paths>`.
- Delete the temporary message file after every commit attempt and confirm its absence.
- Parse the committed revision from the commit command output. When repository access permits, verify that exact revision with `svn log --xml -r N <repository-root-url>` and compare the stored message character-for-character.
- If message verification fails or shows broken Korean, report it. Do not retry the commit or attempt a revision-property change.

## Allowed commands

Read-only SVN commands:

- `svn status`
- `svn status --xml`
- `svn status --xml --no-ignore`
- `svn status -u`
- `svn status -u --xml`
- `svn diff`
- `svn diff --summarize`
- `svn diff --properties-only`
- `svn info`
- `svn info --xml`
- `svn info --show-item repos-root-url <working-copy-root>`
- `svn log --limit N`
- `svn log --xml --limit N`
- `svn log --xml -r N <repository-root-url>`

Conditionally allowed mutations:

- `svn add <explicit-safe-paths>`
- `svn delete <explicit-clean-local-paths>`
- `svn delete --force <explicit-approved-missing-paths>`
- `svn commit --file <message-file> --encoding UTF-8 <explicit-approved-paths>`

Use read-only local commands or APIs to inspect files, search conflict/secret patterns, inspect nested `.svn` directories, and compare the final review scope. Create and remove only the one temporary message file outside the working copy during an approved CLI commit.

## Forbidden commands

- Any SVN command not listed above, including `svn update`, `revert`, `cleanup`, `resolve`, `resolved`, `remove`, `rename`, `move`, `lock`, `unlock`, and revision-property writes.
- `svn commit -m ...`, a commit without `--encoding UTF-8`, or a commit without explicit approved paths.
- Broad `svn add`, `svn delete`, or `svn commit` using `.`, `*`, an unreviewed directory, or another implicit recursive scope.
- `svn delete <URL>`, `svn delete --keep-local`, or automatic deletion of locally modified files.
- Automatic add of generated, local-only, ambiguous, ignored, or sensitive files.
- Any direct `.svn` modification, automatic conflict resolution, automatic revert, or attempt to repair `svn:log`.

## Review workflow

1. Resolve and state the working-copy root with `svn info`.
2. Run `svn status --xml` as the authoritative local scan. Use plain `svn status` when a compact display helps.
3. Run `svn status -u --xml` when possible. On remote-check failure, continue and report the failure.
4. Run `svn diff --summarize`, `svn diff`, and `svn diff --properties-only` when applicable.
5. Inspect text and property states, tree conflicts, replacements, missing/deleted paths, obstructions, switched paths, externals, nested working copies, locks, and repository-side changes. Treat conflicts, obstructions, approved-scope out-of-date changes, and broken lock state as commit blockers.
6. Inspect every property change. Highlight `svn:externals`, `svn:ignore`, `svn:eol-style`, `svn:mergeinfo`, `svn:mime-type`, `svn:needs-lock`, and `svn:executable` when changed.
7. Inspect content for generated/local files, conflict markers `<<<<<<<`, `=======`, `>>>>>>>`, likely secrets, accidental deletion, unusually large or formatting-only churn, and missing tests. Mask secret values.
8. When preparation was requested, perform only the safe add/delete actions above, then rerun status and relevant diffs.
9. Infer intent only from verified paths and diffs. Do not invent issue IDs, requirements, or test results. Draft a concise Korean message: use 1-2 bullets for small changes and group broader work by intent, normally within 7 bullets.
10. Produce the six-section Korean review. For an empty category write `- 없음`; when no meaningful risk exists write `특별한 위험 요소는 보이지 않습니다.`
11. In CLI mode, show the final included and excluded paths, prepared add/delete paths, message, and remaining risks. Ask the one final question.
12. After approval, rerun target-scoped `svn status --xml`, `svn diff --summarize`, `svn diff`, and property diff. If the approved scope or meaningful content changed, stop and show the new review; otherwise commit and verify the exact revision when possible.

## Risk checklist

Report only applicable risks:

- Excluded files: build/generated output, IDE/user files, logs, temporary/cache files, local config, archives, dumps, backups, screenshots.
- Sensitive data: `.env`, token, password, API/access key, private key, credential, private URL, or connection string.
- Blocking SVN state: text/property/tree conflict, obstruction, broken lock, or approved-path out-of-date/remote change.
- Review-required SVN state: unversioned, added, missing, deleted, replaced, switched, external, nested working copy, directory scope, or significant property change.
- Content risk: conflict marker, suspiciously large/generated diff, mixed formatting and logic, accidental deletion, or risky logic without tests.
- Verification risk: tests not run, partial validation, remote freshness failure, final scope change, revision parse failure, or exact message mismatch.
- Encoding risk: PowerShell `-m`, message file without explicit UTF-8, BOM/default-encoding ambiguity, leftover temporary message file, or broken stored Korean message.

## Output format

Always write the result in Korean using this structure.

````markdown
# SVN 커밋 전 검토 결과

## 1. 커밋 가능 여부

판단: 가능 / 조건부 가능 / 보류 권장

이유:
- ...

## 2. 변경 파일 요약

### Modified
- `path`: 변경 내용

### Added / Unversioned
- `path`: add 상태 또는 add 예정

### Deleted / Missing
- `path`: 삭제 의도 또는 delete 예정

### Conflicted
- `path`: text/property/tree conflict 상태

### Out-of-date / Remote-changed
- `path`: 원격 상태

### Other
- `path`: replaced, obstructed, switched, external, nested working copy, lock, property 변경

## 3. 커밋 전 확인 필요 항목

- 위험 유형: 경로와 필요한 조치

문제가 없으면:
특별한 위험 요소는 보이지 않습니다.

## 4. 변경 내용 요약

- 변경 의도와 영향
- ...

## 5. 추천 커밋 메시지

```text
[카테고리] 한 줄 제목

변경사항:
- ...

검토:
- svn status 확인
- svn diff 확인
- 테스트: 실행함 / 미실행 / 확인 필요

주의:
- 없음 / ...
```

## 6. TortoiseSVN에서 할 일

- [ ] Check for Modifications에서 포함/제외 범위 확인
- [ ] 파일 및 property Diff 확인
- [ ] 필요한 신규 파일과 삭제 예약만 확인
- [ ] 충돌, missing, remote, external, switched, lock 상태 확인
- [ ] 로컬/생성/민감 파일 제외
- [ ] 테스트 결과와 추천 커밋 메시지 확인

커밋 범위:
- 포함: ...
- 자동 add: 없음 / ...
- 삭제 예약: 없음 / ...
- 제외: 없음 / ...
- 실행 방식: TortoiseSVN 수동 / Codex CLI
````

In CLI mode, append this section after the commit attempt:

````markdown
## 7. 커밋 실행 결과

- 최종 범위 재확인: 일치 / 변경으로 중단 / 확인 실패
- 실행 결과: 성공 / 실패 / 결과 불명확
- 커밋 리비전: rN / 확인 실패
- 메시지 검증: 정확히 일치 / 불일치 / 확인 실패
- 임시 메시지 파일: 해당 없음 / 삭제 완료 / 삭제 실패
- 최종 SVN 상태: 확인 결과
- 후속 조치: 없음 / ...
````

Judgment:

- `가능`: 필수 파일이 추적되었거나 자동 준비되었고, 차단 요인이 없으며 범위와 검증이 충분하다.
- `조건부 가능`: 테스트, 원격 확인, property 검토, 모호한 파일, 또는 의도 확인이 남아 있다.
- `보류 권장`: 충돌, 민감정보 가능성, obstruction, broken lock, 의심스러운 삭제, 위험한 범위, out-of-date 대상, 또는 변경 의도 불명이 있다.

## TortoiseSVN final checklist

Use section 6 as the single operational checklist. Keep detailed file explanations in section 2 and do not repeat low-value warnings.

- In manual mode, tell the user to paste the Korean message into TortoiseSVN and stop.
- In CLI mode, ask only the one final question after showing the final scope and message.
- After a CLI commit, verify only the revision parsed from that command output; never use `--limit 1` as proof.
- Report temporary message-file cleanup and any encoding-verification gap without exposing its machine-specific path.
