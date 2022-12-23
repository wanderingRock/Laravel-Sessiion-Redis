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


(參考連結:https://nicokie0420.medium.com/laradock-%E5%AF%A6%E8%A3%9D-redis-%E6%9C%8D%E5%8B%99%E6%95%99%E5%AD%B8%E6%96%87%E4%BB%B6-5cd4ef0268f5https://nicokie0420.medium.com/laradock-%E5%AF%A6%E8%A3%9D-redis-%E6%9C%8D%E5%8B%99%E6%95%99%E5%AD%B8%E6%96%87%E4%BB%B6-5cd4ef0268f5)



