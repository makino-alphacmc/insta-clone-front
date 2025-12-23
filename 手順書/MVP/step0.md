# Step 0: 事前準備と計画

## 📋 このステップでやること

プロジェクトを始める前に、必要な情報と設定を決めます。

## ✅ チェックリスト

### 1. リポジトリ構成の決定

- [ ✅ ] **Frontend リポジトリ**: `insta-clone-front` (Nuxt3 SPA + shadcn-vue)
- [ ✅ ] **Backend リポジトリ**: `insta-clone-api` (FastAPI + SQLite + SQLAlchemy)

**注意**: 今回は 2 つの別々のプロジェクトとして作成します。

### 2. ローカル開発ポートの決定

- [ ✅ ] **Frontend**: `http://localhost:3000`
- [ ✅ ] **Backend API**: `http://localhost:8000`

### 3. API 仕様の確認（最低限）

以下のエンドポイントを実装します：

- [ ] `GET /health` - ヘルスチェック
- [ ] `GET /posts` - 投稿一覧取得
- [ ] `POST /posts` - 新規投稿（画像 + キャプション）

### 4. データベーススキーマ（MVP）

**posts テーブル**:

- `id` (整数, 主キー)
- `image_url` (文字列) - Supabase Storage の URL
- `caption` (文字列) - 投稿のキャプション
- `created_at` (日時) - 作成日時

## 📝 メモ欄

以下を記録しておくと便利です：

```
プロジェクト名: insta-clone
作業ディレクトリ: ~/work (または任意の場所)
Frontend ポート: 3000
Backend ポート: 8000
```

## 🎯 次のステップ

準備ができたら、**step1.md** に進んでください。
（環境構築：必要なツールのインストール確認）
