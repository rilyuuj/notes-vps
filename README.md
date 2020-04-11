# notes-vps
something for vps

## system setting
some configurations before services
### set locale
for Minimal installation
```
apt-get install locales
dpkg-reconfigure locales    // choice 158, choice 3
locale
```
### VIM
only for Debian series

- change paste mode
```
vim /usr/share/vim/vim81/defaults.vim

find set mouse
if has(‘mouse’)
set mouse=a
endif

change a to r
if has(‘mouse’)
set mouse=r
endif
```
- change default editor
```
select-editor   // choice /usr/bin/vim.basic
or
update-alternatives --config editor  //choice /usr/bin/vim.basic
```

### apt 
- check installed kernel
```
$ dpkg -l | grep linux-image
ii  linux-image-5.3.0-28-generic               5.3.0-28.30~18.04.1                             amd64        Signed kernel image generic
ii  linux-image-generic-hwe-18.04              5.3.0.28.96                                     amd64        Generic Linux kernel image
```
- check used kernel
```
$ uname -a
Linux bionic 5.3.0-28-generic #30~18.04.1-Ubuntu SMP Fri Jan 1 00:00:01 UTC 2000 x86_64 x86_64 x86_64 GNU/Linux
```
- hold update for kernel
```
$ sudo apt-mark hold linux-image-5.3.0-28-generic linux-image-generic-hwe-18.04
$ dpkg -l | grep linux-image
hi  linux-image-5.3.0-28-generic               5.3.0-28.30~18.04.1                             amd64        Signed kernel image generic
hi  linux-image-generic-hwe-18.04              5.3.0.28.96                                     amd64        Generic Linux kernel image
$ sudo apt update
$ sudo apt upgrade
```
- release hold update for kernel
```
sudo apt-mark unhold linux-image-5.3.0-28-generic linux-image-generic-hwe-18.04
sudo apt update
```
- install tools
> sudo apt update
> apt install vim curl git socat telnet ufw

### sync time from hardware to system
> sudo hwclock
> sudo hwclock –hctosys

### rync
sync file from remote server
> rsync -Pavz -e "ssh -p 22 -t id_rsa" root@10.10.10.10:/data ~/backup

### sync disk size to df
> resize2fs /dev/vda1

### filewall
install ufw
> sudo apt-get install ufw
enable
```
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)?y
Firewall is active and enabled on system startup
> sudo ufw default deny
allow
sudo ufw allow ssh
sudo ufw status numbered
```



