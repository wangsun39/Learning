# Welcome to MkDocs

### MkDocs 搭建与 GitHub Pages 部署全流程指南

本文档完整记录了从零开始，使用 MkDocs 将 Markdown 笔记构建为静态网页，并部署至 GitHub Pages 的全过程，包含了环境配置、避坑指南、数学公式渲染及 Git 版本控制。

---

### 一、 环境准备与 MkDocs 安装

#### 1. Python 版本要求
*   **推荐版本**：Python 3.10 或更高版本。（不建议使用刚发布的最新测试版如 3.15，因为部分依赖包尚未适配，极易引发源码编译报错）。
*   **安装方式**：前往 Python 官网下载安装包，安装时务必勾选 **"Add python.exe to PATH"**。

#### 2. 创建并激活虚拟环境
打开终端，进入你的项目根目录，执行以下命令：

```powershell
# 1. 创建虚拟环境（如果你有多个版本，可用 py -3.10 -m venv .venv 指定版本）
python -m venv .venv

# 2. 激活虚拟环境 (Windows PowerShell)
.venv\Scripts\Activate.ps1
```

#### 3. 安装 MkDocs 扩展
- 系统策略避坑：如果你使用的是 Windows 11，且开启了“智能应用控制（Smart App Control）”，系统可能会拦截虚拟环境自动生成的 pip.exe 和 mkdocs.exe。不要轻易关闭系统安全策略。
- 解决方案：统一使用 python -m 的方式调用模块，绕过 .exe 拦截。

在激活的虚拟环境中执行：

```powershell
# 安装 MkDocs
python -m pip install mkdocs

# 安装数学公式扩展插件
python -m pip install pymdown-extensions

# 安装 Material 主题
python -m pip install mkdocs-material
```
#### 4. 验证安装
```powershell
python -m mkdocs --version
```

### 二、 项目目录结构与 Git 规范

将 Markdown 笔记整理进 docs/ 文件夹，虚拟环境和配置文件留在根目录。
```text
项目根目录/
├── .venv/                 # 虚拟环境（绝对不要提交到 Git）
├── .gitignore             # 必须包含 .venv/ 和 site/
├── docs/                  # 仅存放 Markdown 文档
│   ├── index.md           # 首页
│   ├── Algorithm/         # 分类文件夹
│   ├── Physics/           # 建议在里面放一个 index.md，避免导航 404
│   └── javascripts/       # 用于存放 mathjax.js 渲染脚本
├── mkdocs.yml             # MkDocs 配置文件（提交）
├── requirements.txt       # 依赖清单（提交）
└── README.md              # 项目说明（提交）
```

Git 提交注意：.venv 文件夹体积大且仅限本机使用，必须通过 .gitignore 忽略。Git 提交操作（git add/commit/push）不需要在虚拟环境中执行。

### 三、 核心配置 (mkdocs.yml)
重点解决导航生成与 LaTeX 公式渲染问题。

```yaml
site_name: 你的项目名称
site_url: https://你的用户名.github.io/你的仓库名/

# 推荐不写 nav，让 MkDocs 自动按目录结构生成导航，避免因缺少 index.md 导致 404
# 若需自定义，务必写具体到 .md 文件，如：- 物理: Physics/复数.md

# 启用数学公式渲染扩展
markdown_extensions:
  - md_in_html
  - pymdownx.arithmatex:
      generic: true
  - pymdownx.details
  - pymdownx.superfences:
      custom_fences:
        - name: math # 启用 math 块的用法
          class: arithmatex
          format: !!python/object/apply:pymdownx.arithmatex.arithmatex_fenced_format
            kwds:
              mode: generic

# 引入 MathJax 脚本（用来渲染公式）
extra_javascript:
  - javascripts/mathjax.js
  - https://unpkg.com/mathjax@3/es5/tex-mml-chtml.js
```
⚠️ **注意**：必须在 docs/javascripts/ 目录下创建 mathjax.js 文件，填入适配 MkDocs 的 MathJax 配置代码（此步不可省略，否则公式无法渲染）。

### 四、 LaTeX 公式书写规范
相比于 GitHub 自带的渲染，MkDocs 结合 MathJax 对复杂公式的支持更好。

- **行内公式**：使用单个 $ 包裹，如 \(i=\sqrt{-1}\)。

- **多行/复杂公式**：必须使用 $$ 独占一行包裹。

- **避坑**：不要使用 GitHub 风格的 ```math 代码块，当前的 pymdownx.arithmatex 插件不支持该语法。\begin{aligned} 或 \begin{cases} 必须放在 $$ 内。


### 五、 本地预览、提交与部署流程
#### 1. 本地预览

```powershell
python -m mkdocs serve
```
浏览器访问 http://127.0.0.1:8000 检查排版和公式。确认无误后按 Ctrl+C 停止服务。

#### 2. 提交源码至 main 分支
```powershell
# 记录当前虚拟环境的依赖
python -m pip freeze > requirements.txt

# Git 提交（不需要在虚拟环境下运行）
git add .
git commit -m "docs: 配置 MkDocs 并更新笔记"
git push origin main
```

#### 3. 一键构建并部署
```powershell
python -m mkdocs gh-deploy
```
原理：该命令会自动将 Markdown 编译成纯静态 HTML 文件，并推送到远程仓库的 gh-pages 分支。你的 main 分支保留源码，gh-pages 分支存放网页产物。

#### 4. GitHub 开启 Pages 服务
部署完成后，进入 GitHub 仓库的 Settings -> Pages：

- Source 选择 Deploy from a branch。

- Branch 选择 gh-pages，目录选 / (root)，点击 Save。

等待 1-2 分钟，页面顶部会出现 Your site is live at https://你的用户名.github.io/仓库名/，点击即可访问。

### 六、 日常更新流程
后续修改笔记只需重复以下三步：

- 编写或修改 docs/ 下的 Markdown 文件。

- git add . -> git commit -m "更新" -> git push origin main。

- python -m mkdocs gh-deploy 重新部署发布。
