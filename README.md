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
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: shop
      MYSQL_USER: hieu
      MYSQL_PASSWORD: 1234
    ports:
      - "3306:3306"
    volumes:
      - ./mariadb/data:/var/lib/mysql
    networks:
      - ecommerce-network

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: root123
    ports:
      - "8080:80"
    depends_on:
      - mariadb
    networks:
      - ecommerce-network

  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: always
    environment:
      - TZ=Asia/Ho_Chi_Minh
    ports:
      - "1880:1880"
    user: "1000:1000"
    volumes:
      - ./node-red/data:/data
    depends_on:
      - mariadb
      - influxdb
    networks:
      - ecommerce-network
    command: >
      sh -c "
      npm install -g node-red-node-mysql &&
      node-red
      --httpNodeRoot=/api
      --httpAdminRoot=/nodered
      --functionGlobalContext.mysql=require('mysql').createPool({host:'mariadb',user:'hieu',password:'1234',database:'shop',port:3306,charset:'utf8mb4',connectionLimit:10})
      --functionGlobalContext.crypto=require('crypto')
      "

  influxdb:
    image: influxdb:2.7
    container_name: influxdb
    restart: always
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=admin123
      - DOCKER_INFLUXDB_INIT_ORG=ecommerce
      - DOCKER_INFLUXDB_INIT_BUCKET=statistics
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=my-super-secret-auth-token
    ports:
      - "8086:8086"
    volumes:
      - ./influxdb/data:/var/lib/influxdb2
    networks:
      - ecommerce-network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    environment:
      - GF_SERVER_HTTP_PORT=3000
      - GF_SERVER_ROOT_URL=http://nguyentrunghieu.com/grafana/
      - GF_SERVER_SERVE_FROM_SUB_PATH=true
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    ports:
      - "3000:3000"
    volumes:
      - ./grafana/data:/var/lib/grafana
    depends_on:
      - influxdb
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
      - ./nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
      - ./web:/usr/share/nginx/html:ro
    depends_on:
      - nodered
      - grafana
    networks:
      - ecommerce-network

networks:
  ecommerce-network:
    driver: bridge
```
3. Chạy lệnh `docker compose up -d`
<img width="2333" height="1042" alt="Screenshot 2025-11-04 174150" src="https://github.com/user-attachments/assets/34f64f04-46f0-42c8-a7c9-ba778adf26cd" />

Sau khi mọi thứ đã ổn, sẽ thấy các container chạy:
<img width="3069" height="1731" alt="image" src="https://github.com/user-attachments/assets/37d36bb3-d6d1-42b6-a3be-a5b45c94b0e0" />
<img width="3071" height="1744" alt="image" src="https://github.com/user-attachments/assets/ff976774-7703-4dad-a2ca-ebe3792e9cbc" />
<img width="3066" height="1736" alt="image" src="https://github.com/user-attachments/assets/007e7750-ea18-4c90-9613-843aa7801cdc" />
<img width="3070" height="1745" alt="image" src="https://github.com/user-attachments/assets/259b7743-5a00-43de-b7ac-05bad8fcb72d" />

### 3. CẤU HÌNH NGINX
1. Cấu trúc thư mục
```
webapplinux/
├── docker-compose.yml
└── nginx/
    └── conf.d/
        └── default.conf   ← file cấu hình nginx sẽ tạo ở đây
```
2. Chạy lệnh: `nano ~/webapplinux/nginx/conf.d/default.conf` trong Ubuntu
3. Nội dung file `default.conf`:
```
server {
    listen 80;
    server_name nguyentrunghieu.com www.nguyentrunghieu.com;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff2?|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # === API Backend (Node-RED) ===
    location /api/ {
        proxy_pass http://nodered:1880/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # === API User Orders (Node-RED) ===
    location /user/ {
        proxy_pass http://nodered:1880/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # === Node-RED UI (Subpath) ===
    location ^~ /nodered/ {
        proxy_pass http://nodered:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Fix tài nguyên tĩnh (CSS/JS) cho subpath
        sub_filter_once off;
        sub_filter 'href="/'  'href="/nodered/';
        sub_filter 'src="/'   'src="/nodered/';
        sub_filter 'action="/' 'action="/nodered/';
        sub_filter_types text/css text/javascript text/xml application/javascript;
        proxy_set_header Accept-Encoding "";
    }

    # === Grafana (Subpath) ===
    location /grafana/ {
        proxy_pass http://grafana:3000/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
    }
    # === 404 Fallback cho SPA ===
    error_page 404 /index.html;
}
```
4. `Ctrl+O` -> `Enter` -> `Ctrl +X` để lưu và thoát.
5. Chạy lệnh `docker compose down` -> `docker compose up -d` để cập nhật lại.

Kết quả sau khi cấu hình nginx, domain `nguyentrunghieu.com`:
<img width="3070" height="1743" alt="image" src="https://github.com/user-attachments/assets/b0968e2f-69a1-4b88-a584-fe1f91ad956f" />
### 4. TẠO FRONTEND & BACKEND
1. Tạo database trên phpMyAdmin
- Tạo các bảng trong db shop:

<img width="3048" height="1736" alt="image" src="https://github.com/user-attachments/assets/7f6d7e35-18fc-4546-92d0-e65ee79990cf" />

- Password được mã hóa: 

<img width="3070" height="933" alt="image" src="https://github.com/user-attachments/assets/9652fa30-eb9c-4d10-b63c-cd21323aa5cc" />

2. Tạo api backend qua NodeRed
- liệt kê các nhóm sản phẩm: `http://nguyentrunghieu.com/api/categories`

<img width="1572" height="264" alt="image" src="https://github.com/user-attachments/assets/65b7a6f1-0039-41e6-a733-190a3e5ad5ef" />

- lấy danh sách sản phẩm lọc theo nhóm: `http://nguyentrunghieu.com/api/products?category_id=1`

<img width="1571" height="167" alt="image" src="https://github.com/user-attachments/assets/81ae84ae-7447-4458-b8e3-8d28eec32a22" />

<img width="1293" height="1081" alt="image" src="https://github.com/user-attachments/assets/1a212cd3-5f98-4eb5-a698-fb0abb6803bd" />

- tìm kiếm sản phẩm : `http://nguyentrunghieu.com/api/search?q=iphone`

<img width="1521" height="134" alt="image" src="https://github.com/user-attachments/assets/61bbdd0f-06cb-4357-bcd3-2db6123686d6" />

<img width="1264" height="480" alt="image" src="https://github.com/user-attachments/assets/65afa3b6-d3cf-4cc3-8d44-8128f271a2a4" />

- login + lưu session
 
<img width="1995" height="136" alt="image" src="https://github.com/user-attachments/assets/a59d3b20-22ac-42c9-afb8-98ed4e362f41" />

- liệt kê các sản phẩm bán chạy:`http://nguyentrunghieu.com/api/top-products`

<img width="1608" height="194" alt="image" src="https://github.com/user-attachments/assets/96f75771-6a15-40b5-bb16-12e0a663383e" />

<img width="1142" height="1648" alt="image" src="https://github.com/user-attachments/assets/5b34673a-61d8-4025-aef1-12a30ceb9237" />

- quản lý giỏ hàng

<img width="1599" height="507" alt="image" src="https://github.com/user-attachments/assets/86fb2580-40b0-4f07-8109-0e5ca3f5efd9" />

- đặt hàng

<img width="1996" height="580" alt="image" src="https://github.com/user-attachments/assets/e8a1c1c3-3bf2-4219-aad4-94a7f05d324b" />


