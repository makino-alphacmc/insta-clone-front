# Step 6: Frontend と Backend の連携

## 📋 このステップでやること

フロントエンドのモックデータを削除し、実際の API と連携するように変更します。

- タイムライン画面で API から投稿を取得（3列レイアウト）
- 新規投稿画面で API に投稿を送信
- 各状態の実装（ローディング、エラー、空状態、成功、失敗）

## ✅ 手順

### 6-1) タイムライン画面の更新（pages/index.vue）

モックデータをAPI取得に置き換え、3列レイアウトと各状態を実装します。

`pages/index.vue` を編集します：

```vue
<template>
	<!-- 背景とコンテナ（PC向け） -->
	<div class="min-h-screen bg-gray-50">
		<!-- ヘッダー部分 -->
		<header class="bg-white border-b border-gray-200">
			<div class="container mx-auto max-w-7xl px-4 py-4">
				<div class="flex justify-between items-center">
					<h1 class="text-2xl font-bold">Instagram Clone</h1>
					<!-- 新規投稿ボタン -->
					<Button as-child>
						<NuxtLink to="/post/new">新規投稿</NuxtLink>
					</Button>
				</div>
			</div>
		</header>

		<!-- メインコンテンツ -->
		<main class="container mx-auto max-w-7xl px-4 py-8">
			<!-- ローディング状態（Skeleton表示） -->
			<div v-if="pending" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
				<!-- v-if="pending": pendingがtrueの場合のみ表示（ローディング中） -->
				<!-- grid: グリッドレイアウトを有効化 -->
				<!-- grid-cols-1: デフォルトで1列（スマホ向け） -->
				<!-- md:grid-cols-2: 中サイズ画面（768px以上）で2列 -->
				<!-- lg:grid-cols-3: 大サイズ画面（1024px以上）で3列 -->
				<!-- gap-6: グリッドアイテム間の間隔を24pxに設定 -->
				
				<Card v-for="i in 6" :key="i" class="overflow-hidden">
					<!-- v-for="i in 6": 6回繰り返し（ローディング用のスケルトンを6個表示） -->
					<!-- :key="i": 各要素を識別するためのキー -->
					<!-- overflow-hidden: はみ出した内容を隠す -->
					<CardHeader class="pb-3">
						<!-- pb-3: 下側にパディング12px -->
						<Skeleton class="h-4 w-32" />
						<!-- Skeleton: ローディング中のプレースホルダー（shadcn-vueのコンポーネント） -->
						<!-- h-4: 高さを16px（1rem）に設定 -->
						<!-- w-32: 幅を128px（8rem）に設定 -->
					</CardHeader>
					<CardContent class="p-0">
						<!-- p-0: パディングを0に設定 -->
						<Skeleton class="w-full aspect-square" />
						<!-- w-full: 幅を100%に設定 -->
						<!-- aspect-square: アスペクト比を1:1（正方形）に設定 -->
					</CardContent>
					<CardFooter class="pt-3">
						<!-- pt-3: 上側にパディング12px -->
						<Skeleton class="h-4 w-full" />
						<!-- w-full: 幅を100%に設定 -->
					</CardFooter>
				</Card>
			</div>

			<!-- エラー状態（モーダル表示） -->
			<div v-else-if="error" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
				<!-- v-else-if="error": pendingがfalseでerrorが存在する場合に表示 -->
				<!-- fixed: 固定位置（スクロールしても位置が変わらない） -->
				<!-- inset-0: 上下左右を0に設定（画面全体を覆う） -->
				<!-- bg-black: 背景色を黒に設定 -->
				<!-- bg-opacity-50: 背景の透明度を50%に設定 -->
				<!-- flex: フレックスボックスレイアウト -->
				<!-- items-center: 縦方向の中央揃え -->
				<!-- justify-center: 横方向の中央揃え -->
				<!-- z-50: z-indexを50に設定（他の要素の上に表示） -->
				
				<Card class="w-full max-w-md mx-4">
					<!-- w-full: 幅を100%に設定 -->
					<!-- max-w-md: 最大幅を448px（28rem）に制限 -->
					<!-- mx-4: 左右にマージン16px -->
					<CardHeader class="text-center">
						<!-- text-center: テキストを中央揃え -->
						<div class="w-16 h-16 mx-auto mb-4 bg-red-500 rounded-full flex items-center justify-center">
							<!-- w-16: 幅を64px（4rem）に設定 -->
							<!-- h-16: 高さを64px（4rem）に設定 -->
							<!-- bg-red-500: 背景色を赤に設定 -->
							<!-- rounded-full: 角を完全に丸くする（円形） -->
							<svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<!-- w-8: 幅を32px（2rem）に設定 -->
								<!-- h-8: 高さを32px（2rem）に設定 -->
								<!-- text-white: テキスト（SVGの色）を白に設定 -->
							</svg>
						</div>
						<CardTitle>エラーが発生しました</CardTitle>
					</CardHeader>
					<CardContent class="text-center">
						<p class="text-gray-600 mb-2">投稿の取得に失敗しました。</p>
						<!-- text-gray-600: テキストの色をグレーに設定 -->
						<p class="text-sm text-gray-500">ネットワークエラーが発生した可能性があります。</p>
					</CardContent>
					<CardFooter class="justify-center">
						<!-- justify-center: 横方向の中央揃え -->
						<Button @click="refresh()">再試行</Button>
						<!-- @click: クリック時に実行される関数 -->
						<!-- refresh(): データを再取得する関数 -->
					</CardFooter>
				</Card>
			</div>

			<!-- 空状態 -->
			<div v-else-if="!posts || posts.length === 0" class="text-center py-20">
				<div class="mb-4">
					<div class="w-20 h-20 mx-auto bg-gray-100 rounded-lg flex items-center justify-center">
						<svg class="w-10 h-10 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
						</svg>
					</div>
				</div>
				<p class="text-lg text-gray-500 mb-2">まだ投稿がありません</p>
				<p class="text-sm text-gray-400 mb-6">最初の投稿を作成して、みんなとシェアしましょう</p>
				<Button as-child>
					<NuxtLink to="/post/new">最初の投稿を作成</NuxtLink>
				</Button>
			</div>

			<!-- 投稿一覧（3列グリッドレイアウト） -->
			<div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
				<Card v-for="post in posts" :key="post.id" class="overflow-hidden">
					<!-- ヘッダー部分（アバター + ユーザー名 + 日時） -->
					<CardHeader class="pb-3">
						<div class="flex items-center gap-3">
							<Avatar>
								<AvatarFallback>{{ post.username?.charAt(0) || 'U' }}</AvatarFallback>
							</Avatar>
							<div class="flex-1">
								<p class="font-semibold text-sm">{{ post.username || 'User' }}</p>
								<p class="text-xs text-gray-500">{{ formatDate(post.created_at) }}</p>
							</div>
						</div>
					</CardHeader>

					<!-- 画像エリア -->
					<CardContent class="p-0">
						<img
							:src="post.image_url"
							:alt="post.caption || '投稿画像'"
							class="w-full aspect-square object-cover"
						/>
					</CardContent>

					<!-- フッター部分（キャプション） -->
					<CardFooter class="flex flex-col items-start gap-2 pt-3">
						<p class="font-semibold text-sm">{{ post.username || 'User' }}</p>
						<p class="text-sm text-gray-700">{{ post.caption || '' }}</p>
					</CardFooter>
				</Card>
			</div>
		</main>
	</div>
</template>

<script setup>
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { Button } from '@/components/ui/button'
import { Skeleton } from '@/components/ui/skeleton'

// Nuxtのランタイム設定を取得
const config = useRuntimeConfig()
// useRuntimeConfig(): Nuxtのランタイム設定を取得する関数（Composable）
// nuxt.config.tsで設定した環境変数にアクセスできる

const apiBase = config.public.apiBase
// config.public.apiBase: フロントエンドからアクセス可能な設定を取得
// apiBase: APIのベースURL（例: "http://localhost:8000"）

// APIから投稿を取得
const {
	data: posts,
	// data: 取得したデータ（投稿の配列）
	// posts: データを格納する変数名（リネーム）
	pending,
	// pending: ローディング中かどうか（true/false）
	error,
	// error: エラー情報（エラーが発生した場合に設定される）
	refresh,
	// refresh: データを再取得する関数
} = await useFetch(`${apiBase}/posts`)
// await: 非同期処理の完了を待つ
// useFetch(): NuxtのComposableで、APIからデータを取得する関数
// `${apiBase}/posts`: テンプレートリテラル（文字列を動的に結合）
// 例: "http://localhost:8000/posts"

// 日付を日本語形式に変換する関数
const formatDate = (dateString) => {
	const date = new Date(dateString)
	// new Date(dateString): 文字列をDateオブジェクトに変換
	return date.toLocaleDateString('ja-JP', {
		// toLocaleDateString(): 日時をロケール（言語・地域）に応じた文字列に変換
		// 'ja-JP': 日本語（日本）のロケールを指定
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

// ページが表示されたときに再取得
onMounted(() => {
	// onMounted(): Vueのライフサイクルフック（コンポーネントがDOMに追加された時に実行）
	// () => {}: アロー関数（関数を定義）
	refresh()
	// refresh(): データを再取得する関数（最新の投稿を表示するため）
})
</script>
```

### 6-2) 新規投稿画面の更新（pages/post/new.vue）

実際のAPIに投稿を送信し、成功/失敗の状態を実装します。

`pages/post/new.vue` を編集します：

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
						<!-- 画像選択セクション -->
						<div class="space-y-2">
							<Label for="image">画像を選択</Label>
							<p class="text-sm text-gray-500">JPEG、PNG形式の画像をアップロードできます（最大10MB）</p>
							
							<!-- ファイル入力エリア -->
							<div class="border-2 border-dashed border-gray-300 rounded-lg p-12 text-center">
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
									:disabled="isSubmitting"
									<!-- :disabled: 投稿処理中はファイル選択を無効化 -->
								/>
								<label for="image" class="cursor-pointer">
									<div class="space-y-4">
										<Button type="button" variant="default" :disabled="isSubmitting">ファイルを選択</Button>
										<p class="text-sm text-gray-500">または、ここにドラッグ&ドロップ</p>
									</div>
								</label>
							</div>

							<!-- プレビュー画像 -->
							<div v-if="previewUrl" class="mt-4">
								<img
									:src="previewUrl"
									alt="プレビュー"
									class="w-full max-h-96 object-contain rounded-lg border border-gray-200"
								/>
							</div>
						</div>

						<!-- エラーアラート（フォーム内表示） -->
						<Alert v-if="error" variant="destructive">
							<!-- v-if="error": errorが存在する場合のみ表示 -->
							<!-- variant="destructive": 破壊的なスタイル（赤系のエラー表示） -->
							<AlertTitle>エラー</AlertTitle>
							<!-- AlertTitle: アラートのタイトル -->
							<AlertDescription>
								{{ error }}
								<!-- {{ error }}: エラーメッセージを表示 -->
							</AlertDescription>
							<!-- AlertDescription: アラートの説明文 -->
						</Alert>

						<!-- キャプション入力セクション -->
						<div class="space-y-2">
							<Label for="caption">キャプション</Label>
							<p class="text-sm text-gray-500">投稿に説明文を追加できます（任意）</p>
							<Textarea
								id="caption"
								<!-- id="caption": 要素の識別子（Labelのforと関連付け） -->
								v-model="caption"
								<!-- v-model: 双方向データバインディング（入力値と変数を自動同期） -->
								<!-- caption: 入力値が自動的にcaption変数に保存される -->
								placeholder="キャプションを入力..."
								<!-- placeholder: 入力例を表示（ユーザーの理解を助ける） -->
								rows="4"
								<!-- rows: テキストエリアの行数を4行に設定 -->
								:maxlength="500"
								<!-- :maxlength: 最大文字数を500文字に制限 -->
								:disabled="isSubmitting"
								<!-- :disabled: 投稿処理中は入力欄を無効化 -->
							/>
							<p class="text-xs text-gray-400 text-right">{{ caption.length }} / 500</p>
						</div>

						<!-- ボタンエリア -->
						<div class="flex gap-4">
							<!-- flex: フレックスボックスレイアウト -->
							<!-- gap-4: 子要素間の間隔を16pxに設定 -->
							<Button type="submit" :disabled="!selectedFile || isSubmitting" class="flex-1">
								<!-- type="submit": フォーム送信ボタン -->
								<!-- :disabled: ボタンの有効/無効を動的に設定 -->
								<!-- !selectedFile: selectedFileがnull/falsyの場合にtrue（ボタンが無効化される） -->
								<!-- ||: 論理OR演算子（左側がtrueの場合は左側を返す、falseの場合は右側を評価） -->
								<!-- isSubmitting: 投稿処理中の場合にtrue（ボタンが無効化される） -->
								<!-- flex-1: 残りのスペースを全て使用 -->
								<span v-if="isSubmitting">投稿中...</span>
								<!-- v-if="isSubmitting": isSubmittingがtrueの場合に表示 -->
								<span v-else>投稿する</span>
								<!-- v-else: v-ifの条件がfalseの場合に表示 -->
							</Button>
							<Button type="button" variant="outline" @click="$router.push('/')" :disabled="isSubmitting" class="flex-1">
								<!-- type="button": フォーム送信をしないボタン -->
								<!-- variant="outline": アウトラインスタイルのボタン -->
								<!-- @click: クリック時に実行される関数 -->
								<!-- $router.push('/'): Nuxtのルーターでトップページに遷移 -->
								<!-- :disabled: 投稿処理中はボタンを無効化 -->
								キャンセル
							</Button>
						</div>
					</form>
				</CardContent>
			</Card>
		</main>
	</div>

	<!-- 投稿成功モーダル -->
	<div v-if="showSuccessModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
		<!-- v-if="showSuccessModal": showSuccessModalがtrueの場合のみ表示 -->
		<!-- fixed: 固定位置（スクロールしても位置が変わらない） -->
		<!-- inset-0: 上下左右を0に設定（画面全体を覆う） -->
		<!-- bg-black: 背景色を黒に設定 -->
		<!-- bg-opacity-50: 背景の透明度を50%に設定 -->
		<!-- flex: フレックスボックスレイアウト -->
		<!-- items-center: 縦方向の中央揃え -->
		<!-- justify-center: 横方向の中央揃え -->
		<!-- z-50: z-indexを50に設定（他の要素の上に表示） -->
		
		<Card class="w-full max-w-md mx-4">
			<!-- w-full: 幅を100%に設定 -->
			<!-- max-w-md: 最大幅を448px（28rem）に制限 -->
			<!-- mx-4: 左右にマージン16px -->
			<CardHeader class="text-center">
				<!-- text-center: テキストを中央揃え -->
				<div class="w-16 h-16 mx-auto mb-4 bg-green-500 rounded-full flex items-center justify-center">
					<!-- w-16: 幅を64px（4rem）に設定 -->
					<!-- h-16: 高さを64px（4rem）に設定 -->
					<!-- bg-green-500: 背景色を緑に設定 -->
					<!-- rounded-full: 角を完全に丸くする（円形） -->
					<svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<!-- w-8: 幅を32px（2rem）に設定 -->
						<!-- h-8: 高さを32px（2rem）に設定 -->
						<!-- text-white: テキスト（SVGの色）を白に設定 -->
					</svg>
				</div>
				<CardTitle>投稿が完了しました！</CardTitle>
			</CardHeader>
			<CardContent class="text-center">
				<p class="text-gray-600 mb-2">投稿が正常にアップロードされました</p>
				<!-- text-gray-600: テキストの色をグレーに設定 -->
				<p class="text-sm text-gray-500">タイムラインに表示されます</p>
			</CardContent>
			<CardFooter class="justify-center">
				<!-- justify-center: 横方向の中央揃え -->
				<Button @click="handleSuccessClose">OK</Button>
				<!-- @click: クリック時に実行される関数 -->
				<!-- handleSuccessClose(): モーダルを閉じてタイムラインに戻る関数 -->
			</CardFooter>
		</Card>
	</div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useToast } from '@/components/ui/sonner'
import { Card, CardContent, CardHeader, CardTitle, CardFooter } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Textarea } from '@/components/ui/textarea'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

// Nuxt/VueのComposablesを取得
const router = useRouter()
// useRouter(): Vue Routerのルーターインスタンスを取得（ページ遷移に使用）
const toast = useToast()
// useToast(): トースト通知を表示する関数を取得
const config = useRuntimeConfig()
// useRuntimeConfig(): Nuxtのランタイム設定を取得
const apiBase = config.public.apiBase
// apiBase: APIのベースURL

// リアクティブな状態変数
const selectedFile = ref(null)
// selectedFile: 選択されたファイルオブジェクトを保存
const previewUrl = ref(null)
// previewUrl: 画像プレビュー用のURL（Blob URL）
const caption = ref('')
// caption: キャプションのテキスト
const isSubmitting = ref(false)
// isSubmitting: 投稿処理中かどうか（ローディング状態の管理）
const error = ref(null)
// error: エラーメッセージ（エラーが発生した場合に設定）
const showSuccessModal = ref(false)
// showSuccessModal: 成功モーダルの表示/非表示を管理

// ファイル選択時の処理
const handleFileChange = (event) => {
	// handleFileChange: ファイル選択時に実行される関数
	// (event): イベントオブジェクト
	const file = event.target.files[0]
	// event.target.files[0]: 選択された最初のファイルを取得
	if (file) {
		// if (file): ファイルが選択された場合のみ処理を実行
		// ファイルサイズチェック（10MB以下）
		if (file.size > 10 * 1024 * 1024) {
			// file.size: ファイルサイズ（バイト単位）
			// 10 * 1024 * 1024: 10MBをバイト単位で表現（10 × 1024 × 1024 = 10,485,760バイト）
			error.value = '画像サイズは10MB以下にしてください'
			// error.value: エラーメッセージを設定
			return
			// return: 関数の実行を中断（以降の処理を実行しない）
		}
		selectedFile.value = file
		error.value = null
		// null: エラーをクリア（エラー状態を解除）
		// プレビュー用のURLを作成
		previewUrl.value = URL.createObjectURL(file)
		// URL.createObjectURL(): ファイルオブジェクトからBlob URLを生成
	}
}

// フォーム送信時の処理
const handleSubmit = async () => {
	// async: 非同期関数を定義（awaitを使用するため）
	if (!selectedFile.value) {
		// !selectedFile.value: selectedFileがnull/falsyの場合
		error.value = '画像を選択してください'
		return
	}

	isSubmitting.value = true
	// ローディング状態を開始（ボタンを無効化するため）
	error.value = null
	// エラーをクリア

	try {
		// try: エラーが発生する可能性のある処理を囲む
		// FormDataの作成
		const formData = new FormData()
		// new FormData(): ファイルアップロード用のデータ形式を作成（multipart/form-data）
		// FormData: ブラウザのAPIで、ファイルとテキストを一緒に送信できる
		formData.append('file', selectedFile.value)
		// append(): FormDataにデータを追加
		// 'file': サーバー側で受け取る際のキー名（FastAPIのfileパラメータと対応）
		// selectedFile.value: 選択されたファイルオブジェクト
		formData.append('caption', caption.value || '')
		// 'caption': サーバー側で受け取る際のキー名（FastAPIのcaptionパラメータと対応）
		// caption.value || '': captionが空の場合は空文字列を使用

		// APIにPOSTリクエスト
		const response = await fetch(`${apiBase}/posts`, {
			// await fetch(): HTTPリクエストを送信（非同期処理）
			// fetch(): ブラウザのAPIで、HTTPリクエストを送信する関数
			// `${apiBase}/posts`: リクエスト先のURL（テンプレートリテラルで動的に結合）
			// 例: "http://localhost:8000/posts"
			method: 'POST',
			// method: HTTPメソッドをPOSTに指定
			body: formData,
			// body: リクエストボディ（送信するデータ）
			// FormDataを指定すると、Content-Typeが自動的にmultipart/form-dataに設定される
		})

		// エラーレスポンスの処理
		if (!response.ok) {
			// response.ok: レスポンスのステータスが200-299の場合にtrue
			// !response.ok: エラーレスポンスの場合（400番台や500番台）
			const errorData = await response.json().catch(() => ({ detail: '投稿に失敗しました' }))
			// response.json(): レスポンスをJSON形式で解析
			// .catch(): JSON解析に失敗した場合の処理（デフォルト値を返す）
			// { detail: '投稿に失敗しました' }: デフォルトのエラーオブジェクト
			throw new Error(errorData.detail || '投稿に失敗しました')
			// throw new Error(): エラーを発生させる（catchブロックに処理が移る）
			// errorData.detail: FastAPIから返されたエラーメッセージ
			// || '投稿に失敗しました': detailが存在しない場合のデフォルトメッセージ
		}

		// 成功時の処理
		showSuccessModal.value = true
		// showSuccessModal.value: 成功モーダルを表示
		toast.success('投稿が完了しました！')
		// toast.success(): 成功メッセージをトーストで表示（画面の右上などに表示される）
	} catch (err) {
		// catch: tryブロックでエラーが発生した場合の処理
		// err: 発生したエラーオブジェクト
		// エラー処理
		error.value = err.message || '投稿に失敗しました'
		// err.message: エラーメッセージを取得
		// error.value: エラーメッセージを変数に保存（テンプレートで表示される）
		toast.error('投稿に失敗しました')
		// toast.error(): エラーメッセージをトーストで表示
	} finally {
		// finally: 成功・失敗に関わらず必ず実行される処理
		isSubmitting.value = false
		// ローディング状態を解除（ボタンを再有効化）
	}
}

// 成功モーダルを閉じてタイムラインに戻る
const handleSuccessClose = () => {
	showSuccessModal.value = false
	// 成功モーダルを非表示
	router.push('/')
	// router.push('/'): トップページ（タイムライン）に遷移
}
</script>
```

### 6-3) 動作確認

フロントエンドとバックエンドの連携を確認します。

1. **バックエンドサーバーを起動**（別のターミナルで）:

```bash
cd ~/work/insta-clone-api
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

2. **フロントエンドサーバーを起動**（別のターミナルで）:

```bash
cd ~/work/insta-clone-front
npm run dev
```

3. **ブラウザで動作確認**:
   - `http://localhost:3000` を開く
   - ローディング状態（Skeleton）が表示されることを確認
   - 投稿がない場合は空状態が表示されることを確認
   - 「新規投稿」ボタンをクリック
   - 画像を選択してキャプションを入力
   - 「投稿する」をクリック
   - 成功モーダルが表示されることを確認
   - タイムラインに戻り、投稿が3列レイアウトで表示されることを確認
   - エラーが発生した場合はエラーアラートが表示されることを確認

## ✅ チェックリスト

- [ ] タイムライン画面で API から投稿を取得できる
- [ ] 3列グリッドレイアウトで投稿が表示される
- [ ] ローディング状態（Skeleton）が表示される
- [ ] エラー状態（モーダル）が適切に表示される
- [ ] 投稿がない場合の空状態が表示される
- [ ] 新規投稿画面で画像とキャプションを送信できる
- [ ] 投稿成功時にモーダルとトーストが表示される
- [ ] 投稿後にタイムラインに戻り、新しい投稿が表示される
- [ ] エラーハンドリングが適切に動作する（フォーム内アラート表示）

## 🎯 次のステップ

フロントエンドとバックエンドの連携が完了したら、**step7.md** に進んでください。
（Docker 化：コンテナ化と docker-compose の設定）
