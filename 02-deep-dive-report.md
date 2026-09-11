# 02 — Deep-Dive Report: VinFast Service Ticket Triage (Vin Smart Future)

> **Bài toán được chọn (từ `01-problem-scan.md`, Card #1):**
> Khách hàng VinFast mô tả lỗi xe bằng tiếng Việt đời thường qua app/hotline; Cố vấn dịch vụ (Service Advisor – SA) phải tự "dịch" sang nhóm lỗi kỹ thuật, hỏi thêm khách, kiểm tra phụ tùng rồi mới xác nhận lịch hẹn xưởng.
>
> Các con số vận hành là **ước tính để scoping** (đánh dấu *~*), cần xác nhận với Khối Dịch vụ Hậu mãi VinFast trước khi làm baseline chính thức.

---

# 🏗️ Phase 3 — DEEP-DIVE (Nhóm)

## 3.1. Current-State Workflow Mapping

Quy trình hiện tại khi một khách hàng gửi yêu cầu bảo hành/sửa chữa:

```text
┌──────────────────┐   🔄   ┌──────────────────┐        ┌──────────────────┐
│ Bước 1           │ ─────→ │ Bước 2           │ ─────→ │ Bước 3       🔴  │
│ Khách gửi yêu    │        │ SA đọc ticket &  │        │ SA gọi lại khách │
│ cầu (text/ảnh/   │        │ tra lịch sử xe   │        │ hỏi làm rõ       │
│ video) qua app   │        │ (VIN, km, gói BH)│        │ (kêu khi nào? tốc│
│ hoặc hotline     │        │                  │        │ độ? đèn báo gì?) │
│ Ai: Khách hàng   │        │ Ai: SA           │        │ Ai: SA + Khách   │
│ ⏱ –             │        │ ⏱ ~1.5 phút      │        │ ⏱ ~3-4 phút      │
│ In: Mô tả tự do  │        │ In: Ticket + VIN │        │ In: Ticket       │
│ Out: Ticket CRM  │        │ Out: Hồ sơ xe    │        │ Out: Ghi chú bổ  │
│                  │        │                  │        │      sung        │
└──────────────────┘        └──────────────────┘        └──────────────────┘
                                                                  │
                                                                  ▼
┌──────────────────┐   🔄   ┌──────────────────┐        ┌──────────────────┐
│ Bước 6           │ ←───── │ Bước 5           │ ←───── │ Bước 4       🔴  │
│ KTV nhận Work    │        │ SA xác nhận lịch │        │ SA chọn nhóm lỗi │
│ Order, chuẩn bị  │        │ hẹn với khách +  │        │ (Gầm/Điện/Pin/   │
│ phụ tùng, dụng cụ│        │ tạo Work Order   │        │ HMI/Thân vỏ...), │
│                  │        │ trên DMS         │        │ ước giờ công, tra│
│ Ai: KTV          │        │ Ai: SA           │        │ tồn kho phụ tùng │
│ ⏱ (ngoài scope)  │        │ ⏱ ~1.5 phút      │        │ Ai: SA           │
│ In: Work Order   │        │ In: Nhóm lỗi +   │        │ ⏱ ~3-4 phút      │
│ Out: Xe được sửa │        │     slot xưởng   │        │ In: Ticket + ghi │
│                  │        │ Out: Lịch hẹn +  │        │     chú + kho    │
│                  │        │      Work Order  │        │ Out: Nhóm lỗi +  │
│                  │        │                  │        │  danh sách phụ   │
│                  │        │                  │        │  tùng dự kiến    │
└──────────────────┘        └──────────────────┘        └──────────────────┘

🔴 Bottleneck   🔄 Handoff (Khách → SA ; SA → KTV)
⏱ Tổng cộng = ~10 phút/ticket (phần SA xử lý), chưa tính thời gian khách chờ SA gọi lại.
```

**Chi tiết các điểm nghẽn:**

| Điểm | Vấn đề | Hệ quả |
|---|---|---|
| 🔴 **Bước 3** – Gọi lại khách | Mô tả ban đầu quá mơ hồ (*"xe kêu lạ"*, *"sạc mãi không đầy"*). SA phải gọi, nhiều khi khách không nghe máy → ticket treo vài giờ. | Mất ~3-4 phút/ticket; khách chờ xác nhận lịch trung bình *~2-4 giờ*. |
| 🔴 **Bước 4** – Chọn nhóm lỗi & phụ tùng | Phụ thuộc kinh nghiệm cá nhân của SA; SA mới hay phân loại sai. Không có gợi ý từ dữ liệu lịch sử các ca tương tự. | *~15-20%* ticket phân loại sai nhóm lỗi → KTV không có phụ tùng/dụng cụ đúng → khách phải **hẹn lại lần 2**. |
| 🔄 **Handoff SA → KTV** | Work Order chỉ có nhóm lỗi + vài dòng ghi chú, mất thông tin ngữ cảnh khách kể. | KTV phải hỏi lại khách khi xe đến xưởng, tăng thời gian tiếp nhận. |

*(Sơ đồ trực quan: xem `04-workflow-diagram.png` — xuất từ Mermaid bên dưới.)*

```mermaid
flowchart LR
    A[1. Khách gửi yêu cầu<br/>text/ảnh/video<br/>⏱ –] -->|🔄 handoff| B[2. SA đọc ticket<br/>tra lịch sử xe<br/>⏱ ~1.5 min]
    B --> C[3. 🔴 SA gọi lại khách<br/>hỏi làm rõ<br/>⏱ ~3-4 min]
    C --> D[4. 🔴 SA chọn nhóm lỗi<br/>ước giờ công, tra kho<br/>⏱ ~3-4 min]
    D --> E[5. SA xác nhận lịch hẹn<br/>tạo Work Order<br/>⏱ ~1.5 min]
    E -->|🔄 handoff| F[6. KTV nhận WO<br/>chuẩn bị phụ tùng]
    style C fill:#ffcccc,stroke:#c00
    style D fill:#ffcccc,stroke:#c00
```

---

## 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | **Cố vấn dịch vụ (Service Advisor)** tại các xưởng dịch vụ VinFast (mỗi xưởng lớn *~150-300* ticket/ngày). Stakeholder phụ: Khách hàng (chờ lịch), Kỹ thuật viên (nhận Work Order thiếu thông tin). |
| **2. Current Workflow** | Khách gửi mô tả lỗi qua app VinFast / hotline → SA đọc ticket trên CRM, tra lịch sử xe trên DMS → gọi lại khách hỏi làm rõ → tự chọn nhóm lỗi, ước giờ công, tra kho phụ tùng → xác nhận lịch hẹn và tạo Work Order cho KTV. Công cụ: CRM, DMS, hệ thống kho, điện thoại. **6 bước, ~10 phút/ticket thủ công**, phụ thuộc kinh nghiệm SA. |
| **3. Bottleneck** | **Bước 3 + 4 (~7/10 phút):** phải xử lý ngôn ngữ tự nhiên tiếng Việt đời thường, nhiều phương ngữ, từ tượng thanh (*"cụp cụp"*, *"rè rè"*), mô tả lẫn cảm xúc; rồi ánh xạ sang taxonomy kỹ thuật (nhóm lỗi → hệ thống → phụ tùng). Không có gợi ý tự động, không tận dụng hàng nghìn ticket lịch sử đã có kết luận của KTV. |
| **4. Business Impact** | Với *~200* ticket/ngày/xưởng × *~7 phút* nghẽn = *~23 giờ công SA/ngày/xưởng* bị tiêu vào việc hỏi lại và phân loại. *~15-20%* ticket phân loại sai → khách phải quay lại lần 2 (mỗi lần hẹn lại tốn *~1.5 giờ* công xưởng + slot bị lãng phí + điểm CSAT giảm). Khách chờ xác nhận lịch *~2-4 giờ* thay vì gần tức thì → ảnh hưởng trực tiếp đến trải nghiệm hậu mãi, vốn là điểm cạnh tranh của VinFast so với xe xăng. |
| **5. Success Metric** | 1. **Efficiency:** Thời gian SA xử lý 1 ticket từ *~10 phút* → **dưới 3 phút** (AI làm sẵn bước 3-4, SA chỉ review).<br>2. **Quality:** Tỉ lệ nhóm lỗi đề xuất Top-1 trùng với kết luận cuối của KTV: *~80%* (SA hiện tại) → **≥ 92%**; Top-3 chứa đáp án đúng **≥ 98%** (đo trên *≥ 500* ticket lịch sử đã có ground truth).<br>3. **Outcome:** Tỉ lệ khách phải hẹn lại vì thiếu phụ tùng/KTV không đúng chuyên môn **giảm ≥ 50%** sau 3 tháng.<br>4. **Safety:** **0** trường hợp AI đưa ra lời khuyên vận hành xe/chẩn đoán cuối/báo giá trong tập adversarial test. |
| **6. Operational Boundary** | **AI ĐƯỢC PHÉP:** đọc mô tả khách + lịch sử xe (VIN, km, model, lịch sử sửa chữa) → đề xuất **Top-3 nhóm lỗi kèm độ tin cậy và lý do**; sinh **tối đa 3 câu hỏi làm rõ** để SA/chatbot hỏi khách; gợi ý **danh sách phụ tùng nên chuẩn bị**; gắn cờ `safety_flag` nếu mô tả liên quan phanh/lái/pin/cháy khét.<br>**AI TUYỆT ĐỐI KHÔNG:** (a) kết luận nguyên nhân lỗi cuối cùng hay tuyên bố "xe không có vấn đề"; (b) **báo giá**, hứa thời gian sửa, hứa được bảo hành miễn phí; (c) tư vấn khách *"cứ chạy tiếp/tự sửa"* — đặc biệt với lỗi phanh, lái, pin cao áp, khói/mùi khét; (d) tự tạo Work Order / tự xác nhận lịch hẹn; (e) sử dụng thông tin ngoài input (bịa mã lỗi, bịa chính sách bảo hành); (f) trả lời các yêu cầu ngoài phạm vi (so sánh hãng xe, bàn về giá cổ phiếu, tư vấn pháp lý…).<br>**ĐIỂM DUYỆT (HITL):** SA luôn là người chốt nhóm lỗi và lịch hẹn. Ticket có `safety_flag = true` **bắt buộc** SA gọi khách ngay và ưu tiên slot; ticket có confidence < 0.6 → SA xử lý thủ công như cũ. |

---

## 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix

| Phương án | Có giải quyết được bottleneck? | Nhận xét |
|---|---|---|
| **Rule / State-Machine** | ❌ | Keyword matching (*"phanh"* → nhóm Phanh) vỡ ngay với mô tả kiểu *"đạp xuống thấy mềm mềm, xe trôi thêm một đoạn"*; phương ngữ, sai chính tả, từ tượng thanh vô hạn. Nhưng **rule vẫn dùng** cho phần cứng: tra tồn kho, check gói bảo hành theo VIN/km, chọn slot xưởng. |
| **LLM Feature** ✅ | ✅ | Đúng bản chất bài toán: hiểu ngôn ngữ tự nhiên → phân loại theo taxonomy cố định → JSON có cấu trúc. Một lần gọi, có thể few-shot bằng ticket lịch sử, dễ đo bằng ground truth của KTV, dễ đặt ranh giới. |
| **Agentic Loop** | ⚠️ Không cần | Agent tự gọi khách, tự tra kho, tự đặt lịch nghe hấp dẫn nhưng: quy trình đã có cấu trúc cố định, rủi ro (tự đặt lịch sai, tự hứa với khách) không tương xứng lợi ích. Có thể cân nhắc ở giai đoạn 2 cho chatbot hỏi làm rõ. |

**→ Chọn: `[ ] Rule / State-Machine   [x] LLM Feature   [ ] Agentic Loop`** (LLM cho phần ngôn ngữ, rule cho phần tra cứu và kiểm tra ràng buộc).

### Future-State Flow

```text
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│ Bước 1           │ ─────→ │ Bước 2   🔵 AI   │ ─────→ │ Bước 3   🔵 AI   │
│ Khách gửi yêu    │        │ Auto-pull lịch sử│        │ LLM đọc mô tả +  │
│ cầu qua app /    │        │ xe theo VIN      │        │ lịch sử → JSON:  │
│ hotline          │        │ (rule, tự động)  │        │ top-3 nhóm lỗi,  │
│                  │        │ ⏱ < 5 giây       │        │ confidence, câu  │
│                  │        │                  │        │ hỏi làm rõ, phụ  │
│                  │        │                  │        │ tùng gợi ý,      │
│                  │        │                  │        │ safety_flag      │
│                  │        │                  │        │ ⏱ < 10 giây      │
└──────────────────┘        └──────────────────┘        └────────┬─────────┘
                                                                 │
              ┌──────────────────────────────────────────────────┼─────────────┐
              │ Guardrail (rule, sau LLM):                       ▼             │
              │ • JSON không hợp lệ / thiếu trường  ─────→ ↩️ Fallback A       │
              │ • confidence < 0.6                  ─────→ ↩️ Fallback A       │
              │ • output chứa giá tiền / lời khuyên   ───→ ↩️ Fallback A + log │
              │   vận hành xe (regex + từ khóa cấm)                            │
              │ • safety_flag = true  ──────────────→ 🟢 SA gọi khách NGAY,    │
              │                                        ưu tiên slot xưởng      │
              └────────────────────────────────────────────────────────────────┘
                                                                 │
                                                                 ▼
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│ Bước 6           │ ←───── │ Bước 5   🟢 HITL │ ←───── │ Bước 4   🟢/🔵   │
│ KTV nhận Work    │        │ SA review đề xuất│        │ (Tuỳ chọn) Chatbot│
│ Order có kèm tóm │        │ → chốt nhóm lỗi, │        │ hỏi khách 1-3 câu│
│ tắt ngữ cảnh do  │        │ phụ tùng, slot → │        │ làm rõ do AI sinh │
│ AI sinh          │        │ 1 click tạo WO   │        │ (nếu cần); hoặc  │
│                  │        │ ⏱ ~1-2 phút      │        │ SA hỏi khi gọi   │
└──────────────────┘        └──────────────────┘        └──────────────────┘

🔵 AI Step   🟢 Human Step (HITL)   ↩️ Fallback
↩️ Fallback A: Ticket rơi về hàng đợi thủ công, SA xử lý y như quy trình cũ; log lại để cải thiện prompt/few-shot.
↩️ Fallback B (API Gemini lỗi/timeout > 15s): bỏ qua bước AI, không chặn luồng — SA vẫn nhận ticket như trước.
⏱ Mục tiêu: ~10 phút → < 3 phút/ticket (phần SA).
```

```mermaid
flowchart TD
    A[1. Khách gửi yêu cầu] --> B[2. 🔵 Auto-pull lịch sử xe theo VIN]
    B --> C[3. 🔵 LLM phân loại → JSON<br/>top-3 nhóm lỗi, confidence,<br/>câu hỏi làm rõ, phụ tùng, safety_flag]
    C --> G{Guardrail rule}
    G -->|JSON lỗi / conf < 0.6 /<br/>có giá tiền, lời khuyên cấm| FA[↩️ Fallback: hàng đợi thủ công<br/>SA xử lý như cũ + log]
    G -->|safety_flag = true| S[🟢 SA gọi khách NGAY<br/>ưu tiên slot]
    G -->|OK| D[4. 🟢 Chatbot / SA hỏi khách<br/>1-3 câu làm rõ nếu cần]
    D --> E[5. 🟢 SA review & chốt<br/>1 click tạo Work Order]
    S --> E
    FA --> E
    E --> F[6. KTV nhận WO kèm tóm tắt ngữ cảnh]
    style C fill:#cce5ff,stroke:#0066cc
    style B fill:#cce5ff,stroke:#0066cc
    style E fill:#d4edda,stroke:#28a745
    style S fill:#d4edda,stroke:#28a745
    style D fill:#d4edda,stroke:#28a745
    style FA fill:#fff3cd,stroke:#cc9a06
```

### Cơ chế Human-in-the-loop & Fallback (tóm tắt)

| Tình huống | Ai xử lý | Hành động |
|---|---|---|
| AI trả JSON hợp lệ, confidence ≥ 0.6, không safety flag | 🟢 SA | Review 1-2 phút, sửa nếu cần, chốt. AI **chỉ đề xuất**, không tự tạo WO. |
| `safety_flag = true` (phanh/lái/pin cao áp/khét) | 🟢 SA (ưu tiên) | Gọi khách ngay, khuyến cáo theo kịch bản an toàn chuẩn của VinFast (do người nói, không phải AI). |
| confidence < 0.6 hoặc JSON lỗi | ↩️ Fallback A | Ticket về hàng đợi thủ công; ghi log để bổ sung few-shot. |
| Output vi phạm ranh giới (có giá, có "cứ chạy tiếp") | ↩️ Fallback A + cảnh báo | Chặn không hiển thị cho SA, log incident để chỉnh prompt. |
| Gemini API lỗi / timeout | ↩️ Fallback B | Bỏ qua bước AI, không chặn luồng cũ. |

---

# 💻 Phase 4 — Prompt Prototype & Boundary Test

*(Sẽ cập nhật sau khi chạy `starter-code/prompt_prototype.py`: system prompt, JSON schema, ≥3 adversarial test và kết quả thực tế.)*

---

# 🏁 Phase 5 — EVALUATE (Nhóm)

*(Sẽ hoàn thiện sau Phase 4 — quyết định GO / NOT YET / NO-GO phải dựa trên bằng chứng chạy prototype.)*
