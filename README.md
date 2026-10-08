<!-- markdownlint-disable-file MD033 MD045 -->
# Cloudflare 自托管临时邮箱

基于 Cloudflare Workers、D1 和 Email Routing 的自托管临时邮箱服务，提供自定义域名地址管理、邮件收发和附件查看，可通过 Web 界面、SMTP/IMAP 代理及 Agent 接口访问邮箱。

本仓库源自 [dreamhunter2333/cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email)，保留上游提交历史及 [MIT 许可证](LICENSE)。下述核心功能来自上游；本分支的介绍整理记录在 [更新日志](CHANGELOG.md) 中。

[中文](README.md) · [English](README_EN.md) · [上游部署文档](https://temp-mail-docs.awsl.uk)

## 功能概览

| 场景 | 实现与能力 |
| --- | --- |
| 自定义域名收件 | Email Routing 将来信交给 Worker 处理，邮件保存在 D1，支持地址创建、密码和生命周期管理。 |
| 邮件阅读与附件 | Web 收件箱支持邮件检索、正文和附件查看；前端优先使用 Rust WASM 解析邮件，并以 postal-mime 回退。 |
| 按需发信 | 支持 Resend、SMTP 等发送方式，需配置对应服务、域名和发送额度。 |
| 用户与站点访问 | 支持用户账号、OAuth2、Passkey、地址凭证、站点访问密码和域名权限配置。 |
| 通知与集成 | 支持邮件转发、Webhook、Telegram 推送及 SMTP/IMAP 代理。 |
| Agent 访问 | 提供解析后的邮件查询接口和 [邮箱使用 skill](skills/cf-temp-mail-agent-mail/SKILL.md)，由用户提供邮箱地址凭证。 |

## 技术组成

- 后端：TypeScript、Hono、Cloudflare Workers、Email Routing、D1。
- 前端：Vue 3、Vite、Naive UI；通过 Cloudflare Pages 或 Worker 静态资源部署。
- 邮件解析：Worker 默认使用 postal-mime；前端使用 mail-parser-wasm，并保留 postal-mime 回退。
- 可选组件：KV、R2/S3、Workers AI、Resend、Python SMTP/IMAP 代理。
- 自动化测试：仓库包含基于 Playwright、Docker Compose 和 Mailpit 的 API、浏览器及邮件代理测试。

## 使用边界

收件需要配置自有域名的 Email Routing；发信、对象存储、AI 提取和通知集成取决于实际配置及服务额度。部署成本随域名、服务套餐和使用量变化。

现有 Agent skill 使用邮箱地址凭证查询或发送邮件，不负责创建邮箱，也未提供任务级只读凭证隔离。本文描述代码能力，不代表本分支已完成独立部署、成本评估或性能验收。

## 部署与文档

- [上游快速开始](https://temp-mail-docs.awsl.uk/zh/guide/quick-start.html)
- [GitHub Actions 部署](https://temp-mail-docs.awsl.uk/zh/guide/actions/github-action.html)
- [Worker 配置](worker/wrangler.toml.template)
- [Agent 邮箱接口说明](vitepress-docs/docs/zh/guide/feature/agent-email.md)
- [端到端测试说明](e2e/README.md)
- [更新日志](CHANGELOG.md)

## 上游演示

[mail.awsl.uk](https://mail.awsl.uk/) 是上游项目提供的演示实例，不代表本仓库维护者的独立部署。部署和功能说明请结合本分支版本及所用配置核对。

## 核心功能

<details open>
<summary>核心功能详情（点击收缩/展开）</summary>

### 邮件处理

- [x] 前端优先使用 `mail-parser-wasm` 解析邮件，失败时回退到 `postal-mime`；Worker 默认使用 `postal-mime`
- [x] **AI 邮件识别** - 使用 Cloudflare Workers AI 自动提取邮件中的验证码、认证链接、服务链接等重要信息
- [x] 支持为指定基础域名创建随机二级域名邮箱地址，更适合收件隔离场景
- [x] 支持发送邮件，支持 `DKIM` 验证
- [x] 支持 `SMTP` 和 `Resend` 等多种发送方式 
- [x] 增加查看 `附件` 功能，支持附件图片显示
- [x] 支持 S3 附件存储和删除功能
- [x] 垃圾邮件检测和黑白名单配置
- [x] 邮件转发功能，支持全局转发地址

### 用户管理

- [x] 使用 `凭证` 重新登录之前的邮箱
- [x] 添加完整的用户注册登录功能，可绑定邮箱地址，绑定后可自动获取邮箱JWT凭证切换不同邮箱
- [x] 支持 `OAuth2` 第三方登录（Github、Authentik 等）
- [x] 支持 `Passkey` 无密码登录
- [x] 用户角色管理，支持多角色域名和前缀配置
- [x] 用户收件箱查看，支持地址和关键词过滤

### 管理功能

- [x] 完整的 admin 控制台
- [x] `admin` 后台创建无前缀邮箱
- [x] admin 用户管理页面，增加用户地址查看功能
- [x] 定时清理功能，支持多种清理策略
- [x] 获取自定义名字的邮箱，`admin` 可配置黑名单
- [x] 增加访问密码，可作为私人站点

### 多语言与界面

- [x] 前后台均支持多语言
- [x] 现代化 UI 设计，支持响应式布局
- [x] 支持 Google Ads 集成
- [x] 使用 shadow DOM 防止样式污染
- [x] 支持 URL JWT 参数自动登录

### 集成与扩展

- [x] 完整的 `Telegram Bot` 支持，以及 `Telegram` 推送，Telegram Bot 小程序
- [x] 添加 `SMTP proxy server`，支持 `SMTP` 发送邮件，`IMAP` 查看邮件
- [x] Webhook 支持，消息推送集成
- [x] 支持 `CF Turnstile` 人机验证
- [x] 限流配置，防止滥用
- [x] **Agent 友好**：内置 [`cf-temp-mail-agent-mail`](skills/cf-temp-mail-agent-mail/SKILL.md) skill，AI agent 可直接消费邮箱，详见 [文档](vitepress-docs/docs/zh/guide/feature/agent-email.md)
- [x] 社区移动端管理客户端：[CloudMail](https://github.com/Lur1N77777/CloudMail) 基于 Expo / React Native，面向本项目兼容 API，提供 Android 管理员后台、地址管理、收件/发件/未知邮件、验证码快捷复制、OLED 黑主题和本地分组。

</details>

## 技术架构

<details>
<summary>技术架构详情（点击收缩/展开）</summary>

### 系统架构

- **数据库**: Cloudflare D1 作为主数据库
- **前端部署**: 使用 Cloudflare Pages 部署前端
- **后端部署**: 使用 Cloudflare Workers 部署后端
- **邮件转发**: 使用 Cloudflare Email Routing

### 技术栈

- **前端**: Vue 3 + Vite + Naive UI
- **后端**: TypeScript + Cloudflare Workers
- **邮件解析**: Worker 使用 postal-mime；前端使用 Rust WASM 并保留 postal-mime 回退
- **数据库**: Cloudflare D1 (SQLite)
- **存储**: Cloudflare KV + R2 (可选 S3)
- **代理服务**: Python SMTP/IMAP Proxy Server

### 主要组件

- **Worker**: 核心后端服务
- **Frontend**: Vue 3 用户界面
- **Mail Parser WASM**: Rust 邮件解析模块
- **SMTP Proxy Server**: Python 邮件代理服务
- **Pages Functions**: Cloudflare Pages 中间件
- **Documentation**: VitePress 文档站点

</details>

### 提醒

- 在Resend添加域名记录时，如果您域名解析服务商正在托管您的3级域名a.b.com，请删除Resend生成的默认name中二级域名前缀b，否则将会添加a.b.b.com，导致验证失败。添加记录后，可通过
```bash
nslookup -qt="mx" a.b.com 1.1.1.1
```
进行验证。 

## 加入社区

- [Telegram](https://t.me/cloudflare_temp_email)
