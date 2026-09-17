### Bước 1 Tải file cài đặt từ trang chủ https://rustdesk.com/ về máy tính. Sau đó cài đặt ứng dụng rustdesk.

Click vào nút 3 chấm như hình dưới để tiến hành cài đặt
<img width="659" height="589" alt="image" src="https://github.com/user-attachments/assets/57964e41-daf3-4a11-b276-c00c069168a0" />

<img width="875" height="591" alt="image" src="https://github.com/user-attachments/assets/373c5d8b-cfd0-4918-8a7b-725d4bcbfc1d" />

Tinh chỉnh cài đặt trong mục "Chung" theo nhu cầu sử dụng
<img width="879" height="581" alt="image" src="https://github.com/user-attachments/assets/64d65f49-a584-4d9a-8c37-561a746f177c" />

Tại mục "Bảo mật" chúng ta sẽ chọn Quyền với mục "Toàn quyền truy cập"
<img width="977" height="597" alt="image" src="https://github.com/user-attachments/assets/75e5f65f-4b51-4071-9d79-ae7c75445225" />

Lăn chuột xuống dưới để tiếp tục cài đặt "Bảo mật". Mục mật khẩu chọn "Chấp nhận phiên qua cả 2" để cho phép remote bằng địa chỉ IP và ID ứng dụng. CLick "Dùng mật khẩu vĩnh viễn" để cài đặt mật khẩu cho ứng dụng dụng

<img width="777" height="577" alt="image" src="https://github.com/user-attachments/assets/aed9d0b2-0471-4e74-af4b-e7277537f184" />


Chúng ta sẽ điền mật khẩu cho ứng dụng. Mỗi lần kết nối sẽ dùng mật khẩu này để kết nối đến máy tính cần remote
<img width="1233" height="723" alt="image" src="https://github.com/user-attachments/assets/259ac18c-b273-4bd3-a40c-2589a1a41489" />

Lăn chuột xuống dưới để tiếp tục cài đặt "Bảo mật". Tại mục "Bảo mật" ở dưới cùng chúng ta sẽ click theo thứ tự và tại nút tích số 6 chúng ta sẽ đặt mã PIN. Mã PIN này dùng để quản lý các cài đặt trong mục "Bảo mật". nếu muốn thay đổi cấu hình trong ứng dụng bắt buộc phải nhập mã PIN này. Nếu không có mã PIN sẽ không thay đổi cập nhật được.

<img width="919" height="757" alt="image" src="https://github.com/user-attachments/assets/5da0aa7f-dc19-4f93-917b-6d99541cd916" />

Sau khi thiết lập mã PIN xong chúng ta chuyển xuống cấu hình "Mạng". Tại đây chúng ta sẽ click vào nút "Mở khóa cài đặt mạng" và nhập mã PIN mới thiết lập ở trên để cài đặt các thông số mạng. 

<img width="927" height="649" alt="image" src="https://github.com/user-attachments/assets/60f7a833-6ad4-49e4-838b-b15ac7841a45" />


Sau khi nhập mã PIN chúng ta sẽ click vào nút "Máy chủ ID/Chuyển tiếp" để tiếp tục cài đặt cấu hình mạng

<img width="1057" height="585" alt="image" src="https://github.com/user-attachments/assets/20f63768-1c41-48ff-85ad-ab91ce3ab190" />

Chúng ta sẽ tiến hành khai báo các thông số như địa chỉ IP của RustDesk Server đã triển khai, key bảo mật được lưu trữ trong thư mục /opt/rustdesk/data/id_ed25519.pub như hình dưới sau đó nhấn OK để lưu lại.
<img width="981" height="667" alt="image" src="https://github.com/user-attachments/assets/8a914561-49a5-4d36-ab3d-0425a3d1b9b0" />

Tại mục cấu hình "Hiển thị" chúng ta cấu hình như hình dưới.
<img width="1097" height="763" alt="image" src="https://github.com/user-attachments/assets/f7f4db59-e95a-44ed-a907-59728cd94baa" />

Chúng ta sẽ tiến hành lưu trữ lại cấu hình vừa mới thiết lập bằng cách truy cập đường dẫn : C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\config
Sau đó copy tập tin RustDesk2.toml và lưu trữ lại.

<img width="781" height="403" alt="image" src="https://github.com/user-attachments/assets/d7f54da1-8763-4482-af27-d9826d49c79e" />

### Bước 2: Thực hiện trên Domain Controller để triển khai ứng dụng đến hàng loạt máy tính trong hệ thống

1. Tạo một thư mục chia sẻ (ví dụ: Share) trong ổ đĩa C:\ phân quyền Read cho Domain User. Tải file cài đặt rustdesk-1.4.9-x86_64.msi và sao chép file rustdesk-1.4.9-x86_64.msi cùng file RustDesk2.toml vào thư mục Share. Sau đó lưu đoạn mã dưới thành file Install_RustDesk.bat vào thư mục bất kỳ. Lưu ý phải thay đổi các thông tin trong file Install_RustDesk.bat này cho phù hợp với hệ thống của các bạn.

```bash
@echo off
REM --- BUOC 1: Kiem tra xem may da cai RustDesk chua ---
REM Neu file rustdesk.exe da ton tai, bo qua buoc cai dat (nhay den phan copy cau hinh)
IF EXIST "C:\Program Files\RustDesk\RustDesk.exe" GOTO copy_config

REM --- BUOC 2: Cai dat ngam file MSI (Bo lenh timeout) ---
REM Lenh "start /wait" bat buoc he thong cho cai dat xong 100% moi di tiep
start /wait msiexec.exe /i "\\dc01\Share\rustdesk-1.4.9-x86_64.msi" /qn

:copy_config
REM --- BUOC 3: Tam dung Service de tranh loi file dang su dung ---
net stop RustDesk

REM --- BUOC 4: Tao thu muc (phong truong hop chua co) ---
if not exist "C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\config" mkdir "C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\config"

REM --- BUOC 5: Chép dè c?u hình chu?n ---
copy /Y "\\dc01\Share\RustDesk2.toml" "C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\config\"

REM --- 4. Thiet lap MAT KHAU REMOTE CHUNG bang dong lenh ---
"C:\Program Files\RustDesk\RustDesk.exe" --password "Matkhauremote@1234567"

REM --- BUOC 6: Khoi dong lai Service ---
net start RustDesk
```


2.Tạo Group Policy Object (GPO) mới: Server Manager.Mở Group Policy Management (gpmc.msc). Click chuột phải vào tên Domain (hoặc OU chứa các máy tính cần cài đặt) -> Chọn Create a GPO in this domain, and Link it here.... Đặt tên cho GPO (ví dụ: Deploy_Software).
<img width="1439" height="655" alt="image" src="https://github.com/user-attachments/assets/fe7bebf1-4f21-4e3f-9e0f-da7d5f5a47e2" />


Cấu hình Startup Script:  Click chuột phải vào GPO vừa tạo -> Edit. Điều hướng theo đường dẫn: Computer Configuration -> Policies -> Windows Settings -> Scripts (Startup/Shutdown). Nhấp đúp chuột vào mục Startup. 

<img width="1179" height="693" alt="image" src="https://github.com/user-attachments/assets/e850193b-6a97-43cc-9bde-d5c00420c3ca" />

Khai báo file .bat vào GPO: Trong tab Scripts, bấm nút Show Files.... Một thư mục ẩn của chính sách sẽ hiện ra. Chép file Install_RustDesk.bat (từ thư mục chia sẻ ở bước trên) dán vào thư mục ẩn này. Sau đó, quay lại cửa sổ Startup Properties, bấm Add..., bấm Browse... và chọn đúng file .bat vừa chép vào. Bấm OK để lưu.

<img width="605" height="559" alt="image" src="https://github.com/user-attachments/assets/a7a5db87-7495-4f62-a4a1-bb2f3a2bbd27" />

Chép file Install_RustDesk.bat (từ thư mục chia sẻ ở bước trên) dán vào thư mục ẩn này
<img width="1127" height="511" alt="image" src="https://github.com/user-attachments/assets/20348259-f285-475b-aef9-437b5f055f8a" />

<img width="605" height="561" alt="image" src="https://github.com/user-attachments/assets/0bfab09f-03ac-40da-8f3f-eeff56c8c1d1" />

<img width="941" height="577" alt="image" src="https://github.com/user-attachments/assets/3d57fa92-c90f-42d1-8e45-011c5d1ca62c" />

<img width="667" height="575" alt="image" src="https://github.com/user-attachments/assets/688cd50e-98f0-4407-b83e-165ca56049f8" />

<img width="603" height="561" alt="image" src="https://github.com/user-attachments/assets/8c2d3014-b9e6-4b67-9760-bb84734e9f33" />


3. Áp dụng chính sách và Kiểm tra:
Tại máy tính Client. Bật nguồn một máy tính trạm bất kỳ trong OU Domain vừa áp dụng Policy. Quá trình khởi động sẽ chậm hơn bình thường một chút ở màn hình "Please wait..." do chạy ngầm trình cài đặt. Khi đăng nhập vào Windows, hãy kiểm tra xem biểu tượng RustDesk đã xuất hiện ở khay hệ thống và nhận đúng ID server chưa. (Bạn cũng có thể chạy lệnh gpupdate /force trong CMD trước khi khởi động lại máy để ép Client cập nhật Policy mới nhất).

<img width="525" height="269" alt="image" src="https://github.com/user-attachments/assets/5702b44b-4727-4b55-b3e7-2a8d714cd211" />










