# shadowsocks-server

## 开启BBR

https://github.com/google/bbr

```shell script
$ sudo nano /etc/sysctl.conf
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr

$ sudo sysctl -p
```

## Shadowsocks

https://github.com/shadowsocks/shadowsocks-libev

### 参数优化

https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks

- 编辑 /etc/sysctl.d/local.conf

```shell sctipt
$ sudo touch /etc/sysctl.d/local.conf
$ sudo nano /etc/sysctl.d/local.conf

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

$ sysctl --system
```

### 安装

```shell script
$ sudo apt update
$ sudo apt install shadowsocks-libev
```

### 配置

```shell script
$ sudo nano /etc/shadowsocks-libev/config.json

{
	"server":["::1", "0.0.0.0"],
	"mode":"tcp_and_udp",
	"server_port":8388,
	"local_port":1080,
	"password":"",
	"timeout":60,
	"method":"chacha20-ietf-poly1305",
	"fast_open":true
}
```

### 运行

```shell script
$ sudo systemctl enable shadowsocks-libev
$ sudo systemctl start shadowsocks-libev
$ sudo systemctl status shadowsocks-libev
$ sudo systemctl restart shadowsocks-libev
```

## Kcptun

https://github.com/xtaci/kcptun

### 优化参数

- 执行 ulimit -n 65535

```shell script
$ sudo nano ~/.bashrc

ulimit -n 65535
```

- 编辑 /etc/sysctl.conf

```shell script
$ sudo nano /etc/sysctl.conf

net.core.rmem_max=26214400
net.core.rmem_default=26214400
net.core.wmem_max=26214400
net.core.wmem_default=26214400
net.core.netdev_max_backlog=2048

$ sudo sysctl -p
```

### 安装

- 下载`kcptun`

```shell script
$ wget https://github.com/xtaci/kcptun/releases/download/v20201010/kcptun-linux-amd64-20201010.tar.gz
$ tar -xzf kcptun-linux-amd64-20201010.tar.gz
$ sudo cp server_linux_amd64 /usr/bin/kcptun
$ sudo chmod +x /usr/bin/kcptun
```

### 配置

https://github.com/xtaci/kcptun/issues/251

```shell script
$ sudo mkdir /etc/kcptun
$ sudo touch /etc/kcptun/config.json
$ sudo nano /etc/kcptun/config.json
```

```json
{
  "listen": ":4000",
  "target": "127.0.0.1:8388",
  "key": "HelloKcptun!",
  "crypt": "aes-128",
  "mode": "fast3",
  "mtu": 1400,
  "sndwnd": 5120,
  "rcvwnd": 5120,
  "datashard": 30,
  "parityshard": 15,
  "dscp": 46,
  "nocomp": true,
  "acknodelay": false,
  "nodelay": 0,
  "interval": 20,
  "resend": 2,
  "nc": 1,
  "sockbuf": 4194304,
  "keepalive": 10,
  "log": "/var/log/kcptun.log"
}
```

### 运行

```shell script
$ sudo touch /usr/lib/systemd/system/kcptun.service
$ sudo nano /usr/lib/systemd/system/kcptun.service
```

```
Description=kcptun

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Environment=GOGC=20
ExecStart=/usr/bin/kcptun -c /etc/kcptun/config.json
Restart=on-failure
RestartSec=10
KillMode=process
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```shell script
$ sudo systemctl enable kcptun
$ sudo systemctl start kcptun
$ sudo systemctl status kcptun
$ sudo systemctl restart kcptun
```
