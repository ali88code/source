+++
date = "2016-01-24T09:03:05-05:00"
draft = true
title = "Hugo"

+++
## 安裝hugo

* 使用官網釋出的二進檔 [hugo下載處](https://github.com/spf13/hugo/releases)


* 使用Binary安裝

```sh
cd /usr/local/src 
wget  https://github.com/spf13/hugo/releases/download/v0.15/hugo_0.15_linux_amd64.tar.gz
tar zxvf  hugo_0.15_linux_amd64.tar.gz  -C /usr/bin/
cd /usr/bin/
ln -s hugo_0.15_linux_amd64/hugo_0.15_linux_amd64  hugo
```

## hugo快速開始

* 建立框架到自訂的路徑

```sh
hugo new site  /var/www/html/ali88/ 
```

* 建立內容

```sh
cd /var/www/html/ali77/
hugo new  hugo.md
```

* 編輯內容 /var/www/html/ali77/content/hugo.md

```
+++
date = "2016-01-24T03:49:06-05:00"
draft = true
title = "hugo"

+++
```

* Install some themes  [官方themes 下載路徑](https://github.com/spf13/hugoThemes)

```sh
cd /var/www/html/ali77/
git clone https://github.com/Zenithar/hugo-theme-crisp  themes/crisp
```

* 簡單開啟一個web server 測試

預設hugo 會開啟 127.0.0.1:1313。為了方便透過外網IP(假設為 188.166.241.76:1313)連接到內部Lookback IP 127.0.0.1:1313,這邊使用
iptable DNAT機制的port  forwarding來達到此目的。

```bash
iptables  -I PREROUTING -p tcp -m tcp --dport 1313 -j DNAT --to-destination 127.0.0.1:1313
sysctl -w net.ipv4.conf.all.route_localnet=1
```

使用hugo server 指令

```sh
cd /var/www/html/ali77/
hugo server  --buildDrafts --theme=crisp --baseURL="http://188.166.241.76/"
```

# 參考資料

* [Hugo Quickstart Guide](http://gohugo.io/overview/quickstart/)
* [利用 Hugo & GitHub 搭建個人博客靜態網站](http://blog.bpcoder.com/2015/12/hugo-create-blog/)

