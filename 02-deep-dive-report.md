# 02 — Problem Deep-Dive

## Lab 02 — AI Product Scoping | Vin Smart Future

---

# Selected Problem

## VinFast EV Critical Battery Assistant

Xây dựng hệ thống hỗ trợ tài xế VinFast EV khi pin ở mức thấp hoặc nguy cấp bằng cách kết hợp **deterministic safety rules** và **LLM recommendation**.

Mục tiêu của hệ thống là giúp người dùng lựa chọn phương án sạc an toàn, dễ hiểu và nhanh chóng mà không để LLM tự quyết định các điều kiện safety-critical.

---

# Phase 3 — DEEP-DIVE

# 3.1 Current-State Workflow Mapping

## Actor

**VinFast EV Driver**

Tài xế là người trực tiếp nhận cảnh báo pin yếu, tìm kiếm trạm sạc và quyết định nên tiếp tục di chuyển hay yêu cầu hỗ trợ.

---

## Current Workflow

```text
Vehicle Battery Monitoring
        │
        ▼
Battery becomes low
        │
        ▼
Vehicle sends warning
        │
        ▼
Driver opens navigation/map
        │
        ▼
🔴 Driver manually searches charging stations
        │
        ▼
Driver compares:
- battery level
- station distance
- estimated route
        │
        ▼
🔴 Driver decides whether the vehicle
can safely reach the station
        │
        ▼
Driver selects station
        │
        ▼
Navigate to charging station
```

---

## Workflow Steps

| Step | Actor/System | Activity                                            | Estimated Time | Issue                        |
| ---- | ------------ | --------------------------------------------------- | -------------: | ---------------------------- |
| 1    | Vehicle      | Detect low battery level                            |      Automatic | No major issue               |
| 2    | Vehicle      | Notify driver                                       |      Automatic | No major issue               |
| 3    | Driver       | Open navigation and search charging stations        |      20–40 sec | Manual interaction           |
| 4    | Driver       | Compare station distances and remaining battery     |      30–90 sec | 🔴 Bottleneck                |
| 5    | Driver       | Decide whether the car can reach the station safely |      10–30 sec | 🔴 Safety-critical decision  |
| 6    | Driver       | Select station and start navigation                 |      10–20 sec | Depends on previous decision |

### Estimated Total Manual Decision Time

**Approximately 1–3 minutes per incident**

Thời gian có thể tăng trong trường hợp người dùng đang ở khu vực lạ, có nhiều trạm để lựa chọn hoặc mức pin đang ở trạng thái nguy cấp.

---

# Bottleneck

Bottleneck chính nằm ở bước:

## Driver manually evaluates whether the remaining battery is sufficient to reach a charging station.

Tài xế phải tự kết hợp nhiều thông tin:

- mức pin hiện tại;
- khoảng cách tới trạm;
- vị trí hiện tại;
- nhiều lựa chọn trạm khác nhau;
- mức độ khẩn cấp của tình huống.

Trong tình trạng pin thấp, tài xế có thể:

- lựa chọn một trạm quá xa;
- đánh giá quá cao phạm vi còn lại của xe;
- mất nhiều thời gian so sánh;
- bỏ qua một phương án an toàn hơn;
- tiếp tục lái khi nên yêu cầu hỗ trợ.

---

# Handoffs

```text
Vehicle Telemetry
      │
      ▼
Navigation / Charging Station Data
      │
      ▼
Driver
      │
      ▼
Charging Infrastructure
```

Các điểm handoff quan trọng:

1. **Vehicle → Driver**
   Xe gửi cảnh báo tình trạng pin.

2. **Navigation System → Driver**
   Hệ thống cung cấp danh sách trạm nhưng tài xế vẫn phải tự đánh giá.

3. **Driver → Charging Service**
   Tài xế quyết định trạm và bắt đầu di chuyển.

---

# 3.2 Problem Statement — 6 Fields

## 1. Actor / Operator

**Primary Actor:** VinFast EV Driver.

Actor là tài xế đang sử dụng xe điện VinFast và phải xử lý tình huống pin thấp trong quá trình di chuyển.

Secondary actors có thể bao gồm:

- VinFast navigation system;
- charging station service;
- roadside/mobile charging support.

---

## 2. Current Workflow

Khi mức pin xuống thấp:

1. Xe cảnh báo tài xế.
2. Tài xế mở hệ thống navigation.
3. Tài xế tìm các trạm sạc gần đó.
4. Tài xế tự so sánh khoảng cách với mức pin còn lại.
5. Tài xế lựa chọn một trạm hoặc quyết định yêu cầu hỗ trợ.

Quy trình hiện tại phụ thuộc nhiều vào việc tài xế tự đánh giá tình huống.

---

## 3. Bottleneck

Bottleneck chính là bước:

**Đánh giá xem xe có đủ khả năng di chuyển an toàn tới một trạm sạc cụ thể hay không.**

Vấn đề trở nên nghiêm trọng khi:

- pin ở mức rất thấp;
- trạm sạc ở xa;
- có nhiều lựa chọn;
- tài xế phải đưa ra quyết định nhanh.

Đây là bước vừa tốn thời gian vừa có yếu tố safety-critical.

---

## 4. Business Impact

Nếu tài xế đưa ra quyết định không phù hợp:

- xe có thể hết pin trước khi đến trạm;
- người dùng phải yêu cầu cứu hộ;
- trải nghiệm khách hàng giảm;
- tăng workload cho bộ phận roadside assistance;
- ảnh hưởng tới mức độ tin tưởng vào hệ sinh thái EV.

Ở góc độ vận hành, hệ thống hỗ trợ ra quyết định tốt có thể:

- giảm số tình huống hết pin giữa đường;
- giảm thao tác tìm kiếm thủ công;
- cải thiện thời gian phản ứng;
- tăng mức độ hài lòng của người dùng.

---

## 5. Success Metrics

### Safety Metrics

- **100%** trường hợp `battery_level < 5%` không được đề xuất trạm có khoảng cách `> 5 km`.
- **100%** recommendation vượt qua safety validation trước khi hiển thị cho người dùng.
- **100%** hành động bên ngoài hệ thống yêu cầu xác nhận của người dùng.

### AI Quality Metrics

- **≥ 95% policy compliance** trên bộ test scenarios.
- **100% valid structured output** trong test set.
- **≥ 95%** recommendation phù hợp với decision policy đã định nghĩa.

### Efficiency Metrics

Giảm thời gian tìm và đánh giá phương án từ:

**1–3 phút → dưới 30 giây**

trong các tình huống tiêu chuẩn.

---

## 6. Operational Boundary

AI được phép:

- nhận thông tin mức pin;
- nhận danh sách trạm sạc;
- phân tích các phương án đã vượt qua safety rules;
- giải thích recommendation;
- đề xuất người dùng yêu cầu hỗ trợ;
- tạo output ở trạng thái draft/advisory.

AI **không được phép**:

1. Bỏ qua safety rule.
2. Đề xuất trạm bị Rule Engine đánh dấu không an toàn.
3. Tự ý điều khiển xe.
4. Tự động thay đổi route mà không có xác nhận.
5. Tự động gọi hoặc dispatch roadside assistance.
6. Tự tuyên bố rằng một hành động đã được thực hiện.
7. Cho phép instruction của người dùng override safety constraint.
8. Tự tạo thông tin về trạm sạc không tồn tại trong input data.

---

# 3.3 AI Fit Analysis

Để lựa chọn architecture phù hợp, nhóm đánh giá ba phương án:

- Rule / State Machine
- LLM Feature
- Agentic Loop

---

## Option 1 — Rule / State Machine

Ví dụ:

```python
if battery_level < 5 and station_distance > 5:
    reject_station()
```

### Strengths

- deterministic;
- predictable;
- dễ test;
- dễ audit;
- phù hợp với safety constraint;
- không có hallucination ở logic điều kiện.

### Weaknesses

- khó xử lý natural language;
- khó giải thích recommendation linh hoạt;
- khó tương tác với yêu cầu không có cấu trúc.

### Conclusion

**Rất phù hợp cho safety layer.**

---

# Option 2 — LLM Feature

LLM có thể:

- hiểu yêu cầu người dùng;
- tóm tắt tình huống;
- giải thích vì sao một lựa chọn phù hợp;
- trình bày recommendation theo ngôn ngữ tự nhiên;
- xử lý các cách diễn đạt khác nhau.

### Strengths

- flexible;
- natural language interaction;
- recommendation dễ hiểu hơn;
- hỗ trợ trải nghiệm người dùng.

### Weaknesses

- probabilistic;
- có thể hallucinate;
- có thể không tuân thủ instruction;
- không phù hợp để enforce safety rule một mình.

### Conclusion

**Phù hợp cho interaction và explanation layer.**

---

# Option 3 — Agentic Loop

Agent có thể tự động:

```text
Read vehicle data
      ↓
Search stations
      ↓
Evaluate options
      ↓
Change route
      ↓
Contact assistance
```

### Strengths

- tự động hóa cao;
- có thể thực hiện workflow nhiều bước.

### Weaknesses

- action risk cao;
- phức tạp;
- khó kiểm soát;
- khó audit;
- không cần thiết cho prototype ban đầu;
- failure có thể tạo hậu quả thực tế.

### Conclusion

**Không chọn Agentic Loop cho scope hiện tại.**

---

# Final AI Fit Decision

## Hybrid Architecture

```text
Rule Engine + LLM Feature + Human Confirmation
```

Trong đó:

### Rule Engine

chịu trách nhiệm:

- safety constraints;
- filtering;
- eligibility;
- hard decision boundaries.

### LLM

chịu trách nhiệm:

- natural language understanding;
- explanation;
- summarization;
- recommendation presentation.

### Human

chịu trách nhiệm:

- xác nhận action cuối cùng.

---

# 3.4 Future-State Flow

```text
Vehicle Telemetry
Battery %, Location
        │
        ▼
Charging Station Data
        │
        ▼
┌──────────────────────────┐
│     SAFETY RULE ENGINE   │
└──────────────────────────┘
        │
        ▼
Check Battery Level
        │
        ├────────────────────────────────┐
        │                                │
        ▼                                ▼
Battery >= 5%                    Battery < 5%
        │                                │
        ▼                                ▼
Filter available              Apply CRITICAL
charging stations             safety policy
        │                                │
        │                       Is station <= 5 km?
        │                          /            \
        │                        YES             NO
        │                         │               │
        ▼                         ▼               ▼
Valid station list        Valid safe       Recommend
        │                  station list     Mobile Charging
        │                         │          Assistance
        └────────────┬────────────┘               │
                     │                            │
                     └───────────┬────────────────┘
                                 ▼
                       🔵 LLM Recommendation
                                 │
                                 ▼
                     Structured Output Validator
                                 │
                      ┌──────────┴──────────┐
                      │                     │
                    VALID                INVALID
                      │                     │
                      ▼                     ▼
               🟢 Human Review         ↩️ Fallback
               / Confirmation          Rule Message
                      │
                      ▼
               User-selected Action
```

---

# AI Step

🔵 **LLM Recommendation**

LLM nhận các phương án đã được safety rule lọc trước.

LLM không được quyền tự thay đổi eligibility của các phương án.

Ví dụ input:

```json
{
  "battery_level": 3,
  "safe_stations": [],
  "mobile_assistance_available": true
}
```

LLM có thể tạo:

```json
{
  "status": "CRITICAL",
  "decision": "MOBILE_CHARGING_ASSISTANCE",
  "requires_confirmation": true,
  "reason": "No charging station is available within the permitted safety distance."
}
```

---

# Human-in-the-loop

🟢 **Human confirmation is mandatory before execution.**

Tài xế phải xác nhận trước khi:

- bắt đầu navigation;
- thay đổi route;
- gửi yêu cầu mobile charging;
- liên hệ roadside assistance.

LLM chỉ tạo recommendation.

Luồng:

```text
AI Recommendation
        │
        ▼
Driver sees recommendation
        │
        ▼
Driver reviews
        │
    ┌───┴───┐
    │       │
 Confirm   Reject
    │       │
    ▼       ▼
 Action   Manual selection
```

---

# Fallback

Fallback được kích hoạt khi:

- LLM output không phải valid JSON;
- thiếu required field;
- recommendation vi phạm safety policy;
- model trả về station không có trong input;
- model confidence không đủ;
- service/API bị lỗi;
- LLM không phản hồi.

---

## Fallback Flow

```text
LLM Response
     │
     ▼
Output Validator
     │
 ┌───┴─────────┐
 │             │
PASS          FAIL
 │             │
 ▼             ▼
Display      Reject Response
Result           │
                 ▼
        Deterministic Rule Message
                 │
                 ▼
           Driver decides
```

Ví dụ deterministic fallback:

```text
Battery level is critically low.

No verified charging station is available within
the permitted safety distance.

Please stop in a safe location and request
charging assistance.

No action has been executed automatically.
```

---

# Why Hybrid Instead of LLM-only?

Safety-critical logic không nên phụ thuộc vào probabilistic model.

Ví dụ rule:

```text
battery < 5%
AND
station_distance > 5 km
```

có thể được xác định chính xác bằng code.

Sử dụng LLM để quyết định điều kiện này sẽ:

- tăng complexity;
- giảm predictability;
- tăng khả năng vi phạm policy;
- khó kiểm thử hơn.

Vì vậy:

```text
Rules decide WHAT IS ALLOWED.

LLM explains WHAT THE USER SHOULD CONSIDER.

Human decides WHAT ACTION TO TAKE.
```

---

# Phase 5 — EVALUATE

# AI Readiness Checklist

## 1. Có sẵn dữ liệu mẫu/logs sạch để test?

### Status

**[x] YES — for prototype scope**

Prototype có thể sử dụng dữ liệu synthetic hoặc test fixtures gồm:

- battery percentage;
- station distance;
- station availability;
- assistance availability;
- user instruction.

Ví dụ:

```json
{
  "battery_level": 3,
  "stations": [
    {
      "name": "Station A",
      "distance_km": 8
    },
    {
      "name": "Station B",
      "distance_km": 12
    }
  ]
}
```

Tuy nhiên dữ liệu production thực tế sẽ cần được tích hợp sau.

---

# 2. Rủi ro khi AI sai có nằm trong tầm kiểm soát?

### Status

**[x] YES — if Rule Engine + HITL + Fallback are enforced**

Risk được kiểm soát thông qua:

1. deterministic safety rules;
2. structured output;
3. schema validation;
4. post-generation safety validation;
5. Human-in-the-loop;
6. deterministic fallback.

LLM không được giao quyền trực tiếp thực hiện safety-critical action.

---

# 3. Stakeholders có sẵn sàng thay đổi workflow?

### Status

**[x] POSSIBLY / YES for limited prototype**

Giải pháp không thay thế toàn bộ workflow hiện tại.

Thay vì:

```text
Driver → manually search → manually compare → decide
```

workflow mới là:

```text
Driver → receives recommendation → reviews → confirms
```

Do đó mức độ thay đổi quy trình tương đối thấp.

---

# AI Readiness Summary

| Criteria                          | Status                 | Reason                                          |
| --------------------------------- | ---------------------- | ----------------------------------------------- |
| Test data available               | ✅ Ready for prototype | Synthetic scenarios có thể tạo dễ dàng          |
| AI risk controllable              | ✅ Yes                 | Rule Engine + validation + HITL                 |
| Workflow integration feasible     | ✅ Yes                 | AI đóng vai trò recommendation layer            |
| Success metrics measurable        | ✅ Yes                 | Có policy compliance, latency và safety metrics |
| Safety boundaries defined         | ✅ Yes                 | Quy định rõ quyền của Rule, LLM và Human        |
| Fully autonomous deployment ready | ❌ No                  | Chưa đủ điều kiện cho autonomous actions        |

---

# Final Decision

# ✅ GO — Build a Narrow-Scope Prototype

Nhóm quyết định **GO** với điều kiện prototype chỉ hoạt động trong scope:

```text
Decision Support
NOT
Autonomous Vehicle Control
```

---

# Justification

Bài toán phù hợp để phát triển prototype vì:

1. **Problem rõ ràng**

Tài xế gặp khó khăn khi phải lựa chọn phương án sạc trong tình trạng pin thấp.

2. **Có thể đo lường**

Các metric về safety compliance, response time và structured output đều có thể kiểm thử.

3. **Risk có thể kiểm soát**

Safety-critical decisions không được giao hoàn toàn cho LLM.

Rule Engine enforce các hard constraints trước và sau LLM.

4. **LLM có value rõ ràng**

LLM cải thiện:

- natural language interaction;
- explanation;
- recommendation presentation.

5. **Human vẫn giữ quyền quyết định**

Mọi hành động quan trọng đều yêu cầu driver confirmation.

6. **Prototype có scope nhỏ**

Hệ thống ban đầu chỉ cần:

```text
battery data
+
station list
+
safety rules
+
LLM recommendation
```

nên có thể được triển khai và test độc lập trước khi tích hợp sâu vào hệ thống VinFast.

---

# Conditions Before Production Deployment

Quyết định **GO** chỉ áp dụng cho prototype.

Trước production cần:

- dữ liệu telemetry thực tế;
- charging station API;
- validation với kỹ sư domain;
- large-scale test scenarios;
- latency testing;
- failure-mode testing;
- security testing;
- prompt-injection testing;
- monitoring;
- logging;
- escalation mechanism.

Không nên cho phép autonomous execution cho tới khi các yêu cầu trên được kiểm chứng.

---

# Final Architecture

```text
┌───────────────────────────────┐
│      Vehicle Telemetry        │
│ Battery / Position / Status   │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Station Database        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Safety Rule Engine      │
│                               │
│ deterministic constraints     │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│             LLM               │
│                               │
│ explanation + recommendation  │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Output Validation       │
└───────────────┬───────────────┘
                │
        ┌───────┴───────┐
        │               │
      VALID           INVALID
        │               │
        ▼               ▼
┌───────────────┐  ┌───────────────┐
│ Driver Review │  │   Fallback    │
└───────┬───────┘  │ Rule Message  │
        │          └───────┬───────┘
        ▼                  │
 Driver confirms ◄─────────┘
        │
        ▼
 User-selected action
```

---

# Final Conclusion

Dự án được đánh giá:

## GO — Prototype

với architecture:

## Rule Engine + LLM + HITL + Fallback

Nguyên tắc thiết kế quan trọng nhất là:

> **LLM không phải safety controller.**

Rule Engine kiểm soát những gì hệ thống **được phép** đề xuất.

LLM chỉ hỗ trợ giải thích và giao tiếp.

Human giữ quyền quyết định cuối cùng.

Điều này giúp tận dụng lợi thế của Generative AI nhưng vẫn giữ được tính predictability và khả năng kiểm soát cần thiết cho một bài toán liên quan đến vận hành xe điện.
