# 宝塔面板部署指南

## 1. 本地构建

```bash
cd "e:\Program Develop\silentmoon\New silentmoon"
hugo --minify
```

产物在 `public/` 目录，压缩成一个 zip：

```bash
# PowerShell
Compress-Archive -Path public\* -DestinationPath silentmoon-blog.zip
```

## 2. 宝塔面板操作

### 2.1 上传文件

1. 打开宝塔面板 → **文件**
2. 进入 `/www/wwwroot/`，新建目录 `silentmoon-blog`
3. 点击 **上传**，选择 `silentmoon-blog.zip`
4. 上传完成后右键 zip → **解压** 到当前目录
5. 解压后确认 `index.html` 直接在 `/www/wwwroot/silentmoon-blog/` 下（不是 `/www/wwwroot/silentmoon-blog/public/`）

### 2.2 添加站点

1. 宝塔面板 → **网站** → **添加站点**
2. 填写：
   - **域名**：你的域名（如 `silentmoon.top`），没有域名填服务器 IP
   - **根目录**：选择 `/www/wwwroot/silentmoon-blog`
   - **PHP 版本**：选择 **纯静态**（不需要 PHP）
3. 点击 **提交**

### 2.3 配置 SSL（可选）

1. 网站列表 → 点击域名 → **SSL**
2. 选择 **Let's Encrypt** → **申请证书**
3. 勾选 **强制 HTTPS** → 保存

## 3. 配置伪静态（URL 重写）

网站 → 点击域名 → **伪静态**，粘贴以下内容：

```nginx
# Nginx（宝塔默认）
location / {
    try_files $uri $uri/ =404;
}

# 404 页面
error_page 404 /404.html;

# 静态资源缓存 30 天
location ~* \.(css|js|jpg|jpeg|png|gif|ico|webp|svg|woff|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}

# 禁止访问隐藏文件
location ~ /\. {
    deny all;
}
```

保存。

## 4. 测试

浏览器打开你的域名，检查：
- 首页 Hero 显示背景图和标题
- 导航栏和暗色模式正常
- `/posts/` `/about/` `/friends/` 页面可访问
- 标签筛选功能正常

## 5. 后续更新

每次改完内容后：

```bash
# 本地构建
hugo --minify

# 压缩
Compress-Archive -Path public\* -DestinationPath silentmoon-blog.zip

# 宝塔面板 → 文件 → 上传 zip → 解压覆盖
```

如果文章改得频繁，也可以直接在服务器上装 Hugo，用 Git 拉代码构建：

```bash
# 服务器上
cd /www/wwwroot/silentmoon-blog
git pull
hugo --minify -d .
```
