---
title: "Hugo构建个人博客（一）：基本安装配置"
tags: ["Hugo"]
categories: ["数字生活"]
date: 2023-05-17T18:30:58+08:00
draft: true
---

Hugo号称世界上建站最快的框架，刚好新开了一台VPS，决定在这台VPS上使用Hugo建立一个个人博客好好体验一下。


首先要做的事情自然是安装配置。


## 安装Hugo

安装最新版本的Go

```bash
wget https://go.dev/dl/go1.20.4.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

安装Extended版本的Hugo
```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.111.3/hugo_extended_0.111.3_linux-amd64.deb
dpkg -i hugo_extended_0.111.3_linux-amd64.deb
```

## 使用Hugo创建站点

```bash
hugo new site mysite
cd mysite
git init
```

### 设置`ananke`主题
运行`hugo mod init github.com/glfruit/snail`

编辑`config.toml`文件，设置主题：
```toml
theme = ["github.com/theNewDynamic/gohugo-theme-ananke"]
```

## 安装Caddy

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg 
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list 
sudo apt update 
sudo apt install caddy
```

### 参考资料
1.  [Roll Your Own Static Site Host on VPS with Caddy Server](https://austingil.com/static-host-vps/)

## Caddy基本配置

```
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

### 参考资料
- [Caddy — Configure Logging and Access Logs](https://futurestud.io/tutorials/caddy-configure-logging-and-access-logs)
- [Debian 11 / Ubuntu 22.04 安装 Caddy - 烧饼博客](https://u.sb/debian-install-caddy/)

