Filebeat là một log data shipper, được cài đặt như một agent trên server. Filebeat monitor các thư mục log hoặc các file xác định, và chuyển chúng qua Elasticsearch hay Logstash để indexing.

## 1. Hoạt động của Filebeat 

Filebeat sẽ bắt đầu với một hoặc nhiều prospector để tìm các đường dẫn mà bạn xác định vị trí các file log. Với mỗi file log xác định trên prospector đó, Filebeat bắt đầu harvester để đọc một file log đơn có các nội dung mới và gửi data log mới này cho spooler. 

Spooler tập hợp các sự kiện này và gửi data của các sự kiện cho output - (Elasticsearch,Logstash,Kafka or ...)

<img src="https://www.elastic.co/guide/en/beats/filebeat/current/images/filebeat.png">

## 2. Cài đặt 

Với họ Debian, download và cài đặt từ gói như sau:

	curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.2.1-amd64.deb
	sudo dpkg -i filebeat-5.2.1-amd64.deb
	
Với các distro khác, OS khác xem thêm 

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html

## 3. Cấu hình 

Với các gói `rpm` và `deb`, file cấu hình sẽ ở đường dẫn `/etc/filebeat/filebeat.yml`

#### 3.1 Xác định path (or paths) cho các file log

Ví dụ cấu hình cơ bản cho một prospector đơn với một path đơn 

	filebeat.prospectors:
	- input_type: log
	  paths:
		- /var/log/*.log
		
Trong ví dụ này, prospector sẽ harvest tất cả các file đuôi `.log` ở đường dẫn /var/log/ 

#### 3.2 Gửi output tới Elasticsearch

Mặc định Filebeat cấu hình gửi output cho Elasticsearch, bạn cần để ý đến ip-elasticsearch và port của nó

	output.elasticsearch:
		hosts: ["ip-elasticsearch:9200"]
		
#### 3.3 Gửi output tới Logstash
Để gửi output đến Logstash, bạn phải comment dòng cấu hình cho Elasticsearch (phần trên)

Sau đó uncomment dòng liên quan đến Logstash, chú ý phải thay đổi ip-logstash và port hợp lý

	#----------------------------- Logstash output --------------------------------
	output.logstash:
	  hosts: ["ip-logstash:5044"]

Để test file cấu hình : 

	/usr/bin/filebeat.sh -configtest -e
	
## 4. Khởi động Filebeat

	sudo /etc/init.d/filebeat start
	
Hoặc

	service filebeat start 