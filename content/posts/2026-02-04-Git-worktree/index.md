---
title: "Git worktree"
description: >-
  利用git worktree 提升開發效率
tags: ios, code, chat
toc: true
escape: true
---
現在把使用者資料存在本地，不再存在DB。

## Git worktree
[Git worktree](https://git-scm.com/docs/git-worktree)

利用Git worktree 達到平行開發，大幅提升效率

### **1) 初始狀態（只有一個工作目錄）**

你先正常 clone，目錄大概長這樣 :

```tex
~/dev/acme-web/
├── .git/
├── package.json
├── src/
│   ├── app.ts
│   ├── routes/
│   │   └── index.ts
│   └── ui/
│       └── home.tsx
└── README.md
```

這時你如果要同時做兩個 feature，傳統作法會一直切分支、stash、restore，非常容易打結。

### **2) 用 worktree 建立兩個「平行工作目錄」**

在主 repo 目錄下執行：

```bash
cd ~/dev/acme-web

# 以目前 main 為基礎，建立兩個 worktree（各自對應一個分支）
git worktree add -b feature/oauth-login   ../acme-web-oauth   main
git worktree add -b feature/payment-ui   ../acme-web-pay     main

# 檢查 worktree 狀態
git worktree list
```

建立完成後，你的磁碟上會出現三個資料夾（1 個主 repo + 2 個 worktree) :

```tex
~/dev/
├── acme-web/                      (主工作目錄：通常維持在 main)
│   ├── .git/
│   ├── package.json
│   └── src/...
├── acme-web-oauth/                (worktree：feature/oauth-login)
│   ├── .git                        (注意：這不是完整 git 目錄，通常是指向主 repo 的連結/檔案)
│   ├── package.json
│   └── src/...
└── acme-web-pay/                  (worktree：feature/payment-ui)
    ├── .git
    ├── package.json
    └── src/
```

> 這個結構代表什麼？

* `acme-web/`：你的「主要」工作目錄（常用來看 main、跑整合測試、處理 PR rebase 等）
* `acme-web-oauth/`：對應分支 `feature/oauth-login` 的獨立工作區  
* `acme-web-pay/`：對應分支 `feature/payment-ui` 的獨立工作區  
* 三者共享同一份 Git 物件資料庫（不用再 clone 一次），但工作目錄檔案彼此獨立，所以可同時開兩個 IDE 視窗、同時跑兩個 dev server。

---

### 3) 同時開發：兩個 feature 各自在自己的 worktree 改檔

#### Feature A：OAuth 登入（在 `acme-web-oauth/`）
你在 OAuth worktree 裡新增路由與 UI：

```bash
cd ~/dev/acme-web-oauth
git status
git branch --show-current   # 會看到 feature/oauth-login
```

假設你新增這些檔案：

```
~/dev/acme-web-oauth/
├── src/
│   ├── routes/
│   │   ├── auth.ts              (新增：OAuth callback / login routes)
│   │   └── index.ts             (修改：掛上auth routes)
│   └── ui/
│       └── login.tsx            (新增：Login UI)
└── ...
```

然後你在這個 worktree commit：

```bash
git add .
git commit -m "Add OAuth login routes and login page"
```

#### **Feature B：付款 UI（在** ⁠acme-web-pay/**）**

同時你在付款 worktree 進行 UI 相關改動：

```bash
cd ~/dev/acme-web-pay

git branch --show-current  # 會看到 feature/payment-ui
```

例如新增付款頁與元件：

```tex
~/dev/acme-web-pay/

├── src/
│  └── ui/
│    ├── checkout.tsx     (新增：Checkout 頁)
│    └── components/
│      └── PaymentForm.tsx (新增：付款表單元件)
└── ...
```

並在這個 worktree commit：

```bash
git add .
git commit -m "Add checkout page and payment form UI"
```

### **4) 平行跑起來驗證（避免互相干擾）**

你可以在兩個資料夾各自開終端機跑 dev server（各用不同 port），互不影響：

- ⁠`~/dev/acme-web-oauth`：跑 OAuth login flow
- ⁠`~/dev/acme-web-pay`：跑 checkout / payment form

因為它們是不同工作目錄，所以：

- ⁠npm install、產出檔（例如 ⁠dist/）不會互相踩到 ⁠git status 也不會混在一起

- 你不需要 stash 來回切換

**5) 完成後：合併、清理 worktree**

當其中一個 feature 已經合併回 ⁠main（例如 PR merge）後，你可以移除那個 worktree：

```bash
cd ~/dev/acme-web

git worktree remove ../acme-web-oauth
```

如果對應分支也不需要了，再刪分支：

```bash
git branch -d feature/oauth-login
```

## Merge process

以下假設你的 `~/dev/acme-web` 是主 worktree，平常停在 `main`。

------

### 0) 一次性準備：確認 repo 與 `gh` 可用

在 repo 裡先把 `main` 更新到最新，避免你測到舊狀態。

```bash
cd ~/dev/acme-web
git switch main
git pull --ff-only
```

（可選）列出目前 open PR，方便你決定要測哪些編號。[Source](https://cli.github.com/manual/gh_pr)

```bash
gh pr list
```

------

### 1) 每個 PR 的標準流程（建議用「臨時 worktree」隔離測試環境）

這樣你不會把 `main` worktree 切來切去，也能同時保留其他 worktree（例如你自己的 feature 開發目錄）。[Source](https://git-scm.com/docs/git-worktree)

#### 1.1 建一個專用 worktree（從最新 `main` 開）

```bash
cd ~/dev/acme-web
git worktree add -b pr-123 ../acme-web-pr123 main
cd ../acme-web-pr123
```

#### 1.2 把 PR checkout 到這個 worktree（用 `gh`）

用 `--branch pr-123 --force` 可以確保這個 worktree 的分支就是 PR 最新狀態（不被舊本地分支干擾）。[Source](https://cli.github.com/manual/gh_pr_checkout)

```bash
gh pr checkout 123 --branch pr-123 --force
```

#### 1.3 （可選但很常用）把 PR 分支更新到最新 base（main）

如果你們 policy 要求「branch 必須 up-to-date 才能 merge」，你可以在測試前先更新 PR 分支；預設是 **merge base into head**，想改成 rebase 用 `--rebase`。[Source](https://cli.github.com/manual/gh_pr_update-branch)

```bash
gh pr update-branch 123
# 或：gh pr update-branch 123 --rebase
```

> 如果你在這一步更新了 PR，GitHub 會產生新的 commit（merge 或 rebase），所以**建議更新後再跑測試**一次再 merge。[Source](https://cli.github.com/manual/gh_pr_update-branch)

#### 1.4 跑你本機測試（你們專案自己的指令）

```bash
# 例：
# pnpm i
# pnpm test
# pnpm e2e
```

#### 1.5 等 CI（必要時）並在 CLI 看到 checks 結果

你可以用 `--watch` 等到 checks 結束，搭配 `--required` 只看 required checks；若還在跑會用 exit code 8 表示 pending（方便你寫腳本）。[Source](https://cli.github.com/manual/gh_pr_checks)

```bash
gh pr checks 123 --required --watch --fail-fast
```

#### 1.6 用 `gh` 直接 merge（這一步會在 GitHub 上完成合併）

你要選擇你們團隊的合併策略（擇一）：[Source](https://cli.github.com/manual/gh_pr_merge)

```bash
# squash merge
gh pr merge 123 --squash

# 或 merge commit
# gh pr merge 123 --merge

# 或 rebase merge
# gh pr merge 123 --rebase
```

（可選）你想「等規則都滿足才自動 merge」就加 `--auto`（常見於 required checks / required reviews）。

```bash
gh pr merge 123 --squash --auto
```

（可選）刪 remote 分支：通常可以加 `--delete-branch`，但若 repo 啟用了 **merge queue** 之類限制，CLI 行為可能受規則影響；更穩的做法是到 repo 設定開啟「自動刪除已合併的 head branch」。[Source](https://cli.github.com/manual/gh_pr_merge)

```bash
gh pr merge 123 --squash --delete-branch
```

------

### 2) merge 之後：讓「remote / local main」同步（你才能測下一個 PR 的最新基底）

合併完成後，回到主 worktree 把 `main` 拉到最新。[Source](https://cli.github.com/manual/gh_pr_merge)

```bash
cd ~/dev/acme-web
git switch main
git pull --ff-only
```

------

### 3) 清理：移除該 PR worktree（保持環境乾淨）

測完＋merge 完，建議把 PR worktree 刪掉，避免本機堆一堆暫時分支與目錄。[Source](https://git-scm.com/docs/git-worktree)

```bash
cd ~/dev/acme-web
git worktree remove ../acme-web-pr123
git branch -D pr-123
```

------

### 4) 針對下一個 PR（例如 #456）重複同一套

你只要把編號換掉即可；核心節奏是：**checkout →（可選）update-branch → 測試 → merge → pull main → cleanup**。[Source](https://cli.github.com/manual/gh_pr_checkout)

```bash
# 建 worktree
cd ~/dev/acme-web
git worktree add -b pr-456 ../acme-web-pr456 main
cd ../acme-web-pr456

# 拉 PR
gh pr checkout 456 --branch pr-456 --force

# 更新 + 測試 + checks
gh pr update-branch 456
gh pr checks 456 --required --watch --fail-fast

# merge
gh pr merge 456 --squash

# 同步 main + 清理
cd ~/dev/acme-web
git switch main
git pull --ff-only
git worktree remove ../acme-web-pr456
git branch -D pr-456
```

------

### （可選）你想「更嚴謹」：測 PR “合進 main 後的結果” 而不是只測 PR head

`gh pr checkout` 主要是把 PR 分支（head）抓下來測；若你要測「跟 `main` 合併後會不會炸」，一般會改用 GitHub 的 `pull/<id>/merge` 參照來建立 worktree，再跑整合測試（merge 動作仍可用 `gh pr merge`）。[Source](https://cli.github.com/manual/gh_pr_checkout)
