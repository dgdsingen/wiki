# Concept

> [K8s Docs](https://kubernetes.io/ko/docs/) 
>
> [kubectl Cheatsheet en](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) 
>
> [kubectl Cheatsheet ko](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/) 
>
> [kubectl commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) 
>
> [K8s instance calculator](https://learnk8s.io/kubernetes-instance-calculator) 
>
> [metrics-server](https://github.com/kubernetes-sigs/metrics-server) 
>
> [kubernetes-dashboard](https://github.com/kubernetes/dashboard) 
>
> [K8s 안내서: 설치부터 배포까지](https://subicura.com/k8s/guide/) 



- Pod에서 Service는 `{service-name}.{namespace}.svc.cluster.local` 로 접근 가능하다.

# Distribution

## Minikube

> https://minikube.sigs.k8s.io/docs/



### Install

되도록 minikube driver를 docker가 아닌 vm 기반으로 하자. NodePort 타입의 Service를 만들어서 접속하려는 경우 local > node 부분에서 터널링을 해줘야 한다. 애초에 node를 vm으로 만들고 local에서 접속 가능한 ip를 부여하면 편하다.

> [Minikube drivers](https://minikube.sigs.k8s.io/docs/drivers/) : Linux는 kvm2, Mac은 hyperkit을 사용하자



**Install minikube** 

```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```



**minikube config** 

```sh
# linux
minikube config set driver kvm2

# mac
minikube config set driver hyperkit

# cpu, memory
minikube config set cpus 2
minikube config set memory 4096
```

`~/.zshrc` , `~/.bashrc` 설정

```sh
# minikube
if command -v minikube &> /dev/null; then
    alias mk="minikube"
fi

# mac
if [[ $(uname -a) == *"Darwin"* ]]; then
    # use minikube as a docker host
    eval $(minikube docker-env)
fi
```

Linux kvm2 설정

```sh
# kvm2 사용시 우선 가상화 지원 여부부터 확인
egrep -q 'vmx|svm' /proc/cpuinfo && echo yes || echo no

# install kvm
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils

# add user to group. 이후 reboot 필요
sudo adduser `id -un` libvirt
sudo adduser `id -un` kvm

# validation
virt-host-validate
```



**start minikube** 

```sh
# start control plane node
minikube start

# dashboard
minikube dashboard
```



### Example for nginx

```yaml
cat <<EOF | kubectl apply -f -
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - port: 80
EOF
```

배포된 서비스 확인

```sh
minikube service nginx
```



### Enable LoadBalancer

Cloud Provider가 아닌 이상 LoadBalancer를 사용하기는 어렵다. minikube에서는 현재 떠 있는 노드를 LoadBalancer로 설정해주는 addon이 있으므로 사용해본다.

```sh
minikube addons enable metallb
```

이후 ConfigMap을 설정해준다.

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 100.127.1.0/24
EOF
```

이제 Service type을 LoadBalancer로 지정해주면 minikube ip로 바로 접근 가능하다.

다만 100.127.1.0/24 는 바로 접근할 수가 없으니 아래와 같이 routing을 추가해주면 된다.

```sh
# linux
sudo ip addr add 100.127.1.0/24 dev lo

# mac
sudo route -n add 100.127.1.0/24 `minikube ip`
sudo route -n delete 100.127.1.0/24
netstat -nrf inet
```

> [Add or delete static routes Apple MacOS X](https://www.heelpbook.net/2016/add-or-delete-static-routes-apple-macos-x/) 



### Enable ingress

```sh
minikube addons enable ingress

# ingress-nginx pod 확인
kubectl get po -A
curl -I http://`minikube ip`/healthz
```



### Enable metrics-server

```sh
mk addons enable metrics-server
# 이후 kubectl top 사용 가능해짐
```



## MicroK8s

```sh
sudo snap install microk8s --classic

sudo usermod -a -G microk8s dgdsingen

sudo iptables -P FORWARD ACCEPT
sudo apt-get install -y iptables-persistent

sudo snap alias microk8s.kubectl kubectl
sudo snap unalias microk8s.kubectl kubectl

# start & stop
microk8s.start
microk8s.status --wait-ready
microk8s.stop

# dashboard
microk8s.enable dns dashboard registry

token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token

microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443
# https://localhost:10443 접속하면 dashboard 확인 가능

microk8s.disable
```



## Google 공식 버전

**Hostname 변경** 

```sh
sudo hostnamectl set-hostname master_node_1
```

**Swap off** 

```sh
sudo vi /etc/fstab # swap 부분 주석 처리
sudo swapoff -a
```

**NTP(Network Time Protocol) 설정** 

- 클러스터 내 모든 노드들이 NTP를 통해 시간 동기화가 되어야 한다. 노드 시간이 서로 다를 경우 [문제](https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/)가 될 수 있다고 함. 

```sh
sudo apt install ntp
sudo service ntp restart
sudo ntpq -p
```

**Install Docker** 

```sh
# apt가 HTTPS로 리포지터리를 사용하는 것을 허용하기 위한 패키지 설치
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2

# Docker 공식 GPG 키 추가:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -

# Docker apt 리포지터리 추가:
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Docker CE 설치
# sudo apt-get update && sudo apt-get install -y containerd.io=1.2.13-2 docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
sudo apt-get update && sudo apt-get install -y containerd.io docker-ce docker-ce-cli

# Docker 데몬 설정
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# /etc/systemd/system/docker.service.d 생성
sudo mkdir -p /etc/systemd/system/docker.service.d

# Docker 재시작 & 부팅시 systemd 실행 설정
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker
```

**Install K8s Utils** 

- [공식 문서](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) 

```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

**config Control-Plane** 

```sh
sudo kubeadm init --apiserver-advertise-address ${마스터 노드 접속 가능한 IP} --pod-network-cidr=${클러스터 내부적으로 사용할 네트워크 대역}
sudo kubeadm init --apiserver-advertise-address 192.168.1.59 --pod-network-cidr=192.168.2.0/24

mkdir -p \$HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config
sudo chown \$(id -u):$(id -g) $HOME/.kube/config
```

**config Worker Node** 

```sh
kubeadm join localhost:6443 --token ${TOKEN} --discovery-token-ca-cert-hash ${DISCOVERY_HASH}
```

**Install Overlay Network** 

```sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

**Master Node에서도 Pod 생성 가능하게** 

```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```

**명령어 축약/자동완성 등록** 

>  [공식 문서](https://kubernetes.io/ko/docs/tasks/tools/included/optional-kubectl-configs-zsh/) 

```sh
vi ~/.zshrc

# k8s
alias k=kubectl
complete -F __start_kubectl k
```

**Test** 

```sh
kubectl get nodes
```

**Restart** 

```sh
#!/bin/bash
  
MY_IP=`ifconfig enp2s0 | sed -n '2p' | awk '{print $2}'`

sudo kubeadm reset --force
sudo swapoff -a

rm -rf $HOME/.kube

sudo kubeadm init --apiserver-advertise-address=$MY_IP --pod-network-cidr=192.168.2.0/24

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

kubectl taint nodes --all node-role.kubernetes.io/master-
```



# Tools

> [helm](https://helm.sh/) : package manager
>
> [kustomize](https://kustomize.io/) : k8s native configuration management
>
> [Argo CD](https://argo-cd.readthedocs.io/) : Declarative GitOps CD for k8s
>
> > [Argo Workflows](https://argoproj.github.io/argo-workflows/) : open source container-native workflow engine for orchestrating parallel jobs on k8s
> >
> > [Argo Events](https://argoproj.github.io/argo-events/) : Event-driven Workflow Automation Framework
> >
> > [Argo Rollouts](https://argoproj.github.io/argo-rollouts/) : K8s Progressive Delivery Controller
>
> [Flux CD](https://fluxcd.io/) : GitOps for both apps and infrastructure
>
> > [Flagger](https://flagger.app/) : K8s Progressive Delivery Operator
>
> [Tekton](https://tekton.dev/) : Cloud Native CI/CD
>
> [Jenkins X](https://jenkins-x.io/) : CD on k8s
>
> [Kaniko](https://github.com/GoogleContainerTools/kaniko) : Build images in k8s
>
> [Istio](https://istio.io/) : Service mesh
>
> [Linkerd](https://linkerd.io/) : Service mesh
>
> [Crossplane](https://crossplane.io/) : Compose cloud infrastructure and services into custom platform APIs
>
> [Knative](https://knative.dev/) : Serverless Containers in Kubernetes environments
>
> [Kyverno](https://kyverno.io/) : K8s Native Policy Management
>
> [kubespray](https://github.com/kubernetes-sigs/kubespray) : Deploy a Production Ready K8s Cluster
>
> [Lens](https://k8slens.dev/) : K8s IDE
>
> [Stern](https://github.com/wercker/stern) : Multi pod and container log tailing for K8s
>
> [Kubespy](https://github.com/pulumi/kubespy) : observing K8s resources in real time
>
> [Pixie](https://px.dev/) : Open source Kubernetes observability for developers
>
> [Meshery](https://meshery.io/) : the extensible service mesh manager
>
> [kube-vip](https://kube-vip.chipzoller.dev/) : HA Load Balancer
>
> [KubeVela](https://kubevela.io/) : software delivery platform that makes deploying and operating applications across hybrid, multi-cloud environments easier, faster and more reliable
>
> [Service Mesh Performance](https://smp-spec.io/) : Standardizing Service Mesh Value Measurement
>
> > https://github.com/service-mesh-performance/service-mesh-performance
>
> [Snyk](https://snyk.io/) : Find and automatically fix vulnerabilities
>
> [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) : A K8s controller and tool for one-way encrypted Secrets



## k9s

> [k9s](https://github.com/derailed/k9s) 

Mac에서는 `brew install k9s` , Linux에서는 https://github.com/derailed/k9s/releases 에서 바이너리를 다운받아 사용하자.

그리고 반드시 `export LC_ALL=en_US.UTF-8` 를 먼저 설정해주고 k9s를 실행해야 UI가 깨지지 않는다.



## node-shell

Node에 root로 접속하고 싶은 경우 권한이 있는 container를 해당 Node에 띄워서 접속하면 되는데, 이를 자동화해주는 도구다.

> https://github.com/kvaps/kubectl-node-shell



## K8s + Prometheus + Grafana

우선 k8s에 prometheus부터 배포한다.

> https://github.com/prometheus-community/helm-charts
>
> https://github.com/kubernetes/kube-state-metrics/blob/master/docs/pod-metrics.md

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
```

이후 아래와 같은 svc가 생성된다. 이 중 prometheus-server를 LB, DNS에 등록하여 접속 잘 되면 OK.

```sh
# k get svc | grep prometheus
prometheus-alertmanager         ClusterIP   100.64.36.125   <none>        80/TCP     51m
prometheus-kube-state-metrics   ClusterIP   100.64.36.214   <none>        8080/TCP   51m
prometheus-node-exporter        ClusterIP   100.64.36.114   <none>        9100/TCP   51m
prometheus-pushgateway          ClusterIP   100.64.36.56    <none>        9091/TCP   51m
prometheus-server               ClusterIP   100.64.36.205   <none>        80/TCP     51m
```



Grafana를 실행한다. 초기 계정은 admin / admin 이다.

```sh
docker run -d -p 3000:3000 --name grafana grafana/grafana-oss
```

grafana에 접속해 Datasource로 Prometheus를 등록한다. Domain/IP는 위에서 등록한 값을 그대로 넣어준다.

"grafana kubernetes dashboard" 구글링해서 마음에 드는 것을 골라 Dashboard > Import 로 구성해준다.



### grafana(2) + mysql 로 HA 구성

```sh
helm repo add grafana https://grafana.github.io/helm-charts
git clone https://github.com/grafana/helm-charts.git
```

`helm-charts/charts/grafana/values.yaml` 을 아래와 같이 변경한다. Service type을 LoadBalancer로 사용하는 경우 minikube에서는 metallb 설정을 먼저 해준다.

```diff
-replicas: 1
+replicas: 2
 
 ## Create a headless service for the deployment
 headlessService: false
@@ -151,7 +151,7 @@ podPortName: grafana
 ##
 service:
   enabled: true
-  type: ClusterIP
+  type: LoadBalancer
   port: 80
   targetPort: 3000
     # targetPort: 4181 To be used with a proxy extraContainer
@@ -335,7 +335,7 @@ initChownData:
 
 # Administrator credentials when not using an existing secret (see below)
 adminUser: admin
-# adminPassword: strongpassword
+adminPassword: admin
 
 # Use an existing secret for the admin user.
 admin:
@@ -587,6 +587,12 @@ grafana.ini:
     mode: console
   grafana_net:
     url: https://grafana.net
+  database:
+    type: mysql
+    host: mysql.default.svc.cluster.local
+    name: grafana
+    user: root
+    password: 1212
```

```sh
# mysql을 먼저 설치한다.
cd $ROOT_DIR
kubectl apply -f mysql.yml

# chart 내용을 변경한 경우 반드시 update를 한번 해주자
cd helm-charts/charts
helm dep up grafana

# grafana를 설치한다.
helm install grafana grafana

# 내용 변경시 update
helm upgrade grafana grafana

# 삭제
helm uninstall grafana
```

이제 어느 grafana pod에 접속하더라도 mysql에 의해 데이터가 공유된다.



### Add mysql-exporter

```sh
git clone https://github.com/prometheus-community/helm-charts.git
```

`helm-charts/charts/prometheus-mysql-exporter/values.yaml` 을 아래와 같이 변경한다.

```diff
 mysql:
-  db: ""
-  host: "localhost"
+  db: "mysql"
+  host: "mysql"
   param: ""
-  pass: "password"
+  pass: "1212"
   port: 3306
   protocol: ""
-  user: "exporter"
+  user: "root"
```

이후 grafana에서 datasource는 그대로 prometheus로 두고, mysql dashboard만 추가해보면 mysql server metric이 잘 보이는 것을 확인할 수 있다.



# Issues

**완료된 CronJob Pod 삭제하기** 

1. [TTL mechanism for finished Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically) 를 설정한다.

이 방법은 해보니 잘 안됨. alpha 기능은 feature gate에 의해 막힌 경우가 있는데 수동으로 풀어줘야 하는 듯.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: test
spec:
  ttlSecondsAfterFinished: 100
```

2. [Jobs History Limits](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#jobs-history-limits) 를 설정한다.

아래와 같이 설정하면 cronjob 실행 후 완료된 pod가 바로 삭제된다. 설정해보니 확실하게 잘 됨.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: test
spec:
  successfulJobsHistoryLimit: 0
  failedJobsHistoryLimit: 0
```

3. 수동으로 삭제한다.

```sh
kubectl get pod --field-selector=status.phase==Succeeded
kubectl delete pod --field-selector=status.phase==Succeeded
```

4. 수동 삭제 cronjob을 걸어준다.

> [How to automatically remove completed Kubernetes Jobs created by a CronJob?](https://stackoverflow.com/questions/41385403/how-to-automatically-remove-completed-kubernetes-jobs-created-by-a-cronjob) 



# CKA (Certified Kubernetes Administrator)

> https://kubernetes.io/ko/training/
>
> https://lifeoncloud.kr/k8s/cka/

