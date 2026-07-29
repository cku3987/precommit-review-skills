# svn-precommit-review 使用指南

`svn-precommit-review` 是一个 Codex Agent Skill，用于在 SVN/TortoiseSVN 提交前检查工作副本并生成韩语提交消息。默认只读。用户明确请求 Codex CLI 提交时，技能可以安全地 add 必需的新源文件，在显示最终范围后只需一次确认即可提交已批准的范围。

## 安装

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

或:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

将此仓库中的 `svn-precommit-review/SKILL.md` 复制到目标位置。

## 使用

只读检查:

```text
$svn-precommit-review 在提交前检查当前 SVN 变更，并生成韩语提交消息。
```

请求 CLI 提交:

```text
$svn-precommit-review 检查 <SVN_WORKING_COPY> 的变更，add 必需的新源文件，最终确认后只用批准范围执行 CLI 提交。
```

技能显示范围和消息后，只需回复一次明确的确认。

```text
네, 진행해
```

## 行为

- 检查 `svn status --xml`、远程状态、内容 diff 和 property diff。
- 检查生成文件、IDE/用户文件、日志、本地配置、敏感信息、冲突、缺失文件和测试不足。
- 请求 CLI 提交时，可对意图明确的新源文件、项目文件、资源和必需配置执行 `svn add`。
- 不会 add `bin`、`obj`、日志、缓存、压缩包、`.env`、疑似含密钥的文件、生成文件或不明确的文件。
- 用户要求包含已检查的删除候选项时，只会为干净的 versioned 本地路径安排删除。
- 不会删除本地修改、冲突、switched、external、lock 或意图不明确的路径。
- 提交前会重新检查状态和目标 diff；仅当批准范围、有效内容或阻塞 SVN 状态发生变化时才停止并重新检查。
- 不使用宽泛的 `svn add`、`svn delete` 或 `svn commit`。

## 韩语提交消息

CLI 提交使用无 BOM 的 UTF-8 临时消息文件和 `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

每次尝试后都会删除临时文件。条件允许时，会比较实际提交修订的 XML 日志消息。如果存储的韩语消息损坏，技能会报告编码验证失败，不会重试提交或修改 revision property。
