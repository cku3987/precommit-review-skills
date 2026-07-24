# svn-precommit-review 사용 가이드

`svn-precommit-review`는 SVN/TortoiseSVN 커밋 직전에 변경사항을 검토하고 한국어 커밋 메시지를 작성하는 Codex Agent Skill입니다. 기본 검토는 읽기 전용이며, 사용자가 커밋 준비나 자동 add를 명시적으로 요청한 경우에만 빌드가 깨질 수 있는 필수 신규 파일을 안전 기준에 따라 `svn add`할 수 있습니다. 검토 후 사용자가 명시 경로 삭제를 다시 요청한 경우에는 안전 기준을 통과한 깨끗한 versioned 파일만 `svn delete`로 삭제 예약할 수 있습니다.

## 설치

사용자 전역:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

프로젝트 로컬:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

이 저장소의 `svn-precommit-review/SKILL.md`를 원하는 위치에 복사하세요.

## 기본 사용

```text
$svn-precommit-review 현재 SVN 변경사항을 커밋 전에 검토하고 한국어 커밋 메시지를 작성해줘.
```

경로 지정:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로의 SVN 변경사항을 커밋 전에 검토해줘.
```

필수 신규 파일 safe add:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고, 빌드가 깨질 수 있는 필수 신규 파일은 안전 기준을 통과하면 svn add까지 해줘.
```

검토 후 명시 경로 safe delete:

```text
$svn-precommit-review 방금 검토한 path/to/obsolete-file.cs 삭제가 의도된 것이 맞으니, 안전 기준을 통과하면 svn delete로 삭제 예약해줘.
```

## 주요 기능

- `svn status --xml`, `svn status -u --xml`, `svn diff --summarize`, `svn diff`, property diff 확인
- 빌드 산출물, IDE 파일, 로그/임시 파일, 로컬 설정 파일 점검
- `.env`, 토큰, 비밀번호, API Key 등 민감정보 가능성을 점검하되 실제 값은 `[REDACTED]`로 마스킹
- text/property/tree conflict, missing/deleted/replaced, obstructed, switched, external, nested working copy, lock 상태 점검
- 빌드가 깨질 수 있는 필수 신규 파일은 명시 요청 시 안전 기준을 통과한 경우에만 `svn add`
- 검토 후 재요청된 깨끗한 로컬 명시 경로만 `svn delete`; 로컬 수정 파일과 저장소 URL 삭제는 자동 수행하지 않음
- CLI 승인 전에 상태, diff, property, 대상 파일을 SHA-256으로 기록하고 커밋 직전에 다시 비교하며, 일치하기 전에는 메시지 파일을 만들지 않음
- 파일 단위 커밋을 기본으로 하고 디렉터리는 모든 하위 변경과 SVN 경계를 확인한 경우에만 허용
- 커밋 가능 여부 판단: `가능`, `조건부 가능`, `보류 권장`
- 한국어 커밋 메시지 초안 작성
- TortoiseSVN 수동 커밋 모드에서는 Codex가 `svn commit`을 실행하거나 CLI 실행 승인을 묻지 않음
- Codex CLI 커밋은 사용자가 먼저 CLI 커밋 실행을 명시적으로 요청하고, 정확한 대상과 메시지를 확인한 뒤 직접 실행 질문에 명확히 동의한 경우에만 수행

## 안전 커밋

PowerShell에서 다음 방식은 피하세요.

```powershell
svn commit -m "한글 커밋 메시지"
```

CLI 커밋이 필요하면 BOM 없는 UTF-8 메시지 파일과 다음 형식을 사용합니다.

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

CLI 커밋 모드가 이미 명시적으로 요청되었고 Codex가 직접 실행 여부를 물은 상태라면 `예`, `네`, `응`, `진행해`, `커밋해줘` 같은 명확한 긍정 응답으로 커밋을 승인할 수 있습니다. 검토 전용 요청이면 Codex는 승인 질문 없이 체크리스트 출력 후 중단하고, 사용자가 TortoiseSVN에서 직접 커밋합니다.

커밋 후에는 실행 결과에서 리비전 번호를 읽고 그 리비전의 메시지만 XML로 확인합니다.

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

XML의 `<msg>`를 원본 메시지와 정확히 비교합니다. `--limit 1`은 동시에 발생한 다른 커밋을 가리킬 수 있으므로 검증에 사용하지 않습니다.

## 자세한 문서

영어 기본 문서: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
