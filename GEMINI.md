# GEMINI.md

## 目的
本ドキュメントは、Gemini CLI 向けのコンテキストと作業方針を定義する。

## 出力スタイル
- **言語**: 日本語
- **トーン**: プロフェッショナルかつ簡潔
- **形式**: Markdown

## 共通ルール
- 会話と言語: 日本語。
- コミット規約: Conventional Commits（説明は日本語）。
- 文字間隔: 日本語と英数字の間に半角スペース。

## プロジェクト概要
- **目的**: Docker Compose プロジェクトのイメージ自動更新ツール。
- **主な機能**:
    - `latest` タグイメージのプルと更新検知。
    - バージョン情報の比較（Label `org.opencontainers.image.version` または作成日時を使用）。
    - Discord への通知。
    - 指定プロジェクトの自動再起動。

## コーディング規約
- **言語**: Python
- **コメント**: 日本語
- **エラーメッセージ**: 英語
- **フォーマット**: PEP 8 準拠
- **Docstring**: 日本語で記載

## 開発コマンド
```bash
# インストール
pip install -r requirements.txt

# 単一プロジェクトの実行
./wrapper.sh /path/to/project

# 一括更新実行
./update-all.sh /path/to/projects /path/to/logs
```

## 注意事項
- **セキュリティ**: Webhook URL (`discord_webhook_url.txt`) をコミットしないこと。
- **既存優先**: 既存の `subprocess` による Docker 操作パターンを優先する。
- **制約**: `latest` タグ以外の更新には現在対応していない。

## リポジトリ固有
- `restartable_projects.json` の管理には `generate-restartables.sh` を使用する。
- Discord 通知を有効にするには、環境変数 `CIU_DISCORD_WEBHOOK_URL` または `discord_webhook_url.txt` が必要。
