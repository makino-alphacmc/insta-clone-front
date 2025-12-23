# Step 6: Frontend と Backend の連携

## 📋 このステップでやること

フロントエンドのモックデータを削除し、実際の API と連携するように変更します。

- タイムライン画面で API から投稿を取得
- 新規投稿画面で API に投稿を送信

## ✅ 手順

### 6-1) タイムライン画面の更新（pages/index.vue）

※ 目的: モックを API 取得に置き換え、通信状態（ローディング/エラー/空）を先に整備して UX を守る。

`pages/index.vue` を編集して、API からデータを取得するように変更します：

```vue
<template>
	<div class="container mx-auto max-w-2xl py-8">
		<div class="mb-6 flex justify-between items-center">
			<h1 class="text-3xl font-bold">Instagram Clone</h1>
			<Button as-child>
				<NuxtLink to="/post/new">New Post</NuxtLink>
			</Button>
		</div>

		<!-- ローディング状態 -->
		<div v-if="pending" class="space-y-6">
			<Card v-for="i in 3" :key="i" class="p-0">
				<CardHeader>
					<Skeleton class="h-4 w-32" />
				</CardHeader>
				<CardContent class="p-0">
					<Skeleton class="w-full h-96" />
				</CardContent>
				<CardFooter>
					<Skeleton class="h-4 w-full" />
				</CardFooter>
			</Card>
		</div>

		<!-- エラー状態 -->
		<Alert v-else-if="error" variant="destructive">
			<AlertTitle>エラー</AlertTitle>
			<AlertDescription>
				投稿の取得に失敗しました。{{ error.message }}
			</AlertDescription>
		</Alert>

		<!-- 投稿がない場合 -->
		<div v-else-if="!posts || posts.length === 0" class="text-center py-12">
			<p class="text-gray-500">まだ投稿がありません</p>
			<Button as-child class="mt-4">
				<NuxtLink to="/post/new">最初の投稿を作成</NuxtLink>
			</Button>
		</div>

		<!-- 投稿一覧 -->
		<div v-else class="space-y-6">
			<Card v-for="post in posts" :key="post.id" class="p-0">
				<CardHeader>
					<div class="flex items-center gap-2">
						<Avatar>
							<AvatarFallback>U</AvatarFallback>
						</Avatar>
						<div>
							<p class="font-semibold">User</p>
							<p class="text-sm text-gray-500">
								{{ formatDate(post.created_at) }}
							</p>
						</div>
					</div>
				</CardHeader>
				<CardContent class="p-0">
					<img
						:src="post.image_url"
						:alt="post.caption || '投稿画像'"
						class="w-full object-cover"
					/>
				</CardContent>
				<CardFooter class="flex flex-col items-start gap-2">
					<p class="font-semibold">User</p>
					<p>{{ post.caption || '' }}</p>
				</CardFooter>
			</Card>
		</div>
	</div>
</template>

<script setup>
import { Card, CardContent, CardFooter, CardHeader } from '@/components/ui/card'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { Button } from '@/components/ui/button'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { Skeleton } from '@/components/ui/skeleton'

//
// 【意味】Nuxt のランタイム設定を取得: nuxt.config.ts で設定した環境変数を取得
// 【因果】useRuntimeConfig() で Nuxt の設定にアクセスできる
// 【学び】config.public.apiBase で、フロントエンドからアクセス可能な設定を取得
//
const config = useRuntimeConfig()
const apiBase = config.public.apiBase // 【意味】API のベース URL（例: "http://localhost:8000"）

//
// 【意味】API から投稿を取得: useFetch で非同期にデータを取得
// 【因果】useFetch は Nuxt の Composables で、自動的にローディング状態やエラーを管理してくれる
// 【学び】await useFetch() で、コンポーネントがマウントされる前にデータを取得（SSR 対応）
// 【学び】data: 取得したデータ、pending: ローディング中かどうか、error: エラー情報、refresh: 再取得関数
//
// 【データの流れ（useFetch 実行時）】:
//   1. useFetch(`${apiBase}/posts`) が呼ばれる
//      → apiBase = "http://localhost:8000"
//      → URL = "http://localhost:8000/posts"
//   2. pending = true になる（ローディング開始）
//   3. HTTP GET リクエストが送信される
//      → GET http://localhost:8000/posts
//   4. FastAPI がリクエストを受信
//      → GET /posts エンドポイントが実行される
//   5. データベースから投稿を取得
//      → SELECT * FROM posts ORDER BY created_at DESC
//      → [Post(id=1, ...), Post(id=2, ...), ...]
//   6. FastAPI が JSON に変換
//      → [{"id": 1, "image_url": "...", ...}, {"id": 2, ...}, ...]
//   7. HTTP レスポンスが返される
//      → Status: 200 OK
//      → Body: [{"id": 1, ...}, ...]
//   8. useFetch がレスポンスを受信
//      → data = [{"id": 1, ...}, ...] (JavaScript オブジェクト)
//      → pending = false (ローディング終了)
//   9. posts = data に代入される
//   10. v-for="post in posts" で各投稿が画面に表示される
//
const {
	data: posts, // 【データの流れ】取得した投稿データが入る → テンプレートで使用
	pending, // 【データの流れ】true → ローディング表示、false → データ表示
	error, // 【データの流れ】エラーがある場合にエラー情報が入る → エラー表示
	refresh, // 【データの流れ】再取得関数 → onMounted() で呼ばれる
} = await useFetch(`${apiBase}/posts`)

// 日付フォーマット関数
const formatDate = (dateString) => {
	const date = new Date(dateString)
	return date.toLocaleDateString('ja-JP', {
		year: 'numeric',
		month: 'long',
		day: 'numeric',
		hour: '2-digit',
		minute: '2-digit',
	})
}

//
// 【意味】ページが表示されたときに再取得: コンポーネントがマウントされた時に実行される
// 【因果】投稿後にタイムラインに戻ってきた場合、最新の投稿を表示するために再取得する
// 【学び】onMounted は Vue のライフサイクルフックで、コンポーネントが DOM に追加された時に実行される
//
// 【データの流れ（onMounted 実行時）】:
//   1. ページが表示される（コンポーネントがマウントされる）
//   2. onMounted() が呼ばれる
//   3. refresh() が実行される
//   4. useFetch() が再実行される
//   5. HTTP GET http://localhost:8000/posts が送信される
//   6. データベースから最新の投稿一覧を取得
//   7. posts が更新される
//   8. 画面が自動的に再レンダリングされる（新しい投稿が表示される）
//
onMounted(() => {
	refresh() // 【データの流れ】refresh() → useFetch() を再実行 → 最新データを取得 → 画面更新
})
</script>
```

### 6-2) 新規投稿画面の更新（pages/post/new.vue）

※ 目的: 実際の API に multipart で送信し、成功/失敗の分岐を UI に反映できるようにする（因果: 正常系と異常系の導線をこの段階で作る）。

`pages/post/new.vue` を編集して、実際に API に投稿を送信するように変更します：

```vue
<template>
	<div class="container mx-auto max-w-2xl py-8">
		<h1 class="text-3xl font-bold mb-6">New Post</h1>

		<Card>
			<CardHeader>
				<CardTitle>投稿を作成</CardTitle>
			</CardHeader>
			<CardContent>
				<form @submit.prevent="handleSubmit" class="space-y-4">
					<!-- 画像選択 -->
					<div class="space-y-2">
						<Label for="image">画像を選択</Label>
						<Input
							id="image"
							type="file"
							accept="image/*"
							@change="handleFileChange"
							:disabled="isSubmitting"
						/>
						<img
							v-if="previewUrl"
							:src="previewUrl"
							alt="プレビュー"
							class="w-full max-h-96 object-contain rounded"
						/>
					</div>

					<!-- キャプション -->
					<div class="space-y-2">
						<Label for="caption">キャプション</Label>
						<Textarea
							id="caption"
							v-model="caption"
							placeholder="キャプションを入力..."
							rows="4"
							:disabled="isSubmitting"
						/>
					</div>

					<!-- エラー表示 -->
					<Alert v-if="error" variant="destructive">
						<AlertTitle>エラー</AlertTitle>
						<AlertDescription>{{ error }}</AlertDescription>
					</Alert>

					<!-- ボタン -->
					<div class="flex gap-2">
						<Button type="submit" :disabled="!selectedFile || isSubmitting">
							<span v-if="isSubmitting">投稿中...</span>
							<span v-else>投稿する</span>
						</Button>
						<Button
							type="button"
							variant="outline"
							@click="$router.push('/')"
							:disabled="isSubmitting"
						>
							キャンセル
						</Button>
					</div>
				</form>
			</CardContent>
		</Card>
	</div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useToast } from '@/components/ui/sonner'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Textarea } from '@/components/ui/textarea'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

//
// 【意味】Nuxt/Vue の Composables を取得: ルーティング、トースト通知、設定の取得
// 【因果】useRouter() でページ遷移、useToast() で通知表示、useRuntimeConfig() で設定取得
//
const router = useRouter() // 【意味】ページ遷移を制御するルーター
const toast = useToast() // 【意味】トースト通知を表示する関数
const config = useRuntimeConfig() // 【意味】Nuxt の設定を取得
const apiBase = config.public.apiBase // 【意味】API のベース URL

//
// 【意味】リアクティブな状態変数: コンポーネントの状態を管理
// 【因果】ref() でラップすることで、値が変わると画面が自動更新される
//
const selectedFile = ref(null) // 【意味】選択されたファイルオブジェクト
const previewUrl = ref(null) // 【意味】画像プレビュー用の URL
const caption = ref('') // 【意味】キャプションのテキスト
const isSubmitting = ref(false) // 【意味】投稿処理中かどうか（ローディング状態）
const error = ref(null) // 【意味】エラーメッセージ

//
// 【意味】ファイル選択時の処理: ユーザーが画像を選択した時に実行される
// 【因果】@change="handleFileChange" でこの関数が呼ばれる
//
const handleFileChange = (event) => {
	const file = event.target.files[0] // 【意味】選択された最初のファイルを取得
	if (file) {
		//
		// 【意味】ファイルサイズチェック: 10MB を超える場合はエラー
		// 【因果】大きなファイルをアップロードすると、サーバーやストレージの負荷が高くなる
		// 【学び】10 * 1024 * 1024 で 10MB をバイト単位で表現（1MB = 1024KB = 1024*1024 バイト）
		//
		if (file.size > 10 * 1024 * 1024) {
			error.value = '画像サイズは10MB以下にしてください'
			return // 【意味】エラーの場合は処理を中断
		}
		selectedFile.value = file // 【意味】選択されたファイルを保存
		error.value = null // 【意味】エラーをクリア
		//
		// 【意味】プレビュー用の URL を作成: 選択した画像をすぐに表示できるようにする
		// 【因果】URL.createObjectURL() でファイルオブジェクトから Blob URL を生成
		// 【学び】この URL を <img> の src に設定することで、画像をプレビューできる
		//
		previewUrl.value = URL.createObjectURL(file)
	}
}

//
// 【意味】フォーム送信時の処理: 「投稿する」ボタンをクリックした時に実行される
// 【因果】@submit.prevent="handleSubmit" でこの関数が呼ばれる
// 【学び】async/await を使うことで、非同期処理（API 呼び出し）を同期的に書ける
//
const handleSubmit = async () => {
	// 【意味】バリデーション: ファイルが選択されていない場合はエラー
	if (!selectedFile.value) {
		error.value = '画像を選択してください'
		return
	}

	// 【意味】ローディング状態を開始: 投稿処理中であることを示す
	isSubmitting.value = true
	error.value = null // エラーをクリア

	try {
		//
		// 【データの流れ】selectedFile.value = File { name: "image.jpg", ... }
		//   caption.value = "投稿のキャプション"
		//   ↓ new FormData()
		//   ↓ formData = FormData オブジェクト（空）
		//   ↓ formData.append('file', selectedFile.value)
		//   ↓ formData.append('caption', caption.value)
		//   ↓ formData = { file: File {...}, caption: "投稿のキャプション" }
		// 【意味】FormData の作成: ファイルとテキストを multipart/form-data 形式で送信するために使用
		// 【因果】FormData を使うことで、ファイルアップロードに対応したリクエストを送れる
		// 【学び】append() でキーと値を追加（'file' と 'caption' を追加）
		//
		const formData = new FormData()
		formData.append('file', selectedFile.value) // 【意味】画像ファイルを追加
		formData.append('caption', caption.value || '') // 【意味】キャプションを追加（空の場合は空文字列）

		//
		// 【データの流れ】formData = { file: File {...}, caption: "..." }
		//   ↓ fetch(`${apiBase}/posts`, { method: 'POST', body: formData })
		//   ↓ HTTP POST リクエストが送信される
		//   ↓ Content-Type: multipart/form-data (自動設定)
		//   ↓ リクエストボディ: --boundary123
		//                      Content-Disposition: form-data; name="file"; filename="image.jpg"
		//                      Content-Type: image/jpeg
		//                      [バイナリデータ]
		//                      --boundary123
		//                      Content-Disposition: form-data; name="caption"
		//                      [キャプションテキスト]
		//   ↓ FastAPI がリクエストを受信
		//   ↓ POST /posts エンドポイントが実行される
		//   ↓ file = UploadFile {...}, caption = "..." として受け取られる
		//   ↓ upload_image(file) → Supabase Storage にアップロード
		//   ↓ image_url = "https://xxxxx.supabase.co/storage/.../a1b2c3d4.jpg"
		//   ↓ データベースに保存: INSERT INTO posts (image_url, caption, ...) VALUES (...)
		//   ↓ レスポンス: {"id": 1, "image_url": "https://...", "caption": "...", "created_at": "..."}
		//   ↓ response = Response { ok: true, status: 200, ... }
		// 【意味】API に POST リクエスト: 投稿データをサーバーに送信
		// 【因果】fetch() で HTTP リクエストを送信し、await でレスポンスを待つ
		// 【学び】method: 'POST' で POST メソッド、body: formData でフォームデータを送信
		// 【学び】Content-Type は自動的に multipart/form-data に設定される
		//
		const response = await fetch(`${apiBase}/posts`, {
			method: 'POST',
			body: formData,
		})

		//
		// 【データの流れ】response = Response { ok: false, status: 500, ... } (エラー時)
		//   ↓ response.ok = false
		//   ↓ response.json() → {"detail": "エラーメッセージ"}
		//   ↓ errorData = { detail: "エラーメッセージ" }
		//   ↓ throw new Error("エラーメッセージ")
		//   ↓ catch ブロックに飛ぶ
		// 【意味】エラーレスポンスの処理: サーバーからエラーが返された場合
		// 【因果】response.ok が false の場合（ステータスコードが 200-299 以外）はエラー
		// 【学び】response.json() でエラーメッセージを取得し、適切なエラーを投げる
		//
		if (!response.ok) {
			const errorData = await response
				.json()
				.catch(() => ({ detail: '投稿に失敗しました' }))
			throw new Error(errorData.detail || '投稿に失敗しました')
		}

		//
		// 【データの流れ】response = Response { ok: true, status: 200, ... } (成功時)
		//   ↓ toast.success('投稿が完了しました！')
		//   ↓ トースト通知が表示される
		//   ↓ router.push('/')
		//   ↓ トップページ（タイムライン）に遷移
		//   ↓ タイムライン画面が表示される
		//   ↓ onMounted() で refresh() が呼ばれる
		//   ↓ useFetch() が再実行される
		//   ↓ 新しい投稿がタイムラインに表示される
		// 【意味】成功時の処理: 投稿が完了したことをユーザーに通知し、タイムラインに戻る
		// 【因果】toast.success() で成功メッセージを表示、router.push('/') でトップページに遷移
		// 【学び】ユーザーに操作の結果を明確に伝えることで、UX が向上する
		//
		toast.success('投稿が完了しました！')
		router.push('/') // 【意味】トップページ（タイムライン）に遷移
	} catch (err) {
		//
		// 【意味】エラー処理: 例外が発生した場合にエラーメッセージを表示
		// 【因果】try-catch でエラーを捕捉し、ユーザーに分かりやすいメッセージを表示
		// 【学び】err.message でエラーメッセージを取得、なければデフォルトメッセージを表示
		//
		error.value = err.message || '投稿に失敗しました'
		toast.error('投稿に失敗しました')
	} finally {
		//
		// 【意味】ローディング状態を解除: 成功・失敗に関わらず、必ず実行される
		// 【因果】finally ブロックで、エラーが発生しても isSubmitting を false に戻す
		// 【学び】これにより、エラー後も再度投稿を試みられるようになる
		//
		isSubmitting.value = false
	}
}
</script>
```

### 6-3) 動作確認

※ 目的: フロント/バック双方を起動し、投稿 → 一覧反映までの一連の流れが成立するかを検証する。

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
   - 「New Post」ボタンをクリック
   - 画像を選択してキャプションを入力
   - 「投稿する」をクリック
   - トップページに戻り、投稿が表示されることを確認

## ✅ チェックリスト

- [ ] タイムライン画面で API から投稿を取得できる
- [ ] ローディング状態（Skeleton）が表示される
- [ ] エラー状態が適切に表示される
- [ ] 投稿がない場合の空状態が表示される
- [ ] 新規投稿画面で画像とキャプションを送信できる
- [ ] 投稿成功時にトーストが表示される
- [ ] 投稿後にタイムラインに戻り、新しい投稿が表示される
- [ ] エラーハンドリングが適切に動作する

## 🎯 次のステップ

フロントエンドとバックエンドの連携が完了したら、**step7.md** に進んでください。
（Docker 化：コンテナ化と docker-compose の設定）
