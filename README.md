# shadowsocks-server

## 安装

### Linux

[Github](https://github.com/shadowsocks/shadowsocks-libev#debian--ubuntu)

- 安装 Shadowsocks

```shell script
sudo apt update
sudo apt install git
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
mkdir -p ~/build-area/
cp ./scripts/build_deb.sh ~/build-area/
cd ~/build-area
./build_deb.sh
```

- 配置 Shadowsocks

```shell script
sudo vim /etc/shadowsocks-libev/config.json
```

- 运行

```shell script
sudo /etc/init.d/shadowsocks-libev start    # for sysvinit, or
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

```shell script
sudo vi /etc/sysctl.conf
net.ipv4.tcp_congestion_control=bbr
net.core.default_qdisc=fq

sudo sysctl --system
```

### [Optimizing-Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)

```shell script
sudo vi /etc/sysctl.d/local.conf
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.rmem_default = 65536
net.core.wmem_default = 65536
net.core.netdev_max_backlog = 4096
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla

sudo sysctl --system
```

## 线路优化

### Shadowsocks

- Server

```shell script
./ss-server -s 0.0.0.0 -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- Client

```shell script
./ss-redir -l 1234 -s [SS_SERVER_IP] -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305 -t
./ss-redir -l 1234 -s [SS_SERVER_IP] -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305 -u
```

### kcptun

[Github](https://github.com/xtaci/kcptun)

- Server

```shell script
./server_linux_amd64 -l ":4000" -t "127.0.0.1:443" -mode fast3 -nocomp -sockbuf 16777217 -dscp 46
./ss-server -s 0.0.0.0 -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- Client

```shell script
./ss-redir -l 1234 -s 127.0.0.1 -p 8388 -k "HelloWorld!" -m chacha20-ietf-poly1305 -t
./client_linux_amd64 -l ":8388" -r "[KCP_SERVER_IP]:4000" -mode fast3 -nocomp -autoexpire 900 -sockbuf 16777217 -dscp 46
```

### kcptun + udp2raw

[Github](https://github.com/wangyu-/udp2raw-tunnel)

- Server

```shell script
./udp2raw_amd64 -s -l0.0.0.0:4001 -r 127.0.0.1:4000 -k "passwd" --raw-mode faketcp -a
./server_linux_amd64 -l ":4000" -t "127.0.0.1:443" -mode fast3 -nocomp -sockbuf 16777217 -dscp 46
./ss-server -s 0.0.0.0 -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- Client

```shell script
./ss-redir -l 1234 -s 127.0.0.1 -p 8389 -k "HelloWorld!" -m chacha20-ietf-poly1305 -t
./client_linux_amd64 -l ":8389" -r "127.0.0.1:4000" -mode fast3 -nocomp -autoexpire 900 -sockbuf 16777217 -dscp 46
./udp2raw_amd64 -c -l0.0.0.0:4000 -r[UDP_SERVER_IP]:4001 -k "passwd" --raw-mode faketcp -a
```

### UDPspeeder

[Github](https://github.com/wangyu-/UDPspeeder)

- Server

```shell script
./speederv2 -s -l0.0.0.0:4096 -r 127.0.0.1:443 -f20:10 -k "passwd"
./ss-server -s 0.0.0.0 -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- Client

```shell script
./ss-redir -l 1234 -s 127.0.0.1 -p 8390 -k "HelloWorld!" -m chacha20-ietf-poly1305 -u
./speederv2 -c -l0.0.0.0:8390 -r[UDP_SERVER_IP]:4096 -f20:10 -k "passwd"
```

### UDPspeeder + udp2raw

[Github](https://github.com/wangyu-/udp2raw-tunnel)

- Server

```shell script
./udp2raw_amd64 -s -l0.0.0.0:4097 -r 127.0.0.1:4096 -k "passwd" --raw-mode faketcp -a
./speederv2 -s -l0.0.0.0:4096 -r 127.0.0.1:443 -f20:10 -k "passwd"
./ss-server -s 0.0.0.0 -p 443 -k "HelloWorld!" -m chacha20-ietf-poly1305
```

- Client

```shell script
./ss-redir -l 1234 -s 127.0.0.1 -p 8391 -k "HelloWorld!" -m chacha20-ietf-poly1305 -u
./speederv2 -c -l0.0.0.0:8391 -r127.0.0.1:4096 -f20:10 -k "passwd"
./udp2raw_amd64 -c -l0.0.0.0:4096 -r[UDP_SERVER_IP]:4097 -k "passwd" --raw-mode faketcp -a
```

## 服务器配置

- 安装 Supervisor

```shell script
sudo apt update
sudo apt install supervisor
ps aux | grep kcptun
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

### kcptun

- 下载 kcptun

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
tar -xzf kcptun-linux-amd64-20200409.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 配置 kcptun [参数优化](https://github.com/xtaci/kcptun/issues/251)

```shell script
sudo cp kcptun/server/config.json /etc/config/kcptun.json
```

- 运行

```shell script
sudo cp supervisor/kcptun.conf /etc/supervisor/conf.d/kcptun.conf
sudo service supervisor restart
```

### kcptun + udp2raw

- 下载 kcptun

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
tar -xzf kcptun-linux-amd64-20200409.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 下载 udp2raw

```shell script
wget https://github.com/wangyu-/udp2raw-tunnel/releases/download/20181113.0/udp2raw_binaries.tar.gz
tar -xzf udp2raw_binaries.tar.gz
sudo cp udp2raw_amd64 /usr/bin/udp2raw
sudo chmod +x /usr/bin/udp2raw
```

- 配置 kcptun

```shell script
sudo cp kcptun/server/config.json /etc/kcptun/config.json
```

- 运行

```shell script
sudo cp supervisor/kcptun.conf /etc/supervisor/conf.d/kcptun.conf
sudo cp supervisor/udp2raw_kcptun.conf /etc/supervisor/conf.d/udp2raw_kcptun.conf
sudo service supervisor restart
```

### UDPspeeder

- 下载 UDPspeeder

```shell script
wget https://github.com/wangyu-/UDPspeeder/releases/download/20190121.0/speederv2_binaries.tar.gz
tar -xzf speederv2_binaries.tar.gz
sudo cp speederv2_amd64 /usr/bin/speederv2
sudo chmod +x /usr/bin/speederv2
```

- 运行

```shell script
sudo cp supervisor/speederv2.conf /etc/supervisor/conf.d/speederv2.conf
sudo service supervisor restart
```

### UDPspeeder + udp2raw

- 下载 UDPspeeder

```shell script
wget https://github.com/wangyu-/UDPspeeder/releases/download/20190121.0/speederv2_binaries.tar.gz
tar -xzf speederv2_binaries.tar.gz
sudo cp speederv2_amd64 /usr/bin/speederv2
sudo chmod +x /usr/bin/speederv2
```

- 下载 udp2raw

```shell script
wget https://github.com/wangyu-/udp2raw-tunnel/releases/download/20181113.0/udp2raw_binaries.tar.gz
tar -xzf udp2raw_binaries.tar.gz
sudo cp udp2raw_amd64 /usr/bin/udp2raw
sudo chmod +x /usr/bin/udp2raw
```

- 运行

```shell script
sudo cp supervisor/speederv2.conf /etc/supervisor/conf.d/speederv2.conf
sudo cp supervisor/udp2raw_speederv2.conf /etc/supervisor/conf.d/udp2raw_speederv2.conf
sudo service supervisor restart
```