
## 1.Giới thiệu 

Metricbeat là một shipper thu thập các số liệu từ hệ điều hành và các service đang chạy trên server. Metricbeat có thể thu thập số liệu và đẩy chúng sang Elasticsearch hoặc các nơi lưu trữ khác như Logstash, Kafka, Redis.

Metricbeat giám sát theo các module : 

- Apache
- HAProxy
- MongoDB
- MySQL
- Nginx
- PostgreSQL
- Redis
- System
- Zookeeper


## 2. Cài đặt

Với họ Debian, download và cài đặt từ gói như sau:

	curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.2.2-amd64.deb
	sudo dpkg -i metricbeat-5.2.2-amd64.deb
	
Với các distro khác, OS khác xem thêm:

https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation.html


## 3. Cấu hình 

Với các gói rpm và deb, file cấu hình sẽ ở đường dẫn /etc/metricbeat/metricbeat.yml

Metricbeat sử dụng các module để thu thập số liệu. Mỗi module được cấu hình riêng rẽ.

Ví dụ về cấu hình module `system` và `apache` trong file `metricbeat.yml`

	metricbeat.modules:
	- module: system
	  metricsets:
		- cpu
		- filesystem
		- memory
		- network
		- process
	  enabled: true
	  period: 10s
	  processes: ['.*']
	  cpu_ticks: false
	- module: apache
	  metricsets: ["status"]
	  enabled: true
	  period: 1s
	  hosts: ["http://127.0.0.1"]
	  
	  
Nếu muốn gửi output qua Elasticsearch, cấu hình ip và port ở phần cấu hình sau: 

	output.elasticsearch:
	  hosts: ["ip-elasticsearch:9200"]
	  
Nếu muốn gửi output qua Logstash, comment phần cấu hình cho Elasticsearch bên trên và uncomment phần cấu hình cho Logstash:


	output.logstash:
	  # The Logstash hosts
	  hosts: ["ip-logstash:5044"]
	  template.name: "metricbeat"
	  template.path: "metricbeat.tempolate.json"
	  template.overwrite: false


## 4. Index template

Index template được sử dụng để map các trường khi phân tích log. 

File index template được cài đặt trong Metricbeat package. Nó sẽ tự động được load vào trong `metricbeat.yml` sau khi kết nối thành công đến Elasticsearch. 

Mặc định, Metricbeat sử dụng file `metricbeat.template.json`

Nếu muốn disable tự động loading template, hoặc muốn sử dụng template tự tạo, bạn có thể thay đổi cài đặt trong file cấu hình. 

Khi đó, bạn phải thêm một số thông số như sau : 

	output.elasticsearch:
	  hosts: ["localhost:9200"]
	  template.name: "metricbeat"
	  template.path: "metricbeat.template.json"
	  template.overwrite: false
	  
## 5. Khởi động Metricbeat

	# /etc/init.d/metricbeat start

hoặc

	# service metricbeat start


