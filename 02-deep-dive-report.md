# 02 — Deep-Dive Report: Hỗ trợ sự cố pin cho tài xế Xanh SM

> **Phạm vi:** Bản phân tích cho bài lab, không phải quy trình vận hành chính thức.  
> **Dữ liệu:** Các thời gian và tỷ lệ có dấu `*` là giả định để thiết kế prototype; cần đo bằng log thật.

## 1. Current-State Workflow

```text
[Tài xế báo pin yếu/hết]
            |
            v
[Điều phối viên nhận cuộc gọi/chat - 2 phút]
            |
            +--> Handoff: tài xế cung cấp biển số, pin, vị trí
            v
[Tra cứu GPS và xác minh thông tin - 2 phút]
            |
            v
[Tra cứu trạm sạc và tình trạng trụ - 5 phút]  🔴 BOTTLENECK
            |
            v
[Soạn hướng dẫn bằng tay - 5 phút]              🔴 BOTTLENECK
            |
            +--> Nếu không an toàn: Handoff sang đội cứu hộ
            v
[Điều phối viên gửi hướng dẫn sau khi kiểm tra]
```

| Bước | Người/hệ thống | Thời gian giả định* | Vấn đề |
|---|---|---:|---|
| Nhận thông tin | Điều phối viên + tài xế | 2 phút | Thông tin thường thiếu hoặc không đồng nhất |
| Xác minh vị trí/pin | Điều phối viên + bản đồ | 2 phút | Phải hỏi lại khi GPS không rõ |
| Tìm phương án | Điều phối viên + hệ thống trạm | 5 phút | Nhiều màn hình, dễ chọn nhầm |
| Soạn hướng dẫn | Điều phối viên | 5 phút | Chậm và không nhất quán |
| Cứu hộ/fallback | Điều phối viên + đội cứu hộ | Tuỳ ca | Có thể chậm nếu không có escalation rõ |

## 2. Problem Statement — 6 fields

| Field | Phân tích |
|---|---|
| **1. Actor / Operator** | Điều phối viên trung tâm vận hành là người tiếp nhận, xác minh và quyết định phương án. Tài xế là người cung cấp dữ liệu hiện trường. |
| **2. Current Workflow** | Tài xế báo sự cố qua chat/điện thoại; điều phối viên hỏi lại thông tin; tra GPS, bản đồ và trạng thái trạm; sau đó soạn hướng dẫn hoặc chuyển đội cứu hộ. |
| **3. Bottleneck** | Tra cứu nhiều nguồn và soạn hướng dẫn thủ công. Đây cũng là điểm dễ dẫn đến việc đề xuất trạm quá xa khi pin đang ở mức nguy hiểm. |
| **4. Business Impact** | Theo giả định dùng trong lab, mỗi ca mất khoảng 15 phút*; thời gian chờ tăng, điều phối viên bị chiếm tải và có rủi ro an toàn nếu hướng dẫn không phù hợp. Chưa coi đây là số liệu thực tế của Xanh SM. |
| **5. Success Metric** | P50 thời gian từ lúc nhận đủ thông tin đến bản nháp dưới 4 phút*; 100% ca pin dưới 5% không nhận đề xuất trạm vượt giới hạn; điều phối viên duyệt được bản nháp trong một màn hình. |
| **6. Operational Boundary** | AI chỉ đọc dữ liệu đã được cấp quyền, kiểm tra điều kiện an toàn và tạo bản nháp. AI tuyệt đối không tự gửi tin, không tự điều xe cứu hộ, không tự quyết định khi dữ liệu thiếu, và phải chuyển người khi confidence thấp hoặc có dấu hiệu nguy hiểm. |

## 3. Future-State Flow & AI Fit

### 3.1. AI Fit decision

Chọn **Rule + LLM Feature**, không chọn Agent tự trị:

- **Rule/State Machine:** kiểm tra pin, khoảng cách, trạng thái trạm, dữ liệu bắt buộc và các điều kiện chặn.
- **LLM Feature:** tóm tắt nội dung tự do và tạo bản nháp tiếng Việt theo schema cố định.
- **Không dùng Agentic Loop:** hành động gọi cứu hộ/gửi tin có tác động thực tế, không nên để mô hình tự lập kế hoạch và thực thi.

### 3.2. Future-State Flow

```text
[Input: chat/call transcript + GPS + pin]
                  |
                  v
[Rule: đủ dữ liệu? pin có dưới 5%?]
       | Không                    | Có
       v                         v
[Hỏi bổ sung / HITL]   [Rule: chặn trạm > 5 km]
                                  |
                                  v
                      [Đề xuất dispatch mobile charger]
                                  |
                                  v
                 [LLM tạo DRAFT_ONLY theo JSON schema]
                                  |
                                  v
                    [HITL: điều phối viên kiểm tra]
                         | Duyệt       | Từ chối/lỗi
                         v              v
                   [Gửi thủ công]   [Fallback gọi cứu hộ]
```

### 3.3. Human-in-the-loop và Fallback

1. Điều phối viên phải xác nhận biển số, vị trí, phần trăm pin và tình trạng tài xế trước khi duyệt.
2. Nếu thiếu trường bắt buộc, hệ thống không đoán; trả về danh sách câu hỏi cần hỏi tài xế.
3. Nếu pin dưới 5%, nhánh an toàn được quyết định bằng Rule trước khi gọi LLM.
4. Nếu JSON không hợp lệ, confidence thấp, API lỗi hoặc dữ liệu trạm quá cũ, không gửi output cho tài xế; chuyển sang quy trình gọi cứu hộ thủ công.
5. Lưu audit log gồm input đã chuẩn hóa, quyết định Rule, bản nháp, người duyệt và thời điểm gửi.

## 4. Evaluate — AI Readiness Checklist

| Câu hỏi | Đánh giá | Bằng chứng/việc cần làm |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test chưa? | **Chưa đủ** | Cần tối thiểu một tập transcript đã ẩn thông tin cá nhân, dữ liệu GPS giả lập và trạng thái trạm theo thời điểm. |
| Rủi ro AI sai có kiểm soát được không? | **Có, nếu giữ HITL** | Rule chặn tuyến nguy hiểm, schema validation, confidence threshold và fallback cứu hộ. |
| Stakeholder sẵn sàng đổi quy trình không? | **Cần xác minh** | Phỏng vấn điều phối viên và đội cứu hộ; thử nghiệm shadow mode trước khi gửi thật. |

## 5. Quyết định

### **NOT YET — Làm prototype giới hạn, chưa triển khai tự động**

Lý do:

- Bài toán có giá trị và có thể đo bằng thời gian xử lý, nhưng dữ liệu vận hành thật và baseline hiện chưa được xác nhận.
- Rủi ro an toàn không phù hợp với việc cho LLM tự gửi tin hoặc tự gọi cứu hộ.
- Có thể tiếp tục bằng prototype offline/shadow mode: dùng dữ liệu giả lập, kiểm tra Rule và JSON, sau đó để điều phối viên so sánh với quy trình hiện tại.

**Điều kiện để chuyển sang GO:** có dữ liệu ẩn danh đủ để đánh giá, đạt 100% kiểm thử ca pin dưới ngưỡng không vượt ranh giới, có phê duyệt của vận hành/an toàn và hoàn tất thử nghiệm shadow mode.
