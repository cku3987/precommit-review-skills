# svn-precommit-review 사용 가이드

`svn-precommit-review`는 SVN/TortoiseSVN 커밋 직전에 변경사항을 검토하고 한국어 커밋 메시지를 작성하는 Codex Agent Skill입니다. 기본은 읽기 전용이며, Codex CLI 커밋을 요청하면 필요한 신규 소스 파일을 안전하게 add하고 최종 확인 한 번 뒤 승인 범위만 커밋할 수 있습니다.

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

읽기 전용 검토:

```text
$svn-precommit-review 현재 SVN 변경사항을 커밋 전에 검토하고 한국어 커밋 메시지를 작성해줘.
```

경로 지정 검토:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로의 SVN 변경사항을 검토해줘.
```

필수 신규 파일 준비와 CLI 커밋 요청:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고 필요한 신규 소스 파일은 add한 뒤, 최종 확인을 받으면 승인 범위만 CLI로 커밋해줘.
```

검토된 삭제 후보 포함 요청:

```text
$svn-precommit-review 방금 검토한 삭제 후보도 포함해서 커밋 준비해줘.
```

검토 결과에 포함 범위와 메시지가 표시된 후 한 번만 답하면 됩니다.

```text
네, 진행해
```

## 동작 범위

- `svn status --xml`, `svn status -u --xml`, `svn diff --summarize`, 내용 diff, property diff를 검토합니다.
- 빌드 산출물, IDE/user 파일, 로그/임시 파일, 로컬 설정, 민감정보, 충돌 마커, 누락 파일, 원격 변경, property 변경을 점검합니다.
- CLI 커밋 요청 시 변경 의도가 분명한 신규 소스, 프로젝트, 리소스, 필요한 설정 파일은 자동으로 `svn add`할 수 있습니다.
- `bin`, `obj`, 로그, 캐시, 압축 파일, `.env`, 비밀값 가능 파일, 생성물, 모호한 파일은 add하지 않습니다.
- 사용자가 삭제 후보 포함을 요청하면, 검토된 깨끗한 versioned 로컬 파일만 `svn delete`로 삭제 예약할 수 있습니다.
- 로컬 수정, 충돌, switched, external, lock, 의도 불명 파일은 삭제 예약하지 않습니다.
- 커밋 직전에 상태와 대상 diff를 한 번 더 확인합니다. 범위나 의미 있는 변경이 달라지거나 차단 상태가 생긴 경우에만 다시 검토합니다.
- `svn add .`, `svn delete .`, `svn commit .` 같은 넓은 범위 명령은 사용하지 않습니다.

## 한글 커밋 메시지

CLI 커밋에는 BOM 없는 UTF-8 임시 메시지 파일과 `--encoding UTF-8`을 사용합니다.

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

커밋 뒤 임시 파일을 지우고, 가능하면 실제 커밋 리비전의 XML 로그 메시지를 원본과 비교합니다. 저장소에 한글 메시지가 깨져 저장되면 커밋 재시도나 revision property 수정은 하지 않고 검증 실패로 보고합니다.

## 요구 사항

PowerShell 또는 사용할 셸에서 `svn` 명령이 실행되어야 합니다.

```powershell
svn --version --quiet
svn status
```

TortoiseSVN 설치 시 command-line client를 함께 설치했거나 별도 Apache Subversion CLI가 설치되어 있으면 됩니다.
