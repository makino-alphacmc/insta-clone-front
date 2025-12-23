# Step 2: Frontend セットアップ（Nuxt3 + Tailwind + shadcn-vue）

## 📋 このステップでやること

Instagram クローンのフロントエンド環境を構築します。

- Nuxt3 プロジェクトの作成
- Tailwind CSS の導入
- shadcn-vue の導入と初期設定

## ✅ 手順

### 2-1) プロジェクト作成

```bash
# 作業ディレクトリに移動
cd ~/work

# Nuxt3 プロジェクトを作成
npx nuxi init insta-clone-front

# プロジェクトディレクトリに移動
cd insta-clone-front

# 依存関係をインストール
npm install
```

### 2-2) SPA モード設定

`nuxt.config.ts` を編集して、SPA モードと API のベース URL を設定します：

```bash
# nuxt.config.ts を開く
code nuxt.config.ts  # または任意のエディタ
```

以下の内容に変更します：

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

### 2-3) Tailwind CSS の導入

**重要**: shadcn-vue は Tailwind CSS v3 を必要とします。v4 が自動インストールされる場合は、v3 にダウングレードしてください。

```bash
# Tailwind CSS v3 をインストール（v4がインストールされている場合は先にアンインストール）
npm uninstall tailwindcss @tailwindcss/vite
npm install -D tailwindcss@^3 postcss autoprefixer

# Tailwind CSS の設定ファイルを生成
npx tailwindcss init -p
```

`tailwind.config.js` を編集して、Nuxt 用のコンテンツパスを設定します：

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
	content: [
		'./components/**/*.{js,vue,ts}',
		'./layouts/**/*.vue',
		'./pages/**/*.vue',
		'./plugins/**/*.{js,ts}',
		'./app.vue',
		'./error.vue',
	],
	theme: {
		extend: {},
	},
	plugins: [],
}
```

CSS ファイルを作成します：

```bash
# assets/css ディレクトリを作成
mkdir -p assets/css

# main.css を作成
cat > assets/css/main.css << 'EOF'
@tailwind base;
@tailwind components;
@tailwind utilities;
EOF
```

`nuxt.config.ts` を更新して、CSS ファイルをインポートします：

```typescript
export default defineNuxtConfig({
	ssr: false, // SPA モード
	runtimeConfig: {
		public: {
			apiBase: 'http://localhost:8000', // バックエンドAPIのURL
		},
	},
	modules: ['shadcn-nuxt'],
	css: ['~/assets/css/main.css'],
})
```

### 2-4) shadcn-nuxt の導入

```bash
npx nuxi@latest module add shadcn-nuxt
```

### 2-5) shadcn-vue の初期化

```bash
npx shadcn-vue@latest init
```

**質問に答える際の設定**:

- フレームワーク: **Nuxt**
- コンポーネント出力先: **./components/ui** (デフォルトで OK)

### 2-6) shadcn-vue コンポーネントの一括導入

必要なコンポーネントを一度にインストールします：

```bash
npx shadcn-vue@latest add \
  accordion alert alert-dialog aspect-ratio avatar badge button calendar card \
  carousel checkbox collapsible command context-menu dialog drawer dropdown-menu \
  form hover-card input label menubar navigation-menu pagination popover \
  progress radio-group scroll-area select separator sheet skeleton \
  slider sonner switch table tabs textarea toast toggle toggle-group \
  tooltip
```

**注意**: もしエラーが出た場合は、エラーになったコンポーネントを外して再実行してください。

### ⚠️ トラブルシューティング

**エラー: "No Tailwind CSS configuration found"**

このエラーが出た場合：

1. Tailwind CSS v3 がインストールされているか確認: `npm list tailwindcss`
2. `tailwind.config.js` がプロジェクトルートに存在するか確認
3. `tailwind.config.js` が CommonJS 形式（`module.exports`）になっているか確認
4. `assets/css/main.css` に Tailwind ディレクティブが含まれているか確認

### 2-7) 動作確認

簡単なテストページを作成して、動作を確認します：

```bash
# pages/index.vue を編集
code pages/index.vue  # または任意のエディタ
```

以下の内容を追加：

```vue
<template>
	<div class="p-8">
		<h1 class="text-2xl font-bold mb-4">Instagram Clone</h1>
		<Button>テストボタン</Button>
	</div>
</template>

<script setup>
import { Button } from '@/components/ui/button'
</script>
```

開発サーバーを起動：

```bash
npm run dev
```

ブラウザで `http://localhost:3000` を開いて、ページが表示されることを確認してください。

## ✅ チェックリスト

- [ ✅ ] Nuxt3 プロジェクトが作成された
- [ ✅ ] `nuxt.config.ts` で SPA モードと API ベース URL が設定された
- [ ✅ ] Tailwind CSS が導入された
- [ ✅ ] shadcn-vue が初期化された
- [ ✅ ] コンポーネントがインストールされた
- [ ✅ ] `npm run dev` でサーバーが起動し、ページが表示された

## 🎯 次のステップ

動作確認ができたら、**step3.md** に進んでください。
（Frontend：モック画面の作成）
