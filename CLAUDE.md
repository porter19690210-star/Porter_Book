# Claude Code 指南網站 — AI 撰寫規範

## 專案概述
這是一個用 Astro 6.x 建置的純靜態文件網站，介紹 Claude Code 的完整使用方式。
`npm run build` → `dist/` 純靜態，部署到 AWS S3。

---

## 一、新增章節的標準步驟

### Step 1：更新 `src/data/navigation.js`

在對應的 `part` 陣列加入新章節：
```js
{ num: N, title: "章節標題", slug: "chNN-slug" }
```
- `slug` 必須對應 `src/pages/{slug}.astro` 的檔名
- 附錄使用 `part: 8`，正文 `part: 1~7`

### Step 2：建立 `src/pages/{slug}.astro`

使用以下 boilerplate，逐項填入內容：

```astro
---
import DocsLayout from '../layouts/DocsLayout.astro';
import CodeBlock from '../components/CodeBlock.astro';

// 含特殊字元的程式碼字串一律放在這裡
const exampleCode = `your code here`;
---
<DocsLayout
  title="章節標題"
  currentSlug="chNN-slug"
  description="SEO 描述，80~120 字，說明本章重點。"
  prevChapter={{ title: "上一章標題", slug: "chNN-prev" }}
  nextChapter={{ title: "下一章標題", slug: "chNN-next" }}
>
  <p class="clabel">第 N 章 · Part X：Part 標題</p>
  <h1>章節主標題</h1>
  <p class="cdesc">一句話說明本章核心概念。</p>

  <h2>第一個段落標題</h2>
  <p>內容...</p>

</DocsLayout>
```

### Step 3：Build 驗證

```bash
npm run build
```
必須 **0 errors、0 warnings** 才算完成。

---

## 二、CodeBlock 元件規範

### 基本用法

```astro
<CodeBlock lang="bash" label="bash" code={`npm install`} />
<CodeBlock lang="typescript" label="src/index.ts" code={`const x = 1;`} />
<CodeBlock lang="json" code={`{"key": "value"}`} />
```

### ⚠️ 強制規則：複雜字串必須放 frontmatter

Astro template 區塊（`---` 以外）會將以下字元視為語法：
- `` ` `` → 啟動 template literal，導致 build 錯誤
- `{` / `}` → 啟動 JSX expression，導致 build 錯誤

**凡字串內含 `${}`, `` ` ``, `{`, `}` 的，一律移到 frontmatter `const`：**

```astro
---
// ✅ 正確：複雜字串放 frontmatter
const configCode = `
server.setRequestHandler(CallToolRequestSchema, async (req) => {
  const { name, arguments: args } = req.params;
  return { content: [{ type: "text", text: \`result\` }] };
});`;
---
<CodeBlock lang="typescript" code={configCode} />
```

```astro
<!-- ❌ 錯誤：template 裡直接寫含 {} 的字串 -->
<CodeBlock code={`const x = { key: "value" };`} />
```

### lang 可用值

`bash`, `powershell`, `typescript`, `javascript`, `python`, `json`, `text`, `yaml`, `sql`

---

## 三、HTML 元素規範

### 標題結構

```html
<p class="clabel">第 N 章 · Part X：Part 名稱</p>   <!-- 橘色小標，必填 -->
<h1>章節主標題</h1>                                   <!-- 每頁只有一個 h1 -->
<p class="cdesc">一句話副標題</p>                     <!-- 灰色，下方有分隔線，必填 -->

<h2>段落標題</h2>    <!-- 上方有 border-top，下方有 2px accent 線 -->
<h3>子段落標題</h3>  <!-- 無分隔線 -->
```

### Callout 區塊（三種）

```html
<!-- 💡 提示（綠） -->
<div class="call tip">
  <span class="ci">💡</span>
  <div>
    <strong>標題</strong>
    <p>說明文字。</p>
  </div>
</div>

<!-- ℹ️ 資訊（藍） -->
<div class="call info">
  <span class="ci">ℹ️</span>
  <div>
    <strong>標題</strong>
    <p>說明文字。</p>
  </div>
</div>

<!-- ⚠️ 警告（橘） -->
<div class="call warn">
  <span class="ci">⚠️</span>
  <div>
    <strong>標題</strong>
    <p>說明文字。</p>
  </div>
</div>
```

### 表格規範

```html
<table>
  <thead>
    <tr>
      <th>欄位A</th>
      <th>欄位B</th>
      <th>欄位C</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>值1</code></td>
      <td>說明1</td>
      <td>備註1</td>
    </tr>
  </tbody>
</table>
```

- 必須有 `<thead>` + `<tbody>`，否則樣式不套用
- 程式碼類的值用 `<code>` 包起來
- 表頭自動顯示 2px accent 橘紅色底線
- hover 自動有背景色

### 鍵盤按鍵

```html
<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>X</kbd>
```

### 圖片

```html
<figure class="ss">
  <img src="/img/filename.png" alt="描述文字" />
  <figcaption>圖片說明</figcaption>
</figure>
```
圖片檔放 `public/img/`，路徑從 `/img/` 起。

---

## 四、CSS 架構說明

### 不要在章節頁面加 `<style>` 區塊

除非有頁面專屬的特殊 class（如 ch08 的 `.mem-layer`、ch12 的 `.skill-grid`），否則不加 `<style>`。
全域樣式統一在 `DocsLayout.astro` 管理。

### 為什麼有兩個 style 區塊？

`DocsLayout.astro` 有兩個 `<style>` 區塊：

| 區塊 | 類型 | 負責 |
|------|------|------|
| `<style>` | Scoped | Header、Sidebar、Layout、Mobile |
| `<style is:global>` | Global | Typography、Table、Callout |

**原因**：Astro scoped CSS 對 slot 內容無效（slot 元素沒有 layout 的 CID）。
內容樣式必須用 `is:global` 才能套用到各章節頁面的 h2、table、p 等元素。

---

## 五、CSS 變數參考

| 變數 | 用途 |
|------|------|
| `--accent` | 主色（橘紅）—— 重點、連結、標題線 |
| `--bg` / `--bg2` / `--bg3` | 背景由淺到深 |
| `--text` / `--text2` / `--muted` | 文字由深到淺 |
| `--border` | 分隔線 |
| `--serif` | 標題字型（Georgia） |
| `--mono` | 等寬字型（Fira Code） |
| `--code-bg` | 程式碼區塊背景（`#1e1e1e`） |

---

## 六、章節規模建議

| 章節類型 | 建議 h2 數量 | 建議 CodeBlock 數量 | 建議 Callout 數量 |
|---------|------------|-------------------|-----------------|
| 概念介紹 | 3~5 | 1~3 | 1~2 |
| 實作教學 | 4~6 | 3~6 | 1~3 |
| 參考/索引 | 5~8 | 2~4 | 0~1 |

每章節大約 600~1200 字的正文內容（不含程式碼）。

---

## 七、已知陷阱（禁止重犯）

### ❌ Template 字串含特殊字元

```astro
<!-- 錯誤：build 會噴 "Expected }" 錯誤 -->
<CodeBlock code={`const x = { key: ${value} };`} />

<!-- 正確：移到 frontmatter -->
---
const code = `const x = { key: ${value} };`;
---
<CodeBlock code={code} />
```

### ❌ CSS Media Query 缺少空格

```css
/* 錯誤：build warning */
@media(min-width:769px) and(max-width:1024px) { }

/* 正確：and 前後必須有空格 */
@media (min-width:769px) and (max-width:1024px) { }
```

### ❌ 表格缺少 thead/tbody

```html
<!-- 錯誤：樣式不套用 -->
<table>
  <tr><th>欄位</th></tr>
  <tr><td>值</td></tr>
</table>

<!-- 正確 -->
<table>
  <thead><tr><th>欄位</th></tr></thead>
  <tbody><tr><td>值</td></tr></tbody>
</table>
```

### ❌ 在 navigation.js 加了新章節但沒建立對應 .astro 檔案

Build 不會報錯，但連結會 404。每次改 navigation.js 就要確認頁面檔案存在。

---

## 八、現有章節清單

| Slug | 標題 | Part |
|------|------|------|
| `ch01-what-is-cc` | 什麼是 Claude Code？ | 1 |
| `ch02-install` | 安裝與環境設定 | 1 |
| `ch03-first-chat` | 第一次對話 | 1 |
| `ch04-models` | 三大模型比較 | 2 |
| `ch05-model-switch` | 如何切換模型 | 2 |
| `ch06-claude-md` | CLAUDE.md | 3 |
| `ch07-rules` | Rules | 3 |
| `ch08-memory` | Memory.md | 3 |
| `ch09-hooks-intro` | Hooks 是什麼？ | 4 |
| `ch10-hooks-diary` | 實戰：自動寫工作日誌 | 4 |
| `ch11-hooks-security` | Hooks 安全性應用 | 4 |
| `ch12-skills-intro` | 官方六大類 Skills | 5 |
| `ch13-skills-builtin` | 使用內建 Skills | 5 |
| `ch14-skills-custom` | 自己寫 Skill | 5 |
| `ch15-mcp-intro` | MCP 是什麼？ | 6 |
| `ch16-mcp-install` | 安裝 MCP Server | 6 |
| `ch17-mcp-custom` | 自製 MCP Server | 6 |
| `ch18-multi-agent` | 多 Agent 協作 | 7 |
| `ch19-permissions` | Permissions 模式 | 7 |
| `ch20-faq` | 常見問題 & Debug | 7 |
| `about` | 關於作者 | 8（附錄） |
