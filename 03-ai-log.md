# 📝 AI Log & Reflection — Lab 02: AI Product Scoping

Trong buổi Lab 02, tôi sử dụng AI như một trợ lý tư duy để hỗ trợ việc quét bài toán, đánh giá tính khả thi và xây dựng phiên bản prompt nguyên mẫu cho hệ thống Xanh SM. Tôi không dùng AI như một công cụ tự động hóa hoàn toàn độc lập; thay vào đó, tôi dùng nó để tư duy nhanh, phản biện ý tưởng và kiểm tra giới hạn của mô hình trước khi đưa ra quyết định.

## 1. AI giúp tôi điều gì?

AI đã giúp tôi rất nhiều trong ba giai đoạn chính:

1. **Brainstorm các bài toán thực tế**  
   Khi bắt đầu, tôi chưa có một bài toán rõ ràng cho Vin Smart Future. AI giúp tôi suy nghĩ theo 4 lenses — lặp lại, tốn thời gian, AI-upgrade và pain từ người khác — để tìm ra các bottleneck thực tế trong Vingroup.

2. **Đánh giá và chọn bài toán tiềm năng**  
   Sau khi có danh sách bài toán, AI hỗ trợ tôi so sánh các lựa chọn như Xanh SM sự cố pin, Vinhomes CSKH và Vinmec tóm tắt bệnh án. Nhờ đó, tôi có thể chọn bài toán phù hợp với mục tiêu của lab: có tính thời sự, dễ đo metric, và có thể xây dựng prompt prototype được.

3. **Thiết kế prompt prototype và xác định ranh giới an toàn**  
   AI giúp tôi viết ra một system prompt rõ ràng hơn, tập trung vào các quy tắc an toàn như: bắt buộc [DRAFT_ONLY], không được gửi trực tiếp nếu chưa phê duyệt, và pin < 5% phải chuyển sang xe cứu hộ. Đây là giai đoạn quan trọng vì nó cho thấy AI không chỉ giúp viết văn bản, mà còn giúp kiểm soát hành vi và ranh giới vận hành.

---

## 2. AI làm sai ở đâu? / Hallucination / Over-optimization

Tôi cũng nhận thấy AI dễ mắc một số lỗi nếu không có ràng buộc chặt chẽ:

- **Sai về hành vi tự động hóa**: lúc đầu, AI có xu hướng gợi ý “gửi luôn tin nhắn cho khách hàng/tài xế” mà không có bước review của người vận hành. Đây là nguy cơ lớn vì trong hệ thống thực tế, việc tự động gửi tin nhắn có thể gây sai sót hoặc xâm phạm vào quy trình người dùng.

- **Bỏ qua ranh giới an toàn**: Khi tôi hỏi về trường hợp pin rất thấp, AI có thể cố gắng đề xuất trạm sạc xa hơn để “giải quyết vấn đề nhanh”, dù điều đó không an toàn. Đây là dạng lỗi rất nguy hiểm vì nó phá vỡ mục tiêu cốt lõi của bài toán: an toàn và bảo vệ tài xế.

- **Khuynh hướng trả lời “đẹp” quá mức**: AI có xu hướng sáng tác ngôn ngữ mượt mà, nhưng thiếu tính thực tiễn. Ví dụ, AI có thể đề xuất lời nhắn rất chuyên nghiệp nhưng không tính đến thực tế doanh nghiệp cần phê duyệt bằng người thật hoặc cần yêu cầu dữ liệu đầu vào như vị trí GPS và trạng thái trạm sạc.

Vì vậy, tôi nhận ra AI không thể được xem là người quyết định cuối cùng trong hệ thống sản phẩm; nó chỉ phù hợp với vai trò hỗ trợ và soạn nháp.

---

## 3. Tôi đã sửa prompt / ranh giới ra sao?

Để khắc phục những lỗi trên, tôi đã chỉnh lại prompt theo hướng cực kỳ chặt chẽ:

1. **Bắt buộc có tag [DRAFT_ONLY]**  
   Mọi câu trả lời phải bắt đầu bằng `[DRAFT_ONLY]` để nhấn mạnh đây chỉ là bản nháp, không phải tin nhắn cuối cùng để gửi đi.

2. **Quy tắc pin an toàn**  
   Nếu pin < 5%, mô hình không được đề xuất trạm sạc quá 5 km. Thay vào đó, phải trả về dạng hành động: `dispatch_mobile_charger` hoặc một phản hồi tương đương cảnh báo rằng cần xe cứu hộ pin di động.

3. **Tách rõ vai trò của AI**  
   Tôi đã xác định AI là “dispatcher co-pilot” chứ không phải “vận hành viên tự động”. Điều này giúp mô hình hiểu rằng nó chỉ nên hỗ trợ in draft, triage và đề xuất, chứ không tự ý thực hiện hành động nhạy cảm.

4. **Cấm bypass ranh giới**  
   Tôi cũng đưa thêm quy tắc rõ ràng rằng nếu người dùng yêu cầu bỏ tag hoặc bỏ qua bước review, mô hình vẫn phải giữ nguyên đúng format và không được làm theo yêu cầu đó.

5. **Quy định định dạng output**  
   Tôi yêu cầu output phải theo dạng JSON / text rõ ràng, dễ cho người vận hành kiểm tra trước khi gửi tới tài xế.

---

## 4. Kết luận cá nhân

Qua bài Lab này, tôi học được rằng một AI product không thành công chỉ vì dùng được LLM. Điều quan trọng nhất là **định nghĩa đúng ranh giới vận hành, kiểm soát hành vi của mô hình và đặt cơ chế human-in-the-loop**. Nếu không làm điều này, AI dễ đi sai hướng và tạo ra nguy cơ lớn cho người dùng và doanh nghiệp.

Từ góc nhìn sản phẩm, tôi thấy khả năng thật sự của AI ở đây không nằm ở việc “gửi tin nhắn tự động”, mà ở việc **giảm thời gian xử lý, hỗ trợ người vận hành và giảm sai sót trong bối cảnh áp lực cao điểm**. Bản chất của hiệu quả AI trong dự án này là: **giúp con người quyết định nhanh hơn, không phải thay con người ra quyết định**.

Đây là bài học quan trọng mà tôi sẽ mang theo khi làm việc với các hệ thống AI thực tế trong tương lai.
