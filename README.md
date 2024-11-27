# 摸鱼纪 (Moyu Journal)

> 「上班不摸鱼，和咸鱼有什么区别」

一个简单优雅的个人项目管理工具，帮助你记录和展示你的"摸鱼"成果。无需后端，纯前端实现，数据存储在浏览器本地。

![License](https://img.shields.io/github/license/sennes2/moyu-journal)

## ✨ 特性

- 🎯 简洁的项目管理界面
- 🎨 支持项目图标自定义（emoji）
- 📝 项目描述支持 Markdown 格式
- 🔗 支持添加项目演示和代码仓库链接
- 📱 响应式设计，支持移动端
- 💾 本地数据持久化
- 📤 支持数据导入/导出
- 🎮 支持拖拽排序
- 👤 支持自定义用户名

## 🚀 快速开始

### 在线体验

访问 [🐟.🍕🦊🔧.ws](https://🐟.🍕🦊🔧.ws) 即可开始摸鱼。

> 域名解释：鱼.披萨狐狸工具.ws，这可能是最适合摸鱼的域名了 😎

### 本地开发

```bash
# 克隆项目
git clone https://github.com/sennes2/moyu-journal.git

# 安装依赖
bun install

# 启动开发服务器
bun run dev

# 构建项目
bun run build

# 构建并部署
bun run bd
```

## 🛠️ 技术栈

- 🖼️ [Vue 3](https://vuejs.org/) - 渐进式 JavaScript 框架
- 🏗️ [Vite](https://vitejs.dev/) - 下一代前端构建工具
- 📦 [Bun](https://bun.sh/) - 高性能的包管理器
- 🎨 [vue3-emoji-picker](https://github.com/wobsoriano/vue3-emoji-picker) - Emoji 选择器
- 🎯 [vuedraggable](https://github.com/SortableJS/Vue.Draggable) - 拖拽排序
- 🔑 [nanoid](https://github.com/ai/nanoid) - 唯一 ID 生成

## 📝 功能说明

### 项目管理
- 创建、编辑、删除项目
- 自定义项目图标（emoji）
- Markdown 格式的项目描述
- 添加项目演示链接和代码仓库链接
- 拖拽调整项目顺序

### 数据管理
- 自动保存到浏览器本地存储
- 支持导出数据为 JSON 文件
- 支持导入之前导出的数据

### 个性化
- 自定义用户名
- 项目卡片布局自适应

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 开源协议

[MIT License](LICENSE) © 2024 摸鱼纪
