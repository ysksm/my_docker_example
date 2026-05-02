# ToolJet (Docker Compose)

x86_64 Linux 向けの ToolJet (Community Edition) ローカル実行構成。

## 構成

| サービス | イメージ | 役割 |
|---|---|---|
| `tooljet` | `tooljet/tooljet:latest` | ToolJet 本体 (web + api) |
| `postgres` | `postgres:15` | アプリ DB / ToolJet DB |
| `redis` | `redis:7-alpine` | キャッシュ・キュー |

データは `./data/` 配下に bind mount で永続化されます（`gitignore` 済み）。

## 前提条件

- x86_64 Linux ホスト
- Docker Engine 24+ / Docker Compose v2

> Apple Silicon (arm64) Mac は `tooljet/tooljet:latest` が arm64 ネイティブを提供しておらず Rosetta エミュレーション必須・将来の Rosetta 廃止 (macOS 28) で動作不可になるため非推奨。

## セットアップ

```bash
# 1. 環境変数ファイルを作成
cp .env.example .env

# 2. シークレットを生成して .env を編集
openssl rand -hex 32   # → LOCKBOX_MASTER_KEY に貼る
openssl rand -hex 64   # → SECRET_KEY_BASE に貼る

# 3. 必要なら DB パスワードや TOOLJET_HOST_PORT も変更
```

`.env` の主な項目:

| 変数 | 用途 | デフォルト |
|---|---|---|
| `TOOLJET_HOST_PORT` | ホスト側公開ポート | `8080` |
| `LOCKBOX_MASTER_KEY` | 暗号化マスターキー (必須) | プレースホルダ |
| `SECRET_KEY_BASE` | セッション署名鍵 (必須) | プレースホルダ |
| `PG_PASS` | PostgreSQL パスワード | `tooljet_password` |

## 起動

```bash
docker compose up -d
docker compose logs -f tooljet
```

ブラウザで `http://localhost:8080` (または `.env` の `TOOLJET_HOST_PORT`) にアクセス。

## 停止 / 再起動

```bash
docker compose stop          # 停止
docker compose up -d         # 再起動
docker compose down          # 停止 + コンテナ削除（データは ./data/ に残る）
```

## データの完全削除

```bash
docker compose down
rm -rf data/
```

## アップデート

```bash
docker compose pull
docker compose up -d
```

## トラブルシュート

| 症状 | 確認 |
|---|---|
| `port is already allocated` | 別プロセスが `TOOLJET_HOST_PORT` を使用。`lsof -nP -iTCP:8080 -sTCP:LISTEN` で特定 |
| `wait-for-it.sh` で停止 | `docker compose ps` で依存サービスが `healthy` か確認 |
| 認証エラー | `./data/postgres` が古い credentials で初期化済み。`docker compose down && rm -rf data/postgres` |

ログ:
```bash
docker compose logs --tail=100 tooljet
docker compose logs --tail=50 postgres
docker compose logs --tail=50 redis
```
