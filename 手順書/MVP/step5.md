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
import os
# os: オペレーティングシステム関連の機能（環境変数の取得など）

from supabase import create_client, Client
# create_client: Supabaseクライアントを作成する関数
# Client: Supabaseクライアントの型

from fastapi import UploadFile
# UploadFile: FastAPIが受け取ったファイルオブジェクトの型

import uuid
# uuid: 一意なIDを生成するためのモジュール

# 環境変数からSupabaseの設定を取得
SUPABASE_URL = os.getenv("SUPABASE_URL")
# os.getenv(): 環境変数を取得
# "SUPABASE_URL": 環境変数のキー

SUPABASE_KEY = os.getenv("SUPABASE_ANON_KEY")
# "SUPABASE_ANON_KEY": Supabaseの匿名キー（公開可能）

BUCKET_NAME = os.getenv("SUPABASE_BUCKET", "post-images")
# "SUPABASE_BUCKET": ストレージバケット名
# "post-images": デフォルト値（環境変数が存在しない場合）

# Supabaseクライアントの作成
supabase: Client = create_client(SUPABASE_URL, SUPABASE_KEY)
# create_client(): Supabaseクライアントを作成
# SUPABASE_URL: SupabaseプロジェクトのURL
# SUPABASE_KEY: Supabaseの匿名キー
# Client: 型ヒント（IDEの補完が効くようになる）

# 画像をSupabase Storageにアップロードし、公開URLを返す
async def upload_image(file: UploadFile) -> str:
    # async def: 非同期関数を定義（awaitを使用するため）
    # upload_image: 画像アップロード関数
    # file: UploadFile: アップロードするファイル（FastAPIが受け取ったファイルオブジェクト）
    # -> str: 戻り値の型（文字列、公開URLを返す）
    
    # ファイル拡張子の取得
    file_extension = file.filename.split(".")[-1] if "." in file.filename else "jpg"
    # file.filename: ファイル名（例: "image.jpg"）
    # .split("."): 文字列を"."で分割（例: ["image", "jpg"]）
    # [-1]: 配列の最後の要素を取得（拡張子）
    # if "." in file.filename: 拡張子があるかチェック
    # else "jpg": 拡張子がない場合は"jpg"をデフォルトとする
    
    # 一意なファイル名の生成（UUIDを使用）
    file_name = f"{uuid.uuid4()}.{file_extension}"
    # uuid.uuid4(): ランダムなUUIDを生成（例: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"）
    # f"{...}": f-string構文（文字列を動的に結合）
    # 例: "a1b2c3d4-e5f6-7890-abcd-ef1234567890.jpg"
    
    # ファイル内容の読み込み
    file_content = await file.read()
    # await file.read(): ファイルのバイナリデータを非同期に読み込む
    # file_content: バイト列（例: b'\xff\xd8\xff\xe0...'）
    
    # Supabase Storageへのアップロード
    response = supabase.storage.from_(BUCKET_NAME).upload(
        # supabase.storage: Supabaseのストレージ機能にアクセス
        # .from_(BUCKET_NAME): バケットを指定
        # .upload(): ファイルをアップロード
        file_name,
        # file_name: アップロードするファイル名
        file_content,
        # file_content: ファイルのバイナリデータ
        file_options={"content-type": file.content_type or "image/jpeg"}
        # file_options: ファイルのオプション
        # "content-type": MIMEタイプ（ブラウザが正しく画像として認識できるように）
        # file.content_type: ファイルのContent-Type（例: "image/jpeg"）
        # or "image/jpeg": Content-Typeがなければデフォルトを設定
    )
    
    # エラーチェック
    if response.get("error"):
        # response.get("error"): レスポンスにエラーがあるかチェック
        # .get(): 辞書のキーが存在しない場合はNoneを返す（エラーにならない）
        raise Exception(f"アップロードエラー: {response['error']}")
        # raise Exception(): エラーを発生させる（呼び出し元でエラーハンドリング）
        # f"...": f-string構文（エラーメッセージにエラー内容を埋め込む）
    
    # 公開URLの取得
    public_url = supabase.storage.from_(BUCKET_NAME).get_public_url(file_name)
    # get_public_url(): アップロードした画像の公開URLを取得
    # 例: "https://xxxxx.supabase.co/storage/v1/object/public/post-images/a1b2c3d4.jpg"
    
    return public_url
    # return: 公開URLを返す（データベースに保存するために使用）
```

### 5-6) API エンドポイントの更新（app/main.py）
※ 目的: フォームデータで受けた画像をStorageへ送り、DBへメタデータを保存する流れを実装。

`app/main.py` を更新して、画像アップロード機能を追加します：

```python
from fastapi import FastAPI, Depends, HTTPException, UploadFile, File, Form
# UploadFile: FastAPIが受け取ったファイルオブジェクトの型
# File: ファイルを必須パラメータとして受け取る関数
# Form: フォームデータとして受け取る関数
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session
from typing import List
import os
from dotenv import load_dotenv

from app import models, schemas
from app.db import SessionLocal, engine
from app.storage import upload_image
# upload_image: 画像アップロード関数（app/storage.pyで定義）

# ... 既存のコード ...

# 投稿作成エンドポイント（画像アップロード対応）
@app.post("/posts", response_model=schemas.Post)
# @app.post: POSTリクエストを受け付けるエンドポイントを定義
# "/posts": エンドポイントのパス
# response_model=schemas.Post: レスポンスの型を指定

async def create_post(
    # async def: 非同期関数を定義（awaitを使用するため）
    # create_post: 新規投稿を作成する関数
    file: UploadFile = File(...),
    # file: UploadFile: アップロードされた画像ファイル
    # File(...): FastAPIのFile関数（ファイルを必須パラメータとして受け取る）
    # ...: 必須であることを示す（省略不可）
    caption: str = Form(""),
    # caption: str: キャプション（文字列型）
    # Form(""): FastAPIのForm関数（フォームデータとして受け取る）
    # "": デフォルト値（空文字列、オプション）
    db: Session = Depends(get_db)
    # db: Session: データベースセッション（依存性注入）
):
    # 画像をSupabase Storageにアップロード
    try:
        # try: エラーが発生する可能性のある処理を囲む
        image_url = await upload_image(file)
        # await upload_image(file): 画像アップロード関数を非同期に実行
        # image_url: アップロードした画像の公開URL（文字列）
    except Exception as e:
        # except: tryブロックでエラーが発生した場合の処理
        # Exception as e: 発生したエラーをe変数に格納
        raise HTTPException(status_code=500, detail=f"画像のアップロードに失敗しました: {str(e)}")
        # raise HTTPException(): HTTPエラーレスポンスを返す
        # status_code=500: HTTPステータスコード500（サーバーエラー）
        # detail: エラーメッセージ
        # str(e): エラーオブジェクトを文字列に変換
    
    # データベースに投稿を保存
    db_post = models.Post(
        # models.Post(): Postモデルのインスタンスを作成
        image_url=image_url,
        # image_url: Supabase Storageから取得した公開URL
        caption=caption if caption else None
        # caption if caption else None: キャプションが空文字列の場合はNoneを設定
        # if caption: captionが空文字列でない場合
        # else None: captionが空文字列の場合はNone（データベースのnullable=Trueに対応）
    )
    
    db.add(db_post)
    # db.add(): セッションに追加（まだデータベースには保存されていない）
    
    db.commit()
    # db.commit(): 変更をデータベースに永続化（SQLを実行）
    
    db.refresh(db_post)
    # db.refresh(): データベースから最新データを取得（idやcreated_atが設定される）
    
    return db_post
    # return: 作成された投稿データを返す（FastAPIが自動的にJSONに変換）
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

