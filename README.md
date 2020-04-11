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
[ 4] 80                         DENY IN     192.168.111.0/24
$ sudo ufw delete 4
Deleting:
 deny from 192.168.111.0/24 to any port 80
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
get ssl cer/key by acme
```
$ sudo apt install socat
$ sudo curl https://get.acme.sh | sh
$ export CF_Key="xxxxxxxxxxxxxxxxxxxxxx"
$ export CF_Email="your@mail.com"
```
need clean occupy port for tcp 80 before issue
```
$ acme.sh --issue --standalone -d your.domain.com -k ec-256
Cert success.
$ acme.sh --installcert -d your.domain.com --fullchain-file /etc/ssl/caddy/caddy.crt --key-file /etc/ssl/caddy/caddy.key --ecc //copy  crt&key to path
```
auto install cert to path edit via `crontab -e`
```
0 3 15 */2 * acme.sh --installcert -d your.domain.com --fullchain-file /etc/ssl/caddy/caddy.crt --key-file /etc/ssl/caddy/caddy.key --ecc && docker caddy restart
```

---
## install caddy
```
$ apt install curl
$ curl https://getcaddy.com | bash -s personal
$ setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy    //enable service port running >1024
$ useradd -r -d /var/www -M -s /sbin/nologin caddy    //create system account
$ mkdir /etc/ssl/caddy
$ chown -R caddy:root /etc/ssl/caddy
$ chmod 0770 /etc/ssl/caddy
$ mkdir -p /var/www/example.com
$ chown -R caddy:caddy /var/www/
$ mkdir /etc/caddy
$ chown -R root:caddy /etc/caddy
$ vi /usr/local/caddy/Caddyfile
```
<details>
  <summary>contents of <code>Caddyfile</code></summary>
  
  ```
  http://yourdomain.com {
   root /var/www/example.com
   gzip
   log /var/log/caddy.log
   }
  ```
</details>
 
 adf



