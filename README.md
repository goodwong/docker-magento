
# docker for magento 1.9
为 Magento `1.9.x.x` 准备的docker环境，可能`1.7.x`，`1.8.x`也能用



## 安装 docker & docker-compose
参见docker 官方文档  
https://docs.docker.com/install/linux/docker-ce/ubuntu/  
https://docs.docker.com/compose/install/#master-builds  



## 创建项目文件夹
```shell
# 项目文件夹
mkdir magento-domain-1/ && cd magento-domain-1/

# magento 1.x 代码文件夹
mkdir public/
```

### 文件夹结构
```
magento-domain-1/
├── .docker-compose/ # <---- docker-compose 运行及数据文件夹
│   ├── adminer
│   ├── db
│   ├── docker-compose.yml
│   ├── .env
│   ├── .env.example
│   ├── .git
│   ├── .gitignore
│   ├── nginx
│   ├── php-fpm
│   └── README.md
└── public/ # <------------- magento 1.x 代码文件夹
    ├── api.php
    ├── app
    ├── cron.php
    ├── cron.sh
    ├── dev
    ├── downloader
    ├── error_log
    ├── errors
    ├── favicon.ico
    ├── get.php
    ├── .htaccess
    ├── .htaccess.sample
    ├── includes
    ├── index.php
    ├── index.php.sample
    ├── install.php
    ├── js
    ├── lib
    ├── LICENSE_AFL.txt
    ├── LICENSE.html
    ├── LICENSE.txt
    ├── mage
    ├── media
    ├── php.ini.sample
    ├── RELEASE_NOTES.txt
    ├── shell
    ├── skin
    └── var
```


## 安装 docker-magento

### 克隆
```shell
cd magento-domain-1/
git clone https://github.com/goodwong/docker-magento .docker-compose
```

### 配置
```shell
cd .docker-compose/
cp .env.example .env
```
默认配置即可运行，如果有多个magento站点运行，分别修改以下变量为不同的值：
- `COMPOSE_PROJECT_NAME`=  
- `NGINX_HOST_HTTP_PORT`=  
- `DB_ADMINER_PORT`=  

> 生产环境下，建议修改数据库密码  
> - `DB_ROOT_PASSWORD`=  
> - `DB_PASSWORD`=  
> - `DB_USER`=  
> - `DB_NAME`=  

### 运行
```shell
cd .docker-compose/
docker-compose up -d web

# 如果需要查看日志，增加：
docker-compose logs -f
```



## 安装magento

### 全新magento代码
```shell
cd magento-domain-1/
cd public/
# tar jxf .docker-compose/magento-1.9.3.9-2018-06-27-02-47-24.tar -C public/
# 文件过大，自行下载吧。。。
```
然后在浏览器里访问 `http://你的服务器ip或域名:nginx端口号`
安装过程中，配置数据库： 
> 数据库地址：`db`  
> 数据库用户名：`root` 或.env文件`<DB_USER>`  
> 数据库密码：见.env文件 `<DB_ROOT_PASSWORD>`或`<DB_PASSWORD>`  
> 数据库名称：见.env文件 `<DB_NAME>`，可不填  

### 使用现有的magento代码
1. 解压代码至 `public/`文件夹
2. 修改`app/etc/local.xml`数据库信息：
    > 数据库地址：`db`  
    > 数据库用户名：见.env文件`<DB_USER>`  
    > 数据库密码：见.env文件`<DB_PASSWORD>`  
    > 数据库名称：见.env文件 `<DB_NAME>`  

## 设置文件权限
```shell
cd .docker-compose/
docker-compose exec php-fpm bash
chown -R www-data:www-data public/var/ public/media/
```

## 导入/管理数据库
有两种方式管理数据库：
- 方法一，通过`adminer`的web界面操作
    ```shell
    cd .docker-compose/
    docker-compose up adminer
    ```
    打开浏览器 http://IP地址:<DB_ADMINER_PORT>  
    `Server`地址填写`db`  


- 方法二，进入mysql容器，使用命令行界面  
    首先将数据文件解压并放在`项目文件夹`下
    ```shell
    cd .docker-compose/
    docker-compose exec db bash
    cd /var/www/html/
    mysql -p magento < database_backup_file.sql
    #           ^                ^
    #        数据库名         数据库备份文件
    #       见.env文件
    ```

## 多站点多项目
> 注意分别修改`.env文件`里端口变量为不同的值  

建议使用`nginx`作前端机，配置代理规则
```nginx
# 宿主机
# ／etc/nginx/sites-available/domain-1.conf

server {
    listen 80;
    listen [::]:80;

    server xxx.com www.xxx.com;

    location / {
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:8888/; # <--- 末尾必须有/符号
                                           # <--- 端口号见<NGINX_HOST_HTTP_PORT> 变量
    }
}
```
