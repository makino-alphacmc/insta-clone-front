# Step 1: 環境構築（必要なツールのインストール確認）

## 📋 このステップでやること

プロジェクトを始めるために必要なツールがインストールされているか確認し、不足している場合はインストールします。

## ✅ 必要なツールのチェックリスト

### 1. Node.js と npm の確認

```bash
node --version
npm --version
```

**必要なバージョン**:
- Node.js: v18以上推奨
- npm: v9以上推奨

**インストール方法**（未インストールの場合）:
- macOS: [Homebrew](https://brew.sh/) で `brew install node`
- または [公式サイト](https://nodejs.org/) からダウンロード

### 2. Python の確認

```bash
python3 --version
```

**必要なバージョン**: Python 3.8以上推奨

**インストール方法**（未インストールの場合）:
- macOS: `brew install python3`
- または [公式サイト](https://www.python.org/downloads/) からダウンロード

### 3. Git の確認

```bash
git --version
```

**インストール方法**（未インストールの場合）:
- macOS: `xcode-select --install` または `brew install git`

### 4. Docker の確認（後で使います）

```bash
docker --version
docker compose version
```

**インストール方法**（未インストールの場合）:
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) をインストール

**注意**: Docker は Step 7 で使用します。今すぐインストールしなくても大丈夫です。

## 🔧 作業ディレクトリの準備

プロジェクトを作成する場所を決めます：

```bash
# 作業用ディレクトリを作成（既にある場合はスキップ）
mkdir -p ~/work
cd ~/work
```

または、任意の場所に作成してもOKです。

## ✅ 確認コマンド（まとめて実行）

以下のコマンドで、すべてのツールがインストールされているか確認できます：

```bash
echo "=== 環境確認 ==="
echo "Node.js: $(node --version 2>/dev/null || echo '未インストール')"
echo "npm: $(npm --version 2>/dev/null || echo '未インストール')"
echo "Python: $(python3 --version 2>/dev/null || echo '未インストール')"
echo "Git: $(git --version 2>/dev/null || echo '未インストール')"
echo "Docker: $(docker --version 2>/dev/null || echo '未インストール（Step 7で必要）')"
```

## 🎯 次のステップ

すべてのツールがインストールされていることを確認したら、**step2.md** に進んでください。
（Frontend セットアップ：Nuxt3 プロジェクトの作成）

