# svn-precommit-review ガイド

`svn-precommit-review` は、SVN/TortoiseSVN のコミット直前に working copy の変更を確認し、韓国語のコミットメッセージ案を作成する Codex Agent Skill です。通常のレビューは読み取り専用ですが、ユーザーがコミット準備または auto-add を明示した場合のみ、ビルドを壊す可能性がある必須の新規ファイルを安全条件に従って `svn add` できます。レビュー後にユーザーが明示パスの削除を再依頼した場合のみ、安全条件を満たす versioned ファイルを `svn delete` で削除予定にできます。

## インストール

ユーザー単位:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

プロジェクト単位:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

このリポジトリの `svn-precommit-review/SKILL.md` を必要な場所にコピーしてください。

## 使い方

```text
$svn-precommit-review 現在の SVN 変更内容をコミット前にレビューし、韓国語のコミットメッセージを作成して。
```

特定の working copy を指定する場合:

```text
$svn-precommit-review <SVN_WORKING_COPY> の svn status と svn diff を確認して、コミット可否と推奨コミットメッセージを作成して。
```

必須ファイルの safe add:

```text
$svn-precommit-review <SVN_WORKING_COPY> の変更を確認し、ビルドに必要な新規ファイルは安全条件を満たす場合だけ svn add して。
```

## 主な確認項目

- `svn status --xml`、`svn status -u --xml`、内容 diff、property diff を構造的に確認
- ビルド成果物、IDE ファイル、ログ、一時ファイル、ローカル設定
- `.env`、トークン、パスワード、API Key などの機密情報を確認し、実際の値は常に `[REDACTED]` でマスク
- text/property/tree conflict と replaced、obstructed、switched、external、nested working copy、lock 状態を確認
- コミット可否: `가능`, `조건부 가능`, `보류 권장`
- 明示的な依頼がある場合のみ、ビルドに必要な新規ファイルを安全条件付きで `svn add`
- レビュー後に再依頼された、ローカル変更のない明示的なローカルパスのみ `svn delete`。変更済みファイルとリポジトリ URL は自動削除しない
- 承認前とコミット直前に status、diff、property、対象、ファイル内容の SHA-256 snapshot を比較
- 通常はファイル単位でコミットし、ディレクトリは全子要素と SVN 境界を確認した場合のみ許可
- 空の出力カテゴリは `- 없음` と簡潔に表示
- TortoiseSVN の手動コミットモードでは Codex は `svn commit` を実行せず、CLI 実行承認も求めない
- Codex CLI コミットは、ユーザーが事前に CLI コミット実行を明示的に依頼し、対象とメッセージを確認した後、直接の実行確認に明確に同意した場合のみ実行

CLI モードがすでに明示的に依頼され、Codex が直接の実行確認を行った直後であれば、`はい`、`yes`、`進めて` などの明確な肯定で CLI コミットを承認できます。レビューのみの依頼では、その確認を行わずに終了します。

## 韓国語コミットメッセージのエンコーディング

PowerShell で次の形式は避けてください。

```powershell
svn commit -m "한글 커밋 메시지"
```

CLI コミットでは、BOM なし UTF-8、LF 改行のメッセージファイルと `--encoding UTF-8` を使います。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

コミット結果からリビジョン番号を取得し、そのリビジョンだけを確認します。

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

XML の `<msg>` を元のメッセージと完全一致で比較します。同時コミットがあり得るため、`--limit 1` は検証に使いません。

## 詳細

英語の基本ドキュメント: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
