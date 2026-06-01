# svn-precommit-review 指南

`svn-precommit-review` 是一个 Codex Agent Skill，用于在 SVN/TortoiseSVN 提交前检查 working copy 变更，并生成韩语提交信息草稿。

## 安装

用户级:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

项目级:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

将本仓库中的 `svn-precommit-review/SKILL.md` 复制到所需位置即可。

## 用法

```text
$svn-precommit-review 请在提交前检查当前 SVN 变更，并生成韩语提交信息。
```

指定 working copy:

```text
$svn-precommit-review 请检查 <SVN_WORKING_COPY> 路径下的 svn status 和 svn diff，并给出提交建议。
```

## 检查内容

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- 构建产物、IDE 文件、日志、临时文件、本地配置
- `.env`, token, password, API Key 等敏感信息风险
- 冲突标记以及 missing/deleted 文件
- 提交可行性判断: `가능`, `조건부 가능`, `보류 권장`
- 只有在用户明确请求并最终确认后，才允许受保护的 CLI 提交

## 韩语提交信息编码

请避免在 PowerShell 中使用:

```powershell
svn commit -m "한글 커밋 메시지"
```

如果必须使用 CLI 提交，请使用 UTF-8 消息文件并显式指定 `--encoding UTF-8`。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

提交后用 XML 输出验证仓库中保存的日志信息。

```powershell
svn log --xml --limit 1
```

## 更多信息

默认英文文档: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
