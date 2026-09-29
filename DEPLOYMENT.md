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
| Public URL | Chưa được Render cấp trong phiên làm việc này |
| Ngày deploy | Chưa deploy |

`render.yaml` tạo web service từ Dockerfile và Render Key Value. Các biến được
khai báo dưới đây là **cấu hình dự kiến**; chưa thể xác nhận chúng đã được set
trên cloud khi Blueprint chưa chạy. Không lưu giá trị secret trong repository.

| Biến | Cách cấu hình |
| --- | --- |
| `PORT` | Render tự cấp; Dockerfile dùng giá trị này khi khởi chạy |
| `AGENT_API_KEY` | Render hỏi khi tạo Blueprint (`sync: false`) |
| `REDIS_URL` | Lấy từ connection string nội bộ của Render Key Value |
| `RATE_LIMIT_PER_MINUTE` | `10` trong Blueprint |
| `MONTHLY_BUDGET_USD` | `10.0` trong Blueprint |
| `LOG_LEVEL` | `INFO` trong Blueprint |

## Các bước cần hoàn tất trên Render

1. Đẩy thay đổi của repository lên GitHub.
2. Vào Render → New → Blueprint, chọn repository ở trên và triển khai `render.yaml`.
3. Nhập `AGENT_API_KEY` trong Render khi được hỏi. Không ghi giá trị vào tài liệu này.
4. Chờ web service và Key Value hoạt động, ghi public URL thật vào bảng trên.
5. Chạy các lệnh kiểm tra, dán status và body thật bên dưới, rồi lưu ảnh.

## Kiểm tra public URL

Trong PowerShell, đặt `$URL` thành public URL thật của web service:

```powershell
$URL = '<public URL do Render cấp>'
curl.exe -i "$URL/health"
curl.exe -i "$URL/ready"
curl.exe -i -X POST "$URL/ask" -H 'Content-Type: application/json' -d '{"question":"Hello"}'
```

Kết quả mong đợi: `/health` trả 200 với `status: ok`; `/ready` trả 200 với
`redis: true`; `/ask` không có key trả 401. Nếu muốn kiểm tra `/ask` có xác
thực, đặt `DEPLOY_API_KEY` trong `.env` cục bộ rồi chạy test CP5. Không đưa
giá trị key vào terminal output, screenshot hoặc tài liệu công khai.

### Output thực tế

Chưa có public URL để kiểm tra. Chỉ bổ sung output sau khi các lệnh trên chạy
thành công với service thật.

## Ảnh chụp

- `screenshots/dashboard.png` — chờ ảnh dashboard Render sau khi deploy.
- `screenshots/health.png` — chờ ảnh kết quả `/health` từ public URL.

## Local fallback

Chưa chọn local fallback. Chỉ đặt `LOCAL_FALLBACK=true` trong `.env` nếu không
thể triển khai cloud; phương án này giới hạn điểm CP5 ở 9/15. Khi dùng fallback,
ghi rõ lý do, kiểm tra `docker compose ps`, `/health`, `/ready`, `/ask` và lưu
ảnh thật trong `screenshots/`.
