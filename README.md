# 🤖 Telegram 双向消息机器人（防诈骗增强版）
是一个部署在 Cloudflare Workers 上的 Telegram 机器人，实现用户与管理员双向私聊，并内置 2025 主流诈骗类型检测 + KV 可扩展防诈骗数据库，同时支持整点时间推送（适配免费定时工具）。

## ✨ 功能特点
**双向消息转发
    普通用户发送的任何消息都会自动转发给管理员
    管理员回复转发的消息，机器人将回复内容复制给对应用户

整点时间提醒（免费版适配）
    通过 URL 参数验证的方式调用，可使用 EasyCron 等免费定时工具触发，向管理员推送当前北京时间

防诈骗智能检测
    内置 20+ 类 2025 主流诈骗正则匹配（刷单、公检法、杀猪盘、AI 换脸等）
    自动检测消息中的手机号、银行卡、可疑链接、二维码等敏感信息
    结合 KV 数据库关键词匹配，管理员可动态添加/查询诈骗数据

诈骗数据库管理（管理员专用）
    添加/查询诈骗关键词、手机号、网址等信息
    查看数据库统计
    批量导入/初始化基础数据

命令菜单支持
    通过 /setcommands 接口动态设置机器人命令列表，使用更方便**

## 🚀 快速开始
### 1. 准备工作
    Cloudflare 账号 (https://dash.cloudflare.com)
    Telegram Bot Token (https://t.me/BotFather)（创建机器人后获得）
    管理员 Telegram User ID（可通过 https://t.me/userinfobot 获取）
    自定义密钥（任意字符串，用于 URL 参数验证，例如 mysecret2025）

### 2. 创建 Cloudflare Worker
    登录 Cloudflare Dashboard，进入 Workers 和 Pages。
    点击 创建应用程序 → 创建 Worker。
    将本仓库的 worker.js 代码复制到编辑器中（或直接上传）。
    点击 保存并部署。

### 3. 配置环境变量
   在 Worker 详情页，进入 设置 → 变量，添加以下环境变量：
      | 变量名 | 说明 | 示例 |
   |--------|------|------|
   | `ENV_BOT_TOKEN` | Telegram 机器人 Token | `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11` |
   | `ENV_BOT_SECRET` | 自定义密钥（用于验证 Webhook 请求和访问 `/setcommands`） | 任意随机字符串，如 `my_secret_2025` |
   | `ENV_ADMIN_UID` | 管理员 Telegram ID（数字字符串） | `123456789` |

### 4. 绑定 KV 命名空间
    机器人需要 KV 存储来保存诈骗数据、消息映射关系等。
    在 Worker 详情页，进入 设置 → KV 命名空间绑定。
    点击 添加绑定，变量名称必须为 nfd（代码中固定使用此名称）。
    选择或创建一个 KV 命名空间（例如 tg-bot-kv）。
    保存绑定。

5. 设置 Webhook
    你需要让 Telegram 将消息更新发送到你的 Worker 地址。
    访问以下 URL（在浏览器中打开或使用 curl）：
    https://你的worker域名/registerWebhook
    如果配置正确，会返回 ✅ Webhook注册成功。
    注意：注册 Webhook 时会自动带上 secret_token 参数，Worker 会验证该 token 以防止恶意请求。
