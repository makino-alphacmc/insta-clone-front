# Step 8: 本番環境へのデプロイ準備

## 📋 このステップでやること

本番環境で動作するように設定を調整し、デプロイの準備をします。

## ✅ 手順

### 8-1) 本番環境の選択
※ 目的: デプロイ先の前提を固め、以降の設定（ポート/ドメイン/SSL）を決めやすくする。

以下のいずれかを選択します：

- **VPS**（例: DigitalOcean, Linode, ConoHa）
- **クラウドサービス**（例: AWS, GCP, Azure）
- **自宅サーバー**
- **その他のサーバー**

**必要なもの**:
- Docker と Docker Compose がインストールされている
- ドメイン名（オプション、なくてもIPアドレスでアクセス可能）

### 8-2) 本番用の環境変数設定
※ 目的: 本番固有のキーやオリジンを分離し、セキュアに設定する。

#### Backend（insta-clone-api/.env.production）

本番環境用の `.env.production` を作成します：

```env
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=your_production_anon_key
SUPABASE_BUCKET=post-images
ALLOWED_ORIGINS=https://your-domain.com,http://your-ip-address:3000
```

**注意**: 
- `ALLOWED_ORIGINS` には、本番環境のフロントエンドURLを指定
- 複数のオリジンを指定する場合はカンマ区切り

#### Frontend（insta-clone-front/.env.production）

本番環境用の `.env.production` を作成します：

```env
NUXT_PUBLIC_API_BASE=https://your-api-domain.com
# または
NUXT_PUBLIC_API_BASE=http://your-ip-address:8000
```

### 8-3) docker-compose.prod.yml の作成
※ 目的: 本番用リソース制限やenvを明示し、デプロイをワンコマンド化する。

本番環境用の `docker-compose.prod.yml` を作成します：

```yaml
version: '3.8'

services:
  api:
    build:
      context: ./insta-clone-api
      dockerfile: Dockerfile
    container_name: insta-clone-api-prod
    ports:
      - "8000:8000"
    env_file:
      - ./insta-clone-api/.env.production
    volumes:
      - ./insta-clone-api/app.db:/app/app.db
      # ログを永続化する場合
      - ./insta-clone-api/logs:/app/logs
    restart: always
    # リソース制限（オプション）
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  front:
    build:
      context: ./insta-clone-front
      dockerfile: Dockerfile
      args:
        - NUXT_PUBLIC_API_BASE=${NUXT_PUBLIC_API_BASE}
    container_name: insta-clone-front-prod
    ports:
      - "3000:80"
    environment:
      - NUXT_PUBLIC_API_BASE=${NUXT_PUBLIC_API_BASE}
    depends_on:
      - api
    restart: always
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 256M
```

### 8-4) セキュリティ設定（オプション）
※ 目的: リバースプロキシでHTTPSやルーティングを制御し、安全に公開する。

#### Nginx リバースプロキシの設定（推奨）

本番環境では、Nginxをリバースプロキシとして使用することを推奨します：

```nginx
# /etc/nginx/sites-available/insta-clone
server {
    listen 80;
    server_name your-domain.com;

    # フロントエンド
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # バックエンドAPI
    location /api {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 8-5) デプロイスクリプトの作成
※ 目的: pull→stop→build→up を自動化し、人的ミスを防ぐ。

`deploy.sh` を作成します：

```bash
#!/bin/bash

# デプロイスクリプト

echo "=== デプロイ開始 ==="

# 最新のコードを取得（Git使用の場合）
# git pull

# 既存のコンテナを停止
docker compose -f docker-compose.prod.yml down

# イメージをビルド
docker compose -f docker-compose.prod.yml build --no-cache

# コンテナを起動
docker compose -f docker-compose.prod.yml up -d

# ログを確認
docker compose -f docker-compose.prod.yml logs -f
```

実行権限を付与：

```bash
chmod +x deploy.sh
```

### 8-6) デプロイ手順
※ 目的: サーバーへの配置と起動の手順を定型化し、再現性を確保。

1. **サーバーに接続**:

```bash
ssh user@your-server-ip
```

2. **プロジェクトをアップロード**:

```bash
# SCPでアップロード（例）
scp -r ~/work/insta-clone-* user@your-server-ip:/home/user/
scp docker-compose.prod.yml user@your-server-ip:/home/user/
```

または、Gitリポジトリを使用：

```bash
git clone your-repo-url
cd insta-clone
```

3. **環境変数を設定**:

```bash
# .env.production ファイルを編集
nano insta-clone-api/.env.production
nano insta-clone-front/.env.production
```

4. **デプロイ実行**:

```bash
./deploy.sh
```

### 8-7) 動作確認
※ 目的: 本番でフロント/API/画像表示の一連の因果が成立するか検証する。

- [ ] `http://your-server-ip:3000` でフロントエンドが表示される
- [ ] `http://your-server-ip:8000/health` でAPIが動作する
- [ ] 投稿機能が動作する
- [ ] 画像がアップロードされ、表示される

## ✅ チェックリスト

- [ ] 本番環境のサーバーが準備された
- [ ] Docker と Docker Compose がインストールされた
- [ ] 本番用の環境変数が設定された
- [ ] `docker-compose.prod.yml` が作成された
- [ ] デプロイスクリプトが作成された
- [ ] サーバーにコードがアップロードされた
- [ ] 本番環境でアプリケーションが動作する

## 🔒 セキュリティチェックリスト

- [ ] `.env` ファイルがGitにコミットされていない（`.gitignore`に追加）
- [ ] Supabase のキーが適切に管理されている
- [ ] CORS設定が適切（本番URLのみ許可）
- [ ] ファイアウォール設定（必要に応じて）
- [ ] HTTPSの設定（ドメイン使用の場合）

## 🎯 次のステップ

本番環境の準備が完了したら、**step9.md** に進んでください。
（リリース前チェック）

