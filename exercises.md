# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: viết câu trả lời ngay dưới từng câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Hoàng Anh Tú  Mã học viên: 2A202602643

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi thử khởi tạo `Settings` mà không có `AGENT_API_KEY`, tôi nhận `ValidationError` với trường `agent_api_key` báo `missing`. Nếu quên khai báo secret trên Render, app dừng ngay lúc khởi động nên tôi biết cấu hình sai trước khi nhận traffic. Nếu để mặc định `changeme`, app vẫn lên và người biết khóa mặc định có thể gọi `/ask` để tiêu ngân sách của tôi.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Tôi gọi `/ask` qua TestClient và thu được dòng: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T04:27:26.596835+00:00", "user_id": "exercise-log", "tokens_in": 3, "tokens_out": 41, "cost_usd": 2.505e-05}`. Log JSON cho phép tôi (1) lọc và đếm request theo `user_id` hoặc mốc thời gian, (2) cộng `cost_usd` và token để tìm user dùng nhiều tài nguyên. Dòng `print("đã trả lời xong")` không có các trường đó để máy tổng hợp.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.189 MB (1.188.808.498 byte) |
| Multi-stage | 184 MB (183.895.944 byte) |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build Dockerfile một stage từ bản gốc thành `day12-agent:single` và so với image multi-stage `day12-agent:prod`; chênh lệch khoảng 1.005 MB. Bản cũ dùng `python:3.11` đầy đủ và giữ mọi thứ của quá trình cài đặt trong image cuối. Bản mới dùng `python:3.11-slim`, cài thư viện ở builder rồi chỉ chép phần đã cài cùng source cần chạy sang runtime; `pip --no-cache-dir` cũng tránh giữ cache tải gói.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile hiện tại `COPY requirements.txt` rồi `RUN pip install` trước khi `COPY app`. Vì vậy chỉ sửa `app/main.py` thì layer base, requirements và cài dependency vẫn dùng cache; layer `COPY app` và các layer runtime đứng sau phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, thay đổi một ký tự trong source sẽ làm layer COPY đổi, kéo theo việc cài lại toàn bộ dependency dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng thực thi lệnh, kẻ tấn công trước hết chạy lệnh với quyền của process trong container. Nếu process là root, họ có thêm quyền trong container và có thể lợi dụng Docker socket bị mount, bind mount nhạy cảm hoặc lỗ hổng thoát container để tác động mạnh lên host; quyền root trong container tự nó chưa đồng nghĩa root trên host. Image của tôi khai báo `USER appuser` (đã kiểm tra bằng `docker image inspect`), nên lệnh chiếm được chỉ chạy dưới user thường, giảm quyền ở bước đầu của chuỗi tấn công.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng 2 giây: 10 request ở 10:00:59 và 10 request ở 10:01:00. Bộ đếm theo phút đồng hồ reset tại giây 00 nên mỗi nhóm vẫn nằm trong hạn mức 10 của phút riêng. Sliding window 60 giây giữ cả hai nhóm trong cùng cửa sổ và sẽ chặn nhóm thứ hai sau khi đã đủ 10 request.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đếm tần suất request trong 60 giây; cost guard so tổng USD theo user trong tháng UTC. Ví dụ user chỉ gửi một request trong phút này nhưng đã tiêu quá 10 USD trong tháng: limiter cho qua, guard trả 402. Ngược lại, user mới tiêu 0,001 USD nhưng đã gửi đủ 10 request trong 60 giây: guard còn ngân sách nhưng limiter trả 429. Trên `/ask`, limiter chạy trước guard nên trường hợp thứ hai dừng ở 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu dùng chung một endpoint có kiểm tra Redis làm cả liveness, khi Redis mất kết nối, cả 3 container sẽ đồng loạt trả 503. Orchestrator có thể tưởng cả 3 process hỏng và restart chúng dù code vẫn chạy; trong lúc Redis chưa hồi phục, các container mới lại fail probe, gây vòng lặp restart và cắt request đang xử lý. Tách probe thì `/health` vẫn 200 để giữ process, còn `/ready` 503 để load balancer ngừng gửi request; khi Redis trở lại, `/ready` lên 200 mà không cần restart app.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Tôi gọi hai lần vào service Render với cùng `X-User-Id` và thấy `history_length` lần lượt là 0 rồi 2 (mỗi lượt lưu một message user và một message assistant). Test CP4 cũng cho hai `ConversationStore` khác nhau dùng chung Redis và đọc được cùng history. Tôi chưa chứng minh bằng 3 container local vì Compose hiện map cố định `8000:8000`, khiến các bản sao tranh cổng host. Nếu dùng dict riêng cho từng process, request rơi vào instance khác có thể lại thấy 0 thay vì 2; số history sẽ phụ thuộc instance nào nhận request.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Render build và tạo hai service thành công; lỗi thực tế tôi gặp ở bước xác minh deploy là `pytest tests/test_cp5.py` báo `Chưa điền Public URL thật vào DEPLOYMENT.md` và 4 test public URL lỗi ở setup. Tôi xem traceback của fixture `base_url` và thấy tài liệu vẫn thiếu domain thật, không phải lỗi app hay Redis. Sau khi Render cấp `https://day12-agent-3ziw.onrender.com`, tôi điền URL vào `DEPLOYMENT.md` rồi chạy lại: `/health` 200, `/ready` 200, `/ask` thiếu key 401 và 8 test CP5 đạt.
