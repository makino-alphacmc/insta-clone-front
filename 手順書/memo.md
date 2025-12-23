################################################################################

# ✅ やることリスト完全版（最新版）

# 方針:

# - Repo はパターン B（frontend / backend 分離）

# - UI は Nuxt3(SPA) + Tailwind + shadcn-vue（最初に全部導入）

# - Backend は FastAPI + SQLite + SQLAlchemy

# - 画像は Supabase Storage（無料枠）に保存（DB は Supabase 使わない）

# - デプロイは Docker で疎結合（frontend コンテナ / backend コンテナ分離）

# - 本番は docker compose で連携（フロント →API を HTTP で叩くだけ）

################################################################################

################################################################################

# 0. 事前決め（必ず最初に確定）

################################################################################

# [ ] リポジトリ

# - insta-clone-front (Nuxt SPA + shadcn-vue)

# - insta-clone-api (FastAPI + SQLite + SQLAlchemy)

#

# [ ] ローカル開発ポート（例）

# - Front: http://localhost:3000

# - API : http://localhost:8000

#

# [ ] API 仕様（最低限）

# - GET /health

# - GET /posts

# - POST /posts (multipart: file + caption)

#

# [ ] DB スキーマ（MVP）

# posts: id, image_url, caption, created_at

################################################################################

################################################################################

# 1. Frontend セットアップ（Nuxt3 + Tailwind + shadcn-vue FULL）

################################################################################

# 1-1) プロジェクト作成

mkdir -p ~/work && cd ~/work
npx nuxi init insta-clone-front
cd insta-clone-front
npm install

# 1-2) SPA モード設定（nuxt.config.ts）

# [ ] ssr: false

# [ ] runtimeConfig.public.apiBase をローカル API へ

# 例:

# export default defineNuxtConfig({

# ssr: false,

# runtimeConfig: { public: { apiBase: 'http://localhost:8000' } }

# })

# 1-3) Tailwind 導入（Nuxt モジュール）

npx nuxi@latest module add @nuxtjs/tailwindcss

# 1-4) shadcn-nuxt 導入

npx nuxi@latest module add shadcn-nuxt

# 1-5) shadcn-vue 初期化（質問に回答して設定）

# ※ フレームワーク: Nuxt

# ※ コンポーネント出力: ./components/ui

npx shadcn-vue@latest init

# 1-6) shadcn-vue コンポーネントを最初から全部導入

# ※ もし未対応コンポーネントがあれば、その分だけエラーになるので外して再実行する

npx shadcn-vue@latest add \
 accordion alert alert-dialog aspect-ratio avatar badge button calendar card \
 carousel checkbox collapsible command context-menu dialog drawer dropdown-menu \
 form hover-card input label menubar navigation-menu pagination popover \
 progress radio-group scroll-area select separator sheet skeleton \
 slider sonner switch table tabs textarea toast toggle toggle-group \
 tooltip

# 1-7) 動作確認（Button など置いて起動）

npm run dev

# → http://localhost:3000 で起動確認

################################################################################

# 2. Frontend：モック画面（モックデータあり）を作る

################################################################################

# 2-1) タイムライン画面（pages/index.vue）

# [ ] const posts = [...] を用意（image_url, caption, created_at）

# [ ] shadcn Card / Avatar / Skeleton でインスタ風カード

# [ ] 0 件表示（Empty state）も用意

#

# 2-2) 投稿画面（pages/post/new.vue）

# [ ] file input + caption textarea + Button

# [ ] まだ API は叩かない（UI だけ）

#

# 2-3) ルーティング導線

# [ ] ヘッダーに「New Post」ボタン（/post/new へ）

# [ ] トースト（sonner/toast）導入済みなら UI 通知の枠だけ作る

################################################################################

# 3. Backend セットアップ（FastAPI + SQLite + SQLAlchemy）

################################################################################

# 3-1) プロジェクト作成

cd ~/work
mkdir insta-clone-api
cd insta-clone-api

# 3-2) 仮想環境（任意）

python -m venv venv
source venv/bin/activate

# 3-3) 依存関係（例：同期構成で素早く）

pip install fastapi uvicorn[standard] python-dotenv
pip install sqlalchemy

# 3-4) ディレクトリ構成（例）

mkdir -p app
touch app/main.py app/db.py app/models.py app/schemas.py app/storage.py

# 3-5) SQLite 接続（app/db.py）

# [ ] sqlite:///./app.db を使う

# [ ] SQLAlchemy engine / SessionLocal / Base を用意

# 3-6) posts モデル（app/models.py）

# [ ] Post(id, image_url, caption, created_at)

# 3-7) 起動用（app/main.py）

# [ ] GET /health

# [ ] CORS（フロント origin を許可）

# [ ] GET /posts（DB から取得）

# [ ] POST /posts（画像アップ + DB insert）

# 3-8) ローカル起動

uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# → http://localhost:8000/health 確認

################################################################################

# 4. 画像保存（Supabase Storage）を API 側に実装

################################################################################

# 4-1) Supabase プロジェクト作成（Web UI）

# [ ] Project URL / anon key を控える

# [ ] Storage bucket 作成（例: post-images）

# [ ] Public ON（閲覧できる URL 運用）

#

# 4-2) API 側 .env を作る

cat > .env << 'EOF'
SUPABASE_URL=YOUR_SUPABASE_URL
SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
SUPABASE_BUCKET=post-images
ALLOWED_ORIGINS=http://localhost:3000
EOF

# 4-3) Supabase SDK を使う（例）

pip install supabase

# 4-4) app/storage.py にアップロード関数

# [ ] file bytes を bucket に upload

# [ ] public URL を組み立てて返す（または SDK の getPublicUrl 相当を利用）

################################################################################

# 5. API 開発（MVP 完成ライン）

################################################################################

# 5-1) GET /posts

# [ ] DB から created_at desc で返す

# [ ] レスポンス例:

# [

# { "id": 1, "image_url": "...", "caption": "...", "created_at": "..." }

# ]

# 5-2) POST /posts（multipart）

# [ ] file + caption を受け取る

# [ ] 画像を Supabase Storage に保存 → image_url を得る

# [ ] posts テーブルへ insert

# [ ] 作成した Post を返す

#

# 5-3) 手動テスト（例：curl）

# curl -X POST "http://localhost:8000/posts" \

# -F "caption=hello" \

# -F "file=@/path/to/image.jpg"

# 5-4) CORS を本番向けに拡張

# [ ] ALLOWED_ORIGINS に本番フロント URL を追加できる形にする

################################################################################

# 6. Frontend：モック → fetch に差し替え

################################################################################

# 6-1) TL（pages/index.vue）

# [ ] const posts = [...] を削除（または fallback 用に残す）

# [ ] useFetch(`${apiBase}/posts`) で取得

# [ ] pending → Skeleton / error → Alert で表示

# 6-2) 投稿（pages/post/new.vue）

# [ ] FormData で file + caption を POST /posts

# [ ] 成功: toast 表示 → / に遷移 → 再取得

# [ ] 失敗: toast/alert

################################################################################

# 7. Docker 化（疎結合: frontend / backend 別コンテナ）

################################################################################

# 7-1) Backend: Dockerfile（insta-clone-api/Dockerfile）

# [ ] FastAPI を uvicorn で起動

# [ ] SQLite はコンテナ内 or volume に保存（永続化するなら volume 推奨）

#

# 7-2) Frontend: Dockerfile（insta-clone-front/Dockerfile）

# [ ] Nuxt SPA をビルドして配信

# - シンプル案: Node で serve

# - きれい案: Nginx で静的配信（dist 出力に合わせる）

#

# 7-3) ルートに compose を置く（例: insta-compose/ みたいに）

# [ ] docker-compose.yml で services を分ける

# - api:

# ports: "8000:8000"

# env_file: ./insta-clone-api/.env

# volumes: ./insta-clone-api/app.db:/app/app.db（任意）

# - front:

# ports: "3000:80" or "3000:3000"

# environment:

# - NUXT_PUBLIC_API_BASE=http://api:8000 # ★ コンテナ間通信はサービス名で

#

# 7-4) Compose 起動

# docker compose up -d --build

# → Front(3000) から API(api:8000) を叩けることを確認

################################################################################

# 8. 本番リリース（Docker 前提）

################################################################################

# 8-1) 本番環境に docker + docker compose を用意

# [ ] VPS / 自宅サーバ / どこでも OK（この手順は Docker 疎結合で同じ）

#

# 8-2) 本番用の環境変数を用意

# [ ] insta-clone-api/.env（本番 Supabase キー、ALLOWED_ORIGINS=本番フロント URL）

# [ ] front 側 env（NUXT_PUBLIC_API_BASE=http(s)://api の公開 URL もしくは同一 compose なら http://api:8000）

# 8-3) 起動

# docker compose up -d --build

# 8-4) 動作確認（本番）

# [ ] /health OK

# [ ] 投稿 →Storage に保存 →TL に表示

# [ ] 画像 URL がブラウザで見れる

################################################################################

# 9. リリース前チェック（最低限）

################################################################################

# [ ] スマホ幅で崩れない（TL / 投稿画面）

# [ ] 投稿エラー（画像なし、巨大画像、拡張子変）時のメッセージ

# [ ] SQLite 永続化（再起動で消えないように volume 確認）

# [ ] README に起動手順（npm/dev と docker/本番）を書く

# [ ] v0.1.0 タグを打つ

################################################################################

# 10. リリース 🎉

################################################################################

# [ ] Git tag: v0.1.0

# [ ] URL 共有

# [ ] 次フェーズ TODO:

# - 認証（JWT）

# - ユーザー別投稿

# - ページネーション / 無限スクロール

# - 画像圧縮・サムネ生成（API 側 or Storage 側）

################################################################################
