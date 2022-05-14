# Install

## on VM

```sh
# install packages
sudo yum install -y wget unzip java-1.8.0-openjdk

# download binary
wget https://github.com/keycloak/keycloak/releases/download/12.0.4/keycloak-12.0.4.zip
unzip keycloak-12.0.4.zip

# create admin
./keycloak-12.0.4/bin/add-user-keycloak.sh -u admin

# run keycloak
nohup ./keycloak-12.0.4/bin/standalone.sh -b 0.0.0.0 > server.log 2>&1 &
```

### with External PostgreSQL

```sh
# EC2에서 DB 접속 여부부터 확인. EC2의 Outbound와 DB의 Inbound 방화벽 모두 확인하자
psql --host=ap-dev-keycloak-aurpg-01-instance-1.abcd1234.ap-northeast-2.rds.amazonaws.com --port=5432 --username=postgres --password --dbname=postgres

# JDBC driver 설치
mkdir -p keycloak-12.0.4/modules/system/layers/keycloak/org/postgresql/main/
curl -L https://jdbc.postgresql.org/download/postgresql-42.2.20.jar -o keycloak-12.0.4/modules/system/layers/keycloak/org/postgresql/main/postgresql-42.2.20.jar
```



```xml
vi keycloak-12.0.4/modules/system/layers/keycloak/org/postgresql/main/module.xml

<?xml version="1.0" ?>
<module xmlns="urn:jboss:module:1.3" name="org.postgresql">
    <resources>
        <resource-root path="postgresql-42.2.20.jar"/>
    </resources>
    <dependencies>
        <module name="javax.api"/>
        <module name="javax.transaction.api"/>
    </dependencies>
</module>
```



```xml
vi keycloak-12.0.4/standalone/configuration/standalone.xml

<subsystem xmlns="urn:jboss:domain:datasources:6.0">
    <datasources>
        <datasource jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" use-java-context="true">
            <!-- database 접속 정보 추가 : jdbc:postgresql://endpoint:port/database -->
            <connection-url>jdbc:postgresql://pg1.cluster-abcd1234.ap-northeast-2.rds.amazonaws.com:5432/postgres</connection-url>
            <driver>postgresql</driver>
            <pool>
                <max-pool-size>20</max-pool-size>
            </pool>
            <security>
                <user-name>pgadmin</user-name>
                <password>$PW</password>
            </security>
            <!-- /database 접속 정보 추가 -->
        </datasource>
        <datasource jndi-name="java:jboss/datasources/KeycloakDS" pool-name="KeycloakDS" enabled="true" use-java-context="true">
            <!-- database 접속 정보 추가 : jdbc:postgresql://endpoint:port/database -->
            <connection-url>jdbc:postgresql://pg1.cluster-abcd1234.ap-northeast-2.rds.amazonaws.com:5432/postgres</connection-url>
            <driver>postgresql</driver>
            <pool>
                <max-pool-size>20</max-pool-size>
            </pool>
            <security>
                <user-name>postgres</user-name>
                <password>$PW</password>
            </security>
            <!-- /database 접속 정보 추가 -->
        </datasource>
        <drivers>
            <!-- driver 추가 -->
            <driver name="postgresql" module="org.postgresql">
                <xa-datasource-class>org.postgresql.xa.PGXADataSource</xa-datasource-class>
            </driver>
            <!-- /driver 추가 -->
            <!-- h2 안쓸거면 주석 처리하자 
            <driver name="h2" module="com.h2database.h2">
                <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class>
            </driver>
    		-->
        </drivers>
    </datasources>
</subsystem>

<spi name="connectionsJpa">
    <provider name="default" enabled="true">
        <properties>
            <property name="dataSource" value="java:jboss/datasources/KeycloakDS"/>
            <property name="initializeEmpty" value="true"/>
            <property name="migrationStrategy" value="update"/>
            <property name="migrationExport" value="${jboss.home.dir}/keycloak-database-update.sql"/>
            <!-- schema 추가 (default: public) -->
            <property name="schema" value="public"/>
            <!-- /schema 추가 -->
        </properties>
    </provider>
</spi>
```

- Keycloak container를 사용해서 초기화한 DB를 나중에 VM 버전으로 붙으면 몇몇 테이블이 없다고 나온다. 이땐 그냥 DB 새로 만들어서 VM에 처음 붙이면 초기화 잘 된다. 



### Clustering

- https://chenyangliu.wordpress.com/2020/11/29/keycloak-ha-mode-with-jdbc_ping/ 참조

```xml
vi keycloak-12.0.4/standalone/configuration/standalone-ha.xml

<subsystem xmlns="urn:jboss:domain:infinispan:11.0">
    <cache-container name="keycloak" module="org.keycloak.keycloak-model-infinispan">
		<!-- 세션 클러스터링이 필요한 서버 대수. owners를 1 > 2로 변경 -->
        <distributed-cache name="sessions" owners="2"/>
    </cache-container>
</subsystem>

<subsystem xmlns="urn:jboss:domain:jgroups:8.0">
    <channels default="ee">
        <!-- udp > tcp 변경 -->
        <channel name="ee" stack="tcp" cluster="ejb"/>
    </channels>
    <stacks>
        ...
        <stack name="tcp">
            <!-- 프로토콜 순서는 중요하다. 주로 사용할 프로토콜을 위로 올리자 -->
            <!-- 1번 서버는 1번 서버 IP, 2번 서버는 2번 서버 IP 기재 -->
            <transport type="TCP" socket-binding="jgroups-tcp">
                <property name="external_addr">10.70.20.134</property>
            </transport>
            <protocol type="JDBC_PING">
                <property name="datasource_jndi_name">java:jboss/datasources/KeycloakDS</property>
                <property name="initialize_sql">
                    CREATE TABLE IF NOT EXISTS JGROUPSPING (own_addr varchar(200) NOT NULL, cluster_name varchar(200) NOT NULL, ping_data BYTEA, constraint PK_JGROUPSPING PRIMARY KEY (own_addr, cluster_name));
                </property>
            </protocol>
            <!-- MPING(multicast over TCP)은 AWS VPC에서 지원되지 않으므로 주석 처리
                <socket-protocol type="MPING" socket-binding="jgroups-mping"/>
            -->
            <protocol type="MERGE3"/>
            <socket-protocol type="FD_SOCK" socket-binding="jgroups-tcp-fd"/>
            <protocol type="FD_ALL"/>
            <protocol type="VERIFY_SUSPECT"/>
            <protocol type="pbcast.NAKACK2"/>
            <protocol type="UNICAST3"/>
            <protocol type="pbcast.STABLE"/>
            <protocol type="pbcast.GMS"/>
            <protocol type="MFC"/>
            <protocol type="FRAG3"/>
        </stack>
    </stacks>
</subsystem>

<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
    <!-- interface private > public 변경 -->
    <socket-binding name="jgroups-tcp" interface="public" port="7600"/>
    <socket-binding name="jgroups-tcp-fd" interface="public" port="57600"/>
</socket-binding-group>
```



```sh
# startup.sh
export JAVA_OPTS=" $JAVA_OPTS -server -Xms1024m -Xmx1024m -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true"
export JAVA_OPTS=" $JAVA_OPTS -Djboss.default.jgroups.stack=tcp"
export JAVA_OPTS=" $JAVA_OPTS -Djboss.as.management.blocking.timeout=3600"
nohup ./keycloak-12.0.4/bin/standalone.sh -b 0.0.0.0 --server-config=standalone-ha.xml > server.log 2>&1 &
```



## on Docker

### Install docker, docker-compose

```yaml
# install docker on Amazon Linux 2 EC2
sudo yum -y upgrade
sudo yum -y install docker
sudo service docker start
sudo usermod -aG docker ec2-user

# logout and login again
docker ps -a 

# install docker-compose on Amazon Linux 2 EC2
sudo curl -L https://github.com/docker/compose/releases/download/1.25.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose -v

# run keycloak
docker-compose up
```

### docker-compose.yml

- Keycloak을 그냥 Standalone으로 띄우면 default DB로 H2를 사용한다. 성능과 관리 측면에서 좋지 않으니 PostgreSQL로 대체하자. 

#### with Internal PostgreSQL

```yaml
version: '3'

volumes:
  postgres_data:
      driver: local

services:
  postgres:
      image: postgres
      volumes:
        - ./postgres_data:/var/lib/postgresql/data
      environment:
        POSTGRES_DB: keycloak
        POSTGRES_USER: keycloak
        POSTGRES_PASSWORD: password
      ports:
        - 5432:5432
  keycloak:
      restart: always
      image: jboss/keycloak
      environment:
        DB_VENDOR: POSTGRES
        DB_ADDR: postgres
        DB_DATABASE: keycloak
        DB_USER: keycloak
        DB_SCHEMA: public
        DB_PASSWORD: password
        KEYCLOAK_USER: admin
        KEYCLOAK_PASSWORD: Pa55w0rd
        # Uncomment the line below if you want to specify JDBC parameters. The parameter below is just an example, and it shouldn't be used in production without knowledge. It is highly recommended that you read the PostgreSQL JDBC driver documentation in order to use it.
        #JDBC_PARAMS: "ssl=true"
      ports:
        - 8080:8080
      depends_on:
        - postgres
```

#### with External PostgreSQL

```yaml
version: '3'

services:
  keycloak:
    restart: always
    image: jboss/keycloak
    environment:
      JVM_OPTS: "-Xmx2G -Xms2G"
      DB_VENDOR: POSTGRES
      DB_ADDR: pg1.cluster-abcd1234.ap-northeast-2.rds.amazonaws.com
      DB_DATABASE: postgres
      DB_USER: pgadmin
      DB_SCHEMA: public
      DB_PASSWORD: $PW
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: $PW
    ports:
      - 8080:8080
```

#### Clustering

```yaml
# Keycloak Container를 한 VM 내에 2대 이상 띄우면 알아서 DNS Ping 때려서 clustering 됨
services:
  keycloak:
    restart: always
    image: jboss/keycloak
    environment:
      JVM_OPTS: "-Xmx1G -Xms1G"
      DB_VENDOR: POSTGRES
      DB_ADDR: pg1.cluster-abcd1234.ap-northeast-2.rds.amazonaws.com
      DB_DATABASE: postgres
      DB_USER: pgadmin
      DB_SCHEMA: public
      DB_PASSWORD: $PW
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: $PW
    ports:
      - 8080:8080
```

- 위 Clustering 방식에는 아래와 같은 문제가 있음
    - VM 하나에 Container 이중화 해봤자 VM 죽으면 다같이 죽음
    - Docker Compose에서는 Memory 설정이 안됨. Container 기본 메모리 상한 512m은 너무 작음
- VM 2개에서 각기 docker-compose로 띄우고 clustering 해보자
- 혹은 아예 ECS로 배포하고 서로가 DNS로 Service Discovery할 수 있게 해보자



##### on diffrent VMs

- 1 VM에 1 Container를 띄운다고 가정한다. 
- 우선 VM에서의 Clustering 설정과 동일하게 standalone-ha.xml 파일을 만들고 standalone-ha1.xml로 저장해둔다. 
- network_mode는 host로 설정하고 아래와 같이 띄운다. 이후 `cp /opt/jboss/keycloak/standalone/configuration/standalone-ha1.xml /opt/jboss/keycloak/standalone/configuration/standalone-ha.xml` 로 설정파일을 변경해주자. 
- 설정파일을 변경한 container를 image로 찍자. `docker commit abc123 cg/keycloak`
- 그 다음 container를 종료하고 아래 docker-compose.yml 파일에서 image를 cg/keycloak으로 변경해서 띄워주면 된다.  그냥 설정파일을 바로 volume으로 넣어주면 좋겠지만 무슨 이유인지 모르게 에러가 난다. 이건 나중에 보자. 

```yaml
services:
  keycloak:
    restart: always
    network_mode: host
    image: jboss/keycloak
    environment:
      JVM_OPTS: "-Xmx1G -Xms1G"
      DB_VENDOR: POSTGRES
      DB_ADDR: pg1.cluster-abcd1234.ap-northeast-2.rds.amazonaws.com
      DB_DATABASE: postgres
      DB_USER: pgadmin
      DB_SCHEMA: public
      DB_PASSWORD: $PW
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: $PW
      JGROUPS_DISCOVERY_PROTOCOL: JDBC_PING
      JGROUPS_DISCOVERY_EXTERNAL_IP: 172.31.34.235
      JGROUPS_DISCOVERY_PROPERTIES: datasource_jndi_name=java:jboss/datasources/KeycloakDS,info_writer_sleep_time=500,initialize_sql="CREATE TABLE IF NOT EXISTS JGROUPSPING ( own_addr varchar(200) NOT NULL, cluster_name varchar(200) NOT NULL, created timestamp default current_timestamp, ping_data BYTEA, constraint PK_JGROUPSPING PRIMARY KEY (own_addr, cluster_name))"
    volumes:
      - ./standalone-ha1.xml:/opt/jboss/keycloak/standalone/configuration/standalone-ha1.xml
    ports:
      - 8080:8080
      - 7600:7600
      - 57600:57600
```



##### ECS

- 작업 정의시 AWS Console > ECS > 작업 정의에서 네트워크 모드를 awsvpc로 바꿔준다. 
- 나머지 컨테이너 정의는 아래 docker-compose.yml 내용과 동일하게 입력한다. 
- 서비스 생성시 ALB Target Group의 경로 패턴: /*, 상태 확인 경로: /auth/ 로 한다.
- 서비스 검색 통합 활성화 옵션을 켜주고 네임스페이스: local, 서비스 검색 이름: keycloak로 하면 서비스 검색 엔드포인트는 keycloak.local 이 된다. 

```yaml
version: '3'

services:
  keycloak:
      image: jboss/keycloak
      environment:
        #JVM_OPTS: "-Xmx1G -Xms1G"
        DB_VENDOR: POSTGRES
        DB_ADDR: database-1.cluster-abcd1234.ap-northeast-2.rds.amazonaws.com
        DB_DATABASE: postgres
        DB_USER: postgres
        DB_SCHEMA: public
        DB_PASSWORD: $PW
        KEYCLOAK_USER: admin
        KEYCLOAK_PASSWORD: $PW
        PROXY_ADDRESS_FORWARDING: "true"
        JGROUPS_DISCOVERY_PROTOCOL: dns.DNS_PING
        JGROUPS_DISCOVERY_PROPERTIES: 'dns_query=keycloak.local,dns_record_type=A'
        JGROUPS_TRANSPORT_STACK: tcp
          #JGROUPS_DISCOVERY_PROTOCOL: JDBC_PING
          #JGROUPS_DISCOVERY_PROPERTIES: datasource_jndi_name=java:jboss/datasources/KeycloakDS,info_writer_sleep_time=500,initialize_sql="CREATE TABLE IF NOT EXISTS JGROUPSPING ( own_addr varchar(200) NOT NULL, cluster_name varchar(200) NOT NULL, created timestamp default current_timestamp, ping_data BYTEA, constraint PK_JGROUPSPING PRIMARY KEY (own_addr, cluster_name))"
      ports:
        - 8080:8080
        - 7600:7600
        - 57600:57600
        - 55200:55200
```



# Config

## disable HTTPS required to test

```sh
# config postgres
docker exec -it keycloak_postgres_1 /bin/bash

adduser keycloak
su keycloak
psql

update realm set ssl_required = 'NONE';

# restart keycloak
docker-compose down; docker-compose up
```

## LDAP

Keycloak > Configure > User Federation > Add provider > ldap

- Edit Mode : UNSYNCED (LDAP으로부터 import만 하고 sync back 하지 않는다. Failed authentication: org.keycloak.storage.ReadOnlyException: Federated storage is not writable 발생 방지)
- Vendor : Active Directory
- Username LDAP attribute : cn
- RDN LDAP attribute : cn
- UUID LDAP attribute : objectGUID
- User Object Classes : person, organizationalPerson, user
- Connection URL : ldap://ec2-3-35-4-158.ap-northeast-2.compute.amazonaws.com:389
- Users DN : OU=test-ou,DC=test,DC=com
- Bind Type : simple
- Bind DN : CN=tester,OU=test-ou,DC=test,DC=com
- Bind Credential : ${password}
- Periodic Full Sync : OFF
- Periodic Changed Users Sync : ON
- Save
- Syncronize all users (Sync가 ON인 경우)
    - Sync Users의 경우 몇 가지 옵션이 있을 수 있다. 기본적으로 Keycloak에서 로그인시 인증 요청은 그대로 LDAP에 넘어간다. LDAP에서 인증되면 그 User 정보를 Keycloak이 받아 Local DB에 저장한다. 다만 여기서 Password는 저장하지 않으므로 Keycloak Local DB에 저장된 User라 하더라도 로그인시 인증 요청은 계속해서 LDAP으로 넘어간다. Password 인증을 제외한 User 정보 조회는 Keycloak Local DB에서 바로 이루어진다. 이는 Keycloak에서 Import Users 옵션이 켜져있을 때의 동작이다. 
    - 해당 옵션을 끄면 User의 모든 정보를 매번 AD로부터 가져오고 Keycloak Local DB에 저장해두지 않으므로 AD에 부하가 갈 수 있다. AD 부하를 줄이려면 Import Users 옵션을 사용하는게 좋은데 다만 이 경우 AD에서 User 정보가 바뀌었을때 동기화가 되지 않는 이슈가 있을 수 있다. 그래서 이 때는 Periodic Sync 옵션을 켜주는게 좋은데, Full Sync는 최초 1회 실행해주고 이후 Periodic Changed Users Sync만 켜주면 된다. 이렇게 하면 AD에 새로운 User가 Insert/Update/Delete 되었을때 Keycloak에서 Sync 잘 되는 것 확인함. 



[Active Directory Attributes List](https://docs.classic.secureauth.com/display/KBA/Active+Directory+Attributes+List)를 참조하여 아래 Mappers를 설정하면 된다. 



Keycloak > Configure > User Federation > ldap > Mappers > username

- User Model Attribute : cn
- LDAP Attribute : cn



Keycloak > Configure > User Federation > ldap > Mappers > Create

- Name : mobile
- Mapper Type : user-attribute-ldap-mapper
- User Model Attribute : mobile
- LDAP Attribute : mobile



Keycloak > Configure > User Federation > ldap > Mappers > Create

- Name : department
- Mapper Type : user-attribute-ldap-mapper
- User Model Attribute : department
- LDAP Attribute : department



Keycloak > Configure > User Federation > ldap > Mappers > Create

- Name : group
- Mapper Type : group-ldap-mapper
- LDAP groups DN : ou=test-ou,dc=a,dc=com
- Group name ldap attribute : cn or displayName (unique 해야 함)
- Group object classes : group or groupOfUniqueNames (ldap에서 group생성할때 지정한 object class. AD라면 group)
- Preserve Group Inheritance : OFF
- Ignore Missing Groups : OFF
- Membership LDAP Attribute : member or uniquemember (보통 AD는 member)
- Membership Attribute Type : DN
- Membership User LDAP Attribute : cn
- Mode : READ ONLY
- User Groups Retrieve Strategy : LOAD_GROUPS_BY_MEMBER_ATTRIBUTE_RECURSIVELY (AD인 경우 한 User가 여러 Group에 속해있을때 모든 Group 가져오게 함)
- Member-Of LDAP Attribute : memberOf
- Mapped Group Attributes : name, displayName 등 가져오고 싶은 group의 attributes 입력 (구분자 ',')
- Drop non-existing groups during sync : OFF (만약 잘못된 정보로 group sync가 되서 group을 지우고 싶은 경우 잠깐 ON으로 하고 sync하면 user가 하나도 없는 group들이 전부 삭제된다)
- Groups Path : /



만약 User Sync 시 Email Duplicate이 발생한다면 아래와 같이 설정한다. 

Keycloak > Realm Settings > Login

- Login with email : OFF
- Duplicate emails : ON



## Kerberos

Keycloak > Configure > User Federation > ldap > Kerberos Integration

- 우선 Windows AD DC 서버에 Administrator로 로그인하여 아래와 같이 keytab 파일 생성

```powershell
ktpass /princ HTTP/$ID.test.com@test.com /mapuser $ID /pass $PW /out kb.keytab /crypto all /ptype KRB5_NT_PRINCIPAL /mapop set
ktpass /princ HTTP/$ID.iam-ko.test.com@iam-ko.test.com /mapuser $ID /pass $PW /out $ID.keytab /crypto all /ptype KRB5_NT_PRINCIPAL /mapop set
```

- Allow Kerberos authentication : ON
- Kerberos Realm : test.com
- Server Principal : HTTP/$ID.test.com@test.com
- KeyTab : /home/apuser/kb.keytab



## HTTPS required

Keycloak > Realm Settings > Login > Require SSL

혹은 아래와 같이 설정

I was running the key cloak inside a docker container, The keycloak command line tool was avaialble inside the keycloak container.

```sh
docker exec -it {contaierID} bash
cd keycloak/bin
./kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin
./kcadm.sh update realms/master -s sslRequired=EXTERNAL
```


If the admin user is not created, then the user can be created via this command.

./add-user-keycloak.sh --server http://ip_address_of_the_server:8080/admin --realm master --user admin --password adminPassword

Update: For the newer versions the file in available in the following path: /opt/jboss/keycloak/bin



위와 같이 모든 설정을 완료한 뒤에 만약 Keycloak을 reverse proxy로 ssl offloading 하지 않고 standalone으로만 띄운다면 8080 포트 대신 8443 포트로 서비스한다. 

reverse proxy ssl offloading을 사용하는 경우에는 아래와 같이 [추가 설정](https://www.keycloak.org/docs/latest/server_installation/index.html#_setting-up-a-load-balancer-or-proxy)을 해준다. 

```xml
vi keycloak/standalone/configuration

<subsystem xmlns="urn:jboss:domain:undertow:11.0" default-server="default-server" default-virtual-host="default-host" default-servlet-container="default" default-security-domain="other" statistics-enabled="${wildfly.undertow.statistics-enabled:${wildfly.statistics-enabled:false}}">
    <buffer-cache name="default"/>
    <server name="default-server">
        ...
        <!-- proxy-address-forwarding="true" 추가 -->
        <!-- redirect-socket="https"를 redirect-socket="proxy-https"로 변경하고 proxy-address-forwarding="true" 추가 -->
        <http-listener name="default" socket-binding="http" redirect-socket="proxy-https" enable-http2="true" proxy-address-forwarding="true"/>
        ...
    </server>

<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
    ...
    <!-- 아래 설정 추가 -->
    <socket-binding name="proxy-https" port="443"/>
    ...
</socket-binding-group>
```



## SAML with Slack

Keycloak > Clients > Create

- Client ID : https://test-sandbox.enterprise.slack.com
- Client Protocol : saml
- Client SAML Endpoint : https://test-sandbox.enterprise.slack.com
- Sign Assertions : ON
- Client Signature Required : OFF
- Front Channel Logout : OFF (Keycloak에서 로그아웃시 Slack으로 Redirection 발생하는 것 방지)
- Valid Redirect URIs : https://test-sandbox.enterprise.slack.com*
- Assertion Consumer Service POST Binding URL : https://test-sandbox.enterprise.slack.com/sso/saml
- Logout Service POST Binding URL : https://test-sandbox.enterprise.slack.com/sso/saml/logout



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > mappers > Create

- Name : User.Email
- Mapper Type : User Attribute
- User Attribute : email
- SAML Attribute Name : User.Email
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > mappers > Create

- Name : User.Username
- Mapper Type : User Attribute
- User Attribute : username
- SAML Attribute Name : User.Username
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > mappers > Create

- Name : first_name
- Mapper Type : User Attribute
- User Attribute : firstName
- SAML Attribute Name : first_name
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > mappers > Create

- Name : last_name
- Mapper Type : User Attribute
- User Attribute : lastName
- SAML Attribute Name : last_name
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > Client Scopes

- Assigned Default Client Scopes : Remove role_list



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > Scope

- Full Scope Allowed : OFF



https://test-sandbox.enterprise.slack.com/manage/security > SSO Settings

- SAML 2.0 Endpoint URL : Keycloak > Clients > slack > Installation > bindingUrl
    - https://pubsso.apdigit.com/auth/realms/master/protocol/saml
- Identity Provider Issuer URL : Keycloak > Realm Settings > SAML 2.0 Identity Provider Metadata > entityID
    - https://pubsso.apdigit.com/auth/realms/master
- Service Provider Issuer URL : 
    - https://test-sandbox.enterprise.slack.com
- Public (X.509) Certificate : Keycloak > Realm Settings > SAML 2.0 Identity Provider Metadata > ds:X509Certificate
- AuthnContextClassRef : Don't send this value.
- Sign the AuthnRequest : OFF
- Sign the Response : OFF
- Sign the Assertion : ON



## SAML with Zendesk

- [Zendesk SAML Guide](https://support.zendesk.com/hc/ko/articles/203663676) 



Keycloak > Clients > Create

- Client ID : https://subdomain.zendesk.com
- Client Protocol : saml
- Client SAML Endpoint : https://subdomain.zendesk.com/access/saml/
- Sign Assertions : ON
- Client Signature Required : OFF
- Front Channel Logout : OFF (Keycloak에서 로그아웃시 Zendesk로 Redirection 발생하는 것 방지)
- Valid Redirect URIs : 
    - https://subdomain.zendesk.com*
    - https://subdomain.zendesk.com/access/saml*
- Assertion Consumer Service POST Binding URL : https://subdomain.zendesk.com/access/saml/
- Assertion Consumer Service Redirect Binding URL : https://subdomain.zendesk.com/
- Logout Service POST Binding URL : https://subdomain.zendesk.com/access/saml/
- Logout Service Redirect Binding URL : https://subdomain.zendesk.com/



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : displayName
- Mapper Type : User Attribute
- User Attribute : displayName
- SAML Attribute Name : displayName
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
- Mapper Type : User Attribute
- User Attribute : firstName
- SAML Attribute Name : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
- Mapper Type : User Attribute
- User Attribute : lastName
- SAML Attribute Name : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
- SAML Attribute NameFormat : Unspecified



만약 givenname, surname이 영문이고 displayName이 한글인데, Zendesk에 한글 이름을 넘기고 싶다면 surname을 빼고 아래와 같이 givenname만 displayName으로 설정한다. 

Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
- Mapper Type : User Attribute
- User Attribute : displayName
- SAML Attribute Name : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : phone
- Mapper Type : User Attribute
- User Attribute : mobile
- SAML Attribute Name : phone
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : email
- Mapper Type : User Attribute
- User Attribute : email
- SAML Attribute Name : email
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : user_field_department
- Mapper Type : User Attribute
- User Attribute : department
- SAML Attribute Name : user_field_department
- SAML Attribute NameFormat : Unspecified



~~Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create~~

- ~~Name : tag_aponid~~
- ~~Mapper Type : Javascript Mapper~~
- ~~Script :~~ 

```javascript
var username = user.getUsername();

var StringType = Java.type("java.lang.String");
var output = new StringType();
output = 'aponid:' + username;
output;
```

- ~~SAML Attribute Name : tag_aponid~~
- ~~SAML Attribute NameFormat : Unspecified~~



~~Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create~~

- ~~Name : tag_dept~~
- ~~Mapper Type : Javascript Mapper~~
- ~~Script :~~ 

```javascript
var dept = user.getFirstAttribute('department');
// remove special characters
dept = dept.replace(/[\{\}\[\]\/?.,;:|\)*~`!^\-_+<>@\#$%&\\\=\(\'\"].*/gi, "");

var StringType = Java.type("java.lang.String");
var output = new StringType();
output = 'dept:' + dept;
output;
```

- ~~SAML Attribute Name : tag_dept~~
- ~~SAML Attribute NameFormat : Unspecified~~



Keycloak > Clients > https://subdomain.zendesk.com > mappers > Create

- Name : tags
- Mapper Type : Javascript Mapper
- Script : 

```javascript
var username = user.getUsername();
// remove whitespace
username = username.replace(/ /gi, "_");

var dept = user.getFirstAttribute('department');
// remove special characters
dept = dept.replace(/[\{\}\[\]\/?.,;:|\)*~`!^\-_+<>@\#$%&\\\=\(\'\"].*/gi, "");
// remove whitespace
dept = dept.replace(/ /gi, "_");


var StringType = Java.type("java.lang.String");
var output = new StringType();
output = 'aponid:' + username + ' dept:' + dept;
output;
```

- SAML Attribute Name : tags
- SAML Attribute NameFormat : Unspecified



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > Client Scopes

- Assigned Default Client Scopes : Remove role_list



Keycloak > Clients > https://test-sandbox.enterprise.slack.com > Scope

- Full Scope Allowed : OFF



## SAML with Jenkins

Keycloak > Clients > Create

- Client ID: saml-jenkins-dev
- Client Protocol: saml
- Root URL: https://jenkins-host/
- Client Signature Required: OFF
- Front Channel Logout: OFF
- Valid Redirect URIs
    -  https://jenkins-host
    -  https://jenkins-host/
    -  https://jenkins-host/*
- Authentication Flow Overrides > Browser Flow: Browser - SSO with IdP SAML (옵션. 다른 IdP로 SSO하고 싶다면 선택)



Keycloak > Clients > saml-jenkins-dev > mappers > Create

- Name: email
- Mapper Type: User Attribute
- User Attribute: email
- SAML Attribute Name: email
- SAML Attribute NameFormat: Unspecified



Jenkins > Jenkins 관리 > 플러그인 관리 > 설치 가능 > SAML 검색 후 설치

- 만약 Internet으로의 Outbound가 막혀있는 경우, 고급 > 플러그인 올리기 > 아래 Dependencies hpi 파일 수동 설치
    - [Display URL API](https://plugins.jenkins.io/display-url-api/) 
    - [Mailer](https://plugins.jenkins.io/mailer/) 
    - [bouncycastle API](https://plugins.jenkins.io/bouncycastle-api/) 



Jenkins > Jenkins 관리 > 시스템 설정 > Jenkins Location > Jenkins URL

-  IP:PORT가 아닌 위 Keycloak에 설정한 https://jenkins-host 로 통일한다.



Jenkins > Jenkins 관리 > Configure Global Security > Security Realm > SAML 2.0

-  IdP Metadata: https://keycloak-host/auth/realms/master/protocol/saml/descriptor 내용 그대로 붙여넣기
-  IdP Metadata URL: https://keycloak-host/auth/realms/master/protocol/saml/descriptor
-  Email Attribute: email
-  Advanced Configuration > Force Authentication > SP Entity ID: saml-jenkins-dev



## Theme

- keycloak-12.0.4/themes/base/login/messages/messages_en.properties
    - loginAccountTitle=Login
    - loginTitle=Log in to Test SSO
    - loginTitleHtml=Test SSO
    - username=ID
- keycloak-12.0.4/themes/keycloak/login/resources/img/keycloak-bg.png
- keycloak-12.0.4/themes/keycloak/common/resources/img/favicon.ico
- keycloak-12.0.4/themes/keycloak/common/resources/node_modules/patternfly/dist/img/favicon.ico
- keycloak-12.0.4/themes/keycloak/login/resources/css/login.css
    - `div.kc-logo-text {
            //background-image: url(../img/keycloak-logo-text.png);
        }` 
    - `div.kc-logo-text span {
            //display: none;
        }` 
    - /home/apuser/keycloak-12.0.4/themes/keycloak/login/template.ftl
        - `<title>Log in into Test SSO</title>` 
        - `<div id="kc-header-wrapper-text" style="color: black; font-size: 29px; text-transform: uppercase; letter-spacing: 3px; line-height: 1.2em; padding: 62px 10px 20px; white-space: normal;">Test SSO</div>` 



## LDAPS Certificate

```sh
# Convert cer to jks
keytool -importcert -file test.cer -keystore test.jks
```



```xml
vi keycloak-12.0.4/standalone/configuration/standalone.xml

<spi name="truststore">
    <provider name="file" enabled="true">
        <properties>
            <property name="file" value="/keycloak-12.0.4/test.jks"/>
            <property name="password" value="jsk 생성시 넣은 password"/> 
            <property name="hostname-verification-policy" value="WILDCARD"/>
            <property name="disabled" value="false"/>
        </properties>
    </provider>
</spi>
```



만약 java에서 no subject alternative dns name matching... 과 같은 에러가 발생할때는 아래와 같은 옵션을 넣어준다. [참조](https://www.ibm.com/support/pages/how-resolve-ldap-error-javaxnetsslsslhandshakeexception-javasecuritycertcertificateexception-no-subject-alternative-dns-name-matching-ip-address-found) 

```sh
export JAVA_OPTS=" $JAVA_OPTS -Dcom.sun.jndi.ldap.object.disableEndpointIdentification=true"
```



## Add Identity Providers

- keycloak1 (IDP) - keycloak2 (SP)
- keycloak2 (IDP) - keycloak3 (SP)

인 환경을 가정한다. 사용자는 keycloak3에 로그인하려 하는데, keycloak3의 IDP는 keycloak2이며, 다시 keycloak2의 IDP는 keycloak1으로 chaining된 상태다.  

keycloak1은 SAML Response로 username만 준다. keycloak2는 그 username을 받아 email, firstName, lastName을 조합하여 keycloak3에 넘겨준다. 

만약 docker로 localhost상에서 port만 다르게 하여 keycloak 3대를 띄울 경우 서로 host가 같아서 인증 정보가 꼬이니 hosts에 127.0.0.1을 서로 다른 도메인으로 등록하여 진행하자. 



우선 keycloak 1/2/3에 각각 IDP/SP 설정을 하자.

- IDP > Configure > Realm Settings > Endpoints > SAML 2.0 Identity Provider Metadata 로 SAML IDP XML을 export 한다. 
- SP > Configure > Identity Providers > SAML v2.0 > Import External IDP Config > Import from file > Select SAML IDP XML
- SP > Configure > Identity Providers > Select IDP > Endpoints > SAML 2.0 Service Provider Metadata 로 SAML SP XML을 export 한다. 
- IDP > Configure > Clients > Create > Import > Select file > Select SAML SP XML 
- IDP > Configure > Clients > Select Client > Mappers > Create > Mapper Type: User Attribute, User Attribute: username
    - 필요한 경우 email, firstName, lastName 등 나머지 Attribute 설정
- SP > Configure > Identity Providers > Select IDP > Mappers > Create > Mapper Type: Attribute Importer, Attribute Name: username
    - 필요한 경우 email, firstName, lastName 등 나머지 Attribute 설정

그럼 IDP - SP 간 SAML 연동은 완료된 것이다. 이제 SP 로그인창으로 가면 화면 하단에 "Or sign in with {IDP명}" 버튼이 추가된다. 



그러나 여기서 아래와 같은 조건으로 연동된다고 가정하면 몇 가지 이슈가 생긴다. 

- keycloak1에서 로그인시 keycloak1 > keycloak2 로는 username만 전달되는데 해당 정보만으로 keycloak2에서 로그인이 잘 되는가?
    - 원래 keycloak은 IDP로부터 username, email, firstName, lastName을 필수로 받아야 하는 것으로 상정되어 있다. 만약 빈 값이 있으면 해당 값 입력창이 뜬다. 
    - 또한 keycloak1에서 username이 test01로 왔을때, 만약 keycloak2에 username이 test01인 계정이 이미 존재하면 연결할거냐고 물어보는 확인창이 뜬다. 
    - 위 2개를 모두 없애기 위해서는 keycloak2의 IDP 설정에서 First Login Flow 부분을 바꿔야 한다. 해당 옵션은 처음 로그인이 성공한 직후에 이어서 진행할 프로세스를 의미한다. 
        - keycloak2 > Configure > Authentication > New > Alias: Automatically Set Existing User
            - Add execution >  Create User If Unique >  ALTERNATIVE
            - Add execution >  Automatically Set Existing User >  ALTERNATIVE
        - keycloak2 > Configure > Identity Providers > Select IDP > First Login Flow > Select Automatically Set Existing User
    - 위와 같이 설정하면 keycloak2는 keycloak1에서 인증 완료 후 받은 username으로 keycloak2 내에서 해당 계정을 찾아 로그인시킨다.
- keycloak1 > keycloak2 로는 username만 전달되지만, keycloak2 > keycloak3 로는 username, email, firstName, lastName이 전달되어야 하는 이슈. 위와 같이 keycloak1 - keycloak2 간 이슈가 해결되면 keycloak1 로그인 시 해당 username을 가진 keycloak2 User에 자동으로 로그인되고, keycloak2 User의 username, email, firstName, lastName이 keycloak3로 잘 전달되며 로그인 완료됨.
- keycloak2에서 User Federation으로 AD를 추가했으나 아직 로그인하지 않은 계정이라면, keycloak1 통해서 로그인시 keycloak2에 자동으로 import 될 때 AD 연동 상태로 로그인 되는가? 이 이슈를 해결하기 위해서는 keycloak2에 존재하지 않는 User라면 1회 로그인하게 해야 할거라 생각했다. 그러나 실제로 해보니 keycloak1에서 로그인한 username을 keycloak2가 받아서 User Federation AD로부터 계정 정보를 조회하여 User가 JIT Provisioning 하므로 아무 문제가 없었다.
- keycloak1이 로그인되어 있더라도 keycloak2에 접근해서 sign in with keycloak1 버튼을 클릭해야만 keycloak1 로그인 정보를 keycloak2가 받아 로그인 처리한다. keycloak3 > keycloak2 로그인 시도시 자동으로 kc_idp_hint=keycloak1-saml 을 줘서 자동으로 keycloak1 로그인 화면으로 이동시키려 했으나, 아래와 같이 IDP Redirector를 사용하는 깔끔한 해결책이 있다.
    - keycloak2 > Configure > Authentication > Select Browser > Copy > Name: Browser - IDP Redirector To Keycloak1-saml
    - keycloak2 > Configure > Authentication > Select Browser - IDP Redirector To Keycloak1-saml > Identity Provider Redirector > Actions > Config
        - Alias: keycloak1-saml
        - Default Identity Provider: keycloak1-saml
    - keycloak2 > Configure > Clients > Select keycloak3 > Authentication Flow Overrides > Browser Flow > Browser - IDP Redirector To Keycloak1-saml
    - 위와 같이 설정하면 이제 keycloak3에서 sign in with keycloak2 버튼 클릭하면 keycloak2로 갔다가 바로 keycloak1으로 Redirect 된다. 이때 keycloak1에 이미 로그인되어 있다면 당연히 keycloak1 > keycloak2 > keycloak3 로 SAML Response 타고 자동 로그인된다. SSO 완료.



그리고 chaining하는 프로토콜들이 꼭 모두 동일하게 SAML일 필요는 없다. 아래와 같이 mix 구성해도 문제없이 연동된다. 

- keycloak1 (IDP) ---(SAML)--- keycloak2 (SP)
- keycloak2 (IDP) ---(OIDC)--- keycloak3 (SP)

만약 docker compose로 OIDC 테스트할 경우에는 Server 간 호출 이슈가 있을 수 있다. 

OIDC 정보를 export/import할 때 host 상의 브라우저에서 진행하게 될테니 OIDC meta 정보들은 모두 host network 상의 host:port를 가지게 된다. 

OIDC의 흐름상 SP Server > IDP Server 호출하게 되는데, 이건 Server to Server 호출이므로 docker network 상에서 진행된다. 

그런데 호출하는 Server 정보는 host network의 host:port이므로 docker-compose.yml에 8081:8080과 같이 host와 container port를 다르게 기재했다면 호출이 안된다.

이 때는 port를 8080:8080 동일하게 맞춘 서버를 OIDC Server로 하면 host network와 docker network에서의 호출 host:port가 동일해서 진행이 수월하다. 

(물론 host의 /etc/hosts에 등록한 host와, docker-compose.yml의 service명도 맞춰준다)



# REST API

```sh
# Authorization
access_token=`curl -X POST https://localhost:8080/auth/realms/master/protocol/openid-connect/token \
-d "grant_type=password" \
-d "client_id=$client_id" \
-d "client_secret=$client_secret" \
-d "username=$username" \
-d "password=$password" \
-d "response_type=code" \
-d "response_mode=fragment" \
-d "scope=openid" | jq -r .access_token`

#echo $access_token

# Select Role
#curl -X GET -H "Authorization: Bearer $access_token" https://localhost:8080/auth/admin/realms/master/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43/roles | jq

# Insert Role
#curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer $access_token" https://localhost:8080/auth/admin/realms/master/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43/roles \
    #-d '{"name":"test-role-00"}' | jq

# Update Role
#curl -X PUT -H "Content-Type: application/json" -H "Authorization: Bearer $access_token" https://localhost:8080/auth/admin/realms/master/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43/roles/test-role-00 \
    #-d '{"name":"test-role-00","description":"test-role-00!"}' | jq

# Delete Role
#curl -X DELETE -H "Authorization: Bearer $access_token" https://localhost:8080/auth/admin/realms/master/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43/roles/test-role-00 | jq

# Select User
#curl -X GET -H "Authorization: Bearer $access_token" https://localhost:8080/auth/admin/realms/master/users \
    #-G -d "username=ap35014293" | jq

# Select Available User Role (to get role's availability, id, name)
curl -X GET -H "Authorization: Bearer $access_token" \
    https://localhost:8080/auth/admin/realms/master/users/8f81fd0e-5495-4c5f-8211-f62b5194c746/role-mappings/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43/available | jq

# Select User Role
#curl -X GET -H "Authorization: Bearer $access_token" \
    #https://localhost:8080/auth/admin/realms/master/users/9295f853-abbd-43fd-bf8c-bfdc03a81703/role-mappings/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43 | jq

# Insert User Role
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer $access_token" \
    https://localhost:8080/auth/admin/realms/master/users/8f81fd0e-5495-4c5f-8211-f62b5194c746/role-mappings/clients/e8dbbb54-4c4e-4f98-b950-b2fed0a40b43 \
    -d '[{"name":"test-role-01","id":"af247c1f-b016-4bf7-803d-d912a0bbd095"}]' | jq

# Update User Role

# Delete User Role

```



## API 권한

- Access Token을 얻을때의 Client와 User 모두 API에 대한 권한이 있어야 한다. 
- 예를 들어 Select Client API의 경우
    - Keycloak > Clients > cli-test > Scope > Client Roles > master-realm > Assign query-clients
    - Keycloak > Users > cli-test > Role Mappings > Client Roles > master-realm > Assign query-clients
- Select User API의 경우
    - Keycloak > Clients > cli-test > Scope > Client Roles > master-realm > Assign view-users
    - Keycloak > Users > cli-test > Role Mappings > Client Roles > master-realm > Assign view-users
- Create Client API의 경우
    - Keycloak > Clients > cli-test > Scope > Client Roles > master-realm > Assign manage-clients
    - Keycloak > Users > cli-test > Role Mappings > Client Roles > master-realm > Assign manage-clients



# Issues

## Email Case-Sensitive

- User Federation에 등록한 LDAP으로  User 정보를 가져올 때 기본 email 컬럼에 들어가는 값은 다 소문자로 들어간다. 그런데 실제 email 주소의 ID 부분은 Case-Sensitive일 수 있다. 그럴 때는 아래와 같이 Attribute를 하나 더 파서 email 값을 가져오면 대소문자가 정확하게 들어온다.
    - Keycloak > Configure > User Federation > ldap > Mappers > Create

        - Name : caseSensitiveEmail
        - Mapper Type : user-attribute-ldap-mapper
        - User Model Attribute : caseSensitiveEmail
        - LDAP Attribute : mail



# References

- [Keycloak](https://www.keycloak.org)
- [Keycloak Docs](https://www.keycloak.org/documentation.html)
- [MSA 인증 서비스 Keycloak 소개](https://subji.github.io/posts/2020/07/08/keycloak1)
- [AWS EC2에 docker-compose로 keycloak 설치](https://ratseno.tistory.com/95)
- [Keycloak(User Federation) - LDAP 연계](https://hs-note.tistory.com/23)
- [Setting up Slack SAML SSO with Keycloak](https://davidops.com/posts/setting-up-slack-saml-sso-with-keycloak/)
- [KeyCloak AWS SSO 연동](https://kyleyoon.tistory.com/3)
- [K8s에서 Keycloak, Jenkins, Spinnaker 연동](https://tech.osci.kr/2020/04/04/91699412/)
- [How to create a Script Mapper in Keycloak?](https://stackoverflow.com/questions/52518298/how-to-create-a-script-mapper-in-keycloak) 
- [Role based client log-in access restriction for users](https://stackoverflow.com/questions/57287497/keycloak-role-based-client-log-in-access-restriction-for-users) 

