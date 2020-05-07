# Query

##
## 2. Vector matching
Tìm kiếm các phần từ phù hợp trong vector bên phải cho mỗi vector bên trái. Có 2 basic types của matching behavior: One-to-one và many-to-one/one-to-many.

### 2.1 One-to-one vector matches

Query từ mỗi side khớp nhau về label và giá trị tương ứng.`ignoring` cho phép bỏ qua các label trong khi `on` cho phép khớp label.

Ví dụ: 

Input như sau
```sh
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```
Query như sau:
```sh
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```
Nếu không có `ignoring(code)` sẽ không có kết quả trùng khớp vì không có cùng label. Khi có `ignoring(code)` sẽ có methods `put` và `del` là không khớp. Chỉ có method `get` và `post` lag khớp. Do đó, kết quả là :
```sh
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```
Nếu sử dụng `on(method)` tức là cả 2 phía chỉ cần có cùng label `on(method)` là khớp nhau.

### 2.2 Many-to-one and one-to-many vector matches

Label của query trên "one"-side có thể khớp với label của nhiều query khác. Trường hợp này sử dụng `group_left` hoặc `goup_right`, trong đó left/right xác định label khớp của các query.

Ví dụ: 

Input như sau
```sh
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```
Query như sau:
```
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```
Query bên trái chứa nhiều label hơn nên sử dụng `group_left`. Label bên phải khớp với nhiều query có cùng label ở bên trái. Có 4 query bên trái có label khớp với query bên phải, do đó kết quả như sau:
```sh
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

## 2.3 Aggregation operators

Tham khảo thêm các Aggregation [tại đây](https://prometheus.io/docs/prometheus/latest/querying/operators/#aggregation-operators)

Ví dụ:

Query `metric http_requests_total` sẽ có các label sau: `application`, `instance`, `group`

Tính tổng sood HTTP requests per application and group của tất cả các instances như sau:
```sh
sum without (instance) (http_requests_total)
```
Lệnh trên sử dụng `without` tướng ứng với query sử dụng `by` như sau:
```sh
sum by (application, group) (http_requests_total)
```
Xem tống số HTTP requests per application như sau:
```sh
sum(http_requests_total)
```
Đếm số lượng query có cùng giá trị dựa trên label:

ví dụ:

Query như sau:
```sh
prometheus_http_requests_total
``` 
Kết quả:
|Element|Value|
|-------|-----|
|prometheus_http_requests_total{code="200",handler="/alerts",instance="127.0.0.1:9090",job="prometheus"}|21|
|prometheus_http_requests_total{code="200",handler="/api/v1/label/:name/values",instance="127.0.0.1:9090",job="prometheus"}|11|
|prometheus_http_requests_total{code="200",handler="/api/v1/query",instance="127.0.0.1:9090",job="prometheus"}|124|
|prometheus_http_requests_total{code="200",handler="/api/v1/query_range",instance="127.0.0.1:9090",job="prometheus"}|6|
|prometheus_http_requests_total{code="200",handler="/config",instance="127.0.0.1:9090",job="prometheus"}|3|
|prometheus_http_requests_total{code="200",handler="/graph",instance="127.0.0.1:9090",job="prometheus"}|11|
|prometheus_http_requests_total{code="200",handler="/metrics",instance="127.0.0.1:9090",job="prometheus"}|3894|
|prometheus_http_requests_total{code="200",handler="/rules",instance="127.0.0.1:9090",job="prometheus"}|4|
|prometheus_http_requests_total{code="200",handler="/service-discovery",instance="127.0.0.1:9090",job="prometheus"}|2|
|prometheus_http_requests_total{code="200",handler="/static/*filepath",instance="127.0.0.1:9090",job="prometheus"}|162|
|prometheus_http_requests_total{code="200",handler="/targets",instance="127.0.0.1:9090",job="prometheus"}|112|
|prometheus_http_requests_total{code="302",handler="/",instance="127.0.0.1:9090",job="prometheus"}|4|
|prometheus_http_requests_total{code="400",handler="/api/v1/query",instance="127.0.0.1:9090",job="prometheus"}|17|

Đếm số lượng `Element` có cùng `Value` sử dụng lable `code`
```sh
count_values("code", prometheus_http_requests_total)
```
Kết quả

|Element|Value|
|-------|-----|
|{code="2"}|1|
|{code="162"}|1|
|{code="112"}|1|
|{code="17"}|1|
|{code="124"}|1|
|{code="3"}|1|
|{code="6"}|1|
|{code="3894"}|1|
|{code="4"}|2|
|{code="21"}|1|
|{code="11"}|2|

Lấy 5 giá trị HTTP requests lớn nhất của toàn bộ instances như sau:
```sh
topk(5, http_requests_total)
```

## Tài liệu tham khảo 
- https://prometheus.io/docs/prometheus/latest/querying/operators/


