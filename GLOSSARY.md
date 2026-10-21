# NodeWarden Migration & Deployment

Tài liệu định nghĩa các thuật ngữ chính trong quá trình chuyển đổi từ Vaultwarden (Fly.io) sang NodeWarden (Cloudflare Workers).

## Language

**NodeWarden**:
Máy chủ tương thích giao thức Bitwarden chạy không máy chủ (serverless) trên nền tảng Cloudflare Workers.
_Avoid_: Bitwarden Server, Vaultwarden

**Vaultwarden**:
Máy chủ Bitwarden mã nguồn mở tự lưu trữ viết bằng Rust, hiện đang chạy trên Fly.io với PostgreSQL.
_Avoid_: NodeWarden, Bitwarden Cloud

**Cloudflare D1**:
Cơ sở dữ liệu quan hệ serverless (dựa trên SQLite) trên mạng lưới Cloudflare, dùng để lưu trữ dữ liệu tài khoản, ciphers và cấu hình của NodeWarden.
_Avoid_: PostgreSQL, Cloudflare KV (khi nói về database quan hệ)

**Cloudflare R2**:
Hệ thống lưu trữ đối tượng (Object Storage) tương thích S3 của Cloudflare không tính phí băng thông (egress), dùng để lưu tệp đính kèm và dữ liệu Bitwarden Send.
_Avoid_: Volume, Local disk

**Vault Export**:
Tệp sao lưu chuẩn của Bitwarden (dạng JSON hoặc Password-Protected Encrypted JSON) chứa toàn bộ mật khẩu, ghi chú và thư mục, dùng làm cầu nối di chuyển dữ liệu an toàn.
_Avoid_: Database dump, SQL dump
