### Fork From https://github.com/litespeedtech/lsws-magento-docker-env And Fix bugs

# LiteSpeed Magento / WordPress Docker Container

使用容器部署一个运行在LiteSpeed Enterprise 上的 Magneto2 站点，基础镜像为Ubuntu 22.04

### 运行条件
1. 最低2核心4G内存，生产环境推荐4核心16G内存
2. [安装 Docker](https://www.docker.com/)

## 配置文件
编辑 `.env` 更改 Demo 站点的域名、数据库。
查看 [Docker hub Tag page](https://hub.docker.com/repository/docker/litespeedtech/litespeed/tags) 以使用指定的LiteSpedd和PHP版本。

## 安装
克隆此Repo，或者现在源码到一个空目录:
```
git clone https://github.com/picassochan/magento2-lsws.git
```
打开终端, `cd` 切换至含有 `docker-compose.yml` 的目录，并运行:
```
docker compose up -d
```

## 组件
此Docker镜像将安装以下组件:

|Component|Version|
| :-------------: | :-------------: |
|Base System|Ubuntu 22.04|
|LiteSpeed|[Latest version](https://www.litespeedtech.com/products/litespeed-web-server/download)|
|MariaDB|[Stable version: 10.4](https://hub.docker.com/_/mariadb)|
|PHP|[Latest version](http://rpms.litespeedtech.com/debian/)|
|LiteSpeed Cache for Wordpress|[Latest from WordPress.org](https://wordpress.org/plugins/litespeed-cache/)|
|LiteMage Cache for Magento2|[LiteMage from LiteSpeedtech.com](https://www.litespeedtech.com/products/cache-plugins/magento-acceleration)|
|ACME|[Latest from ACME official](https://github.com/acmesh-official/get.acme.sh)|
|Magento2|[2.4.2](https://devdocs.magento.com/guides/v2.4/release-notes/open-source-2-4-2.html)|
|elasticsearch-kibana|[7.17.9]([https://hub.docker.com/_/elasticsearch](https://hub.docker.com/r/nshou/elasticsearch-kibana/))|
|WordPress|[Latest from WordPress](https://wordpress.org/download/)|
|phpMyAdmin|[Latest from dockerhub](https://hub.docker.com/r/bitnami/phpmyadmin/)|

## Data Structure
Cloned project 
```bash
├── acme
├── bin
│   └── container
├── data
│   └── db
├── logs
│   ├── access.log
│   ├── error.log
│   ├── lsrestart.log
│   └── stderr.log
├── lsws
│   ├── admin-conf
│   └── conf
├── sites
│   └── localhost
├── LICENSE
├── README.md
└── docker-compose.yml
```

  * `acme` contains all applied certificates from Lets Encrypt

  * `bin` contains multiple CLI scripts to allow you add or delete virtual hosts, install applications, upgrade, etc 

  * `data` stores the MySQL database

  * `logs` contains all of the web server logs and virtual host access logs

  * `lsws` contains all web server configuration files

  * `sites` contains the document roots (the WordPress application will install here)

## Usage
### Starting a Container
Start the container with the `up` or `start` methods:
```
docker compose up
```
You can run with daemon mode, like so:
```
docker compose up -d
```
The container is now built and running. 

Note: The container will auto-apply a 15-day trial license. Please contact LiteSpeed to extend the trial, or apply your own license, [starting from $0](https://www.litespeedtech.com/pricing).

### Stopping a Container
```
docker compose stop
```
### Removing Containers
To stop and remove all containers, use the `down` command:
```
docker compose down
```
### Setting the WebAdmin Password
We strongly recommend you set your personal password right away.
```
bash bin/webadmin.sh my_password
```
### Starting a Demo Site
After running the following command, you should be able to access the Magento + LiteMage Cache installation with the configured domain. By default the domain is http://localhost.
```
bash bin/demosite.sh -M
```
or with Sample data installed
```
bash bin/demosite.sh -M -S
```
### Creating a Domain and Virtual Host
```
bash bin/domain.sh [-A, --add] example.com
```
> Please ignore SSL certificate warnings from the server. They happen if you haven't applied the certificate.
### Deleting a Domain and Virtual Host
```
bash bin/domain.sh [-D, --del] example.com
```
### Creating a Database
You can either automatically generate the user, password, and database names, or specify them. Use the following to auto generate:
```
bash bin/database.sh [-D, --domain] example.com
```
Use this command to specify your own names, substituting `user_name`, `my_password`, and `database_name` with your preferred values:
```
bash bin/database.sh [-D, --domain] example.com [-U, --user] USER_NAME [-P, --password] MY_PASS [-DB, --database] DATABASE_NAME
```
### Installing a Magento Site
To preconfigure the Magento2, run the `database.sh` script for your domain, before you use the following command to install Magento:
```
./bin/appinstall.sh [-A, --app] magento [-D, --domain] example.com
```
We can also install with sample data
```
./bin/appinstall.sh [-A, --app] magento [-D, --domain] example.com [-S, --sample]
```
### Installing a WordPress Site
To preconfigure the `wp-config` file, run the `database.sh` script for your domain, before you use the following command to install WordPress:
```
./bin/appinstall.sh [-A, --app] wordpress [-D, --domain] example.com
```
### Installing ACME 
We need to run the ACME installation command the **first time only**. 
With email notification:
```
./bin/acme.sh [-I, --install] [-E, --email] EMAIL_ADDR
```
### Applying a Let's Encrypt Certificate
Use the root domain in this command, and it will check for a certificate and automatically apply one with and without `www`:
```
./bin/acme.sh [-D, --domain] example.com
```

Other parameters:

  * [`-r`, `--renew`]: Renew a specific domain with -D or --domain parameter if posibile. To force renew, use -f parameter.

  * [`-R`, `--renew-all`]: Renew all domains if possible. To force renew, use -f parameter.  

  * [`-f`, `-F`, `--force`]: Force renew for a specific domain or all domains. 

  * [`-v`, `--revoke`]: Revoke a domain.  

  * [`-V`, `--remove`]: Remove a domain.   

### Updating Web Server
To upgrade the web server to latest stable version, run the following:
```
bash bin/webadmin.sh [-U, --upgrade]
```
### Applying OWASP ModSecurity
Enable OWASP `mod_secure` on the web server: 
```
bash bin/webadmin.sh [-M, --mod-secure] enable
```
Disable OWASP `mod_secure` on the web server: 
```
bash bin/webadmin.sh [-M, --mod-secure] disable
```
>Please ignore ModSecurity warnings from the server. They happen if some of the rules are not supported by the server.
### Applying license to LSWS
Apply your license with command:
```
bash bin/webadmin.sh [-S, --serial] YOUR_SERIAL
```
Apply trial license to server with command:
```
bash bin/webadmin.sh [-S, --serial] TRIAL
```

### Accessing the Database
After installation, you can use phpMinAdmin to access the database by visiting http://127.0.0.1:8080 or https://127.0.0.1:8443. The default username is `root`, and the password is the same as the one you supplied in the `.env` file.

## Customization
If you want to customize the image by adding some packages, e.g. `lsphp74-pspell`, just extend it with a Dockerfile. 
1. We can create a `custom` folder and a `custom/Dockerfile` file under the main project. 
2. Add the following example code to `Dockerfile` under the custom folder
```
FROM litespeedtech/litespeed:latest
RUN apt-get update && apt-get install lsphp74-pspell
```
3. Add `build: ./custom` line under the "image: litespeedtech" of docker-composefile. So it will looks like this 
```
  litespeed:
    image: litespeedtech/litespeed:${LSWS_VERSION}-${PHP_VERSION}
    build: ./custom
```
4. Build and start it with command:
```
docker compose up --build
```

## Support & Feedback
If you still have a question after using LiteSpeed Docker, you have a few options.
* Join [MagentoChina.org](https://forum.magentochina.org) for real-time discussion

**Pull requests are always welcome**
