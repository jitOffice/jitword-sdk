# JitWord Web SDK 使用文档

![](./demo.png)

# JitWord Web SDK 使用文档

JitWord SDK 提供了一个简单的方式将强大的协同文档编辑器集成到您的 Web 应用中。支持 Vue、React、Angular 以及原生 HTML/JS 项目。

⚠️ 注：本项目**仅供个人或者非商业用途**使用，受 **GPL3.0** 协议控制，如需商用，请[**联系授权**](https://jitword.com/on-premise.html)。

## 1. 目标

让客户更简单的来接入我们协同文档编辑器，并开放全局可配置，可操控的API，供客户使用。

## 2. SDK 功能设计

### 2.1 实现要求

- **本地 SDK 引入**: 支持通过本地文件引入到项目（后端接口和协同部分需连接远程服务）。
- **多技术栈支持**: 支持 Vue, React, Angular, 原生 HTML 等。
- **API 暴露**: 提供丰富的 API 让开发人员控制编辑器行为。
- **快速上手**: 简单易用，5分钟即可完成接入。
- **完善文档**: 提供详细的 API 和 SDK 使用文档。

## 3. 快速开始 (Quick Start)

### 3.1 引入 SDK

您可以通过本地文件引入 SDK 及其依赖。

```html
<!-- 1. 引入样式 -->
<link rel="stylesheet" href="path/to/arco.css">
<link rel="stylesheet" href="path/to/px-editor.css">

<!-- 2. 引入依赖库 -->
<script src="path/to/vue.global.prod.js"></script>
<script src="path/to/arco-vue.min.js"></script>
<script src="path/to/arco-vue-icon.min.js"></script>
<script src="path/to/echarts.min.js"></script>
<script src="path/to/mind-elixir.js"></script>

<!-- 3. 引入 JitWord SDK -->
<script src="path/to/px-editor.standalone.js"></script>
```

### 3.2 初始化编辑器

在 HTML 中准备一个容器：

```html
<div id="col-editor" style="height: 100vh;"></div>
```

使用 `Jitword` 类初始化：

```javascript
// 确保脚本加载完成后执行
window.onload = function() {
  const { Jitword } = window.PxEditor;

  const editor = new Jitword({
    hold: "col-editor", // 容器 ID
    appTitle: 'JitWord 协作文档',
    logo: 'https://jitword.com/logo.png',
    enableAI: true,
    // ...其他配置
  });
};
```

### 3.3 本地测试与部署

#### ⚠️ 重要提示

**不要直接使用 `file://` 协议打开 HTML 文件！** 这会导致以下问题：

1. **CORS 错误**：浏览器会阻止从 `file://` 向远程 API 发送请求
   ```
   Access to fetch at 'https://api.example.com' from origin 'null' 
   has been blocked by CORS policy
   ```

2. **401 认证失败**：无法正确存储和发送认证 token
   ```
   GET https://api.example.com/documents net::ERR_FAILED 401 (Unauthorized)
   ```

#### ✅ 正确的本地测试方法

**方法 1: 使用 Python 自带的 HTTP 服务器（推荐）**

```bash
# 进入 HTML 文件所在目录
cd /path/to/your/project

# 启动 HTTP 服务器
python3 -m http.server 8080

# 或使用 Python 2
python -m SimpleHTTPServer 8080
```

然后在浏览器访问：`http://localhost:8080/standalone.html`

**方法 2: 使用 Node.js http-server**

```bash
# 安装 http-server (全局)
npm install -g http-server

# 或使用 npx (无需安装)
npx http-server -p 8080

# 启动服务器
http-server -p 8080
```

然后访问：`http://localhost:8080/standalone.html`

**方法 3: 使用 VS Code Live Server 扩展**

1. 安装 VS Code 扩展：**Live Server**
2. 右键点击 HTML 文件
3. 选择 **"Open with Live Server"**
4. 自动在浏览器打开，支持热重载

**方法 4: 使用 PHP 自带服务器**

```bash
php -S localhost:8080
```

#### 📋 测试检查清单

启动本地服务器后，检查以下内容确保正常运行：

- ✅ 浏览器地址栏显示 `http://localhost:8080/...`（不是 `file://`）
- ✅ 浏览器控制台没有 CORS 错误
- ✅ 编辑器正常加载和显示
- ✅ 可以正常编辑文档内容
- ✅ 如果配置了协同功能，可以看到 WebSocket 连接成功

#### 🚀 生产环境部署

将 SDK 文件部署到您的 Web 服务器（如 Nginx、Apache）即可：

```nginx
# Nginx 配置示例
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        root /path/to/your/sdk/files;
        index standalone.html;
    }
    
    # 配置 CORS（如果需要）
    add_header Access-Control-Allow-Origin *;
}
```

#### 🔧 常见问题排查

**问题 1: 提示 "PxEditor SDK 加载失败"**
- 检查 `px-editor.standalone.js` 文件路径是否正确
- 检查所有依赖文件（vue.global.prod.js、arco-vue.min.js 等）是否都已加载
- 打开浏览器开发者工具 → Network 标签，查看哪个文件加载失败

**问题 2: 编辑器功能异常**
- 清除浏览器缓存（Cmd+Shift+R 或 Ctrl+Shift+R 硬刷新）
- 检查 SDK 版本是否最新（查看文件的 `?v=timestamp` 参数）
- 查看控制台是否有 JavaScript 错误

**问题 3: 协同功能无法连接**
- 检查 `wsServer` 配置是否正确
- 检查 WebSocket 服务器是否运行
- 检查网络防火墙是否阻止 WebSocket 连接

## 4. SDK API 设计

### 4.1 配置项 (Configuration)

初始化 `new Jitword(config)` 时支持以下配置参数：

#### 4.1.1 基础配置

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :---: | :--- | :--- |
| `hold` | `string` \| `HTMLElement` | **✓** | - | 编辑器挂载的 DOM 节点 ID 或元素对象 |
| `appTitle` | `string` | | `'JitWord 协同文档'` | 应用标题，显示在编辑器头部 |
| `appTitleShort` | `string` | | `'JitWord'` | 应用简短标题，移动端显示 |
| `documentTitle` | `string` | | `'未命名文档'` | 当前文档标题 |
| `logo` / `logoSrc` | `string` | | - | Logo 图片 URL |
| `logoHref` | `string` | | `'/'` | 点击 Logo 跳转的 URL |
| `locale` | `'zh'` \| `'en'` | | `'zh'` | 界面语言 |
| `theme` | `'light'` \| `'dark'` | | `'light'` | 主题模式 |
| `placeholder` | `string` | | `'请输入文档内容...'` | 编辑器空白时的占位文本 |

#### 4.1.2 编辑器功能配置

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :---: | :--- | :--- |
| `editable` | `boolean` | | `true` | 是否可编辑 |
| `readMode` | `boolean` | | `false` | 只读模式（优先级高于 editable） |
| `showScrollNav` | `boolean` | | `true` | 是否显示右下角滚动导航 |
| `showFloatToolBar` / `showRightToolbar` | `boolean` | | `true` | 是否显示右侧浮动工具栏 |
| `hideToc` | `boolean` | | `false` | 是否隐藏目录（TOC） |
| `hideFooter` | `boolean` | | `false` | 是否隐藏底部状态栏 |
| `hideBubbleMenu` | `boolean` | | `false` | 是否隐藏气泡菜单 |
| `showPageSettings` | `boolean` | | `true` | 是否显示页面设置按钮 |

#### 4.1.3 协同编辑配置

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :---: | :--- | :--- |
| `enableCollaboration` | `boolean` | | `true` | 是否启用协同编辑 |
| `enableCollaborationCursor` | `boolean` | | `true` | 是否显示协同光标 |
| `wsServer` | `string` | 协同时必填 | - | WebSocket 服务器地址 (例: `wss://example.com/ws`) |
| `currentDocumentId` | `string` | 协同时推荐 | - | 当前文档 ID，用于生成稳定的协同房间名 |
| `roomName` | `string` | | 自动生成 | 协同房间名（不设置时自动使用 `doc-{currentDocumentId}`） |
| `resetCollabState` | `boolean` | | **`false`** | 是否每次刷新生成新房间（false=稳定房间，true=临时房间） |
| `user` | `object` | | - | 用户信息 `{ name: string, color: string }` |

> **⚠️ 重要**: `resetCollabState` 默认为 `false` 以确保协同房间稳定。设为 `true` 会导致每次刷新生成新房间，破坏持久化协作。

#### 4.1.4 数据与回调

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :---: | :--- | :--- |
| `value` | `object` \| `string` | | - | 文档初始内容（优先级高于 defaultValue） |
| `defaultValue` | `object` \| `string` | | - | 默认内容（仅当文档为空时使用） |
| `enableAutoSave` | `boolean` | | `false` | 是否启用自动保存 |
| `enableAutoLoad` | `boolean` | | `false` | 是否启用自动加载 |
| `onChange` | `(content) => void` | | - | 内容变化回调 |
| `onSave` | `({ content, currentDocumentId }) => void` | | - | 保存回调 |
| `onEditorReady` | `(editor) => void` | | - | 编辑器就绪回调 |
| `uploadAPI` / `uploadFn` | `(file: File) => Promise<string>` | | - | 文件上传函数，返回文件 URL |

#### 4.1.5 API 配置

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :---: | :--- | :--- |
| `baseApiUrl` | `string` | | - | 后端 API 基础路径 |

#### 4.1.6 UI 事件回调

| 参数名 | 类型 | 说明 |
| :--- | :--- | :--- |
| `onShare` | `() => void` | 点击分享按钮回调 |
| `onAi` | `() => void` | 点击 AI 按钮回调 |
| `onAiSettings` | `() => void` | 点击 AI 设置回调 |
| `onVersion` | `() => void` | 点击版本历史回调 |
| `onDocumentList` | `() => void` | 点击文档列表回调 |
| `onCreateDocument` | `(data) => void` | 创建文档回调 |
| `onSelectDocument` | `(docId) => void` | 选择文档回调 |
| `onExitReadMode` | `() => void` | 退出只读模式回调 |

### 4.2 数据格式说明

`value` 和 `defaultValue` 均支持 JSON 或 HTML 格式，两者的区别在于：
- **`value`**: 用于**受控模式**或**回显已有数据**。如果设置了 `value`，编辑器会优先加载它。
- **`defaultValue`**: 用于**非受控模式**或**新建文档**。只有当编辑器内容为空（且 `value` 未设置或为空）时，才会自动填充此内容。

**建议**：

## 4.3 实例级内容 API

`new Jitword(config)` 返回的实例除了已有的 `setContent`、`getJSON`、`getHTML` 等方法外，现已支持第一阶段的元素级 API。

### 4.3.1 适用范围

当前第一阶段支持的顶层元素类型：

- `paragraph`
- `heading`
- `image`
- `attachment`
- `divider`
- `bulletList`
- `orderedList`
- `columns`
- `table`
- `chart`
- `mindmap`
- `mermaid`
- `flowchart`

说明：

- SDK 中的“元素”指的是文档顶层 block 元素
- `index` 表示顶层元素序号，从 `0` 开始
- 若元素未传 `id`，SDK 会自动生成唯一 ID
- 在只读模式下，写操作会被拒绝

### 4.3.2 插入元素

```ts
insert(index?: number, element: SDKElement): SDKElement
```

示例：

```js
const element = editor.insert(1, {
  type: 'paragraph',
  data: {
    text: '这是一段通过 insert() 插入的文本'
  }
})

console.log(element.id)
```

插入图片：

```js
editor.insert({
  type: 'image',
  data: {
    url: 'https://picsum.photos/640/360',
    alt: '示例图片'
  }
})
```

### 4.3.3 获取指定元素

```ts
getElementById(elementId: string): SDKElement | null
```

示例：

```js
const element = editor.getElementById('paragraph-123')

if (element) {
  console.log('元素类型:', element.type)
  console.log('元素数据:', element.data)
}
```

### 4.3.4 删除指定元素

```ts
deleteElementById(elementId: string): boolean
```

示例：

```js
const ok = editor.deleteElementById('paragraph-123')
console.log('删除结果:', ok)
```

### 4.3.5 更新指定元素

```ts
updateElement(elementId: string, patch: Partial<SDKElement>): SDKElement | null
```

说明：

- `updateElement` 采用 patch 语义
- 只更新传入字段，不会清空未传字段

示例：

```js
const next = editor.updateElement('paragraph-123', {
  data: {
    text: '这段文字已被更新'
  }
})
```

更新图片：

```js
editor.updateElement('image-123', {
  data: {
    url: 'https://picsum.photos/800/400'
  }
})
```

### 4.3.6 SDKElement 数据结构

```ts
type SDKElement = {
  id?: string
  type: 'paragraph' | 'heading' | 'image' | 'attachment' | 'divider' | 'bulletList' | 'orderedList'
  data?: Record<string, any>
  style?: Record<string, any>
  meta?: Record<string, any>
}
```

示例：

```js
{
  id: 'heading-001',
  type: 'heading',
  data: {
    level: 2,
    text: '企业级 SDK 设计'
  },
  style: {}
}
```

### 4.3.7 常见元素示例

段落：

```js
editor.insert({
  type: 'paragraph',
  data: {
    text: '这是一个段落'
  }
})
```

标题：

```js
editor.insert({
  type: 'heading',
  data: {
    level: 2,
    text: '这是一个二级标题'
  }
})
```

无序列表：

```js
editor.insert({
  type: 'bulletList',
  data: {
    items: [
      '第一项',
      '第二项',
      '第三项'
    ]
  }
})
```

附件：

```js
editor.insert({
  type: 'attachment',
  data: {
    url: 'https://example.com/demo.pdf',
    name: '演示附件.pdf',
    size: '245 KB',
    fileType: 'application/pdf'
  }
})
```

分割线：

```js
editor.insert({
  type: 'divider'
})
```

### 4.3.8 导入文件

```ts
importFile(type: 'docx' | 'md' | 'html' | 'json', file: File, options?: {
  mode?: 'replace' | 'append' | 'insert'
  index?: number
  docxEngine?: 'docx4js' | 'mammoth'
}): Promise<SDKImportResult>
```

说明：

- 默认 `mode = 'replace'`
- `append` 会把导入内容追加到文末
- `insert` 会把导入内容插入到指定顶层位置

示例：

```js
const file = input.files[0]

await editor.importFile('md', file, {
  mode: 'replace'
})
```

追加导入：

```js
await editor.importFile('json', file, {
  mode: 'append'
})
```

插入导入：

```js
await editor.importFile('html', file, {
  mode: 'insert',
  index: 2
})
```

### 4.3.9 导出文件

```ts
exportFile(type: 'docx' | 'pdf' | 'md' | 'json' | 'html', options?: {
  autoDownload?: boolean
  filename?: string
}): Promise<Blob | boolean>
```

说明：

- `autoDownload = true` 时自动下载，返回 `true`
- `autoDownload = false` 时返回 `Blob`

示例：

```js
await editor.exportFile('md', {
  filename: 'demo.md'
})
```

返回 Blob：

```js
const blob = await editor.exportFile('pdf', {
  autoDownload: false,
  filename: 'demo.pdf'
})

console.log(blob.size)
```

兼容别名：

```js
await editor.import('docx', file)
await editor.export('html', { autoDownload: true })
```

## 4.4 实例 API 使用示例

```js
const editor = new Jitword({
  hold: 'col-editor',
  enableCollaboration: true,
  currentDocumentId: 'demo-doc-001',
  wsServer: 'wss://your-domain.com/ws'
})

const inserted = editor.insert({
  type: 'paragraph',
  data: { text: 'SDK 已成功接入' }
})

const element = editor.getElementById(inserted.id)

editor.updateElement(inserted.id, {
  data: { text: 'SDK 已成功接入，并完成更新' }
})

editor.deleteElementById(inserted.id)
```

## 4.5 返回值与错误约定

- `getElementById`
  - 找不到时返回 `null`
- `deleteElementById`
  - 删除成功返回 `true`
  - 找不到时返回 `false`
- `insert` / `updateElement`
  - 失败时抛出异常

常见错误场景：

- 编辑器尚未 ready
- 当前实例为只读模式
- 传入了不支持的元素类型
- 元素结构与目标节点 schema 不匹配
- 编辑已有文档时：使用 `value`。
- 新建文档时：使用 `defaultValue` 设置模板或欢迎语。

#### 支持的数据格式：

**1. JSON 格式 (推荐)**

使用 ProseMirror / Tiptap 标准 JSON 结构：
```json
{
  "type": "doc",
  "content": [
    {
      "type": "heading",
      "attrs": { "level": 1 },
      "content": [{ "type": "text", "text": "标题内容" }]
    },
    {
      "type": "paragraph",
      "content": [{ "type": "text", "text": "正文内容..." }]
    }
  ]
}
```

**2. HTML 格式**

直接传入 HTML 字符串，编辑器会自动解析：
```html
<h1>标题内容</h1>
<p>正文内容...</p>
```

## 4.6 其他 Jitword 实例方法

`Jitword` 实例提供以下方法供外部调用：

| 方法名 | 参数 | 返回值 | 说明 |
| :--- | :--- | :--- | :--- |
| `getJData()` | - | `object` | 获取文档 JSON 格式数据 |
| `getHtml()` | - | `string` | 获取文档 HTML 格式数据 |
| `setData(data)` | `object \| string` | `void` | 设置文档内容（JSON 或 HTML） |
| `setEditable(editable)` | `boolean` | `void` | 设置编辑器是否可编辑 |
| `getEditor()` | - | `Editor` | 获取底层 TipTap Editor 实例 |
| `setCurrentDocumentId(docId)` | `string` | `void` | 切换当前文档 ID（触发协同房间切换） |
| `destroy()` | - | `void` | 销毁编辑器实例 |

**使用示例：**

```javascript
const editor = new Jitword({ hold: 'editor-container' });

// 获取内容
const json = editor.getJData();
const html = editor.getHtml();

// 设置内容
editor.setData({ type: 'doc', content: [...] });
editor.setData('<p>Hello World</p>');

// 切换可编辑状态
editor.setEditable(false); // 只读
editor.setEditable(true);  // 可编辑

// 切换文档
editor.setCurrentDocumentId('new-doc-id');

// 销毁实例
editor.destroy();
```

## 4.7 JitWordSDK REST API

`JitWordSDK` 提供完整的后端 REST API 调用方法。

### 4.7.1 初始化 SDK

```javascript
const { JitWordSDK } = window.PxEditor;

JitWordSDK.init({
  baseUrl: 'https://your-api-server.com/api/v1',
  tokenKey: 'jwt_token',  // localStorage key for auth token
  onError: (err) => console.error('SDK Error:', err)
});
```

### 4.7.2 认证 API

| 方法 | 参数 | 返回值 | 说明 |
| :--- | :--- | :--- | :--- |
| `JitWordSDK.auth.register(username, password)` | `string, string` | `Promise<any>` | 用户注册 |
| `JitWordSDK.auth.login(username, password)` | `string, string` | `Promise<any>` | 用户登录（自动存储 token） |
| `JitWordSDK.auth.setToken(token)` | `string` | `void` | 手动设置认证 token |
| `JitWordSDK.auth.getToken()` | - | `string` | 获取当前 token |

### 4.7.3 文档 API

| 方法 | 参数 | 说明 |
| :--- | :--- | :--- |
| `documents.list(options)` | `{ filter?, scope? }` | 获取文档列表 |
| `documents.get(id)` | `string` | 获取文档详情 |
| `documents.create(name)` | `string` | 创建新文档 |
| `documents.delete(id)` | `string` | 删除文档（移至回收站） |
| `documents.restore(id)` | `string` | 从回收站恢复文档 |
| `documents.permanentDelete(id)` | `string` | 永久删除文档 |
| `documents.rename(id, name)` | `string, string` | 重命名文档 |
| `documents.updatePageSettings(id, settings)` | `string, object` | 更新页面设置 |
| `documents.updatePermission(id, permission)` | `string, 'read'\|'edit'\|'private'` | 更新文档权限 |
| `documents.stats(filter)` | `'active'\|'deleted'\|'all'` | 获取文档统计信息 |

**使用示例：**

```javascript
// 获取文档列表
const docs = await JitWordSDK.documents.list({ filter: 'active', scope: 'mine' });

// 创建文档
const newDoc = await JitWordSDK.documents.create('我的新文档');

// 更新权限
await JitWordSDK.documents.updatePermission(docId, 'read');
```

### 4.7.4 评论 API

| 方法 | 参数 | 说明 |
| :--- | :--- | :--- |
| `comments.list(documentId)` | `string` | 获取文档的所有评论 |
| `comments.create(data)` | `{ id, documentId, content, targetText? }` | 创建新评论 |
| `comments.update(id, data)` | `string, { documentId, content }` | 更新评论 |
| `comments.delete(id, documentId)` | `string, string` | 删除评论 |

**使用示例：**

```javascript
// 获取评论
const comments = await JitWordSDK.comments.list(documentId);

// 创建评论
await JitWordSDK.comments.create({
  id: 'comment-123',
  documentId: 'doc-456',
  content: '这里需要修改',
  targetText: '选中的文本'
});
```

### 4.7.5 版本 API

| 方法 | 参数 | 说明 |
| :--- | :--- | :--- |
| `versions.list(docId, page?, limit?)` | `string, number?, number?` | 获取版本列表 |
| `versions.create(docId, data)` | `string, { content, title?, description? }` | 创建版本快照 |
| `versions.get(docId, versionId)` | `string, string` | 获取指定版本 |
| `versions.delete(docId, versionId)` | `string, string` | 删除版本 |
| `versions.update(docId, versionId, data)` | `string, string, { title?, description? }` | 更新版本信息 |
| `versions.compare(docId, v1, v2)` | `string, string, string` | 对比两个版本 |

### 4.7.6 AI API

| 方法 | 参数 | 说明 |
| :--- | :--- | :--- |
| `ai.trialStats(uid?, role?)` | `string?, string?` | 获取 AI 试用统计 |
| `ai.streamTrial(args, onDelta)` | `object, function` | AI 流式生成（试用版） |

### 4.7.7 文件上传 API

| 方法 | 参数 | 说明 |
| :--- | :--- | :--- |
| `files.uploadApi(file)` | `File \| FormData` | 上传文件 |
| `files.uploadDoc(data)` | `FormData \| object` | 上传 Word 文档解析 |
| `files.uploadPdf(data)` | `FormData \| object` | 上传 PDF 文档解析 |

## 5. 后端 API 路由定义

如果你需要自己实现后端服务，以下是 SDK 使用的所有 API 端点：

### 5.1 认证路由

| 方法 | 路径 | 说明 | 请求体 | 响应 |
| :--- | :--- | :--- | :--- | :--- |
| POST | `/user/register` | 用户注册 | `{ username, password }` | `{ code, data }` |
| POST | `/user/login` | 用户登录 | `{ username, password }` | `{ code, data: { token } }` |

### 5.2 文档路由

| 方法 | 路径 | 说明 | Query/Body | 响应 |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/documents` | 获取文档列表 | `?filter=active&scope=mine` | `{ code, data: [...] }` |
| GET | `/documents/stats` | 获取文档统计 | `?filter=active` | `{ code, data }` |
| GET | `/documents/:id` | 获取文档详情 | - | `{ code, data }` |
| POST | `/documents` | 创建文档 | `{ name }` | `{ code, data }` |
| PUT | `/documents/:id` | 重命名文档 | `{ name }` | `{ code, data }` |
| DELETE | `/documents/:id` | 删除文档 | - | `{ code }` |
| PUT | `/documents/:id/restore` | 恢复文档 | - | `{ code }` |
| DELETE | `/documents/:id/permanent` | 永久删除 | - | `{ code }` |
| PUT | `/documents/:id/page-settings` | 更新页面设置 | `{ pageSettings }` | `{ code }` |
| PUT | `/documents/:id/permission` | 更新权限 | `{ publicPermission }` | `{ code }` |

### 5.3 评论路由

| 方法 | 路径 | 说明 | Query/Body | 响应 |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/comments` | 获取评论列表 | `?documentId=xxx` | `{ code, data: [...] }` |
| POST | `/comments` | 创建评论 | `{ id, documentId, content, targetText? }` | `{ code, data }` |
| PUT | `/comments/:id` | 更新评论 | `{ documentId, content }` | `{ code, data }` |
| DELETE | `/comments/:id` | 删除评论 | `?documentId=xxx` | `{ code }` |

### 5.4 版本路由

| 方法 | 路径 | 说明 | Query/Body | 响应 |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/documents/:docId/versions` | 获取版本列表 | `?page=1&limit=20` | `{ code, data: [...] }` |
| POST | `/documents/:docId/versions` | 创建版本 | `{ content, title?, description? }` | `{ code, data }` |
| GET | `/documents/:docId/versions/:versionId` | 获取版本详情 | - | `{ code, data }` |
| PUT | `/documents/:docId/versions/:versionId` | 更新版本 | `{ title?, description? }` | `{ code, data }` |
| DELETE | `/documents/:docId/versions/:versionId` | 删除版本 | - | `{ code }` |
| GET | `/documents/:docId/versions/:v1/compare/:v2` | 对比版本 | - | `{ code, data }` |

### 5.5 AI 路由

| 方法 | 路径 | 说明 | Headers/Body | 响应 |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/ai/trial/stats` | AI 试用统计 | `x-user-id?, x-user-role?` | `{ code, data }` |
| POST | `/ai/trial/stream` | AI 流式生成 | `{ model, messages, temperature? }` | Stream |

### 5.6 文件上传路由

| 方法 | 路径 | 说明 | Body | 响应 |
| :--- | :--- | :--- | :--- | :--- |
| POST | `/upload/free` | 通用文件上传 | `FormData: { file }` | `{ code, data: { url } }` |
| POST | `/parse/doc2html2` | Word 文档解析 | `FormData: { file }` | `{ code, data }` |
| GET | `/parse/doc2html2/:docId/:page` | 获取解析的 Word 页面 | - | `{ code, data }` |
| POST | `/parse/pdf2html` | PDF 文档解析 | `FormData: { file }` | `{ code, data }` |
| GET | `/parse/pdf2html/:page` | 获取解析的 PDF 页面 | `?fid=xxx` | `{ code, data }` |

### 5.7 响应格式规范

所有 API 响应遵循统一格式：

```json
{
  "code": 200,
  "message": "Success",
  "data": { /* 实际数据 */ }
}
```

**错误响应：**

```json
{
  "code": 400,
  "message": "Error message",
  "data": null
}
```

**常见状态码：**
- `200`: 成功
- `400`: 请求参数错误
- `401`: 未授权（需要登录）
- `403`: 权限不足
- `404`: 资源不存在
- `500`: 服务器内部错误

## 6. 完整示例 (standalone.html)

```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>JitWord - 独立构建示例</title>
  
  <!-- 1. SDK 样式 -->
  <link rel="stylesheet" href="cdn/arco.css">
  <link rel="stylesheet" href="cdn/px-editor.css">

  <!-- 2. 依赖库 -->
  <script src="cdn/vue.global.prod.js"></script>
  <script src="cdn/arco-vue.min.js"></script>
  <script src="cdn/arco-vue-icon.min.js"></script>
  <script src="cdn/echarts.min.js"></script>
  <script src="cdn/mind-elixir.js"></script>
  
  <!-- 3. JitWord SDK -->
  <script src="cdn/px-editor.standalone.js"></script>

  <style>
    body { margin: 0; padding: 0; }
    #app { width: 100%; height: 100vh; }
  </style>
</head>
<body>
  <div id="app">
    <div id="col-editor" style="height: 100%"></div>
  </div>

  <script>
    window.onload = function() {
      // 检查 SDK 是否加载成功
      if (!window.PxEditor) {
          console.error('PxEditor SDK 加载失败');
          return;
      }

      const { Jitword, Message } = window.PxEditor;

      // 初始化编辑器
      const editor = new Jitword({
        hold: "col-editor",
        
        // 基础配置
        appTitle: 'JitWord 协作文档',
        logo: 'https://jitword.com/logo.png',
        logoHref: 'https://jitword.com/',
        locale: 'zh',
        placeholder: '在此输入内容...',
        
        // 功能开关
        showScrollNav: true,
        showFloatToolBar: true,
        enableAI: true,
        enableAISettings: true,
        editable: true,
        
        // 协同配置 (可选)
        enableCollaboration: true,
        wsServer: 'wss://your-collab-server.com/ws',
        
        // 数据与回调
        defaultValue: { type: 'doc', content: [] },
        
        onSave: ({ content }) => {
          console.log('文档保存:', content);
          Message.success('保存成功');
        },
        
        onChange: (content) => {
          console.log('内容变更');
        },
        
        uploadAPI: async (file) => {
          console.log('上传文件:', file.name);
          // 模拟上传返回 URL
          return URL.createObjectURL(file);
        }
      });
      
      // 暴露 editor 实例供测试
      window.editor = editor;
    };
  </script>
</body>
</html>
```
