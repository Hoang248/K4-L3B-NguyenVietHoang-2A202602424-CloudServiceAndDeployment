# Checkpoint 5 — Deployment

## Thông tin học viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Nguyễn Việt Hoàng |
| Mã học viên | 2A202602424 |
| Repository | https://github.com/Hoang248/K4-L3B-NguyenVietHoang-2A202602424-CloudServiceAndDeployment |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | https://day12-agent-f69u.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy | 2026-09-29 |
| Web service | day12-agent |
| Redis service | day12-redis (Render Key Value) |

## Environment variables

Các biến được cấu hình trên Render Dashboard hoặc qua Blueprint. Giá trị
secret không được ghi trong repository.

| Biến | Trạng thái | Nguồn |
|---|---|---|
| AGENT_API_KEY | Set | Render Dashboard secret |
| REDIS_URL | Set | Kết nối nội bộ từ day12-redis |
| RATE_LIMIT_PER_MINUTE | Set | Blueprint: 10 |
| MONTHLY_BUDGET_USD | Set | Blueprint: 10.0 |
| LOG_LEVEL | Set | Blueprint: INFO |
| PORT | Tự cấp | Render platform |

## Kiểm tra public URL

~~~text
GET https://day12-agent-f69u.onrender.com/health
HTTP 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET https://day12-agent-f69u.onrender.com/ready
HTTP 200
{"status":"ready","redis":true}

POST https://day12-agent-f69u.onrender.com/ask
Không gửi X-API-Key
HTTP 401
{"detail":"invalid or missing API key"}
~~~

PowerShell commands used:

~~~powershell
$URL = "https://day12-agent-f69u.onrender.com"
(Invoke-WebRequest "$URL/health").Content
(Invoke-WebRequest "$URL/ready").Content
~~~

## Screenshots

- screenshots/dashboard.png — Render service ở trạng thái Live.
- screenshots/health.png — kết quả kiểm tra /health và /ready.

Không lưu API key hoặc giá trị secret trong file này.
