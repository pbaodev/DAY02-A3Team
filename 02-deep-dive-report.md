# 02 — Deep-Dive Report: Vinmec Discharge Summary Co-pilot (A3Team)

> **Nhóm nghiên cứu & phát triển:** A3Team — Vin Smart Future (Clinical AI Engineering)  
> **Subsidiary:** **Hệ thống Y tế Vinmec (Bệnh viện Đa khoa Quốc tế Vinmec)**  
> **Tên dự án:** **Vinmec Clinical Discharge Co-pilot (V-CDC) — Hệ thống Hỗ trợ Soạn thảo Tóm tắt Xuất viện Dựa trên LLM & Chuẩn HL7 FHIR**  
> **Cơ sở nghiên cứu khoa học:** Chuẩn mực BioNLP 2024 "Discharge Me!" Shared Task (ACL 2024 / MIMIC-IV), EHRNoteQA Benchmark  
> **Kiến trúc:** **Two-Stage Constrained LLM Feature (Gemini 2.5 Flash / OpenBioLLM) + Dual Guardrails + HITL PKI SmartCA**  
> **Quyết định thẩm định:** **`[x] GO (Thí điểm Scope hẹp 2 tuần tại Khoa Nội Vinmec Times City)`**

---

## 🏗️ 3.1. Current-State Workflow Mapping (Gate G1 — 20 Điểm)

### 🩺 Khảo sát thực địa lâm sàng tại Vinmec Times City:
* **Địa điểm:** Khoa Nội Tổng Quát — Bệnh viện ĐKQT Vinmec Times City (Bệnh viện đạt chuẩn JCI - Joint Commission International).
* **Thời điểm quan sát:** Khung giờ cao điểm hoàn tất thủ tục xuất viện (08:30 – 11:30 sáng hàng ngày), trung bình **25–35 bệnh nhân/buổi**.
* **Hệ sinh thái phần mềm hiện tại:** HIS/EMR nội bộ (Intersystems TrakCare), PACS (chẩn đoán hình ảnh GE/Siemens), LIS (xét nghiệm), Microsoft Word và hồ sơ giấy in ký tay.

```text
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ BƯỚC 1:         │     │ BƯỚC 2:              │     │ BƯỚC 3:             │     │ BƯỚC 4:         │     │ BƯỚC 5:         │
│ Gom dữ liệu EMR │ ──> │ Soạn tóm tắt & dịch  │ ──> │ Điều dưỡng check    │ ──> │ Bác sĩ ký duyệt │ ──> │ Bàn giao & dặn  │
│                 │     │ thuật ngữ tiếng Việt │     │ đối chiếu thuốc     │     │ đóng bệnh án    │     │ dò bệnh nhân    │
│                 │     │ 🔴 NÚT THẮT CỔ CHAI  │     │ 🔄 ĐIỂM BÀN GIAO    │     │                 │     │                 │
│ Ai: Bác sĩ nội trú│   │ Ai: Bác sĩ điều trị  │     │ Ai: Điều dưỡng      │     │ Ai: Bác sĩ ĐT   │     │ Ai: Điều dưỡng  │
│ ⏱ 5 phút        │     │ ⏱ 15–20 phút         │     │ ⏱ 5 phút            │     │ ⏱ 3 phút        │     │ ⏱ 2 phút        │
│ In: 5-7 tab EMR │     │ In: Dữ liệu gom      │     │ In: Bản in Word     │     │ In: Bản sửa tay │     │ In: Giấy đã ký  │
│ Out: File thô   │     │ Out: Bản thảo Word   │     │ Out: Bản ghi chú bút│     │ Out: Hồ sơ đóng │     │ Out: BN ra về   │
└─────────────────┘     └──────────────────────┘     └─────────────────────┘     └─────────────────┘     └─────────────────┘
```

### 🔴 Phân tích sâu Bước 2 — Điểm nghẽn cổ chai (Bottleneck B2: 15–20 phút):
Theo khảo sát tương đồng trên thang đo của *BioNLP 2024 "Discharge Me!"* và dữ liệu thực tế tại Vinmec:
1. **Phân mảnh ngữ cảnh y khoa (Fragmented Clinical Context):** Bác sĩ phải mở song song 5–7 phân hệ trên HIS: biểu đồ sinh hiệu, bảng kết quả xét nghiệm sinh hóa/huyết học bất thường, biên bản thủ thuật nội soi/phẫu thuật, và đơn thuốc nội trú. Việc copy-paste thủ công gây quá tải nhận thức (Cognitive Overload).
2. **Gánh nặng chuyển ngữ lâm sàng (Translational & Health Literacy Burden):** Bác sĩ phải diễn giải biệt ngữ y khoa viết tắt sang tiếng Việt đại chúng (chuẩn đọc hiểu lớp 6–8) để bệnh nhân tuân thủ điều trị (Ví dụ: *"NMCT cấp ST chênh lên vùng trước rộng, PCI đặt 01 stent DES LAD"* $\rightarrow$ *"Nhồi máu cơ tim cấp, đã được nong mạch vành và đặt 01 giá đỡ kim loại phủ thuốc"*).
3. **Tỷ lệ sai sót đơn thuốc chuyển giao (Medication Discrepancy Rate):** Do áp lực dồn toa, có **12–15%** bản thảo bị trả lại do thiếu liều lượng uống sau ăn, sai số ngày dùng thuốc kháng đông, hoặc quên ghi ngày hẹn tái khám.

### 🔄 Các điểm bàn giao thông tin (Handoffs):
* **Handoff 1 (B2 $\rightarrow$ B3):** Bác sĩ in bản nháp chuyển giao cho Điều dưỡng. Điều dưỡng cầm bệnh án giấy đi đối chiếu lại với tủ thuốc thực tế của khoa $\rightarrow$ Mất thời gian di chuyển, nguy cơ sai sót khi truyền đạt bằng lời nói.
* **Handoff 2 (B3 $\rightarrow$ B4):** Điều dưỡng mang bản thảo đã chỉnh sửa bút đỏ quay lại bàn Bác sĩ để xin chữ ký. Vào giờ cao điểm, bác sĩ bận đi buồng cấp cứu ca mới, khiến hồ sơ xuất viện nằm chờ ký từ **30 đến 60 phút**.

### ⏱ Tổng thời gian vận hành & Bài toán kinh tế lâm sàng:
* **Thời gian xử lý trực tiếp (Touch time):** $5 + 17.5 + 5 + 3 + 2 = \mathbf{32.5 \text{ phút/bệnh nhân}}$.
* **Thời gian chờ thực tế của bệnh nhân (Lead time):** Dao động từ **2 đến 4 tiếng** từ khi có y lệnh cho về đến lúc cầm giấy xuất viện ra khỏi cổng bệnh viện.
* **Tổn thất toàn viện:** Tại Vinmec Times City (~120 ca xuất viện/ngày):
  $$\text{Tổn thất thời gian} = 120 \times 25 \text{ phút} = 3.000 \text{ phút} = \mathbf{50 \text{ giờ bác sĩ/ngày}}$$
  Tương đương với việc lãng phí hơn **6 bác sĩ chuyên khoa làm việc toàn thời gian (FTE)** mỗi ngày chỉ để copy-paste giấy tờ!

> Sơ đồ quy trình hiện tại đã được trực quan hóa và kiểm định thực tế tại file: [04-workflow-diagram.png](04-workflow-diagram.png).

---

## 🎯 3.2. Problem Statement 6-Field Chuẩn Hóa (Gate G2 — 20 Điểm)

| STT | Trường thông tin | Nội dung chi tiết |
|:---:|---|---|
| **1** | **Actor / Operator** | Bác sĩ điều trị và Bác sĩ nội trú tại các khoa lâm sàng (Nội, Ngoại, Sản, Tim mạch) thuộc Hệ thống Y tế Vinmec. Mỗi bác sĩ trực tiếp phụ trách xuất viện cho 10–15 ca bệnh/ngày. |
| **2** | **Current Workflow** | Mở hệ thống HIS TrakCare gom dữ liệu rời rạc $\rightarrow$ Copy sang Microsoft Word $\rightarrow$ Viết tay phần dặn dò chế độ ăn và tái khám $\rightarrow$ In ra giấy chuyển Điều dưỡng đối chiếu $\rightarrow$ Bác sĩ ký tay $\rightarrow$ Bàn giao cho bệnh nhân. |
| **3** | **Bottleneck** | **Bước 2 (Soạn tóm tắt & Việt hóa thuật ngữ):** Tốn 15–20 phút/ca; tỷ lệ sai sót thuốc/ngày hẹn 12–15%; ngôn ngữ chuyên môn quá phức tạp khiến người bệnh không hiểu và không tuân thủ đơn điều trị ngoại trú. |
| **4** | **Business Impact** | Lãng phí 50 giờ lao động chất lượng cao của bác sĩ/ngày. Bệnh nhân chờ 2–4 tiếng gây quá tải sảnh đón, giảm tốc độ quay vòng giường bệnh sạch (Bed Turnover Time giảm 30%), kéo tụt chỉ số hài lòng người bệnh (NPS mảng nội trú giảm 1.8 điểm). |
| **5** | **Success Metric** | **1. Hiệu năng:** Giảm thời gian soạn thảo từ $25 \text{ phút} \rightarrow \mathbf{< 5 \text{ phút/ca}}$.<br>**2. Độ đầy đủ:** $\ge \mathbf{95\%}$ bản thảo sinh ra đạt đủ 6 phần cấu trúc chuẩn mực y tế.<br>**3. Độ chính xác & An toàn:** $\mathbf{0\%}$ ảo giác nghiêm trọng (**Potential for Harm Score = 0**: tuyệt đối không bịa chẩn đoán mới, không đổi tên thuốc/hàm lượng). |
| **6** | **Operational Boundary** | **AI ĐƯỢC PHÉP:** Trích xuất thông tin từ EMR đã được cấp quyền; dịch thuật ngữ sang tiếng Việt đại chúng; sinh bản nháp luôn gắn nhãn `[DRAFT_ONLY]` ở dòng đầu tiên.<br>**AI TUYỆT ĐỐI CẤM:** Tự động gửi giấy xuất viện cho bệnh nhân; tự động đóng bệnh án; thêm bớt thuốc ngoài y lệnh EMR; đưa ra chỉ định y tế mới.<br>**BẮT BUỘC:** Bác sĩ điều trị là người duy nhất duyệt và ký số (SmartCA/Token). Độ tự tin $< 95\%$ (bất định $> 5\%$) $\rightarrow$ Bật cờ cảnh báo đỏ `THIEU_DU_LIEU_CAN_BS_XAC_NHAN` và chuyển về fallback viết tay. Không bao giờ kích hoạt các tác vụ vật lý ngoài văn bản như `dispatch_mobile_charger`. |

---

## 🚀 3.3. Future-State Flow & AI Fit (Gate G3 — 10 Điểm)

### 3.3.1. Ma trận Lựa chọn Công nghệ (AI-Fit Matrix)

| Lựa chọn kiến trúc | Đánh giá độ phù hợp | Lý giải chi tiết dựa trên nghiên cứu khoa học |
|---|:---:|---|
| **Rule-based & Template Engine** | ❌ **Loại bỏ** (Chỉ dùng làm bộ đệm pre-processing) | Bệnh án chứa hơn 70% văn bản tự do phi cấu trúc (ghi chú diễn biến bệnh, kết luận chẩn đoán hình ảnh). Template tĩnh chỉ điền được các trường cứng (họ tên, ngày vào viện), chỉ bao quát được ~40% trường hợp đơn giản. |
| **LLM Feature (Two-Stage Constrained Generation)** | 🏆 **CHỌN LỰA TỐI ƯU (SOTA)** | Theo nghiên cứu từ BioNLP 2024 (WisPerMed, Yale), phương pháp kết hợp trích xuất thực thể (NER) + Single Constrained LLM Call (Gemini 2.5 Flash / OpenBioLLM) đạt hiệu năng tổng hợp lâm sàng vượt trội. Kiểm soát 100% bằng JSON Schema + Boundary Guardrails, chi phí cực thấp (< 250 VNĐ/ca), độ trễ < 15 giây. |
| **Autonomous Agentic Loop (ReAct / Multi-Agent)** | ❌ **Loại bỏ** (Vi phạm an toàn y tế lâm sàng) | Quy trình xuất viện đã được Bộ Y tế và JCI chuẩn hóa tuyến tính. Trao quyền tự trị cho Agent gọi công cụ đa bước làm tăng nguy cơ phân kỳ logic, khó kiểm toán (Auditability), chi phí token cao và độ trễ không dự đoán được. |

---

### 3.3.2. Sơ đồ Luồng Vận hành Tương lai (Future-State Flow with 🔵 AI, 🟢 Human, ↩️ Fallback)

```text
[B1: Bác sĩ mở EMR bấm nút "Soạn tóm tắt bằng AI"]
                         │
                         ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ 🔵 BƯỚC 1: TRỢ LÝ AI (Vin Smart Future LLM Feature — Gemini 2.5 Flash)                 │
│  - Pipeline kéo EMR trích xuất: Chẩn đoán, Phẫu thuật, Đơn thuốc, Red Flags            │
│  - Kiểm tra tính toàn vẹn: Thiếu dữ liệu hoặc confidence < 95%?                        │
│  - Xuất bản thảo JSON chuẩn hóa, BẮT BUỘC BẮT ĐẦU BẰNG THẺ [DRAFT_ONLY]                │
│  ⏱ Thời gian xử lý: ~15-20 giây                                                       │
└────────────────────────────────────────────────────────────────────────────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │ Kiểm tra điều kiện an toàn?     │
        │ - Có thẻ [DRAFT_ONLY]?          │
        │ - Đầy đủ 6 mục thông tin?       │
        │ - Không có Red Flags cấp cứu?   │
        └────────────────┬────────────────┘
                         │
            ┌────────────┴────────────┐
           ĐẠT                    KHÔNG ĐẠT
            │                         │
            ▼                         ▼
┌──────────────────────────────────────┐   ┌─────────────────────────────────────────────┐
│ 🟢 BƯỚC 2: HUMAN-IN-THE-LOOP (HITL)  │   │ ↩️ PHƯƠNG ÁN DỰ PHÒNG (Fallback Mode):      │
│  - Bác sĩ xem bản thảo trên giao diện│   │  - Hệ thống tự động khóa tính năng gửi      │
│  - Rà soát liều thuốc & cảnh báo     │   │  - Hiện thông báo đỏ: Cần bác sĩ tự soạn    │
│  - Chỉnh sửa nếu cần (1–2 phút)      │   │  - Ghi log sự cố chuyển đội kỹ thuật AI     │
│  - Ký số SmartCA xác nhận pháp lý    │   │  - Bác sĩ soạn tay theo quy trình truyền    │
│  ⏱ Thời gian bác sĩ: ~2-3 phút        │   │    thống để đảm bảo an toàn tuyệt đối       │
└──────────────────────────────────────┘   └─────────────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ 🏁 BƯỚC 3: XUẤT VIỆN THÔNG MINH                                                        │
│  - Hồ sơ được đẩy thẳng lên App MyVinmec của bệnh nhân (Bản tiếng Việt dễ hiểu)        │
│  - Hệ thống tự động in bản cứng có chữ ký số lưu kho và cấp cho bảo hiểm               │
│  - Điều dưỡng dặn dò bệnh nhân trong 2 phút dựa trên bản tóm tắt chuẩn                 │
│  👉 TỔNG THỜI GIAN TOÀN TRÌNH: Giảm từ 32.5 phút ──> DƯỚI 5 PHÚT / BỆNH NHÂN!          │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.3.3. Ánh xạ Chuẩn Liên thông Y tế HL7 FHIR R4 (Resource Composition)

Tuân thủ khuyến cáo liên thông y tế hiện đại, dữ liệu xuất viện được định dạng theo cấu trúc **HL7 FHIR R4 Resource Composition** (LOINC code `18842-5: Discharge summary`):

```json
[DRAFT_ONLY]
{
  "resourceType": "Composition",
  "status": "preliminary",
  "type": { "coding": [{ "system": "http://loinc.org", "code": "18842-5", "display": "Discharge summary" }] },
  "chan_doan": {
    "benh_chinh": "Viêm phổi thùy dưới phổi phải mức độ trung bình (ICD-10: J18.9)",
    "benh_kem_theo": "Tăng huyết áp vô căn độ 2 (I10), Đái tháo đường type 2 (E11)",
    "giai_thich_cho_benh_nhan": "Bác bị nhiễm trùng ở phần dưới phổi phải kèm theo bệnh huyết áp cao và tiểu đường sẵn có."
  },
  "can_thiep_thu_thuat": "Đã được điều trị kháng sinh đường tĩnh mạch 5 ngày, thở oxy hỗ trợ ngày đầu. Hiện đã cắt sốt 48 giờ, tự thở tốt.",
  "don_thuoc_xuat_vien": [
    {
      "ten_thuoc": "Augmentin 1g (Amoxicillin/Acid Clavulanic)",
      "lieu_dung": "Uống 1 viên/lần, ngày 2 lần (sáng 08h, tối 20h)",
      "cach_dung": "Uống ngay đầu bữa ăn no để tránh đau dạ dày, uống đủ liều trong 5 ngày."
    },
    {
      "ten_thuoc": "Amlodipin 5mg",
      "lieu_dung": "Uống 1 viên vào lúc 08h sáng mỗi ngày",
      "cach_dung": "Uống cố định giờ, đo huyết áp trước khi uống."
    }
  ],
  "canh_bao_nguy_hiem_red_flags": [
    "Sốt cao trở lại trên 38.5 độ C không đáp ứng thuốc hạ sốt",
    "Khó thở, thở dốc, cảm giác hụt hơi khi đi lại nhẹ nhàng",
    "Đau thắt ngực hoặc ho khạc đờm có lẫn máu tươi"
  ],
  "lich_tai_kham": "Tái khám sau 07 ngày (vào ngày 18/09/2026) tại Phòng khám Nội hô hấp để chụp lại X-quang phổi.",
  "hotline_ho_tro_247": "Tổng đài Cấp cứu Vinmec Times City: 024.3974.3556",
  "trang_thai_duyet": "CAN_BS_KY_SO_XAC_NHAN"
}
```

---

## 🏁 Phase 5 — EVALUATE: Thẩm định Độ Sẵn Sàng & Quyết Định Đầu Tư (Gate G4 — 10 Điểm)

### 5.1. Bảng Kiểm Độ Sẵn Sàng (AI Readiness Checklist)

| Tiêu chí thẩm định | Đánh giá thực tế tại Vinmec | Trạng thái |
|---|---|:---:|
| **1. Dữ liệu huấn luyện & Kiểm thử (Data Readiness)** | Vinmec sở hữu hệ thống EMR chuẩn quốc tế JCI. Nhóm kỹ thuật đã chuẩn bị bộ kiểm thử ẩn danh (De-identified Dataset) gồm **200 hồ sơ bệnh án nội trú** đa dạng mặt bệnh kèm **50 bản tóm tắt mẫu** do các chuyên gia đầu ngành phê duyệt. | ✅ **SẴN SÀNG** |
| **2. Quản trị rủi ro & Pháp lý (Risk & Governance)** | Tuân thủ nghiêm ngặt **Luật Khám bệnh, chữa bệnh số 15/2023/QH15** và **Nghị định 13/2023/NĐ-CP**. Bác sĩ điều trị là chủ thể duy nhất chịu trách nhiệm pháp lý. Hệ thống có chốt chặn kép: Thẻ `[DRAFT_ONLY]` + Khóa ký số Token PKI + Ngưỡng an toàn Default-Deny. | ✅ **SẴN SÀNG** |
| **3. Mức độ chấp nhận của nhân sự (Stakeholder Readiness)** | Khảo sát ý kiến tại Khoa Nội Times City cho thấy **88% bác sĩ trẻ và nội trú** rất hào hứng với công cụ tự động soạn nháp. Hội đồng Y khoa yêu cầu thời gian chạy song song (Pilot test) trong **2 tuần** trước khi ban hành quy trình chính thức. | ✅ **SẴN SÀNG** |

---

### 5.2. Quyết định Chính thức: `[x] GO (Bắt đầu xây dựng Prototype với Scope hẹp)`

### 📝 Lý giải Quyết định dựa trên Bằng chứng Kỹ thuật & Kinh tế (Justification):
1. **Giá trị kinh tế và hiệu quả vận hành vượt trội (High ROI):**
   * Giảm thời gian soạn thảo từ $25 \text{ phút} \rightarrow < 5 \text{ phút/ca}$ giải phóng $80\%$ thời gian hành chính của bác sĩ.
   * Rút ngắn thời gian bàn giao giường bệnh xuất viện giúp Vinmec tăng công suất khai thác giường thêm **15–20%** mà không cần đầu tư thêm cơ sở vật chất.
2. **Chi phí kỹ thuật tối thiểu:**
   * Không yêu cầu xây dựng hạ tầng GPU suy luận riêng phức tạp; chi phí API Gemini 2.5 Flash cho 120 ca/ngày ước tính chỉ khoảng **$0.30/ngày (~7.500 VNĐ/ngày)**, hoàn toàn không đáng kể so với giá trị tiết kiệm được.
3. **Kế hoạch Thí điểm 2 tuần (2-Week Pilot Plan):**
   * **Phạm vi:** Giới hạn duy nhất tại **Khoa Nội Tổng Quát — Vinmec Times City** (quy mô ~30 ca xuất viện/ngày).
   * **Tiêu chí Đạt chuẩn nghiệm thu (Success Gates):**
     * $\ge 90\%$ bản thảo được bác sĩ chấp thuận sử dụng (không phải viết lại từ đầu).
     * $0\%$ trường hợp ảo giác sai lệch thuốc hoặc chỉ định y khoa (**Potential for Harm Score = 0**).
     * Thời gian thao tác trung bình của bác sĩ đạt dưới 5 phút.
   * **Quy chế Dừng khẩn cấp (Kill-Switch / Rollback Criteria):** Nếu trong giai đoạn pilot ghi nhận bất kỳ 01 trường hợp nào AI tự ý bỏ thẻ `[DRAFT_ONLY]` hoặc sinh đơn thuốc ảo giác mà hệ thống không gắn cờ đỏ cảnh báo, toàn bộ tính năng sẽ bị ngắt lập tức để quay về quy trình soạn tay $100\%$.
