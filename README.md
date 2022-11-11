# compose-v2ray

v2ray+caddy+tls在docker上部署，使用docker-compose管理

## 前置操作

* ## 登录服务器(当前centos8)时区及设置时区

  ```sh
  $ date //查看时间
  ```

  ```sh
  $ timedatectl list-timezones|grep Shanghai
  Asia/Shanghai
  ```

  ```sh
  $ sudo timedatectl set-timezone Asia/Shanghai
  ```



* ## 安装bbr加速

  ###### 如没有则需安装

  ```sh
  $ sudo yum install -y wget
  ```

  ```sh
  $ sudo wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && sudo chmod +x bbr.sh && sudo ./bbr.sh
  ```

  #### 查看是否生效

  ```sh
  $ lsmod|grep bbr
  tcp_bbr                20480  1
  ```

  

* ##  安装git

  ```sh
  $ sudo yum install -y git
  ```

## 安装docker

###### 官方文档(https://docs.docker.com/engine/install/centos/)

###### 脚本安装：curl -sSL https://get.docker.com | bash

```sh
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

```sh
$ sudo yum install -y yum-utils
```

```sh
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```sh
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

## 查看docker版本、及添加当前用户到docker组

```sh
$ docker -v
```

```sh
$ sudo usermod -aG docker $USER
```

## 安装docker-compose

```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

```sh
$ docker-compose -v
```

## 拉取github上的脚本

###### 如无git，则需要安装(sudo yum install -y git)

```sh
$ git clone https://github.com/Dingzhenpiyi/compose-v2ray.git
```

###### 进入compose-v2ray, 并创建文件夹

```sh
$ cd compose-v2ray
$ mkdir -p config data log/caddy log/v2ray
```

## 修改配置文件

```sh
$ vim Caddyfile
修改为你的域名
{ 
  log {
    output file /var/log/caddy/caddy.log
  }
  tls charles@hotmail.com
  @websockets {
    header Connection Upgrade
    header Upgrade websocket
  }
  reverse_proxy @websockets v2ray:10000 //端口需与v2ray.json配置一致
}
```

```sh
$ vim v2ray.json
{
 "log": {
  "loglevel": "warning",
  "access": "/var/log/v2ray/access.log",
  "error": "/var/log/v2ray/error.log"
 },
  "inbounds": [
    {
      "port": 10000, //与 caddy里对应
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "修改为你生成的uuid",                                                                                                                                                   
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/v2ray&$"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

```sh
$ vim docker-compose.yml
version: '3'
services:
  v2ray:
    image: v2fly/v2fly-core
    container_name: v2ray
    volumes:
      - ./v2ray.json:/etc/v2ray/config.json
      - ./log/v2ray:/var/log/v2ray
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    expose:
      - 10000 //与v2ray提供的端口一致
  caddy:
    image: caddy
    container_name: caddy
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./log/caddy:/var/log/caddy
      - ./caddy/data:/data
      - ./caddy/config:/config
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "443:443"
      - "80:80"
```

## 启动、停止

```sh
$  sudo docker-compose up -d
```

```sh
$ sudo docker-compose down/restart
```

## docker开机启动

```sh
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

