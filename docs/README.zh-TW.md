# svn-precommit-review 使用指南

`svn-precommit-review` 是在 SVN/TortoiseSVN 提交前檢查工作副本並產生韓語提交訊息的 Codex Agent Skill。預設為唯讀。使用者明確要求 Codex CLI 提交時，技能可以安全地 add 必要的新原始檔，顯示最終範圍後只需一次確認即可提交已核准的範圍。

## 安裝

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

或:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

請將此儲存庫的 `svn-precommit-review/SKILL.md` 複製到目標位置。

## 使用方式

唯讀檢查:

```text
$svn-precommit-review 在提交前檢查目前 SVN 變更，並產生韓語提交訊息。
```

要求 CLI 提交:

```text
$svn-precommit-review 檢查 <SVN_WORKING_COPY> 的變更，add 必要的新原始檔，最終確認後只用核准範圍執行 CLI 提交。
```

技能顯示範圍與訊息後，只要回覆一次明確確認即可。

```text
네, 진행해
```

## 行為

- 檢查 `svn status --xml`、遠端狀態、內容 diff 與 property diff。
- 檢查產生檔、IDE/使用者檔案、日誌、本機設定、敏感資訊、衝突、遺失檔案與測試不足。
- 要求 CLI 提交時，可對意圖明確的新原始檔、專案檔、資源與必要設定執行 `svn add`。
- 不會 add `bin`、`obj`、日誌、快取、壓縮檔、`.env`、疑似含密鑰的檔案、產生檔或不明確檔案。
- 使用者要求包含已檢查的刪除候選項時，只會為乾淨的 versioned 本機路徑安排刪除。
- 不會刪除本機修改、衝突、switched、external、lock 或意圖不明確的路徑。
- 提交前會重新檢查狀態與目標 diff；只有核准範圍、有效內容或阻擋 SVN 狀態變更時才停止並重新檢查。
- 不使用寬泛的 `svn add`、`svn delete` 或 `svn commit`。

## 韓語提交訊息

CLI 提交使用無 BOM 的 UTF-8 暫存訊息檔與 `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

每次嘗試後都會刪除暫存檔。條件允許時，會比對實際提交修訂的 XML 日誌訊息。若儲存的韓語訊息損壞，技能會回報編碼驗證失敗，不會重試提交或修改 revision property。
