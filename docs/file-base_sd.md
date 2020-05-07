# Service Discovery

Sử dụng file_base Service Discovery

## 1. Chuẩn bị

Tạo file `targets.json` với nội dung sau:

```sh
[
  {
    "targets": [ "192.168.1.1:9100", "192.168.1.2:9100" ],
    "labels": {
      "env": "production",
      "job": "node_exporter"
    }
  },
  {
    "targets": [ "192.168.1.3:8080" ],
    "labels":
      "job": "cadvaisor"
  }
]
```
Sủa file config để để scrape sử dụng file_base.
```sh
scrape_configs:
  - job_name: 'file_base_sd'
    file_sd_configs:
      - files:
        - targets.json
```
## 2. Sử dụng package
## 3. Sử dụng docker container
```sh
docker run --restart=always -d --name prometheus \
   -p 9090:9090 \
   -v /root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
   -v /root/prometheus/host_alert.rules.yml:/etc/prometheus/host_alert.rules.yml \
   -v /root/prometheus/targets.json:/etc/prometheus/targets.json \
   -v promql:/prometheus \
   prom/prometheus --config.file=/etc/prometheus/prometheus.yml
```

## Tài liệu tham khảo
- https://www.robustperception.io/using-json-file-service-discovery-with-prometheus
- https://prometheus.io/docs/prometheus/latest/configuration/configuration/
- https://stuarthowlette.me.uk/posts/prometheus-consul-node_exporter/
- https://prometheus.io/blog/2015/06/01/advanced-service-discovery/
