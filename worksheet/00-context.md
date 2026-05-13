---
title: 00 — Bối cảnh sản phẩm của nhóm
section: Day 25 — dùng lại cho mọi cuộc trò chuyện với AI
format: Nhóm (Track 04 — tổng hợp từ Day 24 của dat/ + duong/)
time: Điền 5 phút đầu buổi
---

# 00-context.md — Bối cảnh sản phẩm của nhóm

**Track:** 04 — Trợ lý ghi chú và tổng hợp chi tiêu (AI Expense Assistant)

**Nguồn nội bộ đã dùng để chốt bối cảnh:**  
`day25/dat/01-risk-map.md`, `day25/dat/02-test-eval-plan.md`, `day25/duong/01-risk-map.md`, `day25/duong/02-test-eval-plan (1).md`.

---

## 1. Sản phẩm

- **Tên sản phẩm / bot:** AI Expense Assistant — ứng dụng quản lý chi tiêu cá nhân tích hợp AI (Track 04).
- **Sản phẩm giúp ai làm gì:** Ghi nhanh khoản chi bằng text hoặc giọng nói; trích xuất số tiền, hạng mục, thời gian; lưu DB; tổng hợp theo tuần/tháng; có thể đưa insight xu hướng. (Mở rộng tương lai: import ảnh/file sao kê, đọc báo cáo cuối tháng do AI viết — **chưa** coi là trong phạm vi test v0 hiện tại.)
- **Người dùng gặp sản phẩm ở đâu:** Ứ dụng trên điện thoại (ghi chép vội sau giao dịch).
- **Giai đoạn hiện tại:** Đang thử nghiệm / chuẩn bị ra mắt tính năng nhập tự nhiên (natural language).

---

## 2. Phạm vi

**AI được làm gì**

- Trích xuất thực thể từ câu nói tiếng Việt: số tiền, gợi ý hạng mục, phương thức (tiền mặt / chuyển khoản / ví), bối cảnh thời gian ngắn.
- Cộng tổng nhiều khoản trong một lượt nhập khi dữ liệu đủ rõ.
- Hiển thị thẻ / màn hình xác nhận trước khi ghi CSDL (kỳ vọng thiết kế an toàn từ Day 24).
- Từ chối hoặc giới hạn khi câu hỏi vượt phạm vi (ví dụ: tư vấn đầu tư chuyên sâu).

**AI không được làm gì**

- Tự động lưu số tiền khi đơn vị mơ hồ, tiếng lóng không chắc, hoặc số tiền bất thường so với ngữ cảnh — mà không hỏi lại / không xác nhận.
- "Đoán" số tiền rồi ghi DB khi user dùng từ đa nghĩa (ví dụ: "lít" trong ngữ cảnh tiền).
- Đưa lời khuyên đầu tư / tài chính cá nhân sâu (ví dụ: rút tiết kiệm mua tài sản rủi ro cao) như một chuyên gia.
- Trong phạm vi eval v0: không cam kết OCR sao kê, không cam kết kết nối API ngân hàng thật, không đánh giá bộ nhớ đa phiên dài hạn.

**Vì sao có giới hạn này**

- Sai một đơn vị tiền (50đ thay vì 50.000đ) làm lệch số dư và báo cáo — hậu quả tài chính và niềm tin người dùng (mức độ cao theo risk map nhóm).
- Tiếng Việt đời thường: viết tắt, lóng ("cành", "củ"), số có dấu chấm/phẩy dễ nhầm — cần ranh giới rõ giữa "suy luận" và "xác nhận với người".

---

## 3. Người dùng

- **Là ai:** Người lớn 22–35, nhân viên văn phòng hoặc người dùng phổ thông; thường nhập liệu vội, dùng lóng/quen miệng ("50k", "cành", "củ"). Có persona tham chiếu: người đang thắt chặt chi tiêu để mục tiêu dài hạn (ví dụ: tiết kiệm mua nhà).
- **Họ hỏi AI khi nào:** Ngay sau khi chi tiêu (đi bộ, lái xe, bận việc); muốn "ghi một câu là xong".
- **Họ cần quyết định gì sau khi hỏi AI:** Có lưu giao dịch này không; số tiền cuối cùng là bao nhiêu; hạng mục có đúng không.
- **Khi nào họ dễ bị tổn thương / dễ hiểu sai:** Khi AI trả lời trôi chảy, có tổng số và phân loại nhưng sai số học; khi không có bước xác nhận (silent save); khi insight cuối tháng nghe "hợp lý" nhưng vô căn cứ do thiếu ngữ cảnh.
- **Họ thường tin AI đến mức nào:** Xu hướng tin vào tổng tiền và số dư hiển thị; rủi ro "cognitive offloading" — lười đối chiếu tay nếu app luôn có vẻ đúng.

---

## 4. Bối cảnh ngành

- **Sự cố tương tự đã từng xảy ra:** Các sản phẩm chatbot / trợ lý gợi ý sai thông tin tài chính hoặc sai phép tính đã được thảo luận rộng trong ngành GenAI; nhóm neo rủi ro vào **lỗi cụ thể của track**: hallucination phép tính, NER tiền tệ với lóng Việt Nam, misclassification từ khóa đa nghĩa, insight ảo trên dữ liệu thiếu.
- **Quy định hoặc ràng buộc liên quan:** Dữ liệu tài chính cá nhân — cần đúng số, có dấu vết (log), và ranh giới không tư vấn đầu tư chuyên sâu; tuân thủ chính sách nội bộ sản phẩm khi triển khai thật.
- **Nguồn chính thức nên ưu tiên (khi triển khai thật):** Sao kê / ví / người dùng xác nhận; không coi output LLM là sổ sách kế toán.

---

## 5. Ghi chú thêm

- **Câu hỏi an toàn trung tâm (tổng hợp Day 24):** Khi user nhập nhiều khoản với định dạng số khác nhau (kể cả lóng), AI có trích xuất đúng từng thực thể và giữ **toàn vẹn phép cộng** không? Với tiếng lóng / số mơ hồ: có **xác nhận trước khi lưu** thay vì silent save không?
- **Rủi ro đã thống nhất cần nhớ khi thiết kế test Day 25:**  
  (1) Sai tổng / sai thực thể trong một câu dài;  
  (2) Phân loại sai do từ đa nghĩa;  
  (3) Sai đơn vị / quy đổi lóng ("cành", "củ", "2 lít"…);  
  (4) Insight báo cáo sai khi tháng có biến động bất thường;  
  (5) Cập nhật / ghi đè khoản chi khi user sửa miệng — eval v0 có thể chưa cover hết, cần ghi rõ khi mở rộng test set.
- **Hai nhánh test v0 đã có sẵn:** bộ ca của **dat** (normal / critical / edge / pressure / out-of-scope đầu tư) và bộ **duong** (TC-01–06, nhấn mạnh xác nhận UI + ambiguous "lít" + outlier 15 triệu bánh mì). Day 25: mở rộng → hội tụ → chốt 10–15 ca trong `01-test-set-review/`.

---

## Cách dùng

```text
1. Mở công cụ AI phù hợp với bước đang làm.
2. Đưa toàn bộ nội dung file này vào đầu cuộc trò chuyện.
3. Chọn prompt tham khảo từ thư mục ../prompts/ và chỉnh lại nếu cần.
4. Đọc lại bản nháp AI tạo ra.
5. Sửa lại cho đúng bối cảnh nhóm.
6. Lưu kết quả vào đúng file trong worksheet/.
```
