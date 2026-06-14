# 宝塔面板部署指南

## 准备工作

先在本地 push 最新代码到 GitHub：

```bash
cd "e:\Program Develop\silentmoon\New silentmoon"
git add -A && git commit -m "update" && git push
```

---

## 方案 A：本地构建上传（最简单）

不用在服务器装任何东西。

### 1. 本地构建

```powershell
cd "e:\Program Develop\silentmoon\New silentmoon"
hugo --minify
Compress-Archive -Path public\* -DestinationPath silentmoon.top.zip
```

### 2. 宝塔上传

1. 宝塔面板 → **文件** → 进入 `/www/wwwroot/silentmoon.top/`
2. 上传 `silentmoon.top.zip` → 右键解压到当前目录
3. 确认 `index.html` 在根目录下

### 步骤 3-4 看下面「宝塔站点配置」即可，以后每次更新重复步骤 1-2 覆盖上传。

---

## 方案 B：服务器 Git + Hugo 自动构建（推荐）

一次配置，以后本地 push 完服务器自动更新。

### B1. SSH 连上服务器

```bash
ssh root@你的服务器IP
```

### B2. 安装 Hugo Extended

宝塔面板 → **终端**，粘贴运行：

```bash
# 下载 Hugo（AMD64 架构）
cd /tmp
wget https://github.com/gohugoio/hugo/releases/download/v0.163.1/hugo_extended_0.163.1_linux-amd64.tar.gz
tar -xzf hugo_extended_0.163.1_linux-amd64.tar.gz
mv hugo /usr/local/bin/hugo

# 验证
hugo version
# 输出：hugo v0.163.1+extended linux/amd64 ...
```

> ARM 服务器（如树莓派、某些云服务器）把 `amd64` 换成 `arm64`。

### B2.5 安装 Git（如果没有）

```bash
# 检查是否已装
git --version

# CentOS / RHEL
yum install -y git

# Ubuntu / Debian
apt install -y git

# 配置（替换为你的信息）
git config --global user.name "Mistymoon"
git config --global user.email "mistymoon555@outlook.com"
```

### B3. 克隆仓库并构建

项目源码放在 `/opt/` 下（不对外暴露），构建产物输出到网站根目录。

```bash
# 创建源码目录
mkdir -p /opt && cd /opt

# 克隆 Hugo 项目
git clone https://github.com/mistymoonz/silentmoon-blog.git
cd silentmoon-blog

# 构建到网站根目录（只输出 public 内容）
hugo --minify -d /www/wwwroot/silentmoon.top
```

构建完检查：

```bash
ls /www/wwwroot/silentmoon.top/index.html   # 应该有这个文件
ls /www/wwwroot/silentmoon.top/hugo.toml    # 不应该有这个，源码不会泄露
```

### B4. 宝塔站点配置

1. 宝塔面板 → **网站** → **添加站点**
2. 填写：
   - **域名**：你的域名（如 `silentmoon.top`）
   - **根目录**：`/www/wwwroot/silentmoon.top`
   - **PHP 版本**：选择 **纯静态**
3. 点击 **提交**

### B5. SSL 证书

1. 网站列表 → 点击域名 → **SSL**
2. 选择 **Let's Encrypt** → **申请证书**
3. 勾选 **强制 HTTPS** → 保存

### B6. 伪静态

网站 → 点击域名 → **伪静态**，粘贴：

```nginx
location / {
    try_files $uri $uri/ =404;
}
error_page 404 /404.html;

location ~* \.(css|js|jpg|jpeg|png|gif|ico|webp|svg|woff|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}

location ~ /\. {
    deny all;
}
```

### B7. 设置 Git 自动拉取（免密）

```bash
# 生成 SSH 密钥
ssh-keygen -t ed25519 -C "blog-deploy" -N "" -f ~/.ssh/id_ed25519

# 查看公钥
cat ~/.ssh/id_ed25519.pub
```

复制输出的公钥，到 **GitHub → Settings → Deploy keys → Add deploy key**，粘贴，勾选 **Allow write access**。

然后改成 SSH 拉取：

```bash
cd /opt/silentmoon-blog
git remote set-url origin git@github.com:mistymoonz/silentmoon-blog.git
```

### B8. 宝塔计划任务（自动更新）

宝塔面板 → **计划任务** → **添加任务**：

| 字段 | 值 |
|------|-----|
| 任务名称 | 自动更新博客 |
| 任务类型 | Shell 脚本 |
| 执行周期 | 每 5 分钟 |
| 脚本内容 | 见下方 |

```bash
#!/bin/bash
cd /opt/silentmoon-blog

# 拉取最新代码
git pull origin main 2>&1 | grep -v "Already up to date" || exit 0

# 有新代码才构建到网站根目录
/usr/local/bin/hugo --minify -d /www/wwwroot/silentmoon.top
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Blog updated successfully"
```

保存后点 **执行** 测试一下，看日志输出是否正常。

---

## 测试

浏览器打开你的域名，检查：

- 首页 Hero 显示背景图和标题
- 导航栏、暗色模式正常
- `/posts/`、`/about/`、`/friends/` 可访问
- 标签筛选正常

## 日常使用

### 方案 A（手动上传）

写完文章 → 本地 `hugo --minify` → 压缩 `public/` → 宝塔上传解压

### 方案 B（自动部署）✅

本地写好文章 → push 到 GitHub → 最多等 5 分钟，服务器自动拉取构建上线。也可以手动到宝塔计划任务点 **执行** 立即更新。
