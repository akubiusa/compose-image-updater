# GitHub Copilot Instructions

## プロジェクト概要
- **目的**: `docker-compose.yml` で使用されている `latest` タグのイメージを自動的に更新する
- **主な機能**:
    - `docker compose pull` によるイメージの更新確認
    - `restartable_projects.json` に基づくコンテナの再起動
    - Discord Webhook による更新通知（イメージのバージョン変更、エラーログの確認）
- **対象ユーザー**: Docker Compose を使用してサービスを運用している開発者・運用者

## 共通ルール
- 会話は日本語で行う。
- コミットメッセージは [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従う。
    - `<type>(<scope>): <description>` 形式
    - `<description>` は日本語で記載
- ブランチ命名は [Conventional Branch](https://conventional-branch.github.io) に従う。
    - `<type>/<description>` 形式（`feat/`, `fix/` など）
- 日本語と英数字の間には半角スペースを入れる。

## 技術スタック
- **言語**: Python (3.6+)
- **依存関係管理**: `pip` (`requirements.txt`)
- **外部ツール**: Docker, Docker Compose

## 開発コマンド
```bash
# 依存関係のインストール
pip install -r requirements.txt

# スクリプトの実行 (venv 環境を自動作成して実行)
./wrapper.sh /path/to/docker-compose-project/

# 全プロジェクトの一括更新
./update-all.sh /path/to/projects/ /path/to/logs/

# 再起動対象プロジェクトリストの生成
./generate-restartables.sh /path/to/projects/
```

## コーディング規約
- **フォーマット**: PEP 8 に準拠。
- **命名規則**: スネークケース（`update_images`, `image_name` など）。
- **コメント**: 日本語で記載。
- **エラーメッセージ**: 英語で記載。
- **Docstring**: 関数には日本語で docstring を記載する。

## テスト方針
- 現時点では自動テストは導入されていない。
- 変更時は手動での動作確認を徹底する。

## セキュリティ / 機密情報
- `discord_webhook_url.txt` や環境変数 `CIU_DISCORD_WEBHOOK_URL` に含まれる Webhook URL は Git にコミットしない。
- ログに個人情報や認証情報を出力しない。

## ドキュメント更新
- 重要な変更があった場合は `README.md` を更新する。
- 外部インターフェース（引数や設定ファイル形式）の変更時は必ずドキュメントに反映する。

## リポジトリ固有
- `latest` タグが指定されているイメージのみが更新対象となる。
- 再起動を有効にするには `restartable_projects.json` にプロジェクト名を記載する必要がある。
