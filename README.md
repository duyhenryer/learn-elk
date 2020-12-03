# Grok là gì?

Grok là một dạng biểu thức regular expression, được sử dụng để phân tích cấu trúc văn bản.

# Cách sử dụng
Grok được áp dụng khá đa dạng. Mỗi kiểu dữ liệu sẽ được khai báo các mẫu pattern, và có thể sử dụng lẫn nhau trong bộ lọc filter.

Nghĩa là khi định nghĩa pattern cho kiểu log syslog, nhưng có thể sử dụng được đoạn khai báo pattern đó cho kiểu log MySQL.

# Beats - gửi dữ liệu thu thập từ log của máy đến Logstash

Hiện tại có các Beats được cung cấp sẵn bởi elastic là:

+ Packetbeat : lấy / gửi các gói tin trên các Port của Client, chuyển tiếp dữ liệu về Logstash
+ Filebeat : lấy / gửi các file log của Server
+ Metricbeat (Topbeat): lấy / gửi các log dịch vụ (Apache log, mysql log ...) (Thu thập dữ liệu về tài nguyên hệ thống client và gửi về Logstash)
+ Winlobeat: Thu thập dữ liệu trên windows (Chưa thử, nhưng có thể sử dụng nxlog để thu thập)

# Logstash
Nhận dữ liệu từ các beats, tiến hành phân tích dữ liệu
Phân tích dữ liệu gửi từ filebeat bằng grok
Grok là một dạng khai báo pattern sử dụng regular expression
Grok đã được khai báo rất nhiều pattern có sẵn, bạn có thể sử dụng ngay.
Nếu bạn có các loại dữ liệu đặc thù thì có thể dựa vào regular expression để khai báo các pattern theo yêu cầu
Các dữ liệu packetbeat, topbeat thì có các template sẵn nên không cần khai báo filter, được logstach gửi thẳng vào Elasticsearch
Dữ liệu được gửi sang cho Elasticsearch để lưu trữ

# EFK khác gì ELK
EFK và ELK là cùng là stack có chức năng quản trị tập trung dữ liệu log. Khác nhau duy nhất ở đây là trong ELK stack thì ta sử dụng logstash có nhiệm vụ parser log, và khi sử dụng ELK ta cần khá nhiều shipper log như: filebeat, packetbeat, metricbeat, winlogbeat.

Còn EFK stack thì đã thay thế logstash bằng fluentd/fluentbit để xử lý parser log. Ngoài ra, fluentd/fluentbit còn có thể thực hiện chức năng shipper log.

# Thiết lập cấu hình fluentd
# Cấu trúc file config
Fluentd chia các thành phần trong file cấu hình thành 5 phần chính:

source: Đây là khi báo các kiểu log sẽ đi vào. Ví dụ từ syslog, http, tcp,...
filter: Đây là khai báo các cách phân tích, xử lý. Ví dụ sửa lại time, thêm thông tin,....
match: Đây là khai báo các kiểu log sẽ đi ra. Ví dụ ra file, DB, Elasticsearch,...
label: Dùng để điều hướng các log, không bị phụ thuộc vào kiểu duyệt top-down
system: Khai báo để bắt các lỗi như không thể ghi dữ liệu ra file, hoặc thiết lập cấu hình cho tiến trình.

> Ngoài ra, còn có thêm một chỉ dẫn nữa là @include dùng để import các khai báo nằm trong 5 loại trên được tách ra file riêng.

> Nguyên tắc hoạt động của fluentd là duyệt theo kiểu top - down, nghĩa là khi một event log đi vào, sẽ đi qua tất các các filter, match khớp với khai báo.

# Plugin
Như phần trên có đề cập, cấu trúc file config gồm 5 phần, nhưng fluentd lại sử dụng cách thiết kế plugin cho các khai báo này. Mục đích là để chia nhỏ file cấu hình theo kiểu module.

Trong fluentd có tất cả 8 loại plugin:

Input
Parser
Filter
Output
Formatter
Storage
Service Discovery
Buffer
Một số plugin thì dùng ngay cho các phần khai báo trong file config. ví dụ: plugin input sẽ dành cho phần cấu hình source, dùng để khai báo các kiểu dữ liệu đầu vào. Hoặc plugin output dành cho phần cấu hình match để khai báo các kiểu dữ liệu đầu ra.

Nhưng thực tế không hoàn toàn tách bạch như vậy, các plugin có thể sử dụng lồng nhau. ví dụ như file config ở trên, trong phần match ta sử dụng 02 plugin là output và buffer

# Plugin Input
Với plugin input có nhiệm vụ nhận các dữ liệu đầu vào, hiện tại có 10 loại input native (có sẵn) của fluentd

- in_tail
- in_forward
- in_udp
- in_tcp
- in_unix
- in_http
- in_syslog
- in_exec
- in_dummy
- in_windows_evenlog

## Tail
Dùng để đọc log từ các text file theo cách sử dụng lệnh tail -f trên linux

```
 <source>
  @type tail
  path /var/log/httpd-access.log
  pos_file /var/log/td-agent/httpd-access.log.pos
  tag apache.access
  <parse>
    @type apache2
  </parse>
</source>
```

Nếu fluentd chạy trong container thì phải thực hiện bind mount thư mục log vào trong container mới sử dụng kiểu tail được. Khi dùng kiểu tail này thì có một số tham số phải thiết lập:

- @type: bắt buộc là tail
- tag: bắt buộc để điều hướng log
- path: bắt buộc để chỉ dẫn file sẽ đọc dữ liệu log. Có thể khai báo nhiều file cách nhau bằng dấu phẩy (,). Ví dụ path /path/to/a/*,/path/to/b/c.log

> Trong phần khai báo input là tail ta phải sử dụng thêm một plugin parse để phân tích dữ liệu đầu vào.

## Forward
```
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

```

Ở ví dụ trên, lựa chọn plugin input dùng cơ chế forward. Dùng để nhận các event stream. Thường được dùng để nhận log từ các container.

- @type: require forward
- port: kiểu integer, default là 24224
- bind: IP, default là 0.0.0.0 (all addresses)
- tag: kiểu string, tag dùng để điều hướng log, thường dùng tag từ log nhận được, nếu định nghĩa tag ở đây thì sẽ thay thế tag trên dữ liệu gốc.
- add_tag_prefix: thêm giá trị vào đầu của các tag mặc định nhận được. Ví dụ khai báo add_tag_prefix prod thì nhận được log sẽ đổi tag thành prod.INCOMING_TAG

Trong phần khai báo của forward, còn có một số section khác như:

- <transport> section: để bổ sung TLS cho kết nối, nếu không có thì sẽ là TCP. mẫu khai báo như sau:
```
<transport tls>
  cert_path /etc/td-agent/certs/fluentd.crt
  private_key_path /etc/td-agent/certs/fluentd.key
  private_key_passphrase YOUR_PASSPHRASE
</transport>
```

- <security> section: xác thực bằng mật khẩu cho các kết nối gửi log tới fluentd
```
<source>
  @type forward
  <security>
    self_hostname YOUR_SERVER_NAME
    shared_key PASSWORD
  </security>
</source>
```
- <user> section
- <client> section

# UDP
Dùng để nhận các dữ liệu log đẩy theo định dạng UDP, cấu hình mẫu như sau:
```
<source>
  @type udp
  tag mytag # required
  <parse>
    @type regexp
    expression /^(?<field1>\d+):(?<field2>\w+)$/
  </parse>
  port 20001               # optional. 5160 by default
  bind 0.0.0.0             # optional. 0.0.0.0 by default
  message_length_limit 1MB # optional. 4096 bytes by default
</source>
```

Một số tham số trong cấu hình input UDP:

- type: bắt buộc phải là udp
- tag: dùng để đánh nhãn cho log nhận được, sử dụng cho các filter kế tiếp. đây là trường bắt buộc.
- port: khai báo port listen theo giao thức UDP
- bind: chỉ định IP sẽ listen, với container thì nên để 0.0.0.0
# System session
Dùng để thiết lập cấu hình cho các tiến trình như output log khi không thể ghi được ra ngoài output (match).

Dưới đây là thiết lập chạy multi-process chung một port input:
```
<system>
  workers 3
</system>

<source>
  @type forward
  port 24224
</source>
```

Sẽ chạy 3 worker cùng chia sẻ port 24224, dữ liệu vào sẽ được routed tự động tới cả 3 worker.

