# Laravel-Sessiion-Redis

簡略說明 laravel 提供了幾種session的儲存方式

file - sessions are stored in storage/framework/sessions.  => 儲存於storage/framework/sessions資料夾內   

cookie - sessions are stored in secure, encrypted cookies. => 儲存在cookies

database - sessions are stored in a relational database. => 儲存於資料庫內

memcached / redis - sessions are stored in one of these fast, cache based stores. => 儲存於redis 資料庫緩存的方式

dynamodb - sessions are stored in AWS DynamoDB. => 儲存於 DynamoDB NoSQL資料庫服務 來自Amazon

array - sessions are stored in a PHP array and will not be persisted. => Session 被儲存到一個 PHP 陣列中且不會被保留 !!!不建議使用在正式通常用在測試環境

## 選擇

laravel 文件有說明

默認情況下，Laravel 配置為使用file會話驅動程序，這對許多應用程序都適用。   

如果您的應用程序將在多個 Web 服務器之間進行負載平衡，您應該選擇一個所有服務器都可以訪問的集中存儲，

例如 Redis 或 database。 

所以我們這次選擇使用 redis 方式來存儲session

## 實作

### Step.1 安裝redis  

測試環境是使用laradock  

laradock 安裝 redis 步驟 

進入 \laradock\redis  修改 redis.conf  

把 bind 127.0.0.1 註解掉以及 protected-mode yes 改為 no   

以及把 requirepass 註解打開，後面輸入你想設定的密碼

```
...
# bind 127.0.0.1   

protected-mode no

requirepass 123456
```

接下來編輯 redis 資料夾裡面的 Dockerfile


```
FROM redis:latest

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

## For security settings uncomment, make the dir, copy conf, and also start with the conf, to use it
#RUN mkdir -p /usr/local/etc/redis
#COPY redis.conf /usr/local/etc/redis/redis.conf

VOLUME /data

EXPOSE 6379

#CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
CMD ["redis-server"]
```
之後執行
```
laradock up -d redis

or

docker-compose up -d redis
```
接著檢查密碼是否安裝完成與密碼是否設定成功

進入 redis 容器，或是開啟終端機

輸入 

```
redis-cli

or

redis-cli -p 6379
```
能成功進入後用auth + 密碼開通

```
auth 輸入你的密碼

```
如果回應是 OK 一切準備就緒

參考連結:https://nicokie0420.medium.com/laradock-%E5%AF%A6%E8%A3%9D-redis-%E6%9C%8D%E5%8B%99%E6%95%99%E5%AD%B8%E6%96%87%E4%BB%B6-5cd4ef0268f5https://nicokie0420.medium.com/laradock-%E5%AF%A6%E8%A3%9D-redis-%E6%9C%8D%E5%8B%99%E6%95%99%E5%AD%B8%E6%96%87%E4%BB%B6-5cd4ef0268f5)

### Step.2 laravel redis 設定

在laravel中有phpredis與predis兩種可以選擇，  
phpredis是用C寫的PHP擴充套件，predis是用PHP寫的，   
以效能來說當然是phpredis比較好，    
不過其實兩者速度上沒有差很多，還是要依專案需求來選擇。     
對於使用Laravel來講，可以直接用composer來安裝predis，   

```
composer require predis/predis
```
設定 config/database.php  

```
...
'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        'default' => [
            // 'url' => env('REDIS_URL'),  
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'username' => env('REDIS_USERNAME'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

        'cache' => [
            // 'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'username' => env('REDIS_USERNAME'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

],
...
```
說明:   
● default/cache中的url都註解掉，這是用一行url的方式設定才需要。  
● 在程式中use Illuminate\Support\Facades\Redis;，會使用預設(default)的資料庫0。  


設定 .env 

```
...
CACHE_DRIVER=redis
SESSION_DRIVER=redis
REDIS_HOST=redis
REDIS_PASSWORD=123456
REDIS_PORT=6379
REDIS_CLIENT=predis
...
```
說明:   
● Laravel預設使用檔案(file)儲存快取(cache)，若要改用redis，
  記得把env file中的CACHE_DRIVER從原本的file改為redis， 
  session也是同理(SESSION_DRIVER=file)。  
   
● 因為我們使用laradock所以 REDIS_HOST 要從原本的127.0.0.1 改成 redis
● REDIS_CLIENT predis 是新加上去的

這邊完成後進頁面沒跳出錯誤，基本就可以使用了

### Step.3 使用方法

```
use Illuminate\Support\Facades\Redis;
Redis::set('name', 'Vic');
Redis::get('name');
```

剩下更完整的說明可參考連結
參考連結: https://vocus.cc/article/625ba2bdfd89780001a72207


