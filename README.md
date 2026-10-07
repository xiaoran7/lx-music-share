# LX Music Share (落雪音乐私有云分享服务)

`lx-music-share` 是为“落雪全家桶”（LX Music Desktop / Mobile）量身打造的私有云音乐分享服务与单页播放系统。支持跨设备、微信内置浏览器与移动端高保真还原播放，具备单曲异步持久化、沉浸式大视窗歌词、听友留言互动与自适应过期物理清理（TTL）能力。

---

## 🌟 致敬原作者与开源生态

落雪音乐生态由原作者 **[lyswhut (落雪无痕)](https://github.com/lyswhut)** 首创开发，其极简优雅的设计与自由可控的理念赢得了开源社区的广泛热爱。

- **原作者 GitHub**：[lyswhut](https://github.com/lyswhut)
- **官方项目列表**：[lx-music-desktop](https://github.com/lyswhut/lx-music-desktop) | [lx-music-mobile](https://github.com/lyswhut/lx-music-mobile) | [lx-music-sync-server](https://github.com/lyswhut/lx-music-sync-server)
- **官方文档中心**：[LX Music Document](https://lyswhut.github.io/lx-music-doc/)

本项目作为落雪全家桶生态的独立服务端组件，旨在补全客户端“向好友/微信/移动端随时分享心爱歌曲”的最后一公里，遵循开源社区规范，所有代码完全开源透明。

---

## 🎧 落雪全家桶 (LX Music Suite) 生态架构

本项目是“落雪全家桶”生态中的私有分享服务中枢，与双端客户端及数据同步服务紧密协同：

```text
               ┌───────────────────────────────┐
               │    落雪全家桶 (LX Music Suite)   │
               └───────────────┬───────────────┘
                               │
       ┌───────────────────────┴───────────────────────┐
       ▼                                               ▼
┌──────────────┐                               ┌──────────────┐
│  客户端集群   │                               │  服务端中枢   │
└──────┬───────┘                               └──────┬───────┘
       │                                               │
       ├─► LX Music Desktop (桌面端客户端)               ├─► LX Music Share Server (私有分享服务，本仓库)
       │   - 平台：Windows / macOS / Linux             │   - 架构：FastAPI + APlayer + Nginx 206
       │   - 特性：右键分享歌曲、TTL 自主选择、浮窗交互   │   - 功能：单曲转存、沉浸单页、留言板、TTL 自动清理
       │                                               │
       └─► LX Music Mobile (移动端客户端)               └─► LX Music Sync Server (多端数据同步服务)
           - 平台：Android (React Native)              │   - 协议：WebSocket 双向实时同步
           - 特性：后台保活、一键生成短链、微信/系统分享  │   - 功能：歌单/黑名单同步 + CRDT 听歌统计
```

---

## 🌟 核心特性

1. **1:1 原生级 Web 播放详情单页 (`templates/player.html`)**：
   - **大尺寸沉浸封面**：采用 `min(84vw, 42vh)` 自适应黄金比例大封面，带柔和光晕阴影。
   - **视口锁定与防滚动**：强制 100dvh 固定定位与安全区域适配（`env(safe-area-inset-bottom)`），彻底消除页面晃动与多余滚动条。
   - **平滑过渡动效**：封面与全屏歌词支持 3D 缩放淡入淡出（`scale(0.88)` / `cubic-bezier(0.25, 1, 0.5, 1)`），双向极速切换。
   - **手势防误触识别**：滑动歌词与点击切换双重检测，杜绝误触 Seek 跳转。
   - **收敛式音量调节**：底部 5 键布局集成独立音量悬浮气泡卡片，脱离正常文档流，实现播放器自身增益（0.0 ~ 1.0）与物理硬件音量双重控制。
   - **微信浮窗指引**：顶部置顶指引“轻触右上角 ··· 选择『浮窗』即可后台畅听”。
   - **听友互动留言**：内置轻量化抽屉式留言面板，支持听友随听随评。

2. **高性能 FastAPI 异步后端**：
   - **流式持久化**：流式异步下载远程音频（支持 100MB 限制）、高清封面与 LRC 歌词，内置多平台 UA 伪装。
   - **206 Partial Content**：支持标准 HTML5 音频断点续传与精准拖拽 Seek。
   - **TTL 自动清理机制**：支持自定义有效期限（1天、3天、7天、30天、永久），每小时守护协程自动物理销毁过期音频与数据。
   - **轻量 SQLite 存储**：免外部依赖，单文件自给自足。

---

## 🚀 保姆级部署教程

推荐在具备公网 IP 的云服务器（VPS）或 NAS 上部署，搭配域名与 Nginx 反向代理。

### 1. 环境准备与依赖安装

```bash
# 1. 克隆代码至部署目录
sudo mkdir -p /opt/lx-music-share
cd /opt/lx-music-share
git clone https://github.com/xiaoran7/lx-music-share.git .

# 2. 创建 Python 虚拟环境 (建议 Python 3.10+)
python3 -m venv venv
source venv/bin/activate

# 3. 安装依赖包
pip install -r requirements.txt
```

### 2. 环境变量配置

支持通过环境变量调整服务行为：
- `BASE_URL`：外部访问域名（例如 `https://music.example.com`，末尾不带斜杠）。
- `DATA_DIR`：数据与媒体文件的持久化路径（默认 `./data`）。
- `SHARE_TOKEN`：（可选）调用 `/api/share` 时的安全认证令牌；留空表示公开允许分享。
- `DEFAULT_TTL_DAYS`：未指定有效期限时的默认保存天数（默认 `7` 天）。

### 3. Systemd 守护服务配置

将仓库内的 `lx-music-share.service` 部署到系统守护：

```bash
sudo cp lx-music-share.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now lx-music-share
```

### 4. Nginx 反向代理配置 (关键：Range 206 静态媒体直出)

> **避坑提醒**：iOS Safari 与微信内置浏览器对音频流播放要求极高，必须支持 HTTP 206 Partial Content 分片。
> 通过 Nginx 直接拦截 `/media/` 静态目录，利用 Nginx 原生模块处理媒体直出，性能极佳且完美兼容微信手势播放与进度拖动！

参考配置 `nginx-music-share.conf`：

```nginx
# 80 端口自动重定向到 HTTPS
server {
    listen 80;
    server_name music.example.com;
    return 301 https://$host$request_uri;
}

# 443 端口：HTTPS 反向代理与静态媒体 Range 直出
server {
    listen 443 ssl http2;
    server_name music.example.com;

    ssl_certificate     /etc/letsencrypt/live/music.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/music.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 客户端上传歌曲限制 (单曲音频与媒体包最大允许 100M)
    client_max_body_size 100M;

    # 1. 媒体资源直出 (Nginx 原生极速 Range 206 分片响应，适配 iOS Safari 与微信)
    location /media/ {
        alias /opt/lx-music-share/data/media/;
        expires 7d;
        add_header Accept-Ranges bytes;
        add_header Access-Control-Allow-Origin *;
        add_header Cache-Control "public, max-age=604800";
        access_log off;
        error_page 404 = @fastapi;
    }

    location @fastapi {
        proxy_pass http://127.0.0.1:8920;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 2. 动态 API 与单页路由反代到 FastAPI
    location / {
        proxy_pass http://127.0.0.1:8920;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 60s;
        proxy_read_timeout 120s;
        proxy_send_timeout 120s;
    }
}
```

启用配置并生效：

```bash
sudo ln -s /opt/lx-music-share/nginx-music-share.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## 📱 客户端配置指引

1. **桌面端 (LX Music Desktop)**：
   - 打开“设置” -> “基本设置” -> “分享方式”选择“私有分享服务”；
   - 填入您的服务地址（如 `https://music.example.com`）与 Token（若有）；
   - 在歌曲列表右键任意歌曲，点击“分享歌曲”，弹窗选择有效期后即可生成专属试听单页。
2. **移动端 (LX Music Mobile)**：
   - 进入“设置” -> “基本设置” -> “分享方式”选择“推送到私有云”；
   - 填入相同的服务地址与 Token；
   - 歌曲菜单点击“分享歌曲”，即刻唤起分享浮窗，一键复制链接或分享到微信。

---

## 📜 许可协议与免责声明

1. 本项目基于 [MIT License](LICENSE) 协议开源。
2. 本项目不提供、不储存任何受版权保护的音频或媒体资源。播放与分享功能所用音频直链均由用户本地客户端解析推送，内容的合法性与版权归属由相应原始来源及使用者自行承担，请自觉尊重版权并支持正版音乐。
3. 本项目仅供个人技术研究、全栈工程学习与交流使用，禁止用于任何商业营利场景。
