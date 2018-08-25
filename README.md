
# docker for magento 1.9
为 Magento `1.9.x.x` 准备的docker环境，可能`1.7.x`，`1.8.x`也能用



## 安装 docker
参见docker 官方文档



## 创建项目文件夹
```shell
mkdir magento-domain-1/ && cd magento-domain-1/
```



## 安装 docker-magento

### 克隆
```shell
git clone https://github.com/goodwong/docker-magento
```

### 配置
```shell
cd .docker-compose
cp .env.example .env
```
默认配置即可运行，如果有多个magento站点运行，分别修改一下变量为不同的值：
- COMPOSE_PROJECT_NAME  
- NGINX_HOST_HTTP_PORT  
- NGINX_HOST_HTTPS_PORT  
- DB_ADMINER_PORT  

> 生产环境下，建议修改数据库密码  

### 运行
```shell
cd .docker-compose
docker-compose up -d web

# 如果需要查看日志，增加：
docker-compose logs -f
```



## 安装magento

### 全新magento代码
```shell
tar jxf .docker-compose/magento-1.9.3.9-2018-06-27-02-47-24.tar -C public/
```
然后在浏览器里访问 `http://你的服务器ip或域名:nginx端口号`
安装过程中，配置数据库：
> 数据库地址：`db`  
> 数据库用户名：`app`<默认,可在.env文件修改>  
> 数据库密码：`app`<默认,可在.env文件修改>  
> 数据库名称：`app`<默认,可在.env文件修改>  

### 使用现有的magento代码
1. 解压代码
2. 修改app/etc/local.xml数据库信息：
> 数据库地址：`db`  
> 数据库用户名：`app`<默认,可在.env文件修改>  
> 数据库密码：`app`<默认,可在.env文件修改>  
> 数据库名称：`app`<默认,可在.env文件修改>  

## 导入/管理数据库
1. 通过`adminer`的web界面操作
    ```shell
    docker-compose up adminer
    ```
    打开浏览器 http://IP地址:<DB_ADMINER_PORT>  
    `Server`填写`db`  
2. 进入mysql容器
    ```shell
    docker-compose exec db bash
    mysql -u root -p
    ```
