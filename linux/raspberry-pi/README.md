# Network

- Port Forwarding
- DDNS
  - Asus
  - Duckdns
- ufw
```bash
sudo apt install ufw
sudo ufw allow 80
sudo ufw allow 9122
sudo ufw enable
sudo reboot
```

# OS

### Install

```bash
download img file
insert sd card
df -h
umonut /dev/sd**
# 아래 of에 sd card의 내부 파티션을 적는게 아니라 sd card 자체의 경로를 적어준다.
dd bs=1M if=2012-07-15-wheezy-raspbian.img of=/dev/sdc
```

```bash
sudo raspi-config
expand_rootfs
change_timezone
memory_split
overclock
ssh
auto login
Generic 105-key (Intl) PC -> Other -> Korean -> Korean-Korean(101-104 key compatible) -> The default for the keyboard layout -> No compose key -> No
```

```bash

# Install Applications
sudo apt update
sudo apt -y install curl
sudo apt -y install htop
sudo apt -y install git
sudo apt -y install vim
sudo apt -y install p7zip-full
sudo apt -y install ffmpeg
sudo apt -y install mp4v2-utils
sudo apt -y install libsubtitles-perl
sudo apt -y install python-pip
sudo apt -y install openjdk-8-jdk
sudo apt -y install fonts-unfonts-core
sudo apt -y install transmission-daemon
sudo apt -y install nginx
sudo apt -y install node
sudo apt -y install mongodb
sudo pip install --upgrade yt-dlp
sudo apt -y upgrade
sudo apt -y autoremove
```

### Config
```bash
alias l='ls -h --color=auto'
alias ls='ls -h --color=auto'
alias ll='ls -hl --color=auto'
alias la='ls -hla --color=auto'
alias du='du -h'
alias df='df -h'
```

- fail2ban

```bash
crontab
*/10 * * * * dgdsingen /home/dgdsingen/bak/util/duckdns/duck.sh
00 02 * * * root /usr/bin/letsencrypt renew > /var/log/letsencrypt/renew.log
```

### File System
```bash
# Optimize fstab
sudo vi /etc/fstab
errors=remount-ro
# 위를 아래와 같이 변경
errors=remount-ro,discard,noatime

# External Drive -> 이거 대신 fstab에 auto mount 설정하자
sudo vi /boot/config.txt
max_usb_current=2
safe_mode_gpio=4

trim
```

### Auto Start

```bash
/etc/rc.local
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
printf "My IP address is %s\n" "$_IP"
fi

# 이 부분에 프로그램 등록. 물론 이 프로그램이 추가로 접근할 파일들의 권한은 실행 가능하도록 조정해 주어야 한다.
su pi -c '/etc/init.d/ssh start' &

exit 0
```

# Server

### nginx
- lets encrypt ssl
- bookmark basic authentication
- raspcontrol

```bash
/etc/init.d/nginx
# log dir이 없으면 service가 올라가지 않으므로, nginx 실행 스크립트 최상단에서 log dir 생성
if [ ! -d /var/log/nginx ]; then
    mkdir /var/log/nginx
fi
```

```conf
/etc/nginx/sites-available/default
server {
    root /var/www;
    index index.html index.htm index.php;
    server_name localhost;
 
    location / {
        try_files $uri $uri/ /index.html;
    }
 
    location /doc/ {
        alias /usr/share/doc/;
        autoindex on;
        allow 127.0.0.1;
        allow ::1;
        deny all;
    }
 
    location /wiki/ {
        autoindex on;
        allow 127.0.0.1;
        allow ::1;
    }
 
    location /wiki/data {
        deny all;
    }
 
    location /raspcontrol/ {
        autoindex on;
        try_files $uri $uri/ /index.php;
        allow 127.0.0.1;
        allow ::1;
     }
 
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }
}
```

### node.js
- bookmark

### mongodb
### sshd
### transmission
### plex?
### music?

### /usr/local/bin
- yt-dlp podcast
  - yta for audio, ytv for video
  - --get-id
  - --download-archive ytv-archive
    - youtube mJY-HhPcFhQ
  - -f 'best[height<=720]'
  - --write-sub
  - --sub-lang en
  - -o '%(title)s.%(ext)s'
  - upload to google drive via node.js

# Backup
- mongoexport/mongoimport or mongodump/mongorestore bookmark data
  - mongoexport --db mydb --collection ids --out ids.json
  - mongoexport --db mydb --collection nodes --out nodes.json
  - mongoimport --mode upsert --db mydb --collection ids --file ids.json
  - mongoimport --mode upsert --db mydb --collection nodes --file nodes.json
- config files
- upload to
  - google drive
  - dropbox
  - github
