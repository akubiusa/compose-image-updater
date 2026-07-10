# GitHub Copilot コードレビュー向け指示

このファイルは GitHub Copilot のコードレビュー機能向けの指示です。以下の観点でレビューしてください。

## プロジェクト背景
Python 製の CLI ツールで、Docker Compose プロジェクトの `latest` タグイメージを自動更新し、コンテナ再起動と Discord 通知を行う。外部依存は `requests` のみ。自動テストは未導入。

## 重点的に確認する点
- **エラーハンドリング**: `subprocess.run` の `returncode` を確認しているか。`docker compose` コマンドの失敗が無視されていないか。外部コマンド・ネットワーク (Discord 通知) の失敗時に処理が破綻しないか。
- **機密情報**: Discord Webhook URL (`CIU_DISCORD_WEBHOOK_URL` / `discord_webhook_url.txt`) や認証情報が、コード・ログ・コミットに混入していないか。`.gitignore` から除外対象が漏れていないか。
- **シェルスクリプト**: `wrapper.sh` / `update-all.sh` / `generate-restartables.sh` で、引数チェック・パスのクォート・`set -e` 相当のエラー処理が適切か。ShellCheck 相当の指摘 (未クォート変数など) に注意。
- **回帰リスク**: `latest` タグのみを対象とする前提、バージョン比較 (`org.opencontainers.image.version` → `Created` フォールバック)、`restartable_projects.json` による再起動判定のロジックを壊していないか。
- **テスト不在の補完**: 自動テストが無いため、ロジック変更は手動検証手順が PR 説明に書かれているかを確認する。

## コーディング規約 (レビュー時の目安)
- Python は PEP 8 準拠、命名はスネークケース。リンターによる自動強制は無いため、規約は目安として指摘する。
- コメント・docstring は日本語、エラーメッセージ (print 出力等) は英語。
- コミットメッセージは Conventional Commits (説明は日本語)。

## フラグすべきでない既知パターン
- `subprocess.run([...])` をリスト引数 (`shell=False`) で呼ぶのは意図的で安全。シェルインジェクションの誤検知をしない。
- 外部ライブラリを増やさず標準ライブラリ + `requests` で完結させる方針。「より高機能なライブラリの導入」提案は不要。
- 日本語コメントは仕様。英語化の提案は不要。
