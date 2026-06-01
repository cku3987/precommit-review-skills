# svn-precommit-review ガイド

`svn-precommit-review` は、SVN/TortoiseSVN のコミット直前に working copy の変更を確認し、韓国語のコミットメッセージ案を作成する Codex Agent Skill です。

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

## 主な確認項目

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- ビルド成果物、IDE ファイル、ログ、一時ファイル、ローカル設定
- `.env`, トークン, パスワード, API Key などの機密情報の可能性
- コンフリクトマーカーと missing/deleted ファイル
- コミット可否: `가능`, `조건부 가능`, `보류 권장`
- 明示的な依頼と最終承認がある場合のみ、ガード付き CLI コミット

## 韓国語コミットメッセージのエンコーディング

PowerShell で次の形式は避けてください。

```powershell
svn commit -m "한글 커밋 메시지"
```

CLI コミットが必要な場合は、UTF-8 メッセージファイルと `--encoding UTF-8` を一緒に使います。

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

コミット後は XML 出力で保存されたログメッセージを確認します。

```powershell
svn log --xml --limit 1
```

## 詳細

英語の基本ドキュメント: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
