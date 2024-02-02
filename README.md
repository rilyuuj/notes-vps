# notes-vps
something for vps

## system setting
some configurations before services
### set locale
only for scenario `Minimal Installation`
```
$ apt-get install locales
$ dpkg-reconfigure locales    // choice 158, choice 3 en_US.UTF-8 UTF-8
$ locale
```
### VIM
only for Debian Series

change paste mode
```
$ vim /usr/share/vim/vim$version/defaults.vim

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
install latest kernel and enable bbr
```
$ sudo nano /etc/apt/sources.list
 	//insert following line to end of file sources.list
deb http://deb.debian.org/debian buster-backports main
$ sudo apt update && sudo apt -t buster-backports install linux-image-amd64
$ sysctl net.ipv4.tcp_available_congestion_control    //check available congestion control algorithms
net.ipv4.tcp_available_congestion_control = reno cubic
$ sysctl net.ipv4.tcp_congestion_control    //check the current congestion control algorithm used in your system
net.ipv4.tcp_congestion_control = cubic
$ sudo nano /etc/sysctl.conf    //insert following line to end of file sysctl.conf
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
$ sysctl -p    //refresh your configuration
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
$ sudo reboot
$ sysctl net.ipv4.tcp_congestion_control    //Verify if BBR is enabled in your system
net.ipv4.tcp_congestion_control = bbr
$ lsmod | grep bbr 	//check bbr is open
tcp_bbr                20480  8
$ lsmod | grep fq
sch_fq                 20480  2
```

install common tools
```
$ sudo apt update
$ apt install sudo vim curl git socat telnet ufw net-tools
```

### create swap
```
$ sudo swapon --show
$ free -m
$ sudo dd if=/dev/zero of=/swapfile bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 2.32912 s, 461 MB/s
$ ls -lar /swapfile
-rw-r--r-- 1 root root 1073741824 Jan 29 10:03 /swapfile
$ sudo chmod 600 /swapfile
$ ls -lar /swapfile
-rw------- 1 root root 1073741824 Jan 29 10:03 /swapfile
$ mkswap /swapfile
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=e2654e61-b327-4e46-6642-12dae290c110
$ free -m
              total        used        free      shared  buff/cache   available
Mem:            474          59          15           7         399         395
Swap:             0           0           0
$ sudo swapon /swapfile
$ free -m
              total        used        free      shared  buff/cache   available
Mem:            474          62          10           7         401         392
Swap:          1023           0        1023
$ sudo vi /etc/fstab
/swapfile swap swap defaults 0 0
$ swapon --show
NAME      TYPE  SIZE USED PRIO
/swapfile file 1024M 780K   -2
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          474Mi        60Mi        13Mi       7.0Mi       400Mi       394Mi
Swap:         1.0Gi       0.0Ki       1.0Gi
```

### sync time from hardware to system
```
$ sudo hwclock
$ sudo hwclock –-hctosys
$ date
```

### rync
sync file from remote server
```
make sure rysnc installed on both side
$ sudo apt install rsync 		# yum install rsync (Centos7)
$
$ rsync -Pavz -e "ssh -p 22 -t id_rsa" root@10.10.10.10:/data ~/backup
$ cd ~/backup && mv * .[^.]* .. 
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
$ sudo ufw deny from 10.10.10.0/24 to any port 80
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
80                         DENY IN     10.10.10.0/24
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
$ sudo apt-key fingerprint 0EBFCD88 
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
enable auto update
```
$ acme.sh --upgrade --auto-upgrade
[Wed 30 Dec 2022 04:22:12 AM EST] Already uptodate!
[Wed 30 Dec 2022 04:22:12 AM EST] Upgrade success!
```
get ssl crt&key with api key
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
install acme & get ecc crt&key with api token
```
$ sudo curl https://get.acme.sh | sh
$ export CF_Token="xxxxxxxxxxxxxxxxxxxxxx"  // cloudflare: My Profile --> API tokens --> Create token
$ export CF_Account_ID="xxxxxxxxxxxxxxxxxxxxxx" // cloudflare: Home --> My Site --> Overview --> API
$ acme.sh --issue --test -d 二级域名.你的域名.com -w /home/vpsadmin/www/webpage --keylength ec-256
[Wed 30 Dec 2022 04:25:12 AM EST] Using ACME_DIRECTORY: https://acme-staging-v02.api.letsencrypt.org/directory
[Wed 30 Dec 2022 04:25:13 AM EST] Using CA: https://acme-staging-v02.api.letsencrypt.org/directory
[Wed 30 Dec 2022 04:25:13 AM EST] Create account key ok.
[Wed 30 Dec 2022 04:25:13 AM EST] Registering account: https://acme-staging-v02.api.letsencrypt.org/directory
[Wed 30 Dec 2022 04:25:13 AM EST] Registered
[Wed 30 Dec 2022 04:25:13 AM EST] ACCOUNT_THUMBPRINT='CU6qmPKuRqhyTAIrF4swosR375194z_1ddUlWef8xDc'
[Wed 30 Dec 2022 04:25:13 AM EST] Creating domain key
[Wed 30 Dec 2022 04:25:13 AM EST] The domain key is here: /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.key
[Wed 30 Dec 2022 04:25:13 AM EST] Single domain='二级域名.你的域名.com'
[Wed 30 Dec 2022 04:25:13 AM EST] Getting domain auth token for each domain
[Wed 30 Dec 2022 04:25:14 AM EST] Getting webroot for domain='二级域名.你的域名.com'
[Wed 30 Dec 2022 04:25:14 AM EST] Verifying: 二级域名.你的域名.com
[Wed 30 Dec 2022 04:25:23 AM EST] Pending
[Wed 30 Dec 2022 04:25:25 AM EST] Success
[Wed 30 Dec 2022 04:25:25 AM EST] Verify finished, start to sign.
[Wed 30 Dec 2022 04:25:25 AM EST] Lets finalize the order.
[Wed 30 Dec 2022 04:25:25 AM EST] Le_OrderFinalize='https://acme-staging-v02.api.letsencrypt.org/acme/finalize/490205995/7730242871'
[Wed 30 Dec 2022 04:25:25 AM EST] Downloading cert.
[Wed 30 Dec 2022 04:25:25 AM EST] Le_LinkCert='https://acme-staging-v02.api.letsencrypt.org/acme/cert/xujss5xt8i38waubafz2xujss5xt8i38waubz2'
[Wed 30 Dec 2022 15:21:52 AM EST] Cert success.
-----BEGIN CERTIFICATE-----
sxlYqPvWreKgD5b8JyOQX0Yg2MLoRUoDyqVkd31PthIiwzdckoh5eD3JU7ysYBtN
cTFK4LGOfjqi8Ks87EVJdK9IaSAu7ZC6h5to0eqpJ5PLhaM3e6yJBbHmYA8w1Smp
wAb3tdoHZ9ttUIm9CrSzvDBt6BBT6GqYdDamMyCYBLooMyDEM4CUFsOzCRrEqqvC
2mTTEmhvpojo5rhdTSJxibozyNWTGwoTj0v9pTUeQcGqLIzqi4DowjBHD5guwRid
SjAFnm6JT2xUQgWFm58A1gv1OhbH1TRPUUmtE1nFEN7YiSjI4xgxqAXT3CLD2EUb
wXlUrO6c75zSsQP4bRMzgOjJUqHtSb6IEqELzt4M7KzL5iCOruCChCo2DZxUwvVX
tOoaAyQJzCbTqE6aUqwiKi3gVyoxvDP9mI5JdRYzsDL6GVud7EHPnYeMl9ubLZAK
0vg84mbMP3f6mYM4KRa1cqiyOIcQPT4AzGFYVv4sm049bZQg7sd0Bz9CaFvE7yDA
1y17XlgCDnsjxl66bqI1vkENN9XT5xeFHONqc18b5fZEKSIvdX7iWPFWp1PyMPpG
0pMCP1EymZNFxIMJLgbWqExwLWfPc5Ib3PjBaIqhXPnw6sT2MQSxXwDupq1UJVhV
7E3hQRVlwI4CXi6WLHJMNvNRyyK87gCrLH1bKYsPeRVaz77poWBq49zwBCts6hPY
IeF4ltGXyANNIOPEi8vy138fRU4LYh81d8FjOtFfJZogMjwhfNvapqxPMsioPlmX
TnZu0n7setrVNUEfTMHWqPpDgk5MPrWLA4LapqaDfEX4pwnQJLMwMi6s94z165c0
iMRSKA1yU5zqv8aNsDfPoY4OkSPWs4MaXgRRSLBsUfZ15DwQXPk76kegHIyxWvwF
tYw9HKR5QCMK66fa0z4aJoFVFLK0IIOGEZOanRFUCnkLUDd3QZ3YU8lEcrj7Uxos
haiRNICyC6UfsCJ94a8vcNyMosPv3xBLMp19WXgiFYqEFQkntkv1FLRI35fjeJmg
0fmD9VG9bkzGPHihJgQLRlCHasGf6XrdfkSsODAyCUHUHJ0RzqF4YEZMcxDxzuQ2
YO7bFwj7S3mUdVPZ6MPasjxdyBjJgEBMch2uy4AhmudXfEBQBye8W6ZI4ztZjLVV
FmP4SIuaNUmMe20TjR8b9NVC96AhxOanWT3mRROsdokpKQGTJvl27EHH8KuAbUOc
G6KtPy4wslNZNXWcBy9n63RcWak12r7kAIFn38tZxmlw2WUKoRSMAH64GcDTjRQd
Am65hBHzvGrj93wEuVNIebvNIsJOlng3HFjpIxVqKGMCIfWIKGDE3YzK3p4LbGZ6
NZFQWYJLNVf2M9CCJfbEImPYgvctrxl39H6KVYPCw1SAdaj9NneUqmREOQkKoEB0
x6PmNirbMscHhQPSC0JQaqUgaQFgba1ALmzRYAnYhNb0twkTxWbY7DBkAarxqMIp
yiLKcBFc5H7dgJCImo7us7aJeftC44uWkPIjw9AKH=
-----END CERTIFICATE-----
[Wed 30 Dec 2022 15:21:52 AM EST] Your cert is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.cer
[Wed 30 Dec 2022 15:21:52 AM EST] Your cert key is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.key
[Wed 30 Dec 2022 15:21:52 AM EST] The intermediate CA cert is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/ca.cer
[Wed 30 Dec 2022 15:21:52 AM EST] And the full chain certs is there:  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/fullchain.cer
$ acme.sh --issue -d 二级域名.你的域名.com -w /home/vpsadmin/www/webpage --keylength ec-256 --force
[Wed 30 Dec 2022 15:22:51 AM EST] Using CA: https://acme-v02.api.letsencrypt.org/directory
[Wed 30 Dec 2022 15:22:51 AM EST] Creating domain key
[Wed 30 Dec 2022 15:22:51 AM EST] The domain key is here: /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.key
[Wed 30 Dec 2022 15:22:51 AM EST] Single domain='二级域名.你的域名.com'
[Wed 30 Dec 2022 15:22:51 AM EST] Getting domain auth token for each domain
[Wed 30 Dec 2022 15:22:51 AM EST] Getting webroot for domain='二级域名.你的域名.com'
[Wed 30 Dec 2022 15:22:51 AM EST] Verifying: 二级域名.你的域名.com
[Wed 30 Dec 2022 15:22:51 AM EST] Pending
[Wed 30 Dec 2022 15:22:51 AM EST] Success
[Wed 30 Dec 2022 15:22:51 AM EST] Verify finished, start to sign.
[Wed 30 Dec 2022 15:22:51 AM EST] Lets finalize the order.
[Wed 30 Dec 2022 15:22:51 AM EST] Le_OrderFinalize='https://acme-v02.api.letsencrypt.org/acme/finalize/490205996/7730242872'
[Wed 30 Dec 2022 15:22:51 AM EST] Downloading cert.
[Wed 30 Dec 2022 15:22:51 AM EST] Le_LinkCert='https://acme-v02.api.letsencrypt.org/acme/cert/vsxvk0oldnuobe51ayxz4dms62sk2dwmw9zhuw'
[Wed 30 Dec 2022 15:22:51 AM EST] Cert success.
-----BEGIN CERTIFICATE-----
sxRYqPvWreKgD5b8JyOQX0Yg2MLoRUoDyqVkd31PthIiwzdckoh5eD3JU7ysYBtN
cTFK4LGOfjqi8Ks87EVJdK9IaSAu7ZC6h5to0eqpJ5PLhaM3e6yJBbHmYA8w1Smp
wAb3tdoHZ9ttUIm9CrSzvDBt6BBT6GqYdDamMyCYBLooMyDEM4CUFsOzCRrEqqvC
2mTTEmhvpojo5rhdTSJxibozyNWTGwoTj0v9pTUeQcGqLIzqi4DowjBHD5guwRid
SjAFnm6JT2xUQgWFm58A1gv1OhbH1TRPUUmtE1nFEN7YiSjI4xgxqAXT3CLD2EUb
wXlUrO6c75zSsQP4bRMzgOjJUqHtSb6IEqELzt4M7KzL5iCOruCChCo2DZxUwvVX
tOoaAyQJzCbTqE6aUqwiKi3gVyoxvDP9mI5JdRYzsDL6GVud7EHPnYeMl9ubLZAK
0vg84mbMP3f6mYM4KRa1cqiyOIcQPT4AzGFYVv4sm049bZQg7sd0Bz9CaFvE7yDA
1y17XlgCDnsjxl66bqI1vkENN9XT5xeFHONqc18b5fZEKSIvdX7iWPFWp1PyMPpG
0pMCP1EymZNFxIMJLgbWqExwLWfPc5Ib3PjBaIqhXPnw6sT2MQSxXwDupq1UJVhV
7E3hQRVlwI4CXi6WLHJMNvNRyyK87gCrLH1bKYsPeRVaz77poWBq49zwBCts6hPY
IeF4ltGXyANNIOPEi8vy138fRU4LYh81d8FjOtFfJZogMjwhfNvapqxPMsioPlmX
TnZu0n7setrVNUEfTMHWqPpDgk5MPrWLA4LapqaDfEX4pwnQJLMwMi6s94z165c0
iMRSKA1yU5zqv8aNsDfPoY4OkSPWs4MaXgRRSLBsUfZ15DwQXPk76kegHIyxWvwF
tYw9HKR5QCMK66fa0z4aJoFVFLK0IIOGEZOanRFUCnkLUDd3QZ3YU8lEcrj7Uxos
haiRNICyC6UfsCJ94a8vcNyMosPv3xBLMp19WXgiFYqEFQkntkv1FLRI35fjeJmg
0fmD9VG9bkzGPHihJgQLRlCHasGf6XrdfkSsODAyCUHUHJ0RzqF4YEZMcxDxzuQ2
YO7bFwj7S3mUdVPZ6MPasjxdyBjJgEBMch2uy4AhmudXfEBQBye8W6ZI4ztZjLVV
FmP4SIuaNUmMe20TjR8b9NVC96AhxOanWT3mRROsdokpKQGTJvl27EHH8KuAbUOc
G6KtPy4wslNZNXWcBy9n63RcWak12r7kAIFn38tZxmlw2WUKoRSMAH64GcDTjRQd
Am65hBHzvGrj93wEuVNIebvNIsJOlng3HFjpIxVqKGMCIfWIKGDE3YzK3p4LbGZ6
NZFQWYJLNVf2M9CCJfbEImPYgvctrxl39H6KVYPCw1SAdaj9NneUqmREOQkKoEB0
x6PmNirbMscHhQPSC0JQaqUgaQFgba1ALmzRYAnYhNb0twkTxWbY7DBkAarxqMIp
yiLKcBFc5H7dgJCImo7us7aJeftC44uWkPM=
-----END CERTIFICATE-----
[Wed 30 Dec 2022 15:22:52 AM EST] Your cert is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.cer
[Wed 30 Dec 2022 15:22:52 AM EST] Your cert key is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.key
[Wed 30 Dec 2022 15:22:52 AM EST] The intermediate CA cert is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/ca.cer
[Wed 30 Dec 2022 15:22:52 AM EST] And the full chain certs is there:  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/fullchain.cer
$ acme.sh --install-cert -d 二级域名.你的域名.com --ecc \
            --fullchain-file ~/xray_cert/xray.crt \
            --key-file ~/xray_cert/xray.key
$ chmod +r ~/xray_cert/xray.key
```

force renew cert for your domain
```
$ acme.sh --renew -d 二级域名.你的域名.com --force
[Wed 30 Dec 2022 15:23:29 AM EST] The domain '二级域名.你的域名.com' seems to have a ECC cert already, please add '--ecc' parameter if you want to use that cert.
[Wed 30 Dec 2022 15:23:29 AM EST] Renew: '二级域名.你的域名.com'
[Wed 30 Dec 2022 15:23:29 AM EST] '二级域名.你的域名.com' is not an issued domain, skip.
$ acme.sh --renew -d 二级域名.你的域名.com --ecc --force
[Wed 30 Dec 2022 15:23:46 AM EST] Renew: '二级域名.你的域名.com'
[Wed 30 Dec 2022 15:23:52 AM EST] Using CA: https://acme-v02.api.letsencrypt.org/directory
[Wed 30 Dec 2022 15:23:52 AM EST] Single domain='二级域名.你的域名.com'
[Wed 30 Dec 2022 15:23:52 AM EST] Getting domain auth token for each domain
[Wed 30 Dec 2022 15:23:53 AM EST] Getting webroot for domain='二级域名.你的域名.com'
[Wed 30 Dec 2022 15:23:53 AM EST] Verifying: 二级域名.你的域名.com
[Wed 30 Dec 2022 15:23:56 AM EST] Pending
[Wed 30 Dec 2022 15:23:58 AM EST] Success
[Wed 30 Dec 2022 15:23:58 AM EST] Verify finished, start to sign.
[Wed 30 Dec 2022 15:23:58 AM EST] Lets finalize the order.
[Wed 30 Dec 2022 15:23:58 AM EST] Le_OrderFinalize='https://acme-v02.api.letsencrypt.org/acme/finalize/490205997/7730242873'
[Wed 30 Dec 2022 15:23:59 AM EST] Downloading cert.
[Wed 30 Dec 2022 15:23:59 AM EST] Le_LinkCert='https://acme-v02.api.letsencrypt.org/acme/cert/vsxvk0oldnuobe51ayxz4dms62sk2dwmw9zhuw'
[Wed 30 Dec 2022 15:23:59 AM EST] Cert success.
----BEGIN CERTIFICATE-----
aslYqPvWreKgD5b8JyOQX0Yg2MLoRUoDyqVkd31PthIiwzdckoh5eD3JU7ysYBtN
cTFK4LGOfjqi8Ks87EVJdK9IaSAu7ZC6h5to0eqpJ5PLhaM3e6yJBbHmYA8w1Smp
wAb3tdoHZ9ttUIm9CrSzvDBt6BBT6GqYdDamMyCYBLooMyDEM4CUFsOzCRrEqqvC
2mTTEmhvpojo5rhdTSJxibozyNWTGwoTj0v9pTUeQcGqLIzqi4DowjBHD5guwRid
SjAFnm6JT2xUQgWFm58A1gv1OhbH1TRPUUmtE1nFEN7YiSjI4xgxqAXT3CLD2EUb
wXlUrO6c75zSsQP4bRMzgOjJUqHtSb6IEqELzt4M7KzL5iCOruCChCo2DZxUwvVX
tOoaAyQJzCbTqE6aUqwiKi3gVyoxvDP9mI5JdRYzsDL6GVud7EHPnYeMl9ubLZAK
0vg84mbMP3f6mYM4KRa1cqiyOIcQPT4AzGFYVv4sm049bZQg7sd0Bz9CaFvE7yDA
1y17XlgCDnsjxl66bqI1vkENN9XT5xeFHONqc18b5fZEKSIvdX7iWPFWp1PyMPpG
0pMCP1EymZNFxIMJLgbWqExwLWfPc5Ib3PjBaIqhXPnw6sT2MQSxXwDupq1UJVhV
7E3hQRVlwI4CXi6WLHJMNvNRyyK87gCrLH1bKYsPeRVaz77poWBq49zwBCts6hPY
IeF4ltGXyANNIOPEi8vy138fRU4LYh81d8FjOtFfJZogMjwhfNvapqxPMsioPlmX
TnZu0n7setrVNUEfTMHWqPpDgk5MPrWLA4LapqaDfEX4pwnQJLMwMi6s94z165c0
iMRSKA1yU5zqv8aNsDfPoY4OkSPWs4MaXgRRSLBsUfZ15DwQXPk76kegHIyxWvwF
tYw9HKR5QCMK66fa0z4aJoFVFLK0IIOGEZOanRFUCnkLUDd3QZ3YU8lEcrj7Uxos
haiRNICyC6UfsCJ94a8vcNyMosPv3xBLMp19WXgiFYqEFQkntkv1FLRI35fjeJmg
0fmD9VG9bkzGPHihJgQLRlCHasGf6XrdfkSsODAyCUHUHJ0RzqF4YEZMcxDxzuQ2
YO7bFwj7S3mUdVPZ6MPasjxdyBjJgEBMch2uy4AhmudXfEBQBye8W6ZI4ztZjLVV
FmP4SIuaNUmMe20TjR8b9NVC96AhxOanWT3mRROsdokpKQGTJvl27EHH8KuAbUOc
G6KtPy4wslNZNXWcBy9n63RcWak12r7kAIFn38tZxmlw2WUKoRSMAH64GcDTjRQd
Am65hBHzvGrj93wEuVNIebvNIsJOlng3HFjpIxVqKGMCIfWIKGDE3YzK3p4LbGZ6
NZFQWYJLNVf2M9CCJfbEImPYgvctrxl39H6KVYPCw1SAdaj9NneUqmREOQkKoEB0
x6PmNirbMscHhQPSC0JQaqUgaQFgba1ALmzRYAnYhNb0twkTxWbY7DBkAarxqMIp
yiLKcBFc5H7dgJCImo7us7aJeftC44uWkWE=
-----END CERTIFICATE-----
[Wed 30 Dec 2022 15:24:00 AM EST] Your cert is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.cer
[Wed 30 Dec 2022 15:24:00 AM EST] Your cert key is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/二级域名.你的域名.com.key
[Wed 30 Dec 2022 15:24:00 AM EST] The intermediate CA cert is in  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/ca.cer
[Wed 30 Dec 2022 15:24:00 AM EST] And the full chain certs is there:  /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc/fullchain.cer
[Wed 30 Dec 2022 15:24:01 AM EST] Installing key to:~/xray_cert/xray.key
[Wed 30 Dec 2022 15:24:01 AM EST] Installing full chain to:~/xray_cert/xray.crt
$ acme.sh --list
Main_Domain            KeyLength   SAN_Domains  CA                Created                           Renew
二级域名.你的域名.com    "ec-256"   no            LetsEncrypt.org   Wed 30 Dec 2022 15:22:51 AM EST   Wed 30 Dec 2022 15:24:00 AM EST
$ service caddy restart
```

Revoke and remove certificates
```
$ acme.sh --revoke -d 二级域名.你的域名.com --ecc
[Sat 30 Jan 2023 10:21:44 AM EST] Try domain key first.
[Sat 30 Jan 2023 15:21:44 AM EST] Revoke success.
$ acme.sh --remove -d 二级域名.你的域名.com --ecc
[Sat 30 Jan 2023 10:22:01 AM EST] 二级域名.你的域名.com is removed, the key and cert files are in /home/vpsadmin/.acme.sh/二级域名.你的域名.com_ecc
[Sat 30 Jan 2023 10:22:01 AM EST] You can remove them by yourself.
$ acme.sh --list
Main_Domain     KeyLength       SAN_Domains     CA      Created Renew

```

auto install cert to path edit via `crontab -e`

> 0 3 15 */2 * acme.sh --installcert -d your.domain.com --fullchain-file /etc/ssl/caddy/caddy.crt --key-file /etc/ssl/caddy/caddy.key --ecc && service caddy restart


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
        "target": "127.0.0.1:1080",
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
## v2ray
run v2ray with docker
```
$ mkdir -p /etc/v2ray
$ mkdir -p /var/log/v2ray
$ vi /etc/v2ray/config.json
```

<details>
  <summary>paste content to<code>config.json</code></summary>
	
	{
	"inbounds":[
		{
		"port":10000,
		"listen":"127.0.0.1", //此处记得写127.0.0.1，只监听本地
		"protocol":"vless",
		"settings":{
			"decryption": "none",
			"clients":[
			  {
				"id":"888d163a-80d7-4495-b3d1-fcf61fc6b6ce", //此处的uuid建议自己到uuid generator网站在线生成
				"level":0
			  }
			]
		},
		"streamSettings":{
			"network":"ws",
			"wsSettings":{
			"path":"/ray"  //说明：此处请替换你想写的path分流路径
			}
		}
		}
	],
	"outbounds":[
		{
		"protocol":"freedom",
		"settings":{}
		}
	]
	}
</details>

check the config format is correct
```
$ /usr/bin/v2ray/v2ray -test -config /etc/v2ray/config.json
V2Ray v3.15 (die Commanderin) 20180329
An unified platform for anti-censorship.
Configuration OK.
```

```
$ docker pull v2fly/v2fly-core
$ docker run -d --name v2ray --restart always -v /etc/v2ray:/etc/v2ray -v /var/log/v2ray:/var/log/v2ray --net host v2fly/v2fly-core v2ray -config=/etc/v2ray/config.json
$ docker logs v2ray
```

modify the caddy config
<details>
  <summary>content of<code>Caddyfile</code></summary>
	
	mydomain.me:80 {
  	  redir https://mydomain.me{url}
	}
	
	mydomain.me:443 {
  	  gzip
  	  tls /etc/ssl/caddy/caddy.crt /etc/ssl/caddy/caddy.key
  	  log /var/log/caddy.log
  	  proxy / https://baidu.com
  	  proxy /ray 127.0.0.1:10000 {
    	  websocket
    	  header_upstream -Origin
  	  }
	}	
</details>
restart the caddy service

```
$ systemctl restart caddy
```

---
