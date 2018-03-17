CTFd-uWSGI-Nginx
===
## Host Environment
- ubuntu 16.04

## Install
### 更新 ubuntu 套件
```
$ sudo apt-get update
```

### 下載並啟動資料庫
```
$ sudo apt-get install mariadb-server -y
$ sudo systemctl start mysql            # 啟動資料庫
$ sudo systemctl enable mariadb         # 設定開機自動啟用
$ sudo mysql_secure_installation        # 執行 mysql 的安全性設定
```

### 進入資料庫建立其他使用者 [參考資料](https://blog.gtwang.org/linux/mysql-create-database-add-user-table-tutorial/)
```
$ mysql -u root -p
> CREATE USER '<user>'@'localhost' IDENTIFIED BY '<password>';
> GRANT ALL PRIVILEGES ON <database>.* TO '<user>'@'localhost';
```

### 下載 python 相關套件
```
$ sudo apt-get install python-mysqldb -y
$ sudo apt-get install python-pip -y
$ pip install --upgrade pip
$ pip install Flask
$ pip install uwsgi -I

```

### 下載 CTFd
```
$ sudo apt-get install git -y
$ git clone https://github.com/CTFd/CTFd.git
```

### 安裝 CTFd
```
$ cd CTFd
$ ./prepare.sh
```

### 修改 `CTFd/CTFd/config.py`
將`SQLALCHEMY_DATABASE_URI`參數改為：`SQLALCHEMY_DATABASE_URI = 'mysql://<db帳號>:<db密碼>@localhost/CTFd?charset=utf8mb4'`。並將`HOST`参數改為`HOST = "<serverIP>"`
```
$ cd CTFd
$ vi config.py
```

### 確認 python 的 flask 框架是否能啟動，連接 uwsgi 前請務必關掉
```
$ cd ..
$ sudo python serve.py # 啟動 CTFd
```

### 安裝 nginx
```
$ sudo apt-get install nginx -y
```

### 打開 Nginx 的配置文件，Ubuntu上默認在`/etc/nginx/sites-available/default`
```
server {
    listen 80;
    listen [::]:80;
    
    listen 443 ssl;
    listen [::]:443 ssl;
    
    root /usr/share/nginx/html;
    server_name helloctf.com.tw;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/tmp/uwsgi.sock;
    }
}
```

### 檢查設定檔是否有誤
```
$ sudo nginx -t
```

### 重啟 nginx
```
$ sudo nginx -s reload
```

### 回到 `/CTFd` 利用 uwsgi 將把服務掛到 nginx
```
$ sudo uwsgi -s /tmp/uwsgi.sock -w "CTFd:create_app()" --chmod-socket=666
```
