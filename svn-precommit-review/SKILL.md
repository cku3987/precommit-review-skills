---
name: svn-precommit-review
description: SVN/TortoiseSVN 커밋 전에 변경사항과 위험을 검토하고 한국어 커밋 메시지를 작성한다. 기본 실행은 명확히 필요한 신규 파일의 안전한 svn add까지 포함한다. 읽기 전용 요청은 변경 없이 검토하며, 커밋 요청 시 범위와 메시지에 대한 최종 확인 한 번 뒤 명시한 대상만 CLI로 커밋한다.
---

# SVN Pre-commit Review

## Purpose

Use this instruction-only skill immediately before an SVN/TortoiseSVN commit.

Review the user-specified working copy, prepare clear required new files, explain remaining risks, and draft a concise Korean commit message. Execute a narrowly scoped CLI commit only when requested and finally confirmed. Use the current directory only when no path is supplied. Never silently switch to another working copy or expand an explicitly limited file scope.

## Strict safety rules

- Always write review and commit results in Korean.
- Follow the user's explicit mode and exclusions. Default skill invocation includes review and safe required-file add, not permission to commit or delete.
- Do not edit project file contents or directly modify `.svn`. Only the scoped SVN mutations and temporary message-file operations below are allowed.
- Never expose a secret value. Report only the path, optional line number, and risk type; replace values with `[REDACTED]`.
- Continue when `svn status -u` fails due to network, certificate, authentication, or server errors. Report that remote freshness was not verified.
- Never infer a successful check from missing, truncated, or failed output.

### Execution mode and confirmation

- **Read-only:** explicit `검토만`, `읽기 전용`, `메시지만`, or no-change requests prohibit add, delete, commit, and temporary-file creation. Report required but untracked files without changing them.
- **Prepare (default):** a bare skill invocation or a general request such as `검토해줘` reviews and automatically adds clear required files. Stop after the review unless a commit is requested. Explicit manual TortoiseSVN requests use this preparation but never trigger a CLI commit; read-only instructions still take precedence.
- **CLI commit:** `커밋해줘` or equivalent requests enter this mode; the literal word `CLI` is not required. A preparation-only request does not. Show included paths, exclusions, preparation results, remaining risks, and the exact message, then ask once: `이 범위와 메시지로 CLI 커밋할까요?`
- A clear reply such as `예`, `네`, `응`, `진행`, `진행해`, `커밋해`, or `커밋해줘` to that final question authorizes that scope and message. Carry this authorization across turns; do not restart the approval flow because the user did not repeat `CLI` or every path.
- Immediately before committing, rerun target status and relevant diffs. Request renewed approval only for substantive scope/content changes or newly discovered commit blockers. Otherwise use the existing final approval. An initial commit request alone is not the final confirmation.

### Add and delete preparation

- In prepare or CLI mode, inspect unversioned candidates even though `svn diff` does not include their contents. Use tracked project/build references, imports, or the reviewed change to establish that each source, project, resource, or non-local configuration file is required and within scope. Inspect its content and SVN/ignore state before adding it.
- When these checks pass, run `svn add` on the explicit safe paths without asking a separate add question. Do not stop at a recommendation that an essential source file should be added. An explicitly excluded or out-of-scope dependency needs a reported scope decision, not an automatic override.
- Do not add build output, generated bundles, IDE/user files, logs, caches, local settings, archives, dumps, backups, screenshots, `.env`, or likely secret-bearing files.
- Inspect unversioned directories recursively; status may show only the parent. Add a directory recursively only when every included child is intended and safe. For mixed content, schedule necessary safe parent directories from the top down with `svn add --depth empty <parent>`, then add only the explicit safe files. Do not force-add ignored files or cross externals/nested working copies.
- Check the command result and rerun `svn status --xml`; report add completed only when the path is `added` (`A`). Review its resulting content and properties. A failed or unverified add remains an unresolved item with the actual error; a successful add belongs in the file summary, not a stale missing-source warning.
- Prepare independent safe files even when a code finding or test gap remains. Adding a file does not fix its logic or prove a build; keep those findings for commit judgment. Block a preparation action only when the affected path cannot itself be safely prepared.
- When the user asks to include reviewed deletions, allow `svn delete` or `svn delete --force` for explicitly listed clean versioned local candidates with clear deletion intent. A reply such as `삭제도 포함해줘` may approve the reviewed deletion candidates; do not require the user to repeat every path.
- Never schedule deletion for a locally modified, conflicted, switched, external, locked, or ambiguous path. Never delete a repository URL, use `--keep-local`, or use a broad path.

### Korean commit message and cleanup

- For Korean CLI messages, use a unique temporary file outside the working copy, written as UTF-8 without BOM and normalized to LF. Do not use PowerShell `-m`, `Out-File`, or default `Set-Content`.
- Commit only with `svn commit --file <message-file> --encoding UTF-8 <explicit-approved-paths>`.
- Use a `finally` cleanup path: remove only this temporary message file after success, failure, or cancellation, including errors before the commit command. Confirm its absence.
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
- `svn add --depth empty <explicit-safe-parent-directory>`
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
8. Apply the selected execution mode and complete eligible preparation before reporting. Verify resulting status and diffs; retain only unresolved risks. Do not let an independent code finding turn every safe add into a question.
9. Use only observed test/build evidence. Reuse prior results only when the relevant reviewed content has not changed, naming the prior result and that it was not rerun. Otherwise report unverified coverage; do not run builds/tests that would modify the working copy under this skill.
10. Infer intent only from verified paths and diffs. Do not invent issue IDs, requirements, or test results. Group the Korean summary and message by intent without a fixed bullet count: small changes usually need 1-2 bullets, broader work normally fits within 7. Omit per-line narration and repeated warnings.
11. Produce the six-section Korean review once, including preparation outcomes and unresolved risks. For an empty category write `- 없음`; when no meaningful risk exists write `특별한 위험 요소는 보이지 않습니다.`
12. In CLI mode, follow the confirmation rules above. After final approval, recheck status and relevant content/property diffs, commit the approved scope, clean up, and verify the exact revision when possible. Report only the changed review or execution outcome on follow-up, rather than repeating the entire unchanged review.

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

Write the initial review in Korean using the six-section structure below. On follow-up, output only changed sections or section 7 for a commit attempt; do not repeat the unchanged review.

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
- `path`: add 완료 (A 확인) / 이미 Added / 미추적 유지 및 이유

### Deleted / Missing
- `path`: 삭제 의도 또는 delete 예정

### Conflicted
- `path`: text/property/tree conflict 상태

### Out-of-date / Remote-changed
- `path`: 원격 상태

### Other
- `path`: replaced, obstructed, switched, external, nested working copy, lock, property 변경

## 3. 커밋 전 확인 필요 항목

- 미해결 위험 유형: 경로와 필요한 조치 (완료된 add는 반복하지 않음)

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
- 테스트: 실행함 / 이전 실행 결과와 적용 범위 (이번 검토 미실행) / 미실행 / 확인 필요

주의:
- 없음 / ...
```

## 6. TortoiseSVN에서 할 일

- [ ] Check for Modifications에서 포함/제외 범위 확인
- [ ] 파일 및 property Diff 확인
- [ ] 준비된 신규 파일과 삭제 예약 확인 (미해결 항목이 있으면 별도 처리)
- [ ] 충돌, missing, remote, external, switched, lock 상태 확인
- [ ] 로컬/생성/민감 파일 제외
- [ ] 테스트 결과와 추천 커밋 메시지 확인

커밋 범위:
- 포함: ...
- 자동 add: 없음 / ...
- 삭제 예약: 없음 / ...
- 제외: 없음 / ...
- 실행 방식: TortoiseSVN 수동 / Codex CLI
- 검토 모드: 읽기 전용 / 필수 파일 준비
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

- Judge the intended commit scope after preparation. Files verified as excluded, including excluded local secret candidates, do not by themselves block that commit; ensure directory targets cannot include them.
- `가능`: 필수 파일이 추적되었거나 자동 준비되었고, 차단 요인이 없으며 범위와 검증이 충분하다.
- `조건부 가능`: 테스트, 원격 확인, property 검토, 모호한 파일, 또는 의도 확인이 남아 있다.
- `보류 권장`: 커밋 대상에 충돌, 민감정보 가능성, obstruction, broken lock, 의심스러운 삭제, 위험한 범위, out-of-date 상태, 필수 빌드 파일 누락, 근거 있는 중대한 로직 위험, 또는 변경 의도 불명이 있다.

## TortoiseSVN final checklist

Use section 6 as the single operational checklist and keep only applicable items. Keep file explanations and verified add results in section 2; do not tell the user to add a file already confirmed as `A`. In CLI mode this is a scope summary, not a requirement to open TortoiseSVN or commit manually.

- In manual mode, tell the user to paste the Korean message into TortoiseSVN and stop.
- In CLI mode, use the confirmation rules above without a second approval or a manual-commit handoff.
- After a CLI commit, verify only the revision parsed from that command output; never use `--limit 1` as proof.
- Report temporary message-file cleanup and any encoding-verification gap without exposing its machine-specific path.
