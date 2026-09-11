# 03 — AI Log & Reflection (Cá nhân)

> Nhật ký sử dụng AI (Claude Code trong VS Code + Gemini 3.6 Flash qua API) làm *thought-partner* trong Lab 02.
> Nguyên tắc ghi log: **AI giúp gì → AI sai/lệch ở đâu → tôi sửa thế nào**. Ghi trung thực, kể cả phần AI làm thay tôi.

---

## 1. Cách tôi dùng AI trong buổi lab

| Phase | Tôi làm gì | AI làm gì |
|---|---|---|
| 0-1 SCAN | Yêu cầu AI đọc worksheet, README, worked example, inspiration kit và tóm tắt "cần làm gì". Chốt tiêu chí: **không trùng** 3 bài trong worked example. | Tóm tắt deliverable + rubric; đề xuất 6 bài toán theo 4 lenses, tự thêm bảng so sánh (NLP? rule làm được? rủi ro? dữ liệu?) để xếp hạng. |
| 2 QUICK-ASSESS | Duyệt top 3 AI đề xuất (VinFast ticket / Vinpearl booking / Xanh SM khiếu nại). | Viết 3 Quick Cards + ranh giới sơ bộ; đề xuất chọn VinFast cho Deep-Dive kèm bảng lý do. |
| 3 DEEP-DIVE | Chọn Card #1 VinFast. | Vẽ current-state (ASCII + Mermaid), Problem Statement 6-field, AI-Fit Matrix, Future flow với Guardrail/HITL/Fallback. |
| 4 PROTOTYPE | Cấp API key qua `.env`; quyết định giữ file Xanh SM cho autograder + tạo file VinFast riêng. | Viết `vinfast_triage_prototype.py` (system prompt, JSON schema, 5 test, guardrail rule), chạy, phân tích kết quả, sửa guardrail, chạy lại. |
| 5 EVALUATE | Đọc lại quyết định GO và điều kiện gate. | Viết checklist + justification dựa trên kết quả test thật. |

Prompt hiệu quả nhất tôi dùng: *"đọc file 01-worksheet.md xem cần phải làm gì"* rồi *"bắt đầu từ phase 1 đi"* → để AI **tự đọc toàn bộ ngữ cảnh** (README, example, autograder) trước khi làm, thay vì tôi mô tả lại. Nhờ vậy AI phát hiện được vấn đề autograder ở mục 2.3 mà tôi không hề biết.

---

## 2. AI giúp được gì (những chỗ thực sự có giá trị)

1. **Đọc và đối chiếu nhiều file cùng lúc.** AI phát hiện tên deliverable trong worksheet (`02-lab/`) khác README (`01-problem-scan.md`, `02-deep-dive-report.md`, `03-ai-log.md`, `04-workflow-diagram.png`) và đi theo README + autograder (căn cứ chấm điểm thật).
2. **Ép kỷ luật "Problem first, AI second".** Ở AI-Fit Matrix, AI tự lập luận vì sao rule-based *không* làm được phần ngôn ngữ nhưng *vẫn phải dùng rule* cho tra kho, gói bảo hành, guardrail — thay vì "dùng LLM cho tất cả".
3. **Thiết kế adversarial test có chủ đích.** 4 test bám đúng 4 ranh giới trong Problem Statement (báo giá / "cứ chạy tiếp" / giả danh quản lý bỏ HITL / ngoài phạm vi), mỗi test có hàm check riêng thay vì chỉ nhìn output bằng mắt.
4. **Phân tích kết quả fail thay vì chỉ báo fail.** Khi lần chạy đầu 3/5, AI đọc từng JSON và chỉ ra model đã đúng, guardrail mới sai (xem mục 3.2).

---

## 3. AI sai / lệch ở đâu và tôi sửa thế nào

### 3.1. Model `gemini-2.5-flash` không còn tồn tại
- **Sai:** Worksheet và starter code ghi `gemini-2.5-flash`; AI viết file VinFast theo đó. Chạy → API trả `404: no longer available to new users, use gemini-3.6-flash`.
- **Sửa:** Đổi sang `gemini-3.6-flash` (file Xanh SM có sẵn đã dùng model này). **Bài học:** tên model trong tài liệu học là *snapshot*, luôn phải kiểm tra bằng một lần gọi thật trước khi viết nhiều code.

### 3.2. Guardrail rule do AI viết tạo false positive (lỗi thiết kế quan trọng nhất)
- **Sai:** Test 1 (dụ báo giá) và Test 4 (ngoài phạm vi) bị đánh **Failed** dù model trả lời đúng. Nguyên nhân: regex tiền và keyword "tesla/cổ phiếu" quét **toàn bộ JSON**, kể cả trường `refusals` — nơi model ghi *"Từ chối đưa ra con số ước tính (2 triệu hay 5 triệu)"*. Model nhắc lại yêu cầu của khách để giải thích lý do từ chối — đó là hành vi mong muốn, không phải vi phạm.
- **Sửa:** Guardrail chỉ quét các trường **khách hàng nhìn thấy** (`customer_message_draft`, `suggested_parts_to_prepare`, `clarifying_questions`, `reason`); test ngoài phạm vi kiểm tra **hành vi tư vấn thật** (*"nên mua"*, *"bền hơn"*) thay vì tên chủ đề. Chạy lại: 5/5.
- **Bài học:** Ranh giới phải được định nghĩa theo **kênh** (nội bộ vs khách hàng), không theo từ khóa. Nếu đưa guardrail thô này vào production, ticket sẽ rơi về hàng đợi thủ công (Fallback A) một cách vô ích và SA sẽ mất niềm tin vào hệ thống.

### 3.3. Con số vận hành là do AI "ước lượng hợp lý", không phải dữ liệu thật
- **Lệch:** AI điền `~150-300 ticket/ngày/xưởng`, `~15-20% sai nhóm lỗi`, `~10 phút/ticket`, `~23 giờ SA/ngày`… Các con số này **nghe hợp lý nhưng không có nguồn** — đây là dạng hallucination "mềm" nguy hiểm nhất vì khó nhận ra.
- **Sửa:** Yêu cầu AI đánh dấu `~` và ghi rõ *"ước tính để scoping, cần xác nhận với Khối Dịch vụ Hậu mãi"* ở đầu mỗi file và trong sơ đồ. Ở Phase 5, đưa việc **lấy ≥ 500 ticket thật và đo baseline** thành gate bắt buộc trước khi bật tính năng cho SA.
- **Bài học:** Với bài scoping, AI rất giỏi tạo *cấu trúc* nhưng *số liệu* phải đến từ người vận hành. Rubric G2 chấm "metric có số **bám sát thực tế**" — số của AI chỉ là placeholder.

### 3.4. Xung đột giữa worksheet và autograder
- **Phát hiện (AI):** Autograder chấm `SYSTEM_PROMPT` bằng 3 từ khóa của bài Xanh SM (`draft_only`, `5%`, `dispatch_mobile_charger`) và `__main__` check cứng 2 test Xanh SM. Nếu viết lại `prompt_prototype.py` cho bài VinFast của nhóm sẽ mất điểm tiêu chí 1 và có thể tiêu chí 5.
- **Quyết định (tôi):** Giữ `prompt_prototype.py` (Xanh SM, 2/2 pass, autograder 5.0/5.0) và tạo `vinfast_triage_prototype.py` riêng cho bài toán nhóm. AI đưa ra 3 phương án kèm hệ quả điểm số; tôi chọn, không phải AI tự quyết.

### 3.5. Sơ đồ PNG lần đầu hỏng
- **Sai:** AI vẽ SVG có emoji (🔴🔄⏱) → `rsvg-convert` không có font emoji, render thành chấm đen; chú giải đè lên nhãn; chữ tràn hộp.
- **Sửa:** AI tự xem lại ảnh, bỏ emoji, dời chú giải, rút gọn chữ, render lại. **Bài học:** với output hình ảnh, phải *nhìn* kết quả — không tin "đã render thành công".

---

## 4. Phản tư cá nhân

- **Điều tôi làm tốt:** đặt AI vào chế độ đọc-toàn-bộ-ngữ-cảnh trước khi làm; ra quyết định ở các điểm rẽ quan trọng (chọn Card #1, xử lý xung đột autograder, cấp key qua `.env` thay vì dán vào code).
- **Điều tôi làm chưa tốt:** phần lớn nội dung `.md` là AI viết, tôi mới duyệt ở mức "đọc qua và ok". Với Problem Statement và các con số, tôi cần ngồi lại với nhóm để **thay số ước tính bằng số nhóm tin được** (hoặc ít nhất là số có lập luận), nếu không bài chỉ đẹp về cấu trúc.
- **Điều tôi sẽ làm khác lần sau:** viết adversarial test *trước* khi AI viết system prompt (test-first), và yêu cầu AI liệt kê "những gì tôi đang giả định" sau mỗi phase để tôi kiểm tra thay vì tự tin vào output trôi chảy.
- **Vai trò của AI trong lab này:** là *partner soạn thảo + phản biện kỹ thuật* rất mạnh, nhưng **không thay được người biết quy trình thật**. Bằng chứng: chỗ AI sai nhiều nhất không phải code, mà là *dữ liệu và ranh giới* — đúng hai thứ rubric coi trọng nhất.
