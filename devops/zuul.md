ZUUL CICD TEST by 김희성



\#################



cat > create-sshkey.sh << 'EOF'

rm -rf ~/.ssh/id_rsa ~/.ssh/id_rsa.pub

read -p "Enter github email : " email

echo "Using email $email"

ssh-keygen -t rsa -b 4096 -C "$email"

ssh-add ~/.ssh/id_rsa

\#pub=`cat ~/.ssh/id_rsa.pub`

\#read -p "Enter github username: " githubuser

\#echo "Using username $githubuser"

\#read -s -p "Enter github password for user $githubuser: " githubpass

\#curl -u "$githubuser:$githubpass" -X POST -d "{\"title\":\"`hostname`\",\"key\":\"$pub\"}" https://api.github.com/user/keys

EOF





cat >> /etc/hosts << 'EOF'

192.168.59.20   gitlab.kekehoho.com       # 소스 형상관리 node : gitlab

192.168.59.21   gerrit.kekehoho.com       # 소스 형상관리 node : gerrit

192.168.59.21   zuul.kekehoho.com        # zuul node

192.168.59.21   testnode.kekehoho.com      # CI 할 node-01 : nodeset 으로 nodepool 에 설정

192.168.59.21   zuullognode.kekehoho.com     # logs 를 받을 node 

192.168.59.21   onpremisezuulroles.kekehoho.com # Base job 또는 기본으로 사용할 Ansible roles 파일이 저장되어 있는 저장소, gerrit/gitlab 에 만들면됨

192.168.59.30   targetnode-01.kekehoho.com    # CI 할 node-01 : nodeset 으로 job 에 설정 : CD는 어디다가 정의하나?

EOF





root@zuul:~ # cd zuul/doc/source/examples/; docker-compose down

docker rmi -f $(docker images | grep -E "examples_|zookeeper|zuul|mariadb|ubuntu|httpd|gerrit" | awk '{print $3}')

docker volume rm -f $(docker volume ls | grep examples_ | awk '{print $2}')

docker network rm $(docker network ls | grep zuul | awk '{print $2}')



\## docker volume rm -f $(docker volume ls | awk '{print $2}')





root@zuul:~# cd ~; rm -rf zuul

git config --global http.sslVerify false

git clone https://opendev.org/zuul/zuul

cd zuul/doc/source/examples/







root@zuul:~/zuul/doc/source/examples# cp -f docker-compose.yaml backup.docker-compose.yaml

sed -i "s/CANONICAL_WEB_URL=http:\/\/localhost:8080\//CANONICAL_WEB_URL=http:\/\/gerrit.kekehoho.com:8080\//g" docker-compose.yaml

sed -i "s/docker.io\/zuul\/zuul-executor/docker.io\/zuul\/zuul-executor:latest/g" docker-compose.yaml

sed -i "s/docker.io\/zuul\/zuul-scheduler/docker.io\/zuul\/zuul-scheduler:latest/g" docker-compose.yaml

sed -i "s/docker.io\/zuul\/zuul-web/docker.io\/zuul\/zuul-web:latest/g" docker-compose.yaml

sed -i "s/docker.io\/zuul\/nodepool-launcher/docker.io\/zuul\/nodepool-launcher:latest/g" docker-compose.yaml





\## docker-compose.yaml

ersion: '3'

services:

api:

  build: .

  ports:

​    \- "5003:5003"

  extra_hosts:

​    \- "your-host.name.com:162.242.195.82" #host and ip

​    \- "your-host--1.name.com your-host--2.name.com:50.31.209.229" #multiple hostnames with same ip





  image: docker.io/gerritcodereview/gerrit

  image: docker.io/zuul/zuul-scheduler:latest

  image: docker.io/zuul/zuul-web:latest

  image: docker.io/zuul/zuul-executor:latest

  image: docker.io/zuul/nodepool-launcher:latest







sed -i "/\ \ \ \ image:\ docker.io\/gerritcodereview\/gerrit/i \ \ \ \ extra_hosts:" docker-compose.yaml

sed -i "/\ \ \ \ image:\ docker.io\/zuul\/zuul-scheduler:latest/i \ \ \ \ extra_hosts:" docker-compose.yaml

sed -i "/\ \ \ \ image:\ docker.io\/zuul\/zuul-web:latest/i \ \ \ \ extra_hosts:" docker-compose.yaml

sed -i "/\ \ \ \ image:\ docker.io\/zuul\/zuul-executor:latest/i \ \ \ \ extra_hosts:" docker-compose.yaml

sed -i "/\ \ \ \ image:\ docker.io\/zuul\/nodepool-launcher:latest/i \ \ \ \ extra_hosts:" docker-compose.yaml



sed -i "/\ \ \ \ \ \ -\ \"nodessh:\/root\/.ssh:z\"/a \ \ \ \ extra_hosts:" docker-compose.yaml

sed -i "/\ \ \ \ \ \ -\ \"8000:80\"/a \ \ \ \ extra_hosts:" docker-compose.yaml



sed -i "/\ \ \ \ extra_hosts:/a \ \ \ \ \ \ -\ \"gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21\"" docker-compose.yaml

sed -i "/\ \ \ \ extra_hosts:/a \ \ \ \ \ \ -\ \"targetnode-01.kekehoho.com:192.168.59.30\"" docker-compose.yaml



sed -i "/\ \ node:/a \ \ \ \ \ \ -\ TZ=Asia\/Seoul" docker-compose.yaml

sed -i "/\ \ launcher:/a \ \ \ \ \ \ -\ TZ=Asia\/Seoul" docker-compose.yaml

sed -i "/\ \ logs:/a \ \ \ \ \ \ -\ TZ=Asia\/Seoul" docker-compose.yaml

sed -i "/\ \ node:/a \ \ \ \ environment:" docker-compose.yaml

sed -i "/\ \ launcher:/a \ \ \ \ environment:" docker-compose.yaml

sed -i "/\ \ logs:/a \ \ \ \ environment:" docker-compose.yaml

sed -i "/\ \ \ \ environment:/a \ \ \ \ \ \ -\ TZ=Asia\/Seoul" docker-compose.yaml





sed -i "/\ \ \ \ \ \ -\ \"nodessh:\/root\/.ssh:z\"/a \ \ \ \ \ \ -\ \"22:22\"" docker-compose.yaml

sed -i "/\ \ \ \ \ \ -\ \"nodessh:\/root\/.ssh:z\"/a \ \ \ \ ports:" docker-compose.yaml











cp -f etc_zuul/main.yaml etc_zuul/backup.main.yaml

cp -f etc_zuul/zuul.conf etc_zuul/backup.zuul.conf

cp -f etc_nodepool/nodepool.yaml etc_nodepool/backup_nodepool.yaml



**## tenant file** 

sed -i "s/\ \ \ \ \ \ opendev.org:/#\ \ \ \ \ \ opendev.org:/g" etc_zuul/main.yaml

sed -i "s/\ \ \ \ \ \ \ \ untrusted-projects:/#\ \ \ \ \ \ \ \ untrusted-projects:/g" etc_zuul/main.yaml

sed -i "/\ \ \ \ \ \ \ \ \ \ -\ test1/i \ \ \ \ \ \ \ \ untrusted-projects:" etc_zuul/main.yaml

sed -i "s/\ \ \ \ \ \ \ \ \ \ -\ zuul\/zuul-jobs:/#\ \ \ \ \ \ \ \ \ \ -\ zuul\/zuul-jobs:/g" etc_zuul/main.yaml

sed -i "s/\ \ \ \ \ \ \ \ \ \ \ \ \ \ include:/#\ \ \ \ \ \ \ \ \ \ \ \ \ \ include:/g" etc_zuul/main.yaml

sed -i "s/\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ -\ job/#\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ -\ job/g" etc_zuul/main.yaml

sed -i "s/\ \ \ \ \ \ gerrit:/\ \ \ \ \ \ gerrit.kekehoho.com:/g" etc_zuul/main.yaml



sed -i "/#\ \ \ \ \ \ opendev.org:/i \ \ \ \ \ \ onpremisezuulroles.kekehoho.com:" etc_zuul/main.yaml

sed -i "/#\ \ \ \ \ \ opendev.org:/i \ \ \ \ \ \ \ \ untrusted-projects:" etc_zuul/main.yaml

sed -i "/#\ \ \ \ \ \ opendev.org:/i \ \ \ \ \ \ \ \ \ \ -\ onpremise-zuul\/onpremise-zuul-jobs:" etc_zuul/main.yaml

sed -i "/#\ \ \ \ \ \ opendev.org:/i \ \ \ \ \ \ \ \ \ \ \ \ \ \ include:" etc_zuul/main.yaml

sed -i "/#\ \ \ \ \ \ opendev.org:/i \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ -\ job" etc_zuul/main.yaml



**## zuul config file**

sed -i "s/\[connection\ \"opendev.org\"\]/#\[connection\ \"opendev.org\"\]/g" etc_zuul/zuul.conf

sed -i "s/name=opendev/#name=opendev/g" etc_zuul/zuul.conf

sed -i "s/driver=git/#driver=git/g" etc_zuul/zuul.conf

sed -i "s/baseurl=https:\/\/opendev.org/#baseurl=https:\/\/opendev.org/g" etc_zuul/zuul.conf



sed -i "/#\[connection\ \"opendev.org\"\]/i \[connection\ \"onpremisezuulroles.kekehoho.com\"\]" etc_zuul/zuul.conf

sed -i "/#\[connection\ \"opendev.org\"\]/i name=onpremisezuulroles" etc_zuul/zuul.conf

sed -i "/#\[connection\ \"opendev.org\"\]/i driver=git" etc_zuul/zuul.conf

sed -i "/#\[connection\ \"opendev.org\"\]/i baseurl=http:\/\/onpremisezuulroles.kekehoho.com:8080" etc_zuul/zuul.conf



sed -i "s/\[connection\ \"gerrit\"\]/\[connection\ \"gerrit.kekehoho.com\"\]/g" etc_zuul/zuul.conf

sed -i "s/name=gerrit/name=gerrit.kekehoho.com/g" etc_zuul/zuul.conf

sed -i "s/driver=gerrit/driver=gerrit/g" etc_zuul/zuul.conf

sed -i "s/server=gerrit/server=gerrit.kekehoho.com/g" etc_zuul/zuul.conf

sed -i "s/baseurl=http:\/\/gerrit:8080/baseurl=http:\/\/gerrit.kekehoho.com:8080/g" etc_zuul/zuul.conf

sed -i "s/root=http:\/\/localhost:9000/root=http:\/\/zuul.kekehoho.com:9000/g" etc_zuul/zuul.conf

sed -i "s/realm=zuul.example.com/realm=zuul.kekehoho.com/g" etc_zuul/zuul.conf

sed -i "s/client_id=zuul.example.com/client_id=zuul.kekehoho.com/g" etc_zuul/zuul.conf

sed -i "s/url=http:\/\/localhost:9000/url=http:\/\/zuul.kekehoho.com:9000/g" etc_zuul/zuul.conf



**## test node pool file**

**## ci 수행 노드풀 임. 여러 driver 가 있음. aws, gcp, k8s, openstack, vm**

sed -i "s/\ \ \ \ \ \ \ \ \ \ -\ name:\ node/\ \ \ \ \ \ \ \ \ \ -\ name:\ testnode.kekehoho.com/g" etc_nodepool/nodepool.yaml









\# 변경전 잠깐 확인해 본것

root@zuul:~/zuul/doc/source/examples# grep -ri gerrit:8080

keycloak/etc_zuul/zuul.conf:baseurl=http://gerrit:8080

etc_zuul/backup.zuul.conf:baseurl=http://gerrit:8080

playbooks/wait-to-start.sh:    [ $(curl -s -o /dev/null -w "%{http_code}" http://admin:secret@gerrit:8080/a/accounts/zuul/sshkeys) = "200" ] && return

playbooks/setup.yaml:    url: http://gerrit:8080/a/accounts/self/sshkeys

playbooks/setup.yaml:    url: http://gerrit:8080/a/accounts/self/sshkeys

playbooks/setup.yaml:    git push http://admin:secret@gerrit:8080/All-Projects +HEAD:refs/meta/config

playbooks/setup.yaml:    url: http://gerrit:8080/accounts/zuul

playbooks/setup.yaml:    url: http://gerrit:8080/a/accounts/zuul

playbooks/roles/create_project/tasks/main.yaml:  url: http://gerrit:8080/projects/{{ project }}

playbooks/roles/create_project/tasks/main.yaml:    url: http://gerrit:8080/a/projects/{{ project }}

playbooks/roles/create_project/tasks/main.yaml:     git remote add gerrit http://admin:secret@gerrit:8080/{{ project }}









sed -i "s/http:\/\/admin:secret@gerrit:8080/http:\/\/admin:secret@gerrit.kekehoho.com:8080/g" playbooks/wait-to-start.sh

sed -i "s/gerrit:8080/gerrit.kekehoho.com:8080/g" playbooks/setup.yaml

sed -i "s/gerrit:29418/gerrit.kekehoho.com:29418/g" playbooks/setup.yaml

sed -i "s/http:\/\/gerrit:8080/http:\/\/gerrit.kekehoho.com:8080/g" playbooks/roles/create_project/tasks/main.yaml

sed -i "s/gerrit:8080/gerrit.kekehoho.com:8080/g" playbooks/roles/create_project/tasks/main.yaml

sed -i "s/host=localhost/host=gerrit.kekehoho.com/g" playbooks/roles/create_project/tasks/main.yaml







root@zuul:~/zuul/doc/source/examples# docker-compose --log-level DEBUG up





root@zuul:~/zuul/doc/source/examples# docker-compose ps -a

​     Name            Command        State             Ports            

\--------------------------------------------------------------------------------------------------------------------

examples_executor_1    /usr/bin/dumb-init -- sh - ...  Up                            

examples_gerrit_1     /entrypoint.sh          Up    0.0.0.0:29418->29418/tcp, 0.0.0.0:8080->8080/tcp

examples_gerritconfig_1  /usr/bin/dumb-init -- /usr ...  Exit 2                          

examples_launcher_1    /usr/bin/dumb-init -- sh - ...  Up    0.0.0.0:8005->8005/tcp              

examples_logs_1      httpd-foreground         Up    0.0.0.0:8000->80/tcp               

examples_mysql_1     docker-entrypoint.sh mariadbd  Up    3306/tcp                     

examples_node_1      /usr/sbin/sshd -D        Up    0.0.0.0:22->22/tcp                

examples_scheduler_1   /usr/bin/dumb-init -- sh - ...  Exit 1                          

examples_web_1      /usr/bin/dumb-init -- sh - ...  Up    0.0.0.0:9000->9000/tcp              

examples_zk_1       /docker-entrypoint.sh sh - ...  Up    2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp  





root@zuul:~/zuul/doc/source/examples# docker logs examples_gerritconfig_1

TASK [Get Gerrit first login screen] *******************************************

fatal: [localhost]: FAILED! => {"changed": false, "content": "", "elapsed": 0, "msg": "Status code was -1 and not [200]: Connection failure: Remote end closed connection without response", "redirected": false, "status": -1, "url": "http://gerrit.kekehoho.com:8080/a/accounts/self/sshkeys"}

...ignoring



TASK [Add admin SSH key to Gerrit] *********************************************

fatal: [localhost]: FAILED! => {"changed": false, "connection": "close", "content": "", "date": "Tue, 03 May 2022 02:09:46 GMT", "elapsed": 0, "location": "http://gerrit.kekehoho.com:8080/login/%23%2F?account_id=1000000", "msg": "Status code was 302 and not [201]: HTTP Error 302: Found", "redirected": false, "status": 302, "url": "http://gerrit.kekehoho.com:8080/a/accounts/self/sshkeys"}



root@zuul:~/zuul/doc/source/examples# docker logs examples_scheduler_1

2022-05-03T02:07:56+00:00 Wait for certs to be present

2022-05-03T02:07:56+00:00 Wait for mysql to start

/var/playbooks/wait-to-start.sh: connect: Connection refused

/var/playbooks/wait-to-start.sh: line 9: /dev/tcp/mysql/3306: Connection refused

2022-05-03T02:07:57+00:00 Wait for zuul user to be created

2022-05-03T02:10:02+00:00 Timeout waiting for gerrit







root@zuul:~/zuul/doc/source/examples# docker restart examples_scheduler_1











root@zuul:~/zuul/doc/source/examples# docker-compose ps -a

​     Name            Command        State             Ports            

\--------------------------------------------------------------------------------------------------------------------

examples_executor_1    /usr/bin/dumb-init -- sh - ...  Up                            

examples_gerrit_1     /entrypoint.sh          Up    0.0.0.0:29418->29418/tcp, 0.0.0.0:8080->8080/tcp

examples_gerritconfig_1  /usr/bin/dumb-init -- /usr ...  Exit 2                          

examples_launcher_1    /usr/bin/dumb-init -- sh - ...  Up    0.0.0.0:8005->8005/tcp              

examples_logs_1      httpd-foreground         Up    0.0.0.0:8000->80/tcp               

examples_mysql_1     docker-entrypoint.sh mariadbd  Up    3306/tcp                     

examples_node_1      /usr/sbin/sshd -D        Up    0.0.0.0:22->22/tcp                

examples_scheduler_1   /usr/bin/dumb-init -- sh - ...  Up                            

examples_web_1      /usr/bin/dumb-init -- sh - ...  Up    0.0.0.0:9000->9000/tcp              

examples_zk_1       /docker-entrypoint.sh sh - ...  Up    2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp  





docker logs examples_gerritconfig_1



TASK [Update All-Projects config in Gerrit] ************************************

fatal: [localhost]: FAILED! => {"changed": true, "cmd": "set -e\ngit config user.email 'admin@example.com'\ngit commit -a -m 'update config'\ngit push http://admin:secret@gerrit.kekehoho.com:8080/All-Projects +HEAD:refs/meta/config\n", "delta": "0:00:00.004547", "end": "2022-05-03 02:17:43.288302", "msg": "non-zero return code", "rc": 1, "start": "2022-05-03 02:17:43.283755", "stderr": "", "stderr_lines": [], "stdout": "Not currently on any branch.\nnothing to commit, working tree clean", "stdout_lines": ["Not currently on any branch.", "nothing to commit, working tree clean"]}



PLAY RECAP *********************************************************************

localhost         : ok=12  changed=3  unreachable=0  failed=1  skipped=0  rescued=0  ignored=0  













root@zuul:~/zuul/doc/source/examples# docker-compose ps -a

​    Name           Command        State            Ports            

\----------------------------------------------------------------------------------------------------------------

examples_executor_1  /usr/bin/dumb-init -- sh - ...  Up                            

examples_gerrit_1   /entrypoint.sh          Up   0.0.0.0:29418->29418/tcp, 0.0.0.0:8080->8080/tcp

examples_launcher_1  /usr/bin/dumb-init -- sh - ...  Up   0.0.0.0:8005->8005/tcp              

examples_logs_1    httpd-foreground         Up   0.0.0.0:8000->80/tcp               

examples_mysql_1    docker-entrypoint.sh mariadbd  Up   3306/tcp                     

examples_node_1    /usr/sbin/sshd -D        Up   0.0.0.0:22->22/tcp                

examples_scheduler_1  /usr/bin/dumb-init -- sh - ...  Up                            

examples_web_1     /usr/bin/dumb-init -- sh - ...  Up   0.0.0.0:9000->9000/tcp              

examples_zk_1     /docker-entrypoint.sh sh - ...  Up   2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp 











scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader: 1 errors detected during example-tenant tenant configuration loading

scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader: Zuul encountered an error while accessing the repo onpremise-

scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader:  zuul/onpremise-zuul-jobs. The error was:

scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader:  

scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader:   Cmd('git') failed due to: exit code(128)

scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader:    cmdline: git ls-remote --heads --tags http://onpremisezuulroles.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs

scheduler_1   | 2022-05-10 01:29:11,936 WARNING zuul.ConfigLoader:    stderr: 'fatal: remote error: onpremise-zuul/onpremise-zuul-jobs unavailable'

scheduler_1   | 2022-05-10 01:29:11,955 INFO zuul.Scheduler: Reconfiguration beginning (smart=True, tenants=None)

scheduler_1   | 2022-05-10 01:29:11,958 INFO zuul.ConfigLoader: Loading configuration from /etc/zuul/main.yaml

scheduler_1   | 2022-05-10 01:29:11,999 INFO zuul.Scheduler: Reconfiguration complete (smart: True, tenants: None, duration: 0.043 seconds)

scheduler_1   | 2022-05-10 01:29:12,001 INFO zuul.Scheduler: Reconfigured tenants: []

scheduler_1   | 2022-05-10 01:29:12,002 INFO zuul.Scheduler: Config priming complete (duration: 12.666 seconds)

executor_1   | 2022-05-10 01:29:12,011 INFO zuul.ComponentRegistry: Component scheduler 5b09520fcae70000000000 updated: {'hostname': '5b09520fcae7', 'kind': 'scheduler', 'model_api': 7, 'state': 'running', 'version': '6.0.1.dev7 0c7007ff5'}

web_1      | 2022-05-10 01:29:12,016 INFO zuul.ComponentRegistry: Component scheduler 5b09520fcae70000000000 updated: {'hostname': '5b09520fcae7', 'kind': 'scheduler', 'model_api': 7, 'state': 'running', 'version': '6.0.1.dev7 0c7007ff5'}

scheduler_1   | 2022-05-10 01:29:12,018 INFO zuul.ComponentRegistry: Component scheduler 5b09520fcae70000000000 updated: {'hostname': '5b09520fcae7', 'kind': 'scheduler', 'model_api': 7, 'state': 'running', 'version': '6.0.1.dev7 0c7007ff5'}

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader: 1 errors detected during example-tenant tenant configuration loading

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader: Zuul encountered an error while accessing the repo onpremise-

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader:  zuul/onpremise-zuul-jobs. The error was:

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader:  

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader:   Cmd('git') failed due to: exit code(128)

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader:    cmdline: git ls-remote --heads --tags http://onpremisezuulroles.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs

web_1      | 2022-05-10 01:29:12,079 WARNING zuul.ConfigLoader:    stderr: 'fatal: remote error: onpremise-zuul/onpremise-zuul-jobs unavailable'





**아직 gerrit 에 onpremise-zuul/onpremise-zuul-jobs 저장소를 만들지 않았기 때문에 나는 에러이므로 무시**







root@zuul:~/zuul/doc/source/examples# grep -ri example-tenant

keycloak/keycloak/zuul-demo-data.json:   "name": "example-tenant-admin",

keycloak/keycloak/zuul-demo-data.json:   "path": "/example-tenant-admin",

keycloak/keycloak/zuul-demo-data.json:    "/example-tenant-admin"

keycloak/etc_zuul/main.yaml:  name: example-tenant

etc_zuul/main.yaml:  name: example-tenant





root@zuul:~/zuul/doc/source/examples# cat etc_zuul/main.yaml

\- tenant:

  name: example-tenant

  source:

   gerrit.kekehoho.com:

​    config-projects:

​     \- zuul-config

\#    untrusted-projects:

​    untrusted-projects:

​     \- test1

​     \- test2

\# 기본 ansible role 원격 저장소 정의 부분 

   onpremisezuulroles.kekehoho.com:

​    untrusted-projects:

​     \- onpremise-zuul/onpremise-zuul-jobs:

​       include:

​        \- job

\#   opendev.org:

\#    untrusted-projects:

\#     - zuul/zuul-jobs:

\#       include:

\#        - job





root@zuul:~/zuul/doc/source/examples# cat etc_zuul/zuul.conf 

[zookeeper]

hosts=zk:2281

tls_cert=/var/certs/certs/client.pem

tls_key=/var/certs/keys/clientkey.pem

tls_ca=/var/certs/certs/cacert.pem



[keystore]

password=secret



[scheduler]

tenant_config=/etc/zuul/main.yaml



[connection "gerrit.kekehoho.com"]

name=gerrit.kekehoho.com

driver=gerrit

server=gerrit.kekehoho.com

sshkey=/var/ssh/zuul

user=zuul

password=secret

baseurl=http://gerrit.kekehoho.com:8080

auth_type=basic



***#\* 아래 정의 해야 위 파일이 적용됨 \*# https://zuul-ci.org/docs/zuul-jobs/install.html\***

***[connection "onpremisezuulroles.kekehoho.com"]\***

***name=onpremisezuulroles\***

***driver=git\***

baseurl=http://onpremisezuulroles.kekehoho.com:8080

\#[connection "opendev.org"]

\#name=opendev

\#driver=git

\#baseurl=https://opendev.org



[database]

\# Use variable interpolation to supply the password from the

\# docker-compose file.

\# https://zuul-ci.org/docs/zuul/latest/configuration.html

dburi=mysql+pymysql://zuul:%(ZUUL_MYSQL_PASSWORD)s@mysql/zuul



[web]

listen_address=0.0.0.0

port=9000

root=http://zuul.kekehoho.com:9000



[executor]

private_key_file=/var/ssh/nodepool

default_username=root

trusted_rw_paths=/srv/static/logs



[auth zuul_operator]

driver=HS256

allow_authz_override=true

realm=zuul.kekehoho.com

client_id=zuul.kekehoho.com

issuer_id=zuul_operator

secret=exampleSecret



[webclient]

url=http://zuul.kekehoho.com:9000

verify_ssl=false

















http://gerrit.kekehoho.com:8080/



http://zuul.kekehoho.com:9000/tenants



http://zuul.kekehoho.comt/example-tenant/status















**## gerrit 에 일반 사용자 생성 (hoho 계정)**

![image-20220611215838312](images/zuul/image-20220611215838312.png)



![image-20220611215848791](images/zuul/image-20220611215848791.png)



![image-20220611215858416](images/zuul/image-20220611215858416.png)



![image-20220611215910231](images/zuul/image-20220611215910231.png)



root@zuul:~/zuul/doc/source/examples/playbooks# cat wait-to-start.sh |grep sshkeys

​    [ $(curl -s -o /dev/null -w "%{http_code}" http://admin:secret@gerrit.kekehoho.com:8080/a/accounts/zuul/sshkeys) = "200" ] && return



root@zuul:~/zuul/doc/source/examples/playbooks# curl http://admin:secret@gerrit.kekehoho.com:8080/a/accounts/zuul/sshkeys

)]}'

[{"seq":1,"ssh_public_key":"***ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCwLI1+ew0v/mgrX8Q+dwOfmmOJ97MFi2GJCzs0edj1WmP8WucKWaUGbCIEVAaWKDlVV7xsTuCumf8zCuvoMcwlBMXuzcTLegvHYNXnuKkjAeVBzVqS/e80g3rWMsNkrMF8rwE9kTk2HDhVtCheOoVYuRFhwV2uUudgtIzlwWJcpqwI+HRUC0CJmUoHUA4jKgEpdlscSB/AxF3vM7VZ5BU51eCLkf9jfz2pUFv1xioJjOAWogMvfgS617ypvY2oujOZml6fXPlyQn1WxcyAxoPAFQJ3xhFEF0qUUAIjad/FCStdjVoqsRz6chxmDVj9NX8D56aVXerNe2z9im3QMSQvmROnkv+ao3WMK+VVnJ69+9Cr8wV7xu1TZBeyHlLNhB8A/G6eTNLp2eDXL8OOY089yefQ1gXCI5D8xZUq1cC9G7s9qQtkQewJqn//XcNfXJBwqSv6BD4m6R1i5n/G4i9I+QhMfqexxXvec/hwPm5p3UxnnQHmIFXIYAlVc27+Q8U\u003d root@f72370f49b35\***","encoded_key":"AAAAB3NzaC1yc2EAAAADAQABAAABgQCwLI1+ew0v/mgrX8Q+dwOfmmOJ97MFi2GJCzs0edj1WmP8WucKWaUGbCIEVAaWKDlVV7xsTuCumf8zCuvoMcwlBMXuzcTLegvHYNXnuKkjAeVBzVqS/e80g3rWMsNkrMF8rwE9kTk2HDhVtCheOoVYuRFhwV2uUudgtIzlwWJcpqwI+HRUC0CJmUoHUA4jKgEpdlscSB/AxF3vM7VZ5BU51eCLkf9jfz2pUFv1xioJjOAWogMvfgS617ypvY2oujOZml6fXPlyQn1WxcyAxoPAFQJ3xhFEF0qUUAIjad/FCStdjVoqsRz6chxmDVj9NX8D56aVXerNe2z9im3QMSQvmROnkv+ao3WMK+VVnJ69+9Cr8wV7xu1TZBeyHlLNhB8A/G6eTNLp2eDXL8OOY089yefQ1gXCI5D8xZUq1cC9G7s9qQtkQewJqn//XcNfXJBwqSv6BD4m6R1i5n/G4i9I+QhMfqexxXvec/hwPm5p3UxnnQHmIFXIYAlVc27+Q8U\u003d","algorithm":"ssh-rsa","comment":"root@f72370f49b35","valid":true}]









root@zuul:~/zuul/doc/source/examples# cat ssh_host_ed25519_key.pub 

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOgHJYejINIKzUiuSJ2MN8uPc+dfFrZ9JH1hLWS8gI+g root@3296dc088846



New SSH key 부분에 위 pub key 를 넣는다.

![image-20220611215924328](images/zuul/image-20220611215924328.png)



![image-20220611215935533](images/zuul/image-20220611215935533.png)



![image-20220611215946818](images/zuul/image-20220611215946818.png)



![image-20220611215955938](images/zuul/image-20220611215955938.png)



이렇게 hoho 계정이 생성되어 있음.















\####################



\####

\#### ssh 연결 확인



gerrit에서 clone 받을 때 Permission denied (publickey) 에러가 났을 때 확인해야 할 점 :

\1. ssh 접근할 때 username이 gerrit의 'Settings' -> 'Profile' 메뉴 아래에 있는 username과 동일한지 확인

\2. 'Settings' -> 'SSH Public Keys'에 public key 등록했는지 확인

\3. ssh -vv -p 29418 john.doe@git.example.com 명령어로 접근이 되는지 확인



gerrit 에 hoho 계정 생성할때 hoho 계정이 사용할 ssh public key 를 소스저장소 서버(gerrit server)에 자동로그인할 수 있게 설정 해야함. 



root@zuul:~# cd ~

cp ~/zuul/doc/source/examples/ssh_host_ed25519_key* ~/.ssh

mv ~/.ssh/ssh_host_ed25519_key ~/.ssh/id_ed25519

chmod 600 ~/.ssh/id_ed25519







root@zuul:~# ssh -vv -p 29418 hoho@gerrit.kekehoho.com

OpenSSH_8.4p1 Ubuntu-6ubuntu2, OpenSSL 1.1.1l 24 Aug 2021

debug1: Reading configuration data /etc/ssh/ssh_config

debug1: /etc/ssh/ssh_config line 19: include /etc/ssh/ssh_config.d/*.conf matched no files

debug1: /etc/ssh/ssh_config line 21: Applying options for *

debug2: resolving "gerrit.kekehoho.com" port 29418

debug2: ssh_connect_direct

debug1: Connecting to gerrit.kekehoho.com [192.168.59.21] port 29418.

debug1: Connection established.

debug1: identity file /root/.ssh/id_rsa type -1

debug1: identity file /root/.ssh/id_rsa-cert type -1

debug1: identity file /root/.ssh/id_dsa type -1

debug1: identity file /root/.ssh/id_dsa-cert type -1

debug1: identity file /root/.ssh/id_ecdsa type -1

debug1: identity file /root/.ssh/id_ecdsa-cert type -1

debug1: identity file /root/.ssh/id_ecdsa_sk type -1

debug1: identity file /root/.ssh/id_ecdsa_sk-cert type -1

debug1: identity file /root/.ssh/id_ed25519 type 3

debug1: identity file /root/.ssh/id_ed25519-cert type -1

debug1: identity file /root/.ssh/id_ed25519_sk type -1

debug1: identity file /root/.ssh/id_ed25519_sk-cert type -1

debug1: identity file /root/.ssh/id_xmss type -1

debug1: identity file /root/.ssh/id_xmss-cert type -1

debug1: Local version string SSH-2.0-OpenSSH_8.4p1 Ubuntu-6ubuntu2

debug1: Remote protocol version 2.0, remote software version GerritCodeReview_3.5.0.1 (APACHE-SSHD-2.6.0)

debug1: no match: GerritCodeReview_3.5.0.1 (APACHE-SSHD-2.6.0)

debug2: fd 3 setting O_NONBLOCK

debug1: Authenticating to gerrit.kekehoho.com:29418 as 'hoho'

debug1: SSH2_MSG_KEXINIT sent

debug1: SSH2_MSG_KEXINIT received

debug2: local client KEXINIT proposal

debug2: KEX algorithms: curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256,ext-info-c

debug2: host key algorithms: ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com,rsa-sha2-256-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,sk-ecdsa-sha2-nistp256@openssh.com,ssh-ed25519,sk-ssh-ed25519@openssh.com,rsa-sha2-512,rsa-sha2-256,ssh-rsa

debug2: ciphers ctos: chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com

debug2: ciphers stoc: chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com

debug2: MACs ctos: umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1

debug2: MACs stoc: umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1

debug2: compression ctos: none,zlib@openssh.com,zlib

debug2: compression stoc: none,zlib@openssh.com,zlib

debug2: languages ctos: 

debug2: languages stoc: 

debug2: first_kex_follows 0 

debug2: reserved 0 

debug2: peer server KEXINIT proposal

debug2: KEX algorithms: ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256,diffie-hellman-group18-sha512,diffie-hellman-group17-sha512,diffie-hellman-group16-sha512,diffie-hellman-group15-sha512,diffie-hellman-group14-sha256

debug2: host key algorithms: ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-ed25519,rsa-sha2-512,rsa-sha2-256,ssh-rsa

debug2: ciphers ctos: aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,aes128-cbc,aes192-cbc,aes256-cbc

debug2: ciphers stoc: aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,aes128-cbc,aes192-cbc,aes256-cbc

debug2: MACs ctos: hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1

debug2: MACs stoc: hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1

debug2: compression ctos: none

debug2: compression stoc: none

debug2: languages ctos: 

debug2: languages stoc: 

debug2: first_kex_follows 0 

debug2: reserved 0 

debug1: kex: algorithm: ecdh-sha2-nistp256

debug1: kex: host key algorithm: ecdsa-sha2-nistp256

debug1: kex: server->client cipher: aes128-ctr MAC: hmac-sha2-256-etm@openssh.com compression: none

debug1: kex: client->server cipher: aes128-ctr MAC: hmac-sha2-256-etm@openssh.com compression: none

debug1: expecting SSH2_MSG_KEX_ECDH_REPLY

debug1: Server host key: ecdsa-sha2-nistp256 SHA256:L73lkSv6VXWE2AT1wHctKAkOMrrcDHXhAsqG2X5hDsQ

debug1: Host '[gerrit.kekehoho.com]:29418' is known and matches the ECDSA host key.

debug1: Found key in /root/.ssh/known_hosts:1

debug2: set_newkeys: mode 1

debug1: rekey out after 4294967296 blocks

debug1: SSH2_MSG_NEWKEYS sent

debug1: expecting SSH2_MSG_NEWKEYS

debug1: SSH2_MSG_NEWKEYS received

debug2: set_newkeys: mode 0

debug1: rekey in after 4294967296 blocks

debug1: Will attempt key: /root/.ssh/id_rsa 

debug1: Will attempt key: /root/.ssh/id_dsa 

debug1: Will attempt key: /root/.ssh/id_ecdsa 

debug1: Will attempt key: /root/.ssh/id_ecdsa_sk 

debug1: Will attempt key: /root/.ssh/id_ed25519 ED25519 SHA256:2Rsc0Zen65clbUZ/pSIG0btC/Q01ufOpdYhmQm2nkAI

debug1: Will attempt key: /root/.ssh/id_ed25519_sk 

debug1: Will attempt key: /root/.ssh/id_xmss 

debug2: pubkey_prepare: done

debug2: service_accept: ssh-userauth

debug1: SSH2_MSG_SERVICE_ACCEPT received

debug1: Authentications that can continue: publickey

debug1: Next authentication method: publickey

debug1: Trying private key: /root/.ssh/id_rsa

debug1: Trying private key: /root/.ssh/id_dsa

debug1: Trying private key: /root/.ssh/id_ecdsa

debug1: Trying private key: /root/.ssh/id_ecdsa_sk

debug1: Offering public key: /root/.ssh/id_ed25519 ED25519 SHA256:2Rsc0Zen65clbUZ/pSIG0btC/Q01ufOpdYhmQm2nkAI

debug2: we sent a publickey packet, wait for reply

debug1: Server accepts key: /root/.ssh/id_ed25519 ED25519 SHA256:2Rsc0Zen65clbUZ/pSIG0btC/Q01ufOpdYhmQm2nkAI

debug1: Authentication succeeded (publickey).

Authenticated to gerrit.kekehoho.com ([192.168.59.21]:29418).

debug1: channel 0: new [client-session]

debug2: channel 0: send open

debug1: Entering interactive session.

debug1: pledge: network

debug2: channel_input_open_confirmation: channel 0: callback start

debug2: fd 3 setting TCP_NODELAY

debug2: client_session2_setup: id 0

debug2: channel 0: request pty-req confirm 1

debug1: Sending environment.

debug1: Sending env LANG = en_US.UTF-8

debug2: channel 0: request env confirm 0

debug2: channel 0: request shell confirm 1

debug2: channel_input_open_confirmation: channel 0: callback done

debug2: channel 0: open confirm rwindow 2097152 rmax 32768

debug2: channel_input_status_confirm: type 99 id 0

debug2: PTY allocation request accepted on channel 0

debug2: channel_input_status_confirm: type 99 id 0

debug2: shell request accepted on channel 0

debug2: channel 0: rcvd ext data 284

debug2: channel 0: rcvd eof

debug2: channel 0: output open -> drain

debug1: client_input_channel_req: channel 0 rtype exit-status reply 0

debug2: channel 0: rcvd close

debug2: channel 0: chan_shutdown_read (i0 o1 sock -1 wfd 4 efd 6 [write])

debug2: channel 0: input open -> closed

debug2: channel 0: obuf_empty delayed efd 6/(284)



 ****  Welcome to Gerrit Code Review  ****



 Hi hohofullname, you have successfully connected over SSH.



 Unfortunately, interactive shells are disabled.

 To clone a hosted Git repository, use:



 git clone ssh://hoho@gerrit.kekehoho.com:29418/REPOSITORY_NAME.git



debug2: channel 0: written 284 to efd 6

debug2: channel 0: obuf empty

debug2: channel 0: chan_shutdown_write (i3 o1 sock -1 wfd 5 efd 6 [write])

debug2: channel 0: output drain -> closed

debug2: channel 0: almost dead

debug2: channel 0: gc: notify user

debug2: channel 0: gc: user detached

debug2: channel 0: send close

debug2: channel 0: is dead

debug2: channel 0: garbage collecting

debug1: channel 0: free: client-session, nchannels 1

Connection to gerrit.kekehoho.com closed.

Transferred: sent 2576, received 2012 bytes, in 0.0 seconds

Bytes per second: sent 56258.0, received 43940.6

debug1: Exit status 127



\#####

\#####





root@zuul:~# git config --global user.name "hoho"

git config --global user.email hoho@kekehoho.com.com

git config --global --add gitreview.username "hoho"



root@zuul:~/zuul-config# cat ~/.gitconfig 

\# This is Git's per-user configuration file.

[user]

​    name = hoho

​    email = hoho@kekehoho.com.com

\# Please adapt and uncomment the following lines:

\#    name = root

\#    email = root@zuul.kekehoho.com

[http]

​    sslVerify = false

[gitreview]

​    username = hoho













\##########################





zuul 에서 사용할 Ansible Role 저장소 만들기 : admin 으로 로그인해서 만듬

![image-20220611220015929](images/zuul/image-20220611220015929.png)





Repository name : **onpremise-zuul/onpremise-zuul-jobs**

Owner : user/hohofullname (hoho)

Create initial empty commit : False



\# git clone "http://gerrit.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs"

\# git clone "ssh://admin@gerrit.kekehoho.com:29418/onpremise-zuul/onpremise-zuul-jobs"





cd ~; \

git clone https://opendev.org/zuul/zuul-jobs.git; \

cd zuul-jobs



root@zuul:~/zuul-jobs# git push --mirror ssh://hoho@gerrit.kekehoho.com:29418/onpremise-zuul/onpremise-zuul-jobs

...

...

To ssh://gerrit.kekehoho.com:29418/onpremise-zuul/onpremise-zuul-jobs

 \* [new branch]   master -> master

 ! [remote rejected] refs/meta/config (prohibited by Gerrit: not permitted: delete)

 ! [remote rejected] origin/HEAD -> origin/HEAD (prohibited by Gerrit: not permitted: create)

 ! [remote rejected] origin/master -> origin/master (prohibited by Gerrit: not permitted: create)

error: failed to push some refs to 'ssh://gerrit.kekehoho.com:29418/onpremise-zuul/onpremise-zuul-jobs'



\# 확인

root@zuul:~# cd ~; rm -rf zuul-jobs/



root@zuul:~# git clone "ssh://hoho@gerrit.kekehoho.com:29418/onpremise-zuul/onpremise-zuul-jobs"



root@zuul:~# ls onpremise-zuul-jobs/

bindep.txt    doc   linters-requirements.txt README.rst setup.cfg test-constraints.txt test-requirements.txt tools  util-tasks zuul-tests.d

CONTRIBUTING.rst LICENSE playbooks         roles    setup.py  test-playbooks    tests         tox.ini zuul.d



root@zuul:~# rm -rf onpremise-zuul-jobs/



root@zuul:~# git clone "http://gerrit.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs"



root@zuul:~# rm -rf onpremise-zuul-jobs/



root@zuul:~# git ls-remote --heads --tags http://onpremisezuulroles.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs

fc133bd165054ac758e695d985c53237c3262e16    refs/heads/master









scheduler_1   | 2022-05-10 02:00:12,847 INFO zuul.GerritConnection: Got branches for onpremise-zuul/onpremise-zuul-jobs







\###################

















root@zuul:~/zuul/doc/source/examples# cd ~; rm -rf ~/zuul-config

git clone "ssh://hoho@gerrit.kekehoho.com:29418/zuul-config"



Cloning into 'zuul-config'...

remote: Counting objects: 3, done

remote: Finding sources: 100% (3/3)

Receiving objects: 100% (3/3), 271 bytes | 271.00 KiB/s, done.

remote: Total 3 (delta 0), reused 3 (delta 0)



root@zuul:~# cd zuul-config/

mkdir zuul.d





root@zuul:~/zuul-config# cat .gitreview

[gerrit]

host=gerrit.kekehoho.com

port=29418

project=zuul-config









root@zuul:~/zuul-config# cp ~/zuul/doc/source/examples/zuul-config/zuul.d/pipelines.yaml ./zuul.d/

cp ~/zuul/doc/source/examples/zuul-config/zuul.d/projects.yaml ./zuul.d/

cp ~/zuul/doc/source/examples/zuul-config/zuul.d/jobs.yaml ./zuul.d



**sed -i "s/\ \ \ \ \ \ gerrit:/\ \ \ \ \ \ gerrit.kekehoho.com:/g" ./zuul.d/pipelines.yaml**













root@zuul:~/zuul-config# git add zuul.d

git commit -m "Add initial Zuul configuration"

git review





[master 8b172dd] Add initial Zuul configuration

 3 files changed, 73 insertions(+)

 create mode 100644 zuul.d/jobs.yaml

 create mode 100644 zuul.d/pipelines.yaml

 create mode 100644 zuul.d/projects.yaml





Creating a git remote called 'gerrit' that maps to:

​    ssh://hoho@gerrit.kekehoho.com:29418/zuul-config

Your change was committed before the commit hook was installed.

Amending the commit to add a gerrit change id.

remote: 

remote: Processing changes: new: 1 (\)

remote: Processing changes: refs: 1, new: 1 (\)

remote: Processing changes: refs: 1, new: 1 (\)

remote: Processing changes: refs: 1, new: 1 (\)

remote: Processing changes: refs: 1, new: 1, done       

remote: 

remote: SUCCESS     

remote: 

remote:  http://gerrit.kekehoho.com:8080/c/zuul-config/+/1 Add initial Zuul configuration [NEW]     

remote: 

To ssh://gerrit.kekehoho.com:29418/zuul-config

 \* [new reference]  HEAD -> refs/for/master





















http://gerrit.kekehoho.com:8080

\1. CLICK the avatar image in the top right corner then click "*Sign out"*.

\2. CLICK "Sign In" & login "***admin\***"



![image-20220611220031101](images/zuul/image-20220611220031101.png)



CHANGES -> Open Click

![image-20220611220039350](images/zuul/image-20220611220039350.png)You will then see a list of open changes; click on the change you uploaded



Vote Code-Review: +2, Verified: +2, Workflow: +1, and then click *Send* to leave your approval votes.

![image-20220611220054987](images/zuul/image-20220611220054987.png)



Once the required votes have been set, the *Submit* button will appear in the top right; click it.

This will cause the change to be merged immediately.

This is normally handled by Zuul, but as the administrator you can bypass Zuul to forcibly merge a change.

![image-20220611220113666](images/zuul/image-20220611220113666.png)



![image-20220611220121887](images/zuul/image-20220611220121887.png)



![image-20220611220131921](images/zuul/image-20220611220131921.png)



Now that the initial configuration has been bootstrapped, you should not need to bypass testing and code review again,

so switch back to the account you created for yourself (= hoho account).

Click on the avatar image in the top right corner then click *Sign out*.



Then click the *Sign in* link again.



***And click your username to log into your account. (= hoho account)\***





\##########################

\### TEST what happened to zuul



http://zuul.kekehoho.com:9000/tenants



Zuul은 이제 기본 검사 및 게이트 구성으로 실행됩니다.

시스템의 현재 상태를 보려면 http://zuul.kekehoho.com:9000/tenants --> example-tenants 를 방문하십시오.

유휴 상태여야 하지만 다음 단계에서 이 페이지를 열어 두면 자동으로 업데이트되는 것을 볼 수 있습니다.







We can now begin adding Zuul configuration to one of our [untrusted projects](https://zuul-ci.org/docs/zuul/latest/glossary.html#term-untrusted-project).

Start by cloning the ***test1\*** project which was created by the setup script.



![image-20220611220145584](images/zuul/image-20220611220145584.png)



root@zuul:~# cd ~; rm -rf ~/test1

git clone "ssh://hoho@gerrit.kekehoho.com:29418/test1"

cd test1

mkdir playbooks



Cloning into 'test1'...

remote: Counting objects: 3, done

remote: Finding sources: 100% (3/3)

Receiving objects: 100% (3/3), 265 bytes | 265.00 KiB/s, done.

remote: Total 3 (delta 0), reused 3 (delta 0)





root@zuul:~/test1# cp ~/zuul/doc/source/examples/test1/playbooks/testjob.yaml ~/test1/playbooks/testjob.yaml







Now define a Zuul job which runs that playbook.

Zuul will read its configuration from any of zuul.d/ or .zuul.d/ directories, or the files zuul.yaml or .zuul.yaml.

Generally in an *untrusted project* which isn’t dedicated entirely to Zuul, it’s best to put Zuul’s configuration in a hidden file.

Copy the following to .zuul.yaml in the root of the project:





root@zuul:~/test1# cp ~/zuul/doc/source/examples/test1/zuul.yaml ~/test1/zuul.yaml



root@zuul:~/test1# git add zuul.yaml playbooks

git commit -m "Add test Zuul job"

git review



[master 3aa0f3c] Add test Zuul job

 2 files changed, 15 insertions(+)

 create mode 100644 .zuul.yaml

 create mode 100644 playbooks/testjob.yaml

Creating a git remote called 'gerrit' that maps to:

​    ssh://hoho@gerrit.kekehoho.com:29418/test1

Your change was committed before the commit hook was installed.

Amending the commit to add a gerrit change id.

remote: 

remote: Processing changes: refs: 1, new: 1

remote: Processing changes: refs: 1, new: 1

remote: Processing changes: refs: 1, new: 1

remote: Processing changes: refs: 1, new: 1, done       

remote: 

remote: SUCCESS     

remote: 

remote:  http://gerrit.kekehoho.com:8080/c/test1/+/2 Add test Zuul job [NEW]     

remote: 

To ssh://gerrit.kekehoho.com:29418/test1

 \* [new reference]  HEAD -> refs/for/master







Zuul will dynamically evaluate proposed changes to its configuration in *untrusted projects* immediately, so shortly after your change is uploaded, Zuul will run the new job and report back on the change.











Visit http://gerrit.kekehoho.com:8080/dashboard/self with "***hoho\***" account and

open the change(test1 (Add test Zuul job)) you just uploaded.



If the build is complete, Zuul should have left a Verified: +1 vote on the change,

along with a comment at the bottom.



Expand the comments and you should see that the job succeeded, and a link to the build result in Zuul is provided.

***You can follow that link to see some information about the build, but you won’t find any logs since Zuul hasn’t been told where to save them yet.\***

![image-20220611220202647](images/zuul/image-20220611220202647.png)



This means everything is working so far, but we need to configure a bit more before we have a useful job.

![image-20220611220214979](images/zuul/image-20220611220214979.png)



root@zuul:~/test1# docker logs examples_launcher_1

2022-05-02T06:58:20+00:00 Wait for certs to be present

/usr/local/lib/python3.9/site-packages/paramiko/transport.py:236: CryptographyDeprecationWarning: Blowfish has been deprecated

 "class": algorithms.Blowfish,

2022-05-02 06:58:28,706 INFO nodepool.NodePool: Nodepool launcher 5.0.1.dev70 starting

2022-05-02 06:58:31,954 WARNING nodepool.zk.ZooKeeper: Retrying zookeeper connection

2022-05-02 06:58:34,517 INFO nodepool.CleanupWorker: Starting

2022-05-02 06:58:34,517 INFO nodepool.DeletedNodeWorker: Starting

2022-05-02 06:58:34,517 INFO nodepool.StatsWorker: Starting

2022-05-02 06:58:34,517 INFO nodepool.NodePool: Starting PoolWorker.static-vms-main

2022-05-02 07:22:15,235 INFO nodepool.PoolWorker.static-vms-main: [e: bd38c86524e34e0bbb49ac5f5a0a2515] [node_request: 200-0000000000] Assigning node request <NodeRequest {'state': 'requested', 'state_time': 1651476130.2662716, 'declined_by': [], 'node_types': ['ubuntu-focal'], 'nodes': [], 'reuse': True, 'requestor': '6c98b29488bc4c0989a0f0f87a42dd78', 'requestor_data': {'build_set_uuid': '3dbfa3e53c6945139791d5c6f68644f2', 'tenant_name': 'example-tenant', 'pipeline_name': 'check', 'job_name': 'testjob'}, 'provider': None, 'relative_priority': 0, 'event_id': 'bd38c86524e34e0bbb49ac5f5a0a2515', 'created_time': 1651476130.2662854, 'tenant_name': 'example-tenant', 'id': '200-0000000000', 'stat': ZnodeStat(czxid=119326, mzxid=119326, ctime=1651476130266, mtime=1651476130266, version=0, cversion=0, aversion=0, ephemeralOwner=0, dataLength=452, numChildren=0, pzxid=119326)}>

2022-05-02 07:22:24,902 INFO nodepool.DeletedNodeWorker: Deleting used instance testnode.kekehoho.com from static-vms

2022-05-02 07:22:24,915 INFO nodepool.NodeDeleter: Deleting ZK node id=0000000000, state=deleting, external_id=testnode.kekehoho.com











**############################**

**### Configure a Base Job**

\### LOGGING



root@zuul:~/test1# cd ~/zuul-config/

mkdir -p playbooks/base



 \# pre.yaml : This playbook does two things; first it creates a new SSH key and adds it to all of the hosts in the ***inventory\***, and removes the private key that Zuul normally uses to log into nodes from the running SSH agent.



root@zuul:~/zuul-config# cp ~/zuul/doc/source/examples/zuul-config/playbooks/base/pre.yaml ./playbooks/base/

cp ~/zuul/doc/source/examples/zuul-config/playbooks/base/post-ssh.yaml ./playbooks/base/

cp ~/zuul/doc/source/examples/zuul-config/playbooks/base/post-logs.yaml ./playbooks/base/



***sed -i "s/localhost:8000/zuullognode.kekehoho.com:8000/g" ./playbooks/base/post-logs.yaml\***



root@zuul:~/zuul-config# docker exec -it examples_executor_1 cat /etc/hosts

127.0.0.1    localhost

::1   localhost ip6-localhost ip6-loopback

fe00::0 ip6-localnet

ff00::0 ip6-mcastprefix

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

192.168.59.21  gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com

172.20.0.11   1afd25e2ec99



root@zuul:~/zuul-config# docker exec -it examples_logs_1 cat /etc/hosts

127.0.0.1    localhost

::1   localhost ip6-localhost ip6-loopback

fe00::0 ip6-localnet

ff00::0 ip6-mcastprefix

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

192.168.59.21  gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com

172.20.0.3   768104719c9f







Now that the new playbooks are in place, update the ***base\*** job definition to include them.

Overwrite ***zuul.d/jobs.yaml\*** with the following:



root@zuul:~/zuul-config# cp ~/zuul/doc/source/examples/zuul-config/zuul.d/jobs2.yaml ./zuul.d/jobs.yaml



***sed -i "s/\ \ \ \ \ \ \ \ -\ name:\ ubuntu-focal/\ \ \ \ \ \ \ \ -\ name:\ testnode.kekehoho.com/g" ./zuul.d/jobs.yaml\***



***sed -i "s/\ \ \ \ \ \ -\ zuul:\ zuul\/zuul-jobs/\ \ \ \ \ \ -\ zuul:\ onpremise-zuul\/onpremise-zuul-jobs/g" ./zuul.d/jobs.yaml\***

















root@zuul:~/zuul-config# git add playbooks zuul.d/jobs.yaml

git commit -m "Update Zuul base job"

git review



[master 6d9957e] Update Zuul base job

 4 files changed, 29 insertions(+), 1 deletion(-)

 create mode 100644 playbooks/base/post-logs.yaml

 create mode 100644 playbooks/base/post-ssh.yaml

 create mode 100644 playbooks/base/pre.yaml

remote: 

remote: Processing changes: refs: 1, new: 1

remote: Processing changes: refs: 1, new: 1

remote: Processing changes: refs: 1, new: 1

remote: Processing changes: refs: 1, new: 1, done       

remote: 

remote: SUCCESS     

remote: 

remote:  http://gerrit.kekehoho.com:8080/c/zuul-config/+/3 Update Zuul base job [NEW]     

remote: 

To ssh://gerrit.kekehoho.com:29418/zuul-config

 \* [new reference]  HEAD -> refs/for/master











LOGIN "***hoho\***" account on Gerrit : http://gerrit.kekehoho.com:8080/dashboard/self

![image-20220611220234497](images/zuul/image-20220611220234497.png)



open the ***zuul-config (Update Zuul base job)\*** change you just uploaded.



You should see a Verified +1 vote from Zuul.

Click *Reply* then vote Code-Review: +2 and

​                   Workflow: +1 then

​                   Click *Send*.



![image-20220611220246374](images/zuul/image-20220611220246374.png)



Wait a few moments for Zuul to process the event, and then reload the page.

The change should have been merged like below.

![image-20220611220259919](images/zuul/image-20220611220259919.png)



Visit [http://gerrit.kekehoho.com:8080/dashboard/self](http://localhost:8080/dashboard/self) (login ***hoho\***) and

return to the ***test1 (Add test Zuul job)\*** change you uploaded earlier.

Click *Reply* then

type *recheck* into the text field and

click *Send*.

![image-20220611220315316](images/zuul/image-20220611220315316.png)



![image-20220611220325170](images/zuul/image-20220611220325170.png)



This will cause Zuul to re-run the test job we created earlier.

This time it will run with the updated base job configuration, and when complete,

it will report the published log location as a comment on the change:

![image-20220611220336660](images/zuul/image-20220611220336660.png)



Follow the link and you will be directed to the build result page. If you click on the *Logs* tab, you’ll be able to browse the console log for the job. In the middle of the log, you should see the “Hello, world!” output from the job’s playbook.

Also try the *Console* tab for a more structured view of the log.

Click on the *OK* button in the middle of the page to see the output of just the task we’re interested in.

![image-20220611220348747](images/zuul/image-20220611220348747.png)

![image-20220611220356678](images/zuul/image-20220611220356678.png)

![image-20220611220403613](images/zuul/image-20220611220403613.png)

![image-20220611220416086](images/zuul/image-20220611220416086.png)



\#####

https://opendev.org/zuul/nodepool/raw/branch/master/.zuul.yaml

https://opendev.org/zuul/nodepool/src/branch/master/.zuul.yaml

https://github.com/ansible/zuul-config





**#####**

**#####**

**targetnode-01.kekehoho.com 노드에 실행시켜 보기**

\##### 아래 파이프라인에 check, gate 수행이니 CI 만 수행함 = CI NODE 설정이라 보면됨



root@zuul:~/zuul-config# cat zuul.d/pipelines.yaml 

\- pipeline:

  name: check

  description: |

   Newly uploaded patchsets enter this pipeline to receive an

   initial +/-1 Verified vote.

  manager: independent

  require:

   gerrit.kekehoho.com:

​    open: True

​    current-patchset: True

  trigger:

   gerrit.kekehoho.com:

​    \- event: patchset-created

​    \- event: change-restored

​    \- event: comment-added

​     comment: (?i)^(Patch Set [0-9]+:)?( [\w\\+-]*)*(\n\n)?\s*recheck

  success:

   gerrit.kekehoho.com:

​    Verified: 1

  failure:

   gerrit.kekehoho.com:

​    Verified: -1



\- pipeline:

  name: gate

  description: |

   Changes that have been approved are enqueued in order in this

   pipeline, and if they pass tests, will be merged.

  manager: dependent

  post-review: True

  require:

   gerrit.kekehoho.com:

​    open: True

​    current-patchset: True

​    approval:

​     \- Workflow: 1

  trigger:

   gerrit.kekehoho.com:

​    \- event: comment-added

​     approval:

​      \- Workflow: 1

  start:

   gerrit.kekehoho.com:

​    Verified: 0

  success:

   gerrit.kekehoho.com:

​    Verified: 2

​    submit: true

  failure:

   gerrit.kekehoho.com:

​    Verified: -2



\##

LABELFORTHENODE=another-node-ci-test

THENODEHOSTNAME=targetnode-01.kekehoho.com



sed -i "/^labels:/a \ \ -\ name:\ ${LABELFORTHENODE}" ~/zuul/doc/source/examples/etc_nodepool/nodepool.yaml

sed -i "/\ \ \ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ \ \ \ \ username:\ root" ~/zuul/doc/source/examples/etc_nodepool/nodepool.yaml

sed -i "/\ \ \ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ \ \ \ \ python-path:\ \/usr\/bin\/python3" ~/zuul/doc/source/examples/etc_nodepool/nodepool.yaml



\## root@zuul:~/zuul/doc/source/examples# ssh-keyscan -t ed25519 targetnode-01.kekehoho.com

\## # targetnode-01.kekehoho.com:22 SSH-2.0-OpenSSH_8.4p1 Ubuntu-6ubuntu2

\## targetnode-01.kekehoho.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILAKVEliWt+n5qnqni6z/TrrMubyfvDbQDLAsG0OoEMm



sed -i "/\ \ \ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ \ \ \ \ host-key:\ \"ssh-ed25519\ AAAAC3NzaC1lZDI1NTE5AAAAILAKVEliWt+n5qnqni6z/TrrMubyfvDbQDLAsG0OoEMm\"" ~/zuul/doc/source/examples/etc_nodepool/nodepool.yaml

sed -i "/\ \ \ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ \ \ \ \ labels:\ ${LABELFORTHENODE}" ~/zuul/doc/source/examples/etc_nodepool/nodepool.yaml

sed -i "/\ \ \ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ \ \ -\ name:\ ${THENODEHOSTNAME}" ~/zuul/doc/source/examples/etc_nodepool/nodepool.yaml







sed -i "/^\ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ \ \ label:\ ${LABELFORTHENODE}" ~/zuul-config/zuul.d/jobs.yaml

sed -i "/^\ \ \ \ \ \ nodes:/a \ \ \ \ \ \ \ \ -\ name:\ ${THENODEHOSTNAME}" ~/zuul-config/zuul.d/jobs.yaml





root@zuul:~/zuul/doc/source/examples# cat etc_nodepool/nodepool.yaml 

zookeeper-servers:

 \- host: zk

  port: 2281

zookeeper-tls:

 cert: /var/certs/certs/client.pem

 key: /var/certs/keys/clientkey.pem

 ca: /var/certs/certs/cacert.pem



labels:

 ***- name: another-node-ci-test\***

 \- name: ubuntu-focal



providers:

 \- name: static-vms

  driver: static

  pools:

   \- name: main

​    nodes:

​     ***- name: targetnode-01.kekehoho.com\***

​      ***labels: another-node-ci-test\***

​      ***host-key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILAKVEliWt+n5qnqni6z/TrrMubyfvDbQDLAsG0OoEMm"\***

​      ***python-path: /usr/bin/python3\***

​      ***username: root\***

​     \- name: testnode.kekehoho.com

​      labels: ubuntu-focal

​      host-key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOgHJYejINIKzUiuSJ2MN8uPc+dfFrZ9JH1hLWS8gI+g"

​      python-path: /usr/bin/python3

​      username: root







root@zuul:~/zuul-config# cat zuul.d/jobs.yaml 

\- job:

  name: base

  parent: null

  description: |

   The recommended base job.



   All jobs ultimately inherit from this. It runs a pre-playbook

   which copies all of the job's prepared git repos on to all of

   the nodes in the nodeset.



   It also sets a default timeout value (which may be overidden).

  pre-run: playbooks/base/pre.yaml

  post-run:

   \- playbooks/base/post-ssh.yaml

   \- playbooks/base/post-logs.yaml

  roles:

   \- zuul: onpremise-zuul/onpremise-zuul-jobs

  timeout: 1800

  nodeset:

   nodes:

​    ***- name: targetnode-01.kekehoho.com\***

​     ***label: another-node-ci-test\***

​    \- name: testnode.kekehoho.com

​     label: ubuntu-focal



![image-20220611220437254](images/zuul/image-20220611220437254.png)



**examples_node_1 의 root/.ssh/authorized_keys 키값을 targetnode-01 의 .ssh/authorized_keys 에 넣으면 task 가 실행되니 꼭 넣자. : auto-login 해줘야함.**

**아래 커맨드로 key 확인 가능**



curl http://admin:secret@gerrit.kekehoho.com:8080/a/accounts/zuul/sshkeyszuul/sshkeys



)]}'

[{"seq":1,"ssh_public_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCufjry+NmI+xXWIVa1Yz8QItE8g7B4R6bUE4NS0FOitw+tPb86AOZg4o6lZLounkao5oSBLUvKujl23CovJgufLIEQk2sOloRqkp1u3pU7hqNPkvpnBOwdln7u0LpZ5uzsO5RXdBllaMg+sNFsL2VeCLwUajini+EQjP/opZN39W7rIJzVOQy9z30dN4xNbeY4XOks45KaPO8QukRhvVL/6A/j6WlaWPsgZDbH/5EsjuPVVEPzxwKjyrnmTj8An60OhlIbBdKsqry95RRejg+0gIO+4GH0/Jg+aMtysvQ+9wJIKUxW0uyvtMLexachHqN8064+2ecEP57mWoTrY3DuZ9Zzg/g5aeAhu+z5JEz+BsZpoXno1VeYKETVp7w+WX2qgO2dD4Upgz6Nhq4pTGVwxz1iDiH8+JQaazr9aYKQ4Uiv7n2qNlwxN038BkCU68QNzopdeloBtUfArqazFmhXTMOu6UJT+adesZmj7o2WmObR465QZ7GiWr/8/3Raqtk\u003d root@172e40f2f06e","encoded_key":"AAAAB3NzaC1yc2EAAAADAQABAAABgQCufjry+NmI+xXWIVa1Yz8QItE8g7B4R6bUE4NS0FOitw+tPb86AOZg4o6lZLounkao5oSBLUvKujl23CovJgufLIEQk2sOloRqkp1u3pU7hqNPkvpnBOwdln7u0LpZ5uzsO5RXdBllaMg+sNFsL2VeCLwUajini+EQjP/opZN39W7rIJzVOQy9z30dN4xNbeY4XOks45KaPO8QukRhvVL/6A/j6WlaWPsgZDbH/5EsjuPVVEPzxwKjyrnmTj8An60OhlIbBdKsqry95RRejg+0gIO+4GH0/Jg+aMtysvQ+9wJIKUxW0uyvtMLexachHqN8064+2ecEP57mWoTrY3DuZ9Zzg/g5aeAhu+z5JEz+BsZpoXno1VeYKETVp7w+WX2qgO2dD4Upgz6Nhq4pTGVwxz1iDiH8+JQaazr9aYKQ4Uiv7n2qNlwxN038BkCU68QNzopdeloBtUfArqazFmhXTMOu6UJT+adesZmj7o2WmObR465QZ7GiWr/8/3Raqtk\u003d","algorithm":"ssh-rsa","comment":"root@172e40f2f06e","valid":true}]





초기 만드는 ansible 스크립트는 아래임.

root@zuul:~/zuul/doc/source/examples/playbooks# vim setup.yaml

...

...

  \- name: Generate Zuul SSH key for Nodepool

   command: ssh-keygen -f /var/ssh/nodepool -N '' -t rsa -m PEM

   args:

​    creates: /var/ssh/nodepool.pub

  \- name: Add Nodepool key to node authorized_keys file

   command: cp /var/ssh/nodepool.pub /var/node/authorized_keys

   args:

​    creates: /var/node/authorized_keys

...

...









cd ~/zuul-config

git add playbooks zuul.d/jobs.yaml

git commit -m "another node - Update Zuul base job"

git review



![image-20220611220449736](images/zuul/image-20220611220449736.png)

![image-20220611220458521](images/zuul/image-20220611220458521.png)

![image-20220611220506992](images/zuul/image-20220611220506992.png)



\# test1 job 실행해보자

![image-20220611220516723](images/zuul/image-20220611220516723.png)

![image-20220611220525209](images/zuul/image-20220611220525209.png)



cd ~/test1

touch a

git add *

git commit -m "another node - test - test1"

git review





\###### LOG 참고





**## jobs.yaml 에는 nodeset 으로 노드를 설정했지만 nodepool.yaml 노드 설정이 없을때 나는 error**

**##** 

\## LABEL : ***ubuntu-CD-targetnode\***

**scheduler_1**   | 2022-05-10 02:13:01,387 INFO ***zuul.nodepool\***: [e: 82082e5f53aa43d49a3d0ec12161df5d] Submitted node request <NodeRequest 200-0000000001 ['***ubuntu-CD-targetnode\***']>

**launcher_1**   | 2022-05-10 02:13:02,068 INFO **nodepool.PoolWorker.static-vms-main**: [e: 82082e5f53aa43d49a3d0ec12161df5d] [node_request: 200-0000000001] **Assigning node request** <NodeRequest {'state': 'requested', 'state_time': 1652148781.3839962, 'declined_by': [], 'node_types': ['ubuntu-CD-targetnode'], 'nodes': [], 'reuse': True, 'requestor': '8f18c45dad0944a78f1ff421c74df9aa', 'requestor_data': {'build_set_uuid': 'fbd18d2d1fdb41f0acb8dda04a042efc', 'tenant_name': 'example-tenant', 'pipeline_name': 'check', 'job_name': 'testjob'}, 'provider': None, 'relative_priority': 0, 'event_id': '82082e5f53aa43d49a3d0ec12161df5d', 'created_time': 1652148781.3840067, 'tenant_name': 'example-tenant', 'id': '200-0000000001', 'stat': ZnodeStat(czxid=34725, mzxid=34725, ctime=1652148781384, mtime=1652148781384, version=0, cversion=0, aversion=0, ephemeralOwner=0, dataLength=460, numChildren=0, pzxid=34725)}>

**launcher_1**   | 2022-05-10 02:13:02,069 INFO nodepool.driver.NodeRequestHandler[72769739a348-PoolWorker.static-vms-main-067c8feb5d964f92929b9b1aceba2751]: [e: 82082e5f53aa43d49a3d0ec12161df5d] [node_request: 200-0000000001] ***Declining node request because node type(s) [ubuntu-CD-targetnode] not available\***

**scheduler_1**   | 2022-05-10 02:13:02,108 INFO zuul.Pipeline.example-tenant.check: [e: 82082e5f53aa43d49a3d0ec12161df5d] ***Node request <NodeRequest 200-0000000001 ['ubuntu-CD-targetnode']>: failure for testjob\***

scheduler_1   | 2022-05-10 02:13:02,126 INFO zuul.Pipeline.example-tenant.check: [e: 82082e5f53aa43d49a3d0ec12161df5d] Completed node request <NodeRequest 200-0000000001 ['ubuntu-CD-targetnode']> for job testjob of item <QueueItem 144c61c168cf4e8f8925522585ec8edb for <Change 0x7f2cf061e370 test1 2,1> in check> with nodes []

scheduler_1   | 2022-05-10 02:13:02,133 INFO zuul.Pipeline.example-tenant.check: [e: 82082e5f53aa43d49a3d0ec12161df5d] Reporting item <QueueItem 144c61c168cf4e8f8925522585ec8edb for <Change 0x7f2cf061e370 test1 2,1> in check>, actions: [<zuul.driver.gerrit.gerritreporter.GerritReporter object at 0x7f2cf023bdf0>]



\# nodepool.yaml 예시

\# Uncomment to enable openstack provider

\#---

\#diskimages:

\# - name: dib-centos-7

\#  elements:

\#   - centos-minimal

\#   - nodepool-minimal

\#   - zuul-worker-user

\# - name: cloud-fedora-rawhide

\#  python-path: /usr/bin/python3

\#  dib-cmd: /usr/bin/dib-virt-customize /etc/nodepool/virt_images/cloud-fedora-rawhide.yaml

\#

\#

\#labels:

\# - name: dib-centos-7

\#  min-ready: 1

\# - name: cloud-fedora-rawhide

\#  min-ready: 1

\#

\#providers:

\# - name: default

\#  cloud: default

\#  clean-floating-ips: true

\#  image-name-format: '{image_name}-{timestamp}'

\#  boot-timeout: 120

\#  rate: 10.0

\#  diskimages:

\#   - name: dib-centos-7

\#   - name: cloud-fedora-rawhide

\#  pools:

\#   - name: main

\#    max-servers: 5

\#    networks:

\#     - worker-net-name

\#    labels:

\#     - name: dib-centos-7

\#      min-ram: 1024

\#      diskimage: dib-centos-7

\#     - name: cloud-fedora-rawhide

\#      min-ram: 1024

\#      diskimage: cloud-fedora-rawhide







\###### 

hostkey 파일이 틀리면 틀리다는 로그가 남 



***launcher_1\***   | 2022-05-10 02:52:40,128 WARNING nodepool.driver.static.StaticNodeProvider: Couldn't sync node: Node(hostname='targetnode-01.kekehoho.com', username='root', port=22): host key mismatches (['ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILAKVEliWt+n5qnqni6z/TrrMubyfvDbQDLAsG0OoEMm', 'ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKeYQ9usZ7txYbjlj6qMMCAAYfuAVPhVxybdOSdKW6WTTGL3BaR5+AB0OKGo3lFEyx076DFYXf5tDp7HCVo/j0w=', 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2IOYEQQE+ZEsselMfvbewH3xE2j4VQtULOBLT+wf3r6p1f5ijjSdeUS36/ad6iYDGme4NHB21g5qOXRJfJAE3Jknk4mS34EwheH/KHanh+R9hi6PufWDR+CZyKMVZ/3LrdTAaGGxRVpoFrEzwKXxKGchfUki2TpZXyjJ+MYZwVYypS0XFK7fS5dKhppCvC7UwOyO9JaISE87OJRv9x59ajep+G3jBGlS7XgkRN8WQNK2ZzBiUO+XsFy9HyMRq7tTd6NftLVR/Q+8wAbYgdg1JPpFoRF7qyGTmtlbWI9UlbUg2uSqPANFsFBbY6oFBXduXzRgKjmNARPp7G2htVY7NWooiN1O3aWzmFjIhokl5fCzryxrUxChWLuEs8uswqjEAgPHZbKobnw9nB1kBB6/sYyhOH6W5ei8pbBVyVWh4giGRPgaHGPnOlU7AirO0ari0nsG2r38JERx5xSuDcyJ2e16mXrYyT8re3y/K4OSFgi9soeZFCtY9xlGFTC5h8Xc=', 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2IOYEQQE+ZEsselMfvbewH3xE2j4VQtULOBLT+wf3r6p1f5ijjSdeUS36/ad6iYDGme4NHB21g5qOXRJfJAE3Jknk4mS34EwheH/KHanh+R9hi6PufWDR+CZyKMVZ/3LrdTAaGGxRVpoFrEzwKXxKGchfUki2TpZXyjJ+MYZwVYypS0XFK7fS5dKhppCvC7UwOyO9JaISE87OJRv9x59ajep+G3jBGlS7XgkRN8WQNK2ZzBiUO+XsFy9HyMRq7tTd6NftLVR/Q+8wAbYgdg1JPpFoRF7qyGTmtlbWI9UlbUg2uSqPANFsFBbY6oFBXduXzRgKjmNARPp7G2htVY7NWooiN1O3aWzmFjIhokl5fCzryxrUxChWLuEs8uswqjEAgPHZbKobnw9nB1kBB6/sYyhOH6W5ei8pbBVyVWh4giGRPgaHGPnOlU7AirO0ari0nsG2r38JERx5xSuDcyJ2e16mXrYyT8re3y/K4OSFgi9soeZFCtY9xlGFTC5h8Xc=', 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2IOYEQQE+ZEsselMfvbewH3xE2j4VQtULOBLT+wf3r6p1f5ijjSdeUS36/ad6iYDGme4NHB21g5qOXRJfJAE3Jknk4mS34EwheH/KHanh+R9hi6PufWDR+CZyKMVZ/3LrdTAaGGxRVpoFrEzwKXxKGchfUki2TpZXyjJ+MYZwVYypS0XFK7fS5dKhppCvC7UwOyO9JaISE87OJRv9x59ajep+G3jBGlS7XgkRN8WQNK2ZzBiUO+XsFy9HyMRq7tTd6NftLVR/Q+8wAbYgdg1JPpFoRF7qyGTmtlbWI9UlbUg2uSqPANFsFBbY6oFBXduXzRgKjmNARPp7G2htVY7NWooiN1O3aWzmFjIhokl5fCzryxrUxChWLuEs8uswqjEAgPHZbKobnw9nB1kBB6/sYyhOH6W5ei8pbBVyVWh4giGRPgaHGPnOlU7AirO0ari0nsG2r38JERx5xSuDcyJ2e16mXrYyT8re3y/K4OSFgi9soeZFCtY9xlGFTC5h8Xc='])







\### https://zuul-ci.org/docs/zuul/latest/howtos/nodepool_static.html

When setting up your nodepool.yaml file, you will ***need the host keys for each node for the host-key value.\***

This can be obtained with the command:



root@zuul:~/test1# ssh-keyscan -t ed25519 targetnode-01.kekehoho.com

\# targetnode-01.kekehoho.com:22 SSH-2.0-OpenSSH_8.4p1 Ubuntu-6ubuntu2

targetnode-01.kekehoho.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILAKVEliWt+n5qnqni6z/TrrMubyfvDbQDLAsG0OoEMm









root@zuul:~/zuul/doc/source/examples# cat etc_nodepool/nodepool.yaml 

zookeeper-servers:

 \- host: zk

  port: 2281

zookeeper-tls:

 cert: /var/certs/certs/client.pem

 key: /var/certs/keys/clientkey.pem

 ca: /var/certs/certs/cacert.pem



labels:

 \- name: ubuntu-focal

 ***- name: ubuntu-CD-targetnode\***



providers:

 \- name: static-vms

  driver: static

  pools:

   \- name: main

​    nodes:

​     \- name: testnode.kekehoho.com

​      labels: ubuntu-focal

​      host-key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOgHJYejINIKzUiuSJ2MN8uPc+dfFrZ9JH1hLWS8gI+g"

​      python-path: /usr/bin/python3

​      username: root

​     ***- name: targetnode-01.kekehoho.com\***

​      ***labels: ubuntu-CD-targetnode\***

​      ***host-key: "\***ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILAKVEliWt+n5qnqni6z/TrrMubyfvDbQDLAsG0OoEMm***"\***

​      ***python-path: /usr/bin/python3\***

​      ***username: root\***





***host key\* 파일을 정상적으로 변경하면 아래처럼 정상 작동이됨**



launcher_1  | 2022-05-10 04:05:17,994 INFO nodepool.PoolWorker.static-vms-main: [e: ec6705c258f74019bec7a6377c0ac509] [node_request: 200-0000000002] Assigning node request <NodeRequest {'state': 'requested', 'state_time': 1652152384.4049168, 'declined_by': [], 'node_types': ['ubuntu-CD-targetnode'], 'nodes': [], 'reuse': True, 'requestor': '8f18c45dad0944a78f1ff421c74df9aa', 'requestor_data': {'build_set_uuid': '11a3e9a165104f54aec526c3150800bd', 'tenant_name': 'example-tenant', 'pipeline_name': 'check', 'job_name': 'testjob'}, 'provider': None, 'relative_priority': 0, 'event_id': 'ec6705c258f74019bec7a6377c0ac509', 'created_time': 1652152384.4049296, 'tenant_name': 'example-tenant', 'id': '200-0000000002', 'stat': ZnodeStat(czxid=35955, mzxid=35955, ctime=1652152384405, mtime=1652152384405, version=0, cversion=0, aversion=0, ephemeralOwner=0, dataLength=460, numChildren=0, pzxid=35955)}>



scheduler_1 | 2022-05-10 04:05:18,083 INFO zuul.Pipeline.example-tenant.check: [e: ec6705c258f74019bec7a6377c0ac509] Completed node request <NodeRequest 200-0000000002 ['ubuntu-CD-targetnode']> for job testjob of item <QueueItem 47fd90fdc06940cf82a733c3c6f4d5fe for <Change 0x7f8ab462ec70 test1 21,1> in check> with nodes ['0000000002']

scheduler_1 | 2022-05-10 04:05:18,091 INFO zuul.ExecutorClient: [e: ec6705c258f74019bec7a6377c0ac509] Execute job <FrozenJob testjob> (uuid: 8871c7d75da64f4eb03e3801aafaa7c6) on nodes ['0000000002'] for change <Change 0x7f8ab462ec70 test1 21,1> with dependent changes [{'branch': 'master', 'change': '21', 'change_url': 'http://gerrit.kekehoho.com:8080/21', 'patchset': '1', 'project': {'canonical_hostname': 'gerrit.kekehoho.com', 'canonical_name': 'gerrit.kekehoho.com/test1', 'name': 'test1', 'short_name': 'test1', 'src_dir': 'src/gerrit.kekehoho.com/test1'}}]

executor_1  | 2022-05-10 04:05:18,175 INFO zuul.ExecutorServer: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Started SSH Agent, {'SSH_AUTH_SOCK': '/tmp/ssh-w8PVrgkBPob2/agent.1197', 'SSH_AGENT_PID': '1198'}

executor_1  | 2022-05-10 04:05:18,190 INFO zuul.ExecutorServer: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Added SSH Key /var/ssh/nodepool

executor_1  | 2022-05-10 04:05:18,203 INFO zuul.AnsibleJob: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Beginning job testjob for ref refs/changes/21/21/1 (change http://gerrit.kekehoho.com:8080/21)

executor_1  | 2022-05-10 04:05:18,204 INFO zuul.ExecutorServer: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Updating repo gerrit.kekehoho.com/test1

executor_1  | 2022-05-10 04:05:18,206 INFO zuul.ExecutorServer: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Updating repo gerrit.kekehoho.com/zuul-config

executor_1  | 2022-05-10 04:05:18,226 INFO zuul.Merger: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Skipping updating local repository gerrit.kekehoho.com/test1

executor_1  | 2022-05-10 04:05:18,246 INFO zuul.Merger: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Skipping updating local repository gerrit.kekehoho.com/zuul-config





executor_1  | 2022-05-10 04:05:19,305 INFO zuul.AnsibleJob: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Checking out gerrit.kekehoho.com/test1 zuul branch master

executor_1  | 2022-05-10 04:05:19,324 INFO zuul.nodepool: [e: ec6705c258f74019bec7a6377c0ac509] Setting nodeset <NodeSet [<Node 0000000002 ['targetnode-01.kekehoho.com']:ubuntu-CD-targetnode>]> in use

executor_1  | 2022-05-10 04:05:19,335 INFO zuul.AnsibleJob: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Checking out gerrit.kekehoho.com/zuul-config branch master

executor_1  | 2022-05-10 04:05:19,390 INFO zuul.AnsibleJob: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Checking out gerrit.kekehoho.com/test1 branch master

executor_1  | 2022-05-10 04:05:19,439 INFO zuul.AnsibleJob: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Checking out gerrit.kekehoho.com/zuul-config branch master

executor_1  | 2022-05-10 04:05:24,456 INFO zuul.ExecutorServer: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Held status set to False

executor_1  | 2022-05-10 04:05:24,485 INFO zuul.nodepool: [e: ec6705c258f74019bec7a6377c0ac509] Returning nodeset <NodeSet [<Node 0000000002 ['targetnode-01.kekehoho.com']:ubuntu-CD-targetnode>]>

executor_1  | 2022-05-10 04:05:24,495 INFO zuul.nodepool: [e: ec6705c258f74019bec7a6377c0ac509] Nodeset <NodeSet [<Node 0000000002 ['targetnode-01.kekehoho.com']:ubuntu-CD-targetnode>]> with 1 nodes was in use for 6.315216765999139 seconds for build <BuildRequest 8871c7d75da64f4eb03e3801aafaa7c6, job=testjob, state=completed, path=/zuul/executor/unzoned/requests/8871c7d75da64f4eb03e3801aafaa7c6 zone=None> for project gerrit.kekehoho.com/test1

executor_1  | 2022-05-10 04:05:24,495 INFO zuul.AnsibleJob: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Job execution took: 6.315 seconds

scheduler_1 | 2022-05-10 04:05:24,518 INFO zuul.Scheduler: [e: ec6705c258f74019bec7a6377c0ac509] [build: 8871c7d75da64f4eb03e3801aafaa7c6] Build complete, result None, warnings []

scheduler_1 | 2022-05-10 04:05:24,575 INFO zuul.nodepool: [e: ec6705c258f74019bec7a6377c0ac509] Submitted node request <NodeRequest 200-0000000003 ['ubuntu-CD-targetnode']>

launcher_1  | 2022-05-10 04:05:28,308 INFO nodepool.DeletedNodeWorker: Deleting used instance targetnode-01.kekehoho.com from static-vms

launcher_1  | 2022-05-10 04:05:28,311 INFO nodepool.NodeDeleter: Deleting ZK node id=0000000002, state=deleting, external_id=targetnode-01.kekehoho.com



















root@zuul:~/zuul/doc/source/examples# cat docker-compose.yaml 

version: '2.1'



services:

 gerrit:

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  image: docker.io/gerritcodereview/gerrit

  ports:

   \- "8080:8080"

   \- "29418:29418"

  environment:

   \- CANONICAL_WEB_URL=http://gerrit.kekehoho.com:8080/

  networks:

   \- zuul

***# gerritconfig:\***

***#  extra_hosts:\***

***#   - "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"\***

***#  image: docker.io/zuul/zuul-executor:latest\***

***#  environment:\***

***#   - http_proxy\***

***#   - https_proxy\***

***#   - no_proxy=${no_proxy},gerrit\***

***#  depends_on:\***

***#   - gerrit\***

***#  volumes:\***

***#   - "sshkey:/var/ssh:z"\***

***#   - "nodessh:/var/node:z"\***

***#   - "./playbooks/:/var/playbooks/:z"\***

***#   - "certs:/var/certs:z"\***

***#   - "../../../tools/:/var/zuul-tools/:z"\***

***#  # NOTE(pabelanger): Be sure to update this line each time we change the\***

***#  # default version of ansible for Zuul.\***

***#  command: "/usr/local/lib/zuul/ansible/2.9/bin/ansible-playbook /var/playbooks/setup.yaml"\***

***#  networks:\***

***#   - zuul\***

 zk:

  image: docker.io/zookeeper

  hostname: examples_zk_1.examples_default

  volumes:

   \- "./playbooks/:/var/playbooks/:z"

   \- "certs:/var/certs:z"

   \- "./zoo.cfg:/conf/zoo.cfg:z"

  command: "sh -c '/var/playbooks/wait-to-start-certs.sh && zkServer.sh start-foreground'"

  networks:

   \- zuul

 mysql:

  image: docker.io/mariadb

  environment:

   MYSQL_ROOT_PASSWORD: rootpassword

   MYSQL_DATABASE: zuul

   MYSQL_USER: zuul

   MYSQL_PASSWORD: secret

   \# Work around slow db startup when writing TZINFO data.

   MYSQL_INITDB_SKIP_TZINFO: 1

  networks:

   \- zuul

 scheduler:

  depends_on:

***#   - gerritconfig\***

   \- zk

   \- mysql

  environment:

   \- http_proxy

   \- https_proxy

   \- no_proxy=${no_proxy},gerrit

   \- ZUUL_MYSQL_PASSWORD=secret

  command: |

   sh -c '/var/playbooks/wait-to-start-certs.sh && \

​       /var/playbooks/wait-to-start.sh && \

​       zuul-scheduler -f'

  \# FIXME: The scheduler has no ansible anymore so use the executor image.

  \#    This needs to be changes such that ansible is not required for startup.

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  image: docker.io/zuul/zuul-scheduler:latest

  volumes:

   \- "${ZUUL_TUTORIAL_CONFIG:-./etc_zuul/}:/etc/zuul/:z"

   \- "./playbooks/:/var/playbooks/:z"

   \- "sshkey:/var/ssh:z"

   \- "certs:/var/certs:z"

  networks:

   \- zuul

 web:

  command: |

   sh -c '/var/playbooks/wait-to-start-certs.sh && \

​       zuul-web -f'

  depends_on:

   \- scheduler

   \- mysql

  ports:

   \- "9000:9000"

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  image: docker.io/zuul/zuul-web:latest

  environment:

   ZUUL_MYSQL_PASSWORD: secret

  volumes:

   \- "${ZUUL_TUTORIAL_CONFIG:-./etc_zuul/}:/etc/zuul/:z"

   \- "./playbooks/:/var/playbooks/:z"

   \- "certs:/var/certs:z"

  networks:

   \- zuul

 executor:

  privileged: true

  environment:

   \- http_proxy

   \- https_proxy

   \- no_proxy=${no_proxy},gerrit,scheduler

   \- ZUUL_MYSQL_PASSWORD=secret

  depends_on:

   \- scheduler

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  image: docker.io/zuul/zuul-executor:latest

  volumes:

   \- "${ZUUL_TUTORIAL_CONFIG:-./etc_zuul/}:/etc/zuul/:z"

   \- "./playbooks/:/var/playbooks/:z"

   \- "sshkey:/var/ssh:z"

   \- "logs:/srv/static/logs:z"

   \- "certs:/var/certs:z"

  command: "sh -c '/var/playbooks/wait-to-start-certs.sh && zuul-executor -f'"

  networks:

   \- zuul

 node:

  build:

   dockerfile: node-Dockerfile

   context: ./

   args:

​    http_proxy: "${http_proxy}"

​    https_proxy: "${http_proxy}"

​    no_proxy: "${no_proxy},gerrit"

  volumes:

   \- "nodessh:/root/.ssh:z"

  ports:

   \- "22:22"

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  networks:

   \- zuul

 launcher:

  depends_on:

   \- zk

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  image: docker.io/zuul/nodepool-launcher:latest

  volumes:

   \- "./playbooks/:/var/playbooks/:z"

   \- "./etc_nodepool/:/etc/nodepool/:z"

   \- "certs:/var/certs:z"

  ports:

   \- "8005:8005"

  command: "sh -c '/var/playbooks/wait-to-start-certs.sh && nodepool-launcher -f'"

  networks:

   \- zuul

 logs:

  build:

   dockerfile: logs-Dockerfile

   context: ./

   args:

​    http_proxy: "${http_proxy}"

​    https_proxy: "${http_proxy}"

​    no_proxy: "${no_proxy}"

  ports:

   \- "8000:80"

  extra_hosts:

   \- "gerrit.kekehoho.com zuul.kekehoho.com testnode.kekehoho.com zuullognode.kekehoho.com onpremisezuulroles.kekehoho.com:192.168.59.21"

  volumes:

   \- "logs:/usr/local/apache2/htdocs:z"

  networks:

   \- zuul



volumes:

 sshkey:

 nodessh:

 logs:

 certs:



networks:

 zuul:







docker-compose up





\######

\######

\### nodes



![image-20220611220548799](images/zuul/image-20220611220548799.png)



\###### nodes 은 어디에 정의하나??

nodepool : nodeset 을 사용하여 CI 용 nodes 정의

jobs : nodeset 을 사용하여 CD 용 nodes 정의







\######

root@zuul:~# tree zuul-config/

zuul-config/

├── playbooks

│  └── base ## 아래 task 들에서 필요한 기본 롤을 onpremise-zuul/onpremise-zuul-jobs 에 있는 roles 디렉토리에서 찾아서 수행

│    ├── post-logs.yaml

│    ├── post-ssh.yaml

│    └── pre.yaml

└── zuul.d

  ├── jobs.yaml

  ├── pipelines.yaml

  └── projects.yaml





root@zuul:~# cat zuul-config/zuul.d/jobs.yaml 

\- job:

  name: base ## 어떤 잡을 실행하던 기본으로 수행될 job 을 뜻함. base job

  parent: null

  description: |

   The recommended base job.



   All jobs ultimately inherit from this. It runs a pre-playbook

   which copies all of the job's prepared git repos on to all of

   the nodes in the nodeset.



   It also sets a default timeout value (which may be overidden).

  pre-run: playbooks/base/pre.yaml

  post-run:

   \- playbooks/base/post-ssh.yaml

   \- playbooks/base/post-logs.yaml

  roles:

   \- zuul: onpremise-zuul/onpremise-zuul-jobs ## <--- 수행 job 파일 이부분에 어떤 리포지토리를 참조할 건지 정의함. onpremise-zuul/onpremise-zuul-jobs 안의 roles 디렉토리 안에 필요한 role 들이 있음.

  timeout: 1800

  nodeset:

   nodes:

​    \- name: testnode.kekehoho.com

​     label: ubuntu-focal





root@zuul:~# tree test1

test1

├── playbooks

│  └── testjob.yaml

├── roles        # test1 프로젝트에서 수행할 커스텀 role 파일들을 넣으면됨.

└── zuul.yaml













\###### docker-compose restart 했을때..



docker-compose.yaml 에서

 gerritconfig: # <-- 이부분 실행이 되지 않게 해야한다. gerrit 기본 설정으로 돌아가지 못하게.



web_1      | 2022-05-09 06:41:48,233 WARNING zuul.ConfigLoader: 1 errors detected during example-tenant tenant configuration loading

web_1      | 2022-05-09 06:41:48,234 WARNING zuul.ConfigLoader: Zuul encountered an error while accessing the repo onpremise-

web_1      | 2022-05-09 06:41:48,234 WARNING zuul.ConfigLoader:  zuul/onpremise-zuul-jobs. The error was:

web_1      | 2022-05-09 06:41:48,234 WARNING zuul.ConfigLoader:  

web_1      | 2022-05-09 06:41:48,234 WARNING zuul.ConfigLoader:   Cmd('git') failed due to: exit code(128)

web_1      | 2022-05-09 06:41:48,234 WARNING zuul.ConfigLoader:    cmdline: git ls-remote --heads --tags http://onpremisezuulroles.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs

web_1      | 2022-05-09 06:41:48,234 WARNING zuul.ConfigLoader:    stderr: 'fatal: unable to access 'http://onpremisezuulroles.kekehoho.com:8080/onpremise-zuul/onpremise-zuul-jobs/': Recv failure: Connection reset by peer'





scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler: Error starting Zuul:

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:  Traceback (most recent call last):

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:   File "/usr/local/lib/python3.8/site-packages/zuul/cmd/scheduler.py", line 98, in run

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:    self.sched.prime(self.config)

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:   File "/usr/local/lib/python3.8/site-packages/zuul/scheduler.py", line 959, in prime

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:    tenant = loader.loadTenant(

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:   File "/usr/local/lib/python3.8/site-packages/zuul/configloader.py", line 2535, in loadTenant

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:    new_tenant = self.tenant_parser.fromYaml(

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:   File "/usr/local/lib/python3.8/site-packages/zuul/configloader.py", line 1639, in fromYaml

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:    self._cacheTenantYAML(abide, tenant, loading_errors, min_ltimes,

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:   File "/usr/local/lib/python3.8/site-packages/zuul/configloader.py", line 1910, in _cacheTenantYAML

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:    pb_ltime = min_ltimes[project.canonical_name][branch]

scheduler_1   | 2022-05-09 06:53:11,778 ERROR zuul.Scheduler:  KeyError: 'onpremisezuulroles.kekehoho.com/onpremise-zuul/onpremise-zuul-jobs'



web_1      | 2022-05-09 06:58:48,815 INFO zuul.TenantParser: Loading configuration from zuul-config/zuul.d/jobs.yaml@master

web_1      | 2022-05-09 06:58:48,818 INFO zuul.TenantParser: Loading configuration from zuul-config/zuul.d/pipelines.yaml@master

web_1      | 2022-05-09 06:58:48,821 INFO zuul.TenantParser: Loading configuration from zuul-config/zuul.d/projects.yaml@master

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer: Exception from WebServer:

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:  Traceback (most recent call last):

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/cmd/web.py", line 86, in run

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    self._run()

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/cmd/web.py", line 68, in _run

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    self.web.start()

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/web/__init__.py", line 1965, in start

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    self.updateLayout()

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/web/__init__.py", line 2111, in updateLayout

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    tenant = loader.loadTenant(

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/configloader.py", line 2535, in loadTenant

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    new_tenant = self.tenant_parser.fromYaml(

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/configloader.py", line 1639, in fromYaml

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    self._cacheTenantYAML(abide, tenant, loading_errors, min_ltimes,

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:   File "/usr/local/lib/python3.8/site-packages/zuul/configloader.py", line 1910, in _cacheTenantYAML

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:    pb_ltime = min_ltimes[project.canonical_name][branch]

web_1      | 2022-05-09 06:58:48,881 ERROR zuul.WebServer:  KeyError: 'onpremisezuulroles.kekehoho.com/onpremise-zuul/onpremise-zuul-jobs'



\####################





targetnode-01 에 examples_node_1 의 root/.ssh/authorized_keys 키를 넣으면 task 가 실행됨





root@zuul:~/test1# curl http://gerrit.kekehoho.com:8080/a/accounts/self/sshkeys

Unauthorizedroot@zucurl http://admin:secret@gerrit.kekehoho.com:8080/a/accounts/zuul/sshkeyszuul/sshkeys

)]}'

[{"seq":1,"ssh_public_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCufjry+NmI+xXWIVa1Yz8QItE8g7B4R6bUE4NS0FOitw+tPb86AOZg4o6lZLounkao5oSBLUvKujl23CovJgufLIEQk2sOloRqkp1u3pU7hqNPkvpnBOwdln7u0LpZ5uzsO5RXdBllaMg+sNFsL2VeCLwUajini+EQjP/opZN39W7rIJzVOQy9z30dN4xNbeY4XOks45KaPO8QukRhvVL/6A/j6WlaWPsgZDbH/5EsjuPVVEPzxwKjyrnmTj8An60OhlIbBdKsqry95RRejg+0gIO+4GH0/Jg+aMtysvQ+9wJIKUxW0uyvtMLexachHqN8064+2ecEP57mWoTrY3DuZ9Zzg/g5aeAhu+z5JEz+BsZpoXno1VeYKETVp7w+WX2qgO2dD4Upgz6Nhq4pTGVwxz1iDiH8+JQaazr9aYKQ4Uiv7n2qNlwxN038BkCU68QNzopdeloBtUfArqazFmhXTMOu6UJT+adesZmj7o2WmObR465QZ7GiWr/8/3Raqtk\u003d root@172e40f2f06e","encoded_key":"AAAAB3NzaC1yc2EAAAADAQABAAABgQCufjry+NmI+xXWIVa1Yz8QItE8g7B4R6bUE4NS0FOitw+tPb86AOZg4o6lZLounkao5oSBLUvKujl23CovJgufLIEQk2sOloRqkp1u3pU7hqNPkvpnBOwdln7u0LpZ5uzsO5RXdBllaMg+sNFsL2VeCLwUajini+EQjP/opZN39W7rIJzVOQy9z30dN4xNbeY4XOks45KaPO8QukRhvVL/6A/j6WlaWPsgZDbH/5EsjuPVVEPzxwKjyrnmTj8An60OhlIbBdKsqry95RRejg+0gIO+4GH0/Jg+aMtysvQ+9wJIKUxW0uyvtMLexachHqN8064+2ecEP57mWoTrY3DuZ9Zzg/g5aeAhu+z5JEz+BsZpoXno1VeYKETVp7w+WX2qgO2dD4Upgz6Nhq4pTGVwxz1iDiH8+JQaazr9aYKQ4Uiv7n2qNlwxN038BkCU68QNzopdeloBtUfArqazFmhXTMOu6UJT+adesZmj7o2WmObR465QZ7GiWr/8/3Raqtk\u003d","algorithm":"ssh-rsa","comment":"root@172e40f2f06e","valid":true}]





root@targetnode-01:~/.ssh# cat authorized_keys 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDCa4Vp0Nu0kY8P0mOzNpBRu4M+WZN8OCPRxYQ0OCYNWYw8p/jQLtsZKrB7wIVy6bh9s1lCC3uhjhJ3u+lVNXFekI9h2Ss1vuet1PiQ2jlIyte7FYVD7ypdVKXJp6B0PfWrWxTx+BZRagI5NJflaG6TBjmLZdJsWxBH49ORuW/NP8GmB61v6pP1QxmTDsw2MWvJ9w1GuC/9byatNhsw24eORct5fFy7NKjSqdi4LTBiQJs+S7OLCWR8JP7n9Mi2QV/5iqvtgL9P+a/+H6Awcb9EZcvkwL43icicpjhSQ3WuzdRW1snYZOxJXpjBXGlDNaLo4LjHr3QeIuKOfqWCCzATXcWQPsexqG5thB5vN9R/TL+cIUGgyUKnYLqk+m4UQT2pqW5XvsS40g89eVIAoRHi3kBhl3PBVXtaB5nj6qTPqlAc3VaMVgeWS9OfANr6WzL1V7MEGvvWz30JHgrxHEnJ7iHlehjvhU8hhI4tpCY5IruwWu305clnj5i14koLpcE= root@172e40f2f06e







root@zuul:~/zuul# docker exec -it examples_node_1 /bin/bash



root@368a1b5eee47:/# ssh -i root/.ssh/id_rsa root@192.168.59.30

The authenticity of host '192.168.59.30 (192.168.59.30)' can't be established.

ECDSA key fingerprint is SHA256:GBdEqLSirhvqBz3RFdyyRkLymVo3dbGfR00W21PncUA.

Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

Warning: Permanently added '192.168.59.30' (ECDSA) to the list of known hosts.

root@192.168.59.30's password: 



root@368a1b5eee47:/# ls root/.ssh/

authorized_keys id_rsa      id_rsa.pub    known_hosts



root@368a1b5eee47:/# cat root/.ssh/authorized_keys 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDCa4Vp0Nu0kY8P0mOzNpBRu4M+WZN8OCPRxYQ0OCYNWYw8p/jQLtsZKrB7wIVy6bh9s1lCC3uhjhJ3u+lVNXFekI9h2Ss1vuet1PiQ2jlIyte7FYVD7ypdVKXJp6B0PfWrWxTx+BZRagI5NJflaG6TBjmLZdJsWxBH49ORuW/NP8GmB61v6pP1QxmTDsw2MWvJ9w1GuC/9byatNhsw24eORct5fFy7NKjSqdi4LTBiQJs+S7OLCWR8JP7n9Mi2QV/5iqvtgL9P+a/+H6Awcb9EZcvkwL43icicpjhSQ3WuzdRW1snYZOxJXpjBXGlDNaLo4LjHr3QeIuKOfqWCCzATXcWQPsexqG5thB5vN9R/TL+cIUGgyUKnYLqk+m4UQT2pqW5XvsS40g89eVIAoRHi3kBhl3PBVXtaB5nj6qTPqlAc3VaMVgeWS9OfANr6WzL1V7MEGvvWz30JHgrxHEnJ7iHlehjvhU8hhI4tpCY5IruwWu305clnj5i14koLpcE= root@172e40f2f06e







\## 아래 pub 키를 targetnode-01 -> .ssh/authorized_keys 에 복사하면 pri key 사용하여 로그인 가능함. 

root@368a1b5eee47:/# cat root/.ssh/id_rsa.pub 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCkUWHzPpyhUW9pmdf93jGB1wM+yC5Mk4spMvV9ABCfP1/uKZV/ywee4gDG+Hk4INIEmwsLTkeWiFfmPRA1tjQr8eUYyCDFE52CVvfKACXJQU3ym3GaX2r6xiW+JHsvucCMkHJG3qoLj9u69rLykdfJRT1RkbQUqgX+VkZExBmY9FVvNWyFt+LfoPsxcDpAx9DBmLpQpOUZN6V3r2ccO3Vb8FTc0bK0l5CT5/kcwphTObhxvSVBLFOMAaPMA1nzpZUuLSQvjbNiQIdir2k+YJgRUyqZZOYO9eEGFxV2CkXzooizkS56SC3GnJG1fTfyKsUJgM8QR5hu7aI5IEBm6euBis3jRq4vWF2cAwVdk6vCEOT1x+ObkUnkvhV2BhS1QuVUL8sz8QqScNqbFxqaGgl1MxQ36WfdJSd4TisfmlVUQJkUtwkEnuCrydnnhqJKxR+L+o3PFGvsOp/RIjo2/vC/FA6u2rwJsmDxLzAIevHMzBaUmSJ85kzCd37d1+akyiM= zuul-build-sshkey



root@368a1b5eee47:/# ssh -i root/.ssh/id_rsa root@192.168.59.30

Welcome to Ubuntu 21.10 (GNU/Linux 5.13.0-40-generic x86_64)



 \* Documentation: https://help.ubuntu.com

 \* Management:   https://landscape.canonical.com

 \* Support:    https://ubuntu.com/advantage



 System information as of Mon May 16 06:04:24 AM UTC 2022



 System load:  0.02        Users logged in:     1

 Usage of /home: 0.0% of 248.95GB  IPv4 address for docker0: 172.17.0.1

 Memory usage:  13%        IPv4 address for enp0s3: 10.0.2.15

 Swap usage:   0%         IPv4 address for enp0s8: 192.168.59.30

 Processes:   111





67 updates can be applied immediately.

11 of these updates are standard security updates.

To see these additional updates run: apt list --upgradable



Failed to connect to https://changelogs.ubuntu.com/meta-release. Check your Internet connection or proxy settings





Last login: Mon May 16 06:01:12 2022 from 192.168.59.21

root@targetnode-01:~# exit

logout

Connection to 192.168.59.30 closed.