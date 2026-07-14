# 网易云音乐全栈开源项目教程

> 从零开始，部署你自己的网易云音乐 Web 应用 —— 包含完整后端 API 服务与移动端前端 Web App。

---

## 目录

- [项目简介](#1-项目简介)
- [架构概览](#2-架构概览)
- [目录结构](#3-目录结构)
- [前置准备](#4-前置准备)
- [后端部署](#5-后端部署-api-服务)
- [前端部署与对接](#6-前端部署与对接)
- [环境变量说明](#7-环境变量说明)
- [API 接口速查](#8-api-接口速查)
- [开发指南](#9-开发指南)
- [开源贡献指南](#10-开源贡献指南)
- [常见问题排查](#11-常见问题排查)

---

## 1. 项目简介

本项目是全栈开源的网易云音乐 Web 应用方案，包含两个子项目：

### 🎯 前端：Music App (music-app)

一个移动端优先的纯原生 Web 音乐播放器，**零构建步骤**，直接用浏览器打开 `index.html` 即可运行。

**核心功能：**

- 🔍 **音乐搜索** — 支持歌曲、歌手搜索
- 📀 **完整播放器** — 迷你播放器 + 全屏播放器，播放/暂停/上下首/进度拖拽
- 🎵 **歌单系统** — 推荐歌单、每日推荐、歌单详情
- 📝 **歌词显示** — 逐行歌词滚动同步
- 🔐 **扫码登录** — 网易云音乐 APP 扫码登录
- 📱 **移动端适配** — 触摸手势、底部安全区适配
- 🎨 **动态主题色** — 从封面自动提取主题色
- 🔄 **播放模式** — 顺序/单曲/随机

### ⚙️ 后端：Netease Cloud Music API Enhanced (api-enhanced)

功能完善的网易云音乐第三方 Node.js API 服务，覆盖网易云音乐几乎所有功能接口（200+ 个）。

**核心能力：**

- 用户系统（登录、注册、信息）
- 音乐播放（歌曲 URL、音质选择、解灰）
- 搜索发现（搜索、推荐、排行榜）
- 歌单管理（创建、编辑、收藏）
- 动态社交（评论、点赞、私信、关注）
- 云盘上传、私人 FM、签到等

---

## 2. 架构概览

```
┌─────────────────────────────────────────────────────┐
│                    用户浏览器                          │
│                                                       │
│   ┌─────────────────────────────────────────────┐   │
│   │              Music App (前端)                  │   │
│   │                                               │   │
│   │  index.html ──→ api.js (API 层)               │   │
│   │      │            │                           │   │
│   │      │            ▼                           │   │
│   │  apii.js ←──  fetch('/song/url', '/search'...) │   │
│   │  (业务层)       │                              │   │
│   │      │          │                              │   │
│   │      ▼          ▼                              │   │
│   │  ┌──────────────────────────────┐              │   │
│   │  │   API_BASE 配置请求目标地址    │              │   │
│   │  └──────────────────────────────┘              │   │
│   └─────────────────────────────────────────────┘   │
└─────────────────────┬───────────────────────────────┘
                      │ HTTP / CORS
                      ▼
┌─────────────────────────────────────────────────────┐
│          Netease Cloud Music API Enhanced            │
│                                                       │
│   ┌──────────┐   ┌──────────┐   ┌────────────────┐  │
│   │  Express  │   │ 200+ API │   │  加解密引擎     │  │
│   │  路由层   │──→│  模块层   │──→│  eapi/weapi    │  │
│   └──────────┘   └──────────┘   │  /xeapi 等     │  │
│                                  └───────┬────────┘  │
│                                          │           │
└──────────────────────────────────────────┼───────────┘
                                           │
                                           ▼
                                 ┌──────────────────┐
                                 │  网易云音乐官方    │
                                 │  接口服务器        │
                                 │  interface.163   │
                                 └──────────────────┘
```

### 数据流

1. 用户在**前端**（Music App）上搜索歌曲
2. 前端调用 `api.js` 中的方法，发出 HTTP 请求
3. 请求到达**后端**（api-enhanced）
4. 后端对请求进行加密处理，转发至网易云音乐官方接口
5. 网易云返回数据，后端处理后回传给前端
6. 前端拿到数据，渲染 UI、播放音乐

---

## 3. 目录结构

```
.
├── music-app/                          # 前端项目
│   ├── index.html                      # 主入口（包含所有页面视图）
│   └── yinyueapi/
│       ├── api.js                      # API 请求层（312 行）
│       ├── apii.js                     # 业务逻辑层（1944 行）
│       └── yy.css                      # 全部样式（2037 行）
│
└── api-enhanced/                       # 后端项目
    ├── api-enhanced-4.36.2/
    │   ├── app.js                      # 入口（启动脚本）
    │   ├── server.js                   # Express 服务端核心
    │   ├── main.js                     # 程序化调用入口
    │   ├── package.json                # 依赖与脚本
    │   ├── Dockerfile                  # Docker 构建文件
    │   ├── vercel.json                 # Vercel 部署配置
    │   ├── generateConfig.js           # 配置生成器
    │   ├── module/                     # ★ 200+ API 模块
    │   │   ├── song_url.js             # 歌曲播放地址
    │   │   ├── search.js               # 搜索
    │   │   ├── login_cellphone.js      # 手机号登录
    │   │   ├── lyric.js                # 歌词
    │   │   ├── playlist_detail.js      # 歌单详情
    │   │   └── ...                     # 等等
    │   ├── util/                       # 工具库
    │   │   ├── request.js              # HTTP 请求封装
    │   │   ├── crypto.js               # 加密算法
    │   │   ├── apicache.js             # 缓存中间件
    │   │   ├── client-sign.js          # 客户端签名
    │   │   └── ...
    │   ├── public/                     # 静态页面（登录页、文档等）
    │   ├── data/                       # 数据文件（设备ID、IP 段等）
    │   ├── test/                       # 单元测试
    │   └── .github/workflows/          # CI/CD 自动构建
    └── ...
```

---

## 4. 前置准备

### 4.1 后端环境要求

| 依赖 | 版本要求 | 说明 |
|------|----------|------|
| **Node.js** | ≥ 18（推荐 22+） | 必须 |
| **pnpm** | 最新稳定版 | 推荐（也支持 npm/yarn） |
| **Git** | 任意版本 | 必须 |

### 4.2 前端环境要求

无特殊要求。music-app 是纯静态项目，使用 CDN 加载第三方库（Font Awesome、ColorThief）。

仅需确保浏览器支持：
- ES2017+ （async/await、const/let）
- CSS Grid / Flexbox
- `<audio>` 元素

### 4.3 服务器要求（自行部署时）

最低配置：**1 核 1G 内存，10G 硬盘**

推荐配置：**2 核 2G 内存**（如需部署桌面客户端等衍生项目）

---

## 5. 后端部署 (API 服务)

提供四种部署方式，从易到难排列。

### 5.1 Docker 部署（推荐 ⭐）

最简单快捷，无需安装 Node.js。

```bash
# 1. 拉取镜像
docker pull moefurina/ncm-api:latest

# 2. 启动服务
docker run -d \
  -p 3000:3000 \
  --name ncm-api \
  moefurina/ncm-api:latest
```

> ⚠️ 由于底层 `request` 库会检测代理相关环境变量，如遇到连接问题，可清空代理变量：
>
> ```bash
> docker run -d -p 3000:3000 \
>   -e http_proxy= -e https_proxy= \
>   -e HTTP_PROXY= -e HTTPS_PROXY= \
>   -e no_proxy= -e NO_PROXY= \
>   moefurina/ncm-api:latest
> ```

**验证服务状态**：

```bash
curl http://localhost:3000/status
```

看到返回数据即表示启动成功。

**自行构建镜像**：

```bash
git clone https://github.com/NeteaseCloudMusicApiEnhanced/api-enhanced.git
cd api-enhanced
docker build -t ncm-api .
docker run -d -p 3000:3000 ncm-api
```

---

### 5.2 Node.js 手动部署

适合需要自定义修改或本地开发。

```bash
# 1. 克隆项目
git clone https://github.com/NeteaseCloudMusicApiEnhanced/api-enhanced.git
cd api-enhanced

# 2. 安装依赖（推荐 pnpm）
pnpm install

# 3. 启动
node app.js
```

默认监听 `http://localhost:3000`。指定端口：

```bash
PORT=4000 node app.js
```

**后台运行**（使用 PM2）：

```bash
npm install -g pm2
pm2 start app.js --name ncm-api
pm2 save
pm2 startup
```

---

### 5.3 Vercel 一键部署

适合个人快速上线，免费额度足够日常使用。

1. **Fork** 项目仓库到你的 GitHub 账号
2. 前往 [Vercel](https://vercel.com) → New Project
3. 选择 Fork 的仓库，点击 **Import**
4. 默认配置，点击 **Deploy**
5. 等待部署完成，访问分配的域名即可

> Vercel 部署后，建议将敏感配置（如 Cookie、代理 URL）通过 Vercel 环境变量面板添加。

---

### 5.4 腾讯云 Serverless 部署

适合国内低延迟访问。

1. Fork 项目到 GitHub
2. 腾讯云 Serverless 控制台 → 新建 Web 应用
3. 选择 Express 框架
4. 代码仓库选择 Fork 的项目
5. 启动文件填写：
   ```
   #!/bin/bash
   export PORT=9000
   /var/lang/node16/bin/node app.js
   ```
6. 部署完成后访问 API 网关分配的 URL

---

## 6. 前端部署与对接

### 6.1 连接后端 API

前端通过 `music-app/yinyueapi/api.js` 中的 `API_BASE` 变量来指定后端地址。

**第 18 行附近**：

```javascript
const API_BASE = 'https://wyapi.hzxq.asia';  // ← 改成你自己的后端地址
```

**修改为**：

```javascript
const API_BASE = 'http://localhost:3000';  // 本地测试
```

或部署后的公网地址：

```javascript
const API_BASE = 'https://your-api-domain.com';  // 生产环境
```

### 6.2 本地运行前端

纯静态项目，任选一种方式：

**方式 A：Python 简易 HTTP 服务器**

```bash
cd music-app
python3 -m http.server 8080
# 然后浏览器访问 http://localhost:8080
```

**方式 B：Node.js 的 serve**

```bash
npx serve music-app -l 8080
```

**方式 C：直接用浏览器打开**

双击 `index.html` 即可。⚠️ 但部分浏览器对 `file://` 协议的 CORS 请求有限制，推荐使用 HTTP 服务器。

### 6.3 生产环境部署

纯静态资源，可以部署到任何静态托管服务：

| 平台 | 步骤 |
|------|------|
| **Vercel** | 导入项目根目录 → Framework 选 "Other" → Deploy |
| **Netlify** | 拖拽 music-app 文件夹上传即部署 |
| **GitHub Pages** | Settings → Pages → Source 选 main/root |
| **自有服务器** | 直接用 Nginx/Apache 托管 |
| **OSS + CDN** | 上传至阿里云 OSS / AWS S3 并配置 CDN |

### 6.4 Nginx 配置参考

```nginx
server {
    listen 80;
    server_name music.yourdomain.com;

    root /var/www/music-app;
    index index.html;

    # 开启 gzip
    gzip on;
    gzip_types text/plain text/css application/javascript application/json;

    # 静态资源缓存
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff2?)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 7. 环境变量说明

后端服务支持以下环境变量，按需配置：

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `PORT` | `3000` | 服务端口 |
| `CORS_ALLOW_ORIGIN` | `*` | CORS 跨域允许的域名（如 `https://a.com,https://b.com`） |
| `ENABLE_PROXY` | `false` | 是否启用反向代理 |
| `PROXY_URL` | — | 代理地址（仅 `ENABLE_PROXY=true` 时生效） |
| `ENABLE_RANDOM_CN_IP` | `false` | 启用随机中国 IP |
| `ENABLE_GENERAL_UNBLOCK` | `true` | 全局解灰（尝试自动解锁无版权歌曲） |
| `ENABLE_FLAC` | `true` | 启用 FLAC 无损音质 |
| `SELECT_MAX_BR` | `false` | 选择最高码率音质 |
| `FOLLOW_SOURCE_ORDER` | `true` | 严格按照音源列表顺序匹配 |
| `NETEASE_COOKIE` | — | 网易云 Cookie（敏感，推荐环境变量而非硬编码） |

**配置方式**：

```bash
# Linux/Mac
export PORT=4000
export ENABLE_FLAC=true
node app.js

# Windows CMD
set PORT=4000 && node app.js

# Windows PowerShell
$env:PORT=4000; node app.js

# Docker
docker run -d -p 3000:3000 -e PORT=3000 -e ENABLE_FLAC=true moefurina/ncm-api

# .env 文件（项目根目录创建）
# PORT=4000
# ENABLE_FLAC=true
```

---

## 8. API 接口速查

以下是最常用接口一览。完整接口列表请参考 [在线文档](https://neteasecloudmusicapienhanced.js.org/)。

### 8.1 基础接口

| 接口 | 路由 | 说明 |
|------|------|------|
| 首页 Banner | `GET /banner` | 轮播图数据 |
| 首页发现 | `GET /homepage/block/page` | 推荐内容区块 |
| 热门话题 | `GET /hot/topic` | 热门话题列表 |

### 8.2 搜索接口

| 接口 | 路由 | 说明 |
|------|------|------|
| 搜索 | `GET /search` | 快速搜索（`s=关键词&type=1`） |
| 搜索热词 | `GET /search/hot` | 热搜关键词 |
| 搜索建议 | `GET /search/suggest` | 搜索自动补全 |
| 单曲搜索 | `GET /cloudsearch` | 搜索歌曲（`s=关键词&limit=20`） |

### 8.3 歌曲接口

| 接口 | 路由 | 说明 |
|------|------|------|
| 歌曲详情 | `GET /song/detail` | `?ids=歌曲ID` |
| 播放地址 | `GET /song/url/v1` | `?id=歌曲ID&level=exhigh` |
| 歌词 | `GET /lyric` | `?id=歌曲ID` |
| 歌词(新) | `GET /lyric/new` | `?id=歌曲ID` |
| 歌曲评论 | `GET /comment/music` | `?id=歌曲ID` |
| 相似歌曲 | `GET /simi/song` | `?id=歌曲ID` |
| 最近听过 | `GET /record/recent/song` | 播放历史 |

**音质等级参数** `level`（从高到低）：

```
jymaster → sky → jyeffect → hires → lossless → exhigh → higher → standard
```

### 8.4 歌单接口

| 接口 | 路由 | 说明 |
|------|------|------|
| 推荐歌单 | `GET /personalized` | `?limit=10` |
| 歌单详情 | `GET /playlist/detail` | `?id=歌单ID` |
| 歌单歌曲 | `GET /playlist/track/all` | `?id=歌单ID` |
| 每日推荐 | `GET /recommend/songs` | 需要登录 |
| 歌单分类 | `GET /playlist/catlist` | 全部分类标签 |

### 8.5 用户接口

| 接口 | 路由 | 说明 |
|------|------|------|
| 手机号登录 | `POST /login/cellphone` | `?phone=xxx&password=xxx` |
| 二维码 Key | `GET /login/qr/key` | 第一步 |
| 二维码图片 | `GET /login/qr/create` | 第二步（用上一步返回的 key） |
| 扫码状态 | `GET /login/qr/check` | 第三步（轮询） |
| 登录状态 | `GET /login/status` | 检查是否已登录 |
| 用户信息 | `GET /user/detail` | `?uid=用户ID` |
| 用户歌单 | `GET /user/playlist` | `?uid=用户ID` |
| 退出登录 | `GET /logout` | 退出当前登录 |

### 8.6 调用示例

```bash
# 获取歌曲播放地址
curl "http://localhost:3000/song/url/v1?id=123456&level=exhigh"

# 搜索歌曲
curl "http://localhost:3000/cloudsearch?s=海阔天空&limit=10"

# 获取歌词
curl "http://localhost:3000/lyric?id=123456"

# 获取推荐歌单
curl "http://localhost:3000/personalized?limit=10"
```

### 8.7 Node.js 程序化调用

作为 npm 包直接调用：

```javascript
import { song_url_v1, search } from '@neteasecloudmusicapienhanced/api'

// 获取播放地址
const result = await song_url_v1({ id: '123456', level: 'exhigh' })
console.log(result.data[0].url)

// 搜索歌曲
const searchResult = await search({ keywords: '海阔天空', type: 1 })
console.log(searchResult.result.songs)
```

安装：

```bash
npm install @neteasecloudmusicapienhanced/api
```

---

## 9. 开发指南

### 9.1 本地联调开发

```bash
# Terminal 1: 启动后端
cd api-enhanced
pnpm install
node app.js
# → 监听 http://localhost:3000

# Terminal 2: 启动前端
cd music-app
python3 -m http.server 8080
# → 访问 http://localhost:8080

# Terminal 3: 编辑代码即可热刷新（纯前端无需构建）
```

### 9.2 前端代码架构

```
index.html           ← 页面骨架（视图容器 + 组件）
    │
    ├── api.js       ← API 层：纯封装，无 UI 逻辑
    │   ├── API_BASE               （后端地址）
    │   ├── request()              （GET 请求封装）
    │   ├── postRequest()          （POST 请求封装）
    │   ├── search()               （搜索歌曲）
    │   ├── getSongUrl()           （获取播放 URL）
    │   ├── getSongDetail()        （获取歌曲详情）
    │   ├── getLyric()             （获取歌词）
    │   ├── getPlaylistDetail()    （获取歌单详情）
    │   └── ...                    （更多接口方法）
    │
    └── apii.js      ← 逻辑层：状态管理 + UI 交互
        ├── state                  （全局状态对象）
        ├── audio                  （audio 元素引用）
        ├── showToast()            （提示工具）
        ├── initCardHoverEffects() （交互初始化）
        ├── switchView()           （页面切换）
        ├── doSearch()             （搜索逻辑）
        ├── togglePlay()           （播放/暂停）
        ├── playNext() / playPrev()（上/下一首）
        ├── togglePlayMode()       （播放模式切换）
        ├── loadDailySongs()       （每日推荐）
        ├── showLoginModal()       （登录弹窗）
        ├── handleLogout()         （退出登录）
        ├── toggleLikeCurrentSong()（喜欢/取消喜欢）
        └── ...                    （更多业务方法）
```

### 9.3 添加新的 API 调用

在 `api.js` 中新增方法：

```javascript
// 示例：获取 MV 播放地址
async getMvUrl(id) {
    return this.request('/mv/url', { id });
}
```

在 `apii.js` 中新增调用逻辑：

```javascript
// 在业务逻辑中调用
const result = await api.getMvUrl(mvId);
console.log(result.data.url);
```

### 9.4 添加新的音质等级

修改 `api.js` 中的 `QUALITY_LEVELS` 数组：

```javascript
const QUALITY_LEVELS = ['jymaster', 'sky', 'jyeffect', 'hires', 'lossless', 'exhigh', 'higher', 'standard'];
```

### 9.5 缓存机制

前端内置了简单的内存缓存系统（`dataCache`），避免重复请求：

```javascript
window.dataCache = {
    store: {},
    get: function(type, key) { ... },   // 读取缓存
    set: function(type, key, data, ttl) { ... }  // 写入缓存（ttl 毫秒）
};
```

缓存类型定义在 `window.CacheTypes`：

```javascript
window.CacheTypes = {
    LYRICS: 'lyrics',       // 歌词缓存
    PLAYLIST: 'playlist',   // 歌单缓存
    COMMENTS: 'comments',   // 评论缓存
    LIKED_SONGS: 'liked_songs'  // 我喜欢的音乐缓存
};
```

---

## 10. 开源贡献指南

### 10.1 如何贡献代码

1. **Fork** 本仓库
2. 创建你的 Feature 分支：`git checkout -b feat/your-feature`
3. 提交你的修改：`git commit -am 'feat: 添加新功能'`
4. 推送到分支：`git push origin feat/your-feature`
5. 提交 **Pull Request** 到 `main` 分支

### 10.2 分支命名规范

| 前缀 | 用途 | 示例 |
|------|------|------|
| `feat/` | 新功能 | `feat/share-resource` |
| `fix/` | Bug 修复 | `fix/login-error` |
| `docs/` | 文档更新 | `docs/setup-guide` |
| `refactor/` | 重构代码 | `refactor/api-cache` |
| `test/` | 测试相关 | `test/song-url` |
| `chore/` | 杂项维护 | `chore/update-deps` |

### 10.3 代码风格

后端使用 ESLint + Prettier，提交前请确保：

```bash
pnpm lint        # 检查代码规范
pnpm lint-fix    # 自动修复
pnpm test        # 运行单元测试
```

### 10.4 提交新 API 接口

如果要贡献新的网易云音乐接口，请参考 [如何贡献新接口](https://www.focalors.ltd/post/how-to-contribute-ncm-api)，大致步骤：

1. 在 `module/` 目录创建 `module_name.js` 文件
2. 导出一个接收 `query` 参数的异步函数
3. 使用 `request()` 工具转发到网易云接口
4. 可选：在 `module_example/` 添加使用示例
5. 添加单元测试到 `test/` 目录

### 10.5 Issue 提交规范

- **Bug 报告**：包含复现步骤、预期结果、实际结果、环境信息
- **功能请求**：描述应用场景和预期效果
- **不要**提交涉及破解付费功能的 Issue

---

## 11. 常见问题排查

### Q1：请求返回 400 或连接失败

**可能原因**：

- 后端服务未启动 → 检查端口是否监听：`curl http://localhost:3000/status`
- 跨域问题 → 确认后端 `CORS_ALLOW_ORIGIN` 包含前端域名
- 防火墙拦截 → 临时关闭防火墙测试
- Docker 代理变量干扰 → 清空 `http_proxy` 等环境变量

### Q2：歌曲无法播放（URL 为空）

**解决方案**：

- 网易云音乐 VIP 歌曲需要登录后才能获取播放 URL
- 确认已启用解灰功能：`ENABLE_GENERAL_UNBLOCK=true`
- 尝试降低音质等级：`level=standard`
- 检查是否需要配置代理（`ENABLE_PROXY=true` + `PROXY_URL`）

### Q3：前端页面空白或白屏

**排查步骤**：

1. 浏览器 F12 打开开发者工具 → Console 查看报错
2. 常见原因：`api.js` 加载失败、CORS 被拦截、CDN 字体库加载失败
3. 测试纯后端：直接用 `curl` 测试 API 是否正常

### Q4：扫码登录后状态丢失

**原因**：Cookie 存储在服务端，重启后清空

**解决方案**：

- 生产环境配置 `NETEASE_COOKIE` 环境变量持久化
- 或在 `data/` 目录持久化 Cookie 文件

### Q5：在小程序或 Electron 中使用

**问题**：微信小程序需要合法域名备案

**解决方案**：

- 后端必须使用 HTTPS 及已备案域名
- 小程序后台配置服务器域名白名单
- Electron 环境下可直接使用，无跨域限制

---

## 📄 License

本项目基于 [MIT License](https://opensource.org/licenses/MIT) 开源协议。

> ⚠️ 免责声明：本项目仅供学习交流使用，请勿用于任何商业用途。

---

## 🔗 相关资源

- **后端项目仓库**：https://github.com/NeteaseCloudMusicApiEnhanced/api-enhanced
- **后端在线文档**：https://neteasecloudmusicapienhanced.js.org/
- **NPM 包地址**：https://www.npmjs.com/package/@neteasecloudmusicapienhanced/api
- **Docker 镜像**：https://hub.docker.com/r/moefurina/ncm-api
- **原版项目仓库**：https://github.com/binaryify/NeteaseCloudMusicApi
- **贡献指南**：https://www.focalors.ltd/post/how-to-contribute-ncm-api

---

> 文档版本：v1.0 ｜ 最后更新：2026 年 7 月
