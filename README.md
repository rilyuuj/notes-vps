# notes-vps
something for vps

## system setting
some configurations before services
### set locale
only for scenario `Minimal Installation`
```
$ apt-get install locales
$ dpkg-reconfigure locales    // choice 158, choice 3
$ locale
```
### VIM
only for Debian Series

change paste mode
```
$ vim /usr/share/vim/vim81/defaults.vim

find set mouse
if has(‘mouse’)
set mouse=a
endif

change a to r
if has(‘mouse’)
set mouse=r
endif
```
change default editor
```
$ select-editor   // choice /usr/bin/vim.basic
or
$ update-alternatives --config editor  //choice /usr/bin/vim.basic
```

### apt 
check installed kernel
```
$ dpkg -l | grep linux-image
ii  linux-image-5.3.0-28-generic               5.3.0-28.30~18.04.1                             amd64        Signed kernel image generic
ii  linux-image-generic-hwe-18.04              5.3.0.28.96                                     amd64        Generic Linux kernel image
```
check used kernel
```
$ uname -a
Linux bionic 5.3.0-28-generic #30~18.04.1-Ubuntu SMP Fri Jan 1 00:00:01 UTC 2000 x86_64 x86_64 x86_64 GNU/Linux
```
hold update for kernel
```
$ sudo apt-mark hold linux-image-5.3.0-28-generic linux-image-generic-hwe-18.04
$ dpkg -l | grep linux-image
hi  linux-image-5.3.0-28-generic               5.3.0-28.30~18.04.1                             amd64        Signed kernel image generic
hi  linux-image-generic-hwe-18.04              5.3.0.28.96                                     amd64        Generic Linux kernel image
$ sudo apt update
$ sudo apt upgrade
```
release hold update for kernel
```
$ sudo apt-mark unhold linux-image-5.3.0-28-generic linux-image-generic-hwe-18.04
$ sudo apt update
```
install tools
```
$ sudo apt update
$ apt install vim curl git socat telnet ufw
```

### sync time from hardware to system
```
$ sudo hwclock
$ sudo hwclock –hctosys
$ date
```

### rync
sync file from remote server
```
$ rsync -Pavz -e "ssh -p 22 -t id_rsa" root@10.10.10.10:/data ~/backup
```

### sync disk size to df
```
$ resize2fs /dev/vda1
```
### Firewall
install
```
$ sudo apt-get install ufw
```
enable
```
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)?y
Firewall is active and enabled on system startup
$ sudo ufw default deny incoming    // same as edit /etc/default/ufw  DEFAULT_INPUT_POLICY="DROP"
$ sudo ufw default allow outgoing   // same as edit /etc/default/ufw  DEFAULT_OUTPUT_POLICY="ACCEPT"
```
allow
```
$ sudo ufw allow ssh
$ sudo ufw allow 2048/tcp
$ sudo ufw app list
Available applications:
  OpenSSH
  Nginx HTTP
  Nginx HTTPS
$ ufw app info OpenSSH
Profile: OpenSSH
Title: Secure shell server, an rshd replacement
Description: OpenSSH is a free implementation of the Secure Shell protocol.

Port:
  22/tcp
$ sudo ufw allow 'Nginx HTTPS'
$ sudo ufw deny from 192.168.111.0/24 to any port 80
$ ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
2048/tcp                   ALLOW IN    Anywhere  
80                         DENY IN     192.168.111.0/24
```
del rules
```
$ sudo ufw status numbered
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 443/tcp                    ALLOW IN    Anywhere
[ 3] 2048/tcp                   ALLOW IN    Anywhere
[ 4] 80                         DENY IN     10.10.10.0/24
$ sudo ufw delete 4
Deleting:
 deny from 10.10.10.0/24 to any port 80
Proceed with operation (y|n)? y
Rule deleted
$ sudo ufw delete allow 2048
Proceed with operation (y|n)? y
Rule deleted
$ sudo ufw reload
```
disable
```
$ sudo ufw disable
```
reset
```
$ sudo ufw reset   // del all rules and disable
```
---
## install docker
please refer [docker docs](https://docs.docker.com/engine/install/) for different os
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ apt-key fingerprint 0EBFCD88 
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
$ sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"  
$ sudo apt update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

---
## install acme
get ssl crt&key by scripts
```
$ sudo apt install socat
$ sudo curl https://get.acme.sh | sh
$ export CF_Key="xxxxxxxxxxxxxxxxxxxxxx"
$ export CF_Email="your@mail.com"
```
need clean occupy port for `80/tcp` before issue
```
$ acme.sh --issue --standalone -d your.domain.com -k ec-256
Cert success.
$ acme.sh --installcert -d your.domain.com --fullchain-file /etc/ssl/caddy/caddy.crt --key-file /etc/ssl/caddy/caddy.key --ecc //copy crt&key to path
```
auto install cert to path edit via `crontab -e`

> 0 3 15 */2 * acme.sh --installcert -d your.domain.com --fullchain-file /etc/ssl/caddy/caddy.crt --key-file /etc/ssl/caddy/caddy.key --ecc && docker caddy restart

---
## install caddy
```
$ apt install curl
$ curl https://getcaddy.com | bash -s personal
$ setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy    //enable service port running <1024
$ useradd -r -d /var/www -M -s /sbin/nologin caddy    //create system account
$ mkdir /etc/ssl/caddy
$ chown -R caddy:root /etc/ssl/caddy
$ chmod 0770 /etc/ssl/caddy
$ mkdir -p /var/www/example.com
$ chown -R caddy:caddy /var/www/
$ mkdir /etc/caddy
$ chown -R root:caddy /etc/caddy
$ wget https://raw.githubusercontent.com/caddyserver/dist/master/config/Caddyfile -O /etc/caddy/Caddyfile 
$ vi /etc/caddy/Caddyfile
```
<details>
  <summary>contents of <code>Caddyfile</code>use<code>:wq</code>to save</summary>
  
  ```
  # The Caddyfile is an easy way to configure your Caddy web server.
  #
  # Unless the file starts with a global options block, the first
  # uncommented line is always the address of your site.
  #
  # To use your own domain name (with automatic HTTPS), first make
  # sure your domain's A/AAAA DNS records are properly pointed to
  # this machine's public IP, then replace the line below with your
  # domain name.
  :80 {
   root /var/www/example.com  # Set this path to your site's directory.
   gzip
   log /var/log/caddy.log
   }
  # Enable the static file server.
  # file_server
  
  # Another common task is to set up a reverse proxy:
  # reverse_proxy localhost:8080
  
  # Or serve a PHP site through php-fpm:
  # php_fastcgi localhost:9000

  # Refer to the Caddy docs for more information:
  # https://caddyserver.com/docs/caddyfile
  ```
</details>

```
$ chown caddy:caddy /etc/caddy/Caddyfile
$ chmod 444 /etc/caddy/Caddyfile		//block rewrite config
$ touch /var/log/caddy.log
$ chown caddy:caddy /var/log/caddy.log
$ wget https://github.com/caddyserver/dist/blob/master/init/caddy.service -O /etc/systemd/system/caddy.service
$ vi /etc/systemd/system/caddy.service
```

<details>
  <summary>contents of <code>caddy.service</code>use<code>:wq</code>to save</summary>
  
  ```
  [Unit]
  Description=Caddy HTTP/2 web server
  Documentation=https://caddyserver.com/docs
  After=network-online.target
  Wants=network-online.target systemd-networkd-wait-online.service
  ; Do not allow the process to be restarted in a tight loop. If the
  ; process fails to start, something critical needs to be fixed.
  StartLimitIntervalSec=14400
  StartLimitBurst=10
  [Service]
  Restart=on-abnormal
  ; User and group the process will run as.
  User=caddy
  Group=caddy
  ; Letsencrypt-issued certificates will be written to this directory.
  Environment=CADDYPATH=/etc/ssl/caddy
  ; Always set "-root" to something safe in case it gets forgotten in the Caddyfile.
  ; ExecStart=/usr/local/bin/caddy -log stdout -agree=true -conf=/etc/caddy/Caddyfile -root=/var/tmp
  ExecStart=/usr/local/bin/caddy -log stdout -log-timestamps=false -agree=true -conf=/etc/caddy/Caddyfile -root=/var/tmp
  ExecReload=/bin/kill -USR1 $MAINPID
  ; Use graceful shutdown with a reasonable timeout
  KillMode=mixed
  KillSignal=SIGQUIT
  TimeoutStopSec=5s
  ; Limit the number of file descriptors; see `man systemd.exec` for more limit settings.
  LimitNOFILE=1048576
  ; Unmodified caddy is not expected to use more than that.
  LimitNPROC=512
  ; Use private /tmp and /var/tmp, which are discarded after caddy stops.
  PrivateTmp=true  
  ; Use a minimal /dev
  PrivateDevices=true
  ; Hide /home, /root, and /run/user. Nobody will steal your SSH-keys.
  ProtectHome=true
  ; Make /usr, /boot, /etc and possibly some more folders read-only.
  ProtectSystem=full
  ; … except /etc/ssl/caddy, because we want Letsencrypt-certificates there.
  ;   This merely retains r/w access rights, it does not add any new. Must still be writable on the host!
  ReadWritePaths=/etc/ssl/caddy
  ReadWriteDirectories=/etc/ssl/caddy
  ; The following additional security directives only work with systemd v229 or later.
  ; They further retrict privileges that can be gained by caddy. Uncomment if you like.
  ; Note that you may have to add capabilities required by any plugins in use.
  ;CapabilityBoundingSet=CAP_NET_BIND_SERVICE
  ;AmbientCapabilities=CAP_NET_BIND_SERVICE
  ;NoNewPrivileges=true
  [Install]
  WantedBy=multi-user.target
  ```
</details>

if output with `listen tcp :80: bind: permission denied` uncomment follow lines in caddy.service
> CapabilityBoundingSet=CAP_NET_BIND_SERVICE<br>
> AmbientCapabilities=CAP_NET_BIND_SERVICE<br>
> NoNewPrivileges=true

```
$ systemctl daemon-reload
$ systemctl start caddy
$ systemctl status caddy
$ systemctl enable caddy
$ service caddy restart
```

---
## Kcptun
```
$ useradd -r -d /nonexistent -M -s /usr/sbin/nologin kcptun
$ curl -SLO https://github.com/xtaci/kcptun/releases/download/v20200409/kcptun-linux-amd64-20200409.tar.gz
$ tar -C /usr/local/bin/ -xf kcptun-linux-amd64-20200409.tar.gz
$ mv /usr/local/bin/{client_linux_amd64,kcptun-client}
$ mv /usr/local/bin/{server_linux_amd64,kcptun-server}
$ chmod 755 /usr/local/bin/kcptun-*
$ chown root:root /usr/local/bin/kcptun-*
$ mkdir /etc/kcptun
$ wget https://github.com/xtaci/kcptun/blob/master/examples/server.json -O /etc/kcptun/server-config.json
$ vi /etc/kcptun/server-config.json
```
<details>
  <summary>contents of <code>server-config.json</code>use<code>:wq</code>to save</summary>
  
  ```
  {
        "listen": ":2000",
        "target": "127.0.0.1:9999",
        "key": "PASSWORD",
        "crypt": "aes-128",
        "mode": "fast3",
        "mtu": 1400,
        "sndwnd": 2048,
        "rcvwnd": 2048,
        "datashard": 10,
        "parityshard": 3,
        "dscp": 46,
        "nocomp": true,
        "acknodelay": false,
        "nodelay": 1,
        "interval": 40,
        "resend": 2,
        "nc": 1,
        "sockbuf": 16777217,
        "smuxver": 1,
        "smuxbuf": 16777217,
        "streambuf": 2097152,
        "keepalive": 10,
        "pprof":false,
        "quiet":false,
        "tcp":false
  }
  ```
  </details>

```
$ wget https://raw.githubusercontent.com/xtaci/kcptun/master/examples/kcptun.service -O /etc/systemd/system/kcptun.service
$ vi /etc/systemd/system/kcptun.service
```

<details>
  <summary>contents of <code>kcptun.service</code>use<code>:wq</code>to save</summary>
  
  ```
  [Unit]
  Description=kcptun
  Wants=network.target
  After=syslog.target network-online.target
  
  [Service]
  Type=simple
  User=kcptun
  Group=kcptun
  CapabilityBoundingSet=CAP_NET_BIND_SERVICE
  AmbientCapabilities=CAP_NET_BIND_SERVICE
  NoNewPrivileges=true
  Environment=GOGC=20
  ExecStart=/usr/local/bin/kcptun-server -c /etc/kcptun/server-config.json
  Restart=on-failure
  RestartSec=10
  KillMode=process
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
  ```
  </details>
  
```
$ systemctl daemon-reload
$ touch /var/log/kcptun.log
$ chown kcptun:kcptun /var/log/kcptun.log
$ systemctl restart kcptun
$ systemctl enable kcptun
```

---
## trojan
run trojan with docker
```
$ mkdir /etc/trojan && cd /etc/trojan
$ mkdir /etc/ssl/trojan
$ vi /etc/trojan/config.json
```

<details>
  <summary>contents of <code>config.json</code>use<code>:wq</code>to save</summary>
 
  ```
	{
		"run_type": "server",
		"local_addr": "0.0.0.0",
		"local_port": 443,
		"remote_addr": "127.0.0.1",
		"remote_port": 80,
		"password": [
			"PASSWORD"
		],
		"log_level": 1,
		"ssl": {
			"cert": "/etc/ssl/trojan/trojan.crt",
			"key": "/etc/ssl/trojan/trojan.key",
			"key_password": "",
			"cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
			"cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
			"prefer_server_cipher": true,
			"alpn": [
				"http/1.1"
			],
			"reuse_session": true,
			"session_ticket": false,
			"session_timeout": 600,
			"plain_http_response": "",
			"curves": "",
			"dhparam": ""
		},
		"tcp": {
			"prefer_ipv4": false,
			"no_delay": true,
			"keep_alive": true,
			"reuse_port": false,
			"fast_open": false,
			"fast_open_qlen": 20
		},
		"mysql": {
			"enabled": false,
			"server_addr": "127.0.0.1",
			"server_port": 3306,
			"database": "trojan",
			"username": "trojan",
			"password": ""
		}
	}
  ```
</details>

```
$ docker pull trojangfw/trojan
$ docker run -d --name trojan --restart always --net host -v /etc/ssl/trojan:/etc/ssl/trojan -v /etc/trojan:/config trojangfw/trojan  // add -v the path for cert in host
```

---
## shadowsocks
run shadowsocks-libev with docker
```
$ mkdir /etc/shadowsocks-libev
$ cd /etc/shadowsocks-libev
$ vim /etc/shadowsocks-libev/config.json
```

<details>
  <summary>contents of <code>config.json</code>use<code>:wq</code>to save</summary>
	
	```
		{
		"server":"0.0.0.0",
		"server_port":1080,
		"method":"chacha20-ietf-poly1305",
		"timeout":300,
		"user":"nobody",
		"password":"PASSWORD",
		"nameserver":"8.8.8.8",
		"mode":"tcp_and_udp",
		}
	```
</details>

```
$ docker pull teddysun/shadowsocks-libev
$ docker run -d --name ss-libev --restart always --net host -v /etc/shadowsocks-libev:/etc/shadowsocks-libev teddysun/shadowsocks-libev
$ docker logs ss-libev
```

use install version please running follow line
> $ setcap 'cap_net_bind_service=+ep' /usr/bin/ss-server<br>
> $ systemctl edit shadowsocks-libev		// edit deamon servive

---
