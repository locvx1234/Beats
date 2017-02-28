
## 1.Giới thiệu 

Packetbeat là một bộ phân tích các gói tin mạng thời gian thực. Nó có thể kết hợp với Elasticsearch để cung cấp một ứng dụng phân tích hiệu năng và giám sát hệ thống. 

Packetbeat bắt các gói tin mạng lưu thông qua các ứng dụng server, giải mã theo các giao thức lớp ứng dụng  (HTTP, MySQL, Redis, ...), và biên dịch theo các các trường cụ thể. 

Xem thêm về các trường 

https://www.elastic.co/guide/en/beats/packetbeat/current/exported-fields.html

Packetbeat có thể giúp bạn dễ dàng biểu thị các vấn đề với ứng dụng back-end, như là bug, các vấn đề hiệu năng,...

Hiện tại Packetbeat hộ trợ những giao thức :

- ICMP (v4 and v6)
- DNS
- HTTP
- AMQP 0.9.1
- Cassandra
- Mysql
- PostgreSQL
- Redis
- Thrift-RPC
- MongoDB
- Memcache

Packetbeat có thể đẩy log trực tiếp về Elasticsearch hoặc qua các hệ thống trung gian như Redis, Logstash,...

## 2. Cài đặt

Với họ Debian, download và cài đặt từ gói như sau:

	sudo apt-get install libpcap0.8
	curl -L -O https://artifacts.elastic.co/downloads/beats/packetbeat/packetbeat-5.2.1-amd64.deb
	sudo dpkg -i packetbeat-5.2.1-amd64.deb
	
Với các distro khác, OS khác xem thêm:
	
https://www.elastic.co/guide/en/beats/packetbeat/current/packetbeat-installation.html	

## 3. Cấu hình 

Với các gói rpm và deb, file cấu hình sẽ ở đường dẫn /etc/filebeat/filebeat.yml

#### 3.1 Chọn các interface

Trên Linux, Packetbeat hỗ trợ bắt tất cả các message gửi và nhận bởi server.

	packetbeat.interfaces.device: any

#### 3.2 Cấu hình port cho các protocol 

Nếu giao thức sử dụng các port không giống mặc định, hãy thêm chúng ở đây. Nếu không thì giữ nguyên các giá trị mặc định : 

	packetbeat.protocols.dns:
	  ports: [53]

	  include_authorities: true
	  include_additionals: true

	packetbeat.protocols.http:
	  ports: [80, 8080, 8081, 5000, 8002]

	packetbeat.protocols.memcache:
	  ports: [11211]

	packetbeat.protocols.mysql:
	  ports: [3306]

	packetbeat.protocols.pgsql:
	  ports: [5432]

	packetbeat.protocols.redis:
	  ports: [6379]

	packetbeat.protocols.thrift:
	  ports: [9090]

	packetbeat.protocols.mongodb:
	  ports: [27017]

	packetbeat.protocols.cassandra:
	  ports: [9042]

#### 3.3 Xác định output 

Cấu hình cho Packetbeat đẩy log đến Elasticsearch hoặc các hệ thống khác. Ví dụ Logstash output: 


	#----------------------------- Logstash output --------------------------------
	output.logstash:
	  # The Logstash hosts
	  hosts: ["192.168.169.220:5044"]


Để test file cấu hình :

	/usr/bin/packetbeat.sh -configtest -e

	
## 4. Index template

Index template được sử dụng để map các trường khi phân tích log. 

File index template được cài đặt trong Packetbeat package. Nó sẽ tự động được load vào trong `packetbeat.yml` sau khi kết nối thành công đến Elasticsearch. 

Nếu muốn disable tự động load template, hoặc muốn sử dụng template tự tạo, bạn có thể thay đổi cài đặt trong file cấu hình. 

Khi đó, bạn phải thêm một số thông số như sau : 

	output.elasticsearch:
	  hosts: ["localhost:9200"]
	  template.name: "packetbeat"
	  template.path: "packetbeat.template.json"
	  template.overwrite: false


## 5. Khởi động Packetbeat

	sudo /etc/init.d/packetbeat

hoặc 

	service packetbeat start 
	
	
# 6. Cấu hình Logstash Output

Beat gửi data đến Logstash, Logstash nhận data bằng cách sử dụng `Beats input plugin`. 

Do đó cần cài thêm plugin này : 

	./bin/logstash-plugin install logstash-input-beats
	
Cấu hình Logstash lắng nghe tại port 5044 cho các kết nối tới từ Beats trong file `logstash.conf`

	input {
	  beats {
		port => 5044
	  }
	}

Nếu output là Elasticsearch :

	output {
	  elasticsearch {
		hosts => "localhost:9200"
		manage_template => false
		index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
		document_type => "%{[@metadata][type]}"
	  }
	}

Nếu output là Kafka :

	output {
			kafka {
					topic_id => "%{[@metadata][type]}"
					bootstrap_servers => ["192.168.169.221:9092"]
			}
			stdout { codec => rubydebug }
	}





	
