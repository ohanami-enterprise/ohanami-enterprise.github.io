# コーポレートサイト 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 合同会社おはなみエンタープライズの会社紹介1ページLPを Hugo で構築し、GitHub Pages（`https://ohanami-enterprise.github.io`）で公開する。

**Architecture:** テーマを使わない軽量な Hugo サイト。`layouts/index.html` 1枚に Hero（社名＋猫2匹SVG）と会社概要テーブルを収め、会社情報は `hugo.toml` の params から流し込む。CSS は `static/css/main.css`。デプロイは GitHub Actions の公式 Hugo ワークフローで `main` push 時に自動公開。

**Tech Stack:** Hugo（extended 不要・plain CSS）、HTML/CSS、インラインSVG、GitHub Actions、GitHub Pages。

> **テスト方針の注記:** 静的サイトのため JS 等のユニットテストフレームワークは導入しない。各タスクの「テスト」は (1) `hugo` がエラーなくビルドできること、(2) 生成された `public/index.html` に期待する内容が含まれること（grep で検証）、(3) `hugo server` での目視確認、で代替する。これが本サイトにとって意味のある検証である。

**作業ディレクトリ:** `/home/gurimusan/projects/ohanami-enterprise.github.io`（git 初期化済み・設計書コミット済み）

---

## 前提セットアップ（実装開始前に一度だけ）

- [ ] **P-1: Hugo をインストール**

Gentoo のパッケージマネージャ（推奨）:
```bash
sudo emerge -a www-apps/hugo
```
または sudo を使わずバイナリ取得（最新版タグを確認して使用）:
```bash
mkdir -p ~/.local/bin
# https://github.com/gohugoio/hugo/releases で最新版を確認し <VERSION> を置換
curl -L "https://github.com/gohugoio/hugo/releases/download/v<VERSION>/hugo_<VERSION>_linux-amd64.tar.gz" \
  | tar xz -C ~/.local/bin hugo
# ~/.local/bin が PATH に無ければ通す
export PATH="$HOME/.local/bin:$PATH"
```
検証:
```bash
hugo version
```
期待: `hugo v0.1xx.x ...` のようにバージョンが表示される。

- [ ] **P-2: GitHub Organization とリポジトリを作成（github.com 上で本人が実施）**

  1. github.com → 右上 ＋ → **New organization** → Free プランで `ohanami-enterprise` を作成。
  2. その Org 配下に **New repository** で `ohanami-enterprise.github.io` を作成（public・READMEなし・空のまま）。
  3. ローカルにリモートを登録（実装の最後 Task 7 で push する）:
  ```bash
  cd /home/gurimusan/projects/ohanami-enterprise.github.io
  git remote add origin https://github.com/ohanami-enterprise/ohanami-enterprise.github.io.git
  ```
  検証: `git remote -v` で origin が表示される。

  > Org 名が `ohanami-enterprise` 以外になった場合は、本計画中の `baseURL` とリモートURLを実際の名前に読み替えること。

---

## Task 1: Hugo サイト設定

**Files:**
- Create: `hugo.toml`

- [ ] **Step 1: `hugo.toml` を作成**

```toml
baseURL = "https://ohanami-enterprise.github.io/"
languageCode = "ja"
title = "合同会社おはなみエンタープライズ"
enableRobotsTXT = true

[params]
  companyName = "合同会社おはなみエンタープライズ"
  established = "2026年（登記完了後に確定）"
  representative = "松下永壽"
  capital = "500,000円"
  business = "ソフトウェア・ハードウェアの企画／開発／コンサルティング"
  description = "合同会社おはなみエンタープライズ — ソフトウェア・ハードウェアの企画／開発／コンサルティング"
```

- [ ] **Step 2: ビルドできることを確認**

Run:
```bash
cd /home/gurimusan/projects/ohanami-enterprise.github.io && hugo --gc --minify
```
Expected: エラーなく完了し `public/` が生成される（この時点では index.html は空に近い）。`Total in ...ms` が表示される。

- [ ] **Step 3: コミット**

```bash
git add hugo.toml
git commit -m "feat: Hugoサイト設定を追加"
```

---

## Task 2: デザイントークンとベースCSS

**Files:**
- Create: `static/css/main.css`

- [ ] **Step 1: `static/css/main.css` を作成**

```css
:root {
  --color-bg: #faf5ee;        /* 生成り（washi） */
  --color-text: #2a221e;      /* 墨色 */
  --color-text-sub: #6b5f54;  /* 補助テキスト */
  --color-accent: #a98a6a;    /* 茶アクセント */
  --color-accent-deep: #8a6a4a;
  --color-line: #e3d8c8;      /* 罫線 */

  --font-serif: "Hiragino Mincho ProN", "Yu Mincho", "YuMincho", "MS PMincho", serif;

  --space-section: clamp(3rem, 2rem + 5vw, 6rem);
  --maxw: 880px;
}

* { box-sizing: border-box; }

html { -webkit-text-size-adjust: 100%; }

body {
  margin: 0;
  background: var(--color-bg);
  color: var(--color-text);
  font-family: var(--font-serif);
  line-height: 1.8;
  font-feature-settings: "palt";
}

.wrap {
  max-width: var(--maxw);
  margin: 0 auto;
  padding: 0 24px;
}

/* Hero */
.hero {
  position: relative;
  min-height: 78vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  overflow: hidden;
}
.hero__brand {
  font-size: clamp(1.8rem, 1rem + 4vw, 3.4rem);
  font-weight: 600;
  letter-spacing: 0.04em;
  line-height: 1.5;
  margin: 0;
}
.hero__cats {
  position: absolute;
  right: clamp(8px, 4vw, 48px);
  bottom: 0;
  display: flex;
  align-items: flex-end;
  gap: 2px;
  pointer-events: none;
}

/* About */
.about {
  padding: var(--space-section) 0;
}
.about__title {
  font-size: 1.3rem;
  letter-spacing: 0.1em;
  color: var(--color-accent-deep);
  margin: 0 0 1.5rem;
  font-weight: 600;
}
.about__title span { font-size: 0.8rem; color: var(--color-text-sub); margin-left: .6em; letter-spacing: .15em; }
.about__table {
  width: 100%;
  border-collapse: collapse;
}
.about__table th,
.about__table td {
  text-align: left;
  padding: 16px 8px;
  border-bottom: 1px solid var(--color-line);
  vertical-align: top;
}
.about__table th {
  width: 28%;
  font-weight: 600;
  color: var(--color-text-sub);
  white-space: nowrap;
}

/* Footer */
.site-footer {
  padding: 2.5rem 0;
  font-size: 0.78rem;
  color: var(--color-text-sub);
  border-top: 1px solid var(--color-line);
}
```

- [ ] **Step 2: コミット**（このCSSは次タスクで参照される。単体ビルドは不要）

```bash
git add static/css/main.css
git commit -m "feat: 和モダンのデザイントークンとベースCSSを追加"
```

---

## Task 3: トップページのレイアウト（Hero＋会社概要）

**Files:**
- Create: `layouts/index.html`

> 猫SVGはこのタスクでは仮の四角プレースホルダにしておき、Task 4 で確定SVGに差し替える。

- [ ] **Step 1: `layouts/index.html` を作成**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ site.Title }}</title>
  <meta name="description" content="{{ site.Params.description }}">
  <link rel="stylesheet" href="{{ "css/main.css" | relURL }}">
</head>
<body>
  <header class="hero">
    <div class="wrap">
      <h1 class="hero__brand">{{ site.Params.companyName }}</h1>
    </div>
    <div class="hero__cats" aria-hidden="true">
      <!-- CATS_PLACEHOLDER (Task 4 で差し替え) -->
      <div style="width:118px;height:153px;background:#eadfce;"></div>
      <div style="width:92px;height:120px;background:#e0d3bf;"></div>
    </div>
  </header>

  <main>
    <section class="about wrap">
      <h2 class="about__title">会社概要<span>ABOUT</span></h2>
      <table class="about__table">
        <tbody>
          <tr><th>商号</th><td>{{ site.Params.companyName }}</td></tr>
          <tr><th>設立</th><td>{{ site.Params.established }}</td></tr>
          <tr><th>代表社員</th><td>{{ site.Params.representative }}</td></tr>
          <tr><th>資本金</th><td>{{ site.Params.capital }}</td></tr>
          <tr><th>事業内容</th><td>{{ site.Params.business }}</td></tr>
        </tbody>
      </table>
    </section>
  </main>

  <footer class="site-footer">
    <div class="wrap">© {{ now.Year }} {{ site.Params.companyName }}</div>
  </footer>
</body>
</html>
```

- [ ] **Step 2: ビルドして出力に会社情報が含まれることを確認**

Run:
```bash
hugo --gc --minify && grep -c "松下永壽" public/index.html && grep -c "ソフトウェア・ハードウェア" public/index.html && grep -c "500,000円" public/index.html
```
Expected: ビルド成功後、3行とも `1`（以上）が出力される（各文字列が含まれる）。

- [ ] **Step 3: ローカルで目視確認**

Run:
```bash
hugo server -D
```
ブラウザで `http://localhost:1313` を開き、Hero に社名、下に会社概要テーブルが和モダンの配色で表示されることを確認（猫はまだ四角のプレースホルダ）。確認したら Ctrl+C で停止。

- [ ] **Step 4: コミット**

```bash
git add layouts/index.html
git commit -m "feat: トップページ（Hero＋会社概要）を追加"
```

---

## Task 4: 猫2匹の確定SVGを設置

**Files:**
- Modify: `layouts/index.html`（`hero__cats` 内のプレースホルダを差し替え）

- [ ] **Step 1: `hero__cats` のプレースホルダ2つの `<div>` を、確定版の猫SVG2つに置き換える**

`layouts/index.html` の以下のブロック:
```html
      <!-- CATS_PLACEHOLDER (Task 4 で差し替え) -->
      <div style="width:118px;height:153px;background:#eadfce;"></div>
      <div style="width:92px;height:120px;background:#e0d3bf;"></div>
```
を、次の内容に置き換える（大きい方＝白茶の牛柄・短く太い鍵尻尾／小さい方＝三毛・頭が黒）:
```html
      <!-- 大きい方：白茶の牛柄・短く太い鍵尻尾 -->
      <svg viewBox="0 0 100 130" width="118" height="153" role="img" aria-label="白茶の猫">
        <path d="M66 114 C75 114 78 106 75 101 L80 98" fill="none" stroke="#fdfcf8" stroke-width="16" stroke-linecap="round" stroke-linejoin="round"/>
        <path d="M75 101 L80 98" fill="none" stroke="#d99b5f" stroke-width="16" stroke-linecap="round" stroke-linejoin="round"/>
        <path d="M50 54 C26 54 24 122 50 122 C76 122 74 54 50 54Z" fill="#fdfcf8" stroke="#c9bba8" stroke-width="1.6"/>
        <path d="M34 78 C30 86 31 98 38 100 C44 98 42 86 40 80 C38 76 36 76 34 78Z" fill="#e0a366"/>
        <path d="M56 92 C54 104 58 114 64 112 C68 106 64 96 60 92 C58 90 57 90 56 92Z" fill="#dc9a58"/>
        <path d="M44 60 C40 62 41 70 46 70 C50 68 49 62 47 60 C46 59 45 59 44 60Z" fill="#e0a366"/>
        <circle cx="50" cy="46" r="25" fill="#fdfcf8" stroke="#c9bba8" stroke-width="1.6"/>
        <path d="M29 36 L25 9 L47 28Z" fill="#fdfcf8" stroke="#c9bba8" stroke-width="1.4"/>
        <path d="M71 36 L75 9 L53 28Z" fill="#d99b5f" stroke="#c9bba8" stroke-width="1.4"/>
        <path d="M53 26 C68 24 72 38 67 50 C60 54 53 48 52 40 C51 33 51 28 53 26Z" fill="#e0a366"/>
        <path d="M37 47 q4 4 8 0" fill="none" stroke="#3a2f2a" stroke-width="2" stroke-linecap="round"/>
        <path d="M55 47 q4 4 8 0" fill="none" stroke="#2a2420" stroke-width="2" stroke-linecap="round"/>
        <path d="M48 53 l4 0 l-2 2.5Z" fill="#d98a8a"/>
        <path d="M30 51 h13 M30 56 h12" stroke="#c9bba8" stroke-width="1" fill="none"/>
        <path d="M70 51 h-13 M70 56 h-12" stroke="#c9bba8" stroke-width="1" fill="none"/>
      </svg>
      <!-- 小さい方：三毛・頭が黒（ヘルメット）・黒い鍵尻尾 -->
      <svg viewBox="0 0 100 130" width="92" height="120" role="img" aria-label="三毛の猫">
        <path d="M32 108 C0 108 0 70 18 74" fill="none" stroke="#2a2420" stroke-width="11" stroke-linecap="round"/>
        <path d="M50 58 C28 58 26 122 50 122 C74 122 72 58 50 58Z" fill="#fdfcf8" stroke="#c9bba8" stroke-width="1.6"/>
        <path d="M50 60 C40 60 35 84 40 104 C44 92 46 76 52 70 C50 64 50 60 50 60Z" fill="#2a2420"/>
        <path d="M58 78 C66 80 70 96 64 112 C60 100 58 90 58 78Z" fill="#e7a25c"/>
        <circle cx="50" cy="48" r="24" fill="#2a2420"/>
        <path d="M30 38 L26 11 L48 30Z" fill="#2a2420"/>
        <path d="M70 38 L74 11 L52 30Z" fill="#2a2420"/>
        <path d="M27 44 C27 61 38 71 50 71 C62 71 73 61 73 44 C64 48 36 48 27 44 Z" fill="#fdfcf8"/>
        <path d="M37 53 q4 4 8 0" fill="none" stroke="#3a2f2a" stroke-width="2" stroke-linecap="round"/>
        <path d="M55 53 q4 4 8 0" fill="none" stroke="#3a2f2a" stroke-width="2" stroke-linecap="round"/>
        <path d="M48 60 l4 0 l-2 2.5Z" fill="#d98a8a"/>
        <path d="M30 60 h12 M31 65 h11" stroke="#c9bba8" stroke-width="1" fill="none"/>
        <path d="M70 60 h-12 M69 65 h-11" stroke="#c9bba8" stroke-width="1" fill="none"/>
      </svg>
```

- [ ] **Step 2: ビルドして猫SVGが出力に含まれることを確認**

Run:
```bash
hugo --gc --minify && grep -c "白茶の猫" public/index.html && grep -c "三毛の猫" public/index.html && grep -c "CATS_PLACEHOLDER" public/index.html
```
Expected: 最初の2つは `1`、最後（プレースホルダ残骸）は `0`。

- [ ] **Step 3: ローカルで目視確認**

```bash
hugo server
```
`http://localhost:1313` で Hero 右下に猫2匹（大＝白茶牛柄・短い鍵尻尾／小＝三毛・頭黒）が表示されることを確認。Ctrl+C で停止。

- [ ] **Step 4: コミット**

```bash
git add layouts/index.html
git commit -m "feat: Heroに猫2匹の確定SVGを設置"
```

---

## Task 5: レスポンシブ調整

**Files:**
- Modify: `static/css/main.css`（末尾にメディアクエリを追加）

- [ ] **Step 1: `static/css/main.css` の末尾に以下を追加**

```css
/* スマホ: 猫を縮小し、会社概要テーブルを縦並びに */
@media (max-width: 640px) {
  .hero { min-height: 70vh; }
  .hero__cats svg:first-child { width: 84px; height: 109px; }
  .hero__cats svg:last-child  { width: 66px; height: 86px; }

  .about__table th,
  .about__table td {
    display: block;
    width: 100%;
    padding: 6px 4px;
  }
  .about__table th {
    border-bottom: none;
    padding-top: 16px;
    color: var(--color-accent-deep);
  }
  .about__table td { padding-bottom: 16px; }
}
```

- [ ] **Step 2: スマホ幅で目視確認**

```bash
hugo server
```
ブラウザの開発者ツールで幅を 375px 程度にし、(1) 猫がはみ出さず縮小される、(2) 会社概要が「項目→内容」の縦積みで読める、ことを確認。Ctrl+C で停止。

- [ ] **Step 3: コミット**

```bash
git add static/css/main.css
git commit -m "feat: スマホ向けレスポンシブ調整を追加"
```

---

## Task 6: GitHub Actions デプロイワークフロー

**Files:**
- Create: `.github/workflows/hugo.yml`

- [ ] **Step 1: `.github/workflows/hugo.yml` を作成**

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.140.2
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
        run: hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

> `HUGO_VERSION` は P-1 で確認した実際のバージョンに合わせてよい（0.140.2 は例）。extended 版を使うが plain CSS でも問題なく動く。

- [ ] **Step 2: ローカルで YAML 妥当性を確認**

Run:
```bash
python3 -c "import yaml,sys; yaml.safe_load(open('.github/workflows/hugo.yml')); print('YAML OK')"
```
Expected: `YAML OK`

- [ ] **Step 3: コミット**

```bash
git add .github/workflows/hugo.yml
git commit -m "ci: GitHub PagesへのHugoデプロイワークフローを追加"
```

---

## Task 7: ビルド成果物の除外・README・公開

**Files:**
- Modify: `.gitignore`
- Create: `README.md`

- [ ] **Step 1: `.gitignore` にビルド成果物を追加**

現在の `.gitignore`（`.superpowers/` のみ）に追記して、最終的に以下の内容にする:
```gitignore
.superpowers/
/public/
/resources/
.hugo_build.lock
```

- [ ] **Step 2: `README.md` を作成**

```markdown
# ohanami-enterprise.github.io

合同会社おはなみエンタープライズ コーポレートサイト。

- 公開URL: https://ohanami-enterprise.github.io
- 構築: Hugo（テーマ無し・独自レイアウト）
- デプロイ: `main` への push で GitHub Actions が自動公開

## ローカル開発

```bash
hugo server
# http://localhost:1313
```

## 設計・計画ドキュメント

- 設計書: `docs/superpowers/specs/2026-06-10-corporate-site-design.md`
- 実装計画: `docs/superpowers/plans/2026-06-10-corporate-site.md`
```

- [ ] **Step 3: クリーンビルドが通る最終確認**

Run:
```bash
rm -rf public resources && hugo --gc --minify && test -f public/index.html && echo "BUILD OK"
```
Expected: `BUILD OK`

- [ ] **Step 4: コミット**

```bash
git add .gitignore README.md
git commit -m "chore: ビルド成果物の除外とREADMEを追加"
```

- [ ] **Step 5: リモートへ push して公開（前提 P-2 完了後）**

Run:
```bash
git branch -M main
git push -u origin main
```
その後 github.com のリポジトリ → **Settings → Pages → Build and deployment → Source** を **GitHub Actions** に設定する。

- [ ] **Step 6: 公開確認**

Actions タブでワークフローが成功したら、`https://ohanami-enterprise.github.io` をブラウザで開き、Hero（社名＋猫2匹）と会社概要が表示されることを確認する。

---

## セルフレビュー結果

- **スペック網羅:** ホスティング/技術（Task 1,6,7・前提P）、和モダンデザイン（Task 2,5）、猫2匹SVG（Task 4）、Hero＋会社概要構成（Task 3）、所在地非掲載（Task 3 のテーブルに含めない）、スコープ外項目（実装せず）— すべて対応タスクあり。
- **プレースホルダ:** 各コード手順に実コードを記載。`HUGO_VERSION` と Hugo 取得URLの `<VERSION>` のみ環境依存のため最新版確認を明記（仕様上の保留であり計画の穴ではない）。
- **型/名称整合:** params 名（companyName / established / representative / capital / business / description）は Task 1 定義と Task 3 参照で一致。CSS クラス名（hero / hero__brand / hero__cats / about / about__table 等）は Task 2 定義と Task 3/5 参照で一致。猫SVGの `aria-label`（白茶の猫 / 三毛の猫）は Task 4 と検証 grep で一致。
- **保留事項:** 設立日は登記完了後に `hugo.toml` の `established` を更新（設計書 §6 準拠）。
