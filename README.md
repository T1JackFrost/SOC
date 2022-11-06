# ELK ?

1. ELK là một công cụ quản lý logs tập trung. ELK viết tắt của 3 dự án mã
   nguồn mở:

- E - Elasticsearch: là công cụ phân tích & tìm kiếm, đóng vai trò như 1
  storage để lưu trữ logs
- L - Logstash: công cụ thu thập, xử lý log từ nhiều nguồn đồng thời. Nhiệm
  vụ chính là thu thập log, biến đổi nó và gửi đến Elasticsearch
- K - Kibana: Giao diện quản lý, thống kê log bằng đồ thị, biểu đồ. Đọc
  thông tin từ Elasticsearch

2. Luồng hoạt động

- Đầu tiên log được đưa đến Logstash (thông qua nhiều con đường như server
  gửi UDP request chứa log tới URL của Logstash, hoặc Beat đọc file log rồi
  gửi lên Logstash)
- Logstash đọc những log này, thêm thông tin như thời gian, IP, parse dữ
  liệu từ log, sau đó ghi xuống database là Elasticsearch
- Khi cần xem log, người dùng vào URL của Kibana. Kibana đọc thông tin log
  trong Elasticsearch, hiển thị lên giao diện người dùng

# Build ELK-Stack

1. Cài đặt ELK trên server (hệ điều hành CentOS 7)

- Kiểm tra đảm bảo đã cài đặt java (openjdk)
- Bước 1: Cài đặt Elasticsearch:
  - Thêm repository
  - Cài đặt Elasticsearch
  - Kích hoạt dịch vụ
  - Mở firewall port 9200 cho ES nếu cần
  - Kiểm tra ES
- Bước 2: Cài đặt Kibana:
  - Cài đặt Kibana
  - Kích hoạt dịch vụ
  - Cấu hình truy cập được từ mọi IP: echo 'server.host: 0.0.0.0' >>
    /etc/kibana/kibana.yml
  - Kiểm tra Kibana bằng cách truy cập đến địa chỉ IP server với port 5601:
    https://x.x.x.x:5601 với x.x.x.x là địa chỉ IP
- Bước 3: Cài đặt Logstash:
  - Cài đặt Logstash
  - Cấu hình file input để nó nhận đầu vào do Beats gửi đến
  - Cấu hình file output để sau khi Logstash nhận dữ liệu đầu vào từ Beats
    nó xử lý rồi gửi đến Elasticsearch (localhost:9200)
  - Kiểm tra lại cấu hình
  - Mở firewall port 5044 để nhận dữ liệu từ server khác (optional)
  - Kích hoạt dịch vụ
- Bước 4: Cài đặt Beats/Filebeat  
  Beats là nền tảng để gửi dữ liệu vào Logstash. Ở đây sử dụng Filebeat,
  thu thập file log trên máy cài đặt nó, cấu hình để gửi đến Logstash
  - Cài đặt Filebeat
  - Chỉnh sửa một số nội dung trong file cấu hình filebeat.yml:
    - Không gửi thẳng log đến Elasticsearch
    - Yêu cầu Filebeat gửi đến Logstash
    - Thêm 1 số đường dẫn để nhận log từ các đường dẫn này (optional)
  - Xem trạng thái các module ứng với loại log thu thập
  - Kích hoạt module
  - Kích hoạt dịch vụ Filebeat

2. Cài đặt Filebeat trên máy gửi log đến server (Hệ điều hành Ubuntu)

- Cài đặt apt-transport-https
- Cài đặt Filebeat
- Kích hoạt dịch vụ filebeat
