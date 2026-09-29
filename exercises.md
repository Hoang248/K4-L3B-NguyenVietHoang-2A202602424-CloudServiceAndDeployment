# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: viết câu trả lời của bạn trực tiếp dưới từng câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu một service cloud khởi động mà thiếu AGENT_API_KEY, fail fast giúp phát hiện lỗi cấu hình ngay trong log deploy. Nếu dùng khóa mặc định như changeme, service vẫn chạy và chỉ lộ vấn đề khi có request thật; lúc đó có thể đã có người dùng gọi API bằng một khóa không an toàn.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng mình quan sát được là {"event": "service_started", "level": "info", "timestamp": "...", "service": "day12-agent", "version": "1.0.0"}. Máy có thể parse chính xác từng trường để lọc theo event hoặc cảnh báo theo level; đồng thời mỗi event nằm trên một dòng nên hệ thống thu log không bị ghép hoặc tách sai. print("đã trả lời xong") chỉ là một chuỗi văn bản, không có cấu trúc và không có timestamp chuẩn.

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
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Mình build image một-stage tạm thời và image multi-stage bằng docker images; cả hai đều hiển thị khoảng 271 MB trong môi trường này.
>
> | Bản | Dung lượng đo được |
> |---|---:|
> | 1 stage tạm | 271 MB |
> | Multi-stage | 271 MB |
>
> Hai số gần như bằng nhau vì requirements hiện tại không kéo theo nhiều compiler hoặc package build chỉ tồn tại ở builder. Dù vậy multi-stage vẫn tách nơi cài dependency khỏi runtime; nếu builder cần thêm tool nặng thì những tool đó không bị copy vào image cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile hiện copy requirements.txt và cài dependency trước khi copy app và utils. Khi chỉ sửa app/main.py, layer cài dependency được dùng lại từ cache, còn layer copy source và các layer sau đó phải chạy lại. Nếu COPY . . đặt trước RUN pip install, mọi thay đổi source sẽ làm mất cache của bước pip install và Docker phải cài lại toàn bộ dependency, khiến build chậm hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi rủi ro là: lỗi trong Python cho phép kẻ tấn công chạy lệnh bên trong container, rồi tiến hành đọc hoặc sửa file mà process có quyền truy cập. Nếu process là root, quyền đó rộng hơn nhiều và có thể hỗ trợ khai thác cấu hình Docker hoặc mount sai để ảnh hưởng host. USER appuser cắt chuỗi ở điểm process trong container không còn quyền root; nó không thay thế hoàn toàn sandbox của Docker nhưng giảm đáng kể tác động của lỗi.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với giới hạn 10 request trong 60 giây, cách đếm theo phút đồng hồ có thể cho 20 request trong khoảng 2 giây: 10 request ngay trước thời điểm chuyển phút và 10 request ngay sau giây 00. Sliding window không reset theo đồng hồ; nó xóa request cũ hơn 60 giây rồi đếm các request còn lại, nên không có khe hở ở biên phút.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đo tần suất request, còn cost guard đo tổng chi phí theo user và tháng UTC. Một user có thể gửi 5 request hợp lệ nên rate limit cho qua, nhưng một request gọi model đắt làm chi phí vượt ngân sách thì cost guard phải trả 402. Ngược lại, user có thể còn nhiều ngân sách nhưng gửi liên tục quá 10 request/phút thì rate limiter trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu /health cũng kiểm tra Redis, khi Redis mất 30 giây cả ba container sẽ trả 503. Orchestrator có thể hiểu nhầm rằng cả ba process đã chết, lần lượt restart chúng, rồi tiếp tục restart dù process thực ra vẫn sống; cụm có thể rơi vào vòng lặp restart. Với hai endpoint tách biệt, /health vẫn trả 200 để báo process còn sống, còn /ready trả 503 để load balancer tạm ngừng gửi traffic trong lúc Redis lỗi.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi scale agent lên ba container, history nằm trong Redis List theo user nên request đi vào instance nào cũng đọc được cùng dữ liệu; history_length tiếp tục tăng theo các lượt hỏi. Nếu dùng dict Python, mỗi instance có một bản dict riêng. Load balancer đổi instance sẽ làm history_length quay về thấp hoặc khác nhau, và restart một container sẽ làm mất phần history của instance đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy Render, mình gặp hiện tượng free instance sleep khi không có traffic; dashboard cảnh báo request đầu tiên sau lúc ngủ có thể chậm khoảng 50 giây. Mình kiểm tra bằng Deploy logs và gọi lại /health, /ready sau khi service thức dậy. Kết quả là service bind đúng PORT, /health trả 200 và /ready trả 200 với redis=true; đây là độ trễ cold start của gói free chứ không phải lỗi ứng dụng.
