# ELK ?

## 1. Khái niệm

ELK là một công cụ quản lý logs tập trung. ELK viết tắt của 3 dự án mã
nguồn mở:

- E - _Elasticsearch_: là công cụ phân tích & tìm kiếm, đóng vai trò như 1
  storage để lưu trữ logs
- L - _Logstash_: công cụ thu thập, xử lý log từ nhiều nguồn đồng thời.
  Nhiệm vụ chính là thu thập log, biến đổi nó và gửi đến Elasticsearch
- K - _Kibana_: Giao diện quản lý, thống kê log bằng đồ thị, biểu đồ. Đọc
  thông tin từ Elasticsearch

## 2. Luồng hoạt động

- Đầu tiên log được đưa đến Logstash (thông qua nhiều con đường như server
  gửi UDP request chứa log tới URL của Logstash, hoặc Beat đọc file log rồi
  gửi lên Logstash)
- Logstash đọc những log này, thêm thông tin như thời gian, IP, parse dữ
  liệu từ log, sau đó ghi xuống database là Elasticsearch
- Khi cần xem log, người dùng vào URL của Kibana. Kibana đọc thông tin log
  trong Elasticsearch, hiển thị lên giao diện người dùng

# Build ELK-Stack

## 1. Cài đặt ELK trên server (Hệ điều hành CentOS 7)

- Kiểm tra đảm bảo đã cài đặt java (openjdk)
- Bước 1: Cài đặt Elasticsearch:
  - Thêm repository:
    > echo '[elasticsearch-7.x] <br>name=Elasticsearch repository for
    > 7.xpackages
    > <br>baseurl=https://artifacts.elastic.co/packages/7.x/yum >
    > <br>gpgcheck=1
    > <br>gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch >
    > <br>enabled=1 <br>autorefresh=1 <br>type=rpm-md <br> ' >
    > /etc/yum.repos.d/elasticsearch.repo
  - Cài đặt Elasticsearch:
    > yum install elasticsearch -y
  - Kích hoạt dịch vụ
    > systemctl enable elasticsearch systemctl start elasticsearch
  - Mở firewall port 9200 cho ES nếu cần (optional)
  - Kiểm tra ES:
    > curl -X GET localhost:9200
- Bước 2: Cài đặt Kibana: <br>Kibana là ứng dụng nền web, nó lắng nghe các
  truy vấn http gửi đến cổng mặc định 5601, phần này sẽ cấu hình truy cập
  trực tiếp đến cổng 5601 (cấu hình server.host ở dưới)
  - Cài đặt Kibana
  - Kích hoạt dịch vụ
  - Cấu hình truy cập được từ mọi IP:
    > echo 'server.host: 0.0.0.0' >> /etc/kibana/kibana.yml
  - Kiểm tra Kibana bằng cách truy cập đến địa chỉ IP server với port 5601:
    > https://x.x.x.x:5601 với x.x.x.x là địa chỉ IP
- Bước 3: Cài đặt Logstash:
  - Cài đặt Logstash
  - Cấu hình file input để nó nhận đầu vào do Beats gửi đến:
    > echo 'input { <br> beats { <br> host => "0.0.0.0" <br> port => 5044
    > <br> } <br>}' > /etc/logstash/conf.d/02-beats-input.conf
  - Cấu hình file output để sau khi Logstash nhận dữ liệu đầu vào từ Beats
    Logstash sẽ xử lý rồi gửi đến Elasticsearch (localhost:9200):
    > echo 'output { <br> elasticsearch { <br> hosts =>
    > ["localhost:9200"] > <br> manage_template => false <br> index =>
    > "%{[@metadata][beat]}-%{[@metadata][version]}<br> {+YYYY.MM.dd}" <br>
    > } <br>}' > /etc/logstash/conf.d/30-elasticsearch-output.conf
  - Kiểm tra lại cấu hình:
    > sudo -u logstash /usr/share/logstash/bin/logstash --path.settings
    > /etc/logstash -t
  - Mở firewall port 5044 để nhận dữ liệu từ server khác (optional)
  - Kích hoạt dịch vụ Logstash

## 2. Cài đặt Filebeat trên máy gửi log đến server (Hệ điều hành Ubuntu)

- Beats/Filebeat  
  Beats là nền tảng để gửi dữ liệu vào Logstash. Ở đây sử dụng _Filebeat_,
  thu thập file log trên máy cài đặt nó, cấu hình để gửi đến Logstash.
- Cài đặt Filebeat:
  - Thực hiện tải file:
    > wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo
    > apt-key add -
  - Cài đặt apt-transport-https (nếu chưa có):
    > sudo apt-get install apt-transport-https
  - Cài đặt Filebeat
  - Kích hoạt dịch vụ Filebeat
  - Thu thập toàn bộ syslog từ /var/log/syslog
  - Thực hiện chỉnh sửa 1 số nội dung trong file cấu hình filebeat.yml:
    - Thêm một số đường dẫn để nhận log từ đường dẫn cung cấp log:
      > filebeat.inputs:  
      > path: - /đường-dẫn-để-lấy-file-ghi-log
    - Yêu cầu Filebeat gửi đến Logstash server qua port 5044:
      > output.logstash:  
      > host: ["x.x.x.x:5044"] với x.x.x.x là địa chỉ IP server nhận log
  - Xem trạng thái các module ứng với loại log thu thập
  - Kích hoạt module (ở đây cần dùng module system):
    > filebeat modules enable system
  - Kích hoạt dịch vụ Filebeat

## 3. Xem log trong giao diện Kibana

- Truy cập vào Kibana theo địa chỉ IP của ELK, http://x.x.x.x:5601, nhấn
  vào Discover, chọn mục Index Management của Elasticsearch
- Các index có tiền tố là filebeat-\*, chính là các index lưu dữ liệu log
  do Filebeat gửi đến Logstash và Logstash để chuyển lưu tại Elasticsearch.
  (Nếu có nhiều server gửi đến thì có nhiều index dạng này)
