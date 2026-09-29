# CP5 — Triển khai Render

## Thông tin học viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Hoang Anh Tu |
| Mã học viên / MSSV | 2A202602643 |
| Repository | https://github.com/ttien0181/K4-L3B-DAY12-HoangAnhTu-2A202602643-CloudServicesAndDeployment |

## Trạng thái triển khai

| Mục | Nội dung |
| --- | --- |
| Platform | Render Blueprint (`render.yaml`) |
| Public URL | https://day12-agent-3ziw.onrender.com |
| Ngày deploy | 2026-09-29 |

`render.yaml` tạo web service từ Dockerfile và Render Key Value. Blueprint đã
sync thành công trên Render. `/ready` trả `redis: true`, xác nhận kết nối Redis.
Không lưu giá trị secret trong repository.

| Biến | Cách cấu hình |
| --- | --- |
| `PORT` | Render tự cấp; Dockerfile dùng giá trị này khi khởi chạy |
| `AGENT_API_KEY` | Render hỏi khi tạo Blueprint (`sync: false`) |
| `REDIS_URL` | Lấy từ connection string nội bộ của Render Key Value |
| `RATE_LIMIT_PER_MINUTE` | `10` trong Blueprint |
| `MONTHLY_BUDGET_USD` | `10.0` trong Blueprint |
| `LOG_LEVEL` | `INFO` trong Blueprint |

## Trạng thái Render

Blueprint trên nhánh `main` đã tạo `day12-agent` và `day12-redis`. Web service
đang phục vụ các endpoint công khai tại URL ở trên. Cần lưu ảnh dashboard và
ảnh kiểm tra `/health` vào `screenshots/` trước khi nộp bài.

## Kiểm tra public URL

Trong PowerShell:

```powershell
$URL = 'https://day12-agent-3ziw.onrender.com'
curl.exe -i "$URL/health"
curl.exe -i "$URL/ready"
curl.exe -i -X POST "$URL/ask" -H 'Content-Type: application/json' -d '{"question":"Hello"}'
```

Kết quả mong đợi: `/health` trả 200 với `status: ok`; `/ready` trả 200 với
`redis: true`; `/ask` không có key trả 401. Nếu muốn kiểm tra `/ask` có xác
thực, đặt `DEPLOY_API_KEY` trong `.env` cục bộ rồi chạy test CP5. Không đưa
giá trị key vào terminal output, screenshot hoặc tài liệu công khai.

### Output thực tế

Đã gọi service thật qua HTTPS ngày 2026-09-29:

```text
GET  /health  200  {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready   200  {"status":"ready","redis":true}
POST /ask     401  {"detail":"invalid or missing API key"}
```

## Ảnh chụp

- `screenshots/dashboard.png` — chờ ảnh dashboard Render sau khi deploy.
- `screenshots/health.png` — chờ ảnh kết quả `/health` từ public URL.

## Local fallback

Chưa chọn local fallback. Chỉ đặt `LOCAL_FALLBACK=true` trong `.env` nếu không
thể triển khai cloud; phương án này giới hạn điểm CP5 ở 9/15. Khi dùng fallback,
ghi rõ lý do, kiểm tra `docker compose ps`, `/health`, `/ready`, `/ask` và lưu
ảnh thật trong `screenshots/`.
