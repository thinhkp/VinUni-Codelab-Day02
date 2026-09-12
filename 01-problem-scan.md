# 🔍 Phase 1 — SCAN: Tìm kiếm cơ hội (Cá nhân)

Dùng 4 lenses để quét qua hoạt động vận hành của các công ty thành viên Vingroup. Mục tiêu là tìm ra các pain point lặp lại, tốn thời gian, dễ nâng cấp bằng AI, hoặc tạo ra nỗi đau rõ ràng cho người dùng / nhân viên.

| # | Subsidiary (VinFast/Xanh SM...) | Lens | Mô tả ngắn bài toán |
|---|----------------------------------|------|---------------------|
| 1 | **Xanh SM** | Lặp lại | Tài xế báo hết pin hoặc sự cố sạc giữa đường cần điều phối viên tìm trạm sạc gần nhất và soạn tin nhắn hướng dẫn thủ công. |
| 2 | **VinFast** | Tốn thời gian | Nhân viên dịch vụ và hỗ trợ khách hàng xử lý hóa đơn sạc, xác minh bảo hành xe, và đối chiếu dữ liệu trạm sạc theo từng giao dịch. |
| 3 | **Vinhomes** | AI-upgrade | Các phản hồi, khiếu nại và yêu cầu cư dân được xử lý theo mẫu lặp lại, khiến nhân viên mất nhiều thời gian phân loại và trả lời. |
| 4 | **Vinmec** | Pain từ người khác | Bác sĩ và nhân viên y tế mất thời gian soạn tóm tắt bệnh án, ghi chú tiếp nhận, và hướng dẫn chăm sóc sau khám. |
| 5 | **Vinpearl / VinWonders** | Tốn thời gian | Điều phối viên xử lý nhiều yêu cầu đổi lịch, đặt lại tour, và phản hồi khách hàng trong giờ cao điểm bằng thủ công. |
| 6 | **Xanh SM** | Lặp lại | Điều phối viên phải đối chiếu nhiều cuộc gọi hủy chuyến, phân loại nguyên nhân, và tổng hợp dữ liệu theo khu vực, thời gian, và tài xế. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards (Cá nhân)

Tôi chọn 3 bài toán có tính thời sự, đo lường được bằng KPI và phù hợp với AI product scoping trong Vin Smart Future.

## Quick Problem Card #1 — Xanh SM: sự cố pin tài xế giữa đường

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Tài xế Xanh SM gặp sự cố hết pin hoặc pin quá    │
│ thấp giữa đường và cần được chỉ dẫn trạm sạc / xe cứu hộ   │
│ kịp thời.                                                   │
│ Công ty thành viên: [x] Xanh SM (GSM)                      │
│                                                             │
│ Ai đang đau (Actor)? Tài xế, điều phối viên trung tâm, và  │
│ khách hàng đang chờ đợi.                                    │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Tài xế gọi tổng đài báo sự cố                           │
│   → 2. Điều phối viên tra cứu vị trí GPS xe                  │
│   → 3. Tìm trạm sạc VinFast gần nhất còn trống               │
│   → 4. Soạn tin nhắn chỉ đường và cảnh báo pin              │
│   → 5. Gọi xe cứu hộ nếu pin quá thấp / không an toàn        │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3-4 (⏱ 10-15 phút/lượt)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Tự động tra cứu vị trí,│
│ chọn trạm phù hợp và draft tin nhắn chỉ đường.               │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian xử lý sự cố từ 15 phút xuống dưới 3 phút.     │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Draft + triage)         │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Problem Card #2 — Vinmec: tóm tắt hồ sơ bệnh án và đơn thuốc

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Bác sĩ và nhân viên Vinmec mất thời gian soạn     │
│ tóm tắt bệnh án, ghi chú khám, và hướng dẫn chăm sóc sau    │
│ điều trị từ nhiều nguồn dữ liệu không đồng nhất.             │
│ Công ty thành viên: [x] Vinmec                              │
│                                                             │
│ Ai đang đau (Actor)? Bác sĩ, điều dưỡng, nhân viên tiếp đón  │
│ và bệnh nhân.                                               │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Nhân viên nhập dữ liệu bệnh án từ nhiều phiếu khám      │
│   → 2. Bác sĩ đọc lại triệu chứng và kết quả xét nghiệm       │
│   → 3. Soạn tóm tắt, lưu ý điều trị và hướng dẫn chăm sóc     │
│   → 4. Gửi cho bệnh nhân hoặc chuyển cho khoa khác           │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ 20-30 phút/bệnh │
│ nhân)                                                      │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Tóm tắt bệnh án và    │
│ đề xuất đoạn văn bản chuẩn để bác sĩ duyệt.                  │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian soạn báo cáo từ 25 phút xuống còn dưới 5 phút│
│ và tăng độ nhất quán trong ghi chú.                          │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Summary + draft)        │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Problem Card #3 — Vinhomes: phân loại và xử lý khiếu nại cư dân

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Vinhomes nhận lượng lớn phản hồi, khiếu nại và    │
│ yêu cầu từ cư dân cần được phân loại và phản hồi nhanh,     │
│ nhưng hiện tại vẫn xử lý bằng thủ công.                     │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau (Actor)? Bộ phận CSKH, quản lý tòa nhà và cư dân │
│ trong các dự án Vinhomes.                                   │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Cư dân gửi tin nhắn / phản hồi qua App hoặc hotline     │
│   → 2. Nhân viên đọc và xác định loại khiếu nại               │
│   → 3. Chuyển sang bộ phận phù hợp và soạn phản hồi          │
│   → 4. Theo dõi SLA và cập nhật trạng thái                    │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ 10-15 phút/lượt)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Phân loại nội dung,   │
│ gợi ý phản hồi và route tự động.                             │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian phản hồi trung bình từ 20 phút xuống dưới 4   │
│ phút và tăng tỉ lệ xử lý trong giờ cao điểm.                 │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Classify + draft + route)│
└─────────────────────────────────────────────────────────────┘
```

---

## 🧭 Lý do tôi chọn 3 bài toán trên

- **Xanh SM – Sự cố pin giữa đường**: có tính thời gian thực, tác động trực tiếp đến doanh thu và trải nghiệm khách hàng, đồng thời dễ đo metric hiệu quả.
- **Vinmec – Tóm tắt bệnh án**: giúp tiết kiệm thời gian lớn cho bác sĩ nhưng cần có kiểm soát chặt chẽ vì rủi ro sai sót có thể ảnh hưởng đến chăm sóc bệnh nhân.
- **Vinhomes – Khiếu nại cư dân**: phù hợp với giải pháp AI để phân loại và soạn nháp, nhưng vẫn cần thêm workflow và kiểm soát người dùng để đảm bảo chính xác.

Những bài toán này vừa mang tính thực tế, vừa phù hợp với mục tiêu của Vin Smart Future: tối ưu hóa quy trình vận hành, giảm lãng phí và nâng cao trải nghiệm khách hàng đồng thời giữ kiểm soát an toàn bằng human-in-the-loop.
