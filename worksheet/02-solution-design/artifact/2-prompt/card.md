---
artifact: 2 — Lớp chỉ dẫn AI
bai-tap: 2 — Thiết kế giải pháp
demo: ./demo.md
---

# card.md — Lớp chỉ dẫn AI

**Tình huống xử lý**: F-03 — Lóng số lớn dispute 10x
Xem `../../1-map-and-format.md` Phần A.

**Câu user mẫu**: `mua macbook 3 củ rưỡi trả góp tháng này`
**Lỗi cần chặn**: LLM "đoán" 1 trong 2 nghĩa của "củ rưỡi" thay vì liệt kê 2 nghĩa và để user chọn.

---

## 1. Giải pháp là gì?

System prompt **rule-based** buộc model trả về JSON structured (không phải free text) khi parse câu chứa lóng tiền VN. Nếu detect ambiguity (lóng + số mơ hồ + outlier >5tr), output bắt buộc phải có field `confidence: "ambiguous"` + `options: [...]` với 2-3 cách hiểu. Lớp UIUX render JSON này thành confirmation card. Đây là cách dùng prompt để **ngăn từ gốc** — không cho model output 1 con số duy nhất khi câu nhập có nhiều nghĩa hợp lý.

---

## 2. Vì sao sửa ở lớp chỉ dẫn AI?

- AI đang trả lời quá tự tin khi thiếu nguồn — LLM Vietnamese-trained có xu hướng pick nghĩa phổ biến nhất ("củ" = 1 triệu) mà không flag uncertainty.
- AI cần luật rõ: khi nào trả số trực tiếp, khi nào liệt kê options, khi nào hỏi lại.
- Có thể sửa nhanh bằng prompt — không cần retrain model, không cần ship code mới cho mobile app, chỉ cần update system message.
- Là **lớp ngăn đầu tiên** trong defense-in-depth: nếu prompt fix tốt, lớp UIUX có thể đơn giản hơn (chỉ render output, không cần re-classify).

**Hành động phòng vệ chính**:

- [x] Ngăn câu trả lời sai ngay từ đầu (rule "never silent save khi có lóng")
- [x] Bắt buộc nêu nguồn khi nói về thông tin quan trọng (liệt kê 2-3 cách hiểu thay vì pick 1)
- [ ] Từ chối trả lời khi thiếu căn cứ (không áp dụng — AI vẫn parse, chỉ là parse có flag uncertainty)
- [ ] Chuyển người thật khi vượt phạm vi (không áp dụng cho F-03 — không cần escalate)

---

## 3. Demo nằm ở đâu?

**File demo**: [`demo.md`](./demo.md)

Demo cần có:

- **System prompt** đầy đủ với 5 rule chính (whitelist lóng VN, output structure, outlier rule, pressure rule, audit rule).
- **JSON output schema** cho 2 trạng thái: `confidence: "clear"` (số rõ) và `confidence: "ambiguous"` (cần user chọn).
- **3 ví dụ kiểm tra**: input "3 củ rưỡi" (ambiguous), input "50k cafe" (clear), input "ghi đại 200 đang lái xe" (pressure + clear-ish).
- **Bảng thử lại** với 4-5 case từ Bài 1 (F-02, F-03, F-04, F-05, F-07).

---

## 4. Tác dụng phụ

**Có thể gây vấn đề gì?**

1. **Over-flagging** — model có thể flag mọi câu có "k" hoặc "đ" là ambiguous, kể cả khi rõ ràng → user khó chịu vì phải confirm liên tục.
2. **Hallucinated options** — model tự bịa ra "cách hiểu thứ 3" không có trong whitelist (vd: "3 củ rưỡi = 350.000đ").
3. **JSON malformed** — model trả về JSON sai format → app crash hoặc fallback sang silent save (worse).
4. **Latency** — output JSON dài hơn free text → response time tăng 200-500ms.

**Nhóm giảm vấn đề đó bằng cách nào?**

1. **Whitelist explicit** trong rule — chỉ flag ambiguous khi câu chứa từ trong list `[cành, củ, chai, lít, xị, loét, tờ đỏ]` HOẶC số >5tr. Câu "50k cafe" KHÔNG được trigger.
2. **Constrained options** — rule bắt buộc options chỉ chọn từ giá trị whitelist (3,5tr hoặc 35tr cho "củ rưỡi"), không cho model tự generate giá trị mới.
3. **JSON validation server-side** — nếu output không match schema → server retry với prompt rút gọn 1 lần; nếu fail 2 lần → push user vào State 4 (refuse silent save) thay vì save sai.
4. **Streaming partial output** — UI hiển thị "Đang phân tích..." trong 1s đầu, không chờ full response.

---

## 5. Checklist trước khi nộp

- [x] Luật viết đủ cụ thể để AI làm theo (5 rule có ví dụ cụ thể, có whitelist).
- [x] Có mẫu câu khi AI không có đủ thông tin (template "Tôi thấy 2 cách hiểu...").
- [x] Có ví dụ cho tình huống dễ sai (3 ví dụ + 5 case thử lại từ Bài 1).
- [x] Có thử lại bằng tình huống trong Bài 1 (bảng cuối demo.md).
- [x] Không dùng prompt như cách duy nhất — đã ghi rõ cần phối hợp lớp UIUX (render JSON) và lớp Architecture (validate JSON + audit log).

**Người phụ trách**: Đạt (đề xuất — vì Đạt đã làm risk map prompt-level ở Day 24).
