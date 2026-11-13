# precommit-review-skills 使用指南

[precommit-review-skills](https://github.com/cku3987/precommit-review-skills) 是用于 Git 和 SVN 提交前审查的独立 Codex Agent Skill 集合。版本发布以仓库为单位统一进行，各技能独立安装和调用。现有 `svn-precommit-review` 名称及其 `SKILL.md` 保持不变，请勿重命名已安装的 SVN 技能。

- [SVN 技能](../svn-precommit-review/SKILL.md)
- [Git 技能](../git-precommit-review/SKILL.md)
- [Git 详细指南](git-precommit-review-guide.md)

## Git

`git-precommit-review` 使用韩语撰写审查报告和提交消息。

- 默认调用会审查变更，并自动暂存已确认必需且安全的新文件。忽略文件、生成文件及含有秘密信息的文件均被排除。
- 明确要求只读时，不执行任何写入。
- 请求提交时，会准备已审查的已跟踪文件变更，同时保留现有的部分暂存状态。限定范围的提交不会包含无关的已暂存变更。
- 实际提交需对范围和消息进行一次最终确认。`push` 仅在另行请求时执行。

默认调用、韩语提交请求（`커밋해줘`）及明确只读的示例：

```text
$git-precommit-review
$git-precommit-review 변경사항을 검토하고 커밋해줘.
$git-precommit-review 仅以只读方式检查当前 Git 变更，不执行任何写入。
```

## SVN

`svn-precommit-review` 是一个 Codex Agent Skill，用于在 SVN/TortoiseSVN 提交前检查工作副本并生成韩语提交消息。默认调用会执行审查，并自动对符合安全条件的必需新文件执行 `svn add`。实际提交需要用户提出提交请求，并对范围和消息进行一次最终确认，只提交已批准的范围。

### 安装

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

或:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

将此仓库中的 `svn-precommit-review/SKILL.md` 复制到目标位置。

### 使用

默认审查并准备必需文件（普通的“检查一下”或 `검토해줘` 也相同）:

```text
$svn-precommit-review
```

只读检查:

```text
$svn-precommit-review 仅以只读方式检查当前 SVN 变更。不要执行 svn add/delete/commit。
```

请求审查并提交:

```text
$svn-precommit-review 检查 <SVN_WORKING_COPY> 的变更并提交。
```

提出提交请求后，确认包含和排除的范围、已准备的 add/delete、剩余风险及消息，再对最终问题回复一次。“提交吧”或 `커밋해줘` 即可，无须使用 `CLI` 字样。

```text
네, 진행해
```

### 行为

- 检查 `svn status --xml`、远程状态、内容 diff 和 property diff。
- 检查生成文件、IDE/用户文件、日志、本地配置、敏感信息、冲突、缺失文件和测试不足。
- 默认审查中，若已跟踪文件的引用或已审查的变更证明未跟踪的源文件、项目文件、资源及非本地专用配置确属必需，就必须自动执行 `svn add`，不能仅给出建议。
- 明确要求只读、仅审查、仅生成消息或不做更改时，不执行任何写入。普通的“检查一下”包含默认准备操作。
- 只有 add 命令成功且 `svn status` 确认状态为 `A`，才标记添加完成。已完成的添加记录在变更文件摘要中，不再列为未解决警告。
- 即使仍有代码风险，也可继续独立且安全的 add。仍须报告该风险，不得声称 add 修复了代码。
- 不会 add `bin`、`obj`、生成文件、IDE/用户文件、日志、临时文件、缓存、本地配置、压缩包、转储、备份、截图、被忽略的文件、`.env`、疑似含密钥的文件或不明确的文件。添加目录前递归检查全部内容；仅当全部安全时才添加整个目录，否则只添加安全的具体路径。
- 用户要求包含已检查的删除候选项时，只会为干净的 versioned 本地路径安排删除。
- 不会删除本地修改、冲突、switched、external、lock 或意图不明确的路径。
- 实际提交需要提交请求及对确切范围和消息的最终批准。同一范围和内容的先前批准继续有效；提交前重新检查 status、内容及 property diff。仅当范围或内容有实质变化，或阻塞状态改变时，才重新审查并请求确认。
- 明确选择 TortoiseSVN 手动提交时，仅提供审查与检查清单，不执行 CLI 提交，也不询问是否执行 CLI 提交。除非要求只读，否则仍进行默认的安全准备。
- 不使用宽泛的 `svn add`、`svn delete` 或 `svn commit`。

运行期间的说明、审查、提交消息和结果一律使用韩语。摘要按变更意图分组，不限定条目数，也不要求冗长说明。只有存在真实证据且审查内容未变时，才可复用先前测试，并明确标注为先前结果、本次未重新运行。

### 韩语提交消息

CLI 提交在工作副本外创建唯一的临时消息文件，使用无 BOM 的 UTF-8 和 LF 换行，并通过 `--file` 和 `--encoding UTF-8` 提交:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

每次尝试后都会删除临时文件并确认其已不存在。条件允许时，会逐字比较从提交命令输出中确定的实际修订的 XML 日志消息与原文。如果存储的韩语消息损坏，技能会报告编码验证失败，不会重试提交或修改 revision property。
