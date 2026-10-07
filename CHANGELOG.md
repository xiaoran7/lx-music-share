# 更新日志 (CHANGELOG)

本项目遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/) 格式，版本号遵循 [语义化版本 2.0.0](https://semver.org/lang/zh-CN/)。

---

## [1.0.0] - 2026-10-07

### 新增 (Added)
- **独立工程化沉淀**：从 `lx-music-mobile` 中剥离独立出 `lx-music-share` 项目，建立专属 Gitea 仓库与自动化部署体系。
- **FastAPI 服务端**：
  - `/api/share`：接收客户端分享上报，异步持久化音频（100MB 上限带防盗链伪装与流式写入）、高清封面与 LRC 歌词。
  - `/s/{code}`：自适应移动端与桌面端分享单页，支持动态有效时间计算、访问计数与听友互动留言。
  - `/api/comments/{code}`：支持听友发表和拉取专属评论。
  - `/media/{code}/*`：支持 206 Partial Content 分片音频流分发，自带降级重定向回退机制。
  - TTL 自动清理守护协程：每小时扫描并物理销毁过期音频与元数据。
- **1:1 原生级 Web 播放详情单页 (`player.html`)**：
  - 100dvh 固定全屏视口与安全边距锁定，彻底根治移动端外滚动条与上下弹性晃动。
  - 硬件加速 3D 缩放淡入淡出（`cubic-bezier(0.25, 1, 0.5, 1)`），实现封面与歌词丝滑双向切换。
  - 歌词滚动防误触手势识别，禁止点击歌词误 Seek 跳转。
  - 底栏 5 键布局（LRC | 收藏 | 悬浮音量 | 循环模式 | 听友留言），音量调节采用绝对定位毛玻璃浮层卡片，彻底根治控件堆叠空洞缺陷。
  - 微信内置浏览器置顶后台浮窗播放指引提示。
- **生产配置**：包含 Nginx 反向代理配置 `music-tannerlab-cn.conf` 及 Systemd 守护单元 `lx-music-share.service`。
