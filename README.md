# shadowsocks-server

## Shadowsocks

[Github](https://github.com/shadowsocks/shadowsocks-libev)

### Debian 9

- 安装 Shadowsocks

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

## [kcptun](https://github.com/xtaci/kcptun)

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

- 下载`kcptun`

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
tar -xzf kcptun-linux-amd64-20200409.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 配置`kcptun` [参数优化](https://github.com/xtaci/kcptun/issues/251)

```shell script
sudo mkdir /etc/kcptun
sudo cp kcptun/config.json /etc/kcptun/config.json
sudo vi /etc/kcptun/config.json
"listen": ":4000"
"target": "127.0.0.1:8388"
```

- 运行`kcptun`服务

> ./kcptun -c /etc/kcptun/config.json &

```shell script
sudo apt install supervisor
sudo cp supervisor/kcptun.conf /etc/supervisor/conf.d/
sudo service supervisor restart
```

## [udp2raw](https://github.com/wangyu-/udp2raw-tunnel)

- 修改`kcptun`配置

```shell script
sudo vi /etc/kcptun/config.json
"listen": ":4000"
"target": "127.0.0.1:8388"
"mtu": 1300
```

- 下载`udp2raw`

```shell script
wget https://github.com/wangyu-/udp2raw-tunnel/releases/download/20181113.0/udp2raw_binaries.tar.gz
tar -xzf udp2raw_binaries.tar.gz
sudo cp udp2raw_amd64 /usr/bin/udp2raw
sudo chmod +x /usr/bin/udp2raw
```

- 运行`udp2raw`服务

> ./udp2raw_amd64 -s -l0.0.0.0:4001 -r 127.0.0.1:4000 -k "passwd" --raw-mode faketcp -a &>/var/log/udp2raw.log &

```shell script
sudo apt install supervisor
sudo cp supervisor/udp2raw_kcptun.conf /etc/supervisor/conf.d/
sudo service supervisor restart
```
