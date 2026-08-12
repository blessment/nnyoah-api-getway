# nnyoah-api

一个基于php的轻量级 AI 大模型 API 中转站，**完全兼容 OpenAI Chat Completions API**，支持多模型路由、流式响应、多渠道自动重试、Token 计费与健康检测。**纯 PHP + MySQL 实现，零第三方框架依赖，可在普通 PHP 虚拟主机（PHP 7.3+，无额外扩展）上运行。**

## 特性

- 🔌 **OpenAI 兼容**：`base_url` + `api_key` 即可接入，支持流式（SSE）与非流式
- 🔀 **多模型路由**：请求自动选择健康渠道，失败自动重试其他渠道
- 🔑 **自有 Key 体系**：`sk-nnyoah-xxx` 格式，可创建/禁用/删除，每个 Key 可设余额上限（按累计消费实时校验，尽力而为）
- 💰 **透明计费**：每个模型独立单价，Token 消耗与费用逐笔记录
- ❤️ **健康检测**：定时/手动检测各渠道可用性，面板实时显示健康状态与延迟
- 🧑‍💻 **用户端**：注册（仅用户名+密码）、Key 管理、调用日志、用量图表
- 🛠️ **管理后台**：仪表盘概览、Key 管理、渠道管理、模型管理、调用日志、用户管理
- 🖥️ **虚拟主机友好**：PHP 7.3+ / 8.x，PDO，HTTP 客户端 curl / stream / fsockopen 自适应

## 快速开始

### 1. 上传

将整个项目上传到虚拟主机（或本地 PHP 环境）。

> 若你的虚拟主机支持将网站根目录指向 `public/`，则是标准方式；否则请开启 Apache 的 `mod_rewrite`（大多数虚拟主机默认开启），项目根的 `.htaccess` 会自动把请求转发到 `public/index.php`。

### 2. 安装

浏览器访问 `http://你的域名/install.php`，填写：

- 数据库信息（虚拟主机提供的 MySQL 地址、端口、库名、账号、密码）
- 管理员密码（≥6 位）
- 安全盐（随机长字符串，安装后不可改）
- 上游 API 地址（如 `https://api.openai.com/v1`）
- 上游 API Key（可稍后在后台添加）

安装完成后**立即删除 `install.php`**。

### 3. 配置渠道与模型

进入「管理后台」→「渠道」，添加上游 API Key（一个上游可配多个 Key，自动路由与故障切换）；进入「模型」配置对外模型名、单价与渠道绑定。

### 4. 客户端接入

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://你的域名/v1",
    api_key="sk-nnyoah-你的Key",
)
resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "你好"}],
)
print(resp.choices[0].message.content)
```

## 鉴权方式

`config.php` 中 `auth_mode` 两种模式：

| 模式 | 用户调用方式 | 场景 |
|------|-------------|------|
| `key`（默认） | `Authorization: Bearer sk-nnyoah-xxx` | 用户在控制台创建 Key |
| `proxy` | `Authorization: Basic base64(账号:密码)` | 上游本身就是 OpenAI，用户账号密码即上游 Key，站点只做透明代理 |

## 目录结构

```
├── public/            # 公网入口（网页 + API 路由）
│   └── assets/        # 前端资源
├── app/
│   ├── api/           # OpenAI 兼容 API
│   ├── actions/       # 用户端动作
│   ├── admin/         # 管理后台页面与动作
│   ├── pages/         # 用户端页面
│   └── core/          # 核心：DB / Gateway / HttpClient / Layout
├── cron/              # 定时任务（渠道健康检测）
├── data/              # 安装锁等
├── logs/              # 运行日志
├── install.php        # 安装引导（装完删除）
├── config.example.php # 配置模板（安装会自动生成 config.php）
├── config.php         # 真实配置（安装时自动生成，勿提交）
└── .htaccess          # Apache 重写规则
```

## 定时检测渠道健康

虚拟主机支持 Cron 时：

```
*/5 * * * * php /path/to/nnyoah-api/cron/check_health.php
```

或使用 Web 方式（URL 见「管理后台 → 设置」），配合第三方 uptime 服务定时访问。

## 安全说明

- 所有 SQL 均使用预处理语句（PDO），无注入风险
- 密码使用 `bcrypt` 加盐哈希
- 会话开启 `httponly` / `samesite`，登录后 `session_regenerate_id`
- 管理后台独立登录，未登录自动跳转
- 安装后请删除 `install.php`，并修改 `config.php` 中的 `secret`

## 环境要求

- PHP 7.3+（建议 8.x），需 `pdo_mysql`（绝大多数虚拟主机自带）
- MySQL 5.5+（InnoDB）
- HTTP 能力：`curl` 扩展 或 `allow_url_fopen=On` 或 `fsockopen`（三者任一即可，自动降级）
- 可选扩展：`mbstring`（用户名校验）、`openssl`（随机令牌）

> 说明：代码使用 `??`（7.0+）、`session_set_cookie_params` 数组形式（7.3+）等语法，低于 PHP 7.3 无法运行。

## 免责声明

本项目仅用于学习与合法用途。请遵守上游服务商的使用条款，勿用于滥发、爬取等违规行为。
