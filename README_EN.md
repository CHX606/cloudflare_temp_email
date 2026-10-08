<!-- markdownlint-disable-file MD033 MD045 -->
# Cloudflare Self-hosted Temporary Email

A self-hosted temporary email service built with Cloudflare Workers, D1 and Email Routing. It provides custom-domain mailbox management, sending and receiving mail, and attachment viewing, with access through the web interface, SMTP/IMAP proxy and agent APIs.

This repository is derived from [dreamhunter2333/cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email) and retains its commit history and [MIT license](LICENSE). The core features below come from upstream; documentation updates in this branch are recorded in the [changelog](CHANGELOG_EN.md).

[中文](README.md) · [English](README_EN.md) · [Upstream deployment documentation](https://temp-mail-docs.awsl.uk)

## Overview

| Scenario | Implementation and capabilities |
| --- | --- |
| Custom-domain inboxes | Email Routing delivers incoming mail to the Worker for processing and D1 storage, with address creation, passwords and lifecycle management. |
| Mail and attachments | The web inbox supports mail search and body/attachment viewing. The frontend tries Rust WASM parsing first and falls back to postal-mime. |
| Sending mail | Resend, SMTP and other send methods require their corresponding services, domain settings and send allowances. |
| Access management | User accounts, OAuth2, Passkey, address credentials, site access passwords and domain permissions are available. |
| Integrations | Mail forwarding, webhooks, Telegram notifications and an SMTP/IMAP proxy are supported. |
| Agent access | Parsed-mail APIs and a [mailbox skill](skills/cf-temp-mail-agent-mail/SKILL.md) use address credentials supplied by the user. |

## Technology

- Backend: TypeScript, Hono, Cloudflare Workers, Email Routing and D1.
- Frontend: Vue 3, Vite and Naive UI, deployed with Cloudflare Pages or Worker static assets.
- Parsing: postal-mime is the default Worker parser; the frontend uses mail-parser-wasm with a postal-mime fallback.
- Optional components: KV, R2/S3, Workers AI, Resend and the Python SMTP/IMAP proxy.
- Automated tests: Playwright, Docker Compose and Mailpit cover API, browser and mail-proxy scenarios in the repository.

## Operational boundaries

Receiving mail requires Email Routing configuration for your domain. Sending, object storage, AI extraction and notifications depend on the configured services and quotas. Costs vary with the domain, service plans and usage.

The existing agent skill reads or sends mail using an address credential. It does not create mailboxes or provide task-scoped, read-only credentials. This document describes code capabilities and does not claim an independently deployed instance, cost evaluation or performance acceptance for this branch.

## Deployment and documentation

- [Upstream quick start](https://temp-mail-docs.awsl.uk/en/guide/quick-start.html)
- [GitHub Actions deployment](https://temp-mail-docs.awsl.uk/en/guide/actions/github-action.html)
- [Worker configuration](worker/wrangler.toml.template)
- [Agent email APIs](vitepress-docs/docs/en/guide/feature/agent-email.md)
- [End-to-end tests](e2e/README.md)
- [Changelog](CHANGELOG_EN.md)

## Upstream demo

[mail.awsl.uk](https://mail.awsl.uk/) is an upstream demonstration instance, not an independent deployment by this repository's maintainer. Check deployment guidance against this branch's version and your configuration.

## Core Features

<details open>
<summary>Core Features Details (Click to expand/collapse)</summary>

### Email Processing

- [x] The frontend tries `mail-parser-wasm` first and falls back to `postal-mime`; the Worker uses `postal-mime` by default
- [x] **AI Email Recognition** - Use Cloudflare Workers AI to automatically extract verification codes, authentication links, service links and other important information from emails
- [x] Support optional random second-level subdomain mailbox creation for selected base domains
- [x] Support sending emails with `DKIM` verification
- [x] Support multiple sending methods such as `SMTP` and `Resend`
- [x] Add attachment viewing feature with support for displaying attachment images
- [x] Support S3 attachment storage and deletion
- [x] Spam detection and blacklist/whitelist configuration
- [x] Email forwarding feature with global forwarding address support

### User Management

- [x] Use `credentials` to log in to previously used mailboxes
- [x] Add complete user registration and login functionality. Users can bind email addresses and automatically obtain email JWT credentials to switch between different mailboxes after binding
- [x] Support `OAuth2` third-party login (Github, Authentik, etc.)
- [x] Support `Passkey` passwordless login
- [x] User role management with support for multi-role domain and prefix configuration
- [x] User inbox viewing with address and keyword filtering support

### Admin Features

- [x] Complete admin console
- [x] Create mailboxes without prefix in `admin` backend
- [x] Admin user management page with user address viewing feature
- [x] Scheduled cleanup function with support for multiple cleanup strategies
- [x] Get mailboxes with custom names, `admin` can configure blacklist
- [x] Add access password for use as a private site

### Multi-language & Interface

- [x] Both frontend and backend support multi-language
- [x] Modern UI design with responsive layout
- [x] Google Ads integration support
- [x] Use shadow DOM to prevent style pollution
- [x] Support URL JWT parameter auto-login

### Integration & Extensions

- [x] Complete `Telegram Bot` support, `Telegram` push notifications, and Telegram Bot mini app
- [x] Add `SMTP proxy server` supporting `SMTP` for sending emails and `IMAP` for viewing emails
- [x] Webhook support and message push integration
- [x] Support `CF Turnstile` CAPTCHA verification
- [x] Rate limiting configuration to prevent abuse
- [x] **Agent-friendly**: bundled [`cf-temp-mail-agent-mail`](skills/cf-temp-mail-agent-mail/SKILL.md) skill lets AI agents consume a mailbox directly, see [docs](vitepress-docs/docs/en/guide/feature/agent-email.md)
- [x] Community mobile admin client: [CloudMail](https://github.com/Lur1N77777/CloudMail) is built with Expo / React Native for this project's compatible API, providing an Android admin console, address management, inbox/sent/unknown mail, quick verification-code copy, OLED black theme, and local grouping.

</details>

## Technical Architecture

<details>
<summary>Technical Architecture Details (Click to expand/collapse)</summary>

### System Architecture

- **Database**: Cloudflare D1 as the main database
- **Frontend Deployment**: Deploy frontend using Cloudflare Pages
- **Backend Deployment**: Deploy backend using Cloudflare Workers
- **Email Routing**: Use Cloudflare Email Routing

### Tech Stack

- **Frontend**: Vue 3 + Vite + Naive UI
- **Backend**: TypeScript + Cloudflare Workers
- **Email Parsing**: postal-mime in the Worker; Rust WASM with a postal-mime fallback in the frontend
- **Database**: Cloudflare D1 (SQLite)
- **Storage**: Cloudflare KV + R2 (optional S3)
- **Proxy Service**: Python SMTP/IMAP Proxy Server

### Main Components

- **Worker**: Core backend service
- **Frontend**: Vue 3 user interface
- **Mail Parser WASM**: Rust email parsing module
- **SMTP Proxy Server**: Python email proxy service
- **Pages Functions**: Cloudflare Pages middleware
- **Documentation**: VitePress documentation site

</details>

### Important Notes

- When adding domain records in Resend, if your DNS provider is hosting your 3rd level domain a.b.com, please remove the 2nd level domain prefix b from the default name generated by Resend, otherwise it will add a.b.b.com, causing verification to fail. After adding the record, you can verify it using:
```bash
nslookup -qt="mx" a.b.com 1.1.1.1
```

## Join the Community

- [Telegram](https://t.me/cloudflare_temp_email)
