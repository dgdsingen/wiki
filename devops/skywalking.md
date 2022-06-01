# SkyWalking

> Skywalking
>
> > https://skywalking.apache.org/docs/
> >
> > https://skywalking.apache.org/docs/main/latest/readme/
> >
> > https://skywalking.apache.org/docs/main/latest/en/setup/backend/backend-storage/
> >
> > [skywalking will stop collect unexpectedly when hundreds os service](https://github.com/apache/skywalking/issues/2521) 
> >
> > [Introduction to Skywalking UI](https://www.codetd.com/en/article/13054933) 
>
> Skywalking on K8s
>
> >  https://github.com/apache/skywalking-kubernetes
> >
> > https://github.com/apache/skywalking-kubernetes/blob/master/chart/skywalking/README.md
> >
> > https://amjadhussain3751.medium.com/step-by-step-detailed-guide-to-setup-apache-skywalking-on-kubernetes-8369e3d93242
>
> Skywalking Agents
>
> > https://skywalking.apache.org/docs/skywalking-java/latest/en/setup/service-agent/java-agent/readme/
>
> with other tools
>
> [Chaos Mesh + SkyWalking: Better Observability for Chaos Engineering](https://chaos-mesh.org/blog/better-observability-for-chaos-engineering/) 



```yaml
# /home/hip-dev/moni-devstg/install-skywalking/skywalking-kubernetes/chart/skywalking/files/conf.d/oap

rules:
  service_percent_rule:
    metrics-name: service_percent
    threshold: 1
    op: <
    period: 10
    count: 4
    only-as-condition: false

helm dep up skywalking
helm upgrade skywalking skywalking
```



# Deploy Skywalking Server on k8s

Cluster와 Worker Node를 준비한다.

```sh
# minikube로 테스트하는 경우, memory 부족이 발생할 수 있으므로 적절히 확보해둔다.
minikube config set cpus 4
minikube config set memory 6144

minikube start

minikube addons enable default-storageclass
minikube addons enable storage-provisioner
```

- skywalking oap + ui : 3 nodes (each node 8 CPU, 16 Gib Memory, 100Gib Storage)
- elastic search : 3 nodes (each node 8 CPU, 16 Gib Memory, 100Gib Storage)



minikube로 elasticsearch 구동시 이슈들 해결하기 위한 설정

```sh
git clone https://github.com/elastic/helm-charts.git
cd helm-charts

# elasticsearch/values.yaml
# or skywalking/values.yaml

## Permit co-located instances for solitary minikube virtual machines.
## antiAffinity: "hard"는 이미 Pod가 할당된 Node는 반드시 피해서 다른 Node에 Pod를 배포하는 방식이다.
## antiAffinity: "soft"는 이미 Pod가 할당된 Node더라도 다른 적당한 Node가 없다면 Pod가 배포된다.
## elasticsearch의 경우 보통 3개의 Pod으로 Clustering 하므로,
## antiAffinity: "hard"라면 Single Node에서 1개 Pod만 뜨고 나머지 2개는 할당되지 않아 할당된 1개 Pod에서 "master not discovered yet..." 에러가 난다.
antiAffinity: "soft"

## Shrink default JVM heap.
esJavaOpts: "-Xmx128m -Xms128m"

## Allocate smaller chunks of memory per pod.
resources:
  requests:
    cpu: "100m"
    memory: "512M"
  limits:
    cpu: "1000m"
    memory: "512M"

## Request smaller persistent volumes.
volumeClaimTemplate:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: "standard"
  resources:
    requests:
      storage: 100M

## ERROR: ... Transport SSL must be enabled if security is enabled. Please set [xpack.security.transport.ssl.enabled] to [true] or disable security by setting [xpack.security.enabled] to [false] 발생 시 설정
esConfig:
  elasticsearch.yml: |
    xpack.security.enabled: false
    xpack.security.transport.ssl.enabled: false
    xpack.security.http.ssl.enabled: false

## ElasticsearchException[failed to bind service]; nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes] 발생 시 설정
extraInitContainers:
 - name: file-permissions
   image: busybox
   command: ['chown', '-R', '1000:1000', '/usr/share/elasticsearch/']
   volumeMounts:
   - mountPath: /usr/share/elasticsearch/data
     name: elasticsearch-master
   securityContext:
     privileged: true
     runAsUser: 0
```



NodePort 사용할 경우 env 설정 (그러나 굳이 필요하지 않다. Service type을 LoadBalancer나 ClusterIP + NEG 등으로 사용하면 된다)

```sh
# Ports & Domains
elasticsearch : data 30930, http 30920(elasticsearch-skywalking.test.com)
oap : grpc 31180 (oap-grpc-skywalking.test.com), rest 31280 (oap-rest-skywalking.test.com)
ui : 30808 (ui-skywalking.test.com)

# skywalking/templates/oap-deployment.yaml
- name: SW_STORAGE_ES_CLUSTER_NODES
  value: "{{ .Values.customizing.elasticsearch.service.url.for.oap }}:{{ .Values.customizing.elasticsearch.service.nodeport.for.oap }}"
#{{- if .Values.elasticsearch.enabled }}
#          value: "{{ .Values.elasticsearch.clusterName }}-{{ .Values.elasticsearch.nodeGroup }}:{{ .Values.elasticsearch.httpPort }}"
#{{- else }}
#          value: "{{ .Values.elasticsearch.config.host }}:{{ .Values.elasticsearch.config.port.http }}"
#{{- end }}

OAP CONTAINER ENVIRONMENT
      JAVA_OPTS:                    -Xmx4g -Xms4g -Dmode=init
      SW_STORAGE:                   elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES:  elasticsearch-master:9200

chart/skywalking/templates/ui-deployment.yaml
UI CONTAINER ENVIRONMENT
      SW_OAP_ADDRESS:  http://test-skywalking-oap:12800

# skywalking/templates/ui-deployment.yaml
env:
- name: SW_OAP_ADDRESS
  #value: http://{{ template "skywalking.oap.fullname" . }}:{{ .Values.oap.ports.rest }}
  value: http://{{ .Values.customizing.oap.service.url.for.ui }}:{{ .Values.customizing.oap.service.nodeport.for.ui }}

# skywalking/templates/oap-svc.yaml
  ports:
  #{{- range $key, $value :=  .Values.oap.ports }}
  #- port: {{ $value }}
  #  name: {{ $key }}
  #{{- end }}
  - port: {{ .Values.oap.ports.rest }}
    targetPort: {{ .Values.oap.ports.rest }}
    nodePort: 31280
    name: rest
  - port: {{ .Values.oap.ports.grpc }}
    targetPort: {{ .Values.oap.ports.grpc }}
    name: grpc
    nodePort: 31180

# skywalking/values.yaml
customizing:
  oap:
    service:
      url:
        for:
          ui: oap-rest-skywalking.test.com
      nodeport:
        for:
          ui: 31280
  elasticsearch:
    service:
      url:
        for:
          oap: elasticsearch-skywalking.test.com
      nodeport:
        for:
          oap: 30920
```



Helm으로 Skywalking 설치

참고로 [ElasticSearch는 JVM Heap Size를 Total Memory의 50%를 넘기지 말 것을 권장](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size)하고 있다.

```sh
# helm chart 받아오기
helm repo add ${REPO} https://apache.jfrog.io/artifactory/skywalking-helm
helm repo add elastic https://helm.elastic.co

git clone https://github.com/apache/skywalking-kubernetes
cd skywalking-kubernetes/chart

# env 설정
export SKYWALKING_RELEASE_NAME=skywalking
export SKYWALKING_RELEASE_NAMESPACE=default
export REPO=skywalking

# helm chart 파일들 수정 후에는 설치 직전에 반드시 helm dep up 해주자.
helm dep up ${REPO}

# helm chart로 묶는 파일이 2Mb를 넘어가면 에러가 난다. jar 등의 파일들은 chart에 묶지 말고 initContainer에서 wget 등으로 받아오도록 하자.
# minikube로 테스트하는 경우, memory 부족이 발생할 수 있으므로 제한한다.
helm install "${SKYWALKING_RELEASE_NAME}" ${REPO} -n "${SKYWALKING_RELEASE_NAMESPACE}" \
--set oap.image.tag=8.9.1 \
--set oap.storageType=elasticsearch \
--set oap.javaOpts="-Xmx1g -Xms1g" \
--set oap.service.type=LoadBalancer \
--set elasticsearch.imageTag=7.16.2 \
--set elasticsearch.esJavaOpts="-Xmx1g -Xms1g" \
--set elasticsearch.service.type=ClusterIP \
--set elasticsearch.replicas=1 \
--set elasticsearch.minimumMasterNodes=1 \
--set elasticsearch.persistence.enabled=true \
--set ui.image.tag=8.9.1 \
--set ui.service.type=LoadBalancer \

# --set elasticsearch.esJavaOpts="-Xmx2g -Xms2g" 설정시 OOMKill 에러나는데, elasticesearch.resources.requests.memory=4Gi 를 주더라도 values.yaml에 정의된 2Gi로 설정되고 있어서 OOMKill이 나는 것이었다. values.yaml에서 4Gi로 늘려주니 에러 안남. 최대한 values.yaml에 값을 기재하고 --set은 테스트할때 쓰자.
# 또 자주 reinstall 하다보면 oap 서비스가 "Does not have minimum availability" 에러난다. 언제 자원이 확보되는 건지 확인이 필요하다. 텀을 두고 재설치하면 설치된다.
helm install "${SKYWALKING_RELEASE_NAME}" ${REPO} -n "${SKYWALKING_RELEASE_NAMESPACE}" \
--set oap.image.tag=8.9.1 \
--set oap.storageType=elasticsearch \
--set oap.service.type=LoadBalancer \
--set oap.resources.requests.cpu=1 \
--set oap.resources.requests.memory=4Gi \
--set oap.resources.limits.cpu=2 \
--set oap.resources.limits.memory=4Gi \
--set oap.javaOpts="-Xmx2048m -Xms2048m -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m" \
--set elasticsearch.imageTag=7.16.2 \
--set elasticesearch.resources.requests.cpu=1 \
--set elasticesearch.resources.requests.memory=4Gi \
--set elasticesearch.resources.limits.cpu=2 \
--set elasticesearch.resources.limits.memory=4Gi \
--set elasticsearch.esJavaOpts="-Xmx2048m -Xms2048m -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m" \
--set elasticsearch.service.type=ClusterIP \
--set elasticsearch.persistence.enabled=true \
--set ui.image.tag=8.9.1 \
--set ui.service.type=LoadBalancer \
--set ui.service.externalPort=8080 \
--set ui.service.internalPort=8080 \
--set oap.env.SW_OTEL_RECEIVER=default \ # for k8s prometheus metrics
--set oap.env.SW_OTEL_RECEIVER_ENABLED_OC_RULES="k8s-cluster\,k8s-service\,k8s-node" \ # for k8s prometheus metrics
--set oap.envoy.als.enabled=true \ # for k8s prometheus metrics
--set oap.env.SW_CORE_RECORD_DATA_TTL=
--set oap.env.SW_CORE_METRICS_DATA_TTL=
#--set elasticsearch.service.type=NodePort \ # NodePort 타입으로 사용하는 경우 On
#--set elasticsearch.service.nodePort=30920 \ # NodePort 타입으로 사용하는 경우 On
#--set oap.service.type=NodePort \ # NodePort 타입으로 사용하는 경우 On
#--set ui.service.type=NodePort \ # NodePort 타입으로 사용하는 경우 On
#--set ui.service.nodePort=30808 \ # NodePort 타입으로 사용하는 경우 On
#--set oap.env.SW_NAMESPACE=skywalking-dev # oap env 설정


# 설치 후 확인
watch kubectl get all -o wide

# 잘못 만들어진 경우 삭제
helm uninstall skywalking
kubectl delete pvc -l app=elasticsearch-master

# Service 타입이 ClusterIP라면 port-forward를 사용하자. Local에서 간편하게 사용하기 좋다.
kubectl port-forward svc/skywalking-ui 8080:8080

# Service 타입 변경: Service가 기본으로 ClusterIP 타입으로 생성된다. 그럼 Proxy 없이는 외부에서 호출을 할 수가 없으므로 기존 Service를 삭제하고
kubectl delete service/skywalking-ui

# 이렇게 하면 External LB가 생성된다.
kubectl expose deployment deployment.apps/skywalking-ui --type LoadBalancer --port 80 --target-port 8080

#아래와 같이 annotations를 넣어주면 Internal LB로 변경된다.
kubectl edit service/skywalking-oap
---
metadata:
  annotations:
    networking.gke.io/load-balancer-type: Internal
---

# Port 변경이 필요한 경우 수정
kubectl edit service/elasticsearch-master
  - name: transport
    nodePort: 30930 # 30152 -> 30930
    port: 9300
    protocol: TCP
    targetPort: 9300
```



설치 후 ElasticSearch Index 검증

```sh
kubectl port-forward service/elasticsearch-master 9200:9200 &

# ES의 모든 index를 보여준다.
curl http://localhost:9200/_cat/indices?v

# SkyWalking Index는 sw로 시작하며, 그 중 하나를 열어보면 데이터가 보인다.
curl http://localhost:9200/sw_service_relation_server_side-20211223/_search?pretty
```



# Config Network

```sh
# LB: Service type에 맞게 연결

# DNS
elasticsearch : data 30930, http 30920(elasticsearch-skywalking.test.com)
oap : grpc 31180 (oap-grpc-skywalking.test.com), rest 31280 (oap-rest-skywalking.test.com)
ui : 30808 (ui-skywalking.test.com)

# 방화벽
## 이건 자동 생성되는 것으로 보인다. Cluster 내 통신에 필요한 필수 요소라서 그런가봄
gke-test-master: 100.64.0.128/28 > tcp:443,10250 > gke-test-node
gke-test-all: 100.64.60.0/22 > ah,sctp,tcp,udp,icmp,esp > gke-test-node
gke-test-vms: 172.31.69.192/28 > icmp,tcp:1-65535,udp:1-65535 > gke-test-node
## 필요한 것들 수동으로 추가
172.30.1.0/24 # dev network
172.31.1.32/28 # dev subnet
100.64.31.0/24 # gke service network
100.64.32.0/22 # gke pod network
> all > gke-test-node
# 이건 Cluster 내 통신을 넘어서, 다른 곳으로부터 Metric을 받기 위해 네트워크를 확장해서 방화벽을 열어준다. 프로토콜은 all 말고 좀 조정해야 할 듯.

# 방화벽 적용
GKE tag: gke-test-node

# Test
curl http://ui-skywalking.test.com:30808
```



방화벽의 경우 VPN과 Subnet이 위치한 project로 가서, target을 GKE nodes에 맞추어 아래처럼 설정한다.

- Name : gke-test-from-kubectlvm-to-allnode
- Network : vpc-test
- Priority : 1000
- Direction of traffic : Ingress
- Action on match : Allow
- Targets : Specified target tags
- Target tags : gke-test-node
- Source filters : IPv4 ranges
- Source IPv4 ranges : 172.30.1.2/32
- Second source filter : None # Source tags 를 선택하면 Source tags 를 물어봄
- protocols and ports : Allow all



# Deploy Skywalking Java Agent

```yaml
# 테스트용 sw-java-agent-sidecar.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sw-java-agent-sidecar
  labels:
    app: sw-java-agent-sidecar
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sw-java-agent-sidecar
  template:
    metadata:
      labels:
        app: sw-java-agent-sidecar
    spec:
      volumes:
        - name: skywalking-agent
          emptyDir: { }

      initContainers:
        - name: agent-container
          image: apache/skywalking-java-agent:8.8.0-java8
          volumeMounts:
            - name: skywalking-agent
              mountPath: /agent
          command: [ /bin/sh ]
          args: [ -c, cp -R /skywalking/agent /agent/ ]

      containers:
        - name: app-container
          image: springio/gs-spring-boot-docker
          volumeMounts:
            - name: skywalking-agent
              mountPath: /skywalking
          env:
            - name: JAVA_TOOL_OPTIONS
              value: -javaagent:/skywalking/agent/skywalking-agent.jar=agent.namespace=default,agent.service_name=example-spring-boot-app,collector.backend_service=172.30.1.3:11800,plugin.jdbc.trace_sql_parameters=true,profile.active=true
```



```yaml
# 테스트 후 실제 배포용

apiVersion: apps/v1
kind: Deployment
metadata:
  name: <APP_NAME>-deployment-<VERSION>
  namespace: <NAMESPACE>
spec:
  replicas: <REPLICA_NUM>
  selector:
    matchLabels:
      app: <APP_NAME>-pod
      version: <POD_VERSION>
  template:
    metadata:
      labels:
        app: <APP_NAME>-pod
        version: "<VERSION>"
    spec:
      volumes:
        - name: skywalking-agent
          emptyDir: { }

      initContainers:
        - name: agent-container
          image: apache/skywalking-java-agent:8.8.0-java8
          volumeMounts:
            - name: skywalking-agent
              mountPath: /agent
          command: [ /bin/sh ]
          args: [ -c, cp -R /skywalking/agent /agent/ ]

      containers:
        - name: <APP_NAME>-pod
          image: "<REPO_URL>:<VERSION>"
          volumeMounts:
            - name: skywalking-agent
              mountPath: /skywalking
          env:
            - name: JAVA_TOOL_OPTIONS
              value: -javaagent:/skywalking/agent/skywalking-agent.jar=agent.namespace=default,agent.service_name=<APP_NAME>,collector.backend_service=oap-rest-skywalking.test.com:11800,plugin.jdbc.trace_sql_parameters=true,profile.active=true
```



```sh
kubectl apply -f skywalking-agent.yml
```



# Deploy kube-state-metrics for k8s cluster monitoring

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-state-metrics

wget https://skywalking.apache.org/docs/main/v8.9.1/en/setup/backend/otel-collector-config.yaml

# otel-collector-config.yaml 에 oap endpoint 입력
namespace: default # 사용할 namespace 입력
...
exporters:
  opencensus:
    endpoint: "skywalking-oap.skywalking.svc.cluster.local:11800" # oap endpoint 입력
    insecure: true

kubectl apply -f otel-collector-config.yaml

# 이후 oap 구동시 아래 env를 추가해준다.
--set oap.env.SW_OTEL_RECEIVER=default \ # for k8s prometheus metrics
--set oap.env.SW_OTEL_RECEIVER_ENABLED_OC_RULES="k8s-cluster\,k8s-service\,k8s-node" \ # for k8s prometheus metrics
--set oap.envoy.als.enabled=true \ # for k8s prometheus metrics
```


