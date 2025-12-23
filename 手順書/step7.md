# Step 7: Docker 化（コンテナ化と docker-compose の設定）

## 📋 このステップでやること

アプリケーションをDockerコンテナ化し、docker-composeで管理できるようにします。
- Backend の Dockerfile 作成
- Frontend の Dockerfile 作成
- docker-compose.yml の作成

## ✅ 手順

### 7-1) Backend の Dockerfile 作成
※ 目的: APIをコンテナ化し、依存を閉じ込めてどこでも同一挙動にする。

`insta-clone-api/Dockerfile` を作成します：

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 依存関係をインストール
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# アプリケーションコードをコピー
COPY . .

# ポートを公開
EXPOSE 8000

# アプリケーションを起動
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 7-2) Backend の requirements.txt 作成
※ 目的: 依存を固定し、Dockerビルドで再現性を確保する。

`insta-clone-api/requirements.txt` を作成します：

```txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
python-dotenv==1.0.0
sqlalchemy==2.0.23
supabase==2.0.0
```

### 7-3) Backend の .dockerignore 作成（オプション）
※ 目的: 不要ファイルをビルドコンテキストから除外し、イメージを軽量化する。

`insta-clone-api/.dockerignore` を作成します：

```
venv/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
.env
*.db
*.sqlite
```

### 7-4) Frontend の Dockerfile 作成
※ 目的: フロントをビルドして静的配信可能なコンテナにまとめる。

`insta-clone-front/Dockerfile` を作成します：

```dockerfile
# ビルドステージ
FROM node:18-alpine AS builder

WORKDIR /app

# 依存関係をインストール
COPY package*.json ./
RUN npm ci

# アプリケーションコードをコピー
COPY . .

# ビルド
RUN npm run build

# 本番ステージ
FROM nginx:alpine

# ビルド成果物をコピー
COPY --from=builder /app/.output/public /usr/share/nginx/html

# Nginx設定ファイルをコピー（オプション）
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 7-5) Frontend の nginx.conf 作成
※ 目的: SPAルーティングを成立させ、静的ファイルを最適に配信する。

`insta-clone-front/nginx.conf` を作成します：

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # SPA用の設定（すべてのリクエストをindex.htmlに）
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静的ファイルのキャッシュ設定
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 7-6) Frontend の .dockerignore 作成（オプション）
※ 目的: node_modules 等を除外し、ビルド時間とイメージサイズを削減。

`insta-clone-front/.dockerignore` を作成します：

```
node_modules/
.nuxt/
.output/
.env
.DS_Store
```

### 7-7) docker-compose.yml の作成
※ 目的: front/api をサービス分離しつつ一括起動できるようにする（因果: ローカルと本番の構成を揃えやすくする）。

プロジェクトのルートディレクトリ（`~/work`）に `docker-compose.yml` を作成します：

```yaml
version: '3.8'

services:
  api:
    build:
      context: ./insta-clone-api
      dockerfile: Dockerfile
    container_name: insta-clone-api
    ports:
      - "8000:8000"
    env_file:
      - ./insta-clone-api/.env
    volumes:
      # SQLite データベースを永続化
      - ./insta-clone-api/app.db:/app/app.db
    restart: unless-stopped

  front:
    build:
      context: ./insta-clone-front
      dockerfile: Dockerfile
    container_name: insta-clone-front
    ports:
      - "3000:80"
    environment:
      # コンテナ間通信ではサービス名を使用
      - NUXT_PUBLIC_API_BASE=http://api:8000
    depends_on:
      - api
    restart: unless-stopped
```

**注意**: フロントエンドからAPIにアクセスする際、ブラウザからは `http://localhost:8000` を使い、コンテナ内からは `http://api:8000` を使います。本番環境では、フロントエンドの環境変数を適切に設定する必要があります。

### 7-8) ビルドと起動
※ 目的: compose で両コンテナを立ち上げ、依存関係が正しく連携するかを確認。

```bash
# プロジェクトのルートディレクトリに移動
cd ~/work

# イメージをビルドして起動
docker compose up -d --build

# ログを確認
docker compose logs -f
```

### 7-9) 動作確認
※ 目的: コンテナ経由でも Front 3000 / API 8000 が動作するかを検証。

- **Frontend**: `http://localhost:3000` を開く
- **Backend API**: `http://localhost:8000/health` を開く
- **Swagger UI**: `http://localhost:8000/docs` を開く

### 7-10) 停止とクリーンアップ
※ 目的: 実行環境を安全に停止し、不要リソースを整理して次の起動に備える。

```bash
# コンテナを停止
docker compose down

# コンテナとイメージを削除（データは保持）
docker compose down --rmi local

# ボリュームも削除する場合（データベースも削除される）
docker compose down -v
```

## ✅ チェックリスト

- [ ] Backend の Dockerfile が作成された
- [ ] Backend の requirements.txt が作成された
- [ ] Frontend の Dockerfile が作成された
- [ ] Frontend の nginx.conf が作成された
- [ ] docker-compose.yml が作成された
- [ ] `docker compose up` でコンテナが起動する
- [ ] Frontend（`http://localhost:3000`）が表示される
- [ ] Backend API（`http://localhost:8000/health`）が動作する
- [ ] フロントエンドからバックエンドAPIにアクセスできる

## 🎯 次のステップ

Docker化が完了したら、**step8.md** に進んでください。
（本番環境へのデプロイ準備）

