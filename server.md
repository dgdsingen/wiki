# SSH

![img](images/server/image1-1581421931057.png)

## SSH1

- authorized_keys : pub file contents
- known_hosts : host registration

즉 배포 과정을 정리하자면... 우선 클라이언트에서 ssh-keygen으로 pub/prv key set을 만든다. 그리고 pub key를 server로 전송해 authorized_keys에 내용을 추가한다. 끝. 물론 클라이언트에서 호스트로 처음 접근할 경우 known_hosts에 등록하겠냐고 물어보는데 이때 yes를 선택한 적이 있다면 known_hosts는 작업할 필요가 없다.

## SSH2

- authorization : public filenames (Key ~.pub)
- identification : private filenames (IdKey privateFileName)

## Config

### 권한 설정

```sh
chmod 755 .ssh
chmod 600 -R id_rsa id_rsa.pub authorized_keys
```

### /etc/ssh/ssh_config (client)

### /etc/ssh/sshd_config (server)

> 참조: [[리눅스] sshd_config 환경설정 파일 설명](https://jejuchild.tistory.com/203) 

```sh
# What ports, IPs and protocols we listen for
Port 9022
# Use these options to restrict which interfaces/protocols sshd will bind to
#ListenAddress ::
#ListenAddress 0.0.0.0
Protocol 2
# HostKeys for protocol version 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
#Privilege Separation is turned on for security
UsePrivilegeSeparation yes

# Lifetime and size of ephemeral version 1 server key
KeyRegenerationInterval 3600
ServerKeyBits 768

# Logging
SyslogFacility AUTH
LogLevel INFO

# Authentication:
LoginGraceTime 120
PermitRootLogin yes
StrictModes yes

RSAAuthentication no
PubkeyAuthentication yes
#AuthorizedKeysFile        %h/.ssh/authorized_keys

# Don't read the user's ~/.rhosts and ~/.shosts files
IgnoreRhosts yes
# For this to work you will also need host keys in /etc/ssh_known_hosts
RhostsRSAAuthentication no
# similar for protocol version 2
HostbasedAuthentication no
# Uncomment if you don't trust ~/.ssh/known_hosts for RhostsRSAAuthentication
#IgnoreUserKnownHosts yes

# To enable empty passwords, change to yes (NOT RECOMMENDED)
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Change to no to disable tunnelled clear text passwords
#PasswordAuthentication yes

# Kerberos options
#KerberosAuthentication no
#KerberosGetAFSToken no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes

# for performance UseDNS & X11Forwarding no
UseDNS no
X11Forwarding no
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
#UseLogin no

#MaxStartups 10:30:60
#Banner /etc/issue.net

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

Subsystem sftp /usr/lib/openssh/sftp-server

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes
```

- Protocol 2 : openssh는 프로토콜 버전을 원하는 대로 선택할 수 있습니다. protocol 2로 설정에는 서버는 버전 2로만 작동하기 때문에 ssh1을 사용해 접속을 요청하는 클라이언트를 받아 들일수 없다. protocol 1로 설정해서 가동시킬 경우에는 버전 2를 사용하는 ssh2 사용자의 요청을 받아 들일 수 없다. 보안상 protocol 1 은 사용하지 않습니다.
- KeyRegenerationInterval 3600 : 서버의 키는 한번 접속이 이루어진 뒤에 자동적으로 다시 만들어진다. 다시 만드는 목적은 나중에 호스트의 세션에 있는 키를 캡처해서 암호를 해독하거나 훔친 키를 사용하지 못하도록 하기 위함 위함입니다. 값이 0이면 키는 다시 만들어지지 않습니다. 기본값은 3600초 입니다. 이 값은 자동으로 키를 재생성하기 전까지 서버가 대기할 시간을 초단위로 정의합니다.
- ServerKeyBits 1024 : 서버 키에서 어느 정도의 비트 수를 사용할지 정의합니다. 최소값은 512이고 디폴트 값은 768입니다.
- SyslogFacility AUTH : /etc/syslog.conf에서 정의한 로그 facility 코드입니다. 가능한 값은 DAEMON, USER, AUTH, LOCAL0, LOCAL1, LOCAL2, LOCAL3, LOCAL4, LOCAL5, LOCAL6, LOCAL7입니다. 기본값은 AUTH입니다. Facilith란 메시지를 생성하는 하위 시스템을 말합니다.
- LogLevel INFO : 로그 레벨을 지정하는 것입니다. 가능한 값은 QUIET, FATAL, ERROR, INFO, VERBOSE 그리고 DEBUGS입니다.
- LoginGraceTime 600 : 유저의 로그인이 성공적으로 이루어지지 않았을 때 이 시간 후에 서버가 연결을 끊는 시간입니다. 값이 0 이면 제한 시간이 없으며 기본값은 600초입니다.
- strictModes yes : 사용자의 홈 디렉토리인 /home/username의 권한 값 등을 체크하도록 설정되어 있는 지시자입니다.
- RSAAuthentication yes : RSA 인증의 시도여부를 정의합니다. ssh1 프로토콜에만 사용하기 위해 예약된 것으로, ssh1을 사용하고 운영상 보다 안전하게 운영하려면 이 옵션을 yes로 설정해야 합니다. RSA는 인증을 하기 위해 ssh-keygen 유틸리티에 의해 생성된 공개키 와 비밀키 쌍을 사용합니다. 현재 문서에서는 보안상 ssh1 프로토콜을 사용하지 않으므로 주석 처리합니다.
- PubkeyAuthentication yes : AuthorizedKeysFile .ssh/authorized_keys ssh에서 제공하는 인증에는 공개키 인증과 암호 인증법 이렇게 두가지가 있는데, 공개키 인증을 사용할 것인지에 대해 설정하는 것입니다. 공개키 인증 사용시 공개키가 있어야 하므로 더욱 안전합니다.
- RhostsAuthentication no : sshd가 rhosts 기반의 인증을 사용할 것인지 여부를 정의합니다. rhosts 인증은 안전하지 못하므로 ‘no’로 합니다.
- IgnoreRhosts yes : IgnoreRhosts’ 명령은 인증시 rhosts와 shosts 파일의 사용여부를 정의합니다. 보안상의 이유로 인증할 때 rhosts와 shosts 파일을 사용하지 않도록 합니다.
- RhostsRSAAuthentication no : rhost나 /etc/hosts.equiv파일이 있으면 이것을 사용해 인증합니다. 이것은 보안상 별로 안 좋은 방법이기 때문에 허용하지 않습니다. RSA 호스트 인증과 맞추어 rhosts 인증의 사용여부를 정의합니다.
- HostbasedAuthentication no : 호스트 기반의 인증 허용 여부를 결정합니다.
- IgnoreUserKnownHosts yes : ssh 데몬이 RhostsRSAAuthentication 과정에서 각 사용자의 $HOME/.ssh/known_hosts를 무시할 것인지 여부를 정의합니다. rhosts 파일을 허용하지 않았으므로 yes로 설정하는 것이 안전합니다.
- PasswordAuthentication yes : 패스워드 인증을 허용합니다. 이 옵션은 프로토콜 버전 1과 2 모두 적용됩니다. 인증할 때 암호기반 인증방법의 사용 여부를 결정합니다. 강력한 보안을 위해 이옵션은 항상 ‘ no‘로 설정해야합니다.
- PermitEmptyPasswords no : 패스워드 인증을 할 때 서버가 비어있는 패스워드를 인정하는 것입니다. 기본 값은 no입니다.
- X11Forwarding no : 원격에서 X11 포워딩을 허용하는 것입니다. 이 옵션을 yes로 설정하면 xhost보다 안전한 방법으로 원격에 있는 X프로그램을 사용할 수 있습니다.
- PrintMotd yes : 사용자가 로그인 하는 경우 /etc/motd (the message of the day) 파일의 내용을 보여줄 것인지 여부결정. ssh 로그인을 환영하는 메시지나 혹은 공지사항 같은 것을 적어 놓으면 됩니다.
- Subsystem sftp /usr/libexec/openssh/sftp-server : sftp는 프로토콜 버전 2에서 사용되는 것으로서 ssh와 같이 ftp의 보안을 강화하기 위해 사용되는 보안 ftp프로그램입니다. openssh를 설치하면 /usr/local/ssh/libexec/sftp-server파일이 설치됩니다. 이것은 sftp서버용 프로그램입니다. 클라이언트 sftp프로그램은 설치되지 않습니다. 따라서 서버로 일단 가동시키고 원도용 ssh클라이언트 프로그램이나 SSH2를 설치하면 sftp를 사용할 수 있습니다.
- CheckMail (yes/no) : 사용자가 로그인할 때 새메일이 도착했음을 알리도록 하는 기능을 설정하며 기본값은 yes로 되어있습니다.
- Cipher (ciher) : 세션을 암호화 할 때 사용할 방법을 명시해 줍니다 (idea,des,3des,blowfish,arcfour 또는 없음)
- ForwardAgent : 인증 대리인이 포워드 되어야 하는지를 명시 합니다.
- KeepAlive yes
- RequireReverseMapping no : 클라이언트에게 aliive메시지를 보낼 것인지 명시하는데 접속하는 곳의 도메인이 reversMapping 이 되는지를 활인하여 접속을 허가할지 안 할지를 지정합니다. 실제로 internet상에 호스트들 중 revers mapping 이 안 되는 호스트가 상 당수이므로 되도록 no로 설정할 것을 권장합니다. 만약 여러분이 사용하시는 host가 revers mapping 이 확실히 되면 보안상 yes로 하는 것 이 좋겠지만 revers mapping이 되지 않으면 접속이 불가능 하므로 조심하십시오.
- PasswordAuthentication (yes/no) : 패스워드 기반의 인증방법을 사용할 것인지를 명시 합니다.
- PubkeyAuthentication (yes/no) : 인증 순서를 지정합니다.
- Port 22 : ssh가 사용할 기본 포트를 지정합니다. 포트 변경 시 /etc/services 파일에서 sshd 관련 포트 역시 변경할 포트로 변경해 주어야 합니다.
- AllowUsers root nextline : 로그인 허락할 계정 nextline와 root 두 계정에게만 로그인 허용 합니다.
- PermitRootLogin no : root 로그인 허용여부를 결정하는 것입니다. yes, no, without-password를 사용할 수 있습니다. 현재 no로 되어 있기 때문에 직접 root로 접속이 불가능합니다. 이옵션을 yes 로하기보다는 일반계정으로 로그인후 su 명령으로 root로 전환하는 것이 보안상 안전합니다.]
- ListenAddress 0.0.0.0 : sshd가 귀를 기울일 주소를 정해줍니다. 0.0.0.0은 모든 곳으로 부터 접속 을 받아들이겠다는 의미입니다. 하지만 패키징을 할때 어떻게 한것인지는 모르겠지만 tcp-wrapper의 영향을 받아서 hosts.deny에서 막혀 있으면 접속이 안되니 hosts.allow와 hosts.deny에서 sshd2 항목으로 제어를 할 수가 있습니다.
- AllowedAuthentications publickey,password : Sshd2가 제공하는 인증은 password와 publickey 그리고 hostbased 방식이 있는데, 기본적으로 public,password가 사용됩니다. 이는 순서대로 인증하는 방법을 보여주는데, 먼저 publickey로 인증하고, 두 번째로 password로 인증한다는 의미입니다.
- DenyUsers nextline, 3737 : 접근을 거부할 로컬의 유저를 지정합니다. 위 설정은 nextline 및 uid가 3737인 계정으로 ssh 접속 시도할 경우 접근이 거부됩니다.
- DenyGroups : 명시된 그룹은 ssh서비스에 접근할 수 없도록 하는 기능입니다. (DenyGroups sysadmin accounting) 와일드카드가 지원되며 공백 문자로 그룹을 구분합니다.
- DenyHosts : 명시된 호스트는 ssh서비스에 접근할 수 없도록 하는 기능을 합니다. (Deny Hosts shell.ourcompany.net).호스트 IP를 쓰거나 호스트 명을 쓸 수 있으며 와 일드 카드가 지원되고 공백 문자로 호스트를 구분합니다.
- AllowHosts 1.2.3.0/24 192.168.1.3 : 로그인을 허가할 IP 또는 IP 대역을 지정합니다. 여러 개일 경우에는 공란이나 "," 로 구분하여 나열하면 되고 도메인 이름일 경우에는 reverse mapping이 제공되어야 합니다.
- AllowGroups : ssh서비스에 접근 가능한 그룹을 명시합니다. (예 : AllowGroups sysadmin accounting) 와일드카드가 지원되며 공백문자로 그룹을 구분합니다.
- MaxConnections 0 : 최대 몇개의 접속을 허락할지를 지정합니다. 0은 제한을 하지 않습니다.
- PasswordGuesses 3 : 암호인증 방식으로 인증할 때 최대 몇 차례 시도를 허용할 것인지 지정합니다.
- ssh1Compatibility no : 클라이언트가 ssh1만 지원할 경우 ssh1 데몬을 실행할 것인지 여부를 지정합니다. ssh1은 보안상 취약하므로 no로 하는 것이 좋습니다.

### Tunneling

Client에서 Server로 SSH 접속하면 Tunneling으로 연결된다. 해당 Tunnel을 통과하는 패킷은 모두 SSL로 Encrypt되어 전송된다. (각 Edge에서 Decrypt 된다)

해당 SSH Tunnel을 통해 Port Forwarding이 가능하며, 크게 Local, Remote 2가지 방식이 있다.

#### Local Port Forwarding

Client에서 Server 네트워크에 접속하기 위해 사용할 수 있다.

```sh
ssh -L 8000:localhost:80 remotehost
```

Client의 localhost:8000 접속시 Server와의 SSH Tunnel을 통해 remotehost:80으로 Forwarding 된다. 즉 localhost:8000 => remotehost:22 => remotehost:80 으로 포워딩된다.

```sh
ssh -L 8000:192.168.1.1:80 remotehost
```

localhost:8000 => remotehost:22 => 192.168.1.1:80 으로 포워딩된다.

#### Remote Port Forwarding

Server에서 Client 네트워크에 접속하기 위해 사용할 수 있다.

```sh
ssh -R 8000:localhost:80 remotehost
```

remotehost:8000 => localhost:22 => localhost:80 으로 포워딩된다.

```sh
ssh -R 8000:192.168.1.1:80 remotehost
```

remotehost:8000 => localhost:22 => 192.168.1.1:80 으로 포워딩된다.

### 성능 최적화

/etc/ssh/sshd_config

```sh
# DNS 체크 생략
UseDNS no

# X11 포워딩 생략
X11Forwarding no
```



### refused connect

서버쪽에서 분명히 sshd가 잘 켜져 있고

```sh
sudo systemctl status sshd
```

sshd 포트도 잘 열려 있으며

```sh
netstat -anl --tcp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
```

클라이언트 측에서 서버 포트 체크시에도 문제가 없어 보이는데

```sh
nc -vz 172.31.1.1 22
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to 172.31.1.1:22.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
```

정작 ssh 접속은 안되는 경우가 있다.

```sh
# 클라이언트에서 서버로 접속 시도
ssh -v 172.31.1.1
...
kex_exchange_identification: read: Connection reset by peer

# 서버측 /var/log/auth.log
Jan 20 06:19:51 vm-test sshd[1234567]: refused connect from 172.31.1.2 (172.31.1.2)
```

그럴 때는 sshd 서버를 `-d` 옵션 추가해서 debug 모드로 켠 뒤 접속을 해보면 로그를 확인해볼 수 있다.



접속 실패 로그로 `no matching host key type found. their offer: ssh-rsa` 가 뜬다면 `HostKeyAlgorithms +ssh-rsa` 옵션을 추가한다.

접속 실패 로그로 `signature algorithm ssh-rsa not in pubkey accepted algorithms` 가 뜬다면 `PubkeyAcceptedKeyTypes +ssh-rsa` 옵션을 추가한다.

ssh-rsa 옵션은 보안 이슈로 default 미사용 처리된 알고리즘이라고 한다. 그러나 Android FX 앱에서는 해당 옵션으로만 sshd 접속이 가능하여 Termux sshd 설정을 아래와 같이 수정해주니 잘 됨.

```sh
PrintMotd yes
#ChallengeResponseAuthentication no
PubkeyAuthentication yes
PasswordAuthentication no
Subsystem sftp /data/data/com.termux/files/usr/libexec/sftp-server
#Ciphers aes128-cbc,aes192-cbc,aes256-cbc
KexAlgorithms +diffie-hellman-group1-sha1
HostKeyAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes +ssh-rsa
```



방화벽을 열었는데도 접속이 아예 차단된다면 hosts.allow, hosts.deny 설정도 살펴보자.

```sh
# /etc/hosts.allow
sshd: 172.31.1.2
```

위와 같이 서버에 클라이언트 IP를 등록하니 ssh-keyscan 결과도 잘 나온다.

```sh
ssh-keyscan 172.31.1.1
```



# FTP

## vsftpd

```sh
sudo apt-get install vsftpd
```

```sh
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
sudo vi /etc/vsftpd.conf

listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
pasv_enable=Yes
pasv_min_port=10000
pasv_max_port=10100
allow_writeable_chroot=YES
```

```sh
sudo ufw allow from any to any port 20,21,10000:10100 proto tcp
sudo ufw restart

sudo service vsftpd restart
```

## proftpd

vsftpd 설정이 귀찮으면 그냥 proftpd 깐다.

```sh
sudo service ufw restart
sudo service vsftpd restart
```

## Passive Mode로 접속시 Passive Port 범위 접속이 안될때

다음과 같이 SSH Tunneling으로 ftp 관련 모든 port를 1:1로 매핑하면 정상적으로 접속된다. SSH Tunneling은 ssh 부분을 참고. wget으로 접속시 Passive Port가 랜덤으로 선택되는데 이때 발생하는 케이스가 발견되었다.

```sh
ssh -L9000:localhost:9000 -L9001:localhost:9001 -L9002:localhost:9002 -L9003:localhost:9003 -L9004:localhost:9004 -L9005:localhost:9005 -L9006:localhost:9006 -L9007:localhost:9007 -L9008:localhost:9008 -L9009:localhost:9009 -L9010:localhost:9010 dgdsingen@test.com -p 9122

wget ftp://$ID:$PW@localhost:9000/test/test.txt
```

# Samba

## Install

```sh
sudo apt-get install samba
```

## Config

```sh
# ID 추가 (리눅스 계정 그대로 사용하되 삼바용 패스워드만 새로 설정)
sudo smbpasswd -a $YOUR_LINUX_ID

# ID 삭제
sudo smbpasswd -x $YOUR_LINUX_ID

# Config file 수정
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo vim /etc/samba/smb.conf

[global]
unix charset = UTF8
dos charset = UTF8
display charset = UTF8
workgroup = WORKGROUP
server string = %h server (Samba, Ubuntu)
dns proxy = no
syslog = 0
panic action = /usr/share/samba/panic-action %d
server role = standalone server
passdb backend = tdbsam
obey pam restrictions = yes
unix password sync = yes
passwd program = /usr/bin/passwd %u
passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n
pam password change = yes
map to guest = bad user
usershare allow guests = yes
follow symlinks = yes
wide links = yes
unix extensions = no

[dgdsingen]
comment = dgdsingen samba
path = /home/dgdsingen
writable = yes
create mask = 0644
directory mask = 0755
security = user
valid users = dgdsingen

# 삼바 설정이 끝났으면, 설정이 제대로 되었는지 검사한다.
sudo testparm

# 문제가 없으면 삼바를 실행한다.
sudo service smbd restart
```

## Mount

```sh
# Linux에서 마운트
mount -t cifs //host/dgdsingen /mnt/smb -o username=$ID,password=$PW,iocharset=utf8,file_mode=0777,dir_mode=0777

# Windows에서 마운트
net use o: \\$IP\삼바공유폴더이름 비밀번호 /user:네트워크사용자아이디
# 혹은 탐색기 들어가서 '네트워크 위치 추가' - '\\test.com\dgdsingen' 입력

# 삭제
net use o: /delete

# 포트는 139, 445를 열어주면 된다.
```

### Windows에서 삼바 외부접근

우선 상황은 : smbd가 139, 445로 동작한다. ISP 쪽에서 139, 445 포트를 모두 막아놨으므로 포트를 바꿔야한다. 공유기에서 포트 포워딩을 9139 -> 139, 9445 -> 445 로 설정하면 외부에선 9139, 9445로 접근하면 된다. 리눅스에선 smb:/host:9139/username 으로 접속하면 된다. 안드로이드에서도 alternative port로 9139를 지정하면 된다. 그런데 윈도우는 139, 445 포트로 고정되어 있다. 9139, 9445 접속이 안된다. 그래서 아래와 같이 윈도우는 클라이언트 측에서 포트 포워딩 설정해 주어야 한다. 원리는 루프백 장치를 추가해 해당 장치의 139, 445 포트를 서버측 포트로 포워딩하는 것이다. 아래와 같이 루프백 장치를 추가한다.

Network adapters 선택 후 Action - Add legacy hardware

![img](images/server/image13.png)

![img](images/server/image8.png)

![img](images/server/image12.png)

![img](images/server/image9.png)

Network and Sharing Center - Change adapter settings - 방금 추가한 루프백 장치의 Properties

- 파일공유 기능을 Uninstall
- TCP/IPv4 - Advanced - WINS - Disable NetBIOS over TCP/IP 선택

IP는 아래와 같이 설정한다.

![img](images/server/image3.png)

그리고 나면 윈도우에서 445 포트를 점유하고 있는 서비스를 죽여야 한다.

Services로 가서 아래 스샷에 해당하는 Server란 놈을 Disable 시키고 reboot하자.

![img](images/server/image4.png)

이제 rinetd를 설정한다. [rinetd.7z](https://www.google.com/url?q=https://drive.google.com/open?id%3D1oKNUvRHpoOuUPqMZi9e7t2vOsbYHIhJo&sa=D&ust=1581426285583000)을 다운받으면,

rinetd.conf에 포트 포워딩이 설정되어 있다. 10.0.0.1의 특정 포트에 접속하면 test.com의 특정 포트로 포워딩한다는 설정이다.

- 10.0.0.1 139 test.com 9139
- 10.0.0.1 445 test.com 9445

start.bat에는 다음과 같이 설정되어 있다. rinetd 서비스 시작 배치다.

- set path=c:\rinetd\
- rinetd -c rinetd.conf

다만 이렇게 실행시 윈도우 콘솔창이 계속 떠있게 되므로, [hstart_4.2-bin.7z](https://www.google.com/url?q=http://docs.google.com/hstart_4.2-bin.7z&sa=D&ust=1581426285584000)를 사용해 hidden start 처리한다.

사실 이미 rinet.7z 안에 넣어놨다. hstart.exe가 hidden start 실행 파일이고,

hstart.bat가 hidden start로 rinetd start batch를 실행하는 배치다. 아래와 같이 설정되어 있다.

- cd c:\rinetd
- set path=c:\rinetd\
- hstart /NOCONSOLE "start.bat"



이제 Network Drive를 추가하자. 탐색기에서 아래와 같이 진행한다.

![img](images/server/image11.png)

![img](images/server/image6.png)

마지막으로 hstart의 자동 실행을 위해 Task Scheduler에 아래와 같이 Task를 걸어준다.

그러나 좀 더 편하고 싶다면 trigger를 at system startup으로, security options에서 when wether user is logged on or not으로 설정한다.

![img](images/server/image7.png)

![img](images/server/image5.png)

![img](images/server/image2.png)

![img](images/server/image1-1581422926105.png)

![img](images/server/image10.png)

## etc

그외의 다양한 삼바 설정법은 다음과 같다.

```sh
▶ 네트워크 사용자 추가/편집/삭제 하는 법
☞ 네트워크 사용자 추가하기
sudo smbpasswd -a 네트워크사용자아이디
sudo vim /etc/samba/smbusers 를 한 후에, 아래의 내용을 추가한다.
네트워크사용자아이디 = "network username"
☞ 네트워크 사용자 편집
sudo smbpasswd -a 네트워크사용자아이디
☞ 네트워크 사용자 삭제
sudo smbpasswd -x 네트워크사용자아이디
▶ 읽기 전용으로 폴더 공유하기 (인증=Yes)
sudo vim /etc/samba/smb.conf 로 설정파일을 열고
아래의 내용을 찾아서
security = user
아래처럼 바꾼다
security = user
username map = /etc/samba/smbusers
아래 내용의 앞에 있는 주석 표시를 제거한다.
;[homes]
;comment = Home Directories
;browseable = no
;valid users = %S
;writable = yes
삼바설정을 확인한다.
sudo testparm
삼바를 다시 시작한다.
sudo /etc/init.d/samba restart
▶ 읽기/쓰기 권한으로 폴더 공유하기 (인증=Yes)
설정파일 안의 지정된 폴더에 아래의 옵션을 넣어준다.
[MyFolder]
path = /home/my_id/my_shared_folder
writable = yes
▶ 그룹 폴더들을 읽기 전용으로 공유하기 (인증=Yes)
sudo mkdir /home/group
sudo chmod 777 /home/group/
sudo vim /etc/samba/smb.conf
설정파일에서 아래의 내용을
security = user
아래처럼 바꾸고
security = user
username map = /etc/samba/smbusers
설정파일의 끝에 아래의 내용을 추가한다.
[Group]
comment = Group Folder
path = /home/group
public = yes
writable = no
valid users = system_username1 system_username2
create mask = 0700
directory mask = 0700
force user = nobody
force group = nogroup
▶ 그룹 폴더를 읽기/쓰기 권한으로 공유하기 (인증=Yes)
설정파일의 끝에 추가된 그룹 설정을 아래처럼 바꾼다.
[Group]
comment = Group Folder
path = /home/group
public = yes
writable = yes
valid users = system_username1 system_username2
create mask = 0700
directory mask = 0700
force user = nobody
force group = nogroup
▶ 공개 폴더를 읽기 전용으로 공유하기 (인증=Yes)
[global]
security = share

[public]
comment = Public Folder
path = /home/public
public = yes
writable = no
create mask = 0777
directory mask = 0777
force user = nobody
force group = nogroup
아래처럼 nobody 계정을 만든다.
sudo smbpasswd nobody
▶ 공개 폴더를 읽기/쓰기 권한으로 공유하기 (인증=Yes)
[global]
security = share

[public]
comment = Public Folder
path = /home/public
public = yes
writable = yes
create mask = 0777
directory mask = 0777
force user = nobody
force group = nogroup
▶▶▶ 네트워크 프린터에서 인쇄하기
삼바 설정파일에 다음의 내용을 추가하고
printing = cups
printcap name = cups
프린터 관리 서비스를 다시 시작한다.
sudo /etc/init.d/cupsys restart
▶▶▶ 웹을 통해 삼바 설정 관리하기 : SWAT : Samba Web Administration Tool
▶ INETD과 SWAT 를 설치한다.
sudo apt-get install netkit-inetd
sudo apt-get install swat
inetd daemon 설정을 편집한다.
sudo vim /etc/inetd.conf
아래의 내용을 찾아서
<#off#> swat stream tcp nowait.400 root /usr/sbin/tcpd /usr/sbin/swat
아래처럼 바꾸어 준다.
swat stream tcp nowait.400 root /usr/sbin/swat swat
대몬을 재시작한다.
sudo /etc/init.d/inetd restart
만일, root비밀번호를 설정하지 않았다면, 아래처럼 설정한다.
sudo passwd root
```

# Transmission

아래와 같이 /etc/transmission-daemon/settings.json 설정 후 /etc/group에서 debian-transmission과 dgdsingen을 서로의 group에 넣어줌

```sh
sudo usermod -aG dgdsingen debian-transmission
sudo usermod -aG debian-transmission dgdsingen
```

```json
{
    "alt-speed-down": 50, 
    "alt-speed-enabled": false, 
    "alt-speed-time-begin": 540, 
    "alt-speed-time-day": 127, 
    "alt-speed-time-enabled": false, 
    "alt-speed-time-end": 1020, 
    "alt-speed-up": 50, 
    "bind-address-ipv4": "0.0.0.0", 
    "bind-address-ipv6": "::", 
    "blocklist-enabled": false, 
    "blocklist-url": "http://www.example.com/blocklist", 
    "cache-size-mb": 4, 
    "dht-enabled": true, 
    "download-dir": "/home/dgdsingen/downloads", 
    "download-limit": 100, 
    "download-limit-enabled": 0, 
    "download-queue-enabled": true, 
    "download-queue-size": 5, 
    "encryption": 1, 
    "idle-seeding-limit": 30, 
    "idle-seeding-limit-enabled": false, 
    "incomplete-dir": "/home/dgdsingen/downloads", 
    "incomplete-dir-enabled": false, 
    "lpd-enabled": false, 
    "max-peers-global": 200, 
    "message-level": 2, 
    "peer-congestion-algorithm": "", 
    "peer-id-ttl-hours": 6, 
    "peer-limit-global": 200, 
    "peer-limit-per-torrent": 50, 
    "peer-port": 51413, 
    "peer-port-random-high": 65535, 
    "peer-port-random-low": 49152, 
    "peer-port-random-on-start": false, 
    "peer-socket-tos": "default", 
    "pex-enabled": true, 
    "port-forwarding-enabled": false, 
    "preallocation": 1, 
    "prefetch-enabled": 1, 
    "queue-stalled-enabled": true, 
    "queue-stalled-minutes": 30, 
    "ratio-limit": 2, 
    "ratio-limit-enabled": false, 
    "rename-partial-files": true, 
    "rpc-authentication-required": true, 
    "rpc-bind-address": "0.0.0.0", 
    "rpc-enabled": true, 
    "rpc-password": "$PW",
    "rpc-port": 9091, 
    "rpc-url": "/transmission/", 
    "rpc-username": "transmission", 
    "rpc-whitelist": "test.com", 
    "rpc-whitelist-enabled": false, 
    "scrape-paused-torrents-enabled": true, 
    "script-torrent-done-enabled": false, 
    "script-torrent-done-filename": "", 
    "seed-queue-enabled": false, 
    "seed-queue-size": 10, 
    "speed-limit-down": 100, 
    "speed-limit-down-enabled": false, 
    "speed-limit-up": 10, 
    "speed-limit-up-enabled": true, 
    "start-added-torrents": true, 
    "trash-original-torrent-files": false, 
    "umask": 002, 
    "upload-limit": 100, 
    "upload-limit-enabled": 0, 
    "upload-slots-per-torrent": 14, 
    "utp-enabled": true
}
```

# IMAP

## Inbox, Sent, Trash 사용

### Flow

![image-20200209151811880](images/server/image-20200209151811880.png)

### gmail에서 Inbox, Sent, Trash가 아닌 메일 조회

-{label:[imap]/trash OR label:[imap]/sent OR label:inbox}

## Inbox, Trash 사용

이 경우 메일 전송시 always bcc myself를 켜두면 Sent가 Inbox로 통합된다.

### Flow

![image-20200209151847954](images/server/image-20200209151847954.png)

### gmail에서 Inbox, Trash가 아닌 메일 조회

-{label:[imap]/trash OR label:inbox}

# CalDAV, CardDAV

## SOGo

### Install Package

```sh
sudo apt-key adv --keyserver keys.gnupg.net --recv-key 0x810273C4
sudo apt-get update
sudo apt-get install sogo mysql-server apache2
```

### MySql

```sh
sudo mysql -u root -p
```

```sql
CREATE DATABASE 'sogo' CHARACTER SET='utf8';
CREATE USER 'sogo'@'localhost' IDENTIFIED BY 'sogopasswd';
GRANT ALL PRIVILEGES ON 'sogo'.* TO 'sogo'@'localhost' WITH GRANT OPTION;
USE sogo;
CREATE TABLE sogo_users (c_uid VARCHAR(60) PRIMARY KEY, c_name VARCHAR(60), c_password VARCHAR(32), c_cn VARCHAR(128), mail VARCHAR(128));
INSERT INTO sogo_users VALUES ('dgdsingen', 'dgdsingen', MD5('passwd'), 'dgdsingen example', 'dgdsingen@gmail.com');
FLUSH PRIVILEGES;
COMMIT;
quit
```

### Apache2

/etc/fstab

```sh
# for Ramdisk
tmpfs /var/log/apache2 tmpfs defaults,noatime,mode=1777 0 0
```

/etc/apache2/sites-available/sogo.conf

```sh
<VirtualHost *:8800>
  ServerName test.com
  ServerAlias sogo.local
  DocumentRoot /usr/lib/GNUstep/SOGo/WebServerResources/
  ErrorLog /var/log/apache2/error.log
  Customlog /var/log/apache2/access.log combined
  ServerSignature Off
 
  Alias /SOGo.woa/WebServerResources/ /usr/lib/GNUstep/SOGo/WebServerResources/
  Alias /SOGo/WebServerResources/ /usr/lib/GNUstep/SOGo/WebServerResources/
  AliasMatch /SOGo/so/ControlPanel/Products/(.*)/Resources/(.*) /usr/lib/GNUstep/SOGo/$1.SOGo/Resources/$2
 
  <Directory /usr/lib/GNUstep/SOGo/>
     AllowOverride None
     Order deny,allow
     Allow from all
 
     <IfModule expires_module>
     ExpiresActive On
     ExpiresDefault "access plus 1 year"
     </IfModule>
  </Directory>
 
 
  <LocationMatch "^/SOGo/so/ControlPanel/Products/.*UI/Resources/.*\.(jpg|png|gif|css|js)">
     SetHandler default-handler
  </LocationMatch>
 
  ProxyRequests Off
  SetEnv proxy-nokeepalive 1
  ProxyPreserveHost On
  ProxyPass /SOGo http://127.0.0.1:20000/SOGo retry=0
 
  <Proxy http://127.0.0.1:20000/SOGo>
     RequestHeader set "x-webobjects-server-port" "8800"
     RequestHeader set "x-webobjects-server-name" "test.com:8800"
     RequestHeader set "x-webobjects-server-url" "http://test.com:8800"
     RequestHeader set "x-webobjects-server-protocol" "HTTP/1.0"
     RequestHeader set "x-webobjects-remote-host" %{REMOTE_HOST}e env=REMOTE_HOST
     AddDefaultCharset UTF-8
     Order allow,deny
     Allow from all
  </Proxy>
 
  ## We use mod_rewrite to pass remote address to the SOGo proxy.
  # The remote address will appear in SOGo's log files and in the X-Forward
  # header of emails.
  RewriteEngine On
  RewriteRule ^/SOGo/(.*)$ /SOGo/$1 [env=REMOTE_HOST:%{REMOTE_ADDR},PT]
 
  Redirect permanent /index.html http://test.com:8800/SOGo
</VirtualHost>
```

/etc/apache2/ports.conf

```sh
# port 추가
LISTEN 8800

sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod rewrite
sudo a2ensite sogo.conf
```

### SOGo

/etc/fstab

```sh
# for Ramdisk
tmpfs /var/log/sogo tmpfs defaults,noatime,mode=1777 0 0
```

/etc/init.d/sogo

```sh
# for Ramdisk
if [ ! -d /var/log/sogo ]; then
  mkdir /var/log/sogo
fi
```

/etc/sogo/sogo.conf

```sh
{
 /* Database configuration (mysql:// or postgresql://) */
 SOGoProfileURL = "mysql://$ID:$PW@localhost:3306/sogo/sogo_user_profile";
 OCSFolderInfoURL = "mysql://$ID:$PW@localhost:3306/sogo/sogo_folder_info";
 OCSSessionsFolderURL = "mysql://$ID:$PW@localhost:3306/sogo/sogo_sessions_folder";

 /* Mail */
 SOGoDraftsFolderName = Drafts;
 SOGoSentFolderName = Sent;
 SOGoTrashFolderName = Trash;
 SOGoIMAPServer = 127.0.0.1:143;
 SOGoSieveServer = sieve://127.0.0.1:4190;
 SOGoSMTPServer = 127.0.0.1;
 SOGoMailDomain = domain.local;
 SOGoMailingMechanism = smtp;
 //SOGoForceExternalLoginWithEmail = NO;
 //SOGoMailSpoolPath = /var/spool/sogo;
 //NGImap4ConnectionStringSeparator = "/";

 /* Notifications */
 //SOGoAppointmentSendEMailNotifications = NO;
 //SOGoACLsSendEMailNotifications = NO;
 //SOGoFoldersSendEMailNotifications = NO;

 /* Authentication */
 //SOGoPasswordChangeEnabled = YES;

 /* LDAP authentication example */
 //SOGoUserSources = (
 //  {
 //   type = ldap;
 //   CNFieldName = cn;
 //   UIDFieldName = uid;
 //   IDFieldName = uid; // first field of the DN for direct binds
 //   bindFields = (uid, mail); // array of fields to use for indirect binds
 //   baseDN = "ou=users,dc=acme,dc=com";
 //   bindDN = "uid=sogo,ou=users,dc=acme,dc=com";
 //   bindPassword = qwerty;
 //   canAuthenticate = YES;
 //   displayName = "Shared Addresses";
 //   hostname = ldap://127.0.0.1:389;
 //   id = public;
 //   isAddressBook = YES;
 //  }
 //);


 /* SQL authentication example */
 /*  These database columns MUST be present in the view/table:
 \*   c_uid - will be used for authentication -  it's the username or username@domain.tld)
 \*   c_name - which can be identical to c_uid -  will be used to uniquely identify entries
 \*   c_password - password of the user, plain-text, md5 or sha encoded for now
 \*   c_cn - the user's common name - such as "John Doe"
 \*   mail - the user's mail address
 \*  See the installation guide for more details
 */
 SOGoUserSources =
  (
   {
    type = sql;
    id = users;
    viewURL = "mysql://$ID:$PW@127.0.0.1:3306/sogo/sogo_users";
    canAuthenticate = YES;
    isAddressBook = YES;
    userPasswordAlgorithm = md5;
   }
  );

 /* Web Interface */
 SOGoPageTitle = SOGo;
 SOGoVacationEnabled = YES;
 SOGoForwardEnabled = YES;
 SOGoSieveScriptsEnabled = YES;
 //SOGoMailAuxiliaryUserAccountsEnabled = YES;
 //SOGoTrustProxyAuthentication = NO;

 /* General */
 SOGoLanguage = English;
 SOGoTimeZone = Asia/Seoul;
 SOGoCalendarDefaultRoles = (
  PublicDAndTViewer,
  ConfidentialDAndTViewer
 );
 SOGoSuperUsernames = (Administrator); // This is an array - keep the parens!
 SxVMemLimit = 384;
 WOPidFile = "/var/run/sogo/sogo.pid";
 SOGoMemcachedHost = 127.0.0.1;

 /* Debug */
 //SOGoDebugRequests = YES;
 //SoDebugBaseURL = YES;
 //ImapDebugEnabled = YES;
 //LDAPDebugEnabled = YES;
 //PGDebugEnabled = YES;
 //MySQL4DebugEnabled = YES;
 //SOGoUIxDebugEnabled = YES;
 //WODontZipResponse = YES;
 //WOLogFile = /var/log/sogo/sogo.log;
}
```

### memcached

```sh
sudo apt-get install memcached
```

### Dovecot

- /etc/sogo/sogo.conf에서 login을 mysql 타도록 할 경우에는 굳이 ldap을 쓸 필요가 없으므로, dovecot을 설치하지 않아도 된다.
- 위에서의 mysql, apache2, sogo 설정은 dovecot이 필요없는 설정이다.

```sh
sudo apt-get install dovecot-common dovecot-imapd dovecot-dev dovecot-ldap
```

/etc/dovecot/dovecot.conf

```sh
mail_home = /var/mail/%u
mail_location = maildir:/var/mail/%u/mail
mail_uid = 999
mail_gid = 999

first_valid_uid = 999
first_valid_gid = 999

namespace inbox {
  inbox = yes
  location = maildir:/var/mail/%u/mail
  prefix =
  separator = /
  type = private
}

passdb {
  driver = ldap
  args = /etc/dovecot/dovecot-ldap-passdb.conf.ext
}

userdb {
  driver = ldap
  args = /etc/dovecot/dovecot-ldap-userdb.conf.ext
}

[/etc/dovecot/dovecot-ldap-passdb.conf.ext](https://www.google.com/url?q=http://docs.google.com/sogo.codeblock.12.html&sa=D&ust=1581425915809000)

hosts = localhost
auth_bind = yes
auth_bind_userdn = cn=%u,cn=Users,dc=gsg,dc=local
ldap_version = 3
 
base = cn=Users,dc=gsg,dc=local
pass_filter = (&(objectClass=person)(cn=%u)(mail=*))

[/etc/dovecot/dovecot-ldap-userdb.conf.ext](https://www.google.com/url?q=http://docs.google.com/sogo.codeblock.13.html&sa=D&ust=1581425915809000)

hosts = localhost
dn = cn=ldap,cn=Users,dc=gsg,dc=local
dnpass = [password]
ldap_version = 3
base = cn=Users,dc=gsg,dc=local
 
user_attrs = =uid=999,=gid=999,=home=/home/vmail/%u,=mail=/home/vmail/%u/mail,=master_user=%u
user_filter = (&(objectClass=person)(cn=%u)(mail=*))
 
\# Attributes and filter to get a list of all users
iterate_attrs = cn=user
iterate_filter = (objectClass=person)

groupadd –g 5000 vmail
useradd –m –u 5000 –g 5000 –s /bin/bash –d /var/mail vmail
adduser vmail mail
chown -R vmail:vmail /var/mail



cd /var/mail
mkdir johndoe
mkdir -p johndoe/Drafts johndoe/Sent johndoe/Trash
ln -s johndoe JohnDoe
ln -s johndoe johndoe@oc.local
ln -s johndoe JohnDoe@oc.local
chown -R vmail:vmail *
chown vmail:mail /var/log/dovecot.message



sudo service dovecot restart
```

### Add Calendar

important.json

```sh
{"Calendar": {"WebCalendars": {"dgdsingen:Calendar\/work": "http:\/\/test.com:8800\/SOGo\/dav\/dgdsingen\/Calendar\/work\/"}}}
```

```sh
sogo-tool user-preferences set settings dgdsingen Calendar -f important.json
```

이후 로그인 혹은 서버 리스타트 등. 시간이 좀 걸림.

### Error

#### No child available to handle incoming request!

- hang 걸린 child work가 많아서 그렇다. 간단히 서버 리스타트로 해결.

### Backup & Restore

#### Backup

- In order to backup the data of all, one or some users, you simply need to issue the following command:

sogo-tool backup output-directory ALL

or

sogo-tool backup output-directory user1 user2 ...

- The directory "output-directory" will be created automatically if it does not exist. The backup files will be put there, under the system identifier of the corresponding user. Each backup file will contain the different folders along with their respective acl, the set of user preferences and a copy of the user record found in the LDAP authentication database.

#### Restore

- In order to restore the user data, 3 steps are available from the "restore" command.
- To list the folders that can be restored for a specific user:

sogo-tool restore -l output-directory user

- This will list the different folder ids and corresponding display names. Once chosen, each directory can be restored with the following command:

sogo-tool restore -f folderid output-directory user

- This will restore the folder non-destructively, by restoring the items that were deleted after the backup. To restore all available folders, simpy type "ALL" in place of the folder id. Another variant of that command is to use "-F" instead of "-f", which will restore the data in a destructive manner, restoring the specified folder in the exact state it was when the backup command was issued. Any of those variants will always restore the access lists to how they were at the time of the backup.
- Finally, you can restore the user preferences by specifying this:

sogo-tool restore -p output-directory user

#### 구독

- [http://test.com:8800/SOGo/dav/dgdsingen/Contacts/personal/](https://www.google.com/url?q=http://test.com:8800/SOGo/dav/dgdsingen/Contacts/personal/&sa=D&ust=1581425915811000)
- [http://test.com:8800/SOGo/dav/dgdsingen/Calendar/personal/](https://www.google.com/url?q=http://test.com:8800/SOGo/dav/dgdsingen/Calendar/personal/&sa=D&ust=1581425915812000)
- [http://test.com:8800/SOGo/dav/dgdsingen/Calendar/630A-5245E700-5-375D6E80/](https://www.google.com/url?q=http://test.com:8800/SOGo/dav/dgdsingen/Calendar/630A-5245E700-5-375D6E80/&sa=D&ust=1581425915812000)

## Radicale

### Install

```sh
# version 0.07에는 thunderbird와 연동이 비정상인 오류가 있으므로 0.10 이상으로 확인
pip install radicale
```

### Config

/etc/radicale/config

```sh
[server]
# CalDAV server hostnames separated by a comma
# IPv4 syntax: address:port
# IPv6 syntax: [address]:port
# For example: 0.0.0.0:9999, [::]:9999
# IPv6 adresses are configured to only allow IPv6 connections
hosts = 0.0.0.0:5232
# Daemon flag
daemon = False
# File storing the PID in daemon mode
pid =
# SSL flag, enable HTTPS protocol
ssl = False
# SSL certificate path
certificate = /etc/apache2/ssl/server.crt
# SSL private key
key = /etc/apache2/ssl/server.key
# SSL Protocol used. See python's ssl module for available values
protocol = PROTOCOL_SSLv23
# Ciphers available. See python's ssl module for available ciphers
ciphers =
# Reverse DNS to resolve client address in logs
dns_lookup = True
# Root URL of Radicale (starting and ending with a slash)
base_prefix = /
# Possibility to allow URLs cleaned by a HTTP server, without the base_prefix
can_skip_base_prefix = False
# Message displayed in the client when a password is needed
realm = Radicale - Password Required
 
 
[encoding]
# Encoding for responding requests
request = utf-8
# Encoding for storing local collections
stock = utf-8
 
 
[auth]
# Authentication method
# Value: None | htpasswd | IMAP | LDAP | PAM | courier | http | remote_user | custom
type = htpasswd
 
# Custom authentication handler
custom_handler =
 
# Htpasswd filename
htpasswd_filename = /etc/radicale/users
# Htpasswd encryption method
# Value: plain | sha1 | crypt
htpasswd_encryption = plain
 
# LDAP server URL, with protocol and port
ldap_url = ldap://localhost:389/
# LDAP base path
ldap_base = ou=users,dc=example,dc=com
# LDAP login attribute
ldap_attribute = uid
# LDAP filter string
# placed as X in a query of the form (&(...)X)
# example: (objectCategory=Person)(objectClass=User)(memberOf=cn=calenderusers,ou=users,dc=example,dc=org)
# leave empty if no additional filter is needed
ldap_filter =
# LDAP dn for initial login, used if LDAP server does not allow anonymous searches
# Leave empty if searches are anonymous
ldap_binddn =
# LDAP password for initial login, used with ldap_binddn
ldap_password =
# LDAP scope of the search
ldap_scope = OneLevel
 
# IMAP Configuration
imap_hostname = localhost
imap_port = 143
imap_ssl = False
 
# PAM group user should be member of
pam_group_membership =
 
# Path to the Courier Authdaemon socket
courier_socket =
 
# HTTP authentication request URL endpoint
http_url =
# POST parameter to use for username
http_user_parameter =
# POST parameter to use for password
http_password_parameter =
 
 
[git]
# Git default options
committer = Radicale <radicale@example.com>
 
 
[rights]
# Rights backend
# Value: None | authenticated | owner_only | owner_write | from_file | custom
type = authenticated
 
# Custom rights handler
custom_handler =
 
# File for rights management from_file
file = /etc/radicale/rights
 
 
[storage]
# Storage backend
# Value: filesystem | multifilesystem | database | custom
type = filesystem
 
# Custom storage handler
custom_handler =
 
# Folder for storing local collections, created if not present
filesystem_folder = /etc/radicale/collections
 
# Database URL for SQLAlchemy
# dialect+driver://user:password@host/dbname[?key=value..]
# For example: sqlite:///var/db/radicale.db, postgresql://user:password@localhost/radicale
# See http://docs.sqlalchemy.org/en/rel_0_8/core/engines.html#sqlalchemy.create_engine
database_url =
 
 
[logging]
# Logging configuration file
# If no config is given, simple information is printed on the standard output
# For more information about the syntax of the configuration file, see:
# http://docs.python.org/library/logging.config.html
config = /etc/radicale/logging
# Set the default logging level to debug
debug = False
# Store all environment variables (including those set in the shell)
full_environment = False
 
 
# Additional HTTP headers
#[headers]
#Access-Control-Allow-Origin = *
```

/etc/radicale/logging

```sh
# Loggers, handlers and formatters keys
 
[loggers]
# Loggers names, main configuration slots
keys = root
 
[handlers]
# Logging handlers, defining logging output methods
keys = console,file
 
[formatters]
# Logging formatters
keys = simple,full
 
 
# Loggers
 
[logger_root]
# Root logger
level = INFO
handlers = console,file
 
 
# Handlers
 
[handler_console]
# Console handler
class = StreamHandler
level = INFO
args = (sys.stdout,)
formatter = simple
 
[handler_file]
# File handler
class = FileHandler
args = ('/var/log/radicale',)
formatter = full
 
 
# Formatters
 
[formatter_simple]
# Simple output format
format = %(message)s
 
[formatter_full]
# Full output format
format = %(asctime)s - %(levelname)s: %(message)s
```

/etc/radicale/rights

```sh
[dgdsingen]
user: ^dgdsingen.*$
collection: .*
permission: rw
```

/etc/radicale/users

```sh
user:passwd
# open port
sudo ufw allow 5232
```

실행

```sh
radicale -d
```

접근 경로는 아래와 같이 결정된다.

- CalDav : http://test.com:5232/dgdsingen/private.ics/
- CardDav : http://test.com:5232/dgdsingen/contacts.vcf/

## Thunderbird

> [Thunderbird tags](http://kb.mozillazine.org/Tags)
>
> [Thunderbird logs](https://www.lifewire.com/pop-imap-smtp-traffic-thunderbird-1173156)
>
> http://test.com:8800/SOGo/dav/dgdsingen/Contacts/personal/
>
> http://test.com:8800/SOGo/dav/dgdsingen/Calendar/personal/
>
> http://test.com:8800/SOGo/dav/dgdsingen/Calendar/630A-5245E700-5-375D6E80/

### CardDav

부가기능 'SOGo Connector' 설치 : sogo-connector-31.0.1.xpi.7z

### CalDav

부가기능 'Lightning' 설치

### Edit Refresh Interval

http://superuser.com/questions/591086/increase-calendar-refresh-interval-beyond-defaults-in-thunderbird-lightning

설정 - 고급 - 설정 편집

calendar.reg로 검색하여 refreshInterval 수정



## 대한민국 공휴일

https://www.google.com/calendar/dav/ko.south_korea#holiday@group.v.calendar.google.com/events

## 구독 방법

URL에서 ics 경로를 빼주면 하나의 계정으로 여러 개의 cal을 고를 수 있다.

http://test.com:5232/dgdsingen/

http://test.com:8800/SOGo/dav/dgdsingen/Calendar/

contacts도 마찬가지.

http://test.com:8800/SOGo/dav/dgdsingen/Contacts/

# Tiny Tiny RSS

```sh
#30분마다 feed update
*/30 * * * * php /var/www/html/ttrss/update.php --feeds --quiet
```



# References

## DNS
- [ICANN Lookup](https://lookup.icann.org/) 
- [DNS leak test](https://www.dnsleaktest.com/) 
- [Network Tools: DNS,IP,Email](https://mxtoolbox.com/SuperTool.aspx) 
- [1.1.1.1 DNS](https://1.1.1.1/) 
- [DuckDNS Setup](https://www.thefanclub.co.za/how-to/free-ip-ddns-service-duck-dns-setup-script-linux-and-osx) 
- [SPF 등록 가이드](https://helpdesk.midasinsight.com/hc/ko/articles/900000287723--%EA%B0%80%EC%9D%B4%EB%93%9C-SPF-%EB%93%B1%EB%A1%9D-%EA%B0%80%EC%9D%B4%EB%93%9C) 
- [Email 보안 강화 기능 소개(SPF)](https://meetup.toast.com/posts/244) 
- [메일 전송 보장 및 스푸핑 방지하기(SPF)](https://support.google.com/a/answer/33786?hl=ko) 

