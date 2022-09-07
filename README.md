# Server Monitor Installation Instructions(Ubuntu 20.04)
### Created by Long 

####  Các ứng  dụng liên quan:
- Prometheus
- Node Exporter
- Grafana

##### Prometheus

**Prometheus** là một hệ thống giám sát (monitoring system) và bộ công cụ cảnh báo (alerting toolkit) mã nguồn mở. Dữ liệu sử dụng trong Prometheus là time series database, đây là một khái niệm sử dụng để mô tả những database chuyên dụng được sinh ra cho việc tối ưu lưu trữ dữ liệu theo các mốc thời gian, có thể kể đến như các thông số về tình trạng server, lượng tài nguyên RAM, CPU tiêu thụ, ... Các thông số này được lưu dưới dạng các mốc thời gian, từ những dữ liệu đó ta có thể xử lý để mô phỏng thành các biểu đồ giúp dễ dàng quan sát sự biến đổi của dữ liệu theo các mốc thời gian nhờ đó có thể nắm bắt được tình trạng của server để phát hiện những vấn đề và đưa ra cách giải quyết một cách kịp thời.

**Cách thức hoạt động của Prometheus**
![Prometheus](https://images.viblo.asia/5c075f8a-6a5b-4bbd-b756-ac4fe24dfde1.png)

##### Node Exporter
**Node Exporter** là một chương trình exporter viết bằng ngôn ngữ Golang. Exporter là một chương trình được sử dụng với mục đích thu thập, chuyển đổi các metric không ở dạng kiểu dữ liệu chuẩn Prometheus sang chuẩn dữ liệu Prometheus. Sau đấy exporter sẽ expose web service api chứa thông tin các metrics hoặc đẩy về Prometheus

**Cách thức hoạt động của Node Exporter**
![Prometheus](https://static.cuongquach.com/resources/images/2019/08/cai-dat-node-exporter-8.png)

##### Grafana
**Grafana** là một nền tảng khả năng quan sát mã nguồn mở để trực quan hóa các chỉ số, nhật ký và dấu vết được thu thập từ các ứng dụng của bạn. Đó là một giải pháp gốc đám mây để tập hợp nhanh chóng các bảng điều khiển dữ liệu cho phép bạn kiểm tra và phân tích ngăn xếp của mình.

Grafana kết nối với nhiều nguồn dữ liệu khác nhau như Prometheus, InfluxDB, ElasticSearch và các công cụ cơ sở dữ liệu quan hệ truyền thống. Trang tổng quan phức tạp được tạo bằng cách sử dụng các nguồn này để chọn các trường có liên quan từ dữ liệu của bạn. Trang tổng quan có thể kết hợp một loạt các thành phần trực quan hóa như biểu đồ, bản đồ nhiệt và biểu đồ.


#### Cách cài đặt 
##### Cài đặt Prometheus
.
```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.28.0/prometheus-2.28.0.linux-amd64.tar.gz
```

Giải nén file vừa được tải về
```sh
tar xvzf prometheus-2.28.0.linux-amd64.tar.gz
```
Di chuyển thư mục prometheus-2.28.0.linux-amd64 vừa được giải nén đến thư mục /opt/ và đổi tên nó thành prometheus
```sh
sudo mv -v prometheus-2.28.0.linux-amd64 /opt/prometheus
```
Thay đổi user và group của tất cả file và thư mục tại thư mục /opt/prometheus/ thành root
```sh
sudo chown -Rfv root:root /opt/prometheus
```
Thay đổi quyền cho thư mục /opt/prometheus
```sh
sudo chown -Rfv root:root /opt/prometheus
```
Mở file config prometheus.yml 
```sh
sudo nano /opt/prometheus/prometheus.yml
```
![Prometheus](https://linuxhint.com/wp-content/uploads/2021/07/install-prometheus-on-ubuntu-14.png)
Có thể thay đổi một số thuộc tính trong file theo ý muốn, nhưng với vẫn có thể chạy ổn nếu giữ nguyên

Tạo thư mục data/ trong thư mục /opt/prometheus
```sh
sudo mkdir -v /opt/prometheus/data
```
Thay đổi quyền cho file
```sh
sudo chown -Rfv prometheus:prometheus /opt/prometheus/data
```
Tạo một service systemd prometheus.service
```sh
sudo nano /etc/systemd/system/prometheus.service
```

Dán đoạn code phía dưới vào file và lưu lại 
```sh
[Unit]
Description=Monitoring system and time series database

[Service]
Restart=always
User=prometheus
ExecStart=/opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml --storage.tsdb.path=/opt/prometheus/data
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no
LimitNOFILE=8192

[Install]
WantedBy=multi-user.target
```

Chạy tiếp các lệnh sau để khởi động services
```sh
sudo systemctl daemon-reload

sudo systemctl start prometheus.service

sudo systemctl enable prometheus.service
```

Kiểm tra trạng thái của prometheus

```sh
sudo systemctl status prometheus.service
```

![Prometheus](https://linuxhint.com/wp-content/uploads/2021/07/install-prometheus-on-ubuntu-24.png)
Nếu màn hình hiển thị đúng như trên thì đã cài đặt và khởi động thành công 

***Truy cập vào localhost:9090 để vào trang Prometheus***

##### Cài đặt Node Exporter
.

Tải về file tar
```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
```

Giải nén file
```sh
tar xzf node_exporter-1.1.2.linux-amd64.tar.gz
```

Di chuyển thư mục vừa giải nén sang thư mục /usr/local/bin/
```sh
sudo mv -v node_exporter-1.1.2.linux-amd64/node_exporter /usr/local/bin/
```

Thay đổi quyền cho file
```sh
sudo chown root:root /usr/local/bin/node_exporter
```

Kiểm tra Node Exporter version để xem ứng dụng đã được tải thành công chưa
![Prometheus](https://linuxhint.com/wp-content/uploads/2021/07/install-prometheus-on-ubuntu-39.png)

Tạo file node-exporter.service
```sh
sudo nano /etc/systemd/system/node-exporter.service
```

Dán đoạn code dưới đây vào file service và lưu lại
```sh
[Unit]
Description=Prometheus exporter for machine metrics

[Service]
Restart=always
User=prometheus
ExecStart=/usr/local/bin/node_exporter
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

Chạy các lệnh sau để khởi động Node Exporter Services
```sh
sudo systemctl daemon-reload

sudo systemctl start node-exporter.service

sudo systemctl enable node-exporter.service
```

Kiểm tra trạng thái của node-exporter.service
```sh
sudo systemctl status node-exporter.service
```
![Prometheus](https://linuxhint.com/wp-content/uploads/2021/07/install-prometheus-on-ubuntu-45.png)
Nếu màn hình hiện trạng thái như trên thì đã khởi động thành công

Truy cập vào link http://localhost:9100/metrics nếu mọi thứ hoạt động đúng thì bạn sẽ thấy page như sau
![Prometheus](https://linuxhint.com/wp-content/uploads/2021/07/install-prometheus-on-ubuntu-48.png)

#### Thêm Node Exporter vào Prometheus

Mở file config của Prometheus
```sh
sudo nano /opt/prometheus/prometheus.yml
```

Thêm đoạn sau vào phía dưới dòng scrape_configs
```sh
- job_name: 'node_exporter'
    static_configs:
    - targets: ['192.168.20.131:9100']
```

![Prometheus](https://linuxhint.com/wp-content/uploads/2021/07/install-prometheus-on-ubuntu-50.png)

Khởi động lại prometheus.services
```sh
sudo systemctl restart prometheus.service
```

#### Cài đặt Grafana

apt-update
```sh
sudo apt update
```
Tải về các file pakages
```sh
sudo apt-get install -y gnupg2 curl software-properties-common
```
Add key
```sh
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
```
Tải về Grafana repository
```sh
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

Install Grafana
```sh
sudo apt -y install grafana
```

Khởi động Grafana service
```sh
sudo systemctl start grafana-server

sudo systemctl enable grafana-server
```
Kiểm tra trạng thái của services
```sh
sudo systemctl status grafana-server
```
Nếu trạng thái hoạt động bình thường thì Grafana sẽ hoạt động ở port 3000

Khi truy cập vào localhost:3000 thì sẽ có giao diện tương tự
![Prometheus](https://images.viblo.asia/6efa38ac-8166-4e3f-a97f-5b7a0dabeb50.png)

Vào configure để add soure từ prometheus 
![Prometheus](https://images.viblo.asia/56af1c6a-c338-4117-9aa6-580033b7cd5d.png)

Tiếp theo chúng ta tạo một Dashboard cho Node Exporter một cách đơn giản nhất là sử dụng những Dashboard đã được xây dựng sẵn. Có rất nhiều Dashboard đã được xây dựng sẵn với nhiều mục đich chúng ta có thể tìm kiếm được [Tại đây](https://grafana.com/grafana/dashboards)

Mình sử dụng một Dashboard cho Node Exporter có tên là Node Exporter Full các bạn có thể tìm kiếm tại địa chỉ trên. Dashboard này có ID 1860 chúng ta có thể lấy được dễ dàng thông qua URL.

Tiếp tục vào Dashboard => Import
![Prometheus](https://images.viblo.asia/944aeaed-2f3d-4563-9916-b6f4266b5e81.png)
Đưa ID vào và load sau đó nhấn import 

![Prometheus](https://images.viblo.asia/80a4ce6d-2728-45e2-9c16-94e9c76a402d.png)
Sau đó giao diện dashboard vừa được import sẽ xuất hiện








[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
