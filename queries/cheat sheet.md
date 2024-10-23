# PromQL Queries README

This document provides commonly used Prometheus Query Language (PromQL) queries for monitoring system and application performance metrics. Each section outlines specific metrics for CPU, RAM, SWAP, disk usage, HTTP request rates, Kubernetes container usage, and error rates.

## CPU Metrics

### CPU Usage in Last 5 Minutes
```promql
(1 - avg(rate(node_cpu_seconds_total{instance=~"$instance",mode="idle"}[5m])) by (instance)) * 100
```

### Number of CPU Cores
```promql
sum(count(node_cpu_seconds_total{instance=~"$instance", mode='system'}) by (cpu))
```

## Memory Metrics

### RAM Used
```promql
(1 - node_memory_MemAvailable_bytes{instance=~"$instance"} / node_memory_MemTotal_bytes{instance=~"$instance"}) * 100
```

### Total RAM
```promql
sum(node_memory_MemTotal_bytes{instance=~"$instance"})
```

### Memory Breakdown
- **Total**: `node_memory_MemTotal_bytes{instance=~"$instance"}`
- **Available**: `node_memory_MemAvailable_bytes{instance=~"$instance"}`
- **Cache + Buffer**: 
  ```promql
  node_memory_Cached_bytes{instance=~"$instance"} + node_memory_Buffers_bytes{instance=~"$instance"}
  ```
- **Free**: `node_memory_MemFree_bytes{instance=~"$instance"}`
- **SWAP Used**: 
  ```promql
  (node_memory_SwapTotal_bytes{instance=~"$instance"} - node_memory_SwapFree_bytes{instance=~"$instance"})
  ```

### SWAP Metrics

#### SWAP Used
```promql
((node_memory_SwapTotal_bytes{instance=~"$instance"} - node_memory_SwapFree_bytes{instance=~"$instance"}) / (node_memory_SwapTotal_bytes{instance=~"$instance"})) * 100
```

#### Total SWAP
```promql
node_memory_SwapTotal_bytes{instance=~"$instance"}
```

## Load Metrics

### Node Load Percentage
- **1 Minute**:
  ```promql
  avg(node_load1{instance=~"$instance"}) / count(count(node_cpu_seconds_total{instance=~"$instance"}) by (cpu)) * 100
  ```
- **5 Minutes**:
  ```promql
  avg(node_load5{instance=~"$instance"}) / count(count(node_cpu_seconds_total{instance=~"$instance"}) by (cpu)) * 100
  ```
- **15 Minutes**:
  ```promql
  avg(node_load15{instance=~"$instance"}) / count(count(node_cpu_seconds_total{instance=~"$instance"}) by (cpu)) * 100
  ```

## Disk Metrics

### Disk Usage
```promql
(1 - node_filesystem_avail_bytes{instance=~"$instance",device!~'rootfs'}/ node_filesystem_size_bytes{instance=~"$instance",device!~'rootfs'}) * 100
```

### Disk I/O

#### Completed IOPS
- **Read**:
  ```promql
  rate(node_disk_reads_completed_total{instance=~"$instance"}[5m])
  ```
- **Write**:
  ```promql
  irate(node_disk_writes_completed_total{instance=~"$instance"}[5m])
  ```

#### Read/Write kBs
- **Read**:
  ```promql
  irate(node_disk_read_bytes_total{instance=~"$instance"}[5m])
  ```
- **Write**:
  ```promql
  irate(node_disk_written_bytes_total{instance=~"$instance"}[5m])
  ```

## Top/Bottom Resource Utilization

### Top 5 Used and Unused Nodes in Last Hour (5 min step size)
- **Top**:
  ```promql
  topk(5, ((1 - avg(avg_over_time(rate(node_cpu_seconds_total{mode="idle"}[5m])[1h:1m])) by (host)) * 100))
  ```
- **Bottom**:
  ```promql
  bottomk(5, ((1 - avg(avg_over_time(rate(node_cpu_seconds_total{mode="idle"}[5m])[1h:1m])) by (host)) * 100))
  ```

## HTTP Request Metrics

### HTTP Request Rate (per second, 1 hour ago)
```promql
rate(api_http_requests_total{status=500}[5m] offset 1h)
```

## Kubernetes Metrics

### Kubernetes Container Memory Usage
```promql
sum by(kubernetes_pod_name) (container_memory_usage_bytes{kubernetes_namespace="kube-system"})
```

## Aggregate Request Rates

### Total Request Rates
```promql
sum without(instance, status)(rate(api_requests_total[5m]))
```

### Measure Error Rate Ratios and Percentages
```promql
sum without(status)(rate(api_requests_total{status=~"5.."}[5m])) / sum without(status)(rate(api_requests_total[5m]))
```
```promql
sum without(status)(rate(api_requests_total{status=~"5.."}[5m])) / sum without(status)(rate(api_requests_total[5m])) * 100
```

## Identifying Resource Hogs in Prometheus
```promql
topk(5, sum by (service) (rate(container_cpu_usage_seconds_total[5m])) / sum by (service) (machine_cpu_cores))
```

