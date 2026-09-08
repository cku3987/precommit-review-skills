# svn-precommit-review 사용 가이드

`svn-precommit-review`는 SVN/TortoiseSVN 커밋 직전에 변경사항을 검토하고 한국어 커밋 메시지를 작성하는 Codex Agent Skill입니다. 기본 호출은 검토와 안전 기준을 충족한 필수 신규 파일의 자동 `svn add`를 수행합니다. 실제 커밋은 사용자 요청과 범위·메시지에 대한 최종 확인 한 번을 거쳐 승인 범위만 실행합니다.

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

기본 검토 및 필수 파일 준비 (`검토해줘` 같은 일반 요청도 동일):

```text
$svn-precommit-review
```

읽기 전용 검토:

```text
$svn-precommit-review 현재 SVN 변경사항을 읽기 전용으로 검토만 해줘. svn add/delete/commit은 실행하지 마.
```

경로 지정 검토:

```text
$svn-precommit-review <SVN_WORKING_COPY> 경로의 SVN 변경사항을 검토해줘.
```

검토 및 커밋 요청:

```text
$svn-precommit-review <SVN_WORKING_COPY> 변경사항을 검토하고 커밋해줘.
```

검토된 삭제 후보 포함 요청:

```text
$svn-precommit-review 방금 검토한 삭제 후보도 포함해서 커밋 준비해줘.
```

커밋을 요청한 경우, 포함·제외 범위, 준비된 add/delete, 남은 위험과 메시지를 확인하고 최종 질문에 한 번 답하면 됩니다. `커밋해줘`면 충분하며 `CLI`라는 표현은 필수가 아닙니다.

```text
네, 진행해
```

## 동작 범위

- `svn status --xml`, `svn status -u --xml`, `svn diff --summarize`, 내용 diff, property diff를 검토합니다.
- 빌드 산출물, IDE/user 파일, 로그/임시 파일, 로컬 설정, 민감정보, 충돌 마커, 누락 파일, 원격 변경, property 변경을 점검합니다.
- 기본 검토에서는 추적 중인 파일의 참조나 검토한 변경으로 필요성이 확인된 미추적 소스, 프로젝트, 리소스, 로컬 전용이 아닌 설정 파일을 반드시 자동 `svn add`합니다. 기준을 충족하면 권고만 남기지 않습니다.
- 읽기 전용, 검토만, 메시지만, 변경 금지를 명시하면 어떤 쓰기도 하지 않습니다. 일반적인 `검토해줘`는 기본 준비를 포함합니다.
- add 명령의 성공과 `svn status`의 `A` 상태를 모두 확인해야 완료로 표시합니다. 완료된 추가는 변경 파일 요약에 기록하고 미해결 경고로 남기지 않습니다.
- 코드 위험이 남아 있어도 독립적인 안전한 add는 진행할 수 있습니다. 해당 위험은 계속 보고하며 add로 코드가 수정되었다고 표현하지 않습니다.
- `bin`, `obj`, 생성물, IDE/user 파일, 로그, 임시 파일, 캐시, 로컬 설정, 압축 파일, 덤프, 백업, 스크린샷, 무시된 파일, `.env`, 비밀값 가능 파일, 모호한 파일은 add하지 않습니다. 디렉터리는 하위 전체를 검사하고 모두 안전할 때만 추가하며, 그렇지 않으면 안전한 개별 경로만 추가합니다.
- 사용자가 삭제 후보 포함을 요청하면, 검토된 깨끗한 versioned 로컬 파일만 `svn delete`로 삭제 예약할 수 있습니다.
- 로컬 수정, 충돌, switched, external, lock, 의도 불명 파일은 삭제 예약하지 않습니다.
- 실제 커밋에는 커밋 요청과 정확한 범위·메시지에 대한 최종 승인이 필요합니다. 같은 범위·내용에 대한 이전 승인은 유지하고, 커밋 직전 status와 내용·property diff를 재확인합니다. 범위·내용이 실질적으로 바뀌거나 차단 상태가 달라진 경우에만 다시 검토하고 확인을 요청합니다.
- TortoiseSVN 수동 커밋을 명시하면 CLI 커밋이나 그 확인 질문 없이 검토와 체크리스트에서 마칩니다. 읽기 전용이 아니라면 기본 안전 준비는 그대로 수행합니다.
- `svn add .`, `svn delete .`, `svn commit .` 같은 넓은 범위 명령은 사용하지 않습니다.

실행 중 안내, 검토, 커밋 메시지와 결과는 모두 한국어로 작성합니다. 요약은 변경 의도별로 묶고 고정된 항목 수나 장황한 설명을 강요하지 않습니다. 이전 테스트는 실제 근거가 있고 검토 대상 내용이 그대로일 때만 재사용하며, 이전 결과이고 이번에는 재실행하지 않았음을 명시합니다.

## 한글 커밋 메시지

CLI 커밋에는 작업 사본 밖의 고유한 임시 메시지 파일을 BOM 없는 UTF-8·LF로 작성하고 `--file`과 `--encoding UTF-8`을 사용합니다.

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

매 커밋 시도 뒤 임시 파일을 지우고 부재를 확인합니다. 가능하면 커밋 명령 출력에서 확인한 실제 리비전의 XML 로그 메시지를 원본과 정확히 비교합니다. 저장소에 한글 메시지가 깨져 저장되면 커밋 재시도나 revision property 수정은 하지 않고 검증 실패로 보고합니다.

## 요구 사항

PowerShell 또는 사용할 셸에서 `svn` 명령이 실행되어야 합니다.

```powershell
svn --version --quiet
svn status
```

TortoiseSVN 설치 시 command-line client를 함께 설치했거나 별도 Apache Subversion CLI가 설치되어 있으면 됩니다.
