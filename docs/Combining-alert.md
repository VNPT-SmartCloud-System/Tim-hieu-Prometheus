# Kết hợp cảnh báo

Kết hợp các cảnh báo sử dụng [logical binary operator]() kết hợp với [vector matching]():
```sh
groups:
- name: test.rules
  rules:
  - alert: LatencyTooHigh
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5 and ON(job) job:requests:rate5m{job="myjob"} > 100
```
Trả về kết quả bên trái nếu bên phải khớp. `ON(job)` có nghĩa là job label `job="myjob"` phải khớp ở cả 2 bên.

Giới hạn thời gian
```sh
groups:
- name: test.rules
  rules:
  - alert: LatencyTooHigh
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5 and ON() hour() > 9 < 17
```
`unless` trái ngược với `and`. Nó trả về kết quả bên trái nếu không khớp với kết quả bên phải.

## Tài liệu tham khảo
- https://www.robustperception.io/combining-alert-conditions
