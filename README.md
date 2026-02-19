# 个人博客

基于 VitePress 构建的个人博客，使用 Markdown 编写内容，支持 Obsidian 双链语法。

🌐 在线预览：https://silisland.github.io

## ✨ 特性

- 📝 使用 Markdown 编写，支持 Obsidian 双链语法
- 🎨 基于 VitePress，快速且 SEO 友好
- 💬 集成 Giscus 评论系统
- 🚀 自动部署到 GitHub Pages
- 📱 响应式设计，支持移动端

---

## 📁 项目结构

```
silisland.github.io/                    # 项目根目录
│
├── 📁 .github/                         # GitHub 配置目录
│   └── 📁 workflows/                   # CI/CD 自动化工作流
│       └── 📄 deploy.yml               # 部署配置：构建并发布到 GitHub Pages
│
├── 📁 .vitepress/                      # VitePress 核心配置目录
│   ├── 📄 config.ts                    # 站点主配置：标题、导航、侧边栏、插件
│   ├── 📁 theme/                       # 主题自定义
│   │   └── 📄 index.ts                 # 主题入口配置
│   └── 📁 dist/                        # 构建输出目录（自动生成的静态网站）
│
├── 📁 metadata/                        # 网站元数据配置
│   └── 📄 ...                          # 作者信息、站点描述等
│
├── 📄 index.md                         # 网站首页内容
│
├── 📄 package.json                     # Node.js 项目配置
├── 📄 pnpm-lock.yaml                   # 依赖锁定文件
├── 📄 .gitignore                       # Git 忽略规则
│
└── 📁 笔记/                            # Obsidian 笔记目录
    ├── 📄 开始.md                      # 具体笔记内容
    ├── 📄 主题A.md                     # 更多笔记...
    └── 📁 附件/                        # 图片、PDF 等附件
```

---

## 📝 日常使用

```
📄 index.md              ← 编辑首页
📁 笔记/                  ← 放你的 Obsidian 笔记
📄 .vitepress/config.ts   ← 修改导航/侧边栏（偶尔）
```

其他都是自动运行的，无需手动修改。

---

## 🛠 技术栈

| 层级      | 技术                              | 说明                                 |
| ------- | ------------------------------- | ---------------------------------- |
| **框架**  | VitePress                       | Vue 驱动的静态站点生成器，专为文档和博客优化           |
| **包管理** | pnpm                            | 高效的 Node.js 包管理器                   |
| **内容**  | Markdown + Obsidian             | 使用标准 Markdown 语法，兼容 Obsidian 的双链笔记 |
| **评论**  | Giscus                          | 基于 GitHub Discussions 的评论系统        |
| **部署**  | GitHub Pages / Vercel / Netlify | 支持多种静态托管方案                         |

---

## 🚀 快速开始

```bash
pnpm install      # 安装依赖
pnpm docs:dev     # 本地开发预览   Local:   http://localhost:5173/
pnpm docs:build   # 构建生产版本
```

需要自定义的内容：
- `metadata/index.ts` → 网站标题、描述等元数据
- `index.md` → 首页内容
- `.vitepress/creators.ts` → 作者信息（用于文章底部贡献者链接）

这个模板的优势是**保持 Markdown 纯净性**（不用 Obsidian 专有的 `[[wikilink]]` 语法），确保内容在 GitHub 上也能正常渲染，方便长期存档和迁移。

---

