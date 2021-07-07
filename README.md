# nextcloud docker部署

## 安装docker和docker-compose
略

## 安装mysql或mariadb

```shell
docker pull mariadb #需要mysql8以上版本
```

## 安装nextcloud

```shell
docker pull nextcloud	
```

## 部署
为图床程序创建一个文件夹以存放配置文件,图片文件夹,数据库文件夹.以便方便备份和迁移.推荐直接在root目录下创建nextcloud文件夹.同时在nextcloud文件夹内创建docker-compose.yml文件

```shell
mkdir nextcloud
cd nextcloud
mkdir database
mkdir nextcloud
nano docker-compose.yml
```

yml配置文件：
```yaml
version: '2'

services:
  db:
    image: mariadb
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - ./database:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=GvaFsX7ZJGoTXc
      - MYSQL_PASSWORD=RFaLFYA97UkLhf
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud
    restart: always
    ports:
      - 45003:80
    links:
      - db
    volumes:
      - ./nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD=RFaLFYA97UkLhf
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
```

设置权限:
```shell
chmod -R 777 nextcloud
```

运行 docker-compose:
```shell
docker-compose up -d # 启动
```

## 配置域名及反向代理

配置Caddy
```json
cloud.625564348.xyz:443 {
# 域名
  tls foxljy900@gmail.com # 申请证书的邮箱

  encode gzip

  reverse_proxy 127.0.0.1:45003 # 对应端口
}
```

```shell
caddy reload # 重新加载配置
```

## 添加域名信任
```shell
nano ./nextcloud/config/config.php
```

添加域名
```php
  'trusted_domains' =>
  array (
    0 => '192.168.0.29',
    1 => 'cloud.example.com',#域名
  ),
```