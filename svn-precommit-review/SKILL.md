---
name: svn-precommit-review
description: SVN/TortoiseSVN 커밋 직전에 사용. svn status와 svn diff를 구조적으로 검토하고 위험 파일, 충돌, 누락 파일, property 변경, 민감정보 가능성을 점검한 뒤 한국어 커밋 메시지를 작성한다. 기본은 읽기 전용이다. 사용자가 명시적으로 요청한 경우에만 안전 기준을 통과한 필수 신규 파일의 svn add, 깨끗한 versioned 경로의 svn delete 예약, 최종 승인된 명시 경로의 CLI 커밋을 수행한다. CLI 커밋 전에는 승인 시점의 상태, diff, 파일 해시를 다시 비교하고, 한글 메시지는 BOM 없는 UTF-8 파일과 --encoding UTF-8을 사용하며 커밋 결과의 정확한 리비전으로 검증한다.
---

# SVN Pre-commit Review

## Purpose

Use this instruction-only skill immediately before an SVN/TortoiseSVN commit.

Review the user-specified working copy without editing project content, judge commit readiness, explain only meaningful risks, and produce a concise Korean commit message and final checklist. Use the current directory only when the user did not specify another path, and never operate on a different working copy silently.

## Strict safety rules

- Keep project files and `.svn` contents unchanged except for the explicitly gated SVN scheduling operations below.
- Treat every SVN command not listed in **Allowed commands** as forbidden.
- Never infer a successful check from missing, truncated, or failed command output.
- Continue a review when remote access fails, but report `원격 최신 여부 확인 실패`.
- Never expose a discovered secret value. Report only the path, optional line number, and risk type; replace the value with `[REDACTED]`.
- Always output the review and commit result in Korean.

Finalization modes:

| Mode | Trigger | Final action |
| --- | --- | --- |
| TortoiseSVN manual | Default for review, message drafting, or manual commit requests | Do not ask for CLI approval and never run `svn commit`; stop after the checklist |
| Codex CLI commit | The user explicitly asks Codex to commit | Show exact scope and message, take an approval snapshot, then ask the direct CLI execution question |

In active CLI commit mode, a clear reply such as `예`, `네`, `응`, `진행해`, or `커밋해줘` approves execution only after the direct question `위 대상과 커밋 메시지로 Codex가 지금 CLI 커밋을 실행해도 될까요?`. A prior generic affirmative does not authorize a commit.

Mutation gates:

| Operation | Required conditions |
| --- | --- |
| Safe add | The user asked for preparation, auto-add, or commit; each explicit unversioned file is inside this working copy, inspected, non-sensitive, non-generated, and required by a tracked build/runtime reference |
| Safe delete | The user re-requested exact reviewed paths; each target is a clean versioned local path with clear deletion intent and no conflict, remote change, switch, external boundary, lock anomaly, or local text/property modification |
| CLI commit | CLI mode is active; exact targets and message were reviewed; blockers are absent; approval snapshot creation succeeded; the user approved the direct question; the pre-commit comparison is identical |

Safe-add rules:

- Add explicit files, not `.` or wildcards.
- Exclude build output, generated bundles, IDE/user files, logs, caches, local config, archives, dumps, backups, screenshots, `.env`, and secret-bearing files.
- Add a directory only after recursively inspecting every child and confirming that every added child passes the same rules.
- If build/runtime necessity is uncertain, do not add it; report `수동 확인 후 add 필요`.

Safe-delete rules:

- Accept only working-copy filesystem paths. Never pass a URL to `svn delete`; URL deletion commits directly to the repository.
- Do not automatically delete a file with local text or property modifications, even after a general deletion request. Report it for manual handling.
- Use `svn delete <exact-local-path>` only for clean versioned targets.
- Use `svn delete --force <exact-missing-path>` only when the reviewed target is already missing and the user explicitly approved that exact path.
- Never use `--keep-local`.
- Delete a directory only when every descendant has been inspected, is safe, and the user explicitly approved the whole tree.

Approval snapshot rules:

1. After all approved `svn add` and `svn delete` operations, but before asking the CLI execution question, freeze the ordered commit target list, excluded list, exact message, and working-copy root.
2. Record in memory a SHA-256 fingerprint covering full `svn status --xml`, target-scoped `svn diff`, target property diff, and the ordered target list.
3. Record SHA-256 for every approved existing file so binary or otherwise abbreviated diffs cannot hide content changes. Deleted paths are covered by status and diff.
4. Record the `svn status -u --xml` result for the approved scope when remote access succeeds.
5. Do not create snapshot files in the working copy.
6. Immediately before commit, rerun the same commands and hashes. Any changed fingerprint, file hash, target list, property state, new unapproved local change, conflict, or approved-scope remote change invalidates approval. Stop, re-review, and ask again.

Commit-message and post-commit rules:

- Create the temporary message file only after the post-approval snapshot comparison passes. Use a unique OS-temp path outside the working copy, normalize the intended message to LF with no unintended trailing newline, and write it with a no-BOM encoder such as `[IO.File]::WriteAllText($path, $message, (New-Object Text.UTF8Encoding($false)))`. Do not use PowerShell `Out-File` or `Set-Content` defaults.
- Use only `svn commit --file <message-file> --encoding UTF-8 <approved-paths>`. Never use `-m` for Korean text.
- Prefer explicit file targets. A directory target is allowed only after recursive XML status confirms that every changed descendant exactly matches the approved set and no external, nested working copy, switched path, conflict, obstruction, or unapproved change exists.
- Delete the message file after every commit attempt, including failures, and confirm it is absent.
- On success, parse the committed revision `N` from that commit command's output. Never substitute `svn log --limit 1`, because concurrent commits can make it point to another revision.
- Obtain the repository root URL with `svn info --show-item repos-root-url <working-copy-root>`, or parse `<root>` from `svn info --xml` when `--show-item` is unavailable. Run `svn log --xml -r N <repository-root-url>`, parse XML, confirm the `<logentry>` revision is `N`, and compare `<msg>` character-for-character with the original message.
- If the revision cannot be parsed or exact log verification fails, report `확인 실패` without retrying the commit or changing revision properties. Broken historical log messages may require a server-side `pre-revprop-change` hook.

## Allowed commands

Read-only SVN commands:

- `svn status`
- `svn status --xml`
- `svn status --xml --no-ignore` when ignored/unversioned scope must be inspected
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
- `svn commit --file <message-file> --encoding UTF-8 <approved-paths>`

Use read-only local commands or APIs to inspect files, find nested `.svn` directories, search conflict/secret patterns, and compute SHA-256 in memory. In CLI mode only, create and remove the one temporary UTF-8 message file outside the working copy.

## Forbidden commands

- Any SVN command not listed above, including `svn update`, `revert`, `cleanup`, `resolve`, `resolved`, `remove`, `rename`, `move`, `lock`, `unlock`, and revision-property writes.
- `svn commit -m ...`, `svn commit --file ...` without `--encoding UTF-8`, a commit without explicit approved paths, or a commit after snapshot mismatch.
- Broad `svn add`, `svn delete`, or `svn commit` using `.`, `*`, an unreviewed directory, or another implicit recursive scope.
- `svn delete <URL>`, `svn delete --keep-local`, or automatic deletion of locally modified files.
- Automatic add of generated, local-only, ambiguous, ignored, or sensitive files.
- A directory commit containing an external, nested working copy, switched path, conflict, obstruction, or descendant outside the exact approved set.
- Any edit, format, generation, migration, build, or installer command that can rewrite project files.
- Any direct `.svn` modification, automatic conflict resolution, automatic revert, or attempt to repair `svn:log`.

## Review workflow

1. Resolve and state the working-copy root. Use `svn info` and obtain the repository root URL when CLI commit is requested.
2. Run `svn status --xml` as the authoritative local-state scan. Use plain `svn status` only when a compact human-readable view helps.
3. Run `svn status -u --xml` when possible. On network, certificate, server, or authentication failure, continue and report the failure.
4. Run `svn diff --summarize`, `svn diff`, and `svn diff --properties-only` as applicable.
5. Parse XML structurally rather than relying only on status-column text. Inspect:
   - text `item` and property `props` states
   - `tree-conflicted`, `switched`, and `wc-locked`
   - local/repository lock presence or token mismatch
   - repository-side status from `-u`
   - actual status entries at `/status/target/entry`; never count an empty `<target>` node as a change
6. Treat text/property conflicts, tree conflicts, obstructions, broken lock state, and approved-path out-of-date/remote changes as commit blockers. Review replaced (`R`), missing, deleted, switched, external, and nested-working-copy states explicitly.
7. Inspect every property change. Highlight at least `svn:externals`, `svn:ignore`, `svn:eol-style`, `svn:mergeinfo`, `svn:mime-type`, `svn:needs-lock`, and `svn:executable` when changed.
8. Inspect content for generated/local files, conflict markers `<<<<<<<`, `=======`, `>>>>>>>`, likely secrets, accidental deletion, large or formatting-only churn, and missing tests. Mask every sensitive value.
9. If requested, perform only safe add/delete operations that pass their gates, then rerun structured status and relevant diffs.
10. Infer intent from verified paths and diffs. Do not invent requirements, issue IDs, motivations, or test results. Draft a concise Korean message with only as many bullets as needed.
11. Produce the six-section Korean review. For an empty status category, write `- 없음`; for no meaningful risk, write `특별한 위험 요소는 보이지 않습니다.`
12. In manual mode, stop after the checklist without a CLI approval question.
13. In CLI mode, create the approval snapshot, show exact included/excluded targets and message, then ask the direct execution question.
14. After clear approval, compare the snapshot again. If it differs, do not create a message file. If it is identical, create the message file, commit, verify the exact revision, and clean up.

## Risk checklist

Report only applicable risks, grouped by meaning:

- Excluded files: build/generated output, IDE/user files, logs, temp/cache files, local config, archives, dumps, backups, screenshots.
- Sensitive data: `.env`, token, password, API/access key, private key, credential, private URL, or connection string. Never quote its value.
- Blocking SVN state: text/property/tree conflict, obstruction, broken lock, approved-path out-of-date/remote change, or post-approval snapshot mismatch.
- Review-required SVN state: unversioned, added, missing, deleted, replaced, switched, external, nested working copy, directory target, or significant property change.
- Content risk: conflict marker, suspiciously large/generated diff, mixed formatting and logic, accidental deletion, or risky logic without tests.
- Verification risk: tests not run, partial validation, remote freshness failure, snapshot failure, revision parse failure, or exact message mismatch.
- Encoding risk: PowerShell `-m`, message file without explicit UTF-8, BOM/default-encoding ambiguity, leftover temporary message file, or use of latest-log lookup instead of the committed revision.

## Output format

Always write the result in Korean using this structure. Keep descriptions concise and write `- 없음` under empty file-state categories.

````markdown
# SVN 커밋 전 검토 결과

## 1. 커밋 가능 여부

판단: 가능 / 조건부 가능 / 보류 권장

이유:
- ...

## 2. 변경 파일 요약

### Modified
- `path`: 변경 의미

### Added / Unversioned
- `path`: 의미와 add 상태

### Deleted / Missing
- `path`: 삭제 의도와 delete 상태

### Conflicted
- `path`: text/property/tree conflict 상태

### Out-of-date / Remote-changed
- `path`: 원격 상태

### Other
- `path`: replaced, obstructed, switched, external, nested working copy, lock, property 등

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

- [ ] Check for Modifications에서 포함/제외 대상 확인
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
- 남은 확인 사항: 없음 / ...
- 승인 스냅샷: 수동 모드로 해당 없음 / 기록 완료 / 생성 실패
- 실행 방식: TortoiseSVN 수동 / Codex CLI

마지막 안내:
- 수동 모드: `Codex는 svn commit을 실행하지 않습니다. 위 내용을 확인한 뒤 TortoiseSVN에서 직접 커밋해 주세요.`
- CLI 모드: `위 대상과 커밋 메시지로 Codex가 지금 CLI 커밋을 실행해도 될까요? "예", "네", "응", "진행해"처럼 명확히 동의하면 CLI 커밋을 실행합니다.`
````

Use only the one mode-appropriate final sentence. Use 1-2 summary/message bullets for small changes; group broader work by intent and normally stay within 7 bullets. Include an issue ID only when clearly evidenced.

Judgment:

- `가능`: required files are tracked, blockers are absent, scope is understood, and validation is adequate.
- `조건부 가능`: specific manual confirmation, tests, add/delete action, property review, or remote check remains.
- `보류 권장`: conflict, likely secret, obstruction, broken lock, suspicious deletion, unsafe scope, out-of-date target, snapshot mismatch, or insufficient understanding exists.

After a CLI commit attempt, append:

````markdown
## 7. 커밋 실행 결과

- 사전 스냅샷 비교: 일치 / 불일치로 중단 / 확인 실패
- 실행 결과: 성공 / 실패 / 결과 불명확
- 커밋 리비전: rN / 확인 실패
- 커밋 대상: ...
- 메시지 검증: `svn log --xml -r N <repository-root-url>` 기준 원문과 정확히 일치 / 불일치 / 확인 실패
- 임시 메시지 파일: 해당 없음 / 삭제 완료 / 삭제 실패
- 최종 SVN 상태: 확인 결과
- 남은 조치: 없음 / ...
````

Never expose the temporary file's machine-specific absolute path or any secret value in the result. If commit success is ambiguous, do not retry automatically.

## TortoiseSVN final checklist

Use section 6 as the single checklist; do not repeat the same risks elsewhere merely to fill space.

- Include only actions relevant to the observed working copy.
- Put detailed per-file explanations in section 2 and keep section 6 operational.
- Require manual review of automatically added and scheduled-delete paths.
- Require property review when XML or diff reports property changes.
- In manual mode, instruct the user to paste the Korean message into TortoiseSVN and stop.
- In CLI mode, show the frozen scope/message and snapshot status before asking the direct question.
- After CLI commit, verify only the revision parsed from that commit output; never use `--limit 1` as proof.
- Do not create a temporary message file before the post-approval comparison passes. Report `해당 없음` when none was created; otherwise confirm cleanup and report any verification gap explicitly.
