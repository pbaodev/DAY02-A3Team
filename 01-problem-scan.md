# 01 — Problem Scan & Quick Assess (A3Team — Vin Smart Future)

> **Nhóm thực hiện:** A3Team — Vin Smart Future  
> **Đơn vị giả định:** Vin Smart Future (Khối công nghệ hợp nhất Vingroup: *VinFast / Xanh SM / Vinhomes / Vinmec / Vinpearl / VinUni*)  
> **Bài toán Deep-Dive được lựa chọn:** **Vinmec — Soạn thảo tóm tắt hồ sơ xuất viện thông minh (Discharge Summary Co-pilot)**

---

## 🔍 Phase 1 — SCAN: Quét cơ hội tối ưu hóa bằng AI (5+ bài toán, 4 Lenses)

Áp dụng phương pháp luận **4 Lenses** quét qua toàn bộ chuỗi vận hành của các công ty thành viên Vingroup nhằm tìm kiếm các điểm rò rỉ hiệu suất, lãng phí thời gian và điểm nghẽn trải nghiệm:

| # | Subsidiary | Tên bài toán / Nghiệp vụ | Lens | Mô tả ngắn bài toán & Điểm nghẽn thực tế | Tần suất & Ước tính tổn thất |
|---|------------|--------------------------|------|-------------------------------------------|------------------------------|
| 1 | **Vinmec** | Soạn thảo tóm tắt hồ sơ xuất viện (Discharge Summary) | **Tốn thời gian** | Bác sĩ điều trị phải mở hàng loạt tab EMR (xét nghiệm, chẩn đoán hình ảnh, đơn thuốc, biên bản mổ) để gõ Word tóm tắt bệnh án và dịch thuật ngữ sang tiếng Việt cho bệnh nhân. | 120 ca/ngày toàn viện; tốn **20–30 phút/ca** (~50 giờ bác sĩ/ngày); dồn ứ giường bệnh 2–4 tiếng. |
| 2 | **Vinmec** | Phân loại & Gợi ý chuyên khoa ban đầu (Triage Chatbot) | **Pain từ người khác** | Bệnh nhân đặt lịch qua tổng đài/chatbot tự mô tả triệu chứng sơ sài, nhân viên tiếp đón xếp nhầm khoa (ví dụ: đau ngực do trào ngược dạ dày xếp vào Tim mạch thay vì Tiêu hóa), bệnh nhân phải chuyển khoa 2–3 lần. | 25–30% ca đặt khám ban đầu bị chuyển khoa; bệnh nhân chờ thêm 45–60 phút; lãng phí slot khám của bác sĩ. |
| 3 | **Xanh SM** | Điều phối sự cố cạn pin thực địa (Emergency Battery Dispatch) | **Tốn thời gian** | Tài xế taxi điện báo hết pin khẩn cấp giữa đường; điều phối viên phải tra cứu thủ công tọa độ GPS, tìm trạm sạc VinFast còn trụ trống hoặc điều xe sạc di động qua nhiều công cụ rời rạc. | 15–20 phút/lượt xử lý; rủi ro xe chết máy giữa đường gây ùn tắc và hỏng cell pin nếu pin giảm sâu dưới 5%. |
| 4 | **Vinhomes** | Phân loại & Điều hướng phản ánh cư dân (Resident Ticket Router) | **Lặp lại** | Nhân viên CSKH BQL phải đọc thủ công từng phản ánh gửi qua App Vinhomes Resident (mất nước, hỏng đèn hành lang, tiếng ồn thi công, gửi xe) để gán nhãn và chuyển tiếp về đúng BQL từng tòa nhà. | 300–500 ticket/ngày/đại đô thị; tốn 8–10 phút/ticket; cư dân bức xúc vì chờ phản hồi trung bình 8–12 tiếng. |
| 5 | **VinFast** | Chẩn đoán sơ bộ mã lỗi xe từ mô tả tiếng Việt (Fault Code Predictor) | **AI có thể tốt hơn** | Khách hàng gọi tổng đài mô tả âm thanh và hiện tượng bất thường bằng ngôn ngữ đời thường (*"đi qua gờ giảm tốc nghe cụp cụp ở giảm xóc trước bên phụ"*), nhân viên trực tổng đài không có chuyên môn cơ khí sâu nên phân loại mã lỗi sai lệch. | 30% ticket kỹ thuật bị phân loại sai; kỹ sư xưởng dịch vụ mất thêm 30 phút kiểm tra lại từ đầu; kéo dài thời gian sửa chữa. |
| 6 | **Vinpearl** | Tổng hợp & Phân tích phản hồi khách hàng đa kênh (Voice of Customer) | **Lặp lại** | Bộ phận QA/Vận hành tổng hợp hàng nghìn đánh giá mỗi tuần từ Agoda, Booking.com, Google Maps và phiếu khảo sát giấy để phân loại khiếu nại (buồng phòng, buffet, thái độ nhân viên) và làm báo cáo tuần. | Tốn 2 ngày công/tuần của chuyên viên QA; báo cáo chậm trễ 3–5 ngày khiến các phản ánh tiêu cực không được khắc phục tức thì. |

> **Phân bổ Lenses:**
> - 🔄 *Lặp lại (Repetitive):* Bài toán #4, #6
> - ⏱ *Tốn thời gian (Time-consuming):* Bài toán #1, #3
> - 🤖 *AI có thể tốt hơn (AI-upgrade):* Bài toán #5
> - 💢 *Pain từ người khác (Stakeholder Pain):* Bài toán #2  
> *(Đạt 4/4 lenses — vượt yêu cầu tối thiểu $\ge 3$ lenses của bài Lab).*

---

## 🃏 Phase 2 — QUICK-ASSESS: 3 Thẻ bài toán tiêu biểu (Top 3 Cards)

Nhóm lựa chọn 3 bài toán có tiềm năng ứng dụng AI cao nhất để hoàn thiện 3 thẻ phân tích nhanh:

### QUICK PROBLEM CARD #1 — Vinmec Discharge Summary Co-pilot (LỰA CHỌN DEEP-DIVE)

```text
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1 (CHỌN DEEP-DIVE)                                                   │
│                                                                                          │
│ Bài toán (1 câu): Tự động trích xuất dữ liệu EMR và soạn thảo bản nháp tóm tắt xuất viện │
│ có cấu trúc, diễn giải bằng ngôn ngữ bình dân cho bệnh nhân, hỗ trợ bác sĩ duyệt và ký.  │
│ Công ty thành viên: [x] Vinmec   [ ] VinFast   [ ] Xanh SM   [ ] Vinhomes                │
│                                                                                          │
│ Ai đang đau (Actor)?                                                                     │
│ - Bác sĩ điều trị / Bác sĩ nội trú: Quá tải công việc hành chính giấy tờ cuối ca trực.   │
│ - Điều dưỡng: Mất thời gian đối chiếu đơn thuốc và giải thích lại nhiều lần cho bệnh nhân│
│ - Bệnh nhân & Người nhà: Mệt mỏi vì phải chờ đợi 2–4 tiếng sau khi có quyết định ra viện.│
│                                                                                          │
│ Workflow thủ công hiện tại (5 bước):                                                     │
│   1. Mở HIS/EMR gom dữ liệu rời rạc (lab, imaging, biên bản mổ, đơn thuốc) (⏱ 5 min)    │
│   ──> 2. Gõ Word soạn tóm tắt + dịch thuật ngữ y khoa sang tiếng Việt dễ hiểu (⏱ 15 min) 🔴│
│   ──> 3. In bản thảo đưa Điều dưỡng kiểm tra đối chiếu danh mục thuốc (⏱ 5 min) 🔄       │
│   ──> 4. Bác sĩ kiểm tra lần cuối, ký tay/ký số đóng bệnh án (⏱ 3 min)                  │
│   ──> 5. Bàn giao giấy xuất viện, dặn dò lịch tái khám và trả viện phí (⏱ 2 min)         │
│                                                                                          │
│ Bước tốn thời gian / dễ sai sót nhất?                                                    │
│ ──> Bước 2 (⏱ 15–20 phút/ca). Tỷ lệ thiếu sót thông tin thuốc hoặc ngày tái khám ~12%.   │
│                                                                                          │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                                                    │
│ ──> Bước 1 & 2: Tự động trích xuất thực thể EMR -> Sinh bản nháp có cấu trúc JSON 6 mục │
│     -> Gắn nhãn [DRAFT_ONLY] -> Trình bác sĩ review và ký số trong 2 phút.               │
│                                                                                          │
│ Đo lường thành công bằng gì (Metrics có số cụ thể)?                                      │
│ 1. Hiệu năng (Efficiency): Giảm thời gian soạn tóm tắt từ 25 min ──> dưới 5 min/ca.      │
│ 2. Độ đầy đủ (Quality): ≥ 95% bản nháp đạt chuẩn 6 mục (chẩn đoán, thủ thuật, đơn thuốc, │
│    dấu hiệu cảnh báo đỏ, lịch tái khám, hotline hỗ trợ 24/7).                            │
│ 3. Độ an toàn (Safety): 0% hallucination nghiêm trọng (tuyệt đối không bịa chẩn đoán mới,│
│    không tự ý đổi tên thuốc hay liều dùng so với EMR gốc).                               │
│                                                                                          │
│ Quick Architecture: [ ] No AI   [ ] Rule-based   [x] LLM Feature   [ ] Agentic Loop      │
│ Lý do: Cần xử lý ngôn ngữ tự nhiên tiếng Việt y học đa dạng; quy trình chuẩn hóa cố định │
│ yêu cầu kiểm soát chặt chẽ; bắt buộc Human-in-the-loop (Bác sĩ ký); cấm Agent tự trị.    │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### QUICK PROBLEM CARD #2 — Vinmec Triage Chatbot (Phân loại lịch khám)

```text
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                                                    │
│                                                                                          │
│ Bài toán (1 câu): Trợ lý AI hỏi bệnh sơ bộ qua ngôn ngữ tự nhiên và phân luồng chính xác │
│ chuyên khoa khám bệnh ban đầu cho bệnh nhân đặt lịch trực tuyến.                         │
│ Công ty thành viên: [x] Vinmec   [ ] VinFast   [ ] Xanh SM   [ ] Vinhomes                │
│                                                                                          │
│ Ai đang đau?                                                                             │
│ - Nhân viên tiếp đón / Call center: Thiếu kiến thức chuyên khoa sâu, dễ đoán nhầm.        │
│ - Bệnh nhân: Bị chuyển qua lại giữa các phòng khám, mất thời gian chờ đợi khám lại.      │
│ - Bác sĩ chuyên khoa: Tiếp nhận bệnh nhân không đúng mặt bệnh chuyên sâu của mình.       │
│                                                                                          │
│ Workflow thủ công hiện tại (4 bước):                                                     │
│   1. Bệnh nhân gọi hotline hoặc nhắn tin mô tả triệu chứng (⏱ 3 min)                     │
│   ──> 2. Nhân viên tiếp đón phỏng đoán chuyên khoa và tra lịch trống (⏱ 4 min) 🔴        │
│   ──> 3. Xếp lịch và gửi xác nhận cho bệnh nhân (⏱ 2 min)                                │
│   ──> 4. Bác sĩ khám thực tế; nếu sai chuyên khoa thì viết phiếu chuyển khoa (⏱ 30 min) 🔄│
│                                                                                          │
│ Bước tốn nhất: Bước 2 & Bước 4 (chuyển khoa mất thêm 30–45 phút/ca; tỷ lệ sai ~25%).     │
│ AI hỗ trợ ở đâu: Phân tích mô tả triệu chứng, hỏi thêm 2-3 câu làm rõ (clarifying        │
│ questions), tính xác suất chuyên khoa phù hợp kèm độ tin cậy (confidence score).         │
│ Metric: Giảm tỷ lệ chuyển khoa từ 25% ──> dưới 8%; thời gian đặt lịch giảm từ 9 ──> 2 min│
│ Architecture: [x] LLM Feature kết hợp Rule-based Medical Tree.                           │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### QUICK PROBLEM CARD #3 — Vinhomes Resident Ticket Router (Điều hướng khiếu nại)

```text
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                                                    │
│                                                                                          │
│ Bài toán (1 câu): Tự động đọc hiểu, phân loại và điều phối các phản ánh/khiếu nại của cư │
│ dân trên App Vinhomes Resident về đúng bộ phận kỹ thuật của từng phân khu/tòa nhà.       │
│ Công ty thành viên: [ ] Vinmec   [ ] VinFast   [ ] Xanh SM   [x] Vinhomes                │
│                                                                                          │
│ Ai đang đau?                                                                             │
│ - Nhân sự CSKH trung tâm: Ngập trong hàng trăm tin nhắn phản ánh hỗn tạp mỗi ngày.       │
│ - Cư dân: Chờ đợi lâu cho các sự cố sinh hoạt khẩn cấp (mất nước, rò rỉ ống dẫn...).    │
│ - Đội kỹ thuật tòa nhà: Nhận thông tin chậm trễ hoặc thông tin bị tam sao thất bản.     │
│                                                                                          │
│ Workflow thủ công hiện tại (4 bước):                                                     │
│   1. Cư dân gửi phản ánh bằng văn bản/hình ảnh qua ứng dụng (⏱ 2 min)                   │
│   ──> 2. Nhân viên trực tổng đài đọc nội dung, xác định tòa nhà và loại lỗi (⏱ 5 min) 🔴  │
│   ──> 3. Tạo ticket trên hệ thống ERP nội bộ và chuyển tiếp BQL tòa nhà (⏱ 3 min) 🔄     │
│   ──> 4. Kỹ thuật viên tòa nhà tiếp nhận và phản hồi hẹn giờ sửa chữa (⏱ 2–8 tiếng)     │
│                                                                                          │
│ Bước tốn nhất: Bước 2 & 3 (trung bình 8–10 phút/ticket do lượng ticket dồn ứ).           │
│ AI hỗ trợ ở đâu: Trích xuất loại sự cố, mức độ khẩn cấp và mã căn hộ; tự động gán       │
│ ticket vào hàng đợi kỹ thuật của tòa nhà tương ứng kèm bản tóm tắt nguyên nhân.          │
│ Metric: 90% ticket được phân loại trong dưới 5 giây; thời gian phản hồi cư dân < 15 min. │
│ Architecture: [x] LLM Classifier kết hợp Webhook/API ERP nội bộ.                         │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🗳️ Quyết định Lựa chọn của Nhóm & Ma trận Đánh đổi (Trade-off Analysis)

Hội đồng kỹ sư nhóm **A3Team** đã tiến hành thảo luận, chấm điểm đối sánh giữa 3 thẻ bài toán:

| Tiêu chí đánh giá (Trọng số) | Card #1: Vinmec Discharge | Card #2: Vinmec Triage | Card #3: Vinhomes Ticket |
|---|:---:|:---:|:---:|
| **Tác động trực tiếp đến người dùng (30%)** | **9.5/10** (Tiết kiệm 50h bác sĩ/ngày, giảm 2-4h chờ đợi) | 8.0/10 (Giảm chờ đợi chuyển khoa) | 7.5/10 (Tăng tốc độ hỗ trợ cư dân) |
| **Khả năng thiết lập ranh giới an toàn (30%)** | **9.5/10** (Đóng khung rõ rệt: chỉ draft, cấm tự ký, HITL tuyệt đối) | 6.5/10 (Rủi ro hỏi bệnh sai sót, trách nhiệm pháp lý mơ hồ) | 8.5/10 (Ranh giới nghiệp vụ văn phòng rõ ràng) |
| **Tính sẵn sàng của dữ liệu (20%)** | **9.0/10** (EMR có sẵn, cấu trúc trường dữ liệu chuẩn quốc tế) | 6.0/10 (Thiếu dữ liệu hội thoại tiếng Việt chuẩn hóa y tế) | 8.0/10 (Có log ticket cũ trên hệ thống ERP) |
| **Độ phức tạp kỹ thuật phù hợp phạm vi Lab (20%)** | **9.0/10** (1 call LLM Feature + Structured Output + Assertions) | 7.0/10 (Cần Multi-turn stateful chatbot) | 8.0/10 (Phân loại văn bản chuẩn) |
| **TỔNG ĐIỂM CÓ TRỌNG SỐ (100%)** | **9.30 / 10** 🏆 | **7.05 / 10** | **7.95 / 10** |

### 📌 Lý giải chi tiết quyết định:
1. **LÝ DO CHỌN CARD #1 (Vinmec Discharge Summary):**
   * **Giá trị kinh tế & nhân văn vượt trội:** Giải phóng trực tiếp đội ngũ y bác sĩ khỏi gánh nặng hành chính, cho phép họ dành thời gian quý báu chăm sóc bệnh nhân mới.
   * **Ranh giới an toàn (Operational Boundary) hoàn hảo cho thiết kế AI:** Bác sĩ điều trị là người chịu trách nhiệm pháp lý cuối cùng thông qua chữ ký số $\rightarrow$ Rủi ro khi mô hình AI phát sinh lỗi được kiểm soát 100% qua cơ chế Human-in-the-loop (HITL) và Fallback soạn tay truyền thống.
   * **Định lượng rõ ràng:** Thời gian làm bài toán đo được chính xác theo phút ($25 \text{ min} \rightarrow < 5 \text{ min}$), tỷ lệ hallucination đo bằng $0\%$.
2. **LÝ DO LOẠI TRỪ CARD #2 & CARD #3:**
   * *Card #2 (Vinmec Triage):* Dễ dẫn tới rủi ro chẩn đoán sai triệu chứng cấp cứu khẩn cấp (như nhồi máu cơ tim thầm lặng, bóc tách động mạch chủ). Cần đánh giá lâm sàng lâm thời phức tạp và quy trình pháp lý ngặt nghèo hơn trước khi cho phép AI tương tác trực tiếp với người bệnh.
   * *Card #3 (Vinhomes Ticket):* Tác vụ có giá trị cao nhưng thuần túy là bài toán văn phòng (Back-office automation), ít tính bức thiết và không thể hiện rõ nét năng lực quản trị rủi ro đa tầng như môi trường y tế của Vinmec.
