# NexusStay OS

酒店 / 民宿前台管理系统 — 房态看板、入住退房、预订管理、用户权限、审计日志，开箱即用。

---

## 目录

- [功能特性](#功能特性)
- [项目版本](#项目版本)
- [版本对比](#版本对比)
- [目录结构](#目录结构)
- [部署指南](#部署指南)
  - [版本一：Node.js + JSON 文件数据库](#版本一nodejs--json-文件数据库)
  - [版本二：Node.js + MySQL](#版本二nodejs--mysql)
  - [版本三：PHP + MySQL（推荐）](#版本三php--mysql推荐)
- [默认账号](#默认账号)
- [API 接口文档](#api-接口文档)
- [前端页面说明](#前端页面说明)
- [技术栈](#技术栈)
- [常见问题](#常见问题)
- [许可证](#许可证)

---

## 功能特性

### 前台看板
- 房态总览：按楼层 / 按房型两种视图切换
- 房间卡片：颜色区分状态（空闲 / 预订 / 已住 / 维修打扫）
- 搜索过滤：按房号、房型关键词搜索
- 自适应卡片网格布局（桌面端 3 列卡片，移动端垂直滚动）
- 每个房型卡片支持独立缩放（60%~140%）
- 楼层分组标签 + 分割线
- 实时状态同步（多端自动刷新）

### 房间操作
- 直接入住（散客 Walk-in）
- 新建预订
- 预订转入住
- 取消预订
- 退房结算（自动进入维修/打扫状态）
- 打扫完成释放（恢复空闲）
- 客人信息录入（姓名、电话、预计退房时间、押金）
- 默认退房时间可配置

### 后台管理
- **后台首页**：今日入住/退房/预订统计、订单概览、房态概览、快捷导航
- **历史订单**：全部订单列表、按状态/日期/关键词筛选、分页（每页 8 条）、操作员显示、订单删除
- **账户管理**：管理员/前台账号 CRUD、分页、密码修改、权限控制（至少保留 1 个管理员）
- **系统设置**：退房默认时间（偏移天数 + 小时 + 分钟）、审计日志自动清理天数（1-365 天）
- **房型配置**：房型增删改、排序、分页
- **房间配置**：单个/批量新增房间、编辑房号/楼层/房型、分页
- **审计日志**：全部操作记录、关键词筛选、自动过期清理

### 安全特性
- 密码哈希存储（Node.js: scrypt, PHP: bcrypt）
- Cookie Session 认证（HttpOnly, SameSite=Lax）
- 24 小时 Session 过期
- 登录状态持久化（重启不丢失）
- 管理员权限分级（ADMIN / STAFF）
- 防重复安装（安装锁文件）
- 自定义确认弹窗（非浏览器原生 confirm）

---

## 项目版本

本项目提供三个版本，功能完全相同，仅后端技术栈不同：

| 版本 | 后端 | 数据库 | 目录 |
|------|------|--------|------|
| **版本一** | Node.js（原生 http） | JSON 文件 | `node-json/` |
| **版本二** | Node.js（原生 http） | MySQL | `node-mysql/` |
| **版本三** | PHP | MySQL | `php-mysql/` |

---

## 版本对比

| 特性 | Node.js + JSON | Node.js + MySQL | PHP + MySQL |
|------|---------------|-----------------|-------------|
| **后端语言** | Node.js | Node.js | PHP |
| **数据库** | `data/db.json` 文件 | MySQL | MySQL |
| **依赖** | 零依赖 | mysql2（npm） | PHP 原生 PDO |
| **进程管理** | PM2（推荐） | PM2（推荐） | Apache/Nginx 自动管理 |
| **端口** | 需开端口 + 反向代理 | 需开端口 + 反向代理 | 直接走 80/443 |
| **实时刷新** | WebSocket | WebSocket | SSE（Server-Sent Events） |
| **并发能力** | 低（JSON 全量读写） | 高 | 高 |
| **数据安全** | 低（写入可能丢数据） | 高（事务支持） | 高（事务支持） |
| **安装向导** | 无（默认 admin/123456） | Web 安装向导 | Web 安装向导 |
| **部署难度** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| **适用场景** | 本地测试、快速体验 | 服务器运维熟悉者 | 宝塔面板用户（推荐） |
| **Session 持久化** | ✅ 写入 JSON 文件 | ✅ 写入 MySQL | ✅ PHP 文件 Session |
| **数据迁移** | 复制 db.json | MySQL 导出/导入 | MySQL 导出/导入 |

---

## 目录结构

### 版本一：Node.js + JSON

```
node-json/
├── index.html          # 前端 SPA（React + Tailwind CSS）
├── server.js           # 后端 HTTP 服务器
├── data/
│   └── db.json         # JSON 数据库（运行后自动生成）
├── server.out.log      # 标准输出日志
└── server.err.log      # 错误日志
```

### 版本二：Node.js + MySQL

```
node-mysql/
├── index.html          # 前端 SPA
├── server.js           # 后端 HTTP 服务器
├── db.js               # MySQL 连接池 + 建表 + 配置管理
├── install.html        # Web 安装向导页面
├── package.json        # npm 依赖声明
├── node_modules/       # npm 依赖（npm install 后生成）
└── data/
    ├── config.json     # 数据库配置（安装后自动生成）
    └── install.lock    # 安装锁文件（安装后自动生成）
```

### 版本三：PHP + MySQL

```
php-mysql/
├── index.html          # 前端 SPA（SSE 替代 WebSocket）
├── api.php             # 全部后端逻辑（路由 + 业务 + 数据库）
├── install.html        # Web 安装向导页面
├── .htaccess           # Apache URL 重写
├── config.php          # 数据库配置（安装后自动生成）
└── data/
    ├── install.lock    # 安装锁文件（安装后自动生成）
    └── events.json     # SSE 事件通知文件
```

---

## 部署指南

### 版本一：Node.js + JSON 文件数据库

> 最简版本，零依赖，适合本地测试。

#### 前置要求

- Node.js 18+

#### 步骤

```bash
# 1. 进入目录
cd node-json/

# 2. 直接启动（无需安装依赖）
node server.js

# 3. 访问
# http://localhost:4173
```

#### 使用 PM2 保持运行（可选）

```bash
npm install -g pm2
pm2 start server.js --name nexusstay
pm2 save
pm2 startup
```

#### 宝塔部署

1. 安装 Node.js 版本管理器（软件商店）
2. 上传 `node-json/` 目录下所有文件
3. SSH 执行 `node server.js` 或用 PM2
4. 宝塔防火墙放行 4173
5. 访问 `http://IP:4173`

#### 配置 Nginx 反向代理（域名访问时）

```nginx
location / {
    proxy_pass http://127.0.0.1:4173;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

#### 默认账号

- 用户名：`admin`
- 密码：`123456`

#### 注意事项

- 数据存在 `data/db.json`，重启不丢失
- Session 存在 JSON 文件中，重启不丢失
- 并发写入可能丢数据，不建议多人同时操作
- 不需要 MySQL

---

### 版本二：Node.js + MySQL

> 生产级版本，支持高并发，Web 安装向导。

#### 前置要求

- Node.js 18+
- MySQL 5.7+ 或 8.0+

#### 步骤

```bash
# 1. 创建 MySQL 数据库
# 在宝塔数据库管理中创建数据库 nexusstay

# 2. 进入目录
cd node-mysql/

# 3. 安装依赖
npm install

# 4. 启动
node server.js

# 5. 访问安装向导
# http://localhost:4173
# 按提示填写 MySQL 连接信息和管理员账号
```

#### 使用 PM2 保持运行（可选）

```bash
pm2 start server.js --name nexusstay
pm2 save
pm2 startup
```

#### 环境变量配置（可选）

启动时可通过环境变量覆盖数据库配置：

```bash
MYSQL_HOST=127.0.0.1 \
MYSQL_PORT=3306 \
MYSQL_USER=root \
MYSQL_PASSWORD=your_password \
MYSQL_DATABASE=nexusstay \
node server.js
```

#### 宝塔部署

1. 安装 Node.js 版本管理器（软件商店）
2. 在宝塔 MySQL 中创建数据库
3. 上传 `node-mysql/` 目录下所有文件
4. SSH 执行 `cd node-mysql && npm install`
5. 用 PM2 或直接 `node server.js` 启动
6. 访问 `http://IP:4173` 进入安装向导
7. 配置 Nginx 反向代理（同版本一）

#### 安装向导

首次访问自动跳转安装向导：

1. **数据库配置**：填写 MySQL 地址、端口、用户名、密码、库名 → 测试连接
2. **管理员账号**：填写用户名和密码 → 开始安装
3. 安装完成 → 自动跳转首页

安装后生成：
- `data/config.json` — 数据库配置
- `data/install.lock` — 安装锁

#### 重新安装

```bash
rm data/install.lock data/config.json
# 然后重启服务，再次访问首页
```

---

### 版本三：PHP + MySQL（推荐）

> 宝塔面板首选，部署最简单，无需 Node.js / PM2 / 反向代理。

#### 前置要求

- PHP 7.4+（宝塔自带）
- MySQL 5.7+ 或 8.0+
- Nginx 或 Apache（宝塔自带）

#### 宝塔部署步骤

**1. 创建站点**

宝塔 → 网站 → 添加站点 → 填入域名 → PHP 版本选择 7.4 或 8.x → 提交

**2. 创建数据库**

宝塔 → 数据库 → MySQL → 添加数据库：
- 数据库名：`nexusstay`
- 用户名：自定义
- 密码：自定义

**3. 上传文件**

将 `php-mysql/` 目录下 4 个文件上传到站点根目录：

```
/www/wwwroot/你的域名/
├── index.html
├── install.html
├── api.php
├── .htaccess
└── data/           ← 空目录
```

**4. 配置 Nginx 重写**

宝塔站点设置 → 配置文件，在 `server` 块中添加：

```nginx
location /api/ {
    rewrite ^/api/(.*)$ /api.php last;
}
```

如果使用 Apache，`.htaccess` 文件已包含重写规则，无需额外配置。

**5. 访问安装向导**

浏览器打开 `http://你的域名`：

1. 填写 MySQL 连接信息 → 测试连接
2. 填写管理员用户名和密码 → 开始安装
3. 安装成功 → 点击进入首页

**6. 开启 HTTPS（可选）**

站点设置 → SSL → Let's Encrypt → 申请证书 → 勾选强制 HTTPS

#### 重新安装

```bash
rm data/install.lock config.php
# 然后再次访问首页
```

---

## 默认账号

| 版本 | 用户名 | 密码 | 说明 |
|------|--------|------|------|
| Node.js + JSON | `admin` | `123456` | 内置默认账号 |
| Node.js + MySQL | 自定义 | 自定义 | 安装向导中设置 |
| PHP + MySQL | 自定义 | 自定义 | 安装向导中设置 |

---

## API 接口文档

所有接口统一前缀 `/api/`，请求和响应均为 JSON。

### 认证相关

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/auth/me` | 获取当前登录用户 | 无需登录 |
| POST | `/api/auth/login` | 登录 | 无需登录 |
| POST | `/api/auth/logout` | 登出 | 需要登录 |

### 数据查询

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/inventory` | 获取全部数据（房型、房间、订单、日志、用户） | 需要登录 |

### 系统设置

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| PATCH | `/api/settings` | 更新系统设置 | 管理员 |

### 用户管理

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/users` | 创建用户 | 管理员 |
| PATCH | `/api/users/:id` | 修改用户 | 管理员 |
| DELETE | `/api/users/:id` | 删除用户 | 管理员 |

### 房型管理

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/room-types` | 创建房型 | 管理员 |
| PATCH | `/api/room-types/:id` | 修改房型 | 管理员 |
| DELETE | `/api/room-types/:id` | 删除房型 | 管理员 |

### 房间管理

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/rooms` | 创建单个房间 | 管理员 |
| POST | `/api/rooms/bulk` | 批量创建房间 | 管理员 |
| PATCH | `/api/rooms/:id` | 修改房间 | 管理员 |
| DELETE | `/api/rooms/:id` | 删除房间 | 管理员 |
| POST | `/api/rooms/:id/action` | 房间操作（入住/退房/预订等） | 需要登录 |

### 订单管理

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| DELETE | `/api/orders/:id` | 删除已完成/已取消订单 | 管理员 |

### 安装接口

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/install/status` | 查询安装状态 | 无需登录 |
| POST | `/api/install/test-db` | 测试数据库连接 | 无需登录 |
| POST | `/api/install` | 执行安装 | 无需登录 |

### 实时事件

| 方法 | 路径 | 说明 | 权限 | 协议 |
|------|------|------|------|------|
| GET | `/api/events` | 实时事件流 | 需要登录 | SSE（PHP 版） / WebSocket（Node 版） |

### 房间操作 actionType

| actionType | 说明 | 前置状态 |
|------------|------|---------|
| `CHECK_IN` | 直接入住 | FREE |
| `RESERVE` | 新建预订 | FREE |
| `CONVERT_TO_CHECK_IN` | 预订转入住 | RESERVED |
| `CANCEL_RESERVATION` | 取消预订 | RESERVED |
| `CHECK_OUT` | 退房结算 | OCCUPIED |
| `CLEAN_DONE` | 打扫完成 | MAINTENANCE |

### 房间状态流转

```
空闲(FREE) ──→ 预订(RESERVED) ──→ 已住(OCCUPIED) ──→ 维修/打扫(MAINTENANCE) ──→ 空闲(FREE)
    │                                                        ↑
    └──────────── 直接入住 ──→ 已住(OCCUPIED) ──→ 退房 ──────┘
```

---

## 前端页面说明

### 页面路由

| 路径 | 页面 | 说明 |
|------|------|------|
| `/` | 前台看板 | 房态总览、房间操作 |
| `/admin` | 后台管理 | 系统管理、订单、用户、设置 |

### 前台看板功能

- 房态总览网格（桌面端自适应卡片布局）
- 按楼层 / 按房型切换（移动端）
- 房间搜索
- 点击房间卡片弹出操作面板
- 房型卡片缩放控制
- 实时状态同步

### 后台管理功能

- 后台首页（今日统计 + 订单概览 + 房态概览 + 导航）
- 历史订单（筛选 + 分页 + 操作员 + 删除）
- 账户管理（CRUD + 分页 + 二次确认弹窗）
- 系统设置（退房时间 + 日志清理天数，分卡片显示）
- 房型配置（CRUD + 分页）
- 房间配置（单个/批量 + CRUD + 分页）
- 审计日志（筛选 + 全量显示）

---

## 技术栈

### 前端（三个版本通用）

- **React 18** — UI 框架（CDN 引入，浏览器端 Babel 编译 JSX）
- **Tailwind CSS** — 样式框架（CDN 引入）
- **EventSource / WebSocket** — 实时通信

### 后端

| 组件 | Node.js 版 | PHP 版 |
|------|-----------|--------|
| HTTP 服务 | 原生 `http` 模块 | Apache / Nginx + PHP-FPM |
| 路由 | 手写路径匹配 | `api.php` 统一路由 |
| 数据库驱动 | mysql2（npm） | PDO（PHP 原生） |
| 密码哈希 | `crypto.scryptSync` | `password_hash(PASSWORD_BCRYPT)` |
| Session | 内存 Map + MySQL/JSON 持久化 | Cookie + MySQL |
| 实时推送 | WebSocket（手写帧协议） | Server-Sent Events |
| 进程管理 | PM2 | Apache/Nginx 自动管理 |

### 数据库表结构（MySQL 版）

| 表名 | 说明 |
|------|------|
| `settings` | 系统配置键值对 |
| `room_types` | 房型（名称、描述、排序） |
| `rooms` | 房间（房号、楼层、房型、状态） |
| `orders` | 订单（入住/预订/退房/取消） |
| `users` | 用户（账号、密码哈希、角色） |
| `sessions` | 登录 Session（token、用户、过期时间） |
| `audit_logs` | 审计日志（操作员、动作、详情、时间） |

---

## 常见问题

### Q: 三个版本功能有区别吗？

没有。前端页面完全相同，API 接口完全相同，只是后端实现和数据库不同。

### Q: 可以从 JSON 版迁移到 MySQL 版吗？

需要手动迁移数据。JSON 版的 `data/db.json` 包含所有数据，可以编写脚本导入 MySQL。但由于密码哈希算法不同（scrypt vs bcrypt），用户密码需要重新设置。

### Q: 忘记管理员密码怎么办？

**Node.js + JSON 版**：删除 `data/db.json` 中的 `users` 数组，重启后会自动重建默认管理员。

**MySQL 版**：在数据库中执行：
```sql
DELETE FROM users;
-- 然后删除 install.lock 重新安装
```

### Q: 如何备份数据？

- **JSON 版**：复制 `data/db.json`
- **MySQL 版**：使用 `mysqldump` 或宝塔数据库备份功能

### Q: 多人同时操作会冲突吗？

- **JSON 版**：可能冲突（全量读写），建议单人使用
- **MySQL 版**：不会冲突（行级锁 + 事务）

### Q: 支持多语言吗？

当前仅支持中文。

### Q: 移动端可以用吗？

可以。前端已适配移动端，支持触摸操作和响应式布局。

### Q: Nginx 反向代理 WebSocket 配置？

```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:4173;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

PHP 版使用 SSE，不需要特殊配置。

---

## 许可证

MIT License
