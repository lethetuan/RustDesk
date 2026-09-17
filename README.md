# Chi tiết các bước triển khai RustDesk Server lên Ubuntu Linux 
Để cài đặt RustDesk Server chuẩn bảo mật và hạn chế tối đa lỗi phát sinh trong tương lai, phương án tốt nhất là sử dụng Docker. Cách này giúp cô lập môi trường của RustDesk khỏi hệ điều hành, ngăn chặn xung đột phần mềm và dễ dàng sao lưu hoặc nâng cấp sau này.

### 1. Đầu tiên, cập nhật các gói phần mềm và mở các cổng mạng Tường lửa (UFW) mà RustDesk yêu cầu.

```bash
sudo apt update && sudo apt upgrade -y
```

cài đặt dịch vụ đồng bộ thời gian

```bash
sudo apt install systemd-timesyncd -y
sudo systemctl enable --now systemd-timesyncd
```

mở các cổng mạng Tường lửa (UFW)

```bash
sudo apt update && sudo apt upgrade -y
sudo ufw allow 22/tcp
sudo ufw allow 21115:21117/tcp
sudo ufw allow 21116/udp
sudo ufw enable
```
** Để kiểm tra bước này đã thành công, chạy lệnh ``` sudo ufw status```. Bạn sẽ thấy trạng thái là "active" và danh sách các cổng 22, 21115-21117 (tcp), 21116 (udp) được dán nhãn "ALLOW".
### 2. Cài đặt Docker và Docker Compose để cô lập môi trường chạy các dịch vụ của RustDesk.

Chạy lệnh thiết lập kho lưu trữ của Docker. Cài đặt khóa bảo mật và đường dẫn tải. Bước này giúp máy chủ của bạn nhận diện và tin cậy nguồn tải từ Docker:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ca-certificates curl gnupg -y

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Sau khi đã thêm nguồn, bạn tiến hành cài đặt các gói lõi Docker và Compose v2:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Cấu hình khởi động cùng hệ thống. Tự động chạy lại docker khi khởi động lại Server

```bash
sudo systemctl enable --now docker
```

** Để xác nhận bạn đã cài đặt đúng phiên bản v2, hãy chạy lệnh kiểm tra:  ```docker compose version``` Hệ thống sẽ trả về thông tin phiên bản Docker hiện tại.
(Nếu cài đặt thành công, hệ thống sẽ trả về kết quả có dạng: Docker Compose version v2.x.x)

### 3. Tạo cấu hình RustDesk bằng các thiết lập file docker-compose.yml. Tạo một thư mục riêng biệt cho RustDesk và tạo file cấu hình.

```bash
sudo mkdir -p /opt/rustdesk/data
cd /opt/rustdesk
sudo nano docker-compose.yml
```
Dán nội dung sau vào file docker-compose.yml. Lưu ý: Bạn phải thay thế ĐỊA_CHỈ_IP_SERVER_LINUX bằng IP thật của server trước khi lưu.

```bash
version: '3'
services:
  hbbs:
    container_name: hbbs
    image: rustdesk/rustdesk-server:latest
    network_mode: "host"
    command: hbbs -r ĐỊA_CHỈ_IP_SERVER_LINUX:21117 -k _
    volumes:
      - ./data:/root
    environment:
      - TZ=Asia/Ho_Chi_Minh
    depends_on:
      - hbbr
    restart: unless-stopped
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "3"
        
  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server:latest
    network_mode: "host"
    command: hbbr -k _
    volumes:
      - ./data:/root
    environment:
      - TZ=Asia/Ho_Chi_Minh
    restart: unless-stopped
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "3"
```

Bấm Ctrl + O --> sau đó bấm Enter để lưu và Ctrl + X để thoát.

### 4. Khởi tạo và chạy các container theo định nghĩa trong file docker-compose.yml bằng lệnh:
```bash   
sudo docker compose up -d
```
Kiểm tra hệ thống của bạn đã cài đặt và khởi động thành công hay chưa bằng lệnh: 
```bash  
sudo docker ps
```
Nếu bạn thấy danh sách hiện ra hai container có tên là hbbs và hbbr với trạng thái "Up" -> Cài đặt thành công.
Mặc định sẽ có file *.pub được lưu trữ trong thư mục /opt/rustdesk/data/ chúng ta sẽ dùng lệnh cat để đọc file này để xem nội dung key trong file.
Chúng ta sẽ lưu trữ nội dung key này lại và dùng nó để cấu hình trong ứng dụng RustDesk được cài trên máy tính của Client. 
Lệnh để đọc file *.pub

```bash
cd /opt/rustdesk/data
ls -l  
cat /opt/rustdesk/data/id_ed25519.pub
```



### 5. Cài đặt ứng dụng RustDesk lên máy tính client của các end-user.

Truy cập vào trang chủ : https://rustdesk.com/ để tải file và cài đặt lên máy tính. Sau khi cài đặt chúng ta bấm vào nút 3 chấm tại dòng ID như hình dưới để cấu hình
<img width="790" height="600" alt="image" src="https://github.com/user-attachments/assets/50e66086-fa29-4489-845c-be562677ac12" />

Click vào chức năng "Network" sau đó click vào ID/Relay server để cấu hình.
<img width="919" height="761" alt="image" src="https://github.com/user-attachments/assets/daaafc9d-5795-46bd-8ac4-dafa8aa7eb46" />

Điền thông tin cấu hình như địa chỉ IP của server, public key đã lưu trữ ở bước 4 và click vào nút OK để lưu.
<img width="555" height="351" alt="image" src="https://github.com/user-attachments/assets/43b69adb-ae31-48f4-aa7f-d80e200dfeee" />

Giờ chúng ta có thể sử dụng RustDesk để remote các máy tính client rồi.


