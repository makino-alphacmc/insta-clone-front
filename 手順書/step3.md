# Step 3: Frontend モック画面の作成

## 📋 このステップでやること

API がまだない状態でも、UI を確認できるようにモック画面を作成します。

- タイムライン画面（投稿一覧）
- 新規投稿画面
- ルーティングとナビゲーション

## ✅ 手順

### 3-1) タイムライン画面（pages/index.vue）

※ 目的: モックデータを画面に流し込み、一覧 UI の構造とコンポーネント配置を先に固めることで、後続の API 連携時の差し替えコストを下げる。

`pages/index.vue` を編集します：

```vue
<template>
	<!-- 
		【意味】コンテナ: 画面全体の幅を制限し、中央揃えにする
		【因果】max-w-2xl で最大幅を設定することで、大画面でも読みやすい幅に保つ
		【学び】Tailwind の container クラスは自動的に中央揃えとパディングを適用
	-->
	<div class="container mx-auto max-w-2xl py-8">
		<!-- 
			【意味】ヘッダー部分: タイトルと新規投稿ボタンを横並びに配置
			【因果】flex justify-between で左右に配置し、見た目を整理
		-->
		<div class="mb-6 flex justify-between items-center">
			<!-- アプリのタイトル -->
			<h1 class="text-3xl font-bold">Instagram Clone</h1>
			<!-- 
				【意味】as-child プロップ: Button コンポーネントのラッパーを生成せず、子要素を直接使用
				【因果】NuxtLink を Button のスタイルで表示できる（二重の要素を避ける）
				【学び】shadcn-vue の Button コンポーネントの特殊な使い方
			-->
			<Button as-child>
				<!-- 
					【意味】NuxtLink: Nuxt のルーティング機能を使ったリンク
					【因果】to="/post/new" で /post/new ページに遷移する
					【学び】通常の <a> タグではなく NuxtLink を使うことで、SPA の高速な遷移が可能
				-->
				<NuxtLink to="/post/new">New Post</NuxtLink>
			</Button>
		</div>

		<!-- 
			【意味】条件付きレンダリング: 投稿が0件の場合のみ表示
			【因果】v-if で posts.length === 0 をチェックし、空状態を表示
			【学び】v-if は条件が false の場合、DOM から要素を削除する（v-show は非表示にするだけ）
		-->
		<div v-if="posts.length === 0" class="text-center py-12">
			<p class="text-gray-500">まだ投稿がありません</p>
		</div>

		<!-- 
			【意味】投稿一覧: 投稿がある場合に表示
			【因果】v-else で v-if の条件が false の場合に表示
			【学び】space-y-6 は各要素の間に縦方向のスペースを自動的に追加
		-->
		<div v-else class="space-y-6">
			<!-- 
				【意味】v-for: 配列の各要素に対して要素を繰り返し生成
				【因果】:key="post.id" で Vue が各要素を識別できるようにする（パフォーマンス向上）
				【学び】key がないと、リストの順序が変わった時に正しく更新されない可能性がある
			-->
			<Card v-for="post in posts" :key="post.id" class="p-0">
				<!-- 
					【意味】CardHeader: カードの上部エリア（ユーザー情報を表示）
					【因果】shadcn-vue の Card コンポーネントを使うことで、統一されたデザインを実現
				-->
				<CardHeader>
					<!-- 
						【意味】flex: 要素を横並びに配置
						【因果】items-center で縦方向の中央揃え、gap-2 で要素間のスペースを設定
					-->
					<div class="flex items-center gap-2">
						<!-- 
							【意味】Avatar: ユーザーのアイコンを表示するコンポーネント
							【因果】インスタ風の UI を実現するために使用
						-->
						<Avatar>
							<!-- 
								【意味】AvatarFallback: 画像がない場合の代替表示
								【因果】post.username?.charAt(0) でユーザー名の最初の文字を取得
								【学び】?. はオプショナルチェーン（username が null/undefined の場合エラーにならない）
								【学び】|| 'U' は左側が falsy の場合に 'U' を返す（デフォルト値）
							-->
							<AvatarFallback>{{
								post.username?.charAt(0) || 'U'
							}}</AvatarFallback>
						</Avatar>
						<div>
							<!-- 
								【意味】ユーザー名を表示（存在しない場合は 'User'）
								【因果】{{ }} で JavaScript の式を評価して表示（Vue のテンプレート構文）
							-->
							<p class="font-semibold">{{ post.username || 'User' }}</p>
							<!-- 
								【意味】投稿日時を表示
								【因果】formatDate 関数で日付を日本語形式に変換して表示
							-->
							<p class="text-sm text-gray-500">
								{{ formatDate(post.created_at) }}
							</p>
						</div>
					</div>
				</CardHeader>
				<!-- 
					【意味】CardContent: カードのメインコンテンツエリア（画像を表示）
					【因果】p-0 でパディングを0にすることで、画像を端まで表示
				-->
				<CardContent class="p-0">
					<!-- 
						【意味】投稿画像を表示
						【因果】:src で動的に画像URLを設定、:alt でアクセシビリティのため代替テキストを設定
						【学び】w-full で幅100%、object-cover でアスペクト比を保ちながら領域を埋める
					-->
					<img
						:src="post.image_url"
						:alt="post.caption"
						class="w-full object-cover"
					/>
				</CardContent>
				<!-- 
					【意味】CardFooter: カードの下部エリア（キャプションを表示）
					【因果】flex flex-col で縦並び、items-start で左揃え、gap-2 で要素間のスペースを設定
				-->
				<CardFooter class="flex flex-col items-start gap-2">
					<p class="font-semibold">{{ post.username || 'User' }}</p>
					<!-- 投稿のキャプション（説明文）を表示 -->
					<p>{{ post.caption }}</p>
				</CardFooter>
			</Card>
		</div>
	</div>
</template>

<script setup>
//
// 【意味】コンポーネントのインポート: shadcn-vue の UI コンポーネントを使用
// 【因果】@/ は Nuxt のエイリアスで、app/ ディレクトリを指す
// 【学び】import で他のファイルから機能を読み込む（再利用性を高める）
//
import { Card, CardContent, CardFooter, CardHeader } from '@/components/ui/card'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { Button } from '@/components/ui/button'

//
// 【意味】モックデータ: 実際の API ができるまでの仮のデータ
// 【因果】後で Step 6 で API から取得したデータに置き換える
// 【学び】開発中はモックデータを使うことで、UI の実装と API の実装を並行して進められる
//
// 【データの流れ】この posts 配列は以下のように使われる：
//   1. v-for="post in posts" で各要素が繰り返される
//   2. post.image_url → <img :src="post.image_url"> に流れて画像を表示
//   3. post.caption → <p>{{ post.caption }}</p> に流れてテキストを表示
//   4. post.created_at → formatDate(post.created_at) → 日本語形式の文字列 → 画面に表示
//   5. post.id → :key="post.id" に流れて Vue が要素を識別
//
const posts = [
	{
		// 【データの流れ】id: 1 → v-for の :key="post.id" に流れる → Vue が要素を識別
		id: 1, // 【意味】一意の識別子（データベースの主キーに相当）

		// 【データの流れ】image_url: 'https://...' → <img :src="post.image_url"> に流れる → ブラウザが画像を取得して表示
		image_url: 'https://via.placeholder.com/600x600?text=Post+1', // 【意味】画像のURL（プレースホルダー画像を使用）

		// 【データの流れ】caption: 'これは最初の投稿です！' → <p>{{ post.caption }}</p> に流れる → 画面に表示
		caption: 'これは最初の投稿です！', // 【意味】投稿の説明文

		// 【データの流れ】username: 'user1' → {{ post.username || 'User' }} に流れる → 画面に表示
		username: 'user1', // 【意味】投稿者のユーザー名

		// 【データの流れ】created_at: "2024-01-01T12:00:00.000Z" → formatDate(post.created_at) に流れる
		//   → formatDate 内で new Date(dateString) で Date オブジェクトに変換
		//   → toLocaleDateString() で "2024年1月1日 12:00" 形式に変換
		//   → {{ formatDate(post.created_at) }} に流れる → 画面に表示
		// 【意味】作成日時を ISO 形式の文字列で生成
		// 【因果】new Date().toISOString() で現在時刻を "2024-01-01T12:00:00.000Z" 形式に変換
		// 【学び】ISO 形式は国際標準の日時フォーマットで、データベースや API でよく使われる
		created_at: new Date().toISOString(),
	},
	{
		// 【データの流れ】2つ目の投稿も同様の流れで画面に表示される
		id: 2,
		image_url: 'https://via.placeholder.com/600x600?text=Post+2',
		caption: '2つ目の投稿です',
		username: 'user2',
		// 【データの流れ】1時間前の日時を生成 → formatDate() で変換 → 画面に表示
		// 【意味】1時間前の日時を生成
		// 【因果】Date.now() で現在のミリ秒を取得、- 3600000 で1時間（3600秒 × 1000ミリ秒）を引く
		// 【学び】異なる日時を設定することで、タイムラインの順序を確認できる
		created_at: new Date(Date.now() - 3600000).toISOString(),
	},
]

//
// 【意味】日付フォーマット関数: ISO 形式の日時を日本語の読みやすい形式に変換
// 【因果】テンプレート内で {{ formatDate(post.created_at) }} として使用
// 【学び】関数を定義することで、同じ処理を複数箇所で再利用できる
//
// 【データの流れ（関数呼び出し時）】:
//   入力: post.created_at = "2024-01-01T12:00:00.000Z" (ISO形式の文字列)
//   ↓
//   formatDate("2024-01-01T12:00:00.000Z") が呼ばれる
//   ↓
//   出力: "2024年1月1日 12:00" (日本語形式の文字列)
//   ↓
//   {{ formatDate(post.created_at) }} に流れる → 画面に表示
//
const formatDate = (dateString) => {
	// 【データの流れ】dateString = "2024-01-01T12:00:00.000Z" (文字列)
	//   ↓ new Date(dateString)
	//   ↓ date = Date オブジェクト { year: 2024, month: 0, day: 1, ... }
	// 【意味】文字列を Date オブジェクトに変換
	// 【因果】toLocaleDateString を使うために Date オブジェクトが必要
	const date = new Date(dateString)

	//
	// 【データの流れ】date (Date オブジェクト)
	//   ↓ toLocaleDateString('ja-JP', {...})
	//   ↓ "2024年1月1日 12:00" (日本語形式の文字列)
	//   ↓ return で呼び出し元に返される
	// 【意味】日本語形式で日時を文字列に変換
	// 【因果】'ja-JP' でロケール（言語・地域）を指定
	// 【学び】オプションで表示形式を細かく制御できる
	//   - year: 'numeric' → 2024（4桁の年）
	//   - month: 'long' → 1月（日本語の月名）
	//   - day: 'numeric' → 1（日付）
	//   - hour: '2-digit' → 09（2桁の時間、0埋め）
	//   - minute: '2-digit' → 05（2桁の分、0埋め）
	//
	return date.toLocaleDateString('ja-JP', {
		year: 'numeric',
		month: 'long',
		day: 'numeric',
		hour: '2-digit',
		minute: '2-digit',
	})
}
</script>
```

### 3-2) 新規投稿画面（pages/post/new.vue）

※ 目的: フォーム入力とプレビューの流れを先に作り、API 実装前に UI/UX を検証できるようにする（因果: ここで入力値やバリデーションを固めておくと、Step 6 の API 接続時に改修が最小化される）。

`pages/post/new.vue` を作成します：

```vue
<template>
	<!-- コンテナ: 画面全体の幅を制限し、中央揃えにする -->
	<div class="container mx-auto max-w-2xl py-8">
		<!-- ページタイトル -->
		<h1 class="text-3xl font-bold mb-6">New Post</h1>

		<!-- 
			【意味】Card コンポーネント: フォーム全体をカード形式で表示
			【因果】統一されたデザインで見やすくする
		-->
		<Card>
			<!-- カードのヘッダー部分 -->
			<CardHeader>
				<CardTitle>投稿を作成</CardTitle>
			</CardHeader>
			<!-- カードのコンテンツ部分（フォーム） -->
			<CardContent>
				<!-- 
					【意味】@submit.prevent: フォーム送信時のデフォルト動作（ページリロード）を防ぐ
					【因果】.prevent 修飾子で event.preventDefault() を自動実行
					【学び】SPA ではページリロードを避けて、JavaScript で処理を制御する
				-->
				<form @submit.prevent="handleSubmit" class="space-y-4">
					<!-- 画像選択セクション -->
					<div class="space-y-2">
						<!-- 
							【意味】Label: フォーム要素のラベル（アクセシビリティ向上）
							【因果】for="image" で id="image" の Input と関連付け
							【学び】ラベルをクリックすると、関連する入力欄にフォーカスが移る
						-->
						<Label for="image">画像を選択</Label>
						<!-- 
							【意味】ファイル入力: ユーザーが画像ファイルを選択できる
							【因果】type="file" でファイル選択ダイアログを表示
							【学び】accept="image/*" で画像ファイルのみ選択可能にする
							【学び】@change でファイルが選択された時に handleFileChange を実行
						-->
						<Input
							id="image"
							type="file"
							accept="image/*"
							@change="handleFileChange"
						/>
						<!-- 
							【意味】画像プレビュー: 選択した画像をすぐに表示
							【因果】v-if="previewUrl" で previewUrl が存在する時のみ表示
							【学び】ユーザーが選択した画像を確認できることで、UX が向上
							【学び】max-h-96 で最大高さを制限し、画面からはみ出さないようにする
						-->
						<img
							v-if="previewUrl"
							:src="previewUrl"
							alt="プレビュー"
							class="w-full max-h-96 object-contain rounded"
						/>
					</div>

					<!-- キャプション入力セクション -->
					<div class="space-y-2">
						<Label for="caption">キャプション</Label>
						<!-- 
							【意味】v-model: 双方向データバインディング（入力値と変数を自動同期）
							【因果】caption 変数の値が変更されると、テキストエリアの表示も自動更新
							【学び】v-model は Vue の重要な機能で、フォーム入力の処理を簡単にする
							【学び】placeholder で入力例を表示（ユーザーの理解を助ける）
							
							【データの流れ（双方向）】:
							  入力時: ユーザーがテキストを入力 → caption.value に自動保存
							  表示時: caption.value の値 → テキストエリアに自動表示
							  送信時: caption.value → FormData に追加 → API に送信
						-->
						<Textarea
							id="caption"
							v-model="caption"
							placeholder="キャプションを入力..."
							rows="4"
						/>
					</div>

					<!-- ボタンセクション -->
					<div class="flex gap-2">
						<!-- 
							【意味】投稿ボタン: フォームを送信する
							【因果】:disabled で条件に応じてボタンを無効化
							【学び】!selectedFile || !caption で、ファイルまたはキャプションがない場合は無効化
							【学び】無効化されたボタンはクリックできず、視覚的にもグレーアウトされる
							
							【データの流れ（ボタンの有効/無効）】:
							  selectedFile.value = null → !selectedFile = true → ボタン無効
							  caption.value = '' → !caption = true → ボタン無効
							  selectedFile.value = File {...} && caption.value = 'テキスト' → ボタン有効
							  クリック時 → @submit.prevent → handleSubmit() が呼ばれる
						-->
						<Button type="submit" :disabled="!selectedFile || !caption">
							投稿する
						</Button>
						<!-- 
							【意味】キャンセルボタン: 投稿をキャンセルしてトップページに戻る
							【因果】@click でクリック時に $router.push('/') を実行
							【学び】$router は Nuxt のルーターインスタンスで、ページ遷移を制御
							【学び】variant="outline" でアウトラインスタイルのボタンを表示
						-->
						<Button type="button" variant="outline" @click="$router.push('/')">
							キャンセル
						</Button>
					</div>
				</form>
			</CardContent>
		</Card>
	</div>
</template>

<script setup>
//
// 【意味】ref のインポート: Vue のリアクティブな変数を作成する関数
// 【因果】ref を使うことで、変数の値が変わると自動的に画面が更新される
// 【学び】Vue 3 の Composition API では ref や reactive でリアクティブな状態を管理
//
import { ref } from 'vue'
// UI コンポーネントのインポート
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Textarea } from '@/components/ui/textarea'

//
// 【意味】リアクティブな変数の定義: コンポーネントの状態を管理
// 【因果】ref() でラップすることで、値が変わると画面が自動更新される
// 【学び】selectedFile: 選択されたファイルオブジェクトを保存
// 【学び】previewUrl: 画像プレビュー用の URL（Blob URL）を保存
// 【学び】caption: キャプションのテキストを保存
//
// 【データの流れ（初期状態）】:
//   selectedFile.value = null → :disabled="!selectedFile" でボタンが無効化される
//   previewUrl.value = null → v-if="previewUrl" が false になり、プレビュー画像が非表示
//   caption.value = '' → v-model="caption" でテキストエリアが空になる
//
const selectedFile = ref(null) // 初期値は null（何も選択されていない状態）
const previewUrl = ref(null) // 初期値は null（プレビューなし）
const caption = ref('') // 初期値は空文字列

//
// 【意味】ファイル選択時の処理: ユーザーが画像を選択した時に実行される
// 【因果】@change="handleFileChange" でこの関数が呼ばれる
//
// 【データの流れ（ファイル選択時）】:
//   1. ユーザーが <Input type="file"> でファイルを選択
//   2. ブラウザが event オブジェクトを生成（event.target.files にファイル情報が入る）
//   3. handleFileChange(event) が呼ばれる
//   4. event.target.files[0] → file 変数に保存（File オブジェクト）
//   5. file → selectedFile.value に保存（後で API 送信時に使用）
//   6. file → URL.createObjectURL(file) → previewUrl.value に保存（Blob URL）
//   7. previewUrl.value → <img :src="previewUrl"> に流れる → プレビュー画像が表示される
//
const handleFileChange = (event) => {
	//
	// 【データの流れ】event.target.files = FileList { 0: File {...}, length: 1 }
	//   ↓ [0] で最初の要素を取得
	//   ↓ file = File { name: "image.jpg", size: 12345, type: "image/jpeg", ... }
	// 【意味】選択されたファイルを取得
	// 【因果】event.target.files は FileList オブジェクト（複数ファイルを保持）
	// 【学び】[0] で最初のファイルを取得（単一ファイル選択の場合）
	//
	const file = event.target.files[0]

	//
	// 【意味】ファイルが選択された場合のみ処理を実行
	// 【因果】ファイル選択をキャンセルした場合、file は undefined になる
	//
	if (file) {
		// 【データの流れ】file (File オブジェクト) → selectedFile.value に保存
		//   → 後で handleSubmit() で FormData に追加される
		// 【意味】選択されたファイルを変数に保存（後で API に送信するため）
		selectedFile.value = file

		//
		// 【データの流れ】file (File オブジェクト)
		//   ↓ URL.createObjectURL(file)
		//   ↓ "blob:http://localhost:3000/abc123-def456-..." (Blob URL 文字列)
		//   ↓ previewUrl.value に保存
		//   ↓ <img :src="previewUrl"> に流れる
		//   ↓ ブラウザが Blob URL から画像データを読み込んで表示
		// 【意味】プレビュー用の URL を作成
		// 【因果】URL.createObjectURL() でファイルオブジェクトから Blob URL を生成
		// 【学び】Blob URL はブラウザのメモリ上に作成される一時的な URL
		// 【学び】この URL を <img> の src に設定することで、画像を表示できる
		// 【注意】メモリリークを防ぐため、不要になったら URL.revokeObjectURL() で解放すべき
		//
		previewUrl.value = URL.createObjectURL(file)
	}
}

//
// 【意味】フォーム送信時の処理: 「投稿する」ボタンをクリックした時に実行される
// 【因果】@submit.prevent="handleSubmit" でこの関数が呼ばれる
// 【注意】現在はモック実装（Step 6 で実際の API 呼び出しに置き換える）
//
const handleSubmit = () => {
	// TODO: ここでAPIを呼び出す（Step 6で実装）
	//
	// 【意味】一時的なアラート表示（実装が完了していないことを示す）
	// 【因果】モック画面なので、実際の API 呼び出しはまだ実装していない
	//
	alert('投稿機能はまだ実装されていません（Step 6で実装します）')

	//
	// 【意味】実際の実装時の処理フロー（Step 6 で実装）
	// 1. FormDataを作成: ファイルとキャプションを FormData オブジェクトに格納
	// 2. APIにPOSTリクエスト: fetch や axios を使って API にデータを送信
	// 3. 成功したらトップページに戻る: $router.push('/') でタイムラインに遷移
	//
}
</script>
```

### 3-3) レイアウトの設定（オプション）

※ 目的: 共通レイアウトを作ることで、各ページが同じ余白・背景・フォント設定を共有し、ページ追加時の重複を削減する（因果: 先にベースを決めると後からのデザイン差分対応が容易）。

`app.vue` または `layouts/default.vue` を作成して、共通レイアウトを設定できます：

```vue
<template>
	<!-- 
		【意味】共通レイアウト: すべてのページで共通の背景と最小高さを設定
		【因果】min-h-screen で画面の高さを最低限確保し、コンテンツが少ない時も背景が表示される
		【学び】bg-gray-50 で薄いグレーの背景を設定（Instagram のような見た目）
		【学び】<slot /> で各ページのコンテンツが挿入される（Vue のスロット機能）
	-->
	<div class="min-h-screen bg-gray-50">
		<slot />
	</div>
</template>
```

## ✅ チェックリスト

- [ ] タイムライン画面（`pages/index.vue`）が作成された
- [ ] モックデータが表示される
- [ ] 新規投稿画面（`pages/post/new.vue`）が作成された
- [ ] 画像選択とプレビューが動作する
- [ ] 「New Post」ボタンから投稿画面に遷移できる
- [ ] 投稿画面からトップページに戻れる

## 🎯 次のステップ

モック画面が完成したら、**step4.md** に進んでください。
（Backend セットアップ：FastAPI プロジェクトの作成）
