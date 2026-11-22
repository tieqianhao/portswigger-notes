# 快速开始：30 分钟把仓库推上 GitHub

## 第 1 步：在 GitHub 建仓库（2 分钟）

1. 打开 https://github.com/new
2. Repository name：`portswigger-notes`
3. 选 **Public**（这就是给面试官看的）——建议先建 Private 整理几天再公开，也可以
4. **不要**勾选 "Add a README / .gitignore / license"（本地已有）
5. 点 Create repository，**不要点页面上其他任何按钮**

## 第 2 步：推送（在本目录打开 Git Bash，逐条粘贴）

```bash
cd /c/Users/余星乐/WorkBuddy/2026-10-14-23-18-51/output/portswigger-notes
git init
git add .
git commit -m "init: PortSwigger 学习笔记仓库骨架"
git branch -M main
git remote add origin https://github.com/tieqianhao/portswigger-notes.git
git push -u origin main
```

> 第一次推会弹浏览器登录，用 GitHub 账号授权即可。若报 403，说明凭据过期：控制面板 → 凭据管理器 → Windows 凭据 → 删掉 `git:https://github.com` 再推一次。

## 第 3 步：仓库门面（推完马上做，共 3 处）

1. **About 栏**（仓库右上角齿轮）：Description 填——

   ```
   PortSwigger Web Security Academy 学习笔记：每篇 = 漏洞原理 + 利用步骤 + 防御视角
   ```

   Website 填你的 B 站主页或博客（有的话）；Topics 添加：`web-security`、`portswigger`、`burp-suite`、`security`、`notes`

2. ** topics 是搜索权重**：HR 和安全圈搜 "portswigger" 能搜到你。

## 第 4 步：日常写作流（每天 15 分钟）

1. 刷完一个 lab，**当天**复制 `_template.md` 到对应目录，改名 `LabXX-英文名.md`
2. 四个板块按模板填——重点写"卡住的地方"，那是你独有的东西
3. 更新 README 进度表那一行（如 `| SQL 注入 | 3/23 | ... | 🚧 进行中 |`）
4. 提交：

```bash
git add . && git commit -m "notes: SQL注入 Lab01-03" && git push
```

## 节奏建议（结合你手上的事）

| 时间 | 动作 |
|---|---|
| 本周 | 先把**已刷的 30+ 个 lab** 凭记忆补成笔记，每个 15 分钟，一天 4-5 个 |
| 之后 | 每刷 1 个新 lab 当天 1 篇笔记；与 BUUCTF 一天一题并行 |
| 2027.2 | 简历定稿时，把仓库链接放进"技能与实战"栏 |
| 2027.3 | 投暑期实习前，README 进度表至少让 SQL/XSS/访问控制三行变绿 |

## 可选：GitHub Pages 博客化（以后再说）

仓库够大（20+ 篇）后，Settings → Pages → 选 main 分支，用 Jekyll 主题自动变成博客，简历上就能放 `tieqianhao.github.io/portswigger-notes`。现在不用折腾。
