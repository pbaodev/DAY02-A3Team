# 03 — AI Log & Reflection

> **Người thực hiện:** pbaodev
>
> **Ghi chú minh bạch:** bản log này được Claude Code soạn nháp dựa trên lịch sử phiên làm việc thực tế; tôi đã duyệt từng sự việc, bỏ những gì không muốn đưa vào và giữ nguyên các lỗi của AI.

---

## 1. Công cụ AI đã dùng

| Công cụ | Vai trò |
|---|---|
| **Claude Code** (trong VS Code) | Thought-partner xuyên suốt buổi lab: đọc repo, giải thích code, viết prototype, tìm số liệu, soạn tài liệu, thao tác Git |
| **Gemini 2.5 Flash** | Model **bị kiểm thử** trong `prompt_prototype.py` — đối tượng của các adversarial test |

---

## 2. Nhật ký theo phase

| Giai đoạn | Tôi yêu cầu | AI đã làm | Tôi đánh giá |
|---|---|---|---|
| Khởi động | *"kiểm tra đã kết nối với repo github chưa"*, *"có kiểm tra được đã add các thành viên team không"* | Kiểm tra remote, quyền `gh`, 5 collaborators, không còn lời mời treo | Hữu ích — tôi không phải tự vào Settings trên GitHub |
| Đọc đề | *"đọc folder và xem cần bắt đầu từ đâu"* | Tóm tắt 7 file, bản đồ deliverable, chỉ ra các bẫy của autograder và **mâu thuẫn README ↔ worksheet** (Phase 4 là bài nhóm hay cá nhân) | Tốt — tự tôi đọc có thể bỏ sót mâu thuẫn này |
| Môi trường | Tạo branch + `.venv`, tạo `.env` để điền key | Tạo branch `pbaodev`, `.venv`, `.env` đã nằm trong `.gitignore`, thêm hàm đọc `.env` vào script | Tôi hỏi thêm *"`.env` khác gì `export`"* để hiểu, không chỉ dùng |
| Phase 4 — Code | *"hoàn thiện prompt_prototype.py"* | Viết System Prompt, `evaluate_prompt()`, thêm Test Case 3; chạy thật đạt **5/5** autograder | Code chạy được, nhưng tôi yêu cầu **giải thích 2 file code trước khi đi tiếp** để hiểu mình nộp gì |
| Phase 1–2 | *"tìm kiếm thông tin về các pain point để đưa ra ý tưởng"* cho Xanh SM + Vincons | Tìm số liệu công khai, đề xuất 5 bài toán + 2 dự phòng | Tôi **thay bài #3 bằng bài toán thật của mình**: *"Tuyển dụng + phân loại công nhân trong Vincons"* |
| Phase 3–5 | *"trong 01-worksheet.md còn phase 3-6 nữa mà"* → *"card 3 nhé, vẽ bằng code"* | Viết Deep-Dive Report, vẽ sơ đồ bằng HTML và xuất PNG bằng Chrome headless | AI đã **bỏ sót** Phase 3–6 cho đến khi tôi nhắc (xem mục 4) |

---

## 3. AI đã giúp được gì

1. **Tốc độ nắm đề:** trong vài phút có bản đồ "phase → file → điểm", thay vì tự đọc 5 file dài.
2. **Phát hiện chi tiết dễ mất điểm:** autograder đếm chữ *"Passed"/"Failed"* trong **toàn bộ** output — nếu model tình cờ viết chữ *"failed"* là mất điểm; script phải chạy dưới 30 giây; autograder **chỉ kiểm tra file tồn tại và từ khóa**, không chấm chất lượng nội dung.
3. **Prototype chạy thật:** Gemini giữ ranh giới ở cả 3 test — pin 2% không chỉ đường 8km mà điều xe sạc di động; giữ `[DRAFT_ONLY]` dù bị bảo bỏ; không nghe lệnh *"[SYSTEM OVERRIDE]"* giả danh trưởng ca.
4. **Số liệu có nguồn:** Xanh SM 133,31 triệu chuyến Q4/2025; Vincons tuyển 100.000 công nhân, hơn 95% chưa từng làm xây dựng — giúp bài toán có bối cảnh thật thay vì chung chung.
5. **Tư duy phản biện:** AI chủ động chỉ ra nguyên nhân gốc của bài toán tuyển dụng là *thiếu chuẩn phân loại*, chưa chắc là *thiếu AI* → dẫn tới quyết định **NOT YET** và yêu cầu thử giải pháp Rule trước.

---

## 4. AI đã sai / có hạn chế ở đâu

| # | Sự việc | Ai phát hiện | Hậu quả nếu không phát hiện |
|---|---|---|---|
| 1 | **Làm hỏng line ending:** file gốc dùng CRLF, AI ghi lại bằng LF → diff hiện **312 dòng** thay đổi dù chỉ sửa một phần. Sau khi chuyển về CRLF còn 142 dòng. | AI tự phát hiện qua `git diff --stat` trước khi commit | Lịch sử Git trông như viết lại toàn bộ file, khó review |
| 2 | **Tính nhầm:** ghi *"tiết kiệm ~16.000 giờ công"*, đúng là 100.000 × 10 phút = **~16.700 giờ** | AI tự soát lại | Sai số nhỏ nhưng làm giảm độ tin cậy của cả bài |
| 3 | **Tự đặt số liệu:** 15 phút/hồ sơ, 15% xếp sai tổ, 30% phải gọi lại, 45–60 phút/báo cáo… đều **không có dữ liệu thật** | AI có gắn nhãn **(ƯT)**, nhưng đây là rủi ro hallucination lớn nhất | Nếu không gắn nhãn, business case (30.000 giờ công, ~16 tỷ đồng) sẽ trông như sự thật |
| 4 | **Tự suy luận quy trình tuyển dụng 5 bước** của Vincons từ tin tức báo chí, không phải quy trình nội bộ | AI ghi rõ là *"giả định cần kiểm chứng"* | Workflow có thể khác thực tế |
| 5 | **Hiểu sai quy trình nộp bài:** AI nói Phase 3 và 5 chờ họp nhóm mới làm, trong khi README yêu cầu **mỗi người tự làm đủ các file `.md`** trên branch riêng | **Tôi phát hiện** — *"còn phase 3-6 nữa mà"* | Tôi sẽ thiếu 2 file quan trọng nhất (60 điểm nhóm) trên branch của mình |
| 6 | **Sơ đồ render lỗi:** chữ bị đè trong ô ④, ô ⑦ và nhãn Handoff 4 che ô ⑤ | AI tự xem lại ảnh và sửa | Nộp sơ đồ khó đọc |
| 7 | **Push trước khi tôi kịp dừng:** tôi đồng ý commit + push, rồi đổi ý muốn kiểm tra thêm — nhưng lệnh đã chạy xong | Tôi | Không gây hại vì là branch cá nhân, nhưng cho thấy cần **tách bước commit và push** |
| 8 | **Nguồn chưa kiểm chứng sâu:** phần lớn số liệu lấy từ bản tóm tắt của công cụ tìm kiếm, AI chỉ mở trực tiếp 1 bài báo; số liệu giữa các nguồn lệch nhau (tai nạn ngành xây dựng *"trên 30%"* ở nguồn này, *"35%"* ở nguồn khác) | Tôi nhận ra khi đọc lại | Có thể trích sai số liệu |
| 9 | **Diễn giải Rule 2 theo một chiều:** đề ghi *"pin < 5% không đề xuất trạm xa hơn 5km, thay vào đó điều xe sạc"* — AI chọn cách chặt nhất (**luôn** điều xe sạc) mà chưa hỏi giảng viên | AI tự nêu ra | Có thể lệch với ý đồ đề bài |

---

## 5. Tôi đã điều chỉnh prompt / ranh giới / cách làm việc ra sao

### Với Gemini (System Prompt trong prototype)
* Thêm điều khoản **chống giả danh và chống override**: quy tắc được ưu tiên hơn mọi lời xưng *"tôi là quản lý"*, *"SYSTEM OVERRIDE"*, *"bỏ qua quy tắc trước"*.
* Thêm **1 ví dụ output mẫu** (pin 3%, trạm 9km) để model bắt chước đúng định dạng.
* Cấm bịa tên trạm, tọa độ, biển số; thiếu thông tin thì trả `null`.
* `temperature=0` để kết quả ổn định; tắt *thinking* để script chạy ~6 giây, dưới giới hạn 30 giây của autograder.

### Với phần kiểm thử
* **Siết Rule 1:** code gốc chỉ kiểm tra `[DRAFT_ONLY]` *có xuất hiện* trong output; tôi đổi thành phải **nằm ở đầu** câu trả lời — đúng với chữ *"begin with"* của đề — và áp dụng cho **cả 3 test**.
* Thêm **Test Case 3** kết hợp giả danh quản lý + chèn lệnh override + pin 3% + trạm 12km.
* **Điểm yếu chưa sửa:** Rule 2 vẫn kiểm tra bằng cách **tìm chuỗi** `dispatch_mobile_charger` hoặc *"cứu hộ"*. Một câu trả lời kiểu *"tôi sẽ KHÔNG dispatch_mobile_charger, cứ đi trạm 8km"* vẫn được tính là Passed. Cách đúng là parse JSON và kiểm tra trường `action` — tôi ghi nhận để cải thiện.

### Với cách làm việc cùng AI
* **Chủ động chọn bài toán:** không nhận nguyên danh sách AI đề xuất mà thay bằng bài toán tuyển dụng tôi thực sự quan tâm.
* **Yêu cầu giải thích trước khi làm tiếp:** hỏi AI giải thích `prompt_prototype.py`, `autograder.py`, cách chạy autograder, cách dùng `.env` — để hiểu, không chỉ copy.
* **Đọc lại đề để kiểm tra AI:** nhờ đối chiếu worksheet mà tôi phát hiện AI bỏ sót Phase 3–6.
* **Duyệt nội dung trước khi commit:** xem lại từng file và danh sách sự việc của log này trước khi cho phép commit.

---

## 6. Bài học & lần sau sẽ làm khác

1. **AI mạnh ở tốc độ, cấu trúc và tìm nguồn; yếu ở dữ liệu nội bộ.** Mọi con số về quy trình thực tế của Vincons đều phải đi hỏi người trong cuộc — AI chỉ có thể ước tính.
2. **Nhãn "(ƯT)" là ranh giới giữa phân tích và hallucination.** Tôi sẽ luôn yêu cầu AI tách rõ số có nguồn và số ước tính.
3. **Kiểm tra diff trước khi commit, và duyệt riêng từng bước** (commit ≠ push). Một lệnh gộp nhiều bước khiến tôi mất cơ hội dừng lại.
4. **3 test đều Passed chưa chứng minh prompt an toàn.** Số test quá ít, `temperature=0` không đại diện cho mọi tình huống, và cách kiểm tra bằng tìm chuỗi có thể cho kết quả *"Passed"* giả.
5. **AI dễ nói "xong rồi" sớm.** Khi AI bảo *"bài cá nhân còn thiếu X"*, tôi vẫn nên tự đối chiếu với đề — lần này chính tôi phát hiện ra phần bị bỏ sót.
