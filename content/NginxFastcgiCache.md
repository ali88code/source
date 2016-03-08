+++
draft = true
title = "Nginx_Fastcgi_Cache + Lua 執行過程"

+++
# Nginx Proxy Cache #

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

1. Nginx Cache loader

 - 主要Nginx 一開始初始化時,有呼叫Nginx Cache 相關模組,Nginx master process 會 fork Nginx Cache loader 去抓取 Nginx Cache Store(cache 儲存區),來重新建立索引(hash key)並存放在memory 中,
	   建立完成後, Cache loader process 就隨即結束。

2. Nginx Cache mamager

 - 主要接受 master process 指令 去管理 Nginx Cache 大小事務,像是 Hash key 過期,或是搜尋 此筆資料是否存放在 cache 儲存區中 等工作。

![Ngnx Cache運作流程](images/cache1-4_small.jpeg "Nginx_Cache")

![Ngnx Cache Manager](images/60.26_master_worker_cachemanager_process.PNG "Nginx_Cache_Manager")

# Nginx Fastcgi Cache 簡要參數說明 #

```
http{
....
fastcgi_cache_path /dev/shm/fastcgicache levels=1:2 keys_zone=democache:10m  max_size=1000m inactive=60s;
server {
...
	location ~  ^/other_sport/A0017_0050/(.*) {
		access_log /var/log/nginx/localhost-demo_cache-access.log  main;
		error_log  /var/log/nginx/localhost-demo_cache-error.log;
		try_files	$uri	$uri/  =404;
    	        fastcgi_pass    127.0.0.1:9001;
    	        fastcgi_index   index.php;
    	        include         fastcgi_params;
		set $prefix_fastcgi_name  $1;
		if ($query_string != ""){ set $prefix_fastcgi_name $1?$query_string;}
		
		include	    /etc/nginx/conf.d/skip_cache_new;
		fastcgi_cache_key "$scheme$request_method$host$request_uri";
		fastcgi_cache_use_stale error timeout invalid_header http_500;
		fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
		
		fastcgi_cache_bypass $skip_cache;
		fastcgi_no_cache $skip_cache;
		fastcgi_cache democache;
		fastcgi_cache_valid  1m;

	}
....

    location /123.html {
	internal;
	set  $memc_cmd $arg_cmd;
        set $memc_key  $arg_key;
        memc_pass 127.0.0.1:11211;

    }


}

....
}

```

### fastcgi_cache_path ###
- /dev/shm/fastcgicache   **儲存cache的地方** 

- levels=1:2     	  **設定目錄階層**

```
/dev/shm/fastcgicache
|-- 1
|   `-- a7
|       `-- a5225f331f10bbe6099bfe0e48e82a71
|-- 6
|   `-- 8f
`-- a
    `-- fe
```

- keys_zone=democache:10m   				**為cache 空間設定名字為 democache , 此空間10MB  **
 - 10MB 是儲存在 share memory,,比較常存取(hot data)會被儲存在share memory

- max_size=1000m					 **最大使用1G空間 ,是指定儲存cache的容量**

- inactive=60s			   			**cache 1分鐘,預設值10分鐘**

### fastcgi_cache_key ###

- 將 "$scheme$request_method$host$request_uri" 用MD5方法來 當作 hash key

### fastcgi_cache ###

- 指定 key_zone 名字為 democache

- 參數: inactive  ,指在一段時間內文件沒被存取使用,就會被緩存管理器自動刪除。 ,預設 10m(10分鐘)


### fastcgi_cache_valid ###

指定接收到回應程式 的 http code 快取時間

- fastcgi_cache_valid [http code ...] time;

- 若只有指定時間。預設只有 http code  :200, 301, and 302 等回應狀態會被快取
ex: 當Nginx 收到回應程式 http 狀態碼 200 時,就快取 1 小時

### fastcgi_cache_use_stale ###

- 定義那些情況使用過期cache

 - invalid_header  無效的回應
 
 - error	   回應的Header 發生錯誤
 
 - http_500	   回應500錯誤

 - time		   當Nginx 與 Application 建立連接時,Nginx 等待 Application 回應超時


### fastcgi_ignore_headers ###

- 忽略 Http header 帶有 Cache-Control , Expires ,Set-Cookie 

- 若從後端 Application回應 http header 帶有Cache-Control , Expires ,Set-Cookie 會影響fastcgi_cache 不會cache 回應資料

### fastcgi_no_cache ###

- 不等於空值 及 值不等於 0 都不被 cache。 **只要是 0 或 空值 就指定 cache**

### fastcgi_cache_bypass ###

- 不等於空值 及 值不等於 0 ,即是 不從cache區 回應資料給Client端,而是 By_Pass 到後端 。 **只要是 0 或 空值 就指定 cache**

## $upstream_cache_status 包含以下幾種狀態 ##

| 狀態     |      說明     |
|----------|:-------------:|
|MISS |未命中，請求被傳送到後端|
|HIT |緩存命中|
|EXPIRED |緩存已經過期請求被傳送到後端|
|UPDATING |正在更新緩存，將使用舊的應答|
|STALE |後端將得到過期的應答|
|BYPASS |request會By_Pass到後端伺服器上,因為請求匹配了一個proxy_cache_bypass 或是 fastcgi_cache_bypass|

# 執行Nginx + lua 之前 需要了解的一些事 #

### 載入 lua 模組, 測試是否可以運作 ###

  1-1.
  
   ```
    dso{
        	path /etc/nginx/modules/;
		load ngx_http_lua_module.so;
       }
    http{
	  .....
	}
   ```

   ```sh
    #nginx -t
    #/etc/init.d/nginx reload
   ```

   驗證lua 模組是否有載入
   
   ```sh
    #nginx -m
   ```

   ```
  	...
	...
	ngx_http_lua_module (shared, 3.1)
	...
   ```

  1-2. 測試 lua 模組是否可用

  ```
	http{
		....
		server{
		....
		location /lua3{
			default_type text/html;
			set $RRR I come from Taiwan;
			content_by_lua 'ngx.say("<h1><a href=http://tw.yahoo.com>",ngx.var.RRR,"</a></h1>")';
		}
		.....
		}
	..
	}
  ``` 

### Nginx hander處理模組將 http request 劃分11個階段來處理,及了解每個階段執行順序。 ###

 2-1. 

  - 先由Nginx核心 讀取 http Header

  - 根據Nginx設定檔,Nginx 核心 挑選一個Hander模組 處理 Http 請求 ,並將 http Header 交給 Hander 模組

  - 由Hander 模組 讀取 Http Body ,並處理好 Http 請求

  - Hander 模組處理Http 請求後,再 發送給 Fliters 模組處理。

  - Fliter 模組處理好後,再發送給客戶端

 

![Http請求處理流程圖](images/Http_Request_Nginx_small.png "Http_Request_Nginx")

 2-2. Handers模組 精細處理 Http 請求 ,分成 11 個階段處理。下表所列處理階段分別依序處理

  Handers階段    |   說明
  -------------  | -------------
  NGX_HTTP_POST_READ_PHAS   | 	讀取請求內容階段
  NGX_HTTP_SERVER_REWRITE_PHASE  | 	Server請求位址重寫階段
  NGX_HTTP_FIND_CONFIG_PHASE   |	配置查找階段:
  NGX_HTTP_REWRITE_PHASE   |	Location請求位址重寫階段
  NGX_HTTP_POST_REWRITE_PHASE   |	請求位址重寫提交階段
  NGX_HTTP_PREACCESS_PHASE   |	訪問許可權檢查準備階段
  NGX_HTTP_ACCESS_PHASE   |		訪問許可權檢查階段
  NGX_HTTP_POST_ACCESS_PHASE   |	訪問許可權檢查提交階段
  NGX_HTTP_TRY_FILES_PHASE   |	配置項try_files處理階段  
  NGX_HTTP_CONTENT_PHASE   |	內容產生階段
  NGX_HTTP_LOG_PHASE   |	        日誌模組處理階段


# ngx_http_memc_module.so 模組 #

使Nginx  能利用 memcached 相關指令 (set ,get ,stat..等等)存取memcached server

## Install ##

```sh
# cd /use/local/src;git clone https://github.com/openresty/memc-nginx-module.git
# dso_tool --dst=/etc/nginx/modules/ --add-module=/usr/local/src/memc-nginx-module/
```

## 修改設定檔 ,載入此模組 (/etc/nginx/modules/ngx_http_memc_module.so ) ##

```
dso {
 ....
      load ngx_http_memc_module.so;
}
http{
....
}
```

```sh
#nginx -t && /etc/init.d/nginx reload
```

## Nginx access memcached ##
- 先啟動 memcached ,並插入一些資料 ,這邊測試,插入 andy 和 amy
 
 ```sh
  #echo "Google" > andy
  #echo  "Good"  > amy
  #memcp --verbose --debug --servers=127.0.0.1 --expire=120  andy
  #memcp --verbose --debug --servers=127.0.0.1 --expire=120  amy
 ```
 
- 測試抓取 andy , amy  key 的值 

 ```sh
  curl 'http://192.168.0.3/memt.html?cmd=get&key=andy'
  curl 'http://192.168.0.3/memt.html?cmd=get&key=amy'
 ``` 

- 為了能安全取得 memcached key 值

  可以不要讓外部直接利用Nginx  存取 memcached Server,需要修改 nginx 設定檔

  ```
  http{
    ...
    server {
	location /memt.html {
		internal;
		set  $memc_cmd $arg_cmd;
		set $memc_key  $arg_key;
		memc_pass 127.0.0.1:11211;
	}
    }
  }
  ```

- 測試 
 
  目標結果 : 404 沒找到

 ```sh
   curl 'http://192.168.0.3/memt.html?cmd=get&key=andy'
 ```








# 自行撰寫lua流程說明 

* 先判斷是否有 帶 cookie ; 若沒帶 則 By_Pass 後端php-cgi處理

* 再確定 cookie 是否有帶 "WEBSESSID=" 字串 ; 若沒帶 則 By_Pass 後端php-cgi處理

* 有帶cookie , 利用(memcached API For Nginx) 讀取 cookie值

* 確定cookie 值的內容 是否有帶登入資訊 ;若沒帶 則By_Pass 後端php-cgi處理

* 有帶登入資訊, 是否為列表要cache 的對象 ; 若不是 則By_Pass 後端php-cgi處理


# 簡單演示 及 問題說明 #

## 演示 ##

## 問題說明 ##

- 說明背景:

```
http://220.229.227.26/other_sport/A0017_0050/om/ 
http://220.229.227.26/other_sport/A0017_0050/wlb/
兩個位址都在同一個DocumentRoot /var/www/html/other_sport/A0017_0050/
兩個位址都在同一個Nginx設定檔中 /etc/nginx/conf.d/00-default.conf
```

- 產生問題執行:

```
用Firefox 瀏覽此url 並登入 http://220.229.227.26/other_sport/A0017_0050/om/
執行設定會被cache的url http://220.229.227.26/other_sport/A0017_0050/om/league-listing.php (看 /var/log/nginx/localhost-demo_cache-access.log 是否有hit到,有被Hit到 就馬上執行下一個步驟)
用IE 瀏覽此url 並登入 http://220.229.227.26/other_sport/A0017_0050/wlb/
再瀏覽此url http://220.229.227.26/other_sport/A0017_0050/om/league-listing.php,就發生未登入即可以抓取的到頁面。
```


# 參考資料 #

 - [Module ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
 
 - [Nginx Caching Tutorial - You Can Run Faster](http://czerasz.com/2015/03/30/nginx-caching-tutorial/) 
 
 - [Nginx的緩存：proxy_cache和fastcgi_cache](http://blog.angryfox.com/?p=1930)

 - [Nginx模塊fastcgi_cache的幾個注意點](https://www.cnxct.com/several-reminder-in-nginx-fastcgi_cache-and-php-session_cache_limiter/)

 - [Nginx模塊和請求處理流程簡介](http://281816327.blog.51cto.com/907015/1619920)

 - [handler模組的掛載](https://github.com/chiamingyen/nginxProject/blob/master/nginxProject/source/chapter_3.rst)

 - [nginx與lua的執行順序和步驟說明](http://www.pylist.com/topic/1445510269)

 - [Nginx API for Lua](https://github.com/openresty/lua-nginx-module#nginx-api-for-lua)

 - [nil、null與ngx.null](http://www.pureage.info/2013/09/02/125.html)
