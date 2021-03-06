# Prometheus

## Prometheus + Grafana

> https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion/ : 시계열 데이터로 Prometheus 메모리 사용량 계산하기



우선 k8s에 prometheus부터 배포한다.

> https://github.com/prometheus-community/helm-charts
>
> https://github.com/kubernetes/kube-state-metrics/blob/master/docs/pod-metrics.md

```sh
git clone https://github.com/prometheus-community/helm-charts.git prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus -n prometheus
```

이후 아래와 같은 service가 생성된다. 이 중 prometheus-server를 LB, DNS에 등록하여 접속 잘 되면 prometheus는 준비되었다.

```sh
# k get svc -n prometheus
prometheus-alertmanager         ClusterIP   100.64.36.125   <none>        80/TCP     51m
prometheus-kube-state-metrics   ClusterIP   100.64.36.214   <none>        8080/TCP   51m
prometheus-node-exporter        ClusterIP   100.64.36.114   <none>        9100/TCP   51m
prometheus-pushgateway          ClusterIP   100.64.36.56    <none>        9091/TCP   51m
prometheus-server               ClusterIP   100.64.36.205   <none>        80/TCP     51m
```



grafana를 docker로 띄워서 테스트해본다. 초기 계정은 admin / admin 이다.

```sh
docker run -d -p 3000:3000 --name grafana grafana/grafana-oss
```

grafana에 접속해 Datasource로 Prometheus를 등록한다. Domain/IP는 위에서 등록한 값을 그대로 넣어준다.

"grafana kubernetes dashboard" 구글링해서 마음에 드는 것을 골라 Dashboard > Import 로 구성해준다.



운영 환경에 Prometheus 배포시에는 CPU, Memory, Storage Size, Retention Time/Size 를 수정해야할 수 있다.

CPU는 벤치마킹 결과를 보니 1 Core로 초당 최대 100k metric 처리 가능하다고 하며

Memory, Storage Size 는 주기적으로 발생하는 Compaction과 Index data 등을 고려해서 넉넉하게 잡아준다.

그리고 Retention Time은 기본 15d, Size는 0(무제한)으로 되어 있는데 Size를 Storage의 1/4 정도로 잡아줘보자. (Compation + Index data + Retention 주기 사이에 늘어날 용량 고려)

또한 scrape_interval도 기본 1m 으로 되어 있는데 metric 변화량을 더 자세히 보고 싶다면 더 작게 조정한다.

`helm-charts/charts/prometheus/values.yaml` 을 아래와 같이 변경한다. 

```diff
     ## alertmanager data Persistent Volume size
     ##
-    size: 2Gi
+    size: 10Gi

   ## alertmanager resource requests and limits
   ## Ref: http://kubernetes.io/docs/user-guide/compute-resources/
   ##
-  resources: {}
-    # limits:
-    #   cpu: 10m
-    #   memory: 32Mi
-    # requests:
-    #   cpu: 10m
-    #   memory: 32Mi
+  resources: 
+    limits:
+      cpu: 200m
+      memory: 128Mi
+    requests:
+      cpu: 200m
+      memory: 128Mi
 
   ## node-exporter resource limits & requests
   ## Ref: https://kubernetes.io/docs/user-guide/compute-resources/
   ##
-  resources: {}
-    # limits:
-    #   cpu: 200m
-    #   memory: 50Mi
-    # requests:
-    #   cpu: 100m
-    #   memory: 30Mi
+  resources: 
+    limits:
+      cpu: 200m
+      memory: 128Mi
+    requests:
+      cpu: 200m
+      memory: 128Mi

   global:
     ## How frequently to scrape targets by default
     ##
-    scrape_interval: 1m
+    scrape_interval: 15s 

     ## Prometheus server data Persistent Volume Storage Class
     ## If defined, storageClassName: <storageClass>                                                                                                                                                                    
     ## If set to "-", storageClassName: "", which disables dynamic provisioning
     ## If undefined (the default) or set to null, no storageClassName spec is
     ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
     ##   GKE, AWS & OpenStack)
     ##
-    # storageClass: "-"
+    storageClass: "standard-rwo"
 
   ## Additional Prometheus server container arguments
   ##
-  extraArgs: {}
+  extraArgs:
+    'storage.tsdb.retention.size': "50GB"
 
     ## Prometheus server data Persistent Volume size
     ##
-    size: 8Gi
+    size: 100Gi

   ## Prometheus server resource requests and limits
   ## Ref: http://kubernetes.io/docs/user-guide/compute-resources/
   ##
-  resources: {}
-    # limits:
-    #   cpu: 500m
-    #   memory: 512Mi
-    # requests:
-    #   cpu: 500m
-    #   memory: 512Mi
+  resources:
+    limits:
+      cpu: 1000m
+      memory: 8Gi
+    requests:
+      cpu: 1000m
+      memory: 8Gi

   ## Prometheus data retention period (default if not specified is 15 days)
   ##
-  retention: "15d"
+  retention: "30d"

   ## pushgateway resource requests and limits
   ## Ref: http://kubernetes.io/docs/user-guide/compute-resources/
   ##
-  resources: {}
-    # limits:
-    #   cpu: 10m
-    #   memory: 32Mi
-    # requests:
-    #   cpu: 10m
-    #   memory: 32Mi
+  resources: 
+    limits:
+      cpu: 200m
+      memory: 128Mi
+    requests:
+      cpu: 200m
+      memory: 128Mi
 
     ## pushgateway data Persistent Volume size
     ##
-    size: 2Gi
+    size: 10Gi

  # federation이 필요한 경우 아래와 같이 추가
  prometheus.yml:
    scrape_configs:
+      - job_name: "federate"
+        honor_labels: true
+        metrics_path: "/federate"
+        params:
+          'match[]':
+            - '{job=~".*"}'
+            - '{__name__=~"job:.*"}'
+        static_configs:
+          - targets:
+            - "child-prometheus:9090"
```

helm 설치

```sh
# chart 내용을 변경한 경우 반드시 update를 한번 해주자
cd charts
helm dep up prometheus

# prometheus를 설치한다.
helm install prometheus prometheus -n prometheus

# 내용 변경시 update
helm upgrade prometheus prometheus -n prometheus

# 삭제
helm uninstall prometheus -n prometheus
```



이후 `helm upgrade prometheus prometheus` 시 `level=error ts=2021-10-31T17:26:25.884Z caller=main.go:917 err="opening storage failed: lock DB directory: resource temporarily unavailable"` 와 같은 에러가 발생할 수 있다. 이 때는 `kubectl scale deployment prometheus-server --replicas=0` 로 기존 pod를 종료시킨 뒤에 `kubectl scale deployment prometheus-server --replicas=1` 로 다시 pod를 되살려준다.



### grafana(2) + mysql 로 HA 구성

```sh
git clone https://github.com/grafana/helm-charts.git grafana

helm repo add grafana https://grafana.github.io/helm-charts

helm install grafana grafana
```

`helm-charts/charts/grafana/values.yaml` 을 아래와 같이 변경한다. Service type을 LoadBalancer로 사용하는 경우 minikube에서는 metallb 설정을 먼저 해준다.

```diff
-replicas: 1
+replicas: 2
 
-resources: {}
-#  limits:
-#    cpu: 100m
-#    memory: 128Mi
-#  requests:
-#    cpu: 100m
-#    memory: 128Mi
+resources:
+  limits:
+    cpu: 500m
+    memory: 1Gi
+  requests:
+    cpu: 500m
+    memory: 1Gi

service:
   enabled: true
-  type: ClusterIP
+  type: LoadBalancer
   port: 80
   targetPort: 3000

adminUser: admin
-# adminPassword: strongpassword
+adminPassword: admin

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

mysql을 설치한다.

```sh
cd $ROOT_DIR

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  type: LoadBalancer
  ports:
    - port: 3306
  selector:
    app: mysql
    tier: mysql
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
type: Opaque
data:
  # echo -n 1212 | base64
  password: MTIxMg==
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  selector:
    matchLabels:
      app: mysql
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
        tier: mysql
    spec:
      containers:
      - image: mysql:8.0.26
        name: mysql
        env:
        - name: MYSQL_DATABASE
          value: grafana
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql
EOF

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
-resources: {}
+resources: 
+  limits:
+    cpu: 200m
+    memory: 128Mi
+  requests:
+    cpu: 200m
+    memory: 128Mi

mysql:
-  db: ""
-  host: "localhost"
+  db: "grafana"
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



## HA 구성

### Federation

> https://prometheus.io/docs/prometheus/latest/federation/



Prometheus는 Clustering을 지원하지 않는다. 대신 Federation을 지원한다.

아래와 같이 각 영역을 담당하는 Child Prometheus가 있고, Parent Prometheus가 Child Prometheus들로부터 Scrape하는 방식이다.

```mermaid
graph
A[Parent Prometheus] -->|Scrape| B(Child Prometheus 1)
A[Parent Prometheus] -->|Scrape| C(Child Prometheus 2)
```



```diff
  # federation이 필요한 경우 아래와 같이 추가
  prometheus.yml:
    scrape_configs:
+      - job_name: "federate"
+        honor_labels: true
+        metrics_path: "/federate"
+        params:
+          'match[]':
+            - '{job=~".*"}'
+            - '{__name__=~"job:.*"}'
+        static_configs:
+          - targets:
+            - "child-prometheus:9090"
```

- Parent Grafana의 Datasource: Parent Prometheus
- Child Grafana의 Datasource: Child Prometheus
- Parent Prometheus => Child Prometheus 로 Scrape 하는 Federation 구조라고 가정해보자.

Child Grafana의 Dashboard를 Parent Grafana로 Migration 할 때, JSON Model을 그대로 복붙하면 Datasource UID가 틀리다고 에러가 난다.

Child Grafana Dashboard의 JSON Model을 일단 그대로 복사한 뒤, Datasource UID 부분만 Parent Grafana의 Parent Prometheus의 값으로 Replace하면 된다.



### Thanos + Prometheus + Grafana

Thanos를 Sidecar로 붙혀서 Prometheus HA 구성을 한다. kube-prometheus-stack에서 해당 구성을 통으로 제공한다.

> https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
>
> https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/thanos.md
>
> https://github.com/thanos-io/thanos/blob/main/docs/storage.md
>
> https://www.infracloud.io/blogs/thanos-ha-scalable-prometheus/
>
> https://thanos.io/tip/thanos/quick-tutorial.md/
>
> https://thanos.io/tip/thanos/storage.md/



# Grafana

## Dashboard

### Sync

> https://github.com/mpostument/grafana-sync

dev, stg, prd 등 여러 환경의 grafana를 sync해야 하는 경우에 사용한다. 근데 실제 써보니 패널의 value type이 안맞는 등 다소 이슈가 있어 완벽하게 sync 되지는 않는다.

우선 API key가 필요한데 grafana > configuration > API keys > Add API key > Role: Admin 으로 생성한다.

```sh
# datasources: prd > stg, dev
./grafana-sync.exe pull-datasources --url http://grafana.prd.test.com --directory="datasources" --apikey="apikey-dev"
./grafana-sync.exe push-datasources --url http://grafana.stg.test.com --directory="datasources" --apikey="apikey-stg"
./grafana-sync.exe push-datasources --url http://grafana.dev.test.com --directory="datasources" --apikey="apikey-prd"

# folders: prd > stg, dev
./grafana-sync.exe pull-folders --url http://grafana.prd.test.com --directory="folders" --apikey="apikey-dev"
./grafana-sync.exe push-folders --url http://grafana.stg.test.com --directory="folders" --apikey="apikey-stg"
./grafana-sync.exe push-folders --url http://grafana.dev.test.com --directory="folders" --apikey="apikey-prd"

# dashboards: prd > stg, dev
./grafana-sync.exe pull-dashboards --url http://grafana.prd.test.com --directory="dashboards" --folderName="K8s" --apikey="apikey-dev"
./grafana-sync.exe push-dashboards --url http://grafana.stg.test.com --directory="dashboards" --folderName="K8s" --apikey="apikey-stg"
./grafana-sync.exe push-dashboards --url http://grafana.dev.test.com --directory="dashboards" --folderName="K8s" --apikey="apikey-prd"
rm -r dashboards

./grafana-sync.exe pull-dashboards --url http://grafana.prd.test.com --directory="dashboards" --folderName="Prometheus" --apikey="apikey-dev"
./grafana-sync.exe push-dashboards --url http://grafana.stg.test.com --directory="dashboards" --folderName="Prometheus" --apikey="apikey-stg"
./grafana-sync.exe push-dashboards --url http://grafana.dev.test.com --directory="dashboards" --folderName="Prometheus" --apikey="apikey-prd"
rm -r dashboards

./grafana-sync.exe pull-dashboards --url http://grafana.prd.test.com --directory="dashboards" --folderName="SQL" --apikey="apikey-dev"
./grafana-sync.exe push-dashboards --url http://grafana.stg.test.com --directory="dashboards" --folderName="SQL" --apikey="apikey-stg"
./grafana-sync.exe push-dashboards --url http://grafana.dev.test.com --directory="dashboards" --folderName="SQL" --apikey="apikey-prd"
rm -r dashboards
```



## Datasource: Prometheus

### Variables

Dashboard 맨 위에 Variables가 selectbox로 노출된다. Variables 중 node를 node-1로 선택하면 이후 Query에서 "$node"의 값으로 "node-1" 매핑된다.



#### k8s nodes

- Name: node
- Type: Query
- Query: `label_values(node_uname_info,nodename)` 

#### k8s instances

- Name: node
- Type: Query
- Query: `query_result(node_uname_info{node="$node"})` 
- Regex: `/instance="(.+?)"/` 

#### k8s pods

- Name: pod
- Type: Query
- Query: `label_values(kube_pod_info{namespace="$namespace"},  pod)` 
- 이렇게 설정했을 때 이미 종료된 Pod들이 보이는 경우가 있다. 이 때는 Refresh: On time range change 로 변경해주자. 이제 time range를 변경하면 그 안에 존재하는 metric으로부터 variable을 매핑해오므로 이미 종료되어 시간이 지난 것들은 나오지 않게 된다.



#### MySQL Overview Dashboard 설정

- Name: name
- Type: Query
- Query: `label_values(mysql_up, app_kubernetes_io_instance)` 



- Name: name
- Type: Query
- Query: `label_values(mysql_up{app_kubernetes_io_instance ="$name"}, instance)` 



이렇게 설정하면 `name` 을 고를 때마다 그에 해당하는 `instance` 를 Query 해와서 매핑해준다.

이렇게 하는 이유는 `instance` 값이 Query에 사용되기 때문에 반드시 필요한 값이긴 하나 IP:PORT 로만 된 값이라서 사람이 보고 판단하기 어렵기 때문이다.

사람이 보고 판단 가능한 `name` 값을 먼저 가져온 뒤, 그에 해당하는 `instance` 값도 불러와서 사람과 Query 양쪽에서 모두 사용 가능하도록 변수를 매핑해준다.



### Query

> https://prometheus.io/docs/prometheus/latest/querying/operators/



#### k8s master 정상 가동률

- Query: `avg(avg_over_time((sum without ()(kube_pod_container_status_ready{namespace="kube-system",pod=~".*.dashboard.*|.*.dns.*|kube.*|.*.calico.*|.*.flannel.*|.*.etcd.*"}) / count without ()(kube_pod_container_status_ready{namespace="kube-system",pod=~".*.dashboard.*|.*.dns.*|kube.*|.*.calico.*|.*.flannel.*|.*.etcd.*"}))[$duration:5m]))` 

#### k8s namespace 대수

- Query: ` count(kube_namespace_created)` 

#### k8s node 대수

- Query: `count(kube_node_info)` 

#### k8s pod 대수

- Query: ` count(count by (pod)(container_spec_memory_reservation_limit_bytes{pod!=""}))` 

#### k8s pvc 대수

- Query: `count(kube_persistentvolumeclaim_info) ` 



#### k8s node 별 CPU 사용률

- Query: `sum by(node) (rate(node_cpu_seconds_total{mode!~"guest.*|idle|iowait"}[$duration])) / on(node) group_left() kube_node_status_capacity{resource="cpu"}` 

#### k8s node 별 Memory 사용량

- Query: `sum by(node) (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)` 

#### k8s 초당 네트워크 트래픽

- Query: 
    - `sum(rate(node_network_receive_bytes_total[$duration]))` 
        - Legend: `inbound` 
        - Unit: `bytes(SI)` 
    - `sum(rate(node_network_transmit_bytes_total[$duration]))` 
        - Legend: `outbound` 
        - Unit: `bytes(SI)` 

#### k8s API 서버 호출

- Query: `sum by (verb) (rate(apiserver_request_total[$duration]))` 



#### k8s node CPU 사용률

- Query: `sum by(node) (rate(node_cpu_seconds_total{mode!~"guest.*|idle|iowait", node="$node"}[$duration])) / on(node) group_left() kube_node_status_capacity{resource="cpu", node="$node"}` 

#### k8s node Memory 사용률

- Query: `sum(node_memory_MemTotal_bytes{node="$node"} - node_memory_MemAvailable_bytes{node="$node"}) by (node) / sum(node_memory_MemTotal_bytes{node="$node"}) by(node)` 

#### k8s node Disk 사용률

- Query: `sum(node_filesystem_size_bytes{node="$node"} - node_filesystem_avail_bytes{node="$node"}) by (node) / sum(node_filesystem_size_bytes{node="$node"}) by (node)` 

#### k8s node CPU Core

- Query: `kube_node_status_capacity{resource="cpu", node="$node"}` 

#### k8s node Memory

- Query: `kube_node_status_capacity{resource="memory", node="$node"}` 



#### k8s pod Age

- Query: `time()-kube_pod_start_time{pod="$pod"}` 

#### k8s pod 가동 상태

- Query: `sum(kube_pod_container_status_waiting{pod="$pod", reason!~"ContainerCreating"})` 

#### k8s pod 리소스 사용량

- Query: 
    - `sum(rate(container_cpu_usage_seconds_total{pod="$pod"}[$duration]))` 
        - Legend: `CPU(%)` 
    - `sum(container_memory_working_set_bytes{pod="$pod",container!~"POD|"})`
        - Legend: `Memory(bytes)` 
    - `sum(container_fs_usage_bytes{pod="$pod"})` 
        - Legend: `File system usage` 

```sh
# pod 메모리 사용량 조회시 container_name이 "POD"이거나 ""(없는) 경우가 포함되어 메모리량이 튀는 경우가 있는데 이런 케이스를 제거한다.
# 또한 container_memory_usage_bytes 로 메모리를 조회하는 경우 언제든 kernel에 의해 해제될 수 있는 캐시까지도 포함한 값이므로 뻥튀기될 수 있다.
# 그러므로 OOM Killer가 바라보는 실제 메모리 사용량을 측정하기 위해서는 container_memory_working_set_bytes 를 사용하자.
sum(container_memory_working_set_bytes{pod="$pod",container!~"POD|"})
```

#### k8s pod 리소스 할당 제한값

- Query:
    - `kube_pod_container_resource_limits{pod="$pod", resource="cpu"}` 
    - `kube_pod_container_resource_limits{pod="$pod", resource="memory"}` 

#### k8s pod N분 이내 재시작

- Query: `sum(round(increase(kube_pod_container_status_restarts_total{pod="$pod"}[$duration])))` 

#### k8s pod 초당 네트워크 트래픽

- Query: 
    - `sum(rate(container_network_receive_bytes_total{pod="$pod"}[$duration]))` 
        - Legend: `inbound` 
        - Unit: `bytes(SI)` 
    - `sum(rate(container_network_transmit_bytes_total{pod="$pod"}[$duration]))` 
        - Legend: `outbound` 
        - Unit: `bytes(SI)` 

#### k8s pod 비정상 종료

- Query: `count by(pod, reason) (kube_pod_container_status_terminated_reason{reason!="Completed"})` 
    - 이 쿼리만 넣으면 현재 살아있는 container가 error로 restart만 1회 되고 계속 살아있는 경우는 잡아내지 못한다. 그래서 아래와 같이  restart 횟수와 join 해주는 쿼리를 추가한다.

- Query: `max by (pod, reason) (kube_pod_container_status_last_terminated_reason{reason!="Completed", namespace!~"kube-system|prometheus|default|whatap-monitoring", pod!="", container!~"POD|"}) * on (pod) group_left max by (pod) (rate(kube_pod_container_status_restarts_total{reason!="Completed", namespace!~"kube-system|prometheus|default|whatap-monitoring", pod!="", container!~"POD|"}[30s])) > 0` 
    - 원래 sum을 사용했으나 쿼리가 느려서 max로 바꿈




#### k8s 서비스 별 pod 개수

- Query: `count by (container) (container_spec_memory_reservation_limit_bytes{namespace!~"kube-system|prometheus|default|whatap-monitoring", pod!="", container!~"POD|"})` 
- Value Options:
    - Calculation: `Last *` 

#### k8s 서비스 별 cpu 사용률

- Query: `avg by(container) (avg by(container, pod) (rate(container_cpu_usage_seconds_total{namespace!~"kube-system|default|prometheus|whatap-monitoring", container!~"POD|", image!=""}[$duration])) / on(pod) group_left(container) kube_pod_container_resource_limits{namespace!~"kube-system|default|prometheus|whatap-monitoring", container!~"POD|", resource="cpu"})` 

#### k8s 서비스 별 memory 사용량

- Query: `avg(container_memory_working_set_bytes{namespace!~"kube-system|default|prometheus|whatap-monitoring", container!~"POD|", image!=""}) by(container)` 

#### k8s pod 별 cpu 사용률

- Query: `avg by(pod) (avg by(pod) (rate(container_cpu_usage_seconds_total{namespace!~"kube-system|default|prometheus|whatap-monitoring", container!~"POD|", image!=""}[$duration])) / on(pod) group_left() kube_pod_container_resource_limits{namespace!~"kube-system|default|prometheus|whatap-monitoring", container!~"POD|", resource="cpu"})` 

#### k8s pod 별 memory 사용량

- Query: `sum(container_memory_working_set_bytes{namespace!~"kube-system|default|prometheus|whatap-monitoring", container!~"POD|", image!=""}) by(pod)` 



#### k8s hpa replicas min

- Query: `sort(sum(kube_horizontalpodautoscaler_spec_min_replicas{namespace!~"kube-system|default|prometheus|whatap-monitoring"}) by(horizontalpodautoscaler))` 

#### k8s hpa replicas max

- Query: `sort(sum(kube_horizontalpodautoscaler_spec_max_replicas{namespace!~"kube-system|default|prometheus|whatap-monitoring"}) by(horizontalpodautoscaler))` 

#### k8s hpa metric cpu

- Query: `sort(sum(kube_horizontalpodautoscaler_spec_target_metric{namespace!~"kube-system|default|prometheus|whatap-monitoring", metric_name="cpu", metric_target_type="utilization"}) by(horizontalpodautoscaler))` 

#### k8s hpa metric memory

- Query: `sort(sum(kube_horizontalpodautoscaler_spec_target_metric{namespace!~"kube-system|default|prometheus|whatap-monitoring", metric_name="memory", metric_target_type="utilization"}) by(horizontalpodautoscaler))` 



#### mysql slow queries

- Query: 
    - `changes(mysql_global_status_slow_queries[$__rate_interval])` 
    - `changes(mysql_global_status_slow_queries{instance="$host"}[$__rate_interval])` 



## Datasource: Google Cloud Monitoring

### External/Internal 구분

default dashboard의 Load Balancing에 데이터가 나오지 않는 경우가 있다. 

Query로 들어가서 Metric name을 잘 보면 분명 같은 이름인데 External/Internal 용이 나누어져 있다. 

Metric name을 External/Internal에 맞게 잘 선택한 뒤 Filter: resource.type = internal_tcp_lb_rule과 같이 Filter 부분도 맞춰주면 데이터가 잘 보인다.

### Legend에서 Metric, Project 제거하기

Legent에 Metric, Project가 앞에 붙어서 너무 길게 나오는 경우가 있다. 

ex) Cloud SQL CPU Utilization: `cloudsql.googleapis.com/database/cpu/utilization gcp-dev-test-id:sql-dev-test-mysql` 

아래와 같이 필요한 부분만 보여주도록 설정한다.

- Transform: `Labels to fields` 
    - Mode: `Columns` 
    - Labels: `resource.label.database_id` 
    - Value field name: `resource.label.database_id` 
- Transform: `Rename by regex` 
    - Match: `(gcp-dev-test-id:)` 
    - Replace: ` ` 

