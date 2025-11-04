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
Bài tập này em sử dụng phần mềm VMware để cài Ubuntu.
#### Bước 1: Tải xuống file .iso của Ubuntu từ trang web: `https://ubuntu.com/`
<img width="1402" height="61" alt="image" src="https://github.com/user-attachments/assets/d7ee4d40-4abc-45ae-86de-fcdaff6b62de" />

#### Bước 2: Cài đặt Ubuntu trên máy ảo VMware
Tạo một máy ảo mới, chọn đường dẫn đặt máy ảo:

<img width="2306" height="1279" alt="Screenshot 2025-11-04 145802" src="https://github.com/user-attachments/assets/9a57d54a-99ae-4b19-8ae7-61bfdf827644" />

Thiết lập các thông số cho máy ảo, rồi chọn Finish:

<img width="2303" height="1291" alt="Screenshot 2025-11-04 145857" src="https://github.com/user-attachments/assets/f6c0f784-f374-44dc-b2a5-fd76f69fcef4" />

<img width="2303" height="1286" alt="Screenshot 2025-11-04 145912" src="https://github.com/user-attachments/assets/a5696268-0412-451a-976e-eda4fe172d93" />

Chọn Edit máy ảo và gán đường dẫn file iso vào -> Bấm OK -> Chạy máy ảo:

<img width="1485" height="1423" alt="Screenshot 2025-11-04 145955" src="https://github.com/user-attachments/assets/a655b795-cc9e-491f-8bad-91786c26fa90" />

Cài đặt và cấu hình Ubuntu:

Chọn ngôn ngữ English:

<img width="1291" height="805" alt="Screenshot 2025-11-04 150201" src="https://github.com/user-attachments/assets/8ab9f3b4-50da-4b04-8758-45013830ab5c" />

Thiết lập tài khoản và mật khẩu:

<img width="1278" height="813" alt="Screenshot 2025-11-04 150338" src="https://github.com/user-attachments/assets/2c35638c-d76d-4ed0-be84-e001c4ef4ceb" />

Thêm địa chỉ, vị trí:

<img width="1284" height="799" alt="Screenshot 2025-11-04 150433" src="https://github.com/user-attachments/assets/bdd7f84c-1f39-45d9-8c3e-d518b2a20eb3" />

Kiểm tra thông tin và tiến hành cài đặt Ubuntu -> Install:
<img width="1281" height="809" alt="Screenshot 2025-11-04 150632" src="https://github.com/user-attachments/assets/4ede94f8-fede-47f2-9710-c984eb0cc61e" />

