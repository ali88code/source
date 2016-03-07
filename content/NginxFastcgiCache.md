+++
draft = true
title = "Nginx_Fastcgi_Cache + Lua 執行過程"

+++
# Nginx Proxy Cache#

 - 應用於Reverse Proxy,負載均衡
 - 作用是緩存後端服務器的內容，可能是任何內容，包括靜態的和動態
 - 另外,也可作正向代理 (Nginx 這邊支援 web proxy)

![Nginx_Proxy_Cache](images/nginx_cache.jpeg "Nginx_Proxy_Cache")

# Nginx Fastcgi Cache #

 - proxy_cache 與 fastcgi_cache功能大至一樣。
 - 用是緩存fastcgi生成的內容，很多情況是php生成的動態的內容。
 - 緩存作用。減少了nginx與Application(php,python,cgi等等)通信的次數。

![Nginx_Fastcgi_Cache](images/nginx_Fastcgi_cache.jpeg "Nginx_Fastcgi_Cache")


# 參考資料 #

 - [Module ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
 
 - [Nginx Caching Tutorial - You Can Run Faster](http://czerasz.com/2015/03/30/nginx-caching-tutorial/) 
 
 - [Nginx的緩存：proxy_cache和fastcgi_cache](http://blog.angryfox.com/?p=1930)

 - [Nginx模塊和請求處理流程簡介](http://281816327.blog.51cto.com/907015/1619920)

 - [handler模組的掛載](https://github.com/chiamingyen/nginxProject/blob/master/nginxProject/source/chapter_3.rst)

 - [nginx與lua的執行順序和步驟說明](http://www.pylist.com/topic/1445510269)
