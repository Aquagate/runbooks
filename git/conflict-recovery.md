\# TRPG GitHub 復旧手順書（Codexコンフリクト対応）



\## 0. 目的



Codex（自動生成）による \*\*コンフリクト多発 → 誤って反映 → リポジトリが崩壊\*\* したときに、

手元に残っている「正常なプロジェクト一式（フォルダ群）」を使って \*\*GitHub上の状態を復旧\*\*する。



---



\## 1. 起きたこと（症状）



\* Codexが複数ファイルを“全文書き換え”し、既存との差分が巨大化

\* GitHub上でコンフリクト解消を繰り返すうちに、



&nbsp; \* 重要ファイルが消える/重複する

&nbsp; \* 依存関係が壊れる

&nbsp; \* 起動できない

&nbsp; \* どこが正しいか分からない

\* 結果として「Gitの履歴で戻す」より \*\*正常なローカル一式から再構築\*\*のほうが早い状態になる



---



\## 2. 復旧方針（基本戦略）



\*\*GitHubのリポジトリをクローンして“正史”を手元に用意し、そこへ正常なプロジェクトの中身を上書き投入してコミット・pushする。\*\*



ポイント：



\* 「Webアップロード」ではなく \*\*clone → 置換 → commit → push\*\* で復旧する

\* GitHub側に README が先に作られていても、clone方式なら衝突しにくい

\* `node\_modules/` は絶対に上げない（復旧を重く・汚くするだけ）



---



\## 3. 事前準備（必須チェック）



\### 3.1 Gitが使えるか



```powershell

git --version

```



\### 3.2 作業場所を決める



\* 推奨：`E:\\dev\\` や `C:\\dev\\` など \*\*同期の影響が少ない場所\*\*

\* OneDrive配下でもできるが、同期・ロックでハマることがある



---



\## 4. 復旧手順（Windows / PowerShell）



\### 4.1 GitHubリポジトリをクローン



```powershell

cd E:\\dev

git clone https://github.com/<ユーザー名>/<リポジトリ名>.git

cd <リポジトリ名>

```



リモート確認：



```powershell

git remote -v

```



`origin ... (fetch/push)` が同じURLならOK。



\### 4.2 正常なローカル一式をコピー投入



\* “正常なプロジェクト”のフォルダ（例：`TRPG-main`）から、

&nbsp; \*\*中身（src/、package.json等）をクローン先の直下へ\*\*コピーする。

\* フォルダ丸ごと入れると `TRPG/TRPG-main/...` の二重構造になりがち。



&nbsp; \* 意図がなければ「中身だけ」移す。



\*\*コピーしてはいけないもの（重要）\*\*



\* `node\_modules/`

\* `dist/`（ビルド成果物）

\* `.env`（秘密情報）



\### 4.3 .gitignore を用意（最低限）



クローン先リポジトリ直下で：



```powershell

@"

node\_modules/

dist/

.vite/

.env

.env.\*

"@ | Out-File -Encoding utf8 .gitignore

```



表示確認（ドットファイルが隠れるため）：



```powershell

dir -Force

```



\### 4.4 変更内容を確認



```powershell

git status

```



\* `Untracked files` や `Changes not staged` が出れば、復旧用の差分が乗っている



\### 4.5 コミット（作者情報が必要）



\*\*初回だけ\*\* `Author identity unknown` が出たら、作者情報を登録：



```powershell

git config --global user.name "shinta"

git config --global user.email "you@example.com"

```



コミット実行：



```powershell

git add .

git commit -m "Recover project from local snapshot"

```



\### 4.6 GitHubへ push



```powershell

git push

```



\### 4.7 正常性チェック



ローカル：



```powershell

git status

```



`working tree clean` ならOK。



GitHub：



\* ブラウザでリポジトリを開き、`src/` や `package.json` などが期待通りにあるか確認



---



\## 5. よくある詰まりポイントと対処



\### 5.1 `fatal: not a git repository ...`



\* そのフォルダがGit管理されていない（`.git` がない）

\* 解決：\*\*クローン先にいるか確認\*\*、または `git init` のやり忘れ



確認：



```powershell

pwd

ls

```



\### 5.2 READMEが先に作られていて衝突する



\* clone方式なら大抵回避

\* もし `push rejected` などが出る場合：



```powershell

git pull --rebase

git push

```



\### 5.3 `node\_modules` を間違って追跡してしまった



追跡解除（削除ではなく、Git管理から外す）：



```powershell

git rm -r --cached node\_modules

```



その後：



```powershell

git add .

git commit -m "Stop tracking node\_modules"

git push

```



\### 5.4 認証のポップアップが出る



\* Git Credential Manager が GitHub へのアクセス許可を求めている

\* 正しいドメイン（github.com）であれば許可してOK



---



\## 6. 復旧後にやると強いこと（再発防止）



\### 6.1 main を直接触らない



\* Codex反映は \*\*必ずブランチ\*\*で。



```powershell

git checkout -b codex-work

```



\### 6.2 Codexに「全面改稿禁止」を明示



\* 出力は diff 形式（最小差分）を要求

\* 変更単位を小さく（1コミット相当）



\### 6.3 破壊を防ぐガード



\* GitHubのブランチ保護（mainへの直接push禁止、PR必須）

\* Actionsで `npm run build` など最低限のチェック



---



\## 7. 証跡（今回の成功ログ例）



\* `git push` で `main -> main` とコミット範囲が表示されれば成功

\* `git status` が `working tree clean` なら同期完了



---



\## 8. 最短チェックリスト（1分版）



1\. `git clone ...`

2\. クローン先に正常ファイルをコピー（node\_modules抜き）

3\. `.gitignore` 作成

4\. `git add .`

5\. `git commit -m "Recover..."`

6\. `git push`

7\. `git status` が clean



