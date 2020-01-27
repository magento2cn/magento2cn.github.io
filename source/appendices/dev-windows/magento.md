使用 docker-compose 为每个项目部署开发所需的容器，这基于[本地开发框架](appendices/dev-windows/dev.html)。


## Github 地址

- [Magento 1.9](https://github.com/zengliwei/dev-magento/tree/1.9-dev)
- [Magento 2.1 简化版](https://github.com/zengliwei/dev-magento/tree/2.1-dev)
- [Magento 2.1 完整版](https://github.com/zengliwei/dev-magento/tree/2.1-dev-full)
- [Magento 2.2 简化版](https://github.com/zengliwei/dev-magento/tree/2.2-dev)
- [Magento 2.2 完整版](https://github.com/zengliwei/dev-magento/tree/2.2-dev-full)
- [Magento 2.3 简化版](https://github.com/zengliwei/dev-magento/tree/2.3-dev)
- [Magento 2.3 完整版](https://github.com/zengliwei/dev-magento/tree/2.3-dev-full)


## 部署内容

### 简化版

该版本包含三个容器：

- `mysql:5.7` 容器
- `redis:latest` 容器
- 基于 `php:x.x-fpm` 的自定义容器，包含以下内容：
    - MSMTP - 用于转发测试邮件到 `mailhog/mailhog` 容器
    - Nginx - 用于处理页面请求
    - SSH - 用于执行 CLI 指令，及通过 SFTP 进行文件传输


### 完整版

完整版是在简化版基础上添加了：

- `elasticsearch:6.8.6`（Magento 2.3 +）或 `elasticsearch:2.4.6`（Magento 2.1）容器
- `kibana:6.8.6`（Magento 2.3 +）或 `elastichq/elasticsearch-hq:latest`（Magento 2.1）容器
- 基于 `rabbitmq:management` 的自定义容器，自动创建 magento 用户
- 基于 `varnish:latest` 的自定义容器，添加 Magento 2 对应规则


## 部署步骤

### 创建容器

- 在本地开发框架目录下的 **projects** 文件夹中创建新文件夹，下载文件到该目录中
- 修改 **domain** 文件指定新项目的主域名，该域名会被作为域名转发配置文件的文件名
- 修改 **C:\Windows\System32\drivers\etc\hosts** 文件，添加域名映射
- 修改 **.env** 文件指定项目名，该名字将作为项目各容器名的前缀
- 通过 CMD 或 Power Shell 在项目目录下执行 `docker-compose up --no-recreate -d` 命令创建容器


### 启动容器

每次启动 docker 后须执行 **start.cmd** 文件以开启相关容器，修改域名转发配置文件后缀并重启域名转发服务。
