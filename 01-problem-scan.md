# 01 — Problem Scan: AI Product Scoping (Vin Smart Future)

> **Vai trò:** AI Product Engineer tại Vin Smart Future.
> File này gồm **Phase 1 (SCAN)** và **Phase 2 (QUICK-ASSESS)** theo `01-worksheet.md`.
> Các con số thời gian/tỉ lệ trong file là **ước tính giả định để scoping** (đánh dấu *~*), cần được xác nhận lại với bộ phận vận hành trước khi dùng làm baseline.

---

# 🔍 Phase 1 — SCAN (Cá nhân)

Dùng **4 Lenses** (Lặp lại / Tốn thời gian / AI-upgrade / Stakeholder Pain) quét qua vận hành của các công ty thành viên Vingroup.

## 📝 List bài toán của tôi

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **VinFast** | Lặp lại | Cố vấn dịch vụ (Service Advisor) tại xưởng đọc từng yêu cầu bảo hành/sửa chữa khách gửi qua app VinFast bằng tiếng Việt đời thường (*"xe kêu cụp cụp bánh trước khi qua gờ"*, *"sạc mãi không đầy"*), phải gọi lại hỏi thêm rồi mới nhập được mã nhóm lỗi + tạo lịch hẹn. ~8-10 phút/ticket, hàng trăm ticket/ngày/xưởng, phân loại sai → khách đến xưởng nhưng thiếu phụ tùng/kỹ thuật viên phù hợp. |
| 2 | **Xanh SM** | Pain từ người khác | Khách khiếu nại cước phí (*"tài xế đi vòng"*, *"app tính sai km"*) và tài xế phản ánh ngược lại. CSKH phải mở GPS log của chuyến, ghi chú tài xế, lịch sử chat trong app, rồi tự viết kết luận hoàn/không hoàn tiền. ~15-20 phút/case, khách chờ 24-48h, tài xế bức xúc vì bị trừ điểm oan. |
| 3 | **Vinhomes** | Tốn thời gian | Ban Quản Lý tòa nhà duyệt hồ sơ **đăng ký thi công/sửa chữa nội thất** của cư dân: kiểm tra bản vẽ, hạng mục (có đục tường chịu lực? đổi vị trí ống nước?), giờ thi công, ký quỹ… đối chiếu với quy chế từng tòa. Hồ sơ thường thiếu, BQL phải trả lại nhiều vòng → SLA ~2-3 ngày làm việc, cư dân phàn nàn vì "nộp mấy lần mới xong". |
| 4 | **Vinmec** | AI-upgrade | Tổng đài/chatbot đặt lịch khám: nhân viên hỏi triệu chứng theo kịch bản cứng rồi tự đoán chuyên khoa (Tim mạch vs Hô hấp vs Tiêu hóa…). Phản hồi rập khuôn, ~5-7 phút/cuộc gọi, một tỉ lệ bệnh nhân bị đặt sai khoa và phải chuyển khoa tại viện → mất slot bác sĩ và trải nghiệm kém. |
| 5 | **Vinpearl** | Lặp lại | Nhân viên Reservation đọc email đặt phòng **theo đoàn** từ công ty lữ hành (mỗi công ty một format: text, bảng Excel đính kèm, tiếng Anh/Việt/Hàn/Trung), tự nhập tay từng dòng vào PMS (số phòng, loại phòng, ngày in/out, ăn sáng, ghi chú đặc biệt). ~25-40 phút/đoàn, dễ nhập sai ngày/loại phòng, mùa cao điểm tồn đọng email 1-2 ngày. |
| 6 | **VinFast** (Supply chain) | Pain từ người khác | Planner nhà máy nhận thông báo trễ hàng/thay đổi số lượng từ nhà cung cấp linh kiện qua email (tiếng Anh/Trung, không có format chuẩn). Phải tự đọc, gom vào Excel theo dõi rồi báo cho dây chuyền sản xuất. Thông tin đến muộn → dây chuyền thiếu linh kiện, planner phàn nàn "cả ngày chỉ đọc email". |

## 🧭 Nhận xét nhanh sau khi Scan

| Tiêu chí | #1 VinFast ticket | #2 Xanh SM khiếu nại cước | #3 Vinhomes thi công | #4 Vinmec đặt lịch | #5 Vinpearl booking đoàn | #6 VinFast supply chain |
|---|---|---|---|---|---|---|
| Có ngôn ngữ tự nhiên phi cấu trúc cần xử lý? | ✅ Mạnh | ✅ Mạnh | ✅ Vừa (hồ sơ + bản vẽ) | ✅ Mạnh | ✅ Mạnh (đa ngôn ngữ) | ✅ Mạnh (đa ngôn ngữ) |
| Rule-based làm được không? | ❌ Khó (mô tả tự do) | ⚠️ Một phần (rule km/tiền) | ⚠️ Một phần (checklist) | ❌ Khó | ❌ Khó (format bất kỳ) | ❌ Khó |
| Rủi ro khi AI sai | Trung bình | **Cao** (tiền, uy tín tài xế) | Trung bình (an toàn kết cấu → cần HITL) | **Rất cao** (y tế) | Trung bình (đặt sai phòng) | Trung bình |
| Dữ liệu mẫu có sẵn? | ✅ Ticket lịch sử | ✅ Log GPS + case | ⚠️ Hồ sơ giấy/scan | ⚠️ Nhạy cảm, khó lấy | ✅ Email lưu trữ | ✅ Email lưu trữ |
| Tần suất / quy mô | Rất cao | Cao | Trung bình | Cao | Trung bình-cao | Trung bình |

**Top 3 đưa vào Phase 2:** **#1 VinFast** (tần suất cao, dữ liệu sẵn, AI-fit rõ), **#5 Vinpearl** (bài toán extraction điển hình, rủi ro vừa, dễ đo), **#2 Xanh SM** (pain rõ từ cả khách lẫn tài xế, có dữ liệu GPS để AI đối chiếu; rủi ro cao nên là bài tốt để luyện Operational Boundary).

*Loại #4 Vinmec vì rủi ro y tế quá cao cho scope lab; #3 Vinhomes vì checklist rule-based có thể giải quyết được 70%; #6 là biến thể của #5 (email extraction) nên gộp ý tưởng vào #5.*

---

# 🃏 Phase 2 — QUICK-ASSESS (Cá nhân)

Chọn top 3 từ danh sách SCAN: **#1 VinFast (ticket bảo hành)**, **#5 Vinpearl (booking đoàn)**, **#2 Xanh SM (khiếu nại cước)**.

## Card #1 — VinFast: Phân loại yêu cầu bảo hành từ mô tả tiếng Việt của khách

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán (1 câu): Khách mô tả lỗi xe bằng tiếng Việt đời    │
│ thường trên app VinFast, Cố vấn dịch vụ phải tự dịch sang   │
│ mã nhóm lỗi kỹ thuật + chọn xưởng/kỹ thuật viên/phụ tùng    │
│ phù hợp trước khi xác nhận lịch hẹn.                        │
│ Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác                   │
│                                                             │
│ Ai đang đau (Actor)? Cố vấn dịch vụ (Service Advisor) tại   │
│ xưởng (~150-300 ticket/ngày/xưởng lớn); khách hàng chờ      │
│ xác nhận lịch; kỹ thuật viên nhận job thiếu thông tin.      │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Khách gửi mô tả + ảnh/video qua app / hotline          │
│   ──> 2. SA đọc ticket, tra lịch sử xe (VIN, km, gói BH)    │
│   ──> 3. SA gọi lại khách hỏi thêm (khi nào kêu, tốc độ..)  │
│   ──> 4. SA chọn mã nhóm lỗi (Gầm/Điện/Pin/HMI...) + ước    │
│          lượng giờ công, kiểm tra phụ tùng tồn kho          │
│   ──> 5. Xác nhận lịch hẹn + tạo Work Order cho KTV         │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3-4 (⏱ ~6-8 phút/lượt│
│ trên tổng ~10 phút; ~15-20% ticket phân loại sai nhóm lỗi   │
│ → khách đến xưởng thiếu phụ tùng, phải hẹn lại)             │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3-4:             │
│   - Đọc mô tả + lịch sử xe → đề xuất TOP-3 nhóm lỗi kèm     │
│     độ tin cậy + lý do                                      │
│   - Sinh sẵn 2-3 câu hỏi làm rõ để SA hỏi khách (hoặc       │
│     chatbot hỏi ngay trong app)                             │
│   - Gợi ý phụ tùng cần chuẩn bị theo nhóm lỗi               │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│   - Thời gian xử lý ticket: ~10 phút ──> dưới 3 phút        │
│   - Tỉ lệ phân loại đúng nhóm lỗi (so với KTV kết luận):    │
│     ~80% ──> ≥ 92%                                          │
│   - Tỉ lệ khách phải hẹn lại vì thiếu phụ tùng: giảm 50%    │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
│   (LLM classify + structured JSON; SA luôn là người chốt)   │
└─────────────────────────────────────────────────────────────┘
```

**Ranh giới sơ bộ:** AI **không** được kết luận nguyên nhân lỗi cuối cùng, không báo giá, không hứa thời gian sửa, không tư vấn khách "cứ chạy tiếp" với lỗi liên quan phanh/pin/lái. Mọi output là *đề xuất* cho SA duyệt.

---

## Card #2 — Vinpearl: Trích xuất email đặt phòng theo đoàn vào PMS

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán (1 câu): Nhân viên Reservation phải đọc email đặt  │
│ phòng theo đoàn (mỗi công ty lữ hành một format, nhiều      │
│ ngôn ngữ, kèm Excel) rồi nhập tay từng dòng vào PMS.        │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [x] Khác: Vinpearl         │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên Reservation / Group Desk     │
│ (~20-40 email đoàn/ngày/cụm resort mùa cao điểm); Sales     │
│ B2B bị lữ hành hối; khách đoàn nhận sai phòng khi check-in. │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Email đoàn vào hộp thư chung (text + Excel/PDF)        │
│   ──> 2. NV đọc, dịch (Hàn/Trung/Anh), hiểu yêu cầu         │
│   ──> 3. Tra quỹ phòng trống theo loại/ngày trong PMS       │
│   ──> 4. Nhập tay từng dòng: tên, loại phòng, in/out, ăn    │
│          sáng, giường phụ, ghi chú (dị ứng, honeymoon...)   │
│   ──> 5. Soạn email xác nhận/đàm phán lại nếu thiếu phòng   │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 + 4 (⏱ ~25-40 phút │
│ /đoàn; lỗi nhập sai ngày/loại phòng ~5-8% đoàn; mùa cao     │
│ điểm email tồn 1-2 ngày chưa trả lời)                       │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 + 4 + 5:       │
│   - Đọc email + đính kèm → JSON chuẩn (rooming list) kèm    │
│     trường nào "không chắc / thiếu" để NV kiểm tra          │
│   - Draft email xác nhận hoặc email hỏi lại thông tin thiếu │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│   - Thời gian xử lý 1 đoàn: ~30 phút ──> dưới 8 phút        │
│   - Độ chính xác trường trích xuất (ngày, loại, số phòng):  │
│     ≥ 97% trên tập test 100 email lịch sử                   │
│   - Thời gian phản hồi lữ hành: 24-48h ──> dưới 4h          │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
│   (LLM extraction → NV review → nhập PMS; chưa cần agent)   │
└─────────────────────────────────────────────────────────────┘
```

**Ranh giới sơ bộ:** AI **không** được tự ghi vào PMS, không tự xác nhận giá/khuyến mãi, không tự trả lời lữ hành khi chưa được duyệt. Trường nào không tìm thấy trong email phải để `null` chứ không được "đoán".

---

## Card #3 — Xanh SM: Hỗ trợ xử lý khiếu nại cước phí

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán (1 câu): Khi khách khiếu nại cước ("tài xế đi      │
│ vòng", "app tính sai"), CSKH phải tự gom GPS log, ghi chú   │
│ tài xế, chat trong app rồi viết kết luận hoàn tiền hay không│
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác                   │
│                                                             │
│ Ai đang đau (Actor)? CSKH tier-1 (~200-400 case/ngày toàn   │
│ quốc); khách chờ 24-48h; tài xế bị khóa app/trừ điểm oan.   │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Khách gửi khiếu nại (text tự do) qua app/hotline       │
│   ──> 2. CSKH mở chuyến, tải GPS log, so với route gợi ý    │
│   ──> 3. Đọc ghi chú tài xế + lịch sử chat + lịch sử KH     │
│   ──> 4. Viết kết luận + quyết định hoàn tiền (theo policy) │
│   ──> 5. Soạn phản hồi cho khách và thông báo cho tài xế    │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-4 (⏱ ~12-15 phút   │
│ /case trên tổng ~18 phút; kết luận không nhất quán giữa     │
│ các CSKH → tài xế khiếu nại ngược ~10% case)                │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 1 + 3 + 5:       │
│   - Phân loại khiếu nại (đi vòng / sai km / phụ phí / thái  │
│     độ / khác) + trích xuất thông tin từ text tự do         │
│   - Tóm tắt bằng chứng (GPS đã tính rule sẵn: km chênh,     │
│     % lệch route) thành 1 đoạn + đề xuất nhãn "có căn cứ /  │
│     không / cần người xem"                                  │
│   - Draft phản hồi cho khách và tài xế theo template        │
│   ⚠️ Việc so GPS vs route và tính tiền chênh = RULE, không  │
│      phải LLM.                                              │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│   - Thời gian xử lý: ~18 phút ──> dưới 6 phút/case          │
│   - Thời gian phản hồi khách: 24-48h ──> dưới 4h            │
│   - Tỉ lệ tài xế khiếu nại ngược: ~10% ──> dưới 4%          │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
│   (Rule tính bằng chứng số → LLM phân loại/tóm tắt/draft →  │
│    CSKH quyết định)                                         │
└─────────────────────────────────────────────────────────────┘
```

**Ranh giới sơ bộ:** AI **tuyệt đối không** tự quyết định hoàn tiền, không tự khóa/trừ điểm tài xế, không gửi phản hồi cho khách khi chưa được CSKH duyệt, không dùng thông tin ngoài dữ liệu chuyến được cung cấp.

---

## 🗳️ Đề xuất bài toán đưa vào Deep-Dive (Phase 3)

**Đề xuất: Card #1 — VinFast phân loại yêu cầu bảo hành.**

| | Card #1 VinFast | Card #2 Vinpearl | Card #3 Xanh SM |
|---|---|---|---|
| Giá trị / quy mô | Rất cao (hàng trăm ticket/ngày/xưởng, ảnh hưởng trực tiếp trải nghiệm sau bán) | Trung bình (theo mùa) | Cao |
| AI-fit | Rất rõ: NLP tiếng Việt đời thường → phân loại; rule không làm được | Rõ: extraction | Rõ nhưng phần lõi (đối chiếu GPS) là rule |
| Rủi ro khi sai | Trung bình, kiểm soát được bằng HITL (SA chốt) | Trung bình | **Cao** (tiền, lao động tài xế) |
| Dữ liệu test | Ticket + kết luận KTV lịch sử → có ground truth sẵn | Email lịch sử | Case lịch sử + GPS |
| Prototype Phase 4 | Dễ: text vào → JSON ra, adversarial test rõ (dụ AI chẩn đoán/báo giá) | Dễ | Phức tạp hơn (cần mock GPS) |

*Card #3 giữ làm phương án dự phòng nếu nhóm có thành viên hiểu vận hành Xanh SM tốt hơn.*
