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

- 结构化检查 `svn status --xml`、`svn status -u --xml`、内容 diff 和 property diff
- 构建产物、IDE 文件、日志、临时文件、本地配置
- 检查 `.env`、token、password、API Key 等敏感信息，实际值始终以 `[REDACTED]` 屏蔽
- 检查 text/property/tree conflict，以及 replaced、obstructed、switched、external、nested working copy 和 lock 状态
- 提交可行性判断: `가능`, `조건부 가능`, `보류 권장`
- 仅在明确请求时，对通过安全条件的构建必要新文件执行 `svn add`
- 仅对检查后再次明确请求且没有本地修改的具体本地路径执行 `svn delete`；不自动删除本地已修改文件或仓库 URL
- 在批准前和提交前比较 status、diff、property、目标和文件内容的 SHA-256 snapshot
- 默认按文件提交；只有检查全部后代和 SVN 边界后才允许目录目标
- 空输出分类简洁写为 `- 없음`
- 在 TortoiseSVN 手动提交模式下，Codex 不会执行 `svn commit`，也不会询问 CLI 执行授权
- 只有在用户事先明确要求 Codex 执行 CLI 提交、检查目标和消息后，对直接执行问题作出明确肯定答复，才会执行受保护的 CLI 提交

只有当 CLI 模式已被明确请求且 Codex 刚刚提出直接执行问题时，`是`、`yes`、`继续` 等简短明确的肯定答复才会授权 CLI 提交。仅检查请求会在不提出该问题的情况下结束。

## 韩语提交信息编码

请避免在 PowerShell 中使用:

```powershell
svn commit -m "한글 커밋 메시지"
```

如果必须使用 CLI 提交，请使用无 BOM、LF 换行的 UTF-8 消息文件并显式指定 `--encoding UTF-8`。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

从提交结果读取修订号，并且只验证该修订。

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

将 XML `<msg>` 与原始消息精确比较。由于可能存在并发提交，不使用 `--limit 1` 作为验证依据。

## 更多信息

默认英文文档: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
