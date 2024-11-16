+++
date = '2024-11-16T13:50:41+08:00'
draft = false
title = 'Vercel到Cloudflare：高性价比的迁移实践'
+++


## 背景
尽管 Vercel 的确在便利性和全栈支持上做得不错，但它的成本问题让人望而却步。
经过深入研究，我决定将我的 AI 搜索项目项目迁移到 AWS 和 Cloudflare，最终成功节省了大部分开支。
这篇文章记录了完整的迁移过程，希望对有类似需求的开发者有所帮助。

---

## 为什么会选择 Vercel 起步？

作为全栈开发的热门平台，Vercel 在功能设计上确实非常方便，尤其适合快速启动项目：
- **CI/CD 集成**：与 GitHub 无缝对接，支持代码提交自动上线、滚动更新与版本回退。
- **域名服务**：自动生成子域名，便于快速部署 Demo 版本。
- **分支支持**：为每次部署生成独立访问地址，适合团队调试和验证。
- **附加工具丰富**：包括访问统计、性能分析、日志管理、环境变量配置等。
- **Next.js 支持到位**：作为 Next.js 的官方部署平台，Vercel 提供了一整套开发、运维、部署的生态支持。

Vercel 的这些特性让它成为许多开发者的首选，尤其是初学者或需要快速验证想法的团队。

---

## Vercel 的痛点：高昂的成本

尽管 Vercel 提供了极大的便利，但它的成本问题也不容忽视：
###  1. 基础费用：
   - 免费版本仅支持个人项目，组织项目需要升级到团队版（20 美元/月）。
###  2. 云函数超时限制：
   - 默认 10 秒超时，最多 60 秒；升级到 Pro 版本后，超时限制提升至 300 秒，但很多高延时的任务仍然容易失败。
###  3. 功能单独计费：
   - 例如网页统计、数据库存储按访问量和空间收费，每月账单容易超出预期。
###  4. 流量计费：
   - 云函数调用费用高达 $0.18/GB Hrs；使用 Next.js 的 `Image` 组件会触发图片裁剪和 CDN 加速，这些隐性成本累积后也非常可观。

本质上，Vercel 只是 AWS 的一层封装，其费用显著高于直接使用 AWS 的裸机服务。对于需要控制成本的开发者来说，这是一个不得不解决的问题。

---

## 替代方案：探索更经济的部署方式

为了解决成本问题，我研究了几种替代 Vercel 的部署方式，主要如下：

### 1. 云部署平台
- **Netlify**：Vercel 的主要竞争对手，支持站点分析和活跃用户统计。
- **Railway**：支持 Docker 容器部署，适合多种项目类型。
- **Render、Firebase、Heroku**：各有特色，适合不同规模和场景。

### 2. 开源或自建部署
- **Coolify**：功能强大的开源平台，自托管替代方案。
- **Dokku**：轻量级 Heroku 替代方案，支持多语言框架。
- **自建服务器**：通过 PM2、Docker 或 K8S 管理服务。

### 3. Cloudflare 的托管方案
- **Cloudflare Pages**：适合静态站点和小型全栈应用的几乎免费服务。
- **Cloudflare Workers**：用于无服务器脚本和 API Proxy。
- **Cloudflare R2 和 D1**：低成本的文件存储和数据库托管。

---

## 实践分享：三种替代部署方案

### 1. 在云服务器上用 PM2 部署 Next.js
-   在 AWS EC2 上购买一台 Ubuntu 云服务器。
-   安装 Node.js、PNPM 和 PM2：
    ```bash
    npm install -g pnpm pm2
    pnpm install # 安装依赖
    pnpm build
    pm2 start pnpm --name my-app -- start # 启动服务
    ```

### 2. 使用 Docker 部署
-   修改项目配置，开启 standalone 模式：
    ```javascript
    const nextConfig = { output: "standalone" };
    export default nextConfig;
    ```
-   创建 Dockerfile 并构建 Docker 镜像：
    ```dockerfile
    FROM node:18-alpine
    WORKDIR /app
    COPY . .
    RUN pnpm install && pnpm build
    CMD ["node", "server.js"]
    ```
-   使用 Nginx 和 Certbot 配置公网访问与 HTTPS。

### 3. 使用 Cloudflare Pages 部署
-   安装依赖：
    ```bash
    pnpm add -D @cloudflare/next-on-pages
    ```
-   创建 wrangler.toml 配置文件，并更新 next.config.mjs 以适配 Edge Runtime。
-   执行部署命令：
    ```bash
    pnpm run deploy
    ```

---

## 迁移后的成本与收益
**通过将项目迁移至 AWS 和 Cloudflare：**
每月服务器成本从 5000 美元 降至不足 100 美元。
服务性能和可扩展性均得到了显著提升。
Cloudflare 提供的免费或低成本服务，如 DNS 解析、R2 文件存储、D1 数据库等，进一步降低了我的运维开销。

## 总结
本次迁移是一次高性价比的实践。不仅帮助我节省了大笔成本，也让我更加灵活地控制服务的架构与资源。如果你也面临类似的成本问题，希望这篇文章能为你提供有价值的参考。

> 对于开发者而言，选择合适的部署平台要综合考虑 **成本** 和 **运维复杂度**。

如果你也在寻找更具性价比的部署方案，希望这篇文章能为你提供一些思路。