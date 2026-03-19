Cài đặt:
sudo apt update && sudo apt upgrade -y
sudo apt install default-jre default-jdk -y
sudo apt install curl apt-transport-https -y
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list
sudo apt update
sudo apt install elasticsearch -y

 S-H+Pd65yy4EMkkkymgz

Cấu hình:
Sửa file /etc/elasticsearch/elasticsearch.yml
network.host: 192.168.20.100
discovery.type: single-node
xpack.security.http.ssl:
  enabled: false
  keystore.path: certs/http.p12

sudo systemctl daemon-reload
sudo systemctl enable --now elasticsearch.service
sudo systemctl status elasticsearch.service

Cài kibana:
sudo apt install kibana -y
Cấu hình kibana:
sudo nano /etc/kibana/kibana.yml
server.port: 5601
server.host: "192.168.20.100"
elasticsearch.hosts: ["http://192.168.20.100:9200"]

sudo systemctl daemon-reload
sudo systemctl enable --now kibana.service
sudo systemctl status kibana.service

Cài logstash:
sudo apt install logstash -y
sudo nano /etc/logstash/conf.d/logstash.conf

input {
  # 1. Cổng nhận Log từ tường lửa pfSense (Dùng giao thức Syslog)
  syslog {
    port => 5140
    type => "pfsense"
  }
  
  # 2. Cổng nhận Log từ các máy ảo Windows/Ubuntu (Dùng giao thức Beats)
  beats {
    port => 5044
  }
}

filter {
  # Hiện tại chúng ta cứ để trống bộ lọc. 
  # Log vào thế nào sẽ đẩy đi thế ấy để test kết nối trước.
}

output {
  elasticsearch {
    hosts => ["http://192.168.20.100:9200"]
    user => "elastic"
    password => "S-H+Pd65yy4EMkkkymgz"
    # Dòng dưới đây giúp Logstash tự động tạo tên Index dựa trên loại log gửi về
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}

sudo systemctl daemon-reload
sudo systemctl enable --now logstash.service

sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/logstash.conf
[Chèn hình elastic-UI]
Vào khu vực Quản lý dữ liệu (Stack Management)
Tại màn hình chính của Kibana, bạn bấm vào biểu tượng Menu (3 dấu gạch ngang ở góc trên cùng bên trái).

Cuộn chuột xuống tận dưới cùng của menu, tìm và bấm vào Stack Management (biểu tượng hình bánh răng).
Tạo Data View (Khai báo dữ liệu cho Kibana)
Kho Elasticsearch của bạn hiện tại đã nhận được log, nhưng giao diện Kibana cần bạn "chỉ điểm" tên đường dẫn thì nó mới biết cách lôi ra để hiển thị. (Trong các bản ELK cũ gọi là Index Pattern, từ bản 8.x trở đi gọi là Data View).


[Chèn hình kibana-dataview1]
Ở cột menu bên trái, nhìn dưới phần Kibana, bạn bấm vào Data Views.

Bấm nút Create data view (màu xanh ở góc phải).

[Chèn hình kibana-dataview2]
Một bảng cấu hình hiện ra, bạn điền thông tin như sau:

Name: Gõ tên hiển thị tùy thích, ví dụ: Logs Windows

Index pattern: Đây là mục quan trọng nhất để khớp với tên thư mục mà Logstash đã tạo. Bạn gõ chính xác: winlogbeat-*

Ngay khi bạn gõ xong ký tự *, nếu luồng dữ liệu của chúng ta thông suốt, Kibana sẽ hiện ngay một thông báo màu xanh lá cực kỳ phấn khởi: "Success! Your index pattern matches 1 source" (Thành công! Đã tìm thấy dữ liệu).

Kéo xuống mục Timestamp field, bạn click vào ô drop-down và chọn @timestamp (để hệ thống biết cách sắp xếp log theo thời gian thực).

Bấm nút Save data view to Kibana ở dưới cùng.



Tạo dashboard
[Chèn hình kibana-dashboard-create]
Bước 1: Mở xưởng vẽ (Kibana Lens)
Trên giao diện Kibana, bấm vào biểu tượng Menu (3 dấu gạch ngang) ở góc trên cùng bên trái.

Cuộn xuống mục Analytics, chọn Dashboard.

Bấm vào nút màu xanh Create dashboard (Tạo trang tổng quan mới).

Ở giữa màn hình trống, bạn bấm nút Create visualization (Tạo biểu đồ). Kibana sẽ mở công cụ Lens ra.

Bước 2: Lọc tạp âm (Chỉ lấy log tấn công)
Giống như bên Discover, chúng ta không muốn vẽ tất cả mọi thứ.

Ở thanh tìm kiếm (Search bar) trên cùng, bạn gõ câu lệnh quen thuộc:
event.code: 4625

Nhấn Enter. Lúc này biểu đồ sẽ chỉ tính toán những lần đăng nhập thất bại.

Đảm bảo góc trên bên phải đang chọn mốc thời gian là Last 15 minutes (hoặc thời điểm mà bạn vừa bắn Hydra).

Bước 3: Kéo thả để vẽ "Tội ác"
Bây giờ là lúc phép màu xuất hiện!

Kibana thường mặc định đã để trục ngang (X-axis) là @timestamp (Thời gian) và trục dọc (Y-axis) là Records (Số lượng). Bạn sẽ thấy một cột màu xanh nhô lên tổng hợp số lần bạn dội bom.

Để biết cột màu xanh đó là của IP nào, bạn nhìn sang Cột danh sách trường (Available fields) ở lề bên trái.

Gõ vào ô tìm kiếm (Search fields) chữ: IpAddress

Tìm đến trường có tên winlog.event_data.IpAddress (hoặc winlog.event_data.IpAddress.keyword).

Bạn nhấn giữ chuột vào nó, kéo và thả thẳng vào giữa cái biểu đồ trên màn hình!

BÙM! 🎇
Kibana Lens sẽ tự động chia nhỏ cột đó ra và gán cho nó một màu sắc riêng (ví dụ màu tím hoặc cam). Ở góc biểu đồ sẽ hiện ra chú thích (Legend): Tên IP 192.168.20.5 tương ứng với màu đó, và đỉnh cột sẽ chỉ mức 15, 20 hay hàng chục lần tùy vào số đạn Hydra bạn đã bắn.

Bước 4: Lưu tuyệt tác lại
Bạn nhìn sang cột ngoài cùng bên phải, tìm ô Title và đặt tên cho biểu đồ này, ví dụ: "Giám sát Tấn công RDP (Brute-force)".

Bấm nút Save and return (Lưu và quay lại) ở góc trên cùng bên phải.

Cuối cùng, bấm nút Save màu xanh ở góc phải trang Dashboard để lưu toàn bộ trang báo cáo này lại. Đặt tên là "SOC Dashboard - Main".

