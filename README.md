# Chi tiết các bước triển khai RustDesk Server lên Ubuntu Linux 
Để cài đặt RustDesk Server chuẩn bảo mật và hạn chế tối đa lỗi phát sinh trong tương lai, phương án tốt nhất là sử dụng Docker. Cách này giúp cô lập môi trường của RustDesk khỏi hệ điều hành, ngăn chặn xung đột phần mềm và dễ dàng sao lưu hoặc nâng cấp sau này.

## Cập nhật hệ thống và cấu hình Tường lửa (UFW) để bảo mật cổng mạng cho Server. 
1. Đầu tiên, cập nhật các gói phần mềm và mở các cổng mạng mà RustDesk yêu cầu.
```bash
sudo apt update && sudo apt upgrade -y
sudo ufw allow 21115:21119/tcp
sudo ufw allow 21116/udp
sudo ufw enable
```
** Để kiểm tra bước này đã thành công, chạy lệnh ```bash sudo ufw status```. Bạn sẽ thấy trạng thái là "active" và danh sách các cổng 22, 21115-21119 (tcp), 21116 (udp) được dán nhãn "ALLOW".
2. Cài đặt Docker và Docker Compose để cô lập môi trường chạy các dịch vụ của RustDesk.
```bash
sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
```
Để xác nhận Docker đã được cài đặt và đang hoạt động, hãy chạy lệnh ```docker --version.``` Hệ thống sẽ trả về thông tin phiên bản Docker hiện tại.
3. Tạo cấu hình RustDesk bằng các thiết lập file docker-compose.yml. Tạo một thư mục riêng biệt cho RustDesk và tạo file cấu hình.

```bash
sudo mkdir -p /opt/rustdesk/data
cd /opt/rustdesk
sudo nano docker-compose.yml
```
Dán nội dung sau vào file docker-compose.yml. Lưu ý: Bạn phải thay thế ĐỊA_CHỈ_IP_SERVER_LINUX bằng IP thật của server trước khi lưu.

```bash
version: '3'
networks:
  rustdesk-net:
    external: false
services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
    image: rustdesk/rustdesk-server:latest
    # -r chỉ định IP nội bộ. -k _ ép buộc client phải có Key mới được kết nối
    command: hbbs -r ĐỊA_CHỈ_IP_SERVER_LINUX:21117 -k _
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped
  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
    image: rustdesk/rustdesk-server:latest
    # Bắt buộc mã hóa
    command: hbbr -k _
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    restart: unless-stopped
```

Bấm Ctrl+O --> sau đó bấm Enter để lưu và Ctrl+ X để thoát.

4. Khởi tạo và chạy các container theo định nghĩa trong file docker-compose.yml bằng lệnh:
```bash   
sudo docker-compose up -d
```
Chờ đợi để hệ thống tạo khóa tự động, sau đó đọc khóa công khai (Public Key) bằng lệnh:

```bash  
cat /opt/rustdesk/data/id_ed25519.pub
```
Chúng ta sẽ lưu  trữ key này lại và dùng nó để cấu hình trong ứng dụng RustDesk được cài trên máy tính của Client. 


5. Kiểm tra hệ thống của bạn đã cài đặt và khởi động thành công hay chưa bằng lệnh: 
```bash  
sudo docker ps
```
Nếu bạn thấy danh sách hiện ra hai container có tên là hbbs và hbbr với trạng thái "Up" -> Cài đặt thành công.
