你好！很高兴能以全栈开发者和技术写作专家的身份为你服务。

Hexo 是一个非常适合作为**“数字花园”**的工具，它轻量、纯净，且对程序员极度友好。既然你目前的重点是记录学习笔记，我会特别优化关于**图片管理**和**一键发布**的流程，让你能把更多精力放在思考上，而不是折腾工具。

---

## 🚀 快速上手索引

1. [环境准备：工欲善其事](https://www.google.com/search?q=%231-%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
2. [初始化：建立你的笔记库](https://www.google.com/search?q=%232-%E5%88%9D%E5%A7%8B%E5%8C%96%E5%8D%9A%E5%AE%A2)
3. [写作实战：Markdown 与图片管理](https://www.google.com/search?q=%233-%E5%86%99%E7%AC%94%E8%AE%B0%E7%9A%84%E6%96%B9%E6%B3%95)
4. [上线部署：让全世界看到你的笔记](https://www.google.com/search?q=%234-%E9%83%A8%E7%BD%B2%E5%88%B0-github-pages)
5. [核心流程：日常更新 3 步走](https://www.google.com/search?q=%235-%E6%97%A5%E5%B8%B8%E6%9B%B4%E6%96%B0%E6%B5%81%E7%A8%8B)
6. [效率加持：VS Code 插件与主题推荐](https://www.google.com/search?q=%236-%E6%95%88%E7%8E%87%E5%8A%A0%E6%8C%81)
7. [疑难杂症：排查指南](https://www.google.com/search?q=%237-%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5)

---

## 1. 环境准备

在开始之前，我们需要给电脑装上“发动机”和“传动轴”。

* **Node.js**: Hexo 的运行环境。建议下载 **LTS（长期支持版）**。
* [下载地址](https://nodejs.org/)
* `[截图：官网下载页面，红框圈出 LTS 版本]`


* **Git**: 负责把你的笔记上传到 GitHub。
* [下载地址](https://git-scm.com/)
* `[截图：Git 安装向导中选择默认配置的界面]`



安装完成后，打开终端（Windows 用 CMD 或 PowerShell，Mac 用 Terminal），输入以下命令检查：

```bash
node -v  # 看到 v20.x.x 等字样即成功
git --version # 看到 git version 2.x.x 即成功

```

### 安装 Hexo CLI

在终端输入这行命令，它是 Hexo 的指挥棒：

```bash
npm install -g hexo-cli

```

---

## 2. 初始化博客

找一个你喜欢的地方（比如 `D:/MyBlog` 或 `~/Documents/Blog`），运行：

```bash
hexo init my-notes  # 创建文件夹并初始化
cd my-notes         # 进入文件夹
npm install         # 安装必要的组件

```

### 目录结构说明

```text
.
├── _config.yml         # 【核心】站点的配置文件，改名字、改链接都在这
├── package.json        # 依赖包信息
├── scaffolds           # 文章模板（新笔记的初始样子）
├── source              # 【最重要】你的笔记源码
|   └── _posts          # 所有的 .md 笔记文件都放在这里
├── themes              # 主题文件夹
└── public              # 渲染后的静态文件（自动生成，不用手动改）

```

---

## 3. 写笔记的方法

### 新建笔记

```bash
hexo new "我的第一篇AI学习笔记"

```

完成后，在 `source/_posts/` 下会生成一个同名的 `.md` 文件。

### Markdown 语法速查

| 元素 | 语法 |
| --- | --- |
| **标题** | `# 一级标题`, `## 二级标题` |
| **列表** | `- 项目1`, `1. 项目1` |
| **代码块** | ```python (换行) print("Hello") (换行) ``` |
| **表格** | ` |

### 🖼️ 重点：如何高效管理图片？

学习笔记离不开截图。我推荐使用 **“相对路径资源文件夹”** 模式：

1. **修改配置**：打开根目录的 `_config.yml`，找到 `post_asset_folder`，设为 `true`。
```yaml
post_asset_folder: true

```


2. **效果**：当你 `hexo new "笔记名"` 时，Hexo 会自动创建一个同名的文件夹。
3. **操作**：将截图直接丢进这个同名文件夹。
4. **引用**：在 Markdown 里直接写：
`![描述](图片文件名.png)`

> **专家建议**：配合 VS Code 插件（见下文）可以实现一键粘贴图片并自动保存到对应目录。

---

## 4. 部署到 GitHub Pages

1. **创建仓库**：在 GitHub 新建一个仓库，名字叫 `你的用户名.github.io`（必须这个格式）。
2. **安装部署插件**：在博客根目录运行：
```bash
npm install hexo-deployer-git --save

```


3. **修改 `_config.yml**`（拉到最底部）：
```yaml
deploy:
  type: git
  repo: https://github.com/你的用户名/你的用户名.github.io.git
  branch: main

```



---

## 5. 日常更新流程（核心！）

写完笔记后，你只需要执行这一串“连招”：

### 自动化脚本（推荐）

在 `package.json` 的 `scripts` 块中添加一行：

```json
"scripts": {
  "pub": "hexo clean && hexo g && hexo d"
}

```

### 极简发布流程：

1. **写作**：在 `_posts` 里用 Markdown 尽情挥洒。
2. **预览**（可选）：运行 `hexo s`，在浏览器访问 `localhost:4000` 查看效果。
3. **一键发布**：在终端输入：
```bash
npm run pub

```


**搞定！** 你的笔记已经在云端更新了。

---

## 6. 效率加持

### 适合 Hexo 的 VS Code 插件

* **Markdown All in One**: 提供快捷键、自动预览等增强功能。
* **Hexo Utils**: 可以在 VS Code 里直接点击按钮新建文章、预览、部署。
* **Paste Image**: **必装！** 设置路径为 `${imageSyntaxPrefix}${fileName}${imageSyntaxSuffix}`，实现截图后直接 `Ctrl+V` 插入笔记。

### 笔记风格主题推荐

1. **[Hexo Theme Keep](https://github.com/XPoet/hexo-theme-keep)**: 极简主义，非常适合纯文字和技术笔记。
2. **[Hexo Theme Butterfly](https://github.com/fluid-dev/hexo-theme-fluid)**: 功能强大，支持多种笔记布局和精美图库。
3. **[Hexo Theme NexT](https://github.com/theme-next/hexo-theme-next)**: 经典中的经典，学术气息浓厚，非常耐看。

---

## 7. 常见问题排查

* **图片不显示**：
* 检查 `_config.yml` 里的 `post_asset_folder` 是否为 `true`。
* 检查图片路径是否直接写了文件名（不要带文件夹路径，Hexo 渲染时会自动匹配）。


* **部署失败 (Permission Denied)**：
* 通常是 Git 权限问题。请确保你本地已配置 SSH Key 或者在弹出窗口中登录了 GitHub。


* **页面空白**：
* 运行 `hexo clean` 然后重新生成 `hexo g`。



---

**你想先从配置哪个主题开始尝试？或者需要我帮你写一个自动化的 GitHub Actions 脚本来实现“推代码即发布”？**

我为你准备了详细的配置方案：

### 1. 准备工作：设置 GitHub 访问令牌 (Token)

为了让 GitHub Actions 有权限把编译好的网页推送到你的仓库，我们需要创建一个秘钥。

1. **生成 Token**：点击 GitHub 头像 -> **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**。
2. 点击 **Generate new token (classic)**，勾选 `repo` 权限，生成并**复制**这个字符串。
3. **配置到仓库**：进入你的博客仓库 -> **Settings** -> **Secrets and variables** -> **Actions**。
4. 点击 **New repository secret**，名称填 `HEXO_DEPLOY_KEY`，值贴入刚才生成的 Token。

---

### 2. 创建自动化脚本文件

在你的 Hexo 根目录下，手动创建以下文件夹和文件：
`.github/workflows/deploy.yml`

将以下代码完整复制进去：

```yaml
name: 自动部署 Hexo 博客

on:
  push:
    branches:
      - main  # 监听 main 分支的提交，如果是 master 请修改

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 检出源码
        uses: actions/checkout@v3

      - name: 设置 Node.js 环境
        uses: actions/setup-node@v3
        with:
          node-version: '20' # 使用长期支持版
          cache: 'npm'      # 开启缓存，加快后续部署速度

      - name: 安装依赖
        run: npm install

      - name: 编译静态页面
        run: |
          npx hexo clean
          npx hexo generate

      - name: 部署到 GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: public       # Hexo 生成的静态文件目录
          branch: gh-pages     # 部署到的目标分支
          token: ${{ secrets.HEXO_DEPLOY_KEY }} # 使用我们刚才配置的秘钥

```

---

### 3. 修改 GitHub 仓库设置

提交上述文件后，你会发现仓库多了一个 `gh-pages` 分支。我们需要最后一步设置：

1. 进入仓库 **Settings** -> **Pages**。
2. 在 **Build and deployment** 下的 **Branch** 部分。
3. 将分支选为 `gh-pages`，文件夹选为 `/(root)`。
4. 点击 **Save**。

---

### 4. 以后你的日常流程

现在，你的更新流程变成了极其简单的 **1 个步骤**：

1. **写笔记并 Push**：
```bash
git add .
git commit -m "新增学习笔记"
git push origin main

```



**发生了什么？**

* GitHub 会自动检测到你的 Push。
* 在云端启动一个虚拟电脑，帮你跑 `npm install` 和 `hexo g`。
* 自动把生成的 `public` 文件夹内容推送到 `gh-pages` 分支。
* 你的网站在 1 分钟内就会自动更新。

---

### 💡 进阶：多设备同步

既然有了 GitHub Actions，我建议你把整个博客文件夹（包括 `source`, `_config.yml`, `themes` 等）都托管在 GitHub 上。

* **在家时**：写完笔记 `git push`。
* **在公司/学校时**：换台电脑只需 `git pull`，即可同步所有草稿。
* **无需重复配置**：只要那台电脑有 Git，你甚至不需要安装 Node.js 或 Hexo CLI，因为云端的 Actions 会帮你搞定发布。

如果你对这个自动化流程有任何报错提示，随时告诉我！ 你可以随时让我修改或删除预设操作。预设操作准备就绪时，“近期对话”中的本次对话旁边会出现一个小圆点。