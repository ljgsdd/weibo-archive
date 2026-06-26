# 微博自动归档 · 零基础部署指南

> 📦 这是一套已经做好的"自动抓微博"模板。  
> 部署后，**GitHub 的服务器**会每 2 小时自动去微博抓一次"菩提树下那道光"（UID `1002568141`）的最新动态，存进你自己的仓库里。  
> **你的电脑关机也照跑**，半夜博主发的微博最多 2 小时后就抓到。

---

## 它能做到什么

- ✅ 24 小时不间断监控博主，关机也不影响
- ✅ 每条微博的**文字、原图、视频、评论**全部归档
- ✅ 数据存在你自己的 GitHub 仓库里，永久免费、永久属于你
- ✅ 数据格式 CSV + JSON + SQLite，三种全要，将来想怎么分析都行
- ✅ **完全免费**：GitHub Actions 每月免费 2000 分钟，本任务每次跑 1-3 分钟，月用量 ≈ 300 分钟，远远用不完
- ✅ 抓到新博后自动 commit，你点开仓库就能看历史时间线

## 它**不能**做到什么

- ❌ 不能抓只对粉丝可见的内容（你 Cookie 账号没关注就看不到）
- ❌ 不能实时推送到手机（如果要"博主一发就响"，建议**同时**开微博 App 的"特别关注"）

---

## 部署步骤总览（约 15 分钟）

```
1. 注册 GitHub 账号         （约 3 分钟）
2. 创建一个空仓库          （约 1 分钟）
3. 上传本模板的文件         （约 2 分钟）
4. 从浏览器抓微博 Cookie   （约 3 分钟）
5. 把 Cookie 存进 Secret   （约 2 分钟）
6. 开启 Actions、手动跑一次（约 4 分钟）
```

---

## 第 1 步 · 注册 GitHub 账号

1. 打开 [https://github.com/signup](https://github.com/signup)
2. 输入邮箱、密码、用户名（用户名建议英文，比如 `your_name_archive`，会出现在仓库地址里）
3. 邮箱验证后即可使用，**完全免费**

> 💡 GitHub 是全球最大的代码托管平台，归微软所有，可以放心长期使用。

---

## 第 2 步 · 创建一个空仓库（Repository）

1. 登录后，点右上角 **+** → **New repository**
2. 填写：
   - **Repository name**：`weibo-archive`（随便起名也行）
   - **Description**：可留空
   - **Public / Private**：建议选 **Private（私有）**——你的 Cookie 虽然不会进仓库，但归档下来的微博内容自己留着就好
   - **Initialize this repository with a README**：✅ 勾上
3. 点 **Create repository**

---

## 第 3 步 · 上传本模板的文件

把本压缩包解压后，里面有这些文件：

```
github_actions_weibo/
├── config.json                       ← 配置（已预填好 UID 1002568141）
├── .gitignore                        ← 告诉 Git 忽略哪些临时文件
├── README_零基础.md                  ← 这份文档
└── .github/
    └── workflows/
        └── weibo_daily.yml           ← GitHub Actions 工作流（核心）
```

**上传方法（网页版，最简单）：**

1. 进入你刚创建的仓库主页
2. 点 **Add file** → **Upload files**
3. 把 `config.json`、`.gitignore`、`README_零基础.md` 三个文件直接拖进去
4. 滚到下面点 **Commit changes**

> ⚠️ `.github/workflows/weibo_daily.yml` 因为是子目录里的文件，网页上传稍麻烦。  
> **最简单的办法**：上传完上面三个文件后，再次点 **Add file → Create new file**，在文件名框里**完整输入** `.github/workflows/weibo_daily.yml`（注意斜杠！GitHub 会自动建好目录），然后把模板里这个 yml 的内容**全部复制粘贴**进去，提交。

✅ 完成后，仓库里应该有 4 个内容：`.github/`、`.gitignore`、`README_零基础.md`、`config.json`（GitHub 默认还会有一个 `README.md`）。

---

## 第 4 步 · 从浏览器抓微博 Cookie ⚠️ 关键步骤

> **为什么需要 Cookie**：微博现在不让匿名访问个人主页接口，必须带"登录凭证"。Cookie 就是登录凭证。

### 操作（用 Chrome 或 Edge 浏览器）：

1. 打开 [https://weibo.com](https://weibo.com)，**登录你自己的微博账号**（用小号最佳）
2. **关注一下博主**（UID `1002568141`，方便后续抓评论等数据）
3. 登录状态下，按 **F12** 打开"开发者工具"
4. 切到顶部的 **Network（网络）** 标签
5. **保持 F12 窗口开着**，刷新一下 weibo.com 页面（按 F5）
6. 在 Network 列表里随便点一条名字以 `weibo.com` 开头的请求（比如 `home`、`config`、`getIndex` 都行）
7. 右侧切到 **Headers（标头）** 标签
8. 往下滚到 **Request Headers** → 找到 **`Cookie:`** 那一行
9. **完整复制冒号后面的所有内容**（一长串字符，可能上千字符），不要漏字符

> 📸 找不到？关键词搜 "Cookie" 用 Ctrl+F，整个 Network 里搜。

> 🔐 **安全提示**：Cookie 等于你的微博登录态。**永远不要发给任何陌生人，也不要直接 commit 到仓库**。下面 Secret 是 GitHub 加密存储的，是安全的。

---

## 第 5 步 · 把 Cookie 存进 Secret

1. 回到你的仓库主页
2. 点上方 **Settings**（齿轮图标）
3. 左边菜单 → **Secrets and variables** → **Actions**
4. 点 **New repository secret**
5. 填写：
   - **Name**：`WEIBO_COOKIE`（**必须一字不差**，大小写都要对）
   - **Secret**：粘贴第 4 步复制的整串 Cookie
6. 点 **Add secret**

✅ 完成后，列表里能看到 `WEIBO_COOKIE`（值已加密不可见）。

---

## 第 6 步 · 启用 Actions 并手动跑一次

1. 回仓库主页，点上方 **Actions** 标签
2. 第一次进入，GitHub 会问"是否启用 workflows"，点 **I understand my workflows, go ahead and enable them**
3. 左侧能看到 **Weibo Auto Archive** 这个 workflow
4. 点进去，右边有 **Run workflow** 按钮，**点它 → 再点绿色的 Run workflow** 立即跑一次

### 看运行结果：

- 状态变成 **🟡 黄色转圈** = 正在跑
- 变 **✅ 绿色对勾** = 成功
- 变 **❌ 红色叉** = 失败，点进去看日志（90% 是 Cookie 没贴对，重新做第 4-5 步）

### 首次跑完后：

- 仓库根目录会出现一个 **`weibo/`** 文件夹
- 里面有 `菩提树下那道光.csv`、`.json`、`.db`，以及图片视频子目录
- 之后每 2 小时自动增量更新

---

## 日常使用

### 在哪里看抓到的数据？

- **网页直接看**：进入仓库 → `weibo/` 文件夹 → 点开 csv/json 文件能预览
- **下载到本地**：仓库主页右上角 **Code → Download ZIP**
- **想做分析**：把 `weibo.db` 下载下来，用 [DB Browser for SQLite](https://sqlitebrowser.org/) 打开（免费、零基础友好）

### 自动跑的频率能调吗？

打开 `.github/workflows/weibo_daily.yml`，找到这一行：
```
- cron: '17 */2 * * *'
```
- `*/2` 表示每 2 小时跑一次
- 想每 1 小时一次改成 `*/1`，每 4 小时改成 `*/4`
- **不建议低于 1 小时**（容易被微博风控、也容易超 GitHub 免费额度）

---

## ⚠️ 长期维护注意事项

### 1. Cookie 会过期（重要！）

- 微博 Cookie 一般 **1-3 个月失效一次**
- 失效后 Actions 会跑失败（红叉）、抓不到新博
- **救命操作**：重新做第 4 步抓 Cookie，再做第 5 步**更新**那个 Secret（不用删，直接覆盖）
- **建议**：在手机日历建一个"每月 1 日检查微博归档"的提醒

### 2. 仓库大小

- GitHub 单仓库建议 < 1 GB
- 一年下来如果博主图片视频很多，可能会逼近这个值
- 真到了那一步，把仓库 clone 到本地备份，然后清空 `weibo/` 重新开始即可（旧数据已经在你本地了）

### 3. 博主突然不发了 / 删博了

- weibo-crawler 只抓**现在还能看到**的内容
- 博主自己删的博，本地早抓到的依然在你仓库里（这就是归档的意义）

### 4. 想加博主、想换博主

- 改 `config.json` 里的 `user_id_list`，比如 `["1002568141", "另一个UID"]`
- 改完 commit 到仓库，下次自动跑就生效

---

## 故障排查速查表

| 现象 | 原因 | 解决 |
|---|---|---|
| Actions 跑红叉，日志里有 `登录` / `400` / `Cookie` 字样 | Cookie 过期或没贴对 | 重做第 4-5 步 |
| Actions 跑红叉，日志里有 `Permission denied` 推送失败 | 仓库权限问题 | Settings → Actions → General → Workflow permissions 选 **Read and write permissions** |
| Actions 没自动跑 | GitHub 对不活跃仓库会停 cron | 进仓库随便编辑一下文件提交，或手动 Run workflow 一次激活 |
| `weibo/` 文件夹一直没出现 | 第一次跑没成功 | 看 Actions 日志最后几行的报错 |
| 想停掉自动跑 | —— | Actions → 选 workflow → 右上角 **...** → Disable workflow |

---

## 这套方案的"账"

| 项目 | 数据 |
|---|---|
| **金钱成本** | 0 元/月（GitHub Actions 个人免费版） |
| **时间成本** | 一次部署 ≈ 15 分钟，之后每月 5 分钟检查 Cookie |
| **抓取延迟** | 最长 2 小时（半夜发也最多 2 小时被抓） |
| **数据归属** | 100% 自己所有，存在自己 GitHub 仓库 |
| **可恢复性** | 仓库可一键下载到本地，永不丢失 |

---

## 还是搞不定？

部署完后任何一步卡住，回来直接告诉我**第几步**、**报什么错**或**截图**，我陪你逐步排错。

祝归档愉快 ⛩️
