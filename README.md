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







