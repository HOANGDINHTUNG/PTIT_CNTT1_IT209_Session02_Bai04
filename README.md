# Bài 4: Cấu hình tường lửa bảo vệ máy chủ (UFW & Cloud Firewall Integration)

## Mục tiêu
Thiết lập hệ thống bảo mật 2 lớp cho máy chủ:
1. **Lớp hệ điều hành**: Sử dụng tường lửa UFW (Uncomplicated Firewall) tích hợp sẵn trên Ubuntu để lọc gói tin.
2. **Lớp hạ tầng đám mây**: Cấu hình DigitalOcean Cloud Firewall để chặn các traffic không hợp lệ ngay từ biên mạng (trước khi chúng kịp chạm tới Droplet).

## Các bước thực hiện chi tiết

### Phần 1: Cấu hình lớp bảo vệ UFW trên hệ điều hành
Sử dụng SSH để đăng nhập vào Droplet của bạn (có thể dùng tài khoản `devops` hoặc `root`). Tại giao diện dòng lệnh, lần lượt chạy các cấu hình sau:

1. **Cấu hình chính sách mặc định**: 
   Nguyên tắc bảo mật cơ bản là "cấm tất cả những gì không được phép". Do đó ta sẽ thiết lập chặn mặc định tất cả lưu lượng đi vào và cho phép lưu lượng đi ra:
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   ```

2. **Mở các cổng dịch vụ cần thiết**:
   Cho phép lưu lượng truy cập SSH (cổng 22) để không bị mất kết nối và HTTP (cổng 80) cho trang web Nginx:
   ```bash
   sudo ufw allow 22/tcp
   sudo ufw allow 80/tcp
   ```

3. **Kích hoạt UFW**:
   Bật tường lửa lên để các quy tắc trên có hiệu lực:
   ```bash
   sudo ufw enable
   ```
   *(Hệ thống sẽ hiển thị cảnh báo `Command may disrupt existing ssh connections...`. Hãy nhập `y` và nhấn Enter để tiếp tục vì ta đã mở cổng 22 ở trên).*

4. **Kiểm tra trạng thái**:
   Xem lại các cấu hình vừa thiết lập để đảm bảo mọi thứ đã chính xác:
   ```bash
   sudo ufw status verbose
   ```

> **Log trạng thái UFW:**
> *Hãy dán kết quả text đầu ra của lệnh `sudo ufw status verbose` (hiển thị Status: active và danh sách các rules) vào khối lệnh bên dưới:*
> ```text
> [Dán text đầu ra của lệnh sudo ufw status verbose vào đây]
> ```

---

### Phần 2: Cấu hình lớp bảo vệ bằng DigitalOcean Cloud Firewall
1. Mở trình duyệt và đăng nhập vào **DigitalOcean Console**.
2. Ở thanh menu bên trái, tìm tới phần **Networking**, sau đó chọn tab **Firewalls** ở phía trên.
3. Nhấn vào nút **Create Firewall**.
4. Thiết lập các thông số:
   - **Name**: Đặt tên cho tường lửa (ví dụ: `web-server-fw`).
   - **Inbound Rules** (Quy tắc đầu vào): Cấu hình để chỉ cho phép 2 cổng truy cập sau:
     - Dịch vụ **SSH** / Port `22` / Nguồn: `All IPv4`, `All IPv6` (Hoặc bạn có thể giới hạn chỉ IP nhà bạn).
     - Dịch vụ **HTTP** / Port `80` / Nguồn: `All IPv4`, `All IPv6`.
     - *(Xóa các rule khác nếu có để tránh rủi ro mở cổng không mong muốn).*
   - **Outbound Rules** (Quy tắc đầu ra): Giữ nguyên cấu hình mặc định (cho phép tất cả ra ngoài).
   - **Apply to Droplets**: Gõ và chọn tên Droplet của bạn để áp dụng Cloud Firewall này.
5. Nhấn **Create Firewall** để hoàn thành.

> **Ảnh thiết lập Cloud Firewall:**
> *Chèn ảnh chụp màn hình giao diện cấu hình Inbound/Outbound rules của Cloud Firewall trên DigitalOcean Console vào vị trí này:*
> 
> `[Chèn ảnh chụp DigitalOcean Console tại đây]`

---

### Phần 3: Kiểm tra và đánh giá
Sau khi thiết lập xong 2 lớp tường lửa, hãy dùng máy tính cá nhân để thực hiện các bài test:
1. Đăng nhập lại bằng SSH vào máy chủ -> **Phải thành công**.
2. Truy cập địa chỉ IP public của máy chủ bằng trình duyệt web `http://<IP_ADDRESS_DROPLET>` -> **Phải thành công**.
3. Dùng lệnh ping, telnet hoặc cố gắng truy cập vào một cổng không được mở (ví dụ cổng 8080) -> **Phải bị chặn hoặc rơi vào trạng thái Timeout** do cả UFW và Cloud Firewall đều từ chối kết nối.
