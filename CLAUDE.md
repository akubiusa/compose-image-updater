# CLAUDE.md

## プロジェクト概要
- **目的**: Docker Compose プロジェクトの `latest` タグイメージを自動更新し、必要に応じてコンテナ再起動と Discord 通知を行う。
- **主な機能**:
    - `docker compose pull` によるイメージ更新と、更新前後のバージョン比較。
    - `restartable_projects.json` に登録されたプロジェクトの自動再起動。
    - Discord Webhook による更新結果・エラー検知の通知。

## 重要ルール
- **会話**: 日本語で対話する。
- **コミットメッセージ**: Conventional Commits に従い、説明は日本語で記載する。
- **ブランチ命名**: Conventional Branch (`feat/`, `fix/` など) に従う。
- **コード内コメント / docstring**: 日本語で記載する。
- **エラーメッセージ**: 英語で記載する。
- **日本語と英数字の間**: 半角スペースを挿入する。
- **Renovate**: Renovate が作成した PR には直接コミットしない。必要なら新規 PR を作成する。

## 判断記録のルール
判断を行う際は、次を明示する。前提・仮定を事実として扱わず、不確実な点は明記すること。
1. 判断内容の要約。
2. 検討した代替案と、採用しなかった案とその理由。
3. 前提条件・仮定・不確実性。

## 開発コマンド
```bash
# 依存関係のインストール
pip install -r requirements.txt

# 単一プロジェクトの更新 (venv を自動作成して実行)
./wrapper.sh /path/to/docker-compose-project/

# 指定ディレクトリ配下の全プロジェクトを一括更新 (cron 想定)
./update-all.sh /path/to/projects/ /path/to/logs/

# 再起動対象リスト (restartable_projects.json) の生成
./generate-restartables.sh /path/to/projects/
```

## アーキテクチャと主要ファイル
- **`src/__main__.py`**: エントリポイント。更新チェック → 再起動 → 通知のフローを制御。
- **`src/__init__.py`**: `docker compose` 操作、イメージバージョン取得、Discord 通知の共通ユーティリティ。
- **`wrapper.sh`**: venv セットアップと `python3 -m src` 実行を行うラッパー。Systemd の `ExecStartPre` からも利用される。
- **`update-all.sh`**: `compose.yml` / `compose.yaml` / `docker-compose.yml` を探索し、各プロジェクトに対し `wrapper.sh` を実行してログを保存する。
- **`generate-restartables.sh`**: 対象ディレクトリを走査し `restartable_projects.json` を生成する。

## 実装パターン
- **推奨**: Docker 操作は `subprocess` を通じて `docker compose` コマンドを呼び出す (既存パターンを踏襲)。
- **非推奨**: 外部ライブラリへの過度な依存 (現在の依存は `requests` のみ)。

## リポジトリ固有の仕様・制約
- Docker および Docker Compose が利用可能な環境を前提とする。
- 更新対象は **`latest` タグが指定されたイメージのみ**。それ以外のタグの更新には対応していない。
- イメージのバージョン比較は Label `org.opencontainers.image.version` を優先し、無ければ `Created` (作成日時) を使用する。
- 再起動を有効にするには、プロジェクト名を `restartable_projects.json` に登録する必要がある (`generate-restartables.sh` で生成)。

## セキュリティ / 機密情報
- Discord Webhook URL は環境変数 `CIU_DISCORD_WEBHOOK_URL`、または `wrapper.sh` と同じディレクトリ (通常はリポジトリルート) に置く `discord_webhook_url.txt` で指定する (`wrapper.sh` は自身のディレクトリへ `cd` してから読み込む)。**どちらも Git にコミットしない** (`discord_webhook_url.txt` は `.gitignore` 済み)。
- API キー・パスワード等の機密情報をコードやログに出力しない。

## テスト
- 自動テストは未導入。変更時は手動での動作確認を必須とする。

## ドキュメント更新
- 外部インターフェース (引数・設定ファイル形式・環境変数) を変更した場合は `README.md` と本ファイルを更新する。
