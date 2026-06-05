# Claude Code 完整指南 — 開發說明

本文件說明專案結構、撰寫規範、以及新增/修改章節時 AI 必須遵守的編排規則。

---

## 目錄結構

```
claude-code-guide/
├── src/
│   ├── components/
│   │   └── CodeBlock.astro        # 程式碼區塊元件（含複製按鈕）
│   ├── data/
│   │   └── navigation.js          # 側邊欄導覽結構（新增章節必改）
│   ├── layouts/
│   │   └── DocsLayout.astro       # 全站共用 Layout（含 Header、Sidebar、CSS）
│   └── pages/
│       ├── index.astro            # 首頁
│       ├── ch01-what-is-cc.astro  # 第 1 章
│       ├── ...                    # ch02 ~ ch20
│       ├── about.astro            # 附錄：關於作者
│       └── (新章節放這裡)
├── public/
│   └── img/                       # 靜態圖片資源
├── astro.config.mjs               # output: 'static'（純靜態，無後端）
└── package.json
```

---

## 技術棧

| 項目 | 說明 |
|------|------|
| Framework | Astro 6.x（`output: 'static'`） |
| 語法高亮 | Shiki（透過 `astro:components` 的 `<Code>`） |
| CSS 策略 | Scoped + `is:global`（見下方說明） |
| 部署 | 純靜態，`npm run build` → `dist/` 上傳 AWS S3/Cloudfront |

---

## 開發指令

```bash
npm run dev      # 開發預覽 → http://localhost:4321
npm run build    # 產出 dist/ 靜態檔案
npm run preview  # 預覽 build 結果
```

---

## 新增章節 — 標準流程

### 1. 更新 `navigation.js`

```js
// src/data/navigation.js
{ num: 21, title: "新章節標題", slug: "ch21-slug" }
```

- `num`：章節編號（連續整數）
- `slug`：對應 `src/pages/{slug}.astro` 的檔名（不含副檔名）
- 附錄用 `part: 8`，正文用 `part: 1~7`

### 2. 建立頁面檔案

```astro
---
import DocsLayout from '../layouts/DocsLayout.astro';
import CodeBlock from '../components/CodeBlock.astro';
---
<DocsLayout
  title="章節標題"
  currentSlug="ch21-slug"
  description="SEO 描述（80~120 字）"
  prevChapter={{ title: "上一章標題", slug: "ch20-faq" }}
  nextChapter={{ title: "下一章標題", slug: "ch22-xxx" }}
>
  <p class="clabel">第 21 章 · Part X：Part 標題</p>
  <h1>章節大標題</h1>
  <p class="cdesc">章節副標題描述句。</p>

  <h2>第一個 Section</h2>
  <p>內容...</p>

  <CodeBlock lang="bash" label="bash" code={`指令`} />

</DocsLayout>
```

### 3. 驗證

```bash
npm run build    # 必須 0 errors 才算完成
```

---

## CodeBlock 元件用法

```astro
<!-- 基本用法 -->
<CodeBlock lang="bash" code={`npm install`} />

<!-- 加上 label（顯示在 bar 上方） -->
<CodeBlock lang="typescript" label="src/index.ts" code={`const x = 1;`} />

<!-- 多行 / 含特殊字元 → 用 frontmatter const 避免 Astro 解析錯誤 -->
---
const myCode = `
import { foo } from "bar";
const result = foo({ key: "value" });
`;
---
<CodeBlock lang="typescript" label="example.ts" code={myCode} />
```

### ⚠️ 重要：在 template 中寫 code 字串的限制

Astro template（`---` 外面的 HTML 區塊）對以下字元有特殊解析：

| 字元 | 問題 | 解法 |
|------|------|------|
| `` ` `` | 啟動 template literal | 用 `&#96;` 或移到 frontmatter const |
| `{` | 啟動 JSX expression | 用 `&#123;` 或移到 frontmatter const |
| `}` | 結束 JSX expression | 用 `&#125;` 或移到 frontmatter const |

**原則：凡是含有 `${}`, `{}`, `` ` `` 的程式碼字串，一律移到 frontmatter 的 `const` 變數。**

---

## 常用 HTML 元素

### 標題層級

```html
<p class="clabel">第 N 章 · Part X：Part 名稱</p>   <!-- 橘色小標籤 -->
<h1>章節主標題</h1>
<p class="cdesc">章節一句話描述</p>                   <!-- 灰色副標，下方有分隔線 -->
<h2>段落標題</h2>                                     <!-- 有上下分隔線 -->
<h3>子段落標題</h3>
```

### Callout 區塊

```html
<!-- 綠色提示 -->
<div class="call tip">
  <span class="ci">💡</span>
  <div>
    <strong>標題</strong>
    <p>內容</p>
  </div>
</div>

<!-- 藍色資訊 -->
<div class="call info">
  <span class="ci">ℹ️</span>
  <div>...</div>
</div>

<!-- 橘色警告 -->
<div class="call warn">
  <span class="ci">⚠️</span>
  <div>...</div>
</div>
```

### 表格

```html
<table>
  <thead>
    <tr><th>欄位A</th><th>欄位B</th></tr>
  </thead>
  <tbody>
    <tr><td>值1</td><td>值2</td></tr>
  </tbody>
</table>
```

- 表頭下方自動顯示 2px accent 橘紅色線
- 資料行有 hover 背景效果
- 必須有 `<thead>` / `<tbody>` 結構，否則樣式不會正確

### 圖片

```html
<figure class="ss">
  <img src="/img/filename.png" alt="描述文字" />
  <figcaption>圖片說明</figcaption>
</figure>
```

圖片放 `public/img/` 目錄，路徑從 `/img/` 開頭。

---

## CSS 架構重點

### Scoped vs Global

DocsLayout.astro 有**兩個** `<style>` 區塊：

| 區塊 | 類型 | 負責 |
|------|------|------|
| `<style>` | Scoped（加 Astro CID） | Layout 自身：Header、Sidebar、Chapter Nav、Mobile |
| `<style is:global>` | Global（無 CID） | 內容樣式：Typography、Table、Callout |

**為什麼分開？**

Astro scoped CSS 會把 `.cb th` 編譯成 `.cb[data-astro-cid-XXX] th[data-astro-cid-XXX]`，要求 `th` 也帶有相同 CID。但 `<slot />` 裡的元素（來自各章節頁面）沒有 layout 的 CID → 所有 Typography、Table 樣式全部失效。

解法：內容樣式用 `<style is:global>` 確保不加 CID 限制。

### CSS 變數（`--var`）

定義在 `:root` 及 `html.dark` 中：

| 變數 | 說明 |
|------|------|
| `--accent` | 橘紅色 `#c0533a`（dark: `#e07050`） |
| `--bg` / `--bg2` / `--bg3` | 背景層次 |
| `--text` / `--text2` / `--muted` | 文字層次 |
| `--border` | 分隔線顏色 |
| `--serif` | 標題字型（Georgia） |
| `--mono` | 等寬字型（Fira Code） |

---

## 章節目錄（22 頁）

| Part | 章節 | Slug |
|------|------|------|
| 1 入門 | Ch01 什麼是 CC？ | `ch01-what-is-cc` |
| 1 入門 | Ch02 安裝與環境設定 | `ch02-install` |
| 1 入門 | Ch03 第一次對話 | `ch03-first-chat` |
| 2 模型選擇 | Ch04 三大模型比較 | `ch04-models` |
| 2 模型選擇 | Ch05 如何切換模型 | `ch05-model-switch` |
| 3 設定與記憶 | Ch06 CLAUDE.md | `ch06-claude-md` |
| 3 設定與記憶 | Ch07 Rules | `ch07-rules` |
| 3 設定與記憶 | Ch08 Memory.md | `ch08-memory` |
| 4 Hooks | Ch09 Hooks 是什麼？ | `ch09-hooks-intro` |
| 4 Hooks | Ch10 實戰：自動日誌 | `ch10-hooks-diary` |
| 4 Hooks | Ch11 Hooks 安全性 | `ch11-hooks-security` |
| 5 Skills | Ch12 官方六大類 | `ch12-skills-intro` |
| 5 Skills | Ch13 使用內建 Skills | `ch13-skills-builtin` |
| 5 Skills | Ch14 自己寫 Skill | `ch14-skills-custom` |
| 6 MCP Plugin | Ch15 MCP 是什麼？ | `ch15-mcp-intro` |
| 6 MCP Plugin | Ch16 安裝 MCP Server | `ch16-mcp-install` |
| 6 MCP Plugin | Ch17 自製 MCP Server | `ch17-mcp-custom` |
| 7 進階技巧 | Ch18 多 Agent 協作 | `ch18-multi-agent` |
| 7 進階技巧 | Ch19 Permissions 模式 | `ch19-permissions` |
| 7 進階技巧 | Ch20 常見問題 & Debug | `ch20-faq` |
| 8 附錄 | 關於作者 | `about` |
| — | 首頁 | `index` |

---

## 已知陷阱與解法

### 1. Astro Template 解析錯誤

**症狀**：`Expected "}" but found ":"` 或 build 失敗

**原因**：程式碼字串含 `` ` ``、`{`、`}` 被 Astro template parser 誤解析

**解法**：複雜程式碼一律移到 frontmatter `const`：
```astro
---
const complexCode = `const x = ${someVar};`;
---
<CodeBlock code={complexCode} />
```

### 2. CSS 樣式對 Slot 內容無效

**症狀**：表格、標題字型、Callout 顏色在頁面上看不到效果

**原因**：Astro scoped CSS 加了 CID 限制，slot 內容沒有 layout 的 CID

**解法**：內容樣式必須放在 `<style is:global>` 區塊（已修正）

### 3. CSS Media Query 空格

**症狀**：build 警告 `Expected "{" but found "and("`

**原因**：CSS minifier 需要 `and` 前後有空格

**解法**：`@media (min-width:769px) and (max-width:1024px)` ← `and` 前後必須有空格

### 4. 複製按鈕超出 Code Block

**症狀**：單行程式碼時「複製」按鈕跑出 pre 外

**原因**：按鈕用 `position: absolute` 時 pre 寬度計算問題

**解法**：複製按鈕已改為放在 label bar 右側（flex 排版），不再使用 absolute positioning

---

## 圖片規範

- 存放路徑：`public/img/`
- 引用路徑：`/img/filename.png`（從根目錄起）
- 建議格式：PNG（截圖）、WebP（照片）
- 建議寬度：最大 740px（符合 `.cb` 內容欄寬）
