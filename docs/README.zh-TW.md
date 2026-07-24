# svn-precommit-review 指南

`svn-precommit-review` 是一個 Codex Agent Skill，用於在 SVN/TortoiseSVN 提交前檢查 working copy 變更，並產生韓文提交訊息草稿。預設檢查為唯讀；只有在使用者明確要求提交準備或自動 add 時，才會依安全條件對可能導致 build/runtime 失敗的必要新檔案執行 `svn add`。檢查後只有在使用者再次明確要求刪除具體路徑時，才會依安全條件對已版本控制檔案執行 `svn delete`。

## 安裝

使用者層級:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

專案層級:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

將本儲存庫的 `svn-precommit-review/SKILL.md` 複製到所需位置即可。

## 使用方式

```text
$svn-precommit-review 請在提交前檢查目前 SVN 變更，並產生韓文提交訊息。
```

指定 working copy:

```text
$svn-precommit-review 請檢查 <SVN_WORKING_COPY> 路徑下的 svn status 與 svn diff，並提供提交建議。
```

安全 add 必要新檔案:

```text
$svn-precommit-review 請檢查 <SVN_WORKING_COPY> 的變更；若必要新檔案會影響 build，且通過安全條件，請執行 svn add。
```

## 重點

- 結構化檢查 `svn status --xml`、`svn status -u --xml`、內容 diff 與 property diff
- 偵測建置輸出、IDE 檔案、log、暫存檔、本機設定
- 檢查 `.env`、token、password、API Key 等敏感資訊，實際值一律以 `[REDACTED]` 遮罩
- 檢查 text/property/tree conflict，以及 replaced、obstructed、switched、external、nested working copy 與 lock 狀態
- 只有在明確要求時，才對通過安全條件的 build 必要新檔案執行 `svn add`
- 只對檢查後再次明確要求且沒有本機修改的具體本機路徑執行 `svn delete`；不自動刪除已修改檔案或儲存庫 URL
- 在核准前與 commit 前比較 status、diff、property、目標與檔案內容的 SHA-256 snapshot
- 預設以檔案為單位 commit；只有檢查全部子項目與 SVN 邊界後才允許目錄目標
- 空的輸出分類簡潔寫為 `- 없음`
- 在 TortoiseSVN 手動提交模式下，Codex 不會執行 `svn commit`，也不會詢問 CLI 執行授權
- 只有在使用者事先明確要求 Codex 執行 CLI commit、檢查目標與訊息後，對直接執行問題給出明確肯定答覆，才會執行受保護的 CLI commit

只有在 CLI 模式已被明確要求，而且 Codex 剛提出直接執行問題時，`是`、`yes`、`繼續` 等簡短明確的肯定答覆才會授權 CLI commit。僅檢查的要求會在不提出該問題的情況下結束。

## 韓文提交訊息編碼

避免在 PowerShell 使用:

```powershell
svn commit -m "한글 커밋 메시지"
```

CLI commit 請使用無 BOM、LF 換行的 UTF-8 訊息檔並明確指定 `--encoding UTF-8`。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

從 commit 結果讀取 revision 編號，並只驗證該 revision。

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

將 XML `<msg>` 與原始訊息精確比較。可能有同時 commit，因此不使用 `--limit 1` 作為驗證依據。

## 更多資訊

預設英文文件: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
