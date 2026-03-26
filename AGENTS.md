# 公众号 Markdown 编辑器

一个专为微信公众号设计的 Markdown 编辑器，支持实时预览和一键复制富文本格式到公众号后台。

## 项目概述

这是一个纯前端的 Markdown 编辑器，可以将标准 Markdown 文本转换为适合微信公众号的富文本格式。当前保留 13 个主题，每个主题都针对公众号阅读和复制兼容性做了优化。

## 核心功能

### 1. Markdown 渲染
- 使用 `markdown-it` 进行 Markdown 解析
- 支持完整的 Markdown 语法（标题、列表、引用、代码块、表格等）
- 实时预览渲染结果

### 2. 样式系统
提供 13 种预设样式主题，当前按风格分组如下：
- **通用基础**：默认公众号风格、技术风格、深度阅读
- **国际媒体**：纽约时报、Nikkei 日経、Atlantic 特稿、Monocle 简报
- **中文内容**：晚点风格、少数派专栏、人物特稿、商业观察
- **品牌表达**：Claude、轻品牌手记

每个主题都包含完整的元素定义（标题、段落、列表、引用、代码、表格、图片等）。当前基础排版约束也已经统一：
- `container` 采用更适合复制到公众号的满宽正文布局
- 正文字号统一为 `15px`
- 正文、列表、引用行高统一为 `1.75`
- `h1` 到 `h6` 的字号、粗细、行高、margin 已统一
- `hr`、代码块、表格、图片的节奏和间距已统一
- 图片无边框，单张图片最大高度 `400px`
- 表格文字左对齐

### 3. 图片处理系统（⭐ 核心功能）

#### 技术架构概览

本编辑器采用**自定义图片协议 + 压缩 + IndexedDB 存储**的创新方案，彻底解决了图片粘贴的所有痛点：

```
用户粘贴图片（任何来源）
    ↓
ImageCompressor 压缩（Canvas API）
    ├─ 最大尺寸：1920x1920px
    ├─ 压缩质量：85%
    ├─ GIF/SVG 不压缩（保持动画/矢量）
    └─ 智能对比：压缩后更大则用原图
    ↓
生成唯一 ID（img-timestamp-random）
    ↓
ImageStore 存储到 IndexedDB
    ├─ 数据库名：WechatEditorImages
    ├─ 存储：Blob + 元数据
    └─ 持久化：刷新不丢失
    ↓
编辑器插入短链接：![图片名](img://img-xxx)
    ↓
渲染预览时：
    ├─ 从 IndexedDB 读取 Blob
    ├─ 创建 Object URL
    ├─ 替换 img:// 为 blob:
    └─ 缓存 Object URL（避免重复读取）
    ↓
复制到公众号时：
    ├─ 检测 data-image-id 属性
    ├─ 从 IndexedDB 读取原始 Blob
    ├─ 转为 Base64
    └─ 替换 img 的 src
```

#### 核心组件

##### 1. **ImageStore 类**（app.js:9-213）

负责 IndexedDB 操作的核心类：

```javascript
class ImageStore {
  constructor() {
    this.dbName = 'WechatEditorImages';
    this.storeName = 'images';
    this.version = 1;
  }

  // 核心方法：
  async init()                        // 初始化 IndexedDB
  async saveImage(id, blob, metadata) // 保存图片
  async getImage(id)                  // 获取图片（返回 Object URL）
  async getImageBlob(id)              // 获取 Blob（用于复制时转 Base64）
  async deleteImage(id)               // 删除图片
  async getAllImages()                // 获取所有图片列表
  async getTotalSize()                // 计算总存储大小
}
```

**数据结构**：
```javascript
{
  id: 'img-1736966400000-abc123def',
  blob: Blob,                    // 压缩后的图片 Blob
  name: '图片名',
  originalSize: 2500000,         // 原始大小（字节）
  compressedSize: 487300,        // 压缩后大小
  compressionRatio: '81',        // 压缩率（%）
  mimeType: 'image/jpeg',
  createdAt: 1736966400000
}
```

##### 2. **ImageCompressor 类**（app.js:215-313）

负责图片压缩的核心类：

```javascript
class ImageCompressor {
  constructor(options = {}) {
    this.maxWidth = 1920;
    this.maxHeight = 1920;
    this.quality = 0.85;
    this.mimeType = 'image/jpeg';
  }

  async compress(file) {
    // 使用 Canvas API 压缩图片
    // 1. FileReader 读取文件为 DataURL
    // 2. 创建 Image 对象加载图片
    // 3. 计算缩放比例
    // 4. Canvas 绘制缩放后的图片
    // 5. toBlob 输出压缩后的 Blob
    // 6. 对比大小，如果压缩后更大则用原图
  }

  static formatSize(bytes) {
    // 格式化文件大小（B/KB/MB）
  }
}
```

**压缩策略**：
- PNG 保持 PNG 格式（避免透明度丢失）
- 其他格式转为 JPEG（更好的压缩率）
- GIF 和 SVG 不压缩（保持动画和矢量特性）
- 白色背景填充（处理透明 PNG）

##### 3. **图片粘贴处理**（app.js:1460-1530）

```javascript
async handleImageUpload(file, textarea) {
  // 1. 检查文件类型和大小（最大 10MB）
  // 2. 压缩图片（调用 ImageCompressor）
  // 3. 生成唯一 ID
  // 4. 存储到 IndexedDB
  // 5. 插入短链接到编辑器：![name](img://img-xxx)
  // 6. 显示压缩结果反馈
}
```

##### 4. **图片渲染处理**（app.js:634-689）

```javascript
async processImageProtocol(html) {
  // 1. 解析 HTML，找到所有 <img> 标签
  // 2. 检测 src 是否为 img:// 协议
  // 3. 提取图片 ID
  // 4. 从 IndexedDB 读取图片
  // 5. 创建 Object URL（或使用缓存）
  // 6. 替换 src 为 Object URL
  // 7. 添加 data-image-id 属性（供复制时使用）
  // 8. 错误处理：显示占位符
}
```

##### 5. **复制时转 Base64**（app.js:1186-1242）

```javascript
async convertImageToBase64(imgElement) {
  // 优先处理：检查 data-image-id 属性
  if (imageId && this.imageStore) {
    // 从 IndexedDB 读取 Blob
    const blob = await this.imageStore.getImageBlob(imageId);
    // 转为 Base64
    return fileReaderToBase64(blob);
  }

  // 后备方案：通过 URL fetch（兼容外链图片）
  const response = await fetch(src);
  const blob = await response.blob();
  return fileReaderToBase64(blob);
}
```

#### 连续图片网格布局（类似朋友圈）
- **触发条件**：2张或以上连续的图片
- **布局规则**：
  - 2张图片：并排两列（50% + 50%）
  - 3张图片：一行三列（33.33% × 3）
  - 4张图片：2×2 网格
  - 5张及以上：3列网格布局

#### 图片高度限制
- **单张图片**：最大高度 400px
- **网格布局中的图片**：最大高度 360px（单行）
- 所有图片保持宽高比，使用 `object-fit: contain`

#### 技术亮点

| 指标 | 传统方案（图床/Base64） | 本项目方案 |
|------|----------------------|----------|
| **编辑器体验** | 几千字符卡顿 | 20字符丝滑 |
| **成功率** | 80%（依赖图床） | 100% |
| **刷新后** | 图片丢失 | 完好保留 |
| **文件大小** | 原图大小 | 压缩 50%-80% |
| **网络依赖** | 需要 | 不需要 |
| **隐私性** | 上传到服务器 | 本地存储 |

### 4. 公众号兼容性处理

#### Grid 转 Table
- **问题**：公众号编辑器不支持 CSS Grid 布局
- **解决方案**：复制时自动将 Grid 布局转换为 Table 布局
- **实现位置**：`app.js` 中的 `convertGridToTable()` 方法
- **转换逻辑**：
  ```javascript
  预览阶段：CSS Grid（现代、美观）
           ↓
  复制到公众号：HTML Table（兼容、可靠）
  ```

#### 样式内联化
- 所有 CSS 样式都转换为内联样式（inline style）
- 确保公众号编辑器能够正确识别和保留样式
- 不使用 CSS 类名或外部样式表

### 5. 代码高亮
- 使用 `highlight.js` 进行语法高亮
- macOS 风格的代码块装饰（红黄绿三色圆点）
- 支持多种编程语言
- 深色主题背景（#383a42）

### 6. 响应式设计

#### 桌面端（> 1024px）
- 左右分栏布局
- 编辑器在左，预览在右

#### 平板端（769px - 1024px）
- 保持左右分栏
- 调整内边距优化显示

#### 移动端（≤ 768px）
- **上下布局**
- 编辑器在上（占 40% 高度）
- 预览在下（可滚动）
- 隐藏部分UI元素（星标按钮、提示文字）
- 全宽按钮和选择器

### 7. 其他功能
- **样式收藏**：支持收藏常用样式，使用 localStorage 持久化
- **文件上传**：支持直接上传 .md 或 .markdown 文件
- **一键复制**：点击按钮即可复制富文本到剪贴板
- **Toast 提示**：操作反馈（成功/失败/处理中）

## 技术栈

- **前端框架**：Vue 3（CDN 引入，无需构建）
- **Markdown 解析**：markdown-it 14.0.0
- **代码高亮**：highlight.js 11.9.0
- **样式**：纯 CSS（CSS Variables + 内联样式）
- **部署**：静态文件，可用任何 HTTP 服务器

## 文件结构

```
公众号编辑器/
├── index.html        # 主页面，包含 UI 和样式
├── app.js           # Vue 应用逻辑和核心功能
├── styles.js        # 13 种样式主题配置
├── simple-test.md   # 手工验证用示例文档
├── Dockerfile       # 静态部署镜像
├── README.md        # 项目说明文档
├── CONTRIBUTING.md  # 贡献说明
├── CHANGELOG.md     # 更新日志
└── LICENSE          # 许可证
```

## 关键技术实现

### 1. 图片分组算法
```javascript
// 位置：app.js -> groupConsecutiveImages()
// 功能：识别连续的图片并分组
// 算法：
1. 遍历 DOM 子元素，找出所有图片（包括 <p><img></p> 结构）
2. 按索引位置判断是否连续（允许中间有1个空白节点）
3. 将连续的图片归为一组
4. 对每组图片创建网格布局（group.length >= 2）
```

### 2. 样式应用流程
```javascript
// 位置：app.js -> renderMarkdown() -> applyInlineStyles()
// 流程：
Markdown 输入
    ↓
markdown-it 解析为 HTML
    ↓
应用选中样式的内联 CSS
    ↓
处理图片网格布局
    ↓
包裹容器样式
    ↓
渲染到预览区
```

### 3. 复制到公众号流程
```javascript
// 位置：app.js -> copyToClipboard()
// 流程：
获取渲染后的 HTML
    ↓
将 Grid 转换为 Table（convertGridToTable）
    ↓
图片转 Base64（convertImageToBase64）
    ↓
包裹 section 容器（如有背景色）
    ↓
简化代码块结构
    ↓
列表项扁平化
    ↓
写入剪贴板（text/html + text/plain）
```

## 样式配置规范

每个样式主题的配置结构（`styles.js`）：
```javascript
{
  'style-key': {
    name: '样式显示名称',
    styles: {
      container: '容器样式',
      h1: '一级标题样式',
      h2: '二级标题样式',
      // ... 更多元素
      img: 'max-width: 100%; max-height: 400px; height: auto; display: block; ...',
      // 所有样式必须是完整的内联CSS字符串
    }
  }
}
```

## 约束和限制

1. **图片处理**
   - 单张图片最大高度：400px
   - 网格图片最大高度：360px
   - 所有图片保持原始宽高比

2. **公众号兼容性**
   - 不支持：CSS Grid, Flexbox（部分）, CSS Variables
   - 必须使用：内联样式、Table 布局
   - 图片必须：Base64 编码或公众号可访问的URL

3. **浏览器要求**
   - 需要支持 ES6+
   - 需要支持 Clipboard API
   - 需要支持 Fetch API

## 开发指南

### 本地运行
```bash
# 使用 Python 启动服务器
python3 -m http.server 8080

# 访问 http://localhost:8080
```

```bash
# 或使用 Docker
docker build -t huasheng-editor .
docker run --rm -p 8080:80 huasheng-editor

# 访问 http://localhost:8080
```

### 添加新样式主题
1. 在 `styles.js` 中添加新的样式配置对象
2. 确保对应主题出现在当前 UI 的主题选择列表中
3. 确保所有必需的元素样式都已定义
4. 保持当前统一基础约束：正文 `15px`、正文行高 `1.75`、表格左对齐、图片无边框
5. 测试各种 Markdown 元素的渲染效果和复制结果

### 修改图片布局
1. **网格布局逻辑**：`app.js -> groupConsecutiveImages()`
2. **网格列数规则**：根据图片数量判断（2列、3列等）
3. **高度限制**：修改 `max-height` 值
4. **Grid 转 Table**：`app.js -> convertGridToTable()`

### 调整公众号兼容性
1. 所有新增的 CSS 特性都要检查公众号支持情况
2. 复制功能中可能需要额外的转换步骤
3. 测试方法：实际粘贴到公众号编辑器验证

## 常见问题

### Q1: 为什么预览正常，但粘贴到公众号后图片布局乱了？
A: 公众号不支持 CSS Grid。已实现自动转换为 Table 布局，确保 `convertGridToTable()` 正确执行。

### Q2: 图片在公众号中显示不出来怎么办？
A: 复制时会自动转为 Base64。如果失败，检查图片 CORS 策略或使用公众号素材库的图片。

### Q3: 如何调整图片高度限制？
A:
- 单张图片：修改 `styles.js` 中所有 `img` 样式的 `max-height` 值
- 网格图片：修改 `app.js -> groupConsecutiveImages()` 中的 `max-height` 值

### Q4: 移动端布局如何调整？
A: 修改 `index.html` 中的媒体查询（@media）样式，调整 `#app` 的 grid 布局和各面板的高度比例。

### Q5: 为什么预览区左右有留白，但复制出去没有？
A: 预览区的留白只用于本地阅读，不参与复制。复制时使用的是渲染后的正文 HTML，因此输出仍按主题正文容器样式铺满。

### Q6: 图片存储在哪里？会占用多少空间？
A: 图片存储在浏览器的 IndexedDB 中（数据库名：`WechatEditorImages`）。
- 默认存储空间：约 50MB-100MB（视浏览器而定）
- 压缩后的图片：平均 200KB-500KB
- 可存储：约 100-500 张图片
- 清除浏览器数据会导致图片丢失，建议定期导出重要内容

### Q7: 如何清理存储的图片？
A: 打开浏览器开发者工具（F12）→ Application → IndexedDB → WechatEditorImages，可手动删除数据库。未来版本将提供可视化管理界面。

## 未来规划

### 短期（1-2个月）
- [ ] **图片管理面板**：可视化管理所有存储的图片
- [ ] **批量导出功能**：将 `img://` 协议转为标准 Base64 Markdown
- [ ] **存储空间监控**：实时显示 IndexedDB 使用情况
- [ ] **图片压缩配置**：允许用户自定义压缩参数

### 中期（3-6个月）
- [ ] **支持自定义样式主题**：可视化样式编辑器
- [ ] **云端同步**：可选的图片云存储方案
- [ ] **更多 Markdown 扩展**：支持数学公式、流程图等
- [ ] **撤销/重做功能**：完整的编辑历史

### 长期（6-12个月）
- [ ] **草稿自动保存**：防止内容丢失
- [ ] **导出为图片**：整篇文章转为长图
- [ ] **多人协作编辑**：实时协作功能
- [ ] **AI 辅助写作**：集成 AI 优化建议

---

**注意**：本文档记录项目的技术架构和实现细节，应与仓库当前代码和 README 保持一致。
