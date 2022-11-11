# compose-v2ray

v2ray+caddy+tls在docker上部署，使用docker-compose管理

## 前置操作

* ## 登录服务器(当前centos8)时区及设置时区

  ```sh
  $ date -R  //查看当前时区
  ```

  ```sh
  $ rm -rf /etc/localtime  //删除本地的时间文件
  ```

  ```sh
  $ ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  //加入上海时区
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

