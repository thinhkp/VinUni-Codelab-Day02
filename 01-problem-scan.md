# 01 — Problem Scan & Quick Problem Cards

> **Người thực hiện:** ........................................  
> **Nhóm:** ........................................  
> **Lưu ý:** Các metric có dấu `*` là giả định dùng cho bài lab, cần được xác minh bằng log vận hành trước khi triển khai thật.

## Phase 1 — SCAN

| # | Công ty thành viên | Lens | Bài toán/bottleneck |
|---:|---|---|---|
| 1 | Xanh SM | Tốn thời gian | Điều phối viên phải tiếp nhận, xác minh vị trí và hướng dẫn tài xế khi xe điện báo pin yếu/hết pin ngoài đường. |
| 2 | Xanh SM | Lặp lại | Phân loại các cuộc gọi/chat của tài xế về sự cố chuyến đi và chuyển đúng đội vận hành. |
| 3 | VinFast | Lặp lại | Đối chiếu thủ công dữ liệu giao dịch sạc giữa hệ thống trạm, ví điện tử và báo cáo nội bộ. |
| 4 | Vinhomes | AI-upgrade | Phân loại khiếu nại cư dân theo chủ đề, mức độ khẩn cấp và bộ phận phụ trách. |
| 5 | Vinmec | Tốn thời gian | Tóm tắt hồ sơ và hướng dẫn chăm sóc sau khám để bác sĩ kiểm tra trước khi gửi cho bệnh nhân. |
| 6 | Vinpearl | Pain từ người khác | Tổng hợp câu hỏi của khách về vé, giờ mở cửa và tình trạng dịch vụ để giảm thời gian chờ của tổng đài. |

## Phase 2 — QUICK-ASSESS

### Card #1 — Xanh SM: Hỗ trợ sự cố pin ngoài đường

- **Bài toán:** Khi tài xế báo pin dưới ngưỡng an toàn, điều phối viên phải tra cứu thủ công vị trí, trạm sạc còn khả dụng và phương án cứu hộ.
- **Công ty:** Xanh SM (GSM)
- **Actor chịu ảnh hưởng:** Tài xế, điều phối viên, đội cứu hộ và khách đang chờ chuyến.
- **Workflow hiện tại:** Nhận cuộc gọi/chat → hỏi lại biển số, pin và vị trí → tra cứu bản đồ/trạm sạc → soạn hướng dẫn → gọi cứu hộ nếu không có phương án an toàn.
- **Bottleneck:** Tra cứu và tổng hợp thông tin ở bước 3–4, khoảng **12 phút/lượt***.
- **AI hỗ trợ:** Trích xuất thông tin từ tin nhắn/cuộc gọi đã được ghi nhận và tạo bản nháp hướng dẫn cho điều phối viên.
- **Metric:** Giảm thời gian xử lý trung vị từ **15 phút xuống dưới 4 phút***; 100% ca pin dưới 5% được chuyển qua nhánh an toàn.
- **Quick architecture:** Rule/State Machine để kiểm tra ngưỡng và khoảng cách + LLM để tóm tắt và soạn bản nháp.
- **Boundary:** AI không tự gửi tin, không tự điều xe cứu hộ, không được đề xuất trạm vượt giới hạn an toàn.

### Card #2 — Vinhomes: Phân loại khiếu nại cư dân

- **Bài toán:** Khiếu nại trên ứng dụng cư dân cần được phân loại và chuyển bộ phận phụ trách.
- **Công ty:** Vinhomes
- **Actor chịu ảnh hưởng:** Nhân viên CSKH, ban quản lý và cư dân.
- **Workflow hiện tại:** Nhận ticket → đọc nội dung → gắn nhãn thủ công → chuyển bộ phận → soạn phản hồi ban đầu.
- **Bottleneck:** Đọc và gắn nhãn các ticket có nội dung tự do, khoảng **6 phút/ticket***.
- **AI hỗ trợ:** Phân loại chủ đề, trích xuất mức độ khẩn cấp và tạo bản nháp phản hồi có trích dẫn chính sách.
- **Metric:** 85% ticket được phân loại trong **10 giây***; giảm thời gian xử lý ban đầu xuống dưới 2 phút*.
- **Quick architecture:** LLM classifier có confidence threshold + Rule router.
- **Boundary:** Không tự kết luận tranh chấp phí, pháp lý hoặc bồi thường; các ca nhạy cảm phải chuyển người.

### Card #3 — VinFast: Đối chiếu giao dịch sạc

- **Bài toán:** Nhân viên phải tìm chênh lệch giữa nhiều báo cáo giao dịch sạc.
- **Công ty:** VinFast
- **Actor chịu ảnh hưởng:** Nhân viên tài chính/vận hành trạm.
- **Workflow hiện tại:** Tải các file báo cáo → chuẩn hóa cột → ghép theo mã giao dịch → tìm dòng lệch → liên hệ bộ phận liên quan.
- **Bottleneck:** Chuẩn hóa dữ liệu và điều tra các dòng lệch, khoảng **2 giờ/báo cáo***.
- **AI hỗ trợ:** Giải thích nguyên nhân có khả năng gây lệch và tạo danh sách cần kiểm tra.
- **Metric:** Giảm thời gian rà soát từ 2 giờ xuống 30 phút*; không tự động ghi sổ hay điều chỉnh tiền.
- **Quick architecture:** Rule-based reconciliation trước, LLM chỉ giải thích ngoại lệ.
- **Boundary:** AI không thay đổi dữ liệu tài chính; mọi điều chỉnh cần người có thẩm quyền phê duyệt.

## Lựa chọn cho Deep-Dive

Chọn **Card #1 — Xanh SM: Hỗ trợ sự cố pin ngoài đường** vì đây là tình huống thời gian thực, có tác động trực tiếp đến an toàn tài xế và khách hàng, đồng thời có thể giới hạn rủi ro bằng Rule, Human-in-the-loop và Fallback rõ ràng.
