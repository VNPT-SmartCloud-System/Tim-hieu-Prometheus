# PromQL

PromQL là ngôn ngữ truy vấn cho hệ thống giám sát Prometheus.

## 1. Selecting time series với PromQL

Query time series với name `node_network_receive_bytes_total`
```sh
node_network_receive_bytes_total
```
Kết quả:
```sh
node_network_receive_bytes_total{device="eth0"}
node_network_receive_bytes_total{device="eth1"}
node_network_receive_bytes_total{device="eth2"}
```
Kết quả trả về lệnh query với các label được bỏ trong ngoặc nhọn: `{device="eth0"}`, `{device="eth1"}`, `{device="eth2"}`

## 2. Filter theo label
```sh
node_network_receive_bytes_total{device="eth1"}
```
Muốn lấy kết quả không phải cho `eth1` thực hiện bằng cách thêm cú pháp so sánh như `!=`, `=`, `>`... tham khảo thêm [tại đây](https://prometheus.io/docs/prometheus/latest/querying/operators/#comparison-binary-operators)
```sh
node_network_receive_bytes_total{device!="eth1"}
```
Thực hiện query với tất cả device bắt đầu bằng `eth`
```sh
node_network_receive_bytes_total{device=~"eth.+"}
```
Thực hiện query với tất cả device không bắt đầu bằng `eth`
```sh
node_network_receive_bytes_total{device!~"eth.+"}
```
Filter theo nhiều label `instance="node1:9100"` và `device=~"eth.+"`
```sh
node_network_receive_bytes_total{instance="node42:9100",device=~"eth.+"}
```
Kết hợp `and`, `or`
Filter theo `eth1` hoặc `l0`
```sh
node_network_receive_bytes_total{device=~"eth1|lo"}
```
## 3. Filter theo metric name
Query với `node_network_receive_bytes_total` hoặc `node_network_transmit_bytes_total` sử dụng metric name `node_network` với `receive` hoặc `transmit`:
```sh
{__name__=~"node_network_(receive|transmit)_bytes_total"}
```
Filter kết quả trong 1 tuần
```sh
node_network_receive_bytes_total offset 7d
```
Filter các điểm thỏa mãn điều kiện
```sh
go_memstats_gc_cpu_fraction > 1.5 * (go_memstats_gc_cpu_fraction offset 1h)
```
## 4. Calculating rates





# Tài liệu tham khảo
- https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085
