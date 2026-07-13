# svn-precommit-review 사용 가이드

`svn-precommit-review`는 SVN/TortoiseSVN 커밋 직전에 변경사항을 검토하고 한국어 커밋 메시지를 작성하는 Codex Agent Skill입니다. 기본 검토는 읽기 전용이며, 사용자가 커밋 준비나 자동 add를 명시적으로 요청한 경우에만 빌드가 깨질 수 있는 필수 신규 파일을 안전 기준에 따라 `svn add`할 수 있습니다. 검토 후 사용자가 명시 경로 삭제를 다시 요청한 경우에는 안전 기준을 통과한 versioned 파일만 `svn delete`로 삭제 예약할 수 있습니다.

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

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff` 확인
- 빌드 산출물, IDE 파일, 로그/임시 파일, 로컬 설정 파일 점검
- `.env`, 토큰, 비밀번호, API Key 등 민감정보 가능성 점검
- 충돌 마커와 missing/deleted 파일 점검
- 빌드가 깨질 수 있는 필수 신규 파일은 명시 요청 시 안전 기준을 통과한 경우에만 `svn add`
- 검토 후 재요청된 명시 경로만 안전 기준을 통과한 경우에 `svn delete`
- 커밋 가능 여부 판단: `가능`, `조건부 가능`, `보류 권장`
- 한국어 커밋 메시지 초안 작성
- 명시 요청과 최종 승인 후에만 안전 커밋 수행

## 안전 커밋

PowerShell에서 다음 방식은 피하세요.

```powershell
svn commit -m "한글 커밋 메시지"
```

CLI 커밋이 필요하면 다음 형식을 사용합니다.

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

커밋 후 실제 저장소 메시지는 XML 출력으로 확인합니다.

```powershell
svn log --xml --limit 1
```

## 자세한 문서

영어 기본 문서: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
