# Bili-Calendar (B站追番日历)  
[![Netlify Status](https://api.netlify.com/api/v1/badges/efdead58-803f-40c4-bd24-f0f25a92922f/deploy-status)](https://app.netlify.com/projects/bili-calendar/deploys)
[![Docker Image](https://img.shields.io/badge/Docker-ghcr.io%2Fsilentely%2Fbili--calendar-blue?logo=docker)](https://github.com/Silentely/bili-calendar/pkgs/container/bili-calendar)

> 🎉 **将B站追番列表一键转换为日历订阅，支持 Apple/Google/Outlook 等主流日历应用！**

---

## 功能特点

- 📅 **自动同步**：B站追番列表一键生成日历订阅
- 🕒 **准确时间**：精确解析番剧更新时间，支持时区自动转换
- 🔁 **智能重复**：连载中番剧自动每周重复，完结番剧仅保留首播
- 📱 **多平台支持**：兼容 Apple/Google/Outlook 等所有支持 ICS 的日历应用
- 🚀 **简单易用**：只需提供 B站 UID 即可生成订阅链接
- 🔒 **隐私保护**：服务端不存储任何用户数据，支持自部署

---

## 效果预览

> 下图展示了前端界面、Apple 日历中的订阅效果与事件详情：

| 前端界面 | 日历视图 | 事件详情 |
|:---:|:---:|:---:|
| ![](./assets/front-end-web.jpg) | ![](./assets/calendar-view.jpg) | ![](./assets/event-detail.jpg) |

前端界面展示了用户输入 B站 UID 后生成订阅链接的过程。用户可以根据设备类型选择不同的操作：
- **桌面端**：显示订阅链接和使用说明，用户可以复制链接手动添加到日历应用
- **移动端**：直接跳转到系统日历应用进行订阅

---

## 使用方法

### 公共服务（推荐）

1. 访问 [https://calendar.cosr.eu.org](https://calendar.cosr.eu.org)
2. 输入您的 B站 UID（在 B站个人空间网址中找到，例如：`https://space.bilibili.com/614500` 中的 614500）
3. 点击"生成订阅"按钮
4. 将生成的订阅链接添加到您的日历应用中  
PS: 也可直接通过 `https://calendar.cosr.eu.org/uid` 形式复制到日历新增订阅里填入链接

---

### 私有部署

#### 使用 Docker（推荐）

```bash
# 创建 docker-compose.yml 文件
version: '3.8'
services:
  bili-calendar:
    image: ghcr.io/silentely/bili-calendar:latest
    ports:
      - "3000:3000"
    environment:
      - BILIBILI_COOKIE=  # 可选，用于提高API访问成功率
      - NODE_ENV=production
      - TZ=Asia/Shanghai
    restart: unless-stopped
    volumes:
      - ./logs:/app/logs  # 可选，用于持久化日志

# 启动服务
docker-compose up -d
```

#### 手动部署

```bash
# 克隆仓库
git clone https://github.com/Silentely/bili-calendar.git
cd bili-calendar

# 安装依赖
npm install

# 启动服务
npm start

# 或者在开发模式下运行
npm run dev
```

---

## API 接口

### 获取用户追番日历

```
GET /:uid
```

参数：
- `uid`: B站用户 UID

返回：ICS 格式的日历文件

### 获取用户追番数据（JSON）

```
GET /api/bangumi/:uid
```

参数：
- `uid`: B站用户 UID

返回：B站追番列表的 JSON 数据

> **速率限制**：为防止滥用，API直接访问限制为每个IP每小时3次。项目内部调用不受此限制。API响应头中包含 `X-RateLimit-*` 系列字段，用于了解当前使用情况。

---

## 配置说明

### 环境变量

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `PORT` | 3000 | 服务监听端口 |
| `BILIBILI_COOKIE` | 空 | B站 Cookie，用于提高API访问成功率 |
| `NODE_ENV` | development | 运行环境（development/production） |
| `TZ` | Asia/Shanghai | 时区设置 |
| `API_RATE_LIMIT` | 3 | API调用速率限制（次数/时间窗口） |
| `API_RATE_WINDOW` | 3600000 | 速率限制时间窗口（毫秒，默认1小时） |
| `ENABLE_RATE_LIMIT` | true | 是否启用速率限制（true/false） |

### 注意事项

1. **隐私设置**：您的 B站追番列表必须设置为公开才能被获取
2. **Cookie 设置**：如果遇到访问频率限制，可以设置 `BILIBILI_COOKIE` 环境变量
3. **时区处理**：服务默认使用东八区时间（北京时间），请确保部署环境时区正确

---

## 技术架构

- **后端**：Node.js + Express
- **前端**：原生 HTML/CSS/JavaScript
- **容器化**：Docker + Docker Compose (amd64, arm64)
- **Serverless**：支持 Netlify Functions 部署
- **日历格式**：遵循 RFC 5545 标准的 ICS 格式

---

## 开发指南

### 项目结构

```
bili-calendar/
├── server.js              # Docker部署的主服务文件
├── main.js                # Docker部署的主应用逻辑
├── netlify.toml           # Netlify配置
├── .github/               # GitHub配置目录
│   └── workflows/         # GitHub Actions工作流配置
│       └── docker-build.yml # Docker镜像自动构建配置
├── netlify/               # Netlify函数目录
│   └── functions/         # 函数源代码
│       └── server.js      # Serverless版本的服务器
├── public/                # 静态文件目录
│   └── index.html         # 前端页面
├── Dockerfile             # Docker 镜像配置
├── docker-compose.yml     # Docker Compose 配置
├── package.json           # Node.js 项目配置
└── README.md              # 项目说明文档
```

### 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器（支持热重载）
npm run dev

# 构建并启动生产服务器
npm run start:prod
```

---

## 部署指南

### 部署到云服务器

1. 克隆项目到服务器
2. 安装 Node.js 环境（建议使用 v18 或更高版本）
3. 安装 PM2 进程管理器：`npm install -g pm2`
4. 安装项目依赖：`npm install`
5. 启动服务：`pm2 start npm --name "bili-calendar" -- start`
6. 设置开机自启：`pm2 startup && pm2 save`

### 部署到 Netlify

1. 在 Netlify 导入 GitHub 仓库
2. 配置以下构建设置:
   - 构建命令: `npm run build`
   - 发布目录: `public`
   - 环境变量: 根据需要设置 `BILIBILI_COOKIE` 等
3. Netlify.toml 文件已包含必要配置:
   ```toml
   [build]
     command = "npm run build"
     publish = "public"
     functions = "netlify/functions-build"

   [[redirects]]
     from = "/*"
     to = "/.netlify/functions/server"
     status = 200
   ```
4. 本项目已包含所有必要的Netlify Functions配置，无需额外设置

> **注意**: 项目已通过 `serverless-http` 将Express应用包装为Netlify函数，并使用CommonJS模块格式，所有必要的依赖已添加到package.json中。

---

## 多平台部署说明

本项目支持多种部署方式，每种方式使用不同的代码实现以适应特定平台的要求：

### Docker部署
- 使用 `server.js` 和 `main.js` 作为主应用文件
- 包含完整的 Express 应用实现
- 适用于传统服务器部署或容器化部署

### Netlify Functions部署
- 使用 `netlify/functions/server.js` 作为函数入口
- 通过 `serverless-http` 将 Express 应用包装为无服务器函数
- 使用 CommonJS 模块系统以适应 Netlify 环境

### 各平台实现差异
1. **限流机制**：
   - Docker版本使用定时清理的限流机制
   - Netlify版本使用机会性清理的限流机制

2. **日志处理**：
   - 不同平台使用不同的日志格式以适应各自环境

3. **错误处理**：
   - 各平台针对特定错误场景进行了优化处理

4. **IP地址处理**：
   - Netlify版本包含更完善的IP地址处理逻辑

---

## 常见问题

### 为什么显示"[时间未知]"？

可能原因：
1. 番剧尚未公布具体更新时间
2. B站API返回的数据格式有变化
3. 网络问题导致API访问失败

### 如何获取 B站 UID？

1. 打开 B站个人空间页面（例如：`https://space.bilibili.com/614500`）
2. URL中的数字部分就是您的 UID（示例中为 614500）

### 如何更新日历？

日历订阅链接是动态生成的，会自动获取最新的追番列表。大多数日历应用会定期自动同步更新。

### 为什么有些番剧没有显示在日历中？

项目会自动过滤掉已经完结的番剧和没有明确播出时间的番剧。只有连载中且有明确播出时间的番剧才会显示在日历中。

---

## 贡献指南

欢迎提交 Issue 和 Pull Request 来帮助改进本项目！

### 开发流程

1. Fork 项目
2. 创建功能分支：`git checkout -b feature/your-feature-name`
3. 提交更改：`git commit -am 'Add some feature'`
4. 推送到分支：`git push origin feature/your-feature-name`
5. 创建 Pull Request

---

## 免责声明

本项目仅供学习交流使用，不提供任何BiliBili相关的账号服务。请遵守BiliBili的相关服务条款和使用规范.

---

## License

- 本项目的所有代码除另有说明外,均按照 [MIT License](LICENSE) 发布。
- 本项目的README.MD，wiki等资源基于 [CC BY-NC-SA 4.0][CC-NC-SA-4.0] 这意味着你可以拷贝、并再发行本项目的内容，<br/>
  但是你将必须同样**提供原作者信息以及协议声明**。同时你也**不能将本项目用于商业用途**，按照我们狭义的理解<br/>
  (增加附属条款)，凡是**任何盈利的活动皆属于商业用途**。
- 请在遵守当地相关法律法规的前提下使用本项目。

<p align="center">
  <img src="https://github.com/docker/dockercraft/raw/master/docs/img/contribute.png?raw=true" alt="贡献图示">
</p>

[github-hosts]: https://raw.githubusercontent.com/racaljk/hosts/master/hosts "hosts on Github"
[CC-NC-SA-4.0]: https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh

<div align="center">
  <sub>Made with ❤️ by <a href="https://github.com/Silentely">Silentely</a></sub>
</div>