# Instagram Clone Frontend

Instagram クローンのフロントエンド（Nuxt3 + Tailwind CSS + shadcn-vue）

## 📋 概要

このプロジェクトは、Instagram クローンのフロントエンドアプリケーションです。

- **フレームワーク**: Nuxt 3（SPA モード）
- **スタイリング**: Tailwind CSS
- **UI コンポーネント**: shadcn-vue
- **言語**: TypeScript

## 🛠️ 必要な環境

- Node.js 18 以上
- npm 9 以上

## 🚀 セットアップ手順

### 1. 依存関係のインストール

```bash
# プロジェクトディレクトリに移動
cd insta-clone-front

# 依存関係をインストール
npm install
```

### 2. 環境変数の設定（オプション）

`nuxt.config.ts` で API のベース URL を設定します：

```typescript
export default defineNuxtConfig({
	ssr: false, // SPA モード
	runtimeConfig: {
		public: {
			apiBase: 'http://localhost:8000', // バックエンドAPIのURL
		},
	},
})
```

デフォルトでは `http://localhost:8000` が設定されています。
本番環境では、環境変数で上書きすることも可能です。

### 3. 開発サーバーの起動

```bash
# 開発サーバーを起動
npm run dev
```

ブラウザで `http://localhost:3000` を開いて、アプリケーションが表示されることを確認してください。

## 📁 プロジェクト構成

```
insta-clone-front/
├── app/
│   ├── app.vue              # ルートコンポーネント
│   ├── assets/
│   │   └── css/
│   │       └── main.css     # Tailwind CSS のエントリーポイント
│   ├── components/
│   │   └── ui/              # shadcn-vue の UI コンポーネント
│   │       ├── button/
│   │       ├── card/
│   │       ├── avatar/
│   │       └── ...
│   ├── lib/
│   │   └── utils.ts         # ユーティリティ関数
│   └── pages/
│       ├── index.vue         # タイムライン画面（投稿一覧）
│       └── post/
│           └── new.vue       # 新規投稿画面
├── public/                   # 静的ファイル
├── nuxt.config.ts            # Nuxt の設定ファイル
├── tailwind.config.js        # Tailwind CSS の設定ファイル
├── tsconfig.json             # TypeScript の設定ファイル
└── package.json              # 依存関係の定義
```

## 🎨 主な機能

### タイムライン画面（`/`）

- 投稿一覧を 3 列グリッドレイアウトで表示（PC 向け）
- ローディング状態（Skeleton 表示）
- エラー状態（モーダル表示）
- 空状態（投稿がない場合の表示）

### 新規投稿画面（`/post/new`）

- 画像選択とプレビュー機能
- キャプション入力（最大 500 文字）
- 投稿成功時のモーダル表示
- 投稿エラー時のアラート表示

## 🔌 API 連携

フロントエンドは以下のエンドポイントと連携します：

- `GET /health` - ヘルスチェック
- `GET /posts` - 投稿一覧取得
- `POST /posts` - 新規投稿作成（画像 + キャプション）

API の詳細は、`../insta-clone-api/README.md` を参照してください。

## 🧪 動作確認

### 開発環境での動作確認

1. **バックエンド API を起動**（別のターミナルで）:

```bash
cd ../insta-clone-api
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

2. **フロントエンドを起動**:

```bash
cd insta-clone-front
npm run dev
```

3. **ブラウザで確認**:

- `http://localhost:3000` を開く
- タイムライン画面が表示されることを確認
- 「新規投稿」ボタンから投稿画面に遷移できることを確認

## 📦 ビルド

本番環境用のビルド:

```bash
npm run build
```

ビルドされたファイルは `.output` ディレクトリに生成されます。

## 🔧 トラブルシューティング

### エラー: "Cannot find module 'xxx'"

依存関係がインストールされていない可能性があります。

```bash
npm install
```

### エラー: "No Tailwind CSS configuration found"

Tailwind CSS の設定ファイルが存在しない可能性があります。

```bash
npx tailwindcss init -p
```

### エラー: API に接続できない

- バックエンド API が起動しているか確認
- `nuxt.config.ts` の `apiBase` が正しいか確認
- ブラウザのコンソールでエラーメッセージを確認

### エラー: CORS エラー

バックエンド API の `.env` ファイルで `ALLOWED_ORIGINS` に `http://localhost:3000` が含まれているか確認してください。

## 📚 参考資料

- [Nuxt 3 公式ドキュメント](https://nuxt.com/)
- [Tailwind CSS 公式ドキュメント](https://tailwindcss.com/)
- [shadcn-vue 公式ドキュメント](https://www.shadcn-vue.com/)

## 🎯 次のステップ

詳細な実装手順は、`手順書/` ディレクトリ内の各ステップを参照してください。
