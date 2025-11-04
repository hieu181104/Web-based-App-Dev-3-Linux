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
#### Bước 1: Kích hoạt Hyper-V trên máy
- Mở Control Panel -> Program -> Turn Windows features on or off
- Chọn: 
  - Hyper-V
  - Virtual Machine Platform
  - Windows Subsystem for Linux
- Nhấn OK -> Restart máy tính
#### Bước 2: Cài đặt Ubuntu trên Hyper-V
- Mở Hyper-V Manager
- Click chuột phải vào tên máy chọn New -> Virtual Machine
  Thiết lập các thông số:
  - Name:
  - Generation:
  - Memory:
  - Network:
  - Virtual Hard Disk:
  - Installation Options: "Install an operating system from a bootable CD/DVD-ROM" -> Image file (.iso) -> Chọn file ISO Ubuntu đã tải.
- Hoàn tất wizard, click chuột phải VM -> Connect -> Start
- Trong cửa sổ VM, cài Ubuntu. Chọn ngôn ngữ English. Tạo user + password
- Sau khi cài xong, login Ubuntu
- Mở Terminal (Ctrl + Alt + T) chạy: 
  ```sudo apt update && sudo apt upgrade -y```
- Sau khoảng 5 phút, Nếu xuất hiện câu hỏi Y/N, gõ Y -> Enter
- Cài đặt thêm tools : ```sudo apt install curl wget -y```
- Test Ubuntu đã chạy hay chưa: chạy `lsb_release -a`

#### Bước 3: Cài đặt Docker & Docker Compose
