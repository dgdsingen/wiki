# Concept

- Dashboard 맨 위에 Variables가 selectbox로 노출된다. Variables 중 node를 node-1로 선택하면 이후 Query에서 "$node"의 값으로 "node-1" 매핑된다.



# K8s

## Variables

### List nodes

- Name: node
- Type: Query
- Data source: Prometheus
- Query: `label_values(node_uname_info,nodename)` 



### List instances

- Name: node
- Type: Query
- Data source: Prometheus
- Query: `query_result(node_uname_info{node="$node"})` 
- Regex: `/instance="(.+?)"/` 



### List Pods

- Name: pod
- Type: Query
- Data source: Prometheus
- Query: `label_values(kube_pod_info{namespace="$namespace"},  pod)` 
- 이렇게 설정했을 때 이미 종료된 Pod들이 보이는 경우가 있다. 이 때는 Refresh: On time range change 로 변경해주자. 이제 time range를 변경하면 그 안에 존재하는 metric으로부터 variable을 매핑해오므로 이미 종료되어 시간이 지난 것들은 나오지 않게 된다.



## MySQL Overview Dashboard 설정

- Name: name
- Type: Query
- Data source: Prometheus
- Query: `label_values(mysql_up, app_kubernetes_io_instance)` 



- Name: name
- Type: Query
- Data source: Prometheus
- Query: `label_values(mysql_up{app_kubernetes_io_instance ="$name"}, instance)` 



이렇게 설정하면 `name` 을 고를 때마다 그에 해당하는 `instance` 를 Query 해와서 매핑해준다.

이렇게 하는 이유는 `instance` 값이 Query에 사용되기 때문에 반드시 필요한 값이긴 하나 IP:PORT 로만 된 값이라서 사람이 보고 판단하기 어렵기 때문이다.

사람이 보고 판단 가능한 `name` 값을 먼저 가져온 뒤, 그에 해당하는 `instance` 값도 불러와서 사람과 Query 양쪽에서 모두 사용 가능하도록 변수를 매핑해준다.



## Query

### Node 별 CPU 점유율

- Data source: Prometheus
- Query: `sum by(node) (irate(node_cpu_seconds_total{mode!~"guest.*|idle|iowait", node="$node"}[5m]))` 

### Node 별 Memory 점유율

- Data source: Prometheus
- Query: `sum(node_memory_MemTotal_bytes{node="$node"} - node_memory_MemAvailable_bytes{node="$node"}) by (node) / sum(node_memory_MemTotal_bytes{node="$node"}) by(node)` 

### Node 별 Disk 사용율

- Data source: Prometheus
- Query: `sum(node_filesystem_size_bytes{node="$node"} - node_filesystem_avail_bytes{node="$node"}) by (node) / sum(node_filesystem_size_bytes{node="$node"}) by(node)` 

### Pod 별 리소스 제한값

- Data source: Prometheus
- Query:
    - `kube_pod_container_resource_limits{pod="$pod", resource="cpu"}` 
    - `kube_pod_container_resource_limits{pod="$pod", resource="memory"}` 

### Pod 가동 상태

- Data source: Prometheus
- Query:`sum(kube_pod_container_status_waiting{pod="$pod", reason!~"ContainerCreating"})` 



# Issues

```sql
# pod 메모리 사용량 조회시 container_name이 "POD"이거나 ""(없는) 경우가 포함되어 메모리량이 튀는 경우가 있는데 이런 케이스를 제거한다.
# 또한 container_memory_usage_bytes 로 메모리를 조회하는 경우 언제든 kernel에 의해 해제될 수 있는 캐시까지도 포함한 값이므로 뻥튀기될 수 있다.
# 그러므로 OOM Killer가 바라보는 실제 메모리 사용량을 측정하기 위해서는 container_memory_working_set_bytes 를 사용하자.
sum(container_memory_working_set_bytes{pod="$pod",container!~"POD|"})
```

