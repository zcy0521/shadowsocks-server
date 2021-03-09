# Shadowsocks Server

## Shadowsocks

### 安装[Shadowsocks](https://github.com/shadowsocks/shadowsocks-libev)

```shell
sudo apt update
sudo apt install shadowsocks-libev
```

### 配置Shadowsocks服务

- 编辑 /etc/shadowsocks-libev/config.json

```json
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

- 运行Shadowsocks

```shell
sudo systemctl enable shadowsocks-libev
sudo systemctl start shadowsocks-libev
sudo systemctl status shadowsocks-libev

sudo systemctl restart shadowsocks-libev
```

### 安装[Kcptun](https://github.com/xtaci/kcptun)

```shell
wget https://github.com/xtaci/kcptun/releases/download/v20201010/kcptun-linux-amd64-20201010.tar.gz
tar -xzf kcptun-linux-amd64-20201010.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 编辑`~/.bashrc`并执行`ulimit -n 65535`

```
ulimit -n 65535
```

- 编辑`/etc/sysctl.conf`并执行`sudo sysctl -p`

```
net.core.rmem_max=26214400
net.core.rmem_default=26214400
net.core.wmem_max=26214400
net.core.wmem_default=26214400
net.core.netdev_max_backlog=2048
```

### 配置Kcptun服务

- 编辑[`/etc/kcptun/config.json`](https://github.com/xtaci/kcptun/issues/251)

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

- 编辑`/usr/lib/systemd/system/kcptun.service`

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

- 运行kcptun

```shell
sudo systemctl enable kcptun
sudo systemctl start kcptun
sudo systemctl status kcptun

sudo systemctl restart kcptun
```

## 可选配置

### 修改SSH端口

- 编辑`/etc/ssh/sshd_config`并执行`sudo systemctl restart ssh`

```shell
Port = 8022
```

### 配置[BBR](https://github.com/google/bbr)

- 编辑`/etc/sysctl.conf`并执行`sudo sysctl -p`

```shell
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

### 优化Shadowsocks参数

- 编辑[`/etc/sysctl.d/local.conf`](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks) 并执行`sysctl --system`

```
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
```
