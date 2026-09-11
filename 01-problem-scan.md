# 01 — Problem Scan & Quick Cards

> **Người thực hiện:** pbaodev — AI Product Engineer, Vin Smart Future
> **Phạm vi quét:** 2 công ty thành viên — **Xanh SM** (vận hành gọi xe điện) và **Vincons** (tổng thầu xây dựng thuộc Vinhomes).
>
> **Ký hiệu số liệu:** con số có nguồn được dẫn ở cuối file; con số đánh dấu **(ƯT)** là *ước tính cá nhân*, cần xác lập baseline thực tế trước khi dùng cho quyết định.

---

## 🏛️ Bối cảnh

| | Xanh SM | Vincons |
|---|---|---|
| **Quy mô** | 133,31 triệu chuyến trong Q4/2025, 51,5% thị phần gọi xe VN, hơn 50.000 tài xế | ~130.000 nhân sự (~120.000 công nhân), doanh thu 2025 hơn 25.000 tỷ đồng |
| **Điểm nóng vận hành** | Tổng đài CSKH 1555 (từ 14/04/2026) xử lý đồ thất lạc, khiếu nại tài xế, tranh chấp cước, hóa đơn VAT, sự cố | Đang tuyển 100.000 công nhân; hơn 95% nhân sự mới chưa từng làm xây dựng |

Quy mô lớn khiến mọi tác vụ thủ công dù chỉ vài phút/lượt đều nhân lên thành hàng nghìn — hàng chục nghìn giờ công.

---

# 🔍 Phase 1 — SCAN

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **Xanh SM** | Lặp lại | **Tìm đồ thất lạc:** khách gọi 1555 mô tả mơ hồ (*"sáng nay đi từ Cầu Giấy, để quên túi đen"*), CSKH phải tự tra lịch sử chuyến, gọi tài xế xác nhận, hẹn trả đồ. ~15–20 phút/vụ (ƯT). |
| 2 | **Xanh SM** | Tốn thời gian | **Tranh chấp cước phí:** CSKH đối chiếu thủ công lộ trình GPS với giá báo trước để quyết định hoàn tiền. ~10 phút/vụ (ƯT). |
| 3 | **Vincons** | Lặp lại | **Tuyển dụng & phân loại công nhân:** HR sàng lọc hàng loạt hồ sơ tự khai không chuẩn (*"làm hồ ở quê 2 năm, biết hàn chút"*) rồi xếp thủ công vào nhóm nghề + lớp đào tạo. Mục tiêu 100.000 người × ~15 phút/hồ sơ (ƯT) ≈ **25.000 giờ công HR**. |
| 4 | **Vincons** | Tốn thời gian | **Báo cáo ngày công trường:** kỹ sư giám sát gom tin nhắn Zalo, ảnh, ghi chú của các tổ đội để viết nhật ký thi công theo mẫu. ~45–60 phút/ngày/kỹ sư (ƯT). |
| 5 | **Vincons** | Pain từ người khác | **Điều phối phiếu bảo hành sau bàn giao:** cư dân Vinhomes báo lỗi (thấm, nứt, điện nước), phiếu phải được phân loại hạng mục và chuyển đúng tổ đội/thầu phụ; cư dân phàn nàn thời gian phản hồi chậm, phối hợp thầu phụ chưa tốt. |

**Độ phủ lens:** Lặp lại (#1, #3) · Tốn thời gian (#2, #4) · Pain từ người khác (#5) → **3 lenses**.

---

# 🃏 Phase 2 — QUICK-ASSESS

**Top 3 được chọn:** #1 (Xanh SM — Đồ thất lạc), #3 (Vincons — Tuyển dụng & phân loại công nhân), #4 (Vincons — Báo cáo ngày công trường).

**Lý do không chọn:**
* **#2 Tranh chấp cước phí:** bản chất là so sánh số (quãng đường GPS thực tế vs giá báo trước theo bảng giá) → **Rule-based giải tốt hơn**, rẻ hơn và dễ kiểm toán hơn LLM. Không cần AI.
* **#5 Phiếu bảo hành:** giá trị cao nhưng giao thoa với hệ thống ban quản lý tòa nhà của Vinhomes và hợp đồng thầu phụ — ranh giới trách nhiệm phức tạp, nên để sau khi có dữ liệu phiếu lịch sử.

---

## Card #1 — Xanh SM: Tìm đồ thất lạc qua tổng đài 1555

| Trường | Nội dung |
|---|---|
| **Bài toán (1 câu)** | Khách để quên đồ trên xe mô tả chuyến đi rất mơ hồ, CSKH mất nhiều thời gian dò đúng chuyến và liên hệ tài xế. |
| **Công ty thành viên** | [x] Xanh SM |
| **Ai đang đau (Actor)?** | Nhân viên CSKH tổng đài 1555 (quá tải thao tác tra cứu); khách hàng (chờ lâu, lo mất đồ); tài xế (bị gọi nhiều lần). |
| **Workflow thủ công hiện tại** | 1. Khách gọi 1555 mô tả đồ + chuyến đi theo trí nhớ ──> 2. CSKH tra lịch sử chuyến của tài khoản, dò từng chuyến khớp thời gian/địa điểm ──> 3. Gọi/nhắn tài xế xác nhận có đồ ──> 4. Hẹn phương án trả đồ (tài xế mang đến / gửi văn phòng) ──> 5. Gọi lại khách, đóng ticket |
| **Bước tốn thời gian/lỗi nhất** | Bước 2–3 (⏱ ~10–15 phút/vụ (ƯT)) — khách nhớ sai giờ, đi nhiều chuyến trong ngày, tài xế không nghe máy. |
| **AI nhảy vào ở bước nào?** | **Bước 2:** LLM trích xuất thông tin có cấu trúc từ lời khách (khung giờ, điểm đi/đến, loại xe, mô tả đồ) → hệ thống lọc lịch sử chuyến → xếp hạng top-3 chuyến khớp nhất. **Bước 3:** LLM soạn **nháp** tin nhắn gửi tài xế, CSKH duyệt trước khi gửi. |
| **Metric thành công** | Giảm thời gian xử lý từ ~15–20 phút → **dưới 5 phút/vụ**; **≥ 80%** vụ có chuyến đúng nằm trong top-3 gợi ý (ƯT, cần baseline). |
| **Quick Architecture** | [ ] No AI  [ ] Rule  [x] **LLM Feature**  [ ] Agent |

**Ranh giới sơ bộ:** AI không tiết lộ số điện thoại/thông tin cá nhân tài xế cho khách; không hứa chắc tìm thấy hay cam kết bồi thường; chỉ tra chuyến thuộc tài khoản đã xác thực của người gọi.

---

## Card #3 — Vincons: Tuyển dụng & phân loại công nhân

| Trường | Nội dung |
|---|---|
| **Bài toán (1 câu)** | Vincons cần tuyển 100.000 công nhân phần lớn chưa có kinh nghiệm; HR sàng lọc và xếp nhóm nghề thủ công từ hồ sơ tự khai không chuẩn, chậm và dễ xếp sai. |
| **Công ty thành viên** | [x] Khác: **Vincons** (thuộc Vinhomes) |
| **Ai đang đau (Actor)?** | Chuyên viên tuyển dụng tại các điểm tuyển địa phương (khối lượng hồ sơ khổng lồ); chỉ huy trưởng công trường (nhận người sai tổ, phải điều chuyển lại); ứng viên (chờ phản hồi lâu). |
| **Workflow thủ công hiện tại** | 1. Ứng viên nộp thông tin qua nhiều kênh (hotline, Zalo, form, tại điểm tuyển) ──> 2. HR gọi điện sàng lọc: tuổi, sức khỏe, kinh nghiệm, khu vực muốn làm ──> 3. HR tự xếp nhóm nghề (nề, cốp pha, cốt thép, điện nước, phụ hồ…) + trình độ ──> 4. Xếp lớp đào tạo và phân bổ công trường theo nhu cầu ──> 5. Công trường nhận người, phát hiện xếp sai thì trả về/điều chỉnh |
| **Bước tốn thời gian/lỗi nhất** | Bước 2–3 (⏱ ~15 phút/hồ sơ (ƯT)) — thông tin tự khai bằng lời nói/tin nhắn, dùng từ địa phương, không theo mẫu; mỗi HR phân loại theo cảm tính riêng. |
| **AI nhảy vào ở bước nào?** | **Bước 3:** LLM chuẩn hóa hồ sơ tự khai thành trường có cấu trúc (nghề từng làm, số năm, chứng chỉ, khu vực) và **gợi ý** nhóm nghề + trình độ + lớp đào tạo kèm độ tin cậy. Điều kiện cứng (đủ 18 tuổi, giấy tờ hợp lệ) kiểm tra bằng **Rule**. HR vẫn thực hiện bước 2 và ra quyết định cuối. |
| **Metric thành công** | Giảm thời gian sàng lọc + phân loại từ ~15 phút → **dưới 5 phút/hồ sơ** (tiết kiệm ~16.700 giờ công HR cho 100.000 hồ sơ); tỷ lệ công nhân bị công trường trả về vì xếp sai tổ giảm từ ~15% (ƯT) → **dưới 5%**. |
| **Quick Architecture** | [ ] No AI  [x] **Rule** (điều kiện cứng)  [x] **LLM Feature** (chuẩn hóa + gợi ý)  [ ] Agent |

**Ranh giới sơ bộ:** AI **không được tự loại** ứng viên — mọi quyết định tuyển/loại do HR; không dùng giới tính, dân tộc, quê quán làm tiêu chí gợi ý; không suy diễn tình trạng sức khỏe; dữ liệu cá nhân (CCCD, số điện thoại) phải được bảo vệ theo quy định về dữ liệu cá nhân.

**Giả định cần kiểm chứng:** workflow 5 bước, thời gian 15 phút/hồ sơ và tỷ lệ xếp sai 15% là suy luận từ thông tin tuyển dụng công khai của Vincons — cần xác nhận với bộ phận tuyển dụng thực tế.

---

## Card #4 — Vincons: Báo cáo ngày / nhật ký thi công

| Trường | Nội dung |
|---|---|
| **Bài toán (1 câu)** | Kỹ sư giám sát mất gần một giờ mỗi ngày gom thông tin rời rạc từ các tổ đội để viết nhật ký thi công theo mẫu. |
| **Công ty thành viên** | [x] Khác: **Vincons** (thuộc Vinhomes) |
| **Ai đang đau (Actor)?** | Kỹ sư giám sát / cán bộ kỹ thuật hiện trường (việc giấy tờ lấn thời gian giám sát thực địa); chỉ huy trưởng (nhận báo cáo muộn, thiếu mục). |
| **Workflow thủ công hiện tại** | 1. Tổ trưởng các tổ đội gửi ảnh, tin nhắn, ghi âm qua Zalo trong ngày ──> 2. Cuối ngày kỹ sư lọc tin, đối chiếu khối lượng, nhân lực, vật tư ──> 3. Viết nhật ký thi công theo mẫu (thời tiết, nhân lực, khối lượng, sự cố, an toàn) ──> 4. Gửi chỉ huy trưởng duyệt ──> 5. Lưu hồ sơ phục vụ nghiệm thu |
| **Bước tốn thời gian/lỗi nhất** | Bước 2–3 (⏱ ~45–60 phút/ngày/kỹ sư (ƯT)) — thông tin phân tán trong nhiều nhóm chat, dễ sót mục, số liệu không nhất quán. |
| **AI nhảy vào ở bước nào?** | **Bước 2–3:** LLM tổng hợp tin nhắn + chú thích ảnh trong ngày → soạn **nháp** nhật ký đúng mẫu, **đánh dấu rõ các mục thiếu dữ liệu** để kỹ sư bổ sung. Kỹ sư chỉnh sửa và ký. |
| **Metric thành công** | Giảm thời gian lập báo cáo từ ~45–60 phút → **dưới 15 phút/ngày**; **100%** báo cáo đủ các mục bắt buộc trước khi gửi duyệt. |
| **Quick Architecture** | [ ] No AI  [ ] Rule  [x] **LLM Feature**  [ ] Agent |

**Ranh giới sơ bộ:** AI **không được tự điền hay bịa** số khối lượng, nhân lực, vật tư khi không có trong dữ liệu gốc (để trống + cảnh báo); không nộp thẳng — nhật ký thi công là hồ sơ pháp lý phục vụ nghiệm thu nên bắt buộc kỹ sư ký; sự cố an toàn phải được báo ngay cho chỉ huy trưởng, không chờ báo cáo cuối ngày.

---

## 📚 Nguồn số liệu

* Xanh SM 51,5% thị phần, 133,31 triệu chuyến Q4/2025 — [VOV](https://vov.vn/doanh-nghiep/them-mot-bao-cao-cho-thay-xanh-sm-vuot-50-thi-phan-goi-xe-cong-nghe-viet-nam-post1267413.vov)
* Tổng đài 1555 và các trường hợp cần liên hệ — [Green SM](https://www.greensm.com/vn-vi/news/so-dien-thoai-tong-dai-taxi-xanh-sm-tren-toan-quoc-hotline-24-7), [Viettel Store](https://viettelstore.vn/tin-tuc/cach-lien-he-tong-dai-xanh-sm-cskh-nhanh-nhat-nam-2026)
* Quy mô nhân sự, doanh thu, 95% nhân sự mới chưa từng làm xây dựng — [Znews](https://znews.vn/vincons-pha-hoi-nong-vao-cac-tong-thau-xay-dung-post1681271.html)
* Vincons tuyển 100.000 công nhân, đào tạo tại công trường, lộ trình thu nhập — [VnExpress](https://vnexpress.net/vincons-tuyen-100-000-cong-nhan-xay-dung-toan-quoc-4992000.html), [Vingroup](https://vingroup.net/tin-tuc-su-kien/bai-viet/3737/vincons-tuyen-dung-100000-cong-nhan-xay-dung-tren-toan-quoc)
* Vincons thuộc Vinhomes — [Báo Pháp Luật](https://doanhnhan.baophapluat.vn/vingroup-vic-chuyen-toan-bo-co-phan-tai-vincons-cho-vinhomes-48239.html)
* Hồ sơ giám sát, nhật ký thi công, nghiệm thu — [Tư vấn xây dựng Trường Lũy](https://tuvanxaydungtruongluy.com/ho-so-giam-sat-thi-cong-xay-dung-cong-trinh/)
* Phản ánh của cư dân về xử lý sự cố, phối hợp thầu phụ — [Vinhomes-Land](https://vinhomes-land.vn/cam-nhan-cua-cu-dan-ve-chat-luong-dich-vu-quan-ly-van-hanh-vinhomes/)
