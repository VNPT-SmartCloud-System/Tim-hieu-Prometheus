# Hương dẫn tích hợp Consul vào Prometheus

## 1. Với trường hợp sử dụng docker-prometheus

- **Sửa file cấu hình prometheus và thêm vào nội dung sau:**
```sh
vim prometheus.yml
...
scrape_configs:
  - job_name: 'consul'
    consul_sd_configs:
      - server: '192.168.1.1:8500'
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        ### Map với tất cả các tag
        regex: .*
        action: keep
      - source_labels: [__meta_consul_service]
        ### Lấy tên service khai báo trên Consul làm lablel 
        target_label: job
```
- **Restart lại docker-prometheus**
```sh
docker restart prom/prometheus
```
