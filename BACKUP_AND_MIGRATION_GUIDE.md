# HƯỚNG DẪN DI CHUYỂN DỮ LIỆU & SAO LƯU (BACKUP) NODEWARDEN

Tài liệu này hướng dẫn chi tiết từng bước:
1. Di chuyển toàn bộ mật khẩu từ Vaultwarden (Fly.io) sang NodeWarden (Cloudflare).
2. Thiết lập Bitwarden Client (Tiện ích mở rộng trình duyệt & Điện thoại) trỏ sang máy chủ mới.
3. Các cơ chế sao lưu (Backup) tự động và thủ công cho NodeWarden.

---

## PHẦN 1: DI CHUYỂN DỮ LIỆU TỪ FLY.IO SANG NODEWARDEN

### Bước 1: Xuất dữ liệu (Export) từ Vaultwarden cũ (Fly.io)
1. Đăng nhập vào trang quản lý Vaultwarden hiện tại trên Fly.io.
2. Vào mục **Tools (Công cụ)** > **Export Vault (Xuất dữ liệu)**.
3. Chọn định dạng:
   - **Bitwarden (Encrypted JSON)** (Khuyên dùng: Mã hóa bằng mật khẩu để đảm bảo an toàn tối đa).
   - Hoặc **Bitwarden (JSON)** (Không mã hóa, dùng offline và xóa file ngay sau khi import xong).
4. Nhập mật khẩu xác nhận và tải tệp về máy tính.

### Bước 2: Tạo tài khoản quản trị trên NodeWarden
1. Truy cập Web Vault: [https://bw.selfh.vip](https://bw.selfh.vip) (hoặc link dự phòng [https://nodewarden.trung270310.workers.dev](https://nodewarden.trung270310.workers.dev))
2. Nhấp vào **Create Account (Tạo tài khoản)**.
3. Đăng ký tài khoản với email và Master Password của bạn.
   > **Lưu ý quan trọng**: Tài khoản đầu tiên đăng ký sẽ trở thành tài khoản quản trị. Sau khi tài khoản đầu tiên được tạo, NodeWarden sẽ tự động yêu cầu Mã mời (Invite Code) cho các lần đăng ký sau, giúp ngăn chặn người lạ đăng ký (tương đương `SIGNUPS_ALLOWED = "false"` trên Fly.io).

### Bước 3: Nhập dữ liệu (Import) vào NodeWarden
1. Đăng nhập vào tài khoản vừa tạo trên [https://bw.selfh.vip](https://bw.selfh.vip).
2. Vào mục **Tools (Công cụ)** > **Import Data (Nhập dữ liệu)**.
3. Chọn nguồn nhập: **Bitwarden (JSON)** (hoặc Encrypted JSON tương ứng).
4. Chọn tệp đã tải về ở Bước 1 và nhấn **Import Vault**.
5. Kiểm tra lại danh sách mật khẩu, mã xác thực 2FA (TOTP) và ghi chú bảo mật.

---

## PHẦN 2: CẤU HÌNH CÁC THIẾT BỊ CLIENT

Sau khi nhập dữ liệu thành công, bạn cập nhật lại URL máy chủ trên tất cả các thiết bị:

### Trên Browser Extension (Chrome, Firefox, Brave, Edge):
1. Đăng xuất khỏi tài khoản hiện tại.
2. Tại màn hình đăng nhập, nhấp vào biểu tượng **Bánh răng (Cài đặt)** ở góc trên bên trái.
3. Trong ô **Server URL (URL máy chủ)**, nhập:
   ```text
   https://bw.selfh.vip
   ```
4. Nhấn **Save (Lưu)** và đăng nhập với tài khoản mới.

### Trên App điện thoại (iOS / Android):
1. Đăng xuất khỏi app Bitwarden.
2. Tại màn hình đăng nhập, chọn biểu tượng bánh răng cài đặt.
3. Nhập Server URL: `https://bw.selfh.vip`.
4. Lưu và đăng nhập lại.

---

## PHẦN 3: CÁC PHƯƠNG ÁN SAO LƯU (BACKUP) NODEWARDEN

### Phương án 1: Sao lưu tự động qua Cloud Backup Center (Tích hợp sẵn)
NodeWarden có sẵn tính năng **Cloud Backup Center** cực kỳ mạnh mẽ trong Web Vault:
1. Đăng nhập vào [https://nodewarden.trung270310.workers.dev](https://nodewarden.trung270310.workers.dev).
2. Vào mục **Settings (Cài đặt)** > **Cloud Backup Center**.
3. Cấu hình tự động sao lưu định kỳ qua:
   - **WebDAV** (Nextcloud, TeraBox, Koofr, v.v.)
   - Hoặc **S3 Compatible** (Cloudflare R2, AWS S3, Backblaze B2, v.v.)
4. Bật chế độ chạy theo lịch (Scheduled Cron) để hệ thống tự động backup gia tăng (incremental).

---

### Phương án 2: Sao lưu cơ sở dữ liệu D1 bằng Wrangler CLI (Từ terminal)
Cơ sở dữ liệu lưu mật khẩu nằm trên Cloudflare D1 `nodewarden-db`. Bạn có thể xuất bản sao lưu SQL bất kỳ lúc nào:

1. Chạy lệnh xuất bản sao lưu từ thư mục dự án `/home/mr-trung/docker/nodewardent`:
   ```bash
   npx wrangler d1 export nodewarden-db --remote --output="./backups/d1_backup_$(date +%Y%m%d).sql"
   ```

2. Cách khôi phục (Restore) khi cần:
   ```bash
   npx wrangler d1 execute nodewarden-db --remote --file="./backups/d1_backup_YYYYMMDD.sql"
   ```

---

### Phương án 3: Sao lưu tệp đính kèm R2
Các tệp đính kèm lớn và Bitwarden Send được lưu trong Cloudflare R2 bucket: `nodewarden-attachments`.
Bạn có thể đồng bộ hoặc tải về máy qua `rclone` hoặc S3 CLI nếu cần lưu trữ ngoại tuyến định kỳ.

---

## PHẦN 4: TẮT DỊCH VỤ FLY.IO CŨ

Sau khi đã kiểm tra mọi thứ hoạt động trơn tru trên Cloudflare:
1. Dừng máy ảo Fly.io để tiết kiệm tài nguyên:
   ```bash
   fly scale count 0 -a vaultwardentrung
   ```
2. Giữ nguyên database PostgreSQL trên Aiven khoảng 1 - 2 tuần để đối chiếu trước khi xóa hoàn toàn.
