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

## 服务器优化

### [bbr](https://github.com/google/bbr)

- sudo vi `/etc/sysctl.conf`

```shell script
net.ipv4.tcp_congestion_control=bbr
net.core.default_qdisc=fq
```

execute `sudo sysctl -p`

### [Optimize the shadowsocks server on Linux](http://shadowsocks.org/en/config/advanced.html)

- sudo vi `/etc/security/limits.conf`

```shell script
* soft nofile 51200
* hard nofile 51200
root soft nofile 51200
root hard nofile 51200
```

Then, before you start the shadowsocks server, set the ulimit first

```shell script
ulimit -n 51200
```

- sudo vi `/etc/sysctl.conf`

```shell script
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

execute `sudo sysctl -p`

## 线路优化

- 安装 Supervisor

```shell script
sudo apt update
sudo apt install supervisor
```

- [压缩/解压](https://manpages.debian.org/buster/manpages-zh/tar.1.zh_CN.html)

```shell script
tar -xf foo.tar
tar -cf foo.tar bar/

tar -xzf foo.tar.gz
tar -czf foo.tar.gz bar/

tar -xjf foo.tar.bz2
tar -cjf foo.tar.bz2 bar/
```

- kcptun

```shell script
./server_linux_amd64 -l ":4000" -t "127.0.0.1:8388" -mode fast3 -nocomp -sockbuf 16777217 -dscp 46
./ss-server -s 0.0.0.0 -p 8388 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- UDPspeeder

```shell script
./speederv2 -s -l0.0.0.0:4096 -r 127.0.0.1:8388 -f20:10 -k "passwd"
./ss-server -s 0.0.0.0 -p 8388 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- kcptun + udp2raw

```shell script
./udp2raw_amd64 -s -l0.0.0.0:4001 -r 127.0.0.1:4000 -k "passwd" --raw-mode faketcp -a
./server_linux_amd64 -l ":4000" -t "127.0.0.1:8388" -mode fast3 -nocomp -sockbuf 16777217 -dscp 46
./ss-server -s 0.0.0.0 -p 8388 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- UDPspeeder + udp2raw

```shell script
./udp2raw_amd64 -s -l0.0.0.0:4097 -r 127.0.0.1:4096 -k "passwd" --raw-mode faketcp -a
./speederv2 -s -l0.0.0.0:4096 -r 127.0.0.1:8388 -f20:10 -k "passwd"
./ss-server -s 0.0.0.0 -p 8388 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

### kcptun

[kcptun Github](https://github.com/xtaci/kcptun)

- 下载 kcptun

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
tar -xzf kcptun-linux-amd64-20200409.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 配置 kcptun [参数优化](https://github.com/xtaci/kcptun/issues/251)

```shell script
sudo mkdir -p /etc/kcptun
sudo cp kcptun/config.json /etc/kcptun/
```

- 运行 kcptun

```shell script
sudo mkdir -p /etc/supervisor/conf.d
sudo cp supervisor/kcptun.conf /etc/supervisor/conf.d/
sudo service supervisor restart
ps aux | grep kcptun
```

### UDPspeeder

[UDPspeeder Github](https://github.com/wangyu-/UDPspeeder)

- 下载 UDPspeeder

```shell script
wget https://github.com/wangyu-/UDPspeeder/releases/download/20190121.0/speederv2_binaries.tar.gz
tar -xzf speederv2_binaries.tar.gz
sudo cp speederv2_amd64 /usr/bin/speederv2
sudo chmod +x /usr/bin/speederv2
```

- 运行 UDPspeeder [参数优化](https://github.com/wangyu-/UDPspeeder/wiki/%E6%8E%A8%E8%8D%90%E8%AE%BE%E7%BD%AE)

```shell script
sudo mkdir -p /etc/supervisor/conf.d
sudo cp supervisor/speederv2.conf /etc/supervisor/conf.d/
sudo service supervisor restart
ps aux | grep speederv2
```

### kcptun + udp2raw

[kcptun Github](https://github.com/xtaci/kcptun)
[udp2raw Github](https://github.com/wangyu-/udp2raw-tunnel)

- 下载 kcptun

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
tar -xzf kcptun-linux-amd64-20200409.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 配置 kcptun [参数优化](https://github.com/xtaci/kcptun/issues/251)

```shell script
sudo mkdir -p /etc/kcptun
sudo cp kcptun/config.json /etc/kcptun/
```

- 运行 kcptun

```shell script
sudo mkdir -p /etc/supervisor/conf.d
sudo cp supervisor/kcptun.conf /etc/supervisor/conf.d/
sudo service supervisor restart
ps aux | grep kcptun
```

- 下载 udp2raw

```shell script
wget https://github.com/wangyu-/udp2raw-tunnel/releases/download/20181113.0/udp2raw_binaries.tar.gz
tar -xzf udp2raw_binaries.tar.gz
sudo cp udp2raw_amd64 /usr/bin/udp2raw
sudo chmod +x /usr/bin/udp2raw
```

- 运行 udp2raw

```shell script
sudo mkdir -p /etc/supervisor/conf.d
sudo cp supervisor/udp2raw_kcptun.conf /etc/supervisor/conf.d/
sudo service supervisor restart
ps aux | grep udp2raw
```

### UDPspeeder + udp2raw

[UDPspeeder Github](https://github.com/wangyu-/UDPspeeder)
[udp2raw Github](https://github.com/wangyu-/udp2raw-tunnel)

- 下载 UDPspeeder

```shell script
wget https://github.com/wangyu-/UDPspeeder/releases/download/20190121.0/speederv2_binaries.tar.gz
tar -xzf speederv2_binaries.tar.gz
sudo cp speederv2_amd64 /usr/bin/speederv2
sudo chmod +x /usr/bin/speederv2
```

- 运行 UDPspeeder [参数优化](https://github.com/wangyu-/UDPspeeder/wiki/%E6%8E%A8%E8%8D%90%E8%AE%BE%E7%BD%AE)

```shell script
sudo mkdir -p /etc/supervisor/conf.d
sudo cp supervisor/speederv2.conf /etc/supervisor/conf.d/
sudo service supervisor restart
ps aux | grep speederv2
```

- 下载 udp2raw

```shell script
wget https://github.com/wangyu-/udp2raw-tunnel/releases/download/20181113.0/udp2raw_binaries.tar.gz
tar -xzf udp2raw_binaries.tar.gz
sudo cp udp2raw_amd64 /usr/bin/udp2raw
sudo chmod +x /usr/bin/udp2raw
```

- 运行 udp2raw

```shell script
sudo mkdir -p /etc/supervisor/conf.d
sudo cp supervisor/udp2raw_speederv2.conf /etc/supervisor/conf.d/
sudo service supervisor restart
ps aux | grep udp2raw
```
