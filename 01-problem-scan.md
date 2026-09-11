# 01 — Problem Scan & Quick-Assess (Vin Smart Future)

> **Vai trò:** AI Product Engineer tại Vin Smart Future
> **Người thực hiện:** Bach Doan — branch `doanduybach-2A202602515`
> **Phạm vi:** Phase 1 (SCAN) và Phase 2 (QUICK-ASSESS) của [01-worksheet.md](01-worksheet.md)

---

# 🔍 Phase 1 — SCAN

Quét qua 5 công ty thành viên (Vinpearl, Xanh SM, VinFast, Vinhomes, Vinmec/VinWonders) bằng **4 lenses**: Lặp lại · Tốn thời gian · AI có thể tốt hơn · Pain từ người khác.

| # | Subsidiary | Lens | Mô tả ngắn bài toán | Tín hiệu quy mô (ước tính) |
|---|------------|------|---------------------|----------------------------|
| 1 | **Vinpearl** | Tốn thời gian | Nhân viên Sales đoàn đọc email yêu cầu đặt phòng theo đoàn từ đại lý lữ hành/doanh nghiệp (đa ngôn ngữ, kèm Excel/PDF), **nhập tay** vào form, gửi Reservation kiểm tra quỹ phòng, tự tính giá rồi soạn email báo giá. | ~67 phút công/yêu cầu; khách chờ báo giá 4–10 giờ làm việc, trong khi đại lý thường hỏi giá nhiều resort cùng lúc. |
| 2 | **Xanh SM** | Lặp lại | CSKH xử lý báo **mất đồ trên xe**: khách mô tả mơ hồ (*"tối qua tầm 9h, đi từ Times City"*), CSKH phải tra lịch sử chuyến, gọi tài xế xác nhận, hẹn điểm trả đồ, cập nhật ticket. | 10–15 phút/ca; nhiều ca phải gọi tài xế 2–3 lần. |
| 3 | **VinFast** | AI có thể tốt hơn | Cố vấn dịch vụ tại xưởng ghi phiếu tiếp nhận từ **mô tả lỗi bằng lời** của khách (*"qua gờ giảm tốc kêu cụp cụp ở bánh trước"*) và phân loại sơ bộ bằng kinh nghiệm; phụ tùng chỉ được đặt sau khi KTV kiểm tra. | Xe nằm xưởng thêm 1–2 ngày khi chờ phụ tùng. |
| 4 | **Vinhomes** | Pain từ người khác | Phản ánh của cư dân trên app được lễ tân BQL **phân loại tay**, hay chuyển nhầm bộ phận (kỹ thuật / an ninh / vệ sinh); cư dân phàn nàn phản hồi chậm và rập khuôn. | Phản hồi đầu tiên có thể > 12 giờ. |
| 5 | **Vinmec** | Tốn thời gian | Nhân viên bảo lãnh viện phí **đối chiếu chỉ định điều trị với điều khoản** của từng hãng bảo hiểm (mỗi hãng một mẫu, một danh sách loại trừ) trước khi gửi hồ sơ bảo lãnh. | Bệnh nhân chờ 30–60 phút ở quầy khi xuất viện. |
| 6 | **Xanh SM** | Lặp lại | Giám sát đội xe **xem ảnh đầu ca thủ công** (tài xế tự chụp xe) để phát hiện trầy xước, xe bẩn, sai đồng phục. | Hàng nghìn ảnh/ngày, chỉ kiểm tra ngẫu nhiên nên dễ bỏ sót. |
| 7 | **VinWonders** | AI có thể tốt hơn | Chatbot/fanpage trả lời theo kịch bản cố định, không hiểu câu hỏi kết hợp (*"2 người lớn, 1 bé cao 1m2, thứ 7 này, có combo kèm buffet không?"*) → chuyển sang người thật, khách chờ lâu. | Lượng tin nhắn tăng mạnh vào mùa cao điểm/lễ. |

**Độ phủ lens:** Lặp lại (#2, #6) · Tốn thời gian (#1, #5) · AI có thể tốt hơn (#3, #7) · Pain từ người khác (#4).

### Tiêu chí chọn top 3
1. **Tần suất cao, người đau rõ ràng** (có actor cụ thể mất thời gian mỗi ngày).
2. **Phần việc cốt lõi là xử lý ngôn ngữ tự nhiên**: đây là chỗ LLM hơn hẳn rule-based. Bài toán chỉ cần tra cứu có cấu trúc thì không cần AI.
3. **Rủi ro khi AI sai kiểm soát được** bằng Human-in-the-loop hoặc Fallback.
4. **Đa dạng công ty và lens** để có cơ sở so sánh khi chọn bài deep-dive.

Loại khỏi top 3:
* **#5 (Vinmec bảo lãnh):** rủi ro tài chính và pháp lý cao, dữ liệu y tế nhạy cảm, khó có dữ liệu test.
* **#6 (ảnh đầu ca):** là bài toán Computer Vision, không phải xử lý ngôn ngữ; ngoài phạm vi prompt prototype của lab.
* **#4, #7:** đáng làm, nhưng giống worked example và các chatbot có sẵn trên thị trường; nhóm muốn thử bài toán ít được khai thác hơn.

→ **Top 3: #1 (Vinpearl), #2 (Xanh SM), #3 (VinFast).**

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

## Card #1 — Vinpearl: Báo giá đặt phòng theo đoàn (Group Booking)

```text
┌──────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                                │
├──────────────────────────────────────────────────────────────────────┤
│ Bài toán: Rút ngắn thời gian từ lúc nhận email yêu cầu đặt phòng đoàn
│   đến lúc gửi báo giá, bằng cách tự động trích xuất yêu cầu và soạn
│   nháp báo giá cho Sales duyệt.
│ Công ty thành viên: [ ] VinFast [ ] Xanh SM [ ] Vinhomes [ ] Vinmec
│                     [x] Khác: Vinpearl (khối Sales & Marketing)
│
│ Ai đang đau (Actor)?
│   • Sales đoàn (Group Sales Executive): nhập liệu tay, soạn báo giá
│   • Reservation/Revenue: bị kéo vào hàng đợi kiểm tra quỹ phòng
│   • Đại lý lữ hành/doanh nghiệp: chờ báo giá, dễ chốt với resort khác
│
│ Workflow thủ công hiện tại:
│   1. Đọc email + file đính kèm (Excel/PDF, Vi/En/Ko/Zh)
│   ──> 2. Nhập tay yêu cầu vào "Group Request Form" (Excel)
│   ──> 3. Gửi Reservation kiểm tra quỹ phòng (chờ 2–4 giờ)
│   ──> 4. Tính giá theo contract rate + phụ thu mùa + F&B/MICE
│   ──> 5. Soạn email báo giá theo ngôn ngữ khách & gửi
│
│ Bước nào tốn thời gian/lỗi nhất?
│   Bước 2 (⏱ 15 phút, hay sai ngày/số phòng) và Bước 5 (⏱ 15 phút).
│   Bước 3 là điểm chờ lâu nhất (⏳ 2–4 giờ trong hàng đợi).
│
│ AI có thể nhảy vào hỗ trợ ở bước nào?
│   • Bước 2: LLM trích xuất email → JSON theo schema, đánh dấu thiếu thông tin
│   • Bước 5: LLM soạn nháp báo giá đa ngôn ngữ, gắn [DRAFT_ONLY]
│   • Bước 3–4: KHÔNG dùng AI; thay bằng API quỹ phòng (read-only) + Rate Engine
│
│ Đo thành công bằng gì (Metric có số)?
│   • Thời gian thao tác của Sales: ~67 phút ──> ≤ 20 phút / yêu cầu
│   • Thời gian đến báo giá đầu tiên: 4–10 giờ ──> ≤ 2 giờ làm việc
│   • Trích xuất đúng ≥ 95% trường bắt buộc (ngày, số phòng theo hạng, số khách)
│
│ Quick Architecture: [ ] No AI  [x] Rule  [x] LLM  [ ] Agent
│   (LLM cho ngôn ngữ; Rule cho quỹ phòng & giá)
└──────────────────────────────────────────────────────────────────────┘
```

**🔎 Stress-test (đóng vai CFO + Trưởng phòng Vận hành khó tính):**

| Phản biện | Trả lời / Điều chỉnh |
|---|---|
| *"Con số 67 phút lấy ở đâu ra?"* | Chưa có số thật, đây là ước tính từ phân rã quy trình. **Điều chỉnh:** tuần đầu tiên phải bấm giờ thực tế (time-and-motion) trên 50–100 yêu cầu trước khi cam kết ROI. |
| *"Sao không bắt đại lý điền form web chuẩn? Rule-based là đủ."* | Đúng một phần. Đại lý lớn, thường xuyên nên chuyển sang form/API. Nhưng đại lý nhỏ và khách doanh nghiệp vẫn gửi email tự do, và form không làm bớt bước soạn báo giá. **Điều chỉnh:** làm song song, form cho đại lý lớn và LLM cho email tự do. |
| *"LLM bịa giá hoặc hứa phòng thì ai chịu?"* | LLM **không được viết con số giá**. Giá lấy từ Rate Engine, có code hậu kiểm đối chiếu từng con số. AI không bao giờ tự gửi email hay giữ phòng. |

---

## Card #2 — Xanh SM: Xử lý báo mất đồ trên xe

```text
┌──────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                                │
├──────────────────────────────────────────────────────────────────────┤
│ Bài toán: Giảm thời gian CSKH xử lý báo mất đồ trên xe (tìm đúng chuyến,
│   liên hệ tài xế, hẹn trả đồ).
│ Công ty thành viên: [ ] VinFast [x] Xanh SM [ ] Vinhomes [ ] Vinmec
│
│ Ai đang đau (Actor)?
│   • Nhân viên CSKH Xanh SM (xử lý ticket)
│   • Tài xế (bị gọi nhiều lần giữa ca chạy)
│   • Khách mất đồ (lo lắng, phải kể lại nhiều lần)
│
│ Workflow thủ công hiện tại:
│   1. Khách gọi hotline/chat, mô tả chuyến & món đồ
│   ──> 2. CSKH tra lịch sử chuyến theo SĐT / giờ / địa điểm
│   ──> 3. Gọi tài xế xác nhận có đồ trên xe không
│   ──> 4. Hẹn điểm/giờ trả đồ giữa khách và tài xế
│   ──> 5. Cập nhật & đóng ticket
│
│ Bước nào tốn thời gian/lỗi nhất?
│   Bước 2–3 (⏱ ~8 phút/ca) khi khách không nhớ rõ chuyến, đặt hộ người
│   khác, hoặc vẫy xe trả tiền mặt (không có lịch sử trong app).
│
│ AI có thể nhảy vào hỗ trợ ở bước nào?
│   • Bước 2: LLM chuẩn hóa mô tả tự do → truy vấn (khung giờ, điểm đi/đến)
│     rồi rule truy vấn DB chuyến đi, trả về 1–3 chuyến ứng viên
│   • Bước 3: soạn tin nhắn gửi tài xế qua app thay vì gọi điện
│
│ Đo thành công bằng gì (Metric có số)?
│   • Thời gian xử lý: ~12 phút ──> ≤ 5 phút / ca
│   • ≥ 90% ca tìm đúng chuyến ngay lần tra đầu tiên
│
│ Quick Architecture: [ ] No AI  [x] Rule  [x] LLM (phụ)  [ ] Agent
└──────────────────────────────────────────────────────────────────────┘
```

**🔎 Stress-test:**

| Phản biện | Trả lời / Điều chỉnh |
|---|---|
| *"Khách đặt qua app thì chuyến đã có ID sẵn. Thêm nút 'Báo mất đồ' trong lịch sử chuyến là xong, cần gì AI?"* | **Đồng ý.** Đây là điểm yếu lớn nhất của card: rule-based (nút trong app, tự gửi thông báo cho tài xế) giải quyết được phần lớn ca. LLM chỉ còn giá trị với nhóm nhỏ khách vẫy xe hoặc đặt qua tổng đài. |
| *"Mỗi ngày có bao nhiêu ca?"* | Chưa rõ. Nếu chỉ vài chục ca/ngày trên toàn hệ thống thì ROI của LLM thấp. |
| *"Rủi ro quyền riêng tư?"* | Không được tiết lộ SĐT tài xế/khách cho nhau. Mọi liên lạc phải đi qua kênh ẩn số của app. |

---

## Card #3 — VinFast: Phân loại mô tả lỗi xe khi đặt lịch dịch vụ

```text
┌──────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                                │
├──────────────────────────────────────────────────────────────────────┤
│ Bài toán: Chuẩn hóa mô tả lỗi bằng lời của khách thành triệu chứng có
│   cấu trúc + gợi ý nhóm hệ thống cần kiểm tra, giúp xưởng xếp đúng KTV
│   và chuẩn bị phụ tùng trước khi xe vào.
│ Công ty thành viên: [x] VinFast [ ] Xanh SM [ ] Vinhomes [ ] Vinmec
│
│ Ai đang đau (Actor)?
│   • Cố vấn dịch vụ (Service Advisor): ghi phiếu, trả lời khách
│   • Kỹ thuật viên: nhận xe mà không biết trước cần chuẩn bị gì
│   • Khách hàng: xe nằm xưởng lâu vì chờ phụ tùng
│
│ Workflow thủ công hiện tại:
│   1. Khách gọi/đặt lịch, mô tả triệu chứng bằng lời
│   ──> 2. Cố vấn ghi phiếu tiếp nhận (text tự do, ⏱ ~10 phút)
│   ──> 3. Xe vào xưởng, KTV kiểm tra & chẩn đoán
│   ──> 4. Đặt phụ tùng nếu kho không có (+1–2 ngày)
│   ──> 5. Sửa chữa & bàn giao
│
│ Bước nào tốn thời gian/lỗi nhất?
│   Bước 3–4: chờ phụ tùng vì không biết trước (+1–2 ngày/xe).
│
│ AI có thể nhảy vào hỗ trợ ở bước nào?
│   • Bước 2: LLM chuẩn hóa mô tả → triệu chứng có cấu trúc + gợi ý nhóm
│     hệ thống (treo, phanh, điện, pin...) để xếp lịch KTV phù hợp
│   • Kết hợp mã lỗi DTC từ xe (nếu xe kết nối) — phần này là rule
│
│ Đo thành công bằng gì (Metric có số)?
│   • Gợi ý đúng nhóm hệ thống ≥ 85% so với chẩn đoán cuối của KTV
│   • Giảm 20% số ngày xe nằm xưởng do chờ phụ tùng
│
│ Quick Architecture: [ ] No AI  [x] Rule  [x] LLM  [ ] Agent
└──────────────────────────────────────────────────────────────────────┘
```

**🔎 Stress-test:**

| Phản biện | Trả lời / Điều chỉnh |
|---|---|
| *"Gợi ý sai ở hệ thống phanh/lái thì sao?"* | Rủi ro an toàn thật: KTV có thể bị "neo" vào gợi ý của AI (anchoring bias). Gợi ý chỉ được dùng để **xếp lịch/chuẩn bị**, không được hiển thị như chẩn đoán. |
| *"Mã lỗi DTC từ xe chính xác hơn lời khách kể."* | Đúng. Với xe có kết nối, rule đọc DTC nên là nguồn chính, LLM chỉ bổ sung cho lỗi cơ khí không có mã (tiếng kêu, rung). |
| *"Có dữ liệu phiếu tiếp nhận + kết quả chẩn đoán đã gán nhãn không?"* | Chưa rõ. Nếu phiếu cũ viết tay/không đồng nhất thì chưa đo được độ chính xác → **NOT YET**. |

---

# 🗳️ Quyết định chọn bài toán Deep-Dive

| Tiêu chí (1–3, cao = tốt) | Card #1 Vinpearl | Card #2 Xanh SM | Card #3 VinFast |
|---|:---:|:---:|:---:|
| Tần suất & mức đau | 3 | 2 | 2 |
| Phần việc cốt lõi là ngôn ngữ tự nhiên (LLM fit) | 3 | 1 | 3 |
| Rủi ro khi AI sai kiểm soát được | 3 | 3 | 1 |
| Dữ liệu để test có sẵn | 2 | 3 | 1 |
| Rule-based **chưa** đủ giải quyết | 3 | 1 | 2 |
| **Tổng** | **14** | **10** | **9** |

→ Nhóm chọn **Card #1 — Vinpearl: Báo giá đặt phòng theo đoàn** để deep-dive.

* **Card #2 bị loại** vì chính phần stress-test cho thấy rule-based (nút "Báo mất đồ" trong app) giải quyết được phần lớn ca. Dùng LLM ở đây là "AI vì AI".
* **Card #3 để lại (NOT YET)** vì rủi ro an toàn xe cao và chưa chắc có dữ liệu chẩn đoán đã gán nhãn để đánh giá.
* **Card #1 được chọn** vì phần tốn công nhất (đọc email tự do đa ngôn ngữ, soạn báo giá) đúng là việc LLM làm tốt. Các phần rủi ro (giá, quỹ phòng) tách được sang rule. Khi AI sai, hậu quả chặn được bằng bước duyệt của Sales.
