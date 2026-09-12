# 🏗️ Phase 3 — DEEP-DIVE (Nhóm)

## 3.1. Current-State Workflow Mapping (25 min)

Chúng tôi chọn bài toán: **Xanh SM – xử lý sự cố sạc pin / hết pin của tài xế giữa đường**.

Quy trình hiện tại của điều phối viên Xanh SM khi nhận cuộc gọi sự cố pin như sau:

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Nhận cuộc    │     │ Tra cứu định │     │ Tra cứu trạm │     │ Soạn văn bản │
│ gọi sự cố    │ ──→ │ vị GPS xe   │ ──→ │ sạc VinFast  │ ──→ │ hướng dẫn    │
│              │     │              │     │ còn trụ trống│     │ gửi tài xế   │
│ Ai: Dispatch │     │ Ai: Dispatch │     │ Ai: Dispatch │     │ Ai: Dispatch │
│ ⏱ 2 phút     │     │ ⏱ 2 phút     │     │ ⏱ 5 phút 🔴  │     │ ⏱ 5 phút 🔴  │
│ In: Điện thoại│     │ In: Biển số  │     │ In: Vị trí GPS│     │ In: Raw data │
│ Out: Log sự cố│     │ Out: Toạ độ  │     │ Out: Địa chỉ │     │ Out: SMS     │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │ Bước 5       │
                                                               │ Gọi xe cứu   │
                                                               │ hộ (nếu cần) │
                                                               │ Ai: Dispatch │
                                                               │ ⏱ 1 phút     │
                                                               └──────────────┘
🔴 = Bottleneck
⏱ Tổng thời gian xử lý thủ công: 15 phút/lượt.
```

### Nhận xét:
- **Handoff** xảy ra giữa tài xế và trung tâm điều vận, và giữa trung tâm điều vận với hệ thống bản đồ / trạm sạc.
- **Bottleneck** rõ ràng nằm ở các bước **3 và 4**: tra cứu trạm sạc phù hợp và soạn tin nhắn hướng dẫn cho tài xế.
- Mỗi cuộc xử lý mất khoảng **15 phút/lượt**, điều này là quá dài khi xe đang ở tình trạng sắp cạn pin hoặc không an toàn nếu tiếp tục chạy.

---

## 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) tại trung tâm điều vận Xanh SM và tài xế xe điện đang gặp sự cố pin. |
| **2. Current Workflow** | Khi tài xế báo sự cố hết pin hoặc pin rất thấp, điều phối viên phải tra cứu vị trí GPS xe, mở hệ thống trạm sạc VinFast, tìm trụ sạc gần nhất còn trống, soạn thảo tin nhắn chỉ đường, và nếu cần gọi xe cứu hộ. Tất cả đều làm thủ công và mất khoảng 15 phút/lượt. |
| **3. Bottleneck** | Bước 3 và 4 là nghẽn lớn nhất: tra cứu thủ công trụ sạc khả dụng phù hợp với dòng xe (VF5 / VFe34 / VF8) và soạn thảo tin nhắn chỉ dẫn bằng Tiếng Việt. |
| **4. Business Impact** | Mỗi ngày có khoảng 80 sự cố pin thực địa ở Hà Nội. Team điều vận mất khoảng 20 giờ làm việc mỗi ngày cho xử lý thủ công. Tài xế phải chờ lâu hơn, doanh thu bị rò rỉ do xe không thể đón khách đúng thời điểm, và trải nghiệm khách hàng xấu đi. |
| **5. Success Metric** | 1. Giảm thời gian xử lý sự cố từ 15 phút xuống dưới 3 phút (Efficiency).<br>2. Tỉ lệ đề xuất trạm sạc đúng và phù hợp với xe đạt ≥ 98% (Quality).<br>3. Giảm số trường hợp chờ quá lâu / quá tải của điều phối viên. |
| **6. Operational Boundary** | AI được phép truy xuất vị trí xe, tra cứu trạm sạc gần nhất còn trống và draft một tin nhắn hướng dẫn ngắn gọn để điều phối viên duyệt. **CẤM:** AI không được tự động gửi tin mà không có phê duyệt của người vận hành; không được đề xuất trạm sạc quá xa khi pin dưới 5%; không được chọn trạm không tương thích với loại cổng / mẫu xe. |

---

## 3.3. Future-State Flow & AI Fit

### AI Fit
**Chọn: LLM Feature**

Lý do:
- Đây là quy trình có cấu trúc rõ (nhận tin báo, xác định vị trí, đưa ra trạm phù hợp, viết nháp chỉ dẫn), nên không cần Agentic Loop phức tạp.
- Rủi ro nếu sai sót có thể ảnh hưởng trực tiếp đến an toàn tài xế và tính khả dụng của dịch vụ, nên cần **Human-in-the-loop** và **fallback** rõ ràng.
- AI nên làm tốt nhất ở bước **drafting và triage**, không phải tự quyết định cuối cùng.

### Future-State Flow

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Nhận cuộc    │     │ 🔵 Auto-pull │     │ 🔵 AI draft  │     │ 🟢 Người vận  │
│ gọi sự cố    │ ──→ │ vị trí GPS + │ ──→ │ tin nhắn     │ ──→ │ hành duyệt & │
│              │     │ trạm sạc trống│    │ chỉ đường    │     │ gửi cho xe   │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ↩️ Fallback:
                                                               Nếu AI draft lỗi
                                                               hoặc không chắc chắn,
                                                               điều phối viên tự viết
                                                               tin nhắn thủ công.
```

### Human-in-the-loop & Fallback
- **Human Step (HITL):** điều phối viên xem và phê duyệt lời nhắn, chọn có gửi hay không.
- **Fallback:** nếu AI không có đủ dữ liệu hoặc mức độ tin cậy thấp, hệ thống chuyển về quy trình thủ công cũ.
- **Safety rule:** nếu pin < 5%, AI không đề xuất trạm quá 5 km; thay vào đó, đề xuất xe cứu hộ pin di động.

---

# 🏁 Phase 5 — EVALUATE (Nhóm)

## AI Readiness Checklist

1. [x] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test?  
   Có các log vị trí xe, trạm sạc, và lưu lượng sự cố pin trong vận hành thực tế.
2. [x] Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback)?  
   Có, vì kết quả AI chỉ ở dạng draft và cần được người vận hành duyệt trước khi gửi.
3. [x] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ?  
   Có, vì quy trình hiện tại đang tốn thời gian và tạo áp lực lớn cho team điều vận.

## Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:
[ ] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp.
[x] **NOT YET (Cần tích lũy thêm dữ liệu/xác lập baseline):** Trì hoãn để chuẩn bị thêm.
[ ] **NO-GO (Không khả thi / Rule-based tốt hơn):** Hủy bỏ dự án AI này.

### Justification (Lý giải quyết định dựa trên bằng chứng)

Chúng tôi đánh giá đây là dự án **NOT YET** thay vì **GO** ngay vì:
- Bài toán có giá trị rõ ràng và dễ đo lường, nhưng dữ liệu thực tế về vị trí xe, trạm sạc và lịch sử sự cố cần được thu thập và làm sạch thêm.
- Cần có kiểm tra chính xác với việc chọn trạm phù hợp theo loại pin và mô hình xe, vì nếu sai có thể gây rủi ro an toàn cho tài xế.
- Quy trình hiện tại đã có phần rule-based rõ ràng, nên AI nên được triển khai ở scope hẹp trước, ví dụ: auto-draft + triage, không phải auto-send hay tự xác định hành động độc lập.

Với những yếu tố trên, dự án này nên được bắt đầu bằng một prototype hẹp, có HITL và fallback, rồi mở rộng sau khi có dữ liệu thực nghiệm tốt hơn.

---

## Kết luận

Dự án **Xanh SM – xử lý sự cố pin tài xế giữa đường** phù hợp với hướng tiếp cận AI product scoping của Vin Smart Future vì bài toán này có:
- tác động trực tiếp tới vận hành và trải nghiệm khách hàng,
- tính lặp lại và tốn thời gian rõ ràng,
- metric có thể đo được,
- và ranh giới an toàn có thể kiểm soát bằng prompt + human review.

Nếu triển khai đúng cách, AI có thể giảm thời gian xử lý từ khoảng **15 phút xuống dưới 3 phút**, đồng thời giảm áp lực cho người điều phối và nâng cao độ tin cậy của dịch vụ.
