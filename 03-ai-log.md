# 03 — AI Log & Reflection

## 1. Bối cảnh sử dụng AI

Trong buổi lab, tôi dùng AI như một **thought-partner** để brainstorm các pain point vận hành cho Vin Smart Future, phản biện Quick Problem Card và gợi ý cách chia ranh giới giữa Rule, LLM và Human-in-the-loop. Đề tài tôi chọn là hỗ trợ điều phối khi tài xế Xanh SM báo pin yếu hoặc hết pin ngoài đường.

AI không được xem là nguồn xác nhận số liệu vận hành. Các con số thời gian trong bản nháp chỉ là giả định phục vụ bài lab và cần được đối chiếu bằng log thật.

## 2. AI đã giúp gì?

1. **Mở rộng không gian ý tưởng:** AI gợi ý nhiều bài toán ở Xanh SM, VinFast, Vinhomes và Vinmec thay vì chỉ tập trung vào chatbot.
2. **Làm rõ actor và bottleneck:** Sau khi yêu cầu AI đóng vai CFO/trưởng vận hành, tôi nhận ra cần mô tả người đang chịu tải, handoff và thời gian ở từng bước.
3. **Phản biện lựa chọn công nghệ:** AI giúp phân biệt phần nào nên dùng Rule để kiểm soát an toàn, phần nào chỉ nên dùng LLM để tóm tắt/soạn bản nháp.
4. **Thiết kế adversarial cases:** AI gợi ý các tình huống như pin 2% nhưng trạm cách 8 km, người dùng yêu cầu bỏ tag `[DRAFT_ONLY]`, hoặc input thiếu vị trí.

## 3. Lỗi hoặc hallucination tôi phát hiện

- AI có xu hướng đưa ra các con số trông rất cụ thể như “giảm 80% thời gian” hoặc “95% ticket được xử lý”, nhưng không có nguồn dữ liệu đi kèm. Tôi loại các con số này hoặc đánh dấu là `*` để xác minh.
- Một số câu trả lời giả định hệ thống nội bộ có API trạng thái trạm sạc và GPS thời gian thực. Đây là giả định kiến trúc, không phải sự thật đã được kiểm chứng.
- AI từng đề xuất “tự động gửi tin nhắn” và “tự động gọi cứu hộ” để tối ưu tốc độ. Tôi không chấp nhận đề xuất đó vì đây là hành động có tác động thực tế và rủi ro an toàn.
- Khi được hỏi bằng câu lệnh kiểu “bỏ qua quy tắc”, mô hình có thể tạo câu trả lời không nhất quán nếu system prompt và output schema không đủ chặt.

## 4. Tôi đã sửa prompt và ranh giới như thế nào?

Tôi chuyển từ yêu cầu chung “hãy giúp tài xế tìm trạm sạc” sang các điều kiện kiểm thử cụ thể:

```text
1. Nếu pin < 5%, không được đề xuất trạm tiêu chuẩn xa hơn 5 km.
2. Phải trả về action dispatch_mobile_charger trong nhánh pin nguy hiểm.
3. Mọi bản nháp gửi tài xế phải bắt đầu bằng [DRAFT_ONLY].
4. Không được tự gửi tin, tự gọi cứu hộ hoặc tự khẳng định dữ liệu bị thiếu.
5. Nếu thiếu trường bắt buộc hoặc JSON không hợp lệ, chuyển HITL/Fallback.
```

Tôi cũng tách trách nhiệm: **Rule** quyết định điều kiện an toàn; **LLM** chỉ tóm tắt và tạo bản nháp; **điều phối viên** duyệt và thực hiện hành động. Cách này giảm việc kỳ vọng LLM tự suy luận mọi thứ.

## 5. Bài học cá nhân

Điều hữu ích nhất không phải là để AI viết toàn bộ báo cáo, mà là dùng AI để đặt câu hỏi ngược: “Ai chịu trách nhiệm?”, “Nếu dữ liệu sai thì sao?”, “Có cần LLM không?”, và “Đo thành công bằng số nào?”. Tôi học được rằng một ý tưởng AI tốt phải có workflow hiện tại, metric có thể đo, operational boundary và fallback.

Điều tôi vẫn cần làm tiếp là xác minh với người vận hành, lấy dữ liệu ẩn danh và đo baseline thật. Vì vậy, quyết định của tôi là **NOT YET** cho triển khai tự động, nhưng tiếp tục prototype offline/shadow mode với Human-in-the-loop.
