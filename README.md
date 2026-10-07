# LX Music Share (洛雪音乐私有云分享服务)

`lx-music-share` 是为洛雪音乐（LX Music Mobile / Desktop）量身打造的私有云音乐分享服务与单页播放系统。支持跨设备、微信内置浏览器与移动端高保真还原播放。

---

## 🌟 核心特性

1. **1:1 原生级 Web 播放详情单页 (`player.html`)**：
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

## 🚀 部署指引

### 1. 环境准备
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. 环境变量配置
- `BASE_URL`: 分享外部访问域名（例如 `https://music.tannerlab.cn`）
- `DATA_DIR`: 数据与媒体存储路径（默认 `./data`）
- `SHARE_TOKEN`: （可选）调用 `/api/share` 时的安全认证令牌
- `DEFAULT_TTL_DAYS`: 默认过期天数（默认 `7` 天）

### 3. Systemd 守护服务
将 `lx-music-share.service` 复制到 `/etc/systemd/system/` 并启动：
```bash
sudo cp lx-music-share.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now lx-music-share
```

### 4. Nginx 反向代理
将 `music-tannerlab-cn.conf` 引入 Nginx 配置：
```bash
sudo ln -s $(pwd)/music-tannerlab-cn.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## 📜 许可证
MIT License
