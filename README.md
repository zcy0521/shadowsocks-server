# shadowsocks-server

## 安装

### Linux

[Github](https://github.com/shadowsocks/shadowsocks-libev)

- Debian 9

We strongly encourage you to install shadowsocks-libev from `stretch-backports`.

```shell script
sudo sh -c 'printf "deb http://deb.debian.org/debian stretch-backports main" > /etc/apt/sources.list.d/stretch-backports.list'
sudo apt update
sudo apt -t stretch-backports install shadowsocks-libev
```

- 配置 Shadowsocks

```shell script
sudo vim /etc/shadowsocks-libev/config.json
{
    "server":"0.0.0.0",
    "mode":"tcp_and_udp",
    "server_port":8388,
    "local_port":1080,
    "password":"HelloWorld!",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```

- 运行 Shadowsocks

```shell script
sudo /etc/init.d/shadowsocks-libev start    # for sysvinit
sudo systemctl start shadowsocks-libev      # for systemd
```

### Docker

[DockerHub](https://hub.docker.com/r/shadowsocks/shadowsocks-libev)

[Github](https://github.com/shadowsocks/shadowsocks-libev/tree/master/docker/alpine)

- 下载镜像

```shell script
sudo docker pull shadowsocks/shadowsocks-libev
```

- 运行容器

```shell script
git clone https://github.com/zcy0521/shadowsocks-server.git
cd shadowsocks-server
sudo docker-compose up -d
```

## [bbr](https://github.com/google/bbr)

- 编辑`/etc/sysctl.conf`

```shell script
sudo vi /etc/sysctl.conf
net.ipv4.tcp_congestion_control=bbr
net.core.default_qdisc=fq
```

> 执行`sudo sysctl -p`

## [Optimize the shadowsocks server on Linux](http://shadowsocks.org/en/config/advanced.html)

- 编辑`/etc/security/limits.conf`

```shell script
sudo vi `/etc/security/limits.conf`
* soft nofile 51200
* hard nofile 51200
root soft nofile 51200
root hard nofile 51200
```

> 执行`ulimit -n 51200`

- 编辑`/etc/sysctl.conf`

```shell script
$ sudo vi `/etc/sysctl.conf`
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```

> 执行`sudo sysctl -p`

## kcptun

[kcptun Github](https://github.com/xtaci/kcptun)

- 下载`kcptun`

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
tar -xzf kcptun-linux-amd64-20200409.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 配置`kcptun` [参数优化](https://github.com/xtaci/kcptun/issues/251)

```shell script
sudo mkdir -p /etc/kcptun
sudo cp kcptun/config.json /etc/kcptun/
```

- 运行`kcptun`服务

> ./kcptun -c /etc/kcptun/config.json &

```shell script
sudo apt install supervisor
sudo cp supervisor/kcptun.conf /etc/supervisor/conf.d/
sudo service supervisor restart
```
