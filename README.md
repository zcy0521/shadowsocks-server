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
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
```

> 执行`sudo sysctl -p`

## [Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)

- 新建`/etc/sysctl.d/local.conf`

```shell script
fs.file-max=51200
net.core.rmem_max=67108864
net.core.wmem_max=67108864
net.core.rmem_default=65536
net.core.wmem_default=65536
net.core.netdev_max_backlog=4096
net.core.somaxconn=4096
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_tw_recycle=0
net.ipv4.tcp_fin_timeout=30
net.ipv4.tcp_keepalive_time=1200
net.ipv4.ip_local_port_range=10000 65000
net.ipv4.tcp_max_syn_backlog=4096
net.ipv4.tcp_max_tw_buckets=5000
net.ipv4.tcp_fastopen=3
net.ipv4.tcp_rmem=4096 87380 67108864
net.ipv4.tcp_wmem=4096 65536 67108864
net.ipv4.tcp_mtu_probing=1
```

> 执行`sysctl --system`

## [kcptun](https://github.com/xtaci/kcptun#quickstart)

- 编辑`~/.bashrc`

```shell script
ulimit -n 65535
```

> 执行`ulimit -n 65535`

- 编辑`/etc/sysctl.conf`

```shell script
net.core.rmem_max=26214400
net.core.rmem_default=26214400
net.core.wmem_max=26214400
net.core.wmem_default=26214400
net.core.netdev_max_backlog=2048
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
