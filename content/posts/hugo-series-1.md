---
title: "Hugo构建个人博客（一）：基本安装配置"
tags: ["Hugo"]
categories: ["数字生活"]
date: 2023-05-17T18:30:58+08:00
draft: false
---

Hugo号称世界上建站最快的框架，一直想找个机会体验一下，最近刚好新开了一台VPS，于是迫不及待地开始部署尝试。
整个安装配置过程比较简单，网上关于使用Hugo + Caddy部署的文章不算少，但是基本上都是基于v1的，所以还是花了点时间才算顺利部署起来。

## Caddy的安装配置
### 安装Caddy 
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg 
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list 
sudo apt update 
sudo apt install caddy
```

### 基本配置
```bash
example.com:443 {
	tls {
		protocols tls1.2 tls1.3
		ciphers TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
	}

	header {
		Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
		Referrer-Policy strict-origin-when-cross-origin
		X-Frame-Options SAMEORIGIN
		X-Content-Type-Options nosniff
		X-XSS-Protection "1; mode=block"
	}

	root * /var/www/example.com

	file_server {
		index index.html
	}

	encode gzip zstd
}
```

### 安装配置caddy-webhook
由于想实现推送到GitHub仓库后自动更新网站的效果，需要使用webhook，因此要安装配置caddy-webhook插件：

```bash
xcaddy build \
  --with github.com/WingLim/caddy-webhook
```

修改`Caddyfile`
```yaml
route /webhook {
		webhook {
			repo https://github.com/xxx/xxx.git
                        secret xxxxxxxxx
			path blog
			branch master
			command hugo --destination /var/www/blog
			submodule
		}
	}
```

## 站点的创建与部署
### 安装Hugo
#### 安装最新版本的Go
```bash
wget https://go.dev/dl/go1.20.4.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

#### 安装Extended版本的Hugo
```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.111.3/hugo_extended_0.111.3_linux-amd64.deb
dpkg -i hugo_extended_0.111.3_linux-amd64.deb
```

### 使用Hugo创建站点

```bash
hugo new site mysite
cd mysite
git init
```

### 设置主题
`hugo`创建的站点并没有默认的主题，需要自己安装配置，我选择的是[Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke)主题，安装很简单，运行：
```bash
hugo mod init github.com/glfruit/snail
```

然后编辑`config.toml`文件，设置主题：
```toml
theme = ["github.com/theNewDynamic/gohugo-theme-ananke"]
```

### 站点其它设置
还是编辑`config.toml`文件，进行站点信息和菜单的基本设置：
```toml
baseURL = 'http://xxx.xx/'
defaultContentLanguage = "zh"
languageCode = "zh-CN"
DefaultContentLanguage = "zh"
hasCJKLanguage = true
title = '站点名称'

[languages]
  [languages.zh]
    title = "站点名称"
    weight = 1
    contentDir = "content"

sectionPagesMenu = 'main'
[menu]
  [[menu.main]]
    name = '首页'
    pageRef = '/'
    weight = 10
  [[menu.main]]
    identifier='posts'
    name = '文章'
    pageRef = '/posts'
    weight = 20
  [[menu.main]]
    name = '分类'
    pageRef = '/categories'
    weight = 30
  [[menu.main]]
    name = '关于'
    pageRef = '/about'
    weight = 40

[params]
  custom_css = ["custom.css"] # 自定义样式，放在站点目录的assets/ananke/css目录下
  author = "Your Name"
  date_format = ":date_full" # 日期格式
  favicon = ""
  site_logo = ""
  description = "Your site description"
  # choose a background color from any on this page: https://tachyons.io/docs/themes/skins/ and preface it with "bg-"
  background_color_class = "bg-black"
  recent_posts_number = 3
  show_reading_time = true # 显示文章阅读所需的时间
```

## 参考资料
1.  [Roll Your Own Static Site Host on VPS with Caddy Server](https://austingil.com/static-host-vps/)
2. [Caddy — Configure Logging and Access Logs](https://futurestud.io/tutorials/caddy-configure-logging-and-access-logs)
3. [Debian 11 / Ubuntu 22.04 安装 Caddy - 烧饼博客](https://u.sb/debian-install-caddy/)
4. [caddyserver/xcaddy: Build Caddy with plugins](https://github.com/caddyserver/xcaddy)
5. [WingLim/caddy-webhook: Caddy v2 module for serving a webhook.](https://github.com/WingLim/caddy-webhook)