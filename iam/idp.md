# IdP (Identity Provider)

대표적으로 사용되는 개방형 표준 프로토콜 3가지를 나열하면 아래와 같다.

- SAML
    - Client를 통해서 Server끼리 전문 교환 (Client에서 Server에 Post 호출을 하고 응답을 받아, 다른 Server에 정보를 다시 Post로 넘겨주는 등의 구조)
    - Client가 사실상 웹 브라우저라서 웹 환경에서 사용 가능하며, 앱 지원이 쉽지 않다.
- OAuth, OIDC
    - Server to Server 호출로 전문 교환 (즉 Server끼리 호출할 수 있게 방화벽 해제가 필요함)
    - Server to Server 호출로 인증/인가를 진행하므로, 웹/앱/서버/API 등 다양한 환경 지원이 가능하다.
    - OAuth는 AuthZ(Authorization)을 담당하며, 여기에 AuthN(Authentication)이 추가된 것이 OIDC라고 보면 된다.



# OAuth, OIDC

> [OAuth](https://oauth.net/2/) 
>
> [OAuth - IETF](https://tools.ietf.org/wg/oauth/) 
>
> > https://tools.ietf.org/html/rfc6749
> >
> > https://tools.ietf.org/html/rfc8252
> >
> > https://tools.ietf.org/html/rfc7519
>
> [OAuth 2 Explained](https://www.youtube.com/playlist?list=PLuHgQVnccGMA4guyznDlykFJh28_R08Q-)
>
> [What is OAuth really all about - YouTube](https://www.youtube.com/watch?v=t4-416mg6iU) 
>
> [WEB2 - OAuth 2.0 - YouTube](https://www.youtube.com/playlist?list=PLuHgQVnccGMA4guyznDlykFJh28_R08Q-) 
>
> [OAuth 2를 이용한 SSO 환경 구축 (1/2)](http://www.nextree.co.kr/oauth-2reul-iyonghan-sso-hwangyeong-gucug-1-2/) 
>
> [OAuth 2를 이용한 SSO 환경 구축 (2/2)](http://www.nextree.co.kr/oauth-2reul-iyonghan-sso-hwangyeong-gucug-2-2/) 
>
> [Tutorial - Spring Boot and OAuth2](https://spring.io/guides/tutorials/spring-boot-oauth2/) 
>
> [OAuth2 Boot - Deprecated](https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/htmlsingle/) 
>
> [GitHub - PacktPublishing/OAuth-2.0-Cookbook - Deprecated](https://github.com/PacktPublishing/OAuth-2.0-Cookbook) 
>
> [OAuth 2.0 Migration Guide · spring-projects/spring-security Wiki](https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide) 
>
> [jgrandja/spring-security-oauth-5-2-migrate](https://github.com/jgrandja/spring-security-oauth-5-2-migrate) 
>
> [Spring Security OAuth2.0 파헤치기! - 1(Authorization Server)](https://coding-start.tistory.com/158) 
>
> [Spring Security OAuth2.0 파헤치기! - 2(Authorization Server + Resource Server)](https://coding-start.tistory.com/160?category=738631) 
>
> [Spring Security OAuth2.0 파헤치기! - 3(authorization server + resource server + client)](https://coding-start.tistory.com/163?category=738631) 
>
> [Spring Security and Keycloak to Secure Spring Boot - A First Look](https://www.thomasvitale.com/spring-security-keycloak/) 



# SAML

> [SAML 기반의 web sso 원리 정리](http://bcho.tistory.com/m/755) 



# LDAP

## FreeIPA

우선 LDAP에서 사용할 도메인을 미리 등록해두자.

```sh
# Host의 /etc/hosts에 아래와 같이 등록
127.0.0.1 ldap.t.com
127.0.0.1 t.com
```

이후 아래와 같이 컨테이너를 실행하면 몇 가지 설정 사항들을 물어본다.
기본 설정값을 미리 넣은 것이니 그냥 엔터 치고 넘어가자. 다만 맨 마지막 이 설정으로 시스템 구성하겠냐고 물어볼 때 `yes`를 입력한다.

```sh
docker run --name freeipa -ti \
  --cap-add NET_ADMIN \
  --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  -v /dev/urandom:/dev/random:ro \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  -v $PWD/data:/data:Z \
  -h ldap.t.com \
  -p 443:443 -p 389:389 -p 636:636 \
  -e PASSWORD=12345678 -e DEBUG_NO_EXIT=1 \
  freeipa/freeipa-server:rocky-8 #-r t.com \
  #-U --admin-password=12345678 --ds-password=12345678
```

https://ldap.t.com 로 접속해서 admin / 12345678 로그인하면 FreeIPA 콘솔 사용 가능.

LDAP 연동을 테스트 해보고 싶다면 아래와 같이 진행한다.



1. **Apache Directory Studio로 FreeIPA에 LDAP 접속** 

- Apache Directory Studio > New LDAP Connection
    - Hostname: `localhost` 
    - Port: `389` 
    - Bind DN or user: `uid=admin,cn=users,cn=accounts,dc=t,dc=com` 
    - Bind Password: `12345678` 

이후 FreeIPA의 모든 LDAP Objects를 Root부터 확인해볼 수 있다.



2. **Keycloak으로 FreeIPA에 LDAP 접속** 

우선 Keycloak을 같은 Docker Network 상에서 실행한다.

```sh
docker run --name keycloak -p 8080:8080 jboss/keycloak:15.0.2
docker exec keycloak /opt/jboss/keycloak/bin/add-user-keycloak.sh -u admin -p 1212
docker restart keycloak
```

- Keycloak > User Federation > Add provider > ldap
    - Vendor: `Red Hat Directory Server` 
    - Connection URL: `ldap://172.17.0.2:389` (FreeIPA Container의 IP를 넣어준다)
    - Users DN: `cn=users,cn=accounts,dc=t,dc=com` 
    - Bind DN: `uid=admin,cn=users,cn=accounts,dc=t,dc=com` 
    - Bind Credential: `12345678` 



**References:** 

> https://github.com/freeipa/freeipa-container
>
> https://hub.docker.com/r/freeipa/freeipa-server/tags
>
> [FreeIPA Install Docker in Ubuntu 18.04](https://lasel.kr/archives/576) 
>
> [Radius – Ldap 연동](https://lasel.kr/archives/584) 



## AD (Active Directory)

- Windows Server에서 User, Group 등의 정보를 체계적(Tree 형태)으로 관리하는 서비스
- Domain(Parent > Child) > OU(Organizational Unit) >  Group 등의 단위로 Windows 기반 Computer들의 리소스 관리 가능

### 용어

- AD : Active Directory 
- AD DS : AD Domain Services. 보통 AD라고 하면 AD DS를 의미한다. AD 구성시 해당 역할을 추가한 후 설정함
- AD DC : AD Domain Controller. Domain 당 최소 1개 이상의 DC가 존재하며 해당 DC가 Domain을 관리한다. 
- AD FS : AD Federation Services. 도메인 페더레이션 및 웹 기반 SSO 관리
- AD LDS : AD Lightweight Directory Services. LDAP 방식 구현
- AD RMS : AD rights Management Services. AD 정보보호 및 권한 관리
- AD CS : AD Certificate Services. 공개키 및 인증서 관리
- [Trust](https://blogs.msmvps.com/acefekay/2016/11/02/active-directory-trusts/) : 서로 다른 Domain의 AD를 단/양방향으로 연결하여 AD 서비스를 확장한다. Trust 종류로는 부모 및 자식 Trust, 트리-루트 Trust, 포리스트 Trust, 영역 Trust, 외부 Trust가 있다.
- GC : Global Catalog. Forest 내 모든 도메인에 있는 Object에 대한 정보를 모아놓은 DB다. Global Catalog를 사용해 자주 사용하는 Object의 사본을 저장하여 효율적인 검색을 수행한다. GC의 LDAP Query 가능 범위는 한 Forest 내에 한정된다. 서로 다른 두 Forest 간 Trust를 맺더라도 GC에 Cross Forest로 LDAP Query를 할 수는 없다. (예를 들어 Forest A의 GC에 Forest B의 LDAP Query는 안됨)
- Domain : aaa.com, bbb.com과 같은 말 그대로 도메인. 
- Tree : a1.aaa.com, b1.bbb.com과 같이 Domain 아래 Sub Domain으로 Parent - Child가 구성되면 즉 Tree 구조를 이룬다. 
- Forest : Tree와 Tree가 서로 Trust 관계를 맺으면 Forest가 된다. 

### DS (Domain Services)

#### Install on AWS

##### Install Windows Server on EC2

- AWS Console > EC2 > 인스턴스 시작 > Select Windows Server AMI > Security Group은 3389 포트 열린 상태로 설정 > pem key는 RDP 접속시 사용되니 잘 받아두자
- AWS Console > EC2 > Select the instance > 연결 > RDP 클라이언트 > 암호 가져오기 > Select pem key > RDP 파일 다운로드 및 실행 > Password 입력 > RDP 접속됨

##### Install AD DS

- Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Active Directory Domain Services > Select Restart the destination server automatically if required > Install
    - DS 설치시 왠만하면 DNS Server도 깔자. DC로 쓰든 Trust를 맺든 DNS 설정이 필요하다. 
- (Server Manager > Notifications) > Promote this server to a domain controller > Add a new forest > Root domain name: test.com > Install > OS wil be restarted automatically
    - DC로 올리면 PC Name을 바꾸기 매우 어렵다. PC Name이 Sub Domain으로 사용되니 가능하면 PC Name을 바꿔놓고 DC로 올리자. 
- ~~Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Active Directory Certificate Services > Select Restart the destination server automatically if required > Install~~
- ~~(Server Manager > Notifications) > Configure Active Directory Certificate Services on the destination server > Select Certification Authority > Select Enterprise CA > Select Root CA > Select Create a new private key > Configure~~
- ~~Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Active Directory Lightweight Directory Services > Select Restart the destination server automatically if required > Install~~
- ~~(Server Manager > Notifications) > Run the Active Directory Lightweight Directory Services Setup Wizard~~

##### Config AD DS

- Control Panel > System and Security > Windows Defender Firewall > Turn Windows Defender Firewall on or off > Turn off all
- Administrative Tools > Active Directory Users and Computers > test.com 우클릭 > New > Organizational Unit > Name: ou-test
- ou-test 우클릭 > New > User > First Name: user-test > User logon name: user-test > Uncheck User must change password at next logon > Finish



### Trust

- [방화벽 해제](https://docs.microsoft.com/ko-kr/troubleshoot/windows-server/identity/config-firewall-for-ad-domains-and-trusts) : SG, Windows Firewall
    - 53/TCP/UDP : DNS
    - 88/TCP/UDP : Kerberos
    - 135/TCP : RPC
    - 389/TCP/UDP : LDAP
    - 445/TCP : SMB
    - 636/TCP : LDAP SSL
    - 3268/TCP : LDAP GC
    - 3269/TCP : LDAP GC SSL
    - 1024-65535/TCP : LSA용 RPC, SAM, NetLogon
    - 근데 이걸로도 안되는 경우가 있어서 전체 트래픽 오픈했더니 잘됨.
- DNS 설정
    - 아래 2개의 서버를 Trust 맺는다고 가정한다. 
        - Server A = 10.0.0.100 = pc1.a.a, AD Domain = a.a
        - Server B = 10.0.0.101 = pc2.b.b, AD Domain = b.b
    - Trust를 위해서는 nslookup이 정상적으로 되어야 한다. 우선 각 서버에서 DNS 서버를 자기 자신의 DNS server를 보도록 설정한다.
        - ~~Control Panel > Network and Internet > Network Connections > Right Click Adapter > Properties > IPv4 > Properties > Use the following IP address~~
            - ~~IP address : A = 10.0.0.100, B = 10.0.0.101~~
            - ~~Subnet mask : 255.255.255.0~~
            - ~~Default gateway :10.0.0.1~~
        - Control Panel > Network and Internet > Network Connections > Right Click Adapter > Properties > IPv4 > Properties > Use the following DNS server addresses
            - Preferred DNS server : A = 10.0.0.100, B = 10.0.0.101 (각자 자기 자신의 DNS Server)
    - PC Name만으로 질의하기 위한 설정. 만약 pc1\accountA로 로그인한다면 pc1의 실제 도메인이 pc1.a.a이라는 것을 알아야 한다.
        - Control Panel > Network and Internet > Network Connections > Right click adapter > Properties > IPv4 > Properties > Advanced > DNS > Append these DNS suffiexes > A = a.a, b.b, B = b.b, a.a (예를 들어 pc1.a.a을 pc1으로 조회하기 위해 DNS suffix로 a.a 추가하고 우선순위 올림)
    - hosts 파일 수정 (DNS Server 설치해서 설정하는 경우 이 부분은 건너뛰어도 됨)
        - C:\Windows\System32\drivers\etc\hosts 에 A = 10.0.0.101 b.b, B = 10.0.0.100 a.a 등록
    - Windows DNS Server 설정
        - Administrative Tools > DNS > Right Click Forward Lookup Zones > New Zone > Primary Zone > Zone Name: 같은 도메인으로 묶을거라면 A/B = a.a
        - 이후 같은 도메인 내 A/B 서버 2개가 존재할 경우 AD Domain인 a.a로 A 레코드 10.0.0.101(A 서버), 10.0.0.100(B 서버) 모두 등록한다. nslookup a.a 결과로 10.0.0.101(A 서버), 10.0.0.100(B 서버) 모두 나오게 된다. 또한 Sub Domain으로 dc01.a.a(A 서버)에 A 레코드 10.0.0.101, dc02.a.a(B 서버)에 A 레코드 10.0.0.100을 등록한다. nslookup dc01.a.a 결과로 10.0.0.101(A 서버), nslookup dc02.a.a 결과로 10.0.0.100(B 서버)가 나오게 된다. 
        - Administrative Tools > DNS > Conditional Forwarders 우클릭 > New Conditional Forwarder > DNS Domain: A = b.b, B = a.a, IP Address: A = 10.0.0.101, B = 10.0.0.100
        - Administrative Tools > DNS > Conditional Forwarders > Domain 우클릭 > Properties > Edit > 입력한 IP가 초록색 체크박스로 정상인지 확인
    - 이후 nslookup으로 AD Domain 호출시 그 도메인 내에 묶여야 하는 Server IP가 모두 나오는지, 각 Server가 가지는 Sub Domain으로도 IP 잘 나오는지, 각 Server의 hostname으로도 IP가 잘 나오는지 보면 된다. 
- Trust 설정
    - Administrative Tools > Active Directory Domains and Trusts > Domain 우클릭 > Properties > Trust > New Trust > 상대 Domain 입력 > Forest trust > Two-way > Both this domain and the specified domain > 상대 Domain Admins 그룹에 속한 계정의 Username/Password 입력 > Forest-wide authentication
    - 만약 One-way Trust 설정을 하고자 한다면 Outgoing, Incoming 방향을 확인하자. Trust 방향이 A -> B라면 엑세스 방향은 B -> A다. 
- Trust 테스트. A -> B One-way Trust인 경우를 가정한다. 
    - Client용 Windows PC 생성 > Control Panel > System and Security > System > Advanced system settings > Computer Name > Change > Domain: a.a > Reboot
    - Control Panel > Control Panel > Network and Internet > Network Connections > Right click adapter > Properties > IPv4 > Properties > Use the following DNS server addresses > Preferred DNS server: 10.0.0.100
    - 즉 Client in -> A Outgoing -> B Incoming의 흐름으로 구성된 상태다. 
    - Client에 pc1/accountA 로그인하여 pc2/accountB 커맨드를 실행해본다. 
        - runas /noprofie /user:pc2\accountB cmd



### Cross Domain LDAP Query

- DC를 서로 다른 Forest로 만든 후 Trust 맺어봤자 Cross Domain LDAP Query는 되지 않는다. 예를 들어 Some Application => Forest A(DC A)로 로그인하여 Forest B(DC B)를 LDAP Query하려고 하면 그런 Object가 없다고 나온다. 즉 Forest A에 대한 LDAP Query는 Forest A에 속한 DC나 GC에 Query해야만 알 수 있다. Forest B에 대한 LDAP Query도 Forest B에 속한 DC나 GC에 해야 한다. 
- 만약 Cross Domain LDAP Query를 하고 싶다면 DC A와 DC B를 같은 Forest로 묶어주자. 
- 우선 DC A를 Forest Root로 생성해준다. 이후 DC A에 DC A/B에 대한 DNS 설정을 해준다. 예를 들어 DC A에서 nslookup으로 a.a = DC A's IP, b.b = DC B's IP가 잘 나오면 됐다. 
- 이제 DC B에서 DNS 서버를 DC A로 바라보게 하고, DS를 설치한 뒤 설정시 새로운 Forest가 아닌 기존 Forest > Tree type 선택하여 DC A의 Forest에 가입한다. 
- 그러면 a.a와 b.b는 같은 Forest 내에 속하고 Two way Trust 상태가 되며, 이제 Cross Domain LDAP Query 가능하다. 다만 389가 아닌 LDAP GC인 3268 포트로 Query 하자. 



### OU별 권한 관리

- Administrative Tools > Active Directory Users and Computers > View > Advanced Features
- Right click ou > Properties > Security > 특정 Group이나 User > Read Deny



### FS (Federation Services)

```
+-IDP---------------------+
| +-------+               |
| | AD DC |               |
| +-------+               |
|     |                   |
|     | Trust             |
|     |                   |
| +-------+    +-------+  | SAML +--------+
| | AD DC +----+ AD FS +--+------+ Client |
| +-------+    +-------+  |      +----+---+
+-------------------------+           |
                                      |
        +------------------+          |
        | Service Provider +----------+
        +------------------+ AuthN, AuthZ
```

- AD FS에서 DNS를 AD DC의 IP로 변경한다. 
    - `C:\Windows\System32\control.exe ncpa.cpl`
- AD FS > Control Panel > System and Security > System > Advanced system settings > Computer Name > Change > Domain을 AD DC의 것으로 변경 > AD DC의 계정으로 로그인 > Reboot
- AD FS > Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Web Server (IIS) > Select Restart the destination server automatically if required > Install
- AD FS > Administrative Tools > Internet Information Services (IIS) Manager > Select AD FS Server > Select Server Certificates > Create Self-Signed Certificate > Name: ADFS, Store: Web Hosting
    - Select Certificate > Export
- AD FS > Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Active Directory Federation Services > Select Restart the destination server automatically if required > Install
- AD FS > (Server Manager > Notifications) > Configure the federation service on this server > Select Create the first federation sever in a federation server farm > AD DC의 계정으로 로그인 > Import SSL Certificate > 위에서 만든 Certificate 선택 > Federation Service Display Name: AD-FS > Specify Service Account: AD DC의 계정
- AD FS > Administrative Tools > Internet Information Services (IIS) Manager > AD FS Server > Sites > Right click Default Web Site > Bindings > Add > Type: https, SSL certificate: 위에서 생성한 Web Hosting용 SSL Certificate
    - 정신건강을 위해 IE 말고 다른 브라우저를 쓰자
    - powershell > `wget "https://download.mozilla.org/?product=firefox-latest-ssl&os=win64&lang=ko" -o firefox.exe`
    - powershell > `Set-AdfsProperties -EnableIdpInitiatedSignonPage $true`
    - https://adfs.test.com/adfs/ls/idpinitiatedsignon 접속 > AD DC의 계정으로 로그인
    - IdP metadata file = https://adfs.test.com/FederationMetadata/2007-06/FederationMetadata.xml



### Linux를 AD 도메인에 가입시키기

#### in RHEL

```sh
# install packages
sudo yum install -y realmd oddjob oddjob-mkhomedir sssd samba-common-tools samba-common adcli krb5-workstation openldap-clients policycoreutils-python
sudo sssd adcli

# set DNS server
sudo vi /etc/resolv.conf
nameserver 10.70.20.220 # AD IP
search a.com # AD Domain

# add AD's domain/ip
sudo vi /etc/hosts
10.70.20.220 a.com a # AD Domain, NetBios

# config samba
sudo vi /etc/samba/smb.conf
[global]
security = ads
realm = a.com # 도메인 이름(ex. uk.com)
workgroup = a # 도메인 BIOS ID(ex. UK)
idmap uid = 10000-20000
idmap gid = 10000-20000
winbind enum users = yes
winbind enum group = yes
template homedir = /home/%D/%U template
shell = /bin/bash client use spnego = yes
client ntlmv2 auth = yes
encrypt passwords = yes
winbind use default domain = yes
restrict anonymous = 2
kerberos method = secrets and keytab
winbind refresh tickets = true

# discover AD
sudo realm discover a.com

# if AD server is discovered successfully,
#kinit accountA@a.com

# join AD
sudo realm join a.com -U accountA@a.com -v

# test ldap query
ldapsearch -x -b 'OU=test-ou,DC=test,DC=com' -H ldap://a.com -D 'CN=accountA,CN=Users,DC=a,DC=com' -W "objectclass=user"

# leave ad 
sudo realm leave a.com
```



### DC Replication

- AD DC를 하나 만든다. 해당 도메인이 a.com 이라고 가정한다. 
- 복제용 서버는 우선 AD DS부터 설치 후 Configure로 들어간다. 
    - Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Active Directory Domain Services > Add a domain controller to an existing domain > Domain: a.com > Check DNS server, Read only domain controller (RODC) > Select Restart the destination server automatically if required > Install
    - 설치시 Read-only Domain Controllers에 해당하는 User를 설정해주거나, 아니면 나중에 마스터 AD DC에서 User 하나 추가해서 Member Of에서 Read-only Domain Controllers 권한을 부여한 계정을 사용하자. 해당 계정으로 Windows Server에 접속해서 AD Users를 수정/삭제할 수 없는 것을 확인할 수 있다. 



### DC Domain 가입

- test.com이라는 Domain이 있고 그 도메인의 Root가 dc01이라는 PC Name을 가진 서버라고 가정한다. 
- 여기에 dc02 서버를 Domain 가입시키는 경우 우선 DNS 설정을 진행한다. dc01.test.com이 DNS resolve 될 수 있도록 설정하자. 
- dc01의 DNS Server에 A Record 설정해두고, dc02의 IPv4 설정에서 dc01의 DNS Server IP 주소를 입력한다. 
- 만약 dc01 DNS Server에 SRV Record가 없는 경우 수기로 등록해주고, dc02 IPv4 설정에서 DNS에 Preferred로 dc01 하나만 등록하자. (Secondary에 dc02 넣지 않고 비움)
- 그리고 가입하려는 Server에서 Domain DC의 계정 넣고 로그인할때 user@test.com 말고 test\user 로 해보자. 
- "The domain join cannot be completed because the SID of the domain you attempted to join was identical to the SID of this machine. This is a symptom of an improperly cloned operating system install. You should run Sysprep on this machine in order to generate a new machine SID." 라고 뜨는 경우가 있다. 복제에 의해 만들어진 Windows라면 SID가 겹친다고 하는 것으로 보인다. (예를 들어 amazon default AMI로 첫번째 Windows Server를 만들고, 해당 서버를 설정한 뒤에 그 스냅샷을 떠서 2번째 Windows Server를 만들면 두번째 서버는 동일한 SID를 가진 서버로 생성되는 것으로 보인다) 해당 증상이 발생하는 PC에서 C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /reboot 실행한다. (AWS에서는 별도의 방법으로 실행해야 한다. 그냥 Windows Sysprep 실행하면 그 이후부터 EC2 접속 불가)



### Replication 및 Trust 종류 선택

- Replication은 데이터를 그대로 복제받는 것이기에 별도의 AD가 아니라 Master AD의 Read Only용 Slave AD 정도로 봐도 무방하다. 
- 별도의 도메인에서 자체 AD 관리하면서 두 도메인을 연동해야 한다면, Replication 대신 Cross Domain LDAP Query가 필요하다. 그런데 GC의 Query 범위는 Forest 내로 한정되어 있으니 여러 Domain을 하나의 Forest로 묶어야 하므로 Forest 간 Trust인 Forest/External Trust는 버린다. 
- 그럼 Tree Root Trust or Parent Child Trust 둘 중 하나인데, Parent Child Trust는 하나의 Root Domain으로부터 Sub Domain 형태로 가야한다. 독립적인 도메인 체계를 사용할거라면 Tree Root Trust만 남는다. 
- 회사를 하나의 Forest로 구성하되, 계열사별로 다른 Domain을 가져가며 하나의 회사 Forest에 모두 들어가는 형태로 설계한다. 
- 그러나 기구축된 AD들이 서로 다른 Forest로 이미 만들어져 있다면? 하나의 Forest로 병합 가능한가?



### GC에 Attribute 추가

- 우선 AD Schema부터 설치하자
    - Powershell > `regsvr32 schmmgmt.dll` 
- Powershell > `mmc` 
    - File > Add/Remove Sanp-in
    - Add AD Schema
    - AD Schema > Attributes
    - Double click attribute (ex: department)
    - Check 'Replicate this attribute to the Global Catalog'
- 다만 이렇게 해도 같은 Forest 내 다른 Domain에서 ldap query해도 group의 member 속성은 나오지 않을 것이다. 그럼 group과 user가 매핑되지 않는다. 이는 group scope이 global인 경우에는 Global Catalog로 member 속성이 전달되지 않기 때문이다. GC에서 group의 member 속성을 모두 공유하고 싶다면 group scope을 universal로 변경한다.



### Config LDAPS

- Server Manager > Add roles and features > Role-based or feature-based installation > Select a server from the server pool > Select the server > Select Active Directory Certificate Services > Role services: Certification Authority > Select Restart the destination server automatically if required > Install
- (Server Manager > Notifications) > Configure Active Directory Certificate Services on the destination server > Select Role Services to configure: Certification Authority > Specify the setup type of the CA: Enterprise CA > Specify the type of the CA: Root CA > Specify the type of the private key: Create a new private key > Specify the cryptographic options: SHA256 > Configure
- `certsrv.msc` > Right click Root CA > Properties > General > View Certificate > Details > Copy to File > Export File Format: DER encoded binary X.509 (.CER)
- export된 some.cer 인증서 파일을 Client에 import하면 LDAPS 636/TCP 통신이 잘 되는 것을 확인할 수 있다. 
- 만약 LDAPS 통신이 되지 않는 경우 AD 서버에 들어가서 ldp.exe > Connection > Connect > localhost:636 으로 테스트해본다. 
- 389가 잘 되는데 636이 안된다면 Certificate 에러일 가능성이 높다. 그럼 mmc.exe > File > Add/Remove Snap-in > Certificates 추가 > Personal > Certificates > More Actions > All Tasks > Requiest New Certificate 로 현재 PC Name과 Domain으로 이루어진 Certificate를 발급한다. (예를 들면 dc01.test.com)
- 예를 들어 Confluence(Crowd) - AD 연동시 `java.security.cert.CertificateException: No subject alternative DNS name matching <hostname> found` 에러가 발생하는 경우 https://confluence.atlassian.com/jirakb/java-security-cert-certificateexception-no-subject-alternative-dns-name-matching-hostname-found-297669411.html 를 참조하자
- 그땐 그냥 AD Domain과 동일하게 DNS A or CNAME Record로 Domain 등록하면 잘 된다. 



### Synology NAS와 LDAP 연동

- 프로파일 설정이 기본적으로 Synology NAS의 LDAP에 맞게 설정되어 있다. 이것을 AD에 맞게 "사용자 지정"으로 매핑 대상을 바꿔준다. ([Synology Community 참조](https://community.synology.com/enu/forum/1/post/131822))
    - filter
        - passwd - (objectClass=user)
        - shadow - (objectClass=user)
        - group - (objectClass=group)
    - group
        - cn - sAMAccountName
        - gidNumber - HASH(objectGUID)
        - memberUid - member
    - passwd
        - uidNumber - HASH(userPrincipalName)
        - uid - sAMAccountName
        - gidNumber - primaryGroupID
    - shadow
        - uid - sAMAccountName
        - userPassword - unicodePwd



### Issues

- 도메인 가입시 DC에서 PC Name이 15자를 초과하면 잘려서 보이는 이슈가 있다. 이 경우 PC Name이 다른 PC들인데 15자로 자르면 같은 이름이 되는 것들끼리 충돌이 일어나 서로 1대씩 도메인 조인이 풀리는 것으로 보이는 현상이 있었다. hostname은 15자 이내로 unique하게 지정하도록 하자. 
- Apache Directory Studio 등으로 export 할 때 count limit이 1000으로 고정되는 경우가 있다. export option에서 아무리 count를 늘려도 1000개만 나온다면, connection option에 count limit이 1000으로 되어 있는지 확인해보자.



### References

> [AD 개념](https://forgarden.tistory.com/69)
>
> [AWS AD를 EC2에 설치하는 방법](https://st-soul.tistory.com/70)
>
> [ADFS 구성 방법 및 이중화 (1)](https://practice.hooniworld.io/entry/ADFS-%EA%B5%AC%EC%84%B1-1)
>
> [ADFS 구성 방법 및 이중화 (2)](https://practice.hooniworld.io/entry/ADFS-%EA%B5%AC%EC%84%B1-%EB%B0%A9%EB%B2%95-%EB%B0%8F-%EC%9D%B4%EC%A4%91%ED%99%94-2)
>
> [ADFS 구성 방법 및 이중화 (3)](https://practice.hooniworld.io/entry/ADFS-%EA%B5%AC%EC%84%B1-%EB%B0%A9%EB%B2%95-%EB%B0%8F-%EC%9D%B4%EC%A4%91%ED%99%94-3)
>
> [RHEL VM을 Azure AD Domain Services에 조인](https://docs.microsoft.com/ko-kr/azure/active-directory-domain-services/join-rhel-linux-vm)
>
> [Linux 서버 Windows AD 조인](https://usheep91.tistory.com/m/43)
>
> [Add Linux to Windows Domain using realm (CentOS/RHEL 7/8)](https://www.golinuxcloud.com/add-linux-to-windows-ad-domain-realm/)
>
> [AD 구축과 DC 복제하기](https://m.blog.naver.com/PostView.nhn?blogId=jundory8&logNo=80192915901&proxyReferer=https:%2F%2Fwww.google.com%2F)
>
> [기존 AD에 복제된 DC 구성하기](https://knocxx.tistory.com/48)
>
> [Kerberos 인증 구성](https://techdocs.broadcom.com/kr/ko/ca-enterprise-software/layer7-identity-and-access-management/single-sign-on/12-8/464989729/464989730/464989742/464989755.html)
>
> [ktpass | Microsoft Docs](https://docs.microsoft.com/ko-kr/windows-server/administration/windows-commands/ktpass)

