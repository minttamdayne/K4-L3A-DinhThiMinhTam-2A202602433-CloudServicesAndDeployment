# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng hướng dẫn ở mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Dinh Thi Minh Tam  Mã học viên: 2A202602433

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Theo em nghĩ, fail fast giúp phát hiện ngay khi quên đặt AGENT_API_KEY trên Render, trước khi public URL nhận request. Nếu có mặc định `changeme`, người khác có thể đoán khóa và gọi API làm phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log JSON có thể lọc/đếm theo event và trích timestamp để điều tra deploy. Với `ask_completed`, có thể nhóm theo user_id hoặc cộng cost_usd để theo dõi chi phí; một câu print không có dữ liệu có cấu trúc đó.

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
| 1 stage (bản đầu) | chưa đo được vì bản cũ không còn trong Docker image cache |
| Multi-stage | khoảng 331 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản multi-stage hiện đo được khoảng 331 MB. Tôi chưa giữ lại image một stage để đo đối chiếu trực tiếp nên không bịa số chênh lệch. Multi-stage giúp runtime không mang theo artifact build không cần thiết; dependency được cài ở builder rồi chỉ virtualenv và mã nguồn được copy sang runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Sửa một ký tự trong app/main.py chỉ làm layer COPY app chạy lại; các layer base, virtualenv, requirements và pip install được dùng lại từ cache. Nếu COPY . . đặt trước pip install, mọi thay đổi code làm layer đó đổi và Docker phải cài dependency lại.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗ hổng cho phép thực thi lệnh trong Python process. Nếu process là root, kẻ tấn công có quyền cao trong container và có thể tìm đường ảnh hưởng host qua mount hoặc cấu hình. `USER appuser` chạy bằng UID 10001 không có quyền quản trị, cắt chuỗi leo thang ở bước process bị xâm nhập.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với hạn mức 10/phút, cách đếm theo bucket cho phép 20 request trong khoảng 2 giây: gửi 10 request ở giây 59 rồi 10 request ở giây 00 của phút kế tiếp. Sliding window kiểm tra 60 giây gần nhất nên không có khoảng hở này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong 60 giây; cost guard giới hạn tiền theo user và tháng. Một request ít nhưng dùng nhiều token có thể qua rate limit nhưng bị cost guard chặn. Ngược lại, user còn ngân sách nhưng gửi quá 10 request/phút thì rate limit trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp /health và /ready rồi kiểm tra Redis, khi Redis mất kết nối cả ba container trả probe lỗi. Orchestrator có thể loại cả cụm khỏi traffic hoặc restart chúng dù process vẫn sống. Vì vậy /health chỉ kiểm tra process, còn /ready mới kiểm tra Redis và trả 503.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi scale ba agent, history_length vẫn tăng vì các container dùng chung Redis. Nếu dùng dict Python, mỗi container chỉ biết các request vào chính nó; khi request chuyển instance, history_length sẽ bị chia nhỏ hoặc quay lại thấp.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi kiểm tra local lần đầu, gọi /ask không có API key từng trả 500 vì container chạy image cũ còn verify_api_key là TODO. Log Uvicorn chỉ rõ NotImplementedError trong app/auth.py. Tôi rebuild bằng `docker compose up -d --build`, sau đó request thiếu key trả 401. Khi dùng key hợp lệ, log tiếp tục phát hiện store.get_history còn TODO; tôi hoàn thiện Redis store và rebuild lại. Bản Render sau đó trả 200 ở /health và /ready, còn /ask thiếu key trả 401.
