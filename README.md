# Markdown 排版器

<div align="center">
  <img src="logo.svg" width="120" height="120" alt="Markdown 排版器">
  <p>一个专为微信公众号排版优化的纯前端编辑器。</p>
</div>

## 功能概览

- 实时 Markdown 渲染，左侧编辑，右侧预览
- 一键复制富文本，直接粘贴到公众号后台
- 13 个内置主题，覆盖公众号、媒体、专栏、品牌表达等场景
- 图片本地压缩和 IndexedDB 持久化，刷新后不丢失
- 连续图片自动网格排版，复制时自动转为公众号兼容的 table
- 支持图片粘贴、拖拽、Markdown 文件导入、代码高亮、响应式布局

## 当前主题

当前保留 13 个主题，不删除，仅按风格分组整理，方便选择：

### 通用基础

- `default` / 公众号风格
- `tech` / 技术风格
- `deepread` / 深度阅读

### 国际媒体

- `nyt` / 纽约时报
- `nikkei` / Nikkei 日経
- `atlantic-feature` / Atlantic 特稿
- `monocle-brief` / Monocle 简报

### 中文内容

- `latepost-depth` / 晚点风格
- `sspai-column` / 少数派专栏
- `renwu-profile` / 人物特稿
- `biz-insight` / 商业观察

### 品牌表达

- `wechat-anthropic` / Claude
- `warm-notes` / 轻品牌手记

## 最近样式调整

- 主题容器统一为更适合公众号复制的满宽正文布局
- `container` 外层背景色已移除，四周留白显著减少
- 正文字号统一为 `15px`，正文行高统一为 `1.75`
- `h1` 到 `h6` 的字号、粗细、行高、间距已统一
- `p`、列表、引用、代码块、表格的字号和节奏已统一
- 图片边框已移除，单图高度统一为 `400px`
- `hr`、标题 margin、表格内边距、表格左对齐已统一

## 预览与复制的区别

- 预览区两侧保留少量留白，便于本地阅读和观察排版
- 实际复制使用的是渲染后的正文 HTML，不受预览容器 padding 影响
- 复制到公众号或其他平台时，内容会按主题正文容器样式输出，不会带上预览区的额外留白

## 本地运行

### 方式一：直接启动静态服务

```bash
python3 -m http.server 8080
```

访问 `http://localhost:8080`

### 方式二：使用 Docker

```bash
docker build -t huasheng-editor .
docker run --rm -p 8080:80 huasheng-editor
```

访问 `http://localhost:8080`

## 技术栈

- Vue 3
- markdown-it
- highlight.js
- Turndown
- IndexedDB
- Canvas API
- 纯 CSS + 内联样式输出

## 项目结构

```text
huasheng_editor/
├── index.html        # 页面结构
├── app.css           # 编辑器外壳（布局、工具栏、侧栏等）样式
├── app.js            # 编辑器逻辑、渲染、复制、图片处理
├── styles.js         # 13 个主题的内联样式定义
├── Dockerfile        # 静态部署镜像
├── README.md
├── CHANGELOG.md
├── LICENSE
├── logo.svg
└── favicon.svg
```

## 图片处理

图片处理走本地协议和持久化存储链路：

```text
粘贴或拖拽图片
  -> Canvas 压缩
  -> IndexedDB 存储
  -> Markdown 中写入 img:// 短链接
  -> 预览时恢复为 blob URL
  -> 复制时转为 Base64
```

这样可以同时解决编辑卡顿、图片丢失和公众号兼容性问题。

## 公众号兼容策略

- CSS Grid 在复制前自动转为 table
- 样式统一转为 inline style
- 本地图片在复制时自动转为 Base64
- 针对公众号兼容性保留必要的强制样式

## 开发说明

### 添加或调整主题

1. 直接修改 [styles.js](styles.js) 中对应主题
2. 保持 `container`、标题、正文、列表、引用、代码、表格、图片等标签定义完整
3. 优先保证公众号复制结果，再看预览效果
4. 用编辑器内置示例内容检查常见标签是否一致

### 验证建议

- 本地打开后检查预览区左右留白是否仅影响预览
- 复制到公众号后台或其他富文本平台验证正文是否铺满
- 检查标题、引用、代码块、表格、图片在不同主题下是否一致

## 贡献

欢迎提交 Issue 和 Pull Request。

## 许可证

本项目基于 [MIT License](LICENSE) 开源。
