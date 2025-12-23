# Step 5: 画像保存（Supabase Storage）の設定

## 📋 このステップでやること

画像を保存するために Supabase Storage を設定し、API 側に画像アップロード機能を実装します。

## ✅ 手順

### 5-1) Supabase プロジェクトの作成
※ 目的: 画像保存先を確保し、後続のアップロードAPIで使う認証情報を用意。

1. [Supabase](https://supabase.com/) にアクセスしてアカウントを作成（またはログイン）

2. 新しいプロジェクトを作成：
   - **Project Name**: `insta-clone`（任意）
   - **Database Password**: 安全なパスワードを設定
   - **Region**: 最寄りのリージョンを選択

3. プロジェクトが作成されたら、以下を控えておきます：
   - **Project URL**: `https://xxxxx.supabase.co`
   - **anon/public key**: Settings → API → `anon public` キー

### 5-2) Storage Bucket の作成
※ 目的: 画像を置く公開バケットを準備し、URLで配信できる状態にする。

1. Supabase ダッシュボードで **Storage** を開く

2. **Create a new bucket** をクリック

3. 以下の設定で作成：
   - **Name**: `post-images`
   - **Public bucket**: ✅ **ON**（画像を公開URLでアクセスできるようにする）

4. **Create bucket** をクリック

### 5-3) API 側の環境変数を更新
※ 目的: Supabase接続情報をコードから分離し、デプロイ環境ごとに切り替え可能にする。

`insta-clone-api/.env` を編集します：

```bash
cd ~/work/insta-clone-api
code .env  # または任意のエディタ
```

以下の内容に更新（実際の値に置き換えてください）：

```env
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=your_anon_key_here
SUPABASE_BUCKET=post-images
ALLOWED_ORIGINS=http://localhost:3000
```

### 5-4) Supabase SDK のインストール
※ 目的: Python から Storage へアップロードするためのクライアントを導入。

```bash
# 仮想環境が有効化されていることを確認
cd ~/work/insta-clone-api
source venv/bin/activate  # まだ有効化していない場合

# Supabase SDK をインストール
pip install supabase
```

### 5-5) 画像アップロード関数の実装（app/storage.py）
※ 目的: バケットへのアップロードと公開URL取得を共通関数化し、API本体をシンプルに保つ。

`app/storage.py` を編集します：

```python
# 
# 【意味】必要なライブラリのインポート
# 【因果】os で環境変数取得、supabase でストレージ操作、UploadFile でファイル受信、uuid で一意なファイル名生成
#
import os
from supabase import create_client, Client
from fastapi import UploadFile
import uuid

# 
# 【意味】環境変数から Supabase の設定を取得
# 【因果】.env ファイルから読み込んだ値を os.getenv() で取得
# 【学び】os.getenv("KEY", "default") で、環境変数が存在しない場合はデフォルト値を返す
# 【学び】機密情報をコードに直接書かず、環境変数で管理することでセキュリティを向上
#
SUPABASE_URL = os.getenv("SUPABASE_URL")  # Supabase プロジェクトの URL
SUPABASE_KEY = os.getenv("SUPABASE_ANON_KEY")  # Supabase の匿名キー（公開可能）
BUCKET_NAME = os.getenv("SUPABASE_BUCKET", "post-images")  # ストレージバケット名（デフォルト: post-images）

# 
# 【意味】Supabase クライアントの作成: Supabase の API を呼び出すためのクライアント
# 【因果】create_client() でクライアントを作成し、以降の操作で使用する
# 【学び】Client 型ヒントで、IDE の補完が効くようになる
#
supabase: Client = create_client(SUPABASE_URL, SUPABASE_KEY)

# 
# 【意味】画像アップロード関数: ファイルを Supabase Storage にアップロードし、公開 URL を返す
# 【因果】async/await を使うことで、ファイルの読み込み中に他の処理をブロックしない
# 【学び】-> str で戻り値の型を明示（型ヒントでコードの可読性が向上）
#
# 【データの流れ（画像アップロード時）】:
#   1. 入力: file = UploadFile (FastAPI が受け取ったファイルオブジェクト)
#   2. file.filename = "image.jpg" → 拡張子を抽出 → file_extension = "jpg"
#   3. uuid.uuid4() → "a1b2c3d4-..." → file_name = "a1b2c3d4.jpg"
#   4. await file.read() → file_content = b'\xff\xd8\xff\xe0...' (バイナリデータ)
#   5. supabase.storage.upload() → Supabase Storage にアップロード
#      → ファイルが Supabase のストレージに保存される
#   6. supabase.storage.get_public_url() → public_url = "https://xxx.supabase.co/storage/.../a1b2c3d4.jpg"
#   7. return public_url → この URL が呼び出し元に返される
#   8. 呼び出し元（create_post）で image_url としてデータベースに保存される
#
async def upload_image(file: UploadFile) -> str:
    """
    画像を Supabase Storage にアップロードし、公開URLを返す
    
    Args:
        file: アップロードするファイル（FastAPI の UploadFile 型）
        
    Returns:
        公開URL（文字列）: アップロードした画像にアクセスできる URL
    """
    # 
    # 【データの流れ】file.filename = "my-image.jpg" (文字列)
    #   ↓ split(".") → ["my-image", "jpg"]
    #   ↓ [-1] → "jpg" (最後の要素)
    #   ↓ file_extension = "jpg"
    # 【意味】ファイル拡張子の取得: 元のファイル名から拡張子を抽出
    # 【因果】file.filename.split(".")[-1] で最後の . 以降を取得（例: "image.jpg" → "jpg"）
    # 【学び】"." in file.filename で拡張子があるかチェック、なければ "jpg" をデフォルトとする
    #
    file_extension = file.filename.split(".")[-1] if "." in file.filename else "jpg"
    
    # 
    # 【データの流れ】uuid.uuid4() → "a1b2c3d4-e5f6-7890-abcd-ef1234567890" (UUID文字列)
    #   ↓ f"{uuid.uuid4()}.{file_extension}" → "a1b2c3d4-e5f6-7890-abcd-ef1234567890.jpg"
    #   ↓ file_name = "a1b2c3d4-e5f6-7890-abcd-ef1234567890.jpg"
    # 【意味】一意なファイル名の生成: UUID を使って重複を避ける
    # 【因果】uuid.uuid4() でランダムな UUID を生成し、拡張子を付けてファイル名を作成
    # 【学び】UUID を使うことで、同じファイル名でも上書きされない（例: "a1b2c3d4.jpg"）
    # 【学び】f"{uuid.uuid4()}.{file_extension}" で文字列を結合（f-string 構文）
    #
    file_name = f"{uuid.uuid4()}.{file_extension}"
    
    # 
    # 【データの流れ】file (UploadFile オブジェクト)
    #   ↓ await file.read() → ファイルのバイナリデータを読み込む
    #   ↓ file_content = b'\xff\xd8\xff\xe0\x00\x10JFIF...' (バイト列)
    # 【意味】ファイル内容の読み込み: アップロードされたファイルのバイナリデータを取得
    # 【因果】await file.read() で非同期にファイルを読み込む
    # 【学び】await を使うことで、ファイル読み込み中に他の処理を実行できる（非ブロッキング）
    #
    file_content = await file.read()
    
    # 
    # 【データの流れ】file_name = "a1b2c3d4.jpg", file_content = b'\xff\xd8...'
    #   ↓ supabase.storage.upload(file_name, file_content, ...)
    #   ↓ HTTP リクエストを Supabase API に送信
    #   ↓ Supabase Storage がファイルを受け取り、ストレージに保存
    #   ↓ response = {"path": "a1b2c3d4.jpg", "error": None} (成功時)
    # 【意味】Supabase Storage へのアップロード: ファイルをストレージに保存
    # 【因果】supabase.storage.from_(BUCKET_NAME) でバケットを指定し、upload() でアップロード
    # 【学び】file_options で Content-Type を指定（ブラウザが正しく画像として認識できるように）
    # 【学び】file.content_type or "image/jpeg" で、Content-Type がなければデフォルトを設定
    #
    response = supabase.storage.from_(BUCKET_NAME).upload(
        file_name,  # アップロードするファイル名
        file_content,  # ファイルのバイナリデータ
        file_options={"content-type": file.content_type or "image/jpeg"}  # MIME タイプ
    )
    
    # 
    # 【データの流れ】response = {"error": "..."} または {"error": None}
    #   ↓ response.get("error") → エラーがあるかチェック
    #   ↓ エラーがある場合 → Exception を発生させて処理を中断
    # 【意味】エラーチェック: アップロードが失敗した場合にエラーを発生させる
    # 【因果】response.get("error") でエラーがあるかチェックし、あれば例外を投げる
    # 【学び】エラーハンドリングを適切に行うことで、問題を早期に発見できる
    #
    if response.get("error"):
        raise Exception(f"アップロードエラー: {response['error']}")
    
    # 
    # 【データの流れ】file_name = "a1b2c3d4.jpg"
    #   ↓ supabase.storage.get_public_url(file_name)
    #   ↓ public_url = "https://xxxxx.supabase.co/storage/v1/object/public/post-images/a1b2c3d4.jpg"
    #   ↓ この URL はブラウザで直接アクセスできる（公開 URL）
    # 【意味】公開 URL の取得: アップロードした画像にアクセスできる URL を生成
    # 【因果】get_public_url() で公開 URL を取得（ブラウザで直接アクセス可能）
    # 【学び】この URL をデータベースに保存し、フロントエンドで画像を表示する
    #
    public_url = supabase.storage.from_(BUCKET_NAME).get_public_url(file_name)
    
    # 【データの流れ】public_url = "https://xxxxx.supabase.co/storage/.../a1b2c3d4.jpg"
    #   ↓ return public_url
    #   ↓ 呼び出し元（create_post）の image_url 変数に代入される
    #   ↓ データベースの image_url カラムに保存される
    #   ↓ フロントエンドで <img :src="post.image_url"> として表示される
    return public_url  # 【意味】公開 URL を返す（データベースに保存するために使用）
```

### 5-6) API エンドポイントの更新（app/main.py）
※ 目的: フォームデータで受けた画像をStorageへ送り、DBへメタデータを保存する流れを実装。

`app/main.py` を更新して、画像アップロード機能を追加します：

```python
# 
# 【意味】必要なライブラリのインポート
# 【因果】UploadFile, File, Form でファイルアップロードとフォームデータを受け取る
# 【学び】File(...) でファイルを必須パラメータに、Form(...) でフォームデータを受け取る
#
from fastapi import FastAPI, Depends, HTTPException, UploadFile, File, Form
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session
from typing import List
import os
from dotenv import load_dotenv

from app import models, schemas
from app.db import SessionLocal, engine
from app.storage import upload_image  # 【意味】画像アップロード関数をインポート

# ... 既存のコード ...

# 
# 【意味】投稿作成エンドポイント（画像アップロード対応）: ファイルとキャプションを受け取り、投稿を作成
# 【因果】POST /posts に multipart/form-data 形式でリクエストを送ると、画像とキャプションが保存される
# 【学び】async def で非同期関数として定義（ファイルアップロードは時間がかかるため）
#
@app.post("/posts", response_model=schemas.Post)
async def create_post(
    file: UploadFile = File(...),  # 【意味】アップロードされた画像ファイル（必須）
    caption: str = Form(""),  # 【意味】フォームから送信されたキャプション（オプション、デフォルトは空文字）
    db: Session = Depends(get_db)  # 【意味】データベースセッション（依存性注入）
):
    # 
    # 【データの流れ】file (UploadFile オブジェクト)
    #   ↓ await upload_image(file) → upload_image() 関数を呼び出す
    #   ↓ upload_image() 内で Supabase Storage にアップロード
    #   ↓ public_url = "https://xxxxx.supabase.co/storage/.../a1b2c3d4.jpg" を返す
    #   ↓ image_url = "https://xxxxx.supabase.co/storage/.../a1b2c3d4.jpg"
    # 【意味】画像を Supabase Storage にアップロード
    # 【因果】try-except でエラーハンドリングを行い、アップロード失敗時に適切なエラーレスポンスを返す
    # 【学び】await upload_image(file) で非同期に画像をアップロードし、公開 URL を取得
    #
    try:
        image_url = await upload_image(file)
    except Exception as e:
        # 
        # 【データの流れ】エラーが発生した場合
        #   ↓ HTTPException を発生させる
        #   ↓ FastAPI が HTTP 500 エラーレスポンスを生成
        #   ↓ {"detail": "画像のアップロードに失敗しました: ..."} (JSON)
        #   ↓ フロントエンドに送信される
        # 【意味】エラーレスポンス: アップロードに失敗した場合、500エラーを返す
        # 【因果】HTTPException で FastAPI の標準的なエラーレスポンスを返す
        # 【学び】status_code=500 でサーバーエラー、detail でエラーメッセージを指定
        #
        raise HTTPException(status_code=500, detail=f"画像のアップロードに失敗しました: {str(e)}")
    
    # 
    # 【データの流れ】image_url = "https://xxxxx.supabase.co/storage/.../a1b2c3d4.jpg"
    #   caption = "投稿のキャプション" (文字列、空の場合は "")
    #   ↓ models.Post(image_url=image_url, caption=caption if caption else None)
    #   ↓ db_post = Post(id=None, image_url="https://...", caption="...", created_at=None)
    # 【意味】データベースに投稿を保存: アップロードした画像の URL とキャプションを保存
    # 【因果】image_url をデータベースに保存することで、フロントエンドで画像を表示できる
    # 【学び】caption if caption else None で、空文字列の場合は None を保存（データベースの nullable=True に対応）
    #
    db_post = models.Post(
        image_url=image_url,  # Supabase Storage から取得した公開 URL
        caption=caption if caption else None  # キャプション（空の場合は None）
    )
    
    # 【データの流れ】db_post → db.add() → セッションに追加（まだ DB には保存されていない）
    db.add(db_post)  # セッションに追加
    
    # 【データの流れ】セッション → db.commit() → SQL を実行
    #   ↓ INSERT INTO posts (image_url, caption, created_at) VALUES (..., ..., NOW())
    #   ↓ データベースに保存される
    #   ↓ データベースが自動的に id=1, created_at="2024-01-01T12:00:00" を生成
    db.commit()  # データベースに保存
    
    # 【データの流れ】db_post (id=None のまま) → db.refresh() → DB から最新データを取得
    #   ↓ SELECT * FROM posts WHERE id = 1
    #   ↓ db_post.id = 1, db_post.created_at = "2024-01-01T12:00:00"
    db.refresh(db_post)  # 最新のデータ（id や created_at）を取得
    
    # 【データの流れ】db_post = Post(id=1, image_url="https://...", caption="...", created_at="...")
    #   ↓ return db_post → FastAPI が Pydantic スキーマに変換
    #   ↓ {"id": 1, "image_url": "https://...", "caption": "...", "created_at": "..."}
    #   ↓ JSON にシリアライズ → フロントエンドに送信
    return db_post  # 作成された投稿データを返す
```

### 5-7) 動作確認
※ 目的: curlでの手動テストにより、アップロード→URL応答までの因果を検証し、不具合を早期発見。

サーバーを再起動して、動作を確認します：

```bash
# サーバーを起動（既に起動している場合は再起動）
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**curl でテスト**（別のターミナルで実行）：

```bash
# テスト用の画像ファイルを用意（任意の画像ファイルのパスに置き換えてください）
curl -X POST "http://localhost:8000/posts" \
  -F "caption=テスト投稿" \
  -F "file=@/path/to/your/image.jpg"
```

成功すると、以下のようなレスポンスが返ってきます：

```json
{
  "id": 1,
  "image_url": "https://xxxxx.supabase.co/storage/v1/object/public/post-images/xxxxx.jpg",
  "caption": "テスト投稿",
  "created_at": "2024-01-01T12:00:00"
}
```

ブラウザで `image_url` を開いて、画像が表示されることを確認してください。

## ✅ チェックリスト

- [ ] Supabase プロジェクトが作成された
- [ ] Project URL と anon key を控えた
- [ ] Storage bucket `post-images` が作成された（Public ON）
- [ ] `.env` ファイルに Supabase の設定が追加された
- [ ] Supabase SDK がインストールされた
- [ ] `app/storage.py` に画像アップロード関数が実装された
- [ ] `app/main.py` の POST /posts エンドポイントが更新された
- [ ] curl で画像アップロードが成功した
- [ ] アップロードした画像のURLがブラウザで表示できる

## 🎯 次のステップ

画像アップロード機能が動作したら、**step6.md** に進んでください。
（Frontend と Backend の連携）

