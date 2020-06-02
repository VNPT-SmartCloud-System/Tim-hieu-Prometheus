# Cài đặt Consul Mô hình 1 Node không HA

## Mô hình Lab

|Server|IP|
|------|--|
|Master|192.168.1.1
|Client|192.168.1.2|

## 1. Cài đặt Consul sử dụng Docker

### 1.1 Pull image `consul` trên cả 2 Node
```sh
docker pull consul
```
### 1.2 Thực hiện trên Node `Master`
Container sử dụng Network của Host và bind trên địa chỉ IP của Host
```sh
docker run \
  --network host \
  -d \
  -p 8500:8500 \
  -p 8600:8600/udp \
  --name=master-01 \
  -e CONSUL_LOCAL_CONFIG='{
  "bind_addr":"192.168.1.1"
  }' \
  consul agent -server -ui -node=server-01 -bootstrap-expect=1 -client=0.0.0.0
```
### 1.2 Thực hiện trên Node `Client`
```sh
docker run \
  -d \
  --network host \
  --name=client-01 \
  -e CONSUL_LOCAL_CONFIG='{
  "bind_addr":"192.168.1.2"
  }' \
  consul agent -node=client-01 -join=192.168.1.1
```
## 2. Khai báo Service cho Client
### 2.1 Khai báo Service `Node_exporter`
```sh
docker exec client-01 /bin/sh -c "echo '{\"service\": {\"name\": \"node_exporter\", \"tags\": [\"node-exporter\"], \"port\": 9100}}' >> /consul/config/service_node_exporter.json"
```
### 2.2 Reload lại cấu hình của Client
```
docker exec client-01 consul reload
```
## 3. Kiểm tra

Vào giao diện web để kiểm tra: http://192.168.1.1:8500

## Tài liệu tham khảo
- https://learn.hashicorp.com/consul/day-0/containers-guide
