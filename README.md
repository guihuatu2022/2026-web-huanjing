# 岁月博客 · 部署仓库

这个仓库**不含博客代码**，只负责一件事：**让 GitHub 帮你构建运行时**。

构建出来的东西是**一个文件**，里面装着：

| 内含 | 作用 |
|---|---|
| Caddy | 网页服务器（自动申请 HTTPS 证书） |
| PHP 8.3 | 跑博客程序 |
| naiveproxy | 代理（forwardproxy 模块） |
| Cloudflare DNS 插件 | 用 DNS 方式申请 SSL 证书 |

**一个文件，本地和生产都能用。**这就是选它的最大好处 —— 本地 PHP 和服务器 PHP 是同一个二进制，不会有"本地好使、上线就崩"的问题。

---

## 为什么不用 `apt install php`

可以，但不推荐。因为那样**本地 PHP 和服务器 PHP 是两套东西**，版本、扩展、编译选项都可能不一样。

用同一个单文件，两边**字节级一致**。

---

# 第一部分 · 让 GitHub 构建（你现在要做的）

## 第 1 步 · 在 GitHub 建一个空仓库

1. 打开 https://github.com/new
2. 仓库名填 **`blog-deploy`**
3. 选 **Public**（公开仓库的 Actions 构建**不限时长、不收费**）
4. **不要**勾选 "Add a README file"
5. 点 Create repository

## 第 2 步 · 把本目录的文件传上去

在你这台电脑的终端里，进到本目录（就是有 `README.md` 和 `.github` 的这个文件夹），依次执行：

```bash
git init
git add -A
git commit -m "feat: 构建配置"
git branch -M main
git remote add origin https://github.com/你的用户名/blog-deploy.git
git push -u origin main
```

> 把 `你的用户名` 换成你的 GitHub 用户名。

## 第 3 步 · 点一下开始构建

1. 回到 GitHub 仓库页面
2. 点上方 **Actions** 标签
3. 左边选 **构建运行时单文件**
4. 右边点 **Run workflow** → 再点绿色的 **Run workflow**
5. 等 30~90 分钟（可以去干别的）

> **第一次构建有可能失败。**这很正常 —— 这类构建依赖上游项目的脚本，偶尔会因为上游改动而报错。失败了把红色日志发给我，我来改。

## 第 4 步 · 下载产物

构建成功后：

1. 点仓库右边的 **Releases**
2. 找到最新那条
3. 下载 **`frankenphp-linux-x86_64`**（约 50~80 MB）

---

# 第二部分 · 本地跑起来

下载完，在终端里：

```bash
# 1. 给执行权限（只需一次）
chmod +x frankenphp-linux-x86_64

# 2. 看看它能不能用
./frankenphp-linux-x86_64 version
./frankenphp-linux-x86_64 php-cli -v
```

看到 PHP 版本号和扩展列表就成功了。

## 平时开发用这两个命令

| 要做的事 | 命令 |
|---|---|
| **启动网站** | `./frankenphp-linux-x86_64 php-server -r public` |
| **跑某个脚本** | `./frankenphp-linux-x86_64 php-cli app/seed.php` |

`php-server` 就是 `php -S` 的等价物 —— 本地开发服务器，监听 `http://localhost:8000`。

> **注意路径**：这两个命令要在**博客项目目录**下跑（就是有 `app/` `public/` 的那个目录），不是在本目录。

## 建议：把它放到项目里，少打字

```bash
# 在博客项目目录下
mkdir -p bin
cp /下载路径/frankenphp-linux-x86_64 bin/
chmod +x bin/frankenphp
```

之后就可以：

```bash
bin/frankenphp php-server -r public      # 启动
bin/frankenphp php-cli app/seed.php      # 建库
```

---

# 第三部分 · 装到 VPS（M11 阶段补全）

`install.sh` 和 `templates/Caddyfile.tpl` 还没写 —— 它们属于 M11（部署阶段）。

到时候会做成**一条命令**：

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/你的用户名/blog-deploy/main/install.sh)
```

脚本会自动：识别系统 → 下载对应架构的运行时 → 拉博客代码 → 生成配置 → 建库 → 装成开机自启服务。

---

# 常见问题

| 现象 | 原因 | 怎么办 |
|---|---|---|
| `Permission denied` | 没给执行权限 | `chmod +x frankenphp-linux-x86_64` |
| `cannot execute binary file` | 架构不对 | 确认 VPS/电脑是 x86_64：`uname -m` |
| Actions 里红叉 | 上游构建脚本变了 | 把日志发我，我来改 |
| 构建卡在 1 小时以上 | 正常，PHP 要从源码编译 | 耐心等，别重复触发 |
| 下载太慢 | GitHub 在国内慢 | 用代理，或让 Actions 打包成更小的压缩包 |
| `php-server` 起来了但页面 404 | 路径不对 | 确认在博客目录下、`-r public` 指向正确 |

---

# 文件说明

```
blog-deploy/
├─ .github/workflows/build.yml   ← 构建配置（GitHub 读这个）
├─ install.sh                     ← 待写（M11）
├─ templates/                     ← 待写（M11）
│  ├─ Caddyfile.tpl
│  └─ blog.service
└─ README.md                      ← 你正在看的这份
```
