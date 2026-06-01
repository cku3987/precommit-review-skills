# svn-precommit-review 指南

`svn-precommit-review` 是一個 Codex Agent Skill，用於在 SVN/TortoiseSVN 提交前檢查 working copy 變更，並產生韓文提交訊息草稿。

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

## 重點

- 檢查 `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- 偵測建置輸出、IDE 檔案、log、暫存檔、本機設定
- 偵測 `.env`, token, password, API Key 等敏感資訊風險
- 檢查衝突標記與 missing/deleted 檔案
- 只有在使用者明確要求並最終確認後，才允許受保護的 CLI commit

## 韓文提交訊息編碼

避免在 PowerShell 使用:

```powershell
svn commit -m "한글 커밋 메시지"
```

CLI commit 請使用 UTF-8 訊息檔並明確指定 `--encoding UTF-8`。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

commit 後用 XML 輸出驗證儲存庫中的 log 訊息。

```powershell
svn log --xml --limit 1
```

## 更多資訊

預設英文文件: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
