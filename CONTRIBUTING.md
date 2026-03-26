# 贡献指南

感谢您对本项目的关注！欢迎提交 Issue 和 Pull Request。

## 提交 Issue

在提交 Issue 之前，请：
- 搜索已有的 Issue，避免重复
- 使用清晰的标题描述问题
- 提供详细的问题描述和复现步骤
- 如有可能，提供截图或错误信息

## 提交 Pull Request

1. Fork 本仓库
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的修改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建一个 Pull Request

### 开发规范

- 遵循现有的代码风格
- 添加必要的注释
- 确保所有功能在主流浏览器中正常工作
- 更新相关文档
- 这是一个纯前端静态项目，优先保持实现简单直接
- 修改主题时，直接维护 `styles.js` 中每个主题，不引入额外抽象

### 添加新样式

如果您想添加新的样式主题：

1. 在 `styles.js` 中添加新的样式配置对象
2. 确保包含完整的元素样式定义：`container`、`h1-h6`、`p`、`ul`、`ol`、`li`、`blockquote`、`code`、`pre`、`img`、`table`、`th`、`td`、`hr`
3. 保持与现有主题一致的基础排版约束：正文 `15px`、正文行高 `1.75`、表格左对齐、图片无边框
4. 主题可以有风格差异，但不要破坏当前统一的基础阅读节奏
5. 在 README.md 中更新主题说明或分组
6. 测试预览效果和复制到公众号后的实际结果

### 本地验证

```bash
python3 -m http.server 8080
```

如需容器化预览：

```bash
docker build -t huasheng-editor .
docker run --rm -p 8080:80 huasheng-editor
```

## 行为准则

- 尊重所有贡献者
- 欢迎建设性的批评和建议
- 专注于项目本身，避免人身攻击

## 许可证

通过贡献代码，您同意您的贡献将按照本项目的 MIT 许可证进行许可。
