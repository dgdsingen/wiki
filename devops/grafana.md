# Concept

- Dashboard 맨 위에 Variables가 selectbox로 노출된다. Variables 중 node를 node-1로 선택하면 이후 Query에서 "$node"의 값으로 "node-1" 매핑된다.



# K8s

## Variables

### List nodes

- Name: node
- Type: Query
- Data source: Prometheus
- Query: label_values(node_uname_info,nodename)



## Query

### Node 별 CPU 점유율

- Data source: Prometheus
- Query: `sum by(instance) (irate(node_cpu_seconds_total{mode!~"guest.*|idle|iowait", node="$node"}[5m]))` 

### Node 별 Memory 점유율

- Data source: Prometheus
- Query: `sum(node_memory_MemTotal_bytes{node="$node"} - node_memory_MemAvailable_bytes{node="$node"}) by (node) / sum(node_memory_MemTotal_bytes{node="$node"}) by(node)` 

### Node 별 Disk 사용율

- Data source: Prometheus
- Query: `sum(node_filesystem_size_bytes{node="$node"} - node_filesystem_avail_bytes{node="$node"}) by (node) / sum(node_filesystem_size_bytes{node="$node"}) by(node)` 