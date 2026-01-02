「Gitを使って、ローカルのファイルをGitHubにアップロード（= commit \& push）する手順」。


\# GitでGitHubへファイルをアップロードする手順書（Windows / PowerShell）



\## 目的

ローカルPCにあるファイル/フォルダを、Gitを使ってGitHubリポジトリへ反映する（commit \& push）。



---



\## 0. 前提（最初の1回だけ）

\### 0.1 Gitが使えること

```powershell

git --version

````



\### 0.2 コミット作成者情報（未設定だと commit が失敗する）



```powershell

git config --global user.name "YourName"

git config --global user.email "you@example.com"

```



確認：



```powershell

git config --global --list

```



※ メールを公開したくない場合は、GitHubの `@users.noreply.github.com` を使う（GitHub設定で確認）。



---



\## 1. 「アップロード」の考え方（重要）



GitHubにファイルを上げる＝Webアップロードではなく、基本はこの流れ：



1\. リポジトリをPCに \*\*clone\*\*（初回）

2\. そのフォルダの中にファイルを置く/更新する

3\. 変更を \*\*add\*\* して \*\*commit\*\*（履歴を作る）

4\. \*\*push\*\*（GitHubへ送る）



---



\## 2. 初回：リポジトリをクローンする

## 2.0 クローン元URL（GitHubのURL）の取り方

### 2.0.1 GitHubの画面からURLをコピーする（推奨）
1. ブラウザで対象のGitHubリポジトリを開く  
2. 画面右上あたりの緑の **Code** ボタンを押す  
3. 表示されたタブで **HTTPS** を選ぶ（通常はこれでOK）  
4. 表示されるURL（例：`https://github.com/<ユーザー名>/<リポジトリ名>.git`）をコピーする  
   - 右側のコピーアイコンを押すと確実

### 2.0.2 どのURLを選べばいい？
- 基本は **HTTPS** を使う  
  - Git Credential Manager（Windowsの認証）でログインしやすい
- SSHは、鍵設定（ssh-key）を済ませている人向け（慣れてなければHTTPSでよい）

### 2.0.3 コピーしたURLが正しいかの目安
- 末尾が `.git` で終わっている（例：`.../TRPG.git`）  
- ドメインが `github.com` である  
- リポジトリ名が一致している（例：`Aquagate/TRPG`）



\### 2.1 作業場所へ移動（例）



```powershell

cd E:\\dev

```



\### 2.2 clone



```powershell

git clone https://github.com/<ユーザー名>/<リポジトリ名>.git

cd <リポジトリ名>

```



\### 2.3 接続先確認



```powershell

git remote -v

```



`origin ... (fetch)` と `origin ... (push)` が同じURLならOK。



---



\## 3. ファイル/フォルダを入れる（ここが“アップロード対象”）



\* クローンしたフォルダ（リポジトリ直下）に、アップロードしたいファイル/フォルダを置く

\* \*\*フォルダ丸ごと入れると二重構造になりやすい\*\*（例：repo/TRPG-main/...）



&nbsp; \* 意図がなければ「中身を直下へ」置く



\### 3.1 注意：空フォルダはGitHubに上がらない



Gitは「ファイル」を記録するので、空フォルダは存在しない扱い。

空フォルダを残したいなら、空ファイルを入れる：



```powershell

New-Item -ItemType File .\\path\\to\\empty\\.gitkeep

```



---



\## 4. .gitignore を用意する（不要物を上げない）



例（Node/Vite系の最低限）：



```powershell

@"

node\_modules/

dist/

.vite/

.env

.env.\*

"@ | Out-File -Encoding utf8 .gitignore

```



※ `.gitignore` はリポジトリ直下に置く。



---



\## 5. 変更を確認する



```powershell

git status

```



\* `Untracked files`：新規追加（まだGit管理外）

\* `Changes not staged`：変更はあるが、addしてない

\* `nothing to commit, working tree clean`：送るものがない（正常）



---



\## 6. add → commit → push（基本の3手）



\### 6.1 add（ステージング）



```powershell

git add .

```



\### 6.2 commit（履歴作成）



```powershell

git commit -m "Add/update files"

```



\### 6.3 push（GitHubへ送る）



```powershell

git push

```



---



\## 7. push前に必要になることがある：pull（リモートの更新を取り込む）



GitHub側が先に更新されていて push が拒否されたら、先に取り込む：



```powershell

git pull --rebase

git push

```



---



\## 8. よくある詰まりポイントと対処



\### 8.1 `fatal: not a git repository`



原因：Git管理フォルダ（`.git`）の外でコマンドを打っている

対処：正しい場所へ移動してから実行



```powershell

cd E:\\dev\\<リポジトリ名>

git status

```



\### 8.2 `Author identity unknown`



原因：`user.name` / `user.email` 未設定

対処：0.2を実施してから commit し直す



\### 8.3 `Everything up-to-date`



意味：送るコミットがない（pushするものがない）

対処：`git status` で変更があるか確認



\### 8.4 `push rejected` / `non-fast-forward`



原因：GitHub側に先行コミットがある

対処：



```powershell

git pull --rebase

git push

```



\### 8.5 間違って `node\_modules` を追跡してしまった



追跡解除（ファイル自体は消さず、Git管理から外す）：



```powershell

git rm -r --cached node\_modules

git add .

git commit -m "Stop tracking node\_modules"

git push

```



\### 8.6 100MB超のファイルがある



GitHubは巨大ファイルに厳しい（1ファイル100MB超で基本アウト）。

対処：アップしない/分割/別手段（LFSなど）を検討。



---



\## 9. 最短チェックリスト（1分版）



1\. `cd <repo>`

2\. ファイルを置く（不要物は除外）

3\. `git status`

4\. `git add .`

5\. `git commit -m "..." `

6\. `git push`

7\. `git status` が clean なら完了



---



\## 用語集



\* \*\*clone\*\*：GitHubのリポジトリをPCにコピーして同期可能にする

\* \*\*add\*\*：commit対象として登録する（ステージング）

\* \*\*commit\*\*：変更のセーブポイント（履歴）

\* \*\*push\*\*：PC → GitHubへ送る

\* \*\*pull\*\*：GitHub → PCへ取り込む

\* \*\*rebase\*\*：履歴を整えて取り込む（衝突が少なくなりやすい）



```



これを runbooks に置くなら、例えばこういう場所が分かりやすい：



\- `git/upload-to-github.md`



そしてREADMEを目次にしてリンク貼っとけば、未来のうぜんが迷子にならない。

::contentReference\[oaicite:0]{index=0}

```



