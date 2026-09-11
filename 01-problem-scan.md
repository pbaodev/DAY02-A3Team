# 01 — Problem Scan

## Lab 02 — AI Product Scoping | Vin Smart Future

---

# Phase 1 — SCAN

## Bảng quét cơ hội AI

Sử dụng 4 lenses gồm **Repetitive**, **Time-consuming**, **AI-upgrade** và **Stakeholder Pain** để tìm các bài toán vận hành thực tế tại các công ty thành viên Vingroup.

| #   | Subsidiary            | Lens             | Mô tả ngắn bài toán                                                                                                                                                                          |
| --- | --------------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | VinFast               | Stakeholder Pain | Khi xe điện có mức pin rất thấp, tài xế phải tự tìm và đánh giá trạm sạc phù hợp. Việc lựa chọn trạm quá xa có thể khiến xe hết pin giữa đường.                                              |
| 2   | Xanh SM               | Time-consuming   | Khi tài xế hủy chuyến, đến muộn hoặc có sự cố, điều phối viên phải kiểm tra vị trí xe, tìm tài xế thay thế và điều phối lại chuyến thủ công.                                                 |
| 3   | Vinhomes              | Repetitive       | Nhân viên CSKH phải đọc, phân loại và soạn phản hồi cho nhiều phản ánh của cư dân về điện, nước, an ninh, vệ sinh hoặc dịch vụ.                                                              |
| 4   | Vinmec                | Time-consuming   | Nhân viên và bác sĩ phải đọc, tổng hợp và tóm tắt hồ sơ bệnh án dài trước khi đưa ra đánh giá hoặc chuẩn bị buổi khám.                                                                       |
| 5   | Vinpearl / VinWonders | AI-upgrade       | Khách hàng thường phải hỏi nhân viên về giá vé, lịch hoạt động, dịch vụ, nhà hàng hoặc lịch trình tham quan; chatbot truyền thống khó xử lý các câu hỏi phức tạp hoặc kết hợp nhiều yêu cầu. |

---

# Phase 2 — QUICK-ASSESS

## Quick Problem Card #1 — VinFast EV Critical Battery Assistant

### Bài toán

Hỗ trợ tài xế VinFast EV lựa chọn phương án sạc an toàn khi pin đang ở mức thấp hoặc mức nguy cấp.

### Công ty thành viên

- [x] VinFast
- [ ] Xanh SM
- [ ] Vinhomes
- [ ] Vinmec
- [ ] Khác

### Actor

Tài xế VinFast EV.

### Workflow thủ công hiện tại

1. Xe phát hiện mức pin thấp và gửi cảnh báo cho tài xế.
2. Tài xế mở hệ thống navigation hoặc bản đồ để tìm trạm sạc.
3. Tài xế tự so sánh khoảng cách giữa các trạm.
4. Tài xế tự đánh giá xem lượng pin còn lại có đủ để di chuyển tới trạm hay không.
5. Tài xế lựa chọn trạm hoặc quyết định gọi hỗ trợ.

### Bottleneck

Bước đánh giá **mức pin còn lại so với khoảng cách tới trạm sạc** là bước có rủi ro cao nhất.

Khi pin ở mức rất thấp, người dùng có thể:

- chọn một trạm quá xa;
- đánh giá sai khả năng di chuyển của xe;
- mất thời gian so sánh nhiều lựa chọn;
- tiếp tục lái trong tình trạng pin không an toàn.

### AI có thể hỗ trợ ở đâu?

AI có thể hỗ trợ tại bước phân tích tình trạng pin và đề xuất phương án phù hợp.

Giải pháp gồm hai phần:

**Rule Engine**

- Kiểm tra các safety constraints.
- Nếu pin `< 5%`, không cho phép đề xuất trạm xa hơn `5 km`.
- Nếu không có trạm phù hợp, chuyển sang phương án Mobile Charging Assistance.

**LLM**

- Hiểu yêu cầu của người dùng.
- Giải thích recommendation bằng ngôn ngữ tự nhiên.
- Trình bày phương án dễ hiểu cho tài xế.

### Success Metrics

- **100%** trường hợp pin `< 5%` không đề xuất trạm sạc xa hơn `5 km`.
- **100%** output tuân thủ structured output yêu cầu.
- **≥ 95%** test cases đưa ra đúng quyết định theo safety policy.
- **100%** hành động quan trọng yêu cầu người dùng xác nhận trước khi thực hiện.

### Quick Architecture

- [ ] No AI
- [x] Rule
- [x] LLM
- [ ] Agent

### Lý do lựa chọn

Rule-based logic phù hợp với các điều kiện safety-critical vì có tính deterministic và dễ kiểm thử.

LLM chỉ nên đảm nhiệm phần hiểu ngôn ngữ và tạo recommendation, không được trực tiếp quyết định hoặc bỏ qua các safety constraints.

---

## Quick Problem Card #2 — Vinhomes Resident Complaint Assistant

### Bài toán

Hỗ trợ nhân viên CSKH Vinhomes tự động phân loại, tóm tắt và soạn bản nháp phản hồi khiếu nại của cư dân.

### Công ty thành viên

- [ ] VinFast
- [ ] Xanh SM
- [x] Vinhomes
- [ ] Vinmec
- [ ] Khác

### Actor

Nhân viên chăm sóc khách hàng Vinhomes.

### Workflow thủ công hiện tại

1. Nhận phản ánh từ cư dân qua ứng dụng, email hoặc hotline.
2. Đọc toàn bộ nội dung phản ánh.
3. Xác định nhóm vấn đề như điện, nước, vệ sinh, an ninh hoặc phí dịch vụ.
4. Tra cứu thông tin liên quan.
5. Soạn phản hồi và chuyển tới bộ phận phụ trách.

### Bottleneck

Bước đọc, phân loại và soạn nội dung phản hồi phải lặp lại nhiều lần trong ngày.

Nhân viên phải xử lý nhiều nội dung có cấu trúc tương tự nhưng cách diễn đạt khác nhau.

### AI có thể hỗ trợ ở đâu?

LLM có thể:

- tự động phân loại khiếu nại;
- tóm tắt nội dung chính;
- xác định mức độ ưu tiên;
- đề xuất bộ phận xử lý;
- tạo bản nháp phản hồi.

Nhân viên vẫn là người kiểm tra và phê duyệt trước khi gửi.

### Success Metrics

- Giảm thời gian xử lý một phản ánh từ khoảng **10 phút xuống dưới 2 phút**.
- **≥ 90%** phản ánh được phân loại đúng nhóm.
- **≥ 90%** bản nháp chỉ cần chỉnh sửa nhỏ trước khi gửi.
- **100%** phản hồi phải được nhân viên phê duyệt trước khi gửi tới cư dân.

### Quick Architecture

- [ ] No AI
- [ ] Rule
- [x] LLM
- [ ] Agent

### Lý do lựa chọn

Bài toán chứa nhiều dữ liệu ngôn ngữ tự nhiên và nội dung phản ánh có cách diễn đạt đa dạng, do đó LLM phù hợp hơn hệ thống rule-based thuần túy.

Tuy nhiên cần Human-in-the-loop trước khi gửi phản hồi chính thức.

---

## Quick Problem Card #3 — Xanh SM Intelligent Incident Dispatcher

### Bài toán

Hỗ trợ điều phối viên Xanh SM xử lý các chuyến xe bất thường và nhanh chóng đề xuất phương án điều phối xe thay thế.

### Công ty thành viên

- [ ] VinFast
- [x] Xanh SM
- [ ] Vinhomes
- [ ] Vinmec
- [ ] Khác

### Actor

Điều phối viên vận hành Xanh SM.

### Workflow thủ công hiện tại

1. Hệ thống hoặc khách hàng báo chuyến xe có vấn đề.
2. Điều phối viên kiểm tra vị trí của tài xế hiện tại.
3. Kiểm tra các tài xế khác đang hoạt động gần khu vực.
4. So sánh khoảng cách, trạng thái và khả năng nhận chuyến.
5. Liên hệ hoặc điều phối tài xế thay thế.

### Bottleneck

Việc tìm và đánh giá tài xế thay thế phù hợp có thể mất nhiều thời gian, đặc biệt trong giờ cao điểm hoặc khi số lượng chuyến bất thường tăng.

### AI có thể hỗ trợ ở đâu?

Hệ thống có thể:

- tổng hợp trạng thái của chuyến xe;
- xác định các tài xế có khả năng thay thế;
- xếp hạng các phương án theo khoảng cách, thời gian đến và trạng thái tài xế;
- tạo recommendation cho điều phối viên.

Các điều kiện bắt buộc như phạm vi địa lý, trạng thái xe hoặc trạng thái tài xế có thể được xử lý bằng rule-based logic.

AI chỉ hỗ trợ đánh giá và đề xuất phương án.

### Success Metrics

- Giảm thời gian tìm phương án điều phối từ khoảng **5 phút xuống dưới 1 phút**.
- **≥ 90%** recommendation nằm trong nhóm phương án mà điều phối viên đánh giá là hợp lý.
- Giảm ít nhất **50%** số thao tác kiểm tra thủ công.
- **100%** các trường hợp ngoại lệ hoặc confidence thấp được chuyển cho điều phối viên xử lý.

### Quick Architecture

- [ ] No AI
- [x] Rule
- [x] LLM
- [ ] Agent

### Lý do lựa chọn

Rule-based logic phù hợp để lọc các phương án không hợp lệ dựa trên vị trí, trạng thái xe và các constraint vận hành.

LLM có thể hỗ trợ tổng hợp tình huống và giải thích recommendation, nhưng quyết định điều phối cuối cùng vẫn thuộc về điều phối viên.

---

# Kết luận Quick Assessment

Sau khi so sánh ba bài toán, nhóm lựa chọn:

## VinFast EV Critical Battery Assistant

làm bài toán chính để tiếp tục Deep-Dive.

### Lý do

1. Bài toán có **Actor** và workflow rõ ràng.
2. Có bottleneck cụ thể và có thể đo lường.
3. Có operational boundary quan trọng liên quan tới safety.
4. Có thể thể hiện rõ sự khác biệt giữa **Rule-based** và **LLM**.
5. Dễ xây dựng adversarial test để kiểm tra việc AI có vượt ranh giới hay không.
6. Có thể xây dựng prototype nhỏ bằng Gemini 2.5 Flash trong phạm vi bài Lab.

Thiết kế ban đầu được lựa chọn là:

**Vehicle Data → Rule Engine → LLM Recommendation → User Confirmation → Action**

Trong đó các safety constraints bắt buộc được kiểm soát bằng Rule Engine thay vì giao hoàn toàn cho LLM.
