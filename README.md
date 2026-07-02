# bbs1org

bbs1org 是一个轻量级 PHP 论坛程序，使用 SQLite 存储数据，核心功能集中在少量文件内，适合个人站点、小型社区或作为可直接改造的论坛基底。


## 功能特性

- 主题、回复、浏览量、收藏、个人主页
- 用户注册、登录、资料编辑、找回密码
- 用户组权限：浏览、发帖、回帖、后台管理、内容管理
- 后台管理：设置、版块、用户组、主题、回帖、用户、回收站
- 通知系统：@ 提及通知、站内私信
- 主题置顶、高亮、移动、删除和回收站恢复
- Markdown 风格的基础内容渲染
- SQLite 数据库，默认开启 WAL 和常用性能 PRAGMA
- 可选伪静态 URL
- CSRF 防护、登录/注册/重置密码 IP 频率限制

## 环境要求

- PHP 8.0 或更高版本，建议 PHP 8.1+
- PHP 扩展：`pdo_sqlite`
- Web 服务器：Apache、Nginx、Caddy 或 PHP 内置服务器
- `data/` 和 `cache/` 目录需要 PHP 进程可写

可选能力：

- `mbstring`：用于更好的中文字符串截断和大小写处理
- `mail()` 可用的邮件发送环境：用于找回密码邮件
- Apache `mod_rewrite`：用于伪静态 URL

## 快速安装

1. 将项目文件放到 Web 根目录或子目录。
2. 确保 PHP 可以写入数据和缓存目录：

```bash
mkdir -p data cache
chmod 755 data cache
```

3. 访问安装入口：

```text
https://你的域名/install.php
```

4. 按页面提示填写站点名称、管理员账号、默认版块和首个主题内容。
5. 安装完成后，系统会创建：

- `data/db.php`
- `data/*.sqlite`
- `data/install.lock`
- `cache/*.php`

6. 保存安装完成页显示的管理员密码，然后进入后台修改站点设置。

## 本地开发

如果本机安装了 PHP，可以直接用内置服务器运行：

```bash
php -S 127.0.0.1:8000
```

然后访问：

```text
http://127.0.0.1:8000/install.php
```

PHP 内置服务器适合本地测试。生产环境建议使用 Apache、Nginx 或 Caddy。

## 目录说明

```text
index.php       主程序入口
install.php     初始化安装入口
index.css       前台和后台样式
index.js        前端交互脚本
logo.svg        站点图标
.htaccess       Apache 重写和 data/cache 访问拦截
data/           SQLite 数据库、安装锁和数据库配置
cache/          论坛、用户组、设置和统计缓存
```

`data/` 和 `cache/` 不应被公网直接访问。仓库自带的 `.htaccess` 已在 Apache 下拦截这两个目录；如果使用 Nginx/Caddy，请在服务器配置中额外禁止访问它们。

## Nginx 配置要点

示例片段：

```nginx
location ^~ /data/ { deny all; }
location ^~ /cache/ { deny all; }

location / {
    try_files $uri $uri/ /index.php?$query_string;
}

location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass unix:/run/php/php-fpm.sock;
}
```

请按实际 PHP-FPM socket 或 TCP 地址调整 `fastcgi_pass`。

## 后台设置

登录管理员账号后访问：

```text
/index.php?a=admin
```

如果启用了伪静态，也可以访问：

```text
/admin
```

常用设置：

- `站点地址`：建议填写完整域名，例如 `https://example.com`。找回密码邮件会优先使用该地址生成链接。
- `启用伪静态`：需要 Web 服务器 rewrite 支持。
- `系统发件邮箱`：用于找回密码邮件的 From 地址。
- `是否虚拟发送邮件`：仅建议本地调试使用；线上应配置真实邮件发送。
- `注册/登录/重置限制`：控制同一 IP 每小时操作次数。

## 权限模型

默认安装会创建两个用户组：

- `管理员`：拥有全部权限，第一个用户 ID 为 `1`
- `会员`：普通注册用户组

后台中可配置：

- 用户组是否允许进入后台
- 用户组是否允许管理用户和内容
- 版块允许哪些用户组浏览、发帖、回帖

超级管理员用户 `#1` 受到特殊保护，不能被普通管理员删除、禁用或降权。

## 备份与升级

升级或修改代码前，至少备份：

```text
data/
cache/
index.php
index.css
index.js
install.php
```

最重要的是 `data/*.sqlite` 和 `data/db.php`。缓存文件可以重建，但数据库文件不可丢失。

## 常见问题

### 提示“请先安装”

说明 `data/install.lock` 不存在。访问 `install.php` 完成初始化。

### 伪静态 URL 访问 404

确认：

- 后台已启用伪静态
- Apache 已开启 `mod_rewrite` 并允许 `.htaccess`
- Nginx/Caddy 已配置 fallback 到 `index.php`

### 找回密码链接域名不对

到后台设置里填写 `站点地址`，例如：

```text
https://forum.example.com
```

### 邮件发送失败

确认服务器支持 PHP `mail()`，或在服务器层配置可用的邮件发送服务。调试时可以启用虚拟发送，但虚拟发送只适合本地环境。

## 安全建议

- 生产环境使用 HTTPS
- 禁止公网访问 `data/` 和 `cache/`
- 安装后保留 `data/install.lock`
- 定期备份 SQLite 数据库
- 不要把真实数据库、缓存或本地配置提交到公开仓库
- 后台配置 `站点地址`，避免重置密码链接依赖请求 Host

