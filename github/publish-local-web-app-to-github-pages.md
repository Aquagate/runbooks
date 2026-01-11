# ローカルWebアプリをGitHub Pagesで公開してスマホから見る

## 概要
ローカルで作成したWebアプリ（静的サイト）をGitHub Pagesで公開し、スマホから閲覧できるようにする手順です。

## 前提
- GitHubアカウントを持っている
- 公開したいWebアプリが静的ファイル（HTML/CSS/JS）で構成されている
- Gitがインストール済み

## 手順

### 1. GitHubにリポジトリを作成する
1. GitHubで新規リポジトリを作成する。
2. リポジトリ名は任意（例: `my-web-app`）。

### 2. ローカルのWebアプリをGitHubにアップロードする
1. 公開したいフォルダでGitを初期化する。
   ```bash
   git init
   ```
2. 変更を追加してコミットする。
   ```bash
   git add .
   git commit -m "Initial commit"
   ```
3. GitHubのリポジトリと紐づけてプッシュする。
   ```bash
   git remote add origin https://github.com/<ユーザー名>/<リポジトリ名>.git
   git branch -M main
   git push -u origin main
   ```

### 3. GitHub Pagesを有効にする
1. GitHubのリポジトリ画面で「Settings」を開く。
2. 左メニューの「Pages」を開く。
3. 「Build and deployment」→「Source」で `Deploy from a branch` を選ぶ。
4. Branch を `main`、フォルダを `/ (root)` に設定し「Save」。

### 4. 公開URLを確認する
1. 数十秒待つと、GitHub PagesのURLが表示される。
2. 表示されたURL（例: `https://<ユーザー名>.github.io/<リポジトリ名>/`）にアクセスして表示を確認する。

### 5. スマホで確認する
1. スマホのブラウザで同じURLを開く。
2. 必要に応じてホーム画面に追加してショートカット化する。

## 補足
- フレームワーク（React/Vueなど）を使っている場合は、ビルド成果物（`dist` や `build`）をリポジトリに配置する必要があります。
- privateリポジトリの場合、GitHub Pagesの公開範囲に注意してください。
