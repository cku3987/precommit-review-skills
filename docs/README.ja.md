# svn-precommit-review ガイド

`svn-precommit-review` は、SVN/TortoiseSVN のコミット直前に変更を確認し、韓国語のコミットメッセージを作成する Codex Agent Skill です。標準では読み取り専用です。Codex CLI によるコミットを明示的に依頼した場合は、必要な新規ソースファイルを安全に add し、最終確認を一度受けてから承認範囲だけをコミットできます。

## インストール

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

または:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

このリポジトリの `svn-precommit-review/SKILL.md` をコピーしてください。

## 使い方

読み取り専用の確認:

```text
$svn-precommit-review 現在の SVN 変更をコミット前に確認し、韓国語のコミットメッセージを作成して。
```

CLI コミットを依頼:

```text
$svn-precommit-review <SVN_WORKING_COPY> の変更を確認し、必要な新規ソースは add して、最終確認後に承認範囲だけ CLI でコミットして。
```

表示された範囲とメッセージを確認した後、一度だけ明確に返答します。

```text
네, 진행해
```

## 動作

- `svn status --xml`、リモート状態、内容 diff、property diff を確認します。
- 生成物、IDE/ユーザー固有ファイル、ログ、ローカル設定、秘密情報、競合、欠落ファイル、テスト不足を確認します。
- CLI コミットの依頼時は、変更意図が明確な新規ソース、プロジェクト、リソース、必要な設定を `svn add` できます。
- `bin`、`obj`、ログ、キャッシュ、アーカイブ、`.env`、秘密情報の可能性があるファイル、生成物、曖昧なファイルは add しません。
- 削除候補を含めるよう依頼された場合、確認済みのクリーンな versioned ローカルパスだけを削除予約できます。
- 競合、ローカル修正、switched、external、lock、意図不明のパスは削除予約しません。
- コミット直前に状態と対象 diff を再確認し、承認範囲・意味のある内容・ブロック状態が変わった場合だけ停止して再確認します。
- 広範囲の `svn add`、`svn delete`、`svn commit` は使用しません。

## 韓国語コミットメッセージ

CLI コミットでは BOM なし UTF-8 の一時メッセージファイルと `--encoding UTF-8` を使用します。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

一時ファイルは試行後に削除します。可能なら実際にコミットしたリビジョンの XML ログとメッセージを比較します。保存済みメッセージの韓国語が壊れていた場合、再コミットや revision property の変更は行わず、検証失敗として報告します。
