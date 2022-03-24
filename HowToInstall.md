---
> 2022/3
> https://www.mediawiki.org/wiki/Manual:Installing_MediaWiki
---

# 安裝流程
- 一起裝在 nextcloud 的虛擬機
- apache/SQL/PHP 大部分偷懶直接照抄 Nextcloud

## 建立 mediawiki 使用的資料庫
- 以 root 使用者登入 MySQL
```
sudo /etc/init.d/mysql start    # 啟動 MySQL 服務
sudo mysql -uroot -p            # 以 root 使用者登入
# 第一次以 root 登入時，會提示設定登入密碼
# 不輸入任何密碼直接 enter 進入 == 不設定密碼
```
這時應該會出現 `MariaDB [root]>` 在畫面中
> 雖然我實際運作看到的是 `MariaDB [none]>`

### 建立普通使用者並初始化資料庫
```
CREATE USER 'wikiuser'@'localhost' IDENTIFIED BY 'password';
CREATE DATABASE IF NOT EXISTS wikidb CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON wikidb.* TO 'wikiuser'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
# username/password 可以修改成自己喜歡的，不過我直接照抄
quit;
```

## 下載 MediaWIKI本體

## 解壓縮移到 `/var/www/`，把原本的 html 資料夾改成其他名字或砍掉，把剛剛下載的移來命名成 html
- 重新命名成 html 為重點，不知道為啥一定要改成 html 才會運作
## 設定 apache server
### 在 /etc/apache2/sites-available 建立 mediawiki.conf 檔案
- 直接抄 nextcloud 的設定檔
- 砍掉大部分設定，只留下 DocumentRoot/ServerName
- 記得修改 DocumentRoot 的路徑(/var/www/html/)
```
sudo cp /etc/apache2/sites-available/nextcloud.conf /etc/apache2/sites-available/mediawiki.conf
sudo vim /etc/apache2/sites-available/mediawiki.conf
```

### 啟動剛設定好的網站
```
sudo a2ensite mediawiki.conf
systemctl reload apache2
```

## 啟動瀏覽器，連線到 http://[IP of nextcloud]
### 連線到資料庫
* 對應前面建立的資料庫
- 資料庫主機                             : localhost
- 資料庫名稱                             : wikidb
- 資料庫資料表名稱的字首                 : wikidb
- 資料庫使用者名稱                       : wikiuser
- 資料庫密碼                             : password
### 供網頁存取使用的資料庫帳號
看來只能使用與安裝程序相同的帳號

## 完成後把 LocalSettings.php 丟進資料夾，設好權限
## 大概就完成了

# 最佳化
## short URL
### mediawiki.conf of apache2
新增以下規則
```
# Enable the rewrite engine
RewriteEngine OnRewriteRule ^/?wiki(/.*)?$ %{DOCUMENT_ROOT}/index.php [L]
# Short URL for wiki pages
RewriteRule ^/?wiki(/.*)?$ %{DOCUMENT_ROOT}/index.php [L]
# Redirect / to Main Page
RewriteRule ^/*$ %{DOCUMENT_ROOT}/index.php [L]
```
### LocalSettings.php
額外新增 $wgArticlePath 項目
```
$wgScriptPath = "/";    # this should already have been configured this way
$wgArticlePath = "/wiki/$1";
```

## SSL
### 先用 cerbot 搞定 443 連線
- 80, 443 port 都需要打開
- 這台虛擬機已經從 snap 裝好 cerbot 了
- 但不知道為啥一定要指定全路徑，不能用軟連結
```
sudo /snap/bin/certbot --apache
# 會要我輸入數字選擇哪個網址加入 HTTPS
# 輸入數字按下 enter 之後就全部自動跑完了
```

### 測試自動更新 SSL
```
sudo certbot renew --dry-run
```

### 調整 LocalSettings.php
- 強制使用 HTTPS
```
$wgServer = 'https://wiki.bouzzing.com';    # 改成 https 開頭
$wgForceHTTPS = true;                       # 新增這項
```

## LocalSettings.php 雜七雜八
```
$wgServer           # 設定 wiki 的運作位址
$wgLocaltimezone    # 時區，改成 Asia/Taipei
$wgLogos            # 設定 Logo，額外上傳 Strawberry_wiki.png 並指定
$wgFavicon          # 設定網頁上小圖
```