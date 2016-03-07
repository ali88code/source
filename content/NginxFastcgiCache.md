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

# Nginx Cache 運作流程 #

Nginx 除了 work Process 管理 Http Request 請求等其他工作外,Nginx Cache 也由另外程序來作管理。

*. Nginx Cache loader

 - 主要Nginx 一開始初始化時,有呼叫Nginx Cache 相關模組,Nginx master process 會 fork Nginx Cache loader 去抓取 Nginx Cache Store(cache 儲存區),來重新建立索引(hash key)並存放在memory 中,
	   建立完成後, Cache loader process 就隨即結束。

*. Nginx Cache mamager

 - 主要接受 master process 指令 去管理 Nginx Cache 大小事務,像是 Hash key 過期,或是搜尋 此筆資料是否存放在 cache 儲存區中 等工作。

![Ngnx Cache運作流程](images/cache1-4_small.jpeg "Nginx_Cache")

![Ngnx Cache Manager](images/60.26_master_worker_cachemanager_process.PNG "Nginx_Cache_Manager")





# 參考資料 #

 - [Module ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
 
 - [Nginx Caching Tutorial - You Can Run Faster](http://czerasz.com/2015/03/30/nginx-caching-tutorial/) 
 
 - [Nginx的緩存：proxy_cache和fastcgi_cache](http://blog.angryfox.com/?p=1930)

 - [Nginx模塊和請求處理流程簡介](http://281816327.blog.51cto.com/907015/1619920)

 - [handler模組的掛載](https://github.com/chiamingyen/nginxProject/blob/master/nginxProject/source/chapter_3.rst)

 - [nginx與lua的執行順序和步驟說明](http://www.pylist.com/topic/1445510269)
