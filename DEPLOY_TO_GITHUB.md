# 部署到 GitHub 的完整指南

> 从你电脑上零基础开始，到一个可分享的公网链接，全程 10 分钟。

## 你需要准备的

- GitHub 账号（github.com 注册，免费）
- 电脑装好 Git（macOS 自带；Windows 装 https://git-scm.com/download/win）
- 终端能用（macOS 用 Terminal；Windows 用 Git Bash 或 PowerShell）

## 一次性配置（如果你以前没用过 Git）

打开终端，跑这两行（替换成你自己的名字和邮箱）：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## 第一步：在 GitHub 网站上创建仓库

1. 登录 [github.com](https://github.com)
2. 点右上角的 **+** 号 → **New repository**
3. **Repository name** 填 `vibe-check`（小写、用连字符）
4. **Description** 可填："An AI music curator that turns moods into playlists"
5. 选 **Public**（不然 GitHub Pages 用不了，除非你升级账号）
6. **不要**勾 "Add a README file" / "Add .gitignore" / "Choose a license"（我们等会儿用本地的）
7. 点 **Create repository**

GitHub 会显示一个页面，**保留这个页面打开**——等会儿要用到上面的命令。

---

## 第二步：在你电脑上整理项目文件夹

在你电脑上任意位置，建一个文件夹叫 `vibe-check`，把这些文件放进去：

```
vibe-check/
├── vibe_check.html          ← 我做的 demo（重命名为 index.html 会更方便，见下文）
├── README.md                ← 我做的 README
├── PRD.md                   ← 我做的 PRD
├── build_index.py           ← 数据预处理脚本
├── data/
│   └── spotify_songs.csv    ← 原始数据
└── original_analysis/       ← 你原来的学校项目
    ├── research_question_1.py
    ├── research_question_2.py
    ├── research_question_3.py
    ├── test_project.py
    └── dataset_summary.py
```

**💡 重要小技巧**：把 `vibe_check.html` **改名为 `index.html`**——这样 GitHub Pages 会自动加载它，访问链接更短更专业。

---

## 第三步：把项目推送到 GitHub

在终端里，进入你的项目文件夹：

```bash
cd ~/path/to/vibe-check    # ← 替换成你的实际路径
```

然后**按顺序**跑这几条命令（每条单独跑一次）：

```bash
# 1. 把这个文件夹标记成一个 Git 仓库
git init

# 2. 把所有文件加入暂存区
git add .

# 3. 创建第一个 commit
git commit -m "Initial commit: Vibe Check v0.1 with bilingual support"

# 4. 主分支命名为 main
git branch -M main

# 5. 关联到 GitHub 仓库（注意改成你的用户名）
git remote add origin https://github.com/你的用户名/vibe-check.git

# 6. 推送
git push -u origin main
```

第 6 步可能会弹出登录窗口——**用你的 GitHub 账号登录就行**。

> 💡 如果它要你输入 Personal Access Token：
> 1. 去 [github.com/settings/tokens](https://github.com/settings/tokens)
> 2. 点 **Generate new token (classic)**
> 3. 勾上 `repo` 权限，生成
> 4. 复制 token，粘到密码栏（**不要**输入你的 GitHub 密码）

跑完之后刷新你的 GitHub 仓库页面，应该能看到所有文件出现了。

---

## 第四步：开启 GitHub Pages（让网页能在公网访问）

1. 进入你的仓库页面
2. 点顶部菜单的 **Settings**
3. 左侧栏滚到 **Pages**
4. **Source** 选 `Deploy from a branch`
5. **Branch** 选 `main`，文件夹保持 `/(root)`
6. 点 **Save**

等 1–2 分钟，刷新页面，上面会出现绿色框：

> ✅ Your site is live at https://你的用户名.github.io/vibe-check/

这就是你的 **永久公网链接**，可以直接发简历、发给朋友、贴到 LinkedIn。

如果你把 `vibe_check.html` 改名为 `index.html`了，那这个链接直接就能打开。如果没改名，链接得是 `https://你的用户名.github.io/vibe-check/vibe_check.html`。

---

## 第五步：以后改了文件怎么更新？

只要在项目文件夹里跑这三条：

```bash
git add .
git commit -m "你的更新说明"
git push
```

GitHub Pages 会自动重新部署，1–2 分钟后链接就能看到新版本。

---

## 常见踩坑

| 问题 | 解决方案 |
|---|---|
| `git push` 报 `remote: Repository not found` | 检查 `git remote -v`，看 URL 写对没；用户名拼对没 |
| 推送时要密码但密码总错 | GitHub 早就不让用密码了，用上面说的 Personal Access Token |
| GitHub Pages 页面 404 | 等 1-2 分钟；确认 `index.html` 或 `vibe_check.html` 在仓库根目录；URL 拼对 |
| Chinese characters 显示成问号 | 确认 `<meta charset="UTF-8">` 在 HTML 里（我做的版本已经有了，没事） |
| 文件太大 push 失败 | `spotify_songs.csv` 是 8MB，正常 GitHub 范围内；如果有更大文件再说 |

---

## 完成后的检查清单

- [ ] 公网链接能打开
- [ ] 默认显示英文，点 EN / 中 切换能切到中文
- [ ] 点 6 个 chip 按钮都能正常出歌单
- [ ] 自己输入英文或中文 prompt 也能出歌单
- [ ] README 在 GitHub 主页面正常渲染（Markdown 有格式）
- [ ] PRD.md 能点开看

全部 ✅ 就可以贴简历了。

---

## 在简历 / LinkedIn 上怎么写

**项目栏推荐写法（中英文版本各一）：**

> **Vibe Check — AI Music Curator (Personal Project, 2025)**
> Designed and built an end-to-end natural-language music recommender as an AI PM portfolio piece. Translates free-text mood descriptions into audio-feature targets (energy, valence, tempo, etc.), retrieves from a 2,048-track index, and generates human-readable explanations. Bilingual (EN/ZH), zero-cost deployment, shipped via GitHub Pages.
> **Tech**: HTML/JS, optional Claude API integration. **[Live demo](URL)** · **[GitHub](URL)**

> **Vibe Check — AI 音乐策展人（个人项目，2025）**
> 作为 AI PM 作品集独立设计并实现的端到端自然语言音乐推荐器。将用户的自由文本心情描述翻译成音频特征目标（energy/valence/tempo 等），从 2,048 首索引中检索匹配，生成人类可读的解释文本。支持中英双语、零成本部署、通过 GitHub Pages 上线。
> **技术栈**：HTML/JS，可选 Claude API 集成。**[Live demo](URL)** · **[GitHub](URL)**
