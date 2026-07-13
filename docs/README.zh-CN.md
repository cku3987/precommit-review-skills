# svn-precommit-review 指南

`svn-precommit-review` 是一个 Codex Agent Skill，用于在 SVN/TortoiseSVN 提交前检查 working copy 变更，并生成韩语提交信息草稿。默认检查是只读的；只有在用户明确要求提交准备或自动 add 时，才会按安全条件对可能导致构建/运行失败的必要新文件执行 `svn add`。在检查后，只有用户再次明确请求删除具体路径时，才会按安全条件对已版本控制文件执行 `svn delete`。

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

安全 add 必要新文件:

```text
$svn-precommit-review 请检查 <SVN_WORKING_COPY> 的变更；如果必要新文件会影响构建，并且通过安全条件，请执行 svn add。
```

## 检查内容

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- 构建产物、IDE 文件、日志、临时文件、本地配置
- `.env`, token, password, API Key 等敏感信息风险
- 冲突标记以及 missing/deleted 文件
- 提交可行性判断: `가능`, `조건부 가능`, `보류 권장`
- 仅在明确请求时，对通过安全条件的构建必要新文件执行 `svn add`
- 仅对检查后再次明确请求的具体路径执行安全 `svn delete`
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
