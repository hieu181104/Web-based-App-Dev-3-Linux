# BÀI TẬP 3 - PHÁT TRIỂN ỨNG DỤNG TRÊN NỀN WEB - LINUX
#### Giảng viên: Đỗ Duy Cốp
#### Lớp học phần: K58KTP
#### Sinh viên thực hiện: Nguyễn Trung Hiếu
#### Mã sinh viên: K225480106019
## CHỦ ĐỀ: LẬP TRÌNH WEB THƯƠNG MẠI ĐIỆN TỬ TRÊN NỀN LINUX
### 1. GIỚI THIỆU
Bài tập này xây dựng một ứng dụng web thương mại điện tử dạng Single Page Application, triển khai trên Linux, chạy trên Hyper-V

Bài tập này sử dụng Docker Compose để cài đặt và quản lý các container:

- mariadb (3306) : cơ sở dữ liệu lưu user, đơn hàng, các sản phẩm,...
- phpmyadmin (8080) : giao diện quản trị database
- nodered (1880) : backend xử lý request trả về json
- influxdb (8086) : lưu lịch sử
- grafana (3000) : hiển thị đồ thị, thống kê lịch sử bán hàng
- nginx (80,443) : web server

### 2. CÀI ĐẶT MÔI TRƯỜNG LINUX
#### Bước 1: Kích hoạt WSL và cài đặt Ubuntu
1. Mở PowerShell (Run as Administrator) chạy: `wsl --install -d Ubuntu`
<img width="1832" height="1023" alt="Screenshot 2025-11-04 132000" src="https://github.com/user-attachments/assets/d20f4092-af75-4b4f-b82d-792b321bc952" />

2. Sau khi cài xong khởi động lại máy 
3. Mở ứng dụng Ubuntu, thiết lập username và password:
<img width="2338" height="1211" alt="Screenshot 2025-11-04 132657" src="https://github.com/user-attachments/assets/c1f8959a-5c0b-45e6-b974-22c5ef0d62de" />

4. Cập nhật hệ thống : `sudo apt update && sudo apt upgrade -y`
5. Cài thêm tool cơ bản: `sudo apt install curl wget -y`
6. Kiểm tra Ubuntu: `lsb_release -a`
<img width="1956" height="217" alt="Screenshot 2025-11-04 134608" src="https://github.com/user-attachments/assets/56573f11-27f0-4029-9d6d-af970e5e3b43" />

#### Bước 2: Cài đặt Docker & Docker Compose
1. Trong Ubuntu, chạy lệnh:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
2. Thêm user vào group để chạy docker không cần sudo :`sudo usermod -aG docker $USER`
3. Áp dụng thay đổi:  `sudo reboot`
4. Test thử:
`docker run hello-world`
<img width="2338" height="829" alt="Screenshot 2025-11-04 170303" src="https://github.com/user-attachments/assets/3a1e037f-fdfd-4793-86ad-8f89aacdb558" />

Nếu thấy "Hello from Docker!" là thành công.

5. Thêm repo docker:
```sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose  ```

<img width="2320" height="306" alt="Screenshot 2025-11-04 170422" src="https://github.com/user-attachments/assets/05fc7b05-7c4a-42a2-bbfa-336d7149d984" />

6. Thay đổi quyền thực thi cho file: `sudo chmod +x /usr/local/bin/docker-compose`
7. Kiểm tra Docker: `docker compose version`
#### Bước 3: Cấu hình Docker Compose
1. Tạo thư mục và chuyển đến nó
```
mkdir webapplinux
cd webapplinux
```
2. Tạo file `nano docker-compose.yml`:
```
version: '3.9'

networks:
  ecommerce-network:
    driver: bridge

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: shop
      MYSQL_USER: hieu
      MYSQL_PASSWORD: 1234
    volumes:
      - mariadb_data:/var/lib/mysql
    expose:
      - "3306"
    networks:
      - ecommerce-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    depends_on:
      - mariadb
    environment:
      PMA_HOST: mariadb
      PMA_USER: hieu
      PMA_PASSWORD: 1234
    expose:
      - "80"
    networks:
      - ecommerce-network

  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: always
    expose:
      - "1880"
    volumes:
      - nodered_data:/data
    networks:
      - ecommerce-network

  influxdb:
    image: influxdb:latest
    container_name: influxdb
    restart: always
    expose:
      - "8086"
    volumes:
      - influxdb_data:/var/lib/influxdb
    networks:
      - ecommerce-network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    depends_on:
      - influxdb
    expose:
      - "3000"
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - ecommerce-network

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./frontend:/usr/share/nginx/html
    depends_on:
      - nodered
      - grafana
      - phpmyadmin
    networks:
      - ecommerce-network

volumes:
  mariadb_data:
  nodered_data:
  influxdb_data:
  grafana_data:

```
3. Chạy lệnh `docker compose up -d`
<img width="2333" height="1042" alt="Screenshot 2025-11-04 174150" src="https://github.com/user-attachments/assets/34f64f04-46f0-42c8-a7c9-ba778adf26cd" />

Sau khi mọi thứ đã ổn, sẽ thấy các container chạy:
<img width="3069" height="1731" alt="image" src="https://github.com/user-attachments/assets/37d36bb3-d6d1-42b6-a3be-a5b45c94b0e0" />
<img width="3062" height="1738" alt="image" src="https://github.com/user-attachments/assets/91e4ff3a-cd84-4459-9d64-408a72de59c1" />
<img width="3066" height="1736" alt="image" src="https://github.com/user-attachments/assets/007e7750-ea18-4c90-9613-843aa7801cdc" />
<img width="3070" height="1745" alt="image" src="https://github.com/user-attachments/assets/259b7743-5a00-43de-b7ac-05bad8fcb72d" />

#### Bước 4: Cấu hình Nginx
1. Cấu trúc thư mục
```
webapplinux/
├── docker-compose.yml
└── nginx/
    └── conf.d/
        └── default.conf   ← file cấu hình nginx sẽ tạo ở đây
```
2. Chạy lệnh: `nano ~/webapplinux/nginx/conf.d/default.conf` trong Ubuntu
3. Nội dung file default.conf:
```
server {
    listen 80;
    server_name nguyentrunghieu.com;

    # Trang web chính (frontend)
    root /usr/share/nginx/html;
    index index.html;

    # Gửi các yêu cầu JS/HTML đến frontend (SPA)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy đến Node-RED (qua cổng 1880)
    location /nodered/ {
        proxy_pass http://nodered:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Proxy đến Grafana (qua cổng 3000)
    location /grafana/ {
        proxy_pass http://grafana:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    # Xử lý lỗi
    error_page 404 /index.html;
  }
```
