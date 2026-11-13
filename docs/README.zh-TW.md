# precommit-review-skills 使用指南

[precommit-review-skills](https://github.com/cku3987/precommit-review-skills) 是用於 Git 與 SVN 提交前審查的獨立 Codex Agent Skill 集合。版本發佈以儲存庫為單位統一進行，各技能獨立安裝與呼叫。既有 `svn-precommit-review` 名稱及其 `SKILL.md` 維持不變，請勿重新命名已安裝的 SVN 技能。

- [SVN 技能](../svn-precommit-review/SKILL.md)
- [Git 技能](../git-precommit-review/SKILL.md)
- [Git 詳細指南](git-precommit-review-guide.md)

## Git

`git-precommit-review` 使用韓語撰寫審查報告與提交訊息。

- 預設呼叫會審查變更，並自動暫存已確認必要且安全的新檔案。忽略檔案、產生檔及含有秘密資訊的檔案均被排除。
- 明確要求唯讀時，不執行任何寫入。
- 要求提交時，會準備已審查的已追蹤檔案變更，同時保留既有的部分暫存狀態。限定範圍的提交不會包含無關的已暫存變更。
- 實際提交需對範圍與訊息進行一次最終確認。`push` 僅在另行要求時執行。

預設呼叫、韓語提交要求（`커밋해줘`）及明確唯讀的範例：

```text
$git-precommit-review
$git-precommit-review 변경사항을 검토하고 커밋해줘.
$git-precommit-review 僅以唯讀方式檢查目前 Git 變更，不執行任何寫入。
```

## SVN

`svn-precommit-review` 是在 SVN/TortoiseSVN 提交前檢查工作副本並產生韓語提交訊息的 Codex Agent Skill。預設呼叫會執行審查，並自動對符合安全條件的必要新檔案執行 `svn add`。實際提交需要使用者提出提交要求，並對範圍與訊息進行一次最終確認，只提交已核准的範圍。

### 安裝

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

或:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

請將此儲存庫的 `svn-precommit-review/SKILL.md` 複製到目標位置。

### 使用方式

預設審查並準備必要檔案（一般的「檢查一下」或 `검토해줘` 也相同）:

```text
$svn-precommit-review
```

唯讀檢查:

```text
$svn-precommit-review 僅以唯讀方式檢查目前 SVN 變更。不要執行 svn add/delete/commit。
```

要求審查並提交:

```text
$svn-precommit-review 檢查 <SVN_WORKING_COPY> 的變更並提交。
```

提出提交要求後，確認包含與排除的範圍、已準備的 add/delete、剩餘風險及訊息，再對最終問題回覆一次。「提交吧」或 `커밋해줘` 即可，不必使用 `CLI` 字樣。

```text
네, 진행해
```

### 行為

- 檢查 `svn status --xml`、遠端狀態、內容 diff 與 property diff。
- 檢查產生檔、IDE/使用者檔案、日誌、本機設定、敏感資訊、衝突、遺失檔案與測試不足。
- 預設審查中，若已追蹤檔案的參照或已審查的變更證明未追蹤的原始檔、專案檔、資源及非本機專用設定確屬必要，就必須自動執行 `svn add`，不能只給建議。
- 明確要求唯讀、僅審查、僅產生訊息或不做變更時，不執行任何寫入。一般的「檢查一下」包含預設準備作業。
- 只有 add 命令成功且 `svn status` 確認狀態為 `A`，才標記新增完成。已完成的新增記錄在變更檔案摘要中，不再列為未解決警告。
- 即使仍有程式碼風險，也可繼續獨立且安全的 add。仍須回報該風險，不得聲稱 add 修復了程式碼。
- 不會 add `bin`、`obj`、產生檔、IDE/使用者檔案、日誌、暫存檔、快取、本機設定、壓縮檔、傾印、備份、螢幕截圖、被忽略的檔案、`.env`、疑似含密鑰的檔案或不明確檔案。新增目錄前遞迴檢查全部內容；僅當全部安全時才新增整個目錄，否則只新增安全的明確路徑。
- 使用者要求包含已檢查的刪除候選項時，只會為乾淨的 versioned 本機路徑安排刪除。
- 不會刪除本機修改、衝突、switched、external、lock 或意圖不明確的路徑。
- 實際提交需要提交要求及對確切範圍與訊息的最終核准。同一範圍與內容的先前核准持續有效；提交前重新檢查 status、內容及 property diff。只有範圍或內容有實質變更，或阻擋狀態改變時，才重新審查並要求確認。
- 明確選擇 TortoiseSVN 手動提交時，僅提供審查與檢查清單，不執行 CLI 提交，也不詢問是否執行 CLI 提交。除非要求唯讀，否則仍進行預設的安全準備。
- 不使用寬泛的 `svn add`、`svn delete` 或 `svn commit`。

執行期間的說明、審查、提交訊息與結果一律使用韓語。摘要依變更意圖分組，不限定項目數，也不要求冗長說明。只有存在真實證據且審查內容未變時，才可沿用先前測試，並明確標註為先前結果、本次未重新執行。

### 韓語提交訊息

CLI 提交在工作副本外建立唯一的暫存訊息檔，使用無 BOM 的 UTF-8 與 LF 換行，並透過 `--file` 和 `--encoding UTF-8` 提交:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

每次嘗試後都會刪除暫存檔並確認其已不存在。條件允許時，會逐字比對從提交命令輸出中確認的實際修訂的 XML 日誌訊息與原文。若儲存的韓語訊息損壞，技能會回報編碼驗證失敗，不會重試提交或修改 revision property。
