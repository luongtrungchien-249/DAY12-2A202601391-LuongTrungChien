# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng trả lời mẫu dưới mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Lương Trung Chiến  Mã học viên: 2A202601391

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Trong lần deploy Railway, nếu tôi quên đặt `AGENT_API_KEY`, `Settings` làm ứng dụng dừng ngay với `ValidationError` trước khi nhận traffic. Nhờ vậy tôi biết cấu hình cloud đang thiếu và sửa ngay. Nếu dùng mặc định `"changeme"`, service vẫn báo khỏe nhưng bất kỳ ai biết khóa mặc định đều có thể gọi `/ask`, làm phát sinh chi phí mà tôi khó phát hiện sớm.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được là: `{"event": "ask_completed", "level": "info", "timestamp": "2026-08-10T06:13:26.697829+00:00", "user_id": "sv-test", "tokens_in": 4, "tokens_out": 28, "cost_usd": 1.74e-05}`. Với JSON này, tôi có thể lọc hoặc đếm request theo `user_id`/`event`, đồng thời tổng hợp và cảnh báo theo `cost_usd` hoặc số token. Dòng `print("đã trả lời xong")` không có các trường có cấu trúc để máy thực hiện hai việc đó.

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
| 1 stage (bản đầu) | 1.66 GB (khoảng 1660 MB) |
| Multi-stage | 294 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage giữ image Python đầy đủ, toàn bộ công cụ và dữ liệu trung gian dùng khi cài package, cùng pip cache nên đạt khoảng 1.66 GB. Bản multi-stage dùng `python:3.11-slim`; stage builder chỉ tạo wheel, còn runtime chỉ nhận wheel đã cần và xóa `/wheels`, nên không mang môi trường build sang image cuối. Phần chênh lệch khoảng 1.37 GB chủ yếu là base image đầy đủ, công cụ build và cache/trung gian không cần khi chạy ứng dụng.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile hiện tại, khi chỉ sửa `app/main.py`, Docker vẫn dùng lại cache của base image, bước tạo user, `COPY requirements.txt`, build wheel và `pip install` vì `requirements.txt` không đổi. Layer `COPY app ./app` và các layer phía sau nó phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source đều làm layer copy đổi, kéo theo việc cài lại toàn bộ dependency dù `requirements.txt` không thay đổi, nên build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi rủi ro là: kẻ tấn công khai thác một lỗi trong API Python để thực thi lệnh trong container; nếu process chạy root thì lệnh đó có toàn quyền trong container, có thể sửa file hệ thống, đọc dữ liệu nhạy cảm hoặc tận dụng thêm lỗi runtime/cấu hình mount để tấn công host với quyền cao. Lệnh `USER appuser` cắt chuỗi tại bước thực thi sau khi chiếm ứng dụng: mã độc chỉ có quyền của user thường, không còn mặc nhiên có quyền root. Biện pháp này giảm phạm vi thiệt hại, dù vẫn cần Docker và host được cấu hình an toàn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong 2 giây: gửi 10 request ở giây cuối của phút trước, ví dụ 10:00:59, rồi gửi thêm 10 request ngay sau khi bộ đếm reset ở 10:01:00. Cả hai phút đều không vượt 10 request nhưng tải thực tế là 20 request gần như liên tiếp. Sliding window 60 giây vẫn nhìn thấy cả hai nhóm nên ngăn được cách lách này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ/số request trong một cửa sổ ngắn, còn cost guard giới hạn tổng số tiền theo user trong cả tháng. Một user gửi một request rất đắt sau khi đã gần hết ngân sách vẫn còn quota tốc độ, nên rate limit cho qua nhưng cost guard chặn 402. Ngược lại, user còn nguyên ngân sách nhưng gửi hơn 10 request nhỏ trong một phút thì cost guard vẫn cho phép về mặt tiền, còn rate limiter chặn 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp hai endpoint, Redis mất kết nối làm probe chung thất bại trên cả 3 container. Orchestrator hiểu nhầm cả ba process đã chết, loại chúng khỏi traffic rồi lần lượt restart. Các container mới vẫn không kết nối được Redis nên probe tiếp tục thất bại và rơi vào vòng lặp restart, dù bản thân FastAPI vẫn sống. Tách riêng giúp `/health` tiếp tục trả 200 để không restart process; `/ready` trả 503 để load balancer tạm ngừng gửi request cho đến khi Redis phục hồi.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi hai instance `ConversationStore` cùng dùng Redis, test của project cho thấy request đầu có `history_length = 0`, request tiếp theo với cùng user thấy hai message trước và có `history_length = 2`; các replica nhìn chung một lịch sử. Nếu dùng dict Python, mỗi container có dict riêng nên request được phân phối sang replica khác sẽ thấy 0 hoặc một số nhỏ hơn; con số sẽ tăng không đều theo container nhận request và lịch sử còn mất hoàn toàn khi container restart.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi thật tôi gặp trên Railway là `Error: Invalid value for '--port': '$PORT' is not a valid integer`, sau đó health check báo `1/1 replicas never became healthy`. Tôi xem deploy log bằng Railway CLI và thấy start command truyền nguyên chuỗi `$PORT` cho Uvicorn thay vì mở rộng biến môi trường. Tôi sửa `railway.toml` để chạy lệnh qua shell: `sh -c 'uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8000}'`, push commit mới và redeploy. Deployment sau đó thành công, `/health` trả 200 tại public URL.
