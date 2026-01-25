# GitHub Runbook: Issue → Branch → PR（＋自動化オプション）

## 0. 目的

Issue を「作業の依頼票（目的・完了条件・議論の箱）」として起点にし、ブランチ作成〜Pull Request（PR）〜クローズまでを一貫した手順で回す。
必要に応じて **AI（Codex）** や **GitHub Actions** を使い、作業の作成・レビュー・修正提案を効率化する。

---

## 1. 適用範囲

* 本リポジトリでの機能追加・バグ修正・ドキュメント修正
* 「Issue を起点に作業を開始する」運用全般

---

## 2. 前提（権限・設定）

### 2.1 GitHub 側

* Issues が有効
* ブランチ作成に必要な権限（通常は write）がある
  ※ Issue からブランチを作る機能は **write 権限が必要** ([GitHub Docs][1])
* （推奨）main への直接 push を禁止（Branch protection）
* （推奨）PR の必須レビュー、必須 CI（Status checks）

### 2.2 Issue / PR テンプレート（推奨）

* Issue テンプレート：`.github/ISSUE_TEMPLATE/` に配置 ([GitHub Docs][2])
* PR テンプレート：`.github/PULL_REQUEST_TEMPLATE.md` などで運用（GitHub Docs参照）

---

## 3. 用語

* **Issue**：課題・要望・バグの「依頼票」。背景、完了条件、会話、証跡をまとめる。
* **Branch**：作業用の分岐。main を汚さないための安全装置。
* **PR（Pull Request）**：変更提案。レビューして main に取り込むための箱。

---

## 4. 標準フロー（手動・最も確実）

> 迷ったらこれ。事故率が最も低い。

### Step 1: Issue を作成する

1. リポジトリ → **Issues** → **New issue** ([GitHub Docs][3])
2. 以下を必ず埋める（テンプレがあればそれに従う）

   * **目的（何を達成するか）**
   * **背景（なぜ必要か）**
   * **受け入れ条件 / Done定義（何が揃えば終わりか）**
   * **影響範囲（影響しそうな画面/機能/ファイル）**
   * **確認方法（手動テスト手順 or 期待ログ）**
3. 必要ならラベル付与（例：`feature` `bug` `docs` `priority:P1`）

### Step 2: Issue からブランチを作成する（推奨）

1. Issue ページで **Create a branch** を押す ([GitHub Docs][1])
2. ブランチ命名例（ルールを固定すると迷子が減る）

   * `feat/123-short-title`
   * `fix/123-short-title`
   * `docs/123-short-title`
3. 作成元は通常 **default branch（main）** ([GitHub Docs][1])

### Step 3: 実装・修正する

1. ローカルでブランチをチェックアウト
2. 変更
3. テスト（最小で良いが「確認方法」に沿うこと）
4. コミット（Issue 番号を含めると追跡が楽）

   * 例：`feat: add xxx (#123)`

### Step 4: PR を作成する（Issue とリンクする）

1. GitHub 上で **Compare & pull request** から PR 作成
2. PR 本文に以下を記載

   * 変更概要
   * テスト結果
   * リスク/ロールバック
3. **Issue を自動クローズ**するなら PR 本文にキーワードを入れる ([GitHub Docs][4])

   * `Fixes #123`
   * `Closes #123`
     ※ マージ時に Issue が自動で閉じる

### Step 5: レビュー → マージ → Issue クローズ確認

1. レビュー対応
2. CI が通ったらマージ
3. Issue が自動で閉じたか確認（閉じない場合は手動クローズ）

---

# 5. オプションA：GitHub標準「Issue→Branch」運用（最短着手）

> “Create a branch” ボタンがある前提。ブランチをすぐ切れる。

* やること自体は **標準フローの Step2 を必須化**するだけ。
* これができると「Issue は作ったけど作業ブランチがない」状態が減る。 ([GitHub Docs][1])

---

# 6. オプションB：Codex を使って「PRを賢く」する（安全寄り）

> **Issue→いきなりAIに全部**は事故りやすいので、Runbookでは“PRを作ってからAIに作業させる”形に寄せる。

## B-1. Codex（GitHub上）でレビューを自動化

### 事前設定

* Codex cloud をセットアップし、対象リポジトリで Code review を有効化 ([OpenAI Developers][5])

### 手順

1. PR を作成（標準フロー Step4 まで）
2. PR コメントで `@codex review` を投稿 ([OpenAI Developers][5])
3. 指摘を反映して更新

## B-2. Codex（GitHub上）に「修正タスク」を投げる

Codex は PR コメントで `@codex` に **review以外** を書くと、PR を文脈に **cloud task** を開始できる ([OpenAI Developers][5])

### 手順

1. PR を **Draft** でもいいので作る（作業ブランチが必要）
2. PR コメントに例のように書く：

   * `@codex fix the CI failures`
   * `@codex implement the acceptance criteria from Issue #123`
3. Codex の提案/コミットを確認し、必要なら追加指示を出す
4. 最終的に人間が差分をレビューしてマージ

## B-3. Codex cloud（ChatGPT側）でタスク→PRへ

* Codex cloud は、リポジトリが読み込まれたクラウド環境でタスクを実行し、結果をレビューして **GitHub PR に持っていく**運用が可能 ([OpenAI][6])
* ただし Workspace 管理者が GitHub connector / Codex cloud を許可している必要がある（組織設定次第で詰まる） ([OpenAI Help Center][7])

---

# 7. オプションC：GitHub Actionsで“自動PR”（定型/CI失敗向け）

> 「CIが落ちたら自動で修正PRを出す」みたいなやつ。便利だが設定を誤ると地獄。

Codex CLI を CI に組み込み、CI 失敗時に修正を生成して PR を作る例が公式にある ([cookbook.openai.com][8])

## 概要（やること）

1. GitHub Actions にワークフロー追加
2. `OPENAI_API_KEY` を GitHub Secrets に登録 ([cookbook.openai.com][8])
3. Actions が PR を作成できる権限設定を有効化 ([cookbook.openai.com][8])
4. 失敗時に Codex CLI を動かして修正 → PR 作成

## ガードレール（推奨）

* 自動PRは **Draft** で作る
* 重要ブランチには必須レビュー＋必須CI
* 変更範囲を限定（特定ディレクトリのみ、など）

---

# 8. トラブルシュート（よくある）

## Issueから「Create a branch」が出ない

* Issuesが無効
* write権限がない ([GitHub Docs][1])
* 組織ポリシーで制限されている

## PRに `Fixes #123` を書いたのにIssueが閉じない

* キーワード構文が違う / Issue番号が違う
* そもそもPRが default branch にマージされていない
* サポートされるキーワードを確認 ([GitHub Docs][4])

## Codexが使えない（連携できない）

* Workspace 管理者が connector / Codex を許可していない可能性 ([OpenAI Help Center][7])
* ChatGPT 側で GitHub 接続ができていない ([OpenAI Help Center][9])

---

# 9. 最終チェックリスト（運用の品質）

* [ ] Issueに **目的** と **Done定義** がある
* [ ] Issue からブランチ作成（または命名規則に沿うブランチ）
* [ ] PRに `Fixes #IssueNo` が書かれている（自動クローズしたい場合） ([GitHub Docs][4])
* [ ] CI が通る
* [ ] レビューを通す（人間 or Codexレビュー） ([OpenAI Developers][5])
* [ ] マージ後に Issue の状態を確認

---

[1]: https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/creating-a-branch-for-an-issue?utm_source=chatgpt.com "Creating a branch to work on an issue"
[2]: https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository?utm_source=chatgpt.com "Configuring issue templates for your repository"
[3]: https://docs.github.com/articles/creating-an-issue?utm_source=chatgpt.com "Creating an issue - GitHub Docs"
[4]: https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue?utm_source=chatgpt.com "Linking a pull request to an issue"
[5]: https://developers.openai.com/codex/integrations/github/ "Use Codex in GitHub"
[6]: https://openai.com/index/introducing-codex/ "Introducing Codex | OpenAI"
[7]: https://help.openai.com/en/articles/11487775-connectors-in-chatgpt?utm_source=chatgpt.com "Apps in ChatGPT"
[8]: https://cookbook.openai.com/examples/codex/autofix-github-actions "Use Codex CLI to automatically fix CI failures"
[9]: https://help.openai.com/en/articles/11145903-connecting-github-to-chatgpt?utm_source=chatgpt.com "Connecting GitHub to ChatGPT"
