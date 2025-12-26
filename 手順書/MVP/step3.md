# Step 3: Frontend モック画面の作成

## 📋 このステップでやること

API がまだない状態でも、UI を確認できるようにモック画面を作成します。

- タイムライン画面（投稿一覧）- 3 列レイアウト（PC 向け）
- 新規投稿画面
- 各状態の実装（ローディング、エラー、空状態）

## ✅ 手順

### 3-1) タイムライン画面（pages/index.vue）

モックデータを画面に流し込み、3 列レイアウトで一覧 UI を実装します。

`pages/index.vue` を編集します：

```vue
<template>
	<!-- 背景とコンテナ（PC向け） -->
	<div class="min-h-screen bg-gray-50">
		<!-- min-h-screen: 最小高さを画面の高さに設定（100vh相当） -->
		<!-- bg-gray-50: 背景色を薄いグレーに設定 -->

		<!-- ヘッダー部分 -->
		<header class="bg-white border-b border-gray-200">
			<!-- bg-white: 背景色を白に設定 -->
			<!-- border-b: 下側にボーダーを追加 -->
			<!-- border-gray-200: ボーダーの色をグレーに設定 -->

			<div class="container mx-auto max-w-7xl px-4 py-4">
				<!-- container: レスポンシブな最大幅を設定（画面サイズに応じて変わる） -->
				<!-- mx-auto: 左右のマージンを自動にして中央揃え -->
				<!-- max-w-7xl: 最大幅を1280px（80rem）に制限 -->
				<!-- px-4: 左右にパディング16px -->
				<!-- py-4: 上下にパディング16px -->

				<div class="flex justify-between items-center">
					<!-- flex: フレックスボックスレイアウトを有効化 -->
					<!-- justify-between: 子要素を両端に配置 -->
					<!-- items-center: 子要素を縦方向の中央に配置 -->

					<h1 class="text-2xl font-bold">Instagram Clone</h1>
					<!-- text-2xl: フォントサイズを1.5rem（24px）に設定 -->
					<!-- font-bold: フォントの太さを太字に設定 -->

					<!-- 新規投稿ボタン -->
					<Button as-child>
						<!-- as-child: Buttonコンポーネントのラッパーを生成せず、子要素を直接使用 -->
						<NuxtLink to="/post/new">新規投稿</NuxtLink>
						<!-- NuxtLink: Nuxtのルーティング機能を使ったリンク -->
						<!-- to="/post/new": 遷移先のパスを指定 -->
					</Button>
				</div>
			</div>
		</header>

		<!-- メインコンテンツ -->
		<main class="container mx-auto max-w-7xl px-4 py-8">
			<!-- py-8: 上下にパディング32px -->

			<!-- 空状態 -->
			<div v-if="posts.length === 0" class="text-center py-20">
				<!-- v-if: 条件がtrueの場合のみ要素を表示（posts.length === 0の場合） -->
				<!-- text-center: テキストを中央揃え -->
				<!-- py-20: 上下にパディング80px -->

				<div class="mb-4">
					<!-- mb-4: 下側にマージン16px -->
					<div
						class="w-20 h-20 mx-auto bg-gray-100 rounded-lg flex items-center justify-center"
					>
						<!-- w-20: 幅を80px（5rem）に設定 -->
						<!-- h-20: 高さを80px（5rem）に設定 -->
						<!-- bg-gray-100: 背景色をグレーに設定 -->
						<!-- rounded-lg: 角を丸くする（0.5rem相当） -->
						<!-- flex: フレックスボックスレイアウト -->
						<!-- items-center: 縦方向の中央揃え -->
						<!-- justify-center: 横方向の中央揃え -->
						<svg
							class="w-10 h-10 text-gray-400"
							fill="none"
							stroke="currentColor"
							viewBox="0 0 24 24"
						>
							<!-- w-10: 幅を40px（2.5rem）に設定 -->
							<!-- h-10: 高さを40px（2.5rem）に設定 -->
							<!-- text-gray-400: テキスト（SVGの色）をグレーに設定 -->
						</svg>
					</div>
				</div>
				<p class="text-lg text-gray-500 mb-2">まだ投稿がありません</p>
				<!-- text-lg: フォントサイズを1.125rem（18px）に設定 -->
				<!-- text-gray-500: テキストの色をグレーに設定 -->
				<!-- mb-2: 下側にマージン8px -->
				<p class="text-sm text-gray-400 mb-6">
					最初の投稿を作成して、みんなとシェアしましょう
				</p>
				<!-- text-sm: フォントサイズを0.875rem（14px）に設定 -->
				<!-- mb-6: 下側にマージン24px -->
				<Button as-child>
					<NuxtLink to="/post/new">最初の投稿を作成</NuxtLink>
				</Button>
			</div>

			<!-- 投稿一覧（3列グリッドレイアウト） -->
			<div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
				<!-- v-else: v-ifの条件がfalseの場合に表示 -->
				<!-- grid: グリッドレイアウトを有効化 -->
				<!-- grid-cols-1: デフォルトで1列（スマホ向け） -->
				<!-- md:grid-cols-2: 中サイズ画面（768px以上）で2列 -->
				<!-- lg:grid-cols-3: 大サイズ画面（1024px以上）で3列 -->
				<!-- gap-6: グリッドアイテム間の間隔を24pxに設定 -->

				<Card v-for="post in posts" :key="post.id" class="overflow-hidden">
					<!-- v-for: 配列の各要素に対して要素を繰り返し生成 -->
					<!-- post in posts: posts配列の各要素をpostとして処理 -->
					<!-- :key="post.id": 各要素を識別するためのキー（Vueが要素を追跡するために必要） -->
					<!-- overflow-hidden: はみ出した内容を隠す -->

					<!-- ヘッダー部分（アバター + ユーザー名 + 日時） -->
					<CardHeader class="pb-3">
						<!-- pb-3: 下側にパディング12px -->
						<div class="flex items-center gap-3">
							<!-- gap-3: 子要素間の間隔を12pxに設定 -->
							<Avatar>
								<AvatarFallback>{{
									post.username?.charAt(0) || 'U'
								}}</AvatarFallback>
								<!-- {{ }}: Vueのテンプレート構文でJavaScriptの式を評価して表示 -->
								<!-- post.username?.charAt(0): オプショナルチェーン（usernameがnull/undefinedの場合はエラーにならない） -->
								<!-- charAt(0): 文字列の最初の文字を取得 -->
								<!-- || 'U': 左側がfalsy（null/undefined/空文字など）の場合に'U'を返す -->
							</Avatar>
							<div class="flex-1">
								<!-- flex-1: 残りのスペースを全て使用 -->
								<p class="font-semibold text-sm">
									{{ post.username || 'User' }}
								</p>
								<!-- font-semibold: フォントの太さをセミボールドに設定 -->
								<p class="text-xs text-gray-500">
									{{ formatDate(post.created_at) }}
								</p>
								<!-- text-xs: フォントサイズを0.75rem（12px）に設定 -->
							</div>
						</div>
					</CardHeader>

					<!-- 画像エリア -->
					<CardContent class="p-0">
						<!-- p-0: パディングを0に設定 -->
						<img
							:src="post.image_url"
							<!--
							:src:
							v-bind:srcの省略形、動的に属性値を設定
							--
						/>
						:alt="post.caption || '投稿画像'"
						<!-- :alt: 画像の代替テキストを動的に設定 -->
						class="w-full aspect-square object-cover"
						<!-- w-full: 幅を100%に設定 -->
						<!-- aspect-square: アスペクト比を1:1（正方形）に設定 -->
						<!-- object-cover: 画像を領域いっぱいに表示（アスペクト比を保ちながら） -->
						/>
					</CardContent>

					<!-- フッター部分（キャプション） -->
					<CardFooter class="flex flex-col items-start gap-2 pt-3">
						<!-- flex-col: フレックスボックスの方向を縦に設定 -->
						<!-- items-start: 子要素を左揃え（縦方向の開始位置） -->
						<!-- gap-2: 子要素間の間隔を8pxに設定 -->
						<!-- pt-3: 上側にパディング12px -->
						<p class="font-semibold text-sm">{{ post.username || 'User' }}</p>
						<p class="text-sm text-gray-700">{{ post.caption || '' }}</p>
						<!-- text-gray-700: テキストの色を濃いグレーに設定 -->
					</CardFooter>
				</Card>
			</div>
		</main>
	</div>
</template>

<script setup>
// <script setup>: Vue 3のComposition APIを使用（setup構文）
// この構文では、変数や関数を自動的にテンプレートで使用可能にする

import { Card, CardContent, CardFooter, CardHeader } from '@/components/ui/card'
// import: 他のファイルから機能を読み込む
// @/: Nuxtのエイリアスで、app/ディレクトリを指す
// {}: 名前付きエクスポートを読み込む（複数のコンポーネントを一度に読み込む）

import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { Button } from '@/components/ui/button'

// モックデータ（後でAPIから取得するデータに置き換える）
const posts = [
	// const: 定数を宣言（再代入不可）
	// posts: 投稿データの配列
	// []: 配列リテラル（複数の要素を格納）
	{
		// {}: オブジェクトリテラル（キーと値のペアを格納）
		id: 1,
		// id: 投稿の一意の識別子（数値）
		image_url: 'https://via.placeholder.com/480x480?text=Post+1',
		// image_url: 画像のURL（文字列）
		// '...': 文字列リテラル（シングルクォートで囲む）
		caption: 'これは最初の投稿です！',
		// caption: 投稿のキャプション（説明文）
		username: 'user1',
		// username: ユーザー名
		created_at: new Date().toISOString(),
		// new Date(): 現在の日時を表すDateオブジェクトを作成
		// .toISOString(): 日時をISO 8601形式の文字列に変換（例: "2024-01-01T12:00:00.000Z"）
	},
	{
		id: 2,
		image_url: 'https://via.placeholder.com/480x480?text=Post+2',
		caption: '2つ目の投稿です',
		username: 'user2',
		created_at: new Date(Date.now() - 3600000).toISOString(),
		// Date.now(): 現在時刻をミリ秒で取得
		// - 3600000: 1時間前（3600秒 × 1000ミリ秒）
	},
	{
		id: 3,
		image_url: 'https://via.placeholder.com/480x480?text=Post+3',
		caption: '3つ目の投稿です',
		username: 'user3',
		created_at: new Date(Date.now() - 7200000).toISOString(),
		// - 7200000: 2時間前（7200秒 × 1000ミリ秒）
	},
]

// 日付を日本語形式に変換する関数
const formatDate = (dateString) => {
	// const formatDate: 関数を定数として宣言
	// (dateString): 関数の引数（パラメータ）
	// =>: アロー関数の構文（functionキーワードの代わり）

	const date = new Date(dateString)
	// new Date(dateString): 文字列をDateオブジェクトに変換

	return date.toLocaleDateString('ja-JP', {
		// return: 関数の戻り値を返す
		// toLocaleDateString(): 日時をロケール（言語・地域）に応じた文字列に変換
		// 'ja-JP': 日本語（日本）のロケールを指定
		// {}: オプションオブジェクト（表示形式を指定）
		year: 'numeric',
		// year: 'numeric': 年を4桁の数字で表示（例: 2024）
		month: 'long',
		// month: 'long': 月を長い形式で表示（例: 1月）
		day: 'numeric',
		// day: 'numeric': 日を数字で表示（例: 1）
		hour: '2-digit',
		// hour: '2-digit': 時間を2桁で表示（例: 09）
		minute: '2-digit',
		// minute: '2-digit': 分を2桁で表示（例: 05）
	})
}
</script>
```

### 3-2) 新規投稿画面（pages/post/new.vue）

フォーム入力とプレビューの流れを実装します。

`pages/post/new.vue` を作成します：

```vue
<template>
	<!-- 背景とコンテナ -->
	<div class="min-h-screen bg-gray-50">
		<!-- ヘッダー部分 -->
		<header class="bg-white border-b border-gray-200">
			<div class="container mx-auto max-w-4xl px-4 py-4">
				<div class="flex items-center gap-4">
					<!-- 戻るボタン -->
					<Button variant="ghost" size="icon" @click="$router.push('/')">
						<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
						</svg>
					</Button>
					<h1 class="text-xl font-semibold">新規投稿</h1>
				</div>
			</div>
		</header>

		<!-- メインコンテンツ -->
		<main class="container mx-auto max-w-4xl px-4 py-8">
			<Card>
				<CardHeader>
					<CardTitle>投稿を作成</CardTitle>
				</CardHeader>
				<CardContent>
					<form @submit.prevent="handleSubmit" class="space-y-6">
						<!-- @submit.prevent: フォーム送信時のイベントハンドラー -->
						<!-- @submit: submitイベントを監視 -->
						<!-- .prevent: デフォルトの動作（ページリロード）を防ぐ（event.preventDefault()と同じ） -->
						<!-- handleSubmit: 送信時に実行される関数 -->
						<!-- space-y-6: 子要素間の縦方向の間隔を24pxに設定 -->

						<!-- 画像選択セクション -->
						<div class="space-y-2">
							<!-- space-y-2: 子要素間の縦方向の間隔を8pxに設定 -->
							<Label for="image">画像を選択</Label>
							<!-- Label: フォーム要素のラベル（アクセシビリティ向上） -->
							<!-- for="image": id="image"の要素と関連付け -->
							<p class="text-sm text-gray-500">JPEG、PNG形式の画像をアップロードできます（最大10MB）</p>

							<!-- ファイル入力エリア（ドラッグ&ドロップ対応風） -->
							<div class="border-2 border-dashed border-gray-300 rounded-lg p-12 text-center">
								<!-- border-2: ボーダーの太さを2pxに設定 -->
								<!-- border-dashed: ボーダーのスタイルを破線に設定 -->
								<!-- border-gray-300: ボーダーの色をグレーに設定 -->
								<!-- rounded-lg: 角を丸くする -->
								<!-- p-12: パディングを48pxに設定 -->
								<Input
									id="image"
									<!-- id="image": 要素の識別子（Labelのforと関連付け） -->
									type="file"
									<!-- type="file": ファイル選択入力欄 -->
									accept="image/*"
									<!-- accept: 選択可能なファイルタイプを指定（画像ファイルのみ） -->
									<!-- image/*: すべての画像形式を許可 -->
									@change="handleFileChange"
									<!-- @change: ファイルが選択された時にhandleFileChange関数を実行 -->
									class="hidden"
									<!-- hidden: 要素を非表示にする（カスタムデザインのラベルを使用するため） -->
								/>
								<label for="image" class="cursor-pointer">
									<!-- for="image": Input要素と関連付け（ラベルをクリックするとファイル選択ダイアログが開く） -->
									<!-- cursor-pointer: マウスカーソルをポインターに変更（クリック可能であることを示す） -->
									<div class="space-y-4">
										<Button type="button" variant="default">ファイルを選択</Button>
										<!-- type="button": ボタンのタイプ（フォーム送信をしない） -->
										<!-- variant="default": ボタンのスタイル（デフォルトスタイル） -->
										<p class="text-sm text-gray-500">または、ここにドラッグ&ドロップ</p>
									</div>
								</label>
							</div>

							<!-- プレビュー画像 -->
							<div v-if="previewUrl" class="mt-4">
								<!-- v-if="previewUrl": previewUrlが存在する場合のみ表示 -->
								<!-- mt-4: 上側にマージン16px -->
								<img
									:src="previewUrl"
									<!-- :src: 画像のURLを動的に設定 -->
									alt="プレビュー"
									<!-- alt: 画像の代替テキスト（アクセシビリティ） -->
									class="w-full max-h-96 object-contain rounded-lg border border-gray-200"
									<!-- w-full: 幅を100%に設定 -->
									<!-- max-h-96: 最大高さを384px（24rem）に制限 -->
									<!-- object-contain: 画像を領域内に収める（アスペクト比を保つ） -->
									<!-- rounded-lg: 角を丸くする -->
									<!-- border: ボーダーを追加 -->
									<!-- border-gray-200: ボーダーの色をグレーに設定 -->
								/>
							</div>
						</div>

						<!-- キャプション入力セクション -->
						<div class="space-y-2">
							<Label for="caption">キャプション</Label>
							<p class="text-sm text-gray-500">投稿に説明文を追加できます（任意）</p>
							<Textarea
								id="caption"
								v-model="caption"
								<!-- v-model: 双方向データバインディング（入力値と変数を自動同期） -->
								<!-- caption: 入力値が自動的にcaption変数に保存される -->
								placeholder="キャプションを入力..."
								<!-- placeholder: 入力例を表示（ユーザーの理解を助ける） -->
								rows="4"
								<!-- rows: テキストエリアの行数を4行に設定 -->
								:maxlength="500"
								<!-- :maxlength: 最大文字数を500文字に制限 -->
							/>
							<p class="text-xs text-gray-400 text-right">{{ caption.length }} / 500</p>
							<!-- text-right: テキストを右揃え -->
							<!-- {{ caption.length }}: 現在の文字数を表示 -->
						</div>

						<!-- ボタンエリア -->
						<div class="flex gap-4">
							<!-- gap-4: 子要素間の間隔を16pxに設定 -->
							<Button type="submit" :disabled="!selectedFile" class="flex-1">
								<!-- type="submit": フォーム送信ボタン -->
								<!-- :disabled: ボタンの有効/無効を動的に設定 -->
								<!-- !selectedFile: selectedFileがnull/falsyの場合にtrue（ボタンが無効化される） -->
								<!-- flex-1: 残りのスペースを全て使用 -->
								投稿する
							</Button>
							<Button type="button" variant="outline" @click="$router.push('/')" class="flex-1">
								<!-- type="button": フォーム送信をしないボタン -->
								<!-- variant="outline": アウトラインスタイルのボタン -->
								<!-- @click: クリック時に実行される関数 -->
								<!-- $router.push('/'): Nuxtのルーターでトップページに遷移 -->
								キャンセル
							</Button>
						</div>
					</form>
				</CardContent>
			</Card>
		</main>
	</div>
</template>

<script setup>
import { ref } from 'vue'
// import { ref }: Vueのref関数を読み込む（リアクティブな変数を作成するため）
// ref: 値が変わると自動的に画面が更新される変数を作成

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Textarea } from '@/components/ui/textarea'

// リアクティブな状態変数
const selectedFile = ref(null)
// const selectedFile: 選択されたファイルを保存する変数
// ref(null): nullを初期値とするリアクティブな変数を作成
// .value: refで作成した変数の値にアクセスするには.valueを使う

const previewUrl = ref(null)
// previewUrl: 画像プレビュー用のURLを保存する変数

const caption = ref('')
// caption: キャプションのテキストを保存する変数
// '': 空文字列を初期値とする

// ファイル選択時の処理
const handleFileChange = (event) => {
	// handleFileChange: ファイル選択時に実行される関数
	// (event): イベントオブジェクト（ファイル選択の情報が含まれる）

	const file = event.target.files[0]
	// event.target: イベントが発生した要素（ファイル入力要素）
	// .files: 選択されたファイルのリスト（FileListオブジェクト）
	// [0]: 配列の最初の要素（最初に選択されたファイル）を取得

	if (file) {
		// if (file): ファイルが選択された場合のみ処理を実行
		selectedFile.value = file
		// selectedFile.value: refで作成した変数の値に代入
		// file: 選択されたファイルオブジェクト

		// プレビュー用のURLを作成
		previewUrl.value = URL.createObjectURL(file)
		// URL.createObjectURL(): ファイルオブジェクトからBlob URLを生成
		// Blob URL: ブラウザのメモリ上に作成される一時的なURL
		// このURLを<img>のsrcに設定することで、画像をプレビューできる
	}
}

// フォーム送信時の処理（モック実装）
const handleSubmit = () => {
	// handleSubmit: フォーム送信時に実行される関数
	// TODO: ここでAPIを呼び出す（Step 6で実装）
	alert('投稿機能はまだ実装されていません（Step 6で実装します）')
	// alert(): ブラウザのアラートダイアログを表示（開発中の一時的な処理）
}
</script>
```

### 3-3) レイアウトの設定（app.vue）

共通レイアウトを設定します：

```vue
<template>
	<div class="min-h-screen bg-gray-50">
		<slot />
	</div>
</template>
```

## ✅ チェックリスト

- [✅ ] タイムライン画面（`pages/index.vue`）が作成された
- [✅ ] 3 列グリッドレイアウトで投稿が表示される
- [✅ ] モックデータが表示される
- [✅ ] 空状態が表示される
- [✅ ] 新規投稿画面（`pages/post/new.vue`）が作成された
- [✅ ] 画像選択とプレビューが動作する
- [✅ ] 「新規投稿」ボタンから投稿画面に遷移できる
- [✅ ] 投稿画面からトップページに戻れる

## 🎯 次のステップ

モック画面が完成したら、**step4.md** に進んでください。
（Backend セットアップ：FastAPI プロジェクトの作成）
