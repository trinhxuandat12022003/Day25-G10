---
artifact: 1 — Lớp giao diện
bai-tap: 2 — Thiết kế giải pháp
demo: ./demo.md
---

# card.md — Lớp giao diện

**Tình huống xử lý**: F-03 — Lóng số lớn dispute 10x
Xem `../../1-map-and-format.md` Phần A.

**Câu user mẫu**: `mua macbook 3 củ rưỡi trả góp tháng này`
**Lỗi cần chặn**: AI silent save 3.500.000đ (hoặc 35.000.000đ) mà không cho user chọn — chênh 10x trên giao dịch lớn.

---

## 1. Giải pháp là gì?

Khi AI parse câu nhập có lóng tiền VN (cành/củ/chai/lít/xị/tờ đỏ) hoặc số mơ hồ, giao diện **không** show một số duy nhất — mà hiện **confirmation card với 2-3 lựa chọn quy đổi** kèm gợi ý theo ngữ cảnh ("macbook" → gợi ý 35tr). User phải tap chọn 1 trong các option (hoặc "Sửa lại") trước khi DB ghi. Với giao dịch >5tr, thêm bước "Xác nhận giao dịch lớn" có lưu ảnh chụp màn hình để dispute sau.

---

## 2. Vì sao sửa ở lớp giao diện?

- Người dùng dễ tin câu trả lời của AI quá mức — đặc biệt khi AI nói trôi chảy nhưng số sai.
- Rủi ro xảy ra ở khoảnh khắc người dùng đọc câu trả lời — user nhập vội sau giao dịch, không tự cộng lại.
- Giao diện cần làm rõ: số nào AI chắc, số nào AI đoán giữa nhiều cách hiểu.
- Nếu prompt/dữ liệu vẫn sót lỗi (model parse sai), giao diện là **lớp chặn cuối** trước khi DB ghi.

**Hành động phòng vệ chính**:

- [x] Thông báo rõ giới hạn (badge "có 2 cách hiểu")
- [x] Phát hiện dấu hiệu thiếu nguồn / mơ hồ (outlier warning cho giao dịch >5tr)
- [ ] Chuyển người thật khi cần (không áp dụng cho F-03 — không cần escalate)
- [x] Giúp người dùng kiểm tra lại nguồn (raw input hiện ở đầu card để user đối chiếu lời mình vừa nói)

---

## 3. Demo nằm ở đâu?

**File demo**:
- [`demo.md`](./demo.md) — ASCII sketch source-of-truth (4 states)
- [`demo.html`](./demo.html) — Phiên bản HTML tĩnh, mở trực tiếp trên browser (CSS inline, không JS)

**Định dạng demo**: ASCII sketch (theo prompt `05a-ascii-ui-sketch.md`) + HTML tĩnh cho demo trực quan trong phản biện chéo.

- [x] Phác thảo màn hình
- [ ] Luồng màn hình (đã có trong 3-architecture/demo.md)
- [x] Bản HTML đơn giản
- [ ] Ảnh hoặc link prototype

**Thành phần cần có trong demo**:

- 4 states: Default (số rõ) · Ambiguous-Unit (case "3 củ rưỡi") · Outlier-Warning (giao dịch >5tr) · Refuse-Silent-Save (user ép "ghi nhanh").
- Raw input hiện ở đầu card → user thấy AI parse từ câu gì.
- Lựa chọn radio (○/●) cho mỗi cách hiểu + option "Đây là đơn vị khác".
- Nút "Sửa lại" để gõ số chính xác.
- Audit indicator: số phiên / thời gian / ai xác nhận.

---

## 4. Tác dụng phụ

**Có thể gây vấn đề gì?**

1. **Confirmation friction** — user vốn dùng app vì "ghi một câu là xong"; giờ phải tap chọn thêm 1 bước. Có thể gây churn nếu hỏi quá nhiều.
2. **Decision fatigue** — nếu 3-4 khoản trong 1 câu đều ambiguous, user phải confirm 3-4 lần liên tiếp.
3. **Anchor bias** — option đầu tiên hoặc option "highlighted" (●) có thể anchor user chọn theo dù sai.

**Nhóm giảm vấn đề đó bằng cách nào?**

1. **Chỉ confirm khi thật sự mơ hồ** — số có "k" rõ ràng (50k, 200k) thì save thẳng; chỉ trigger card khi gặp slang trong whitelist hoặc số >5tr.
2. **Gộp confirm** — khi 1 câu có ≥3 khoản ambiguous, hiện 1 card duy nhất với multi-row chọn cùng lúc (không split 3 màn).
3. **Không pre-highlight** — để cả 2-3 option ngang nhau (○), chỉ highlight option AI gợi ý dựa trên context ("macbook" → 35tr) nhưng đánh dấu rõ "Gợi ý theo ngữ cảnh, không phải chắc chắn".

---

## 5. Checklist trước khi nộp

- [x] Giải pháp gắn đúng với một rủi ro chính (F-03).
- [x] Demo nhìn vào là hiểu vấn đề được chặn ở đâu (ambiguous unit → confirmation card).
- [x] Có đủ trạng thái bình thường và trạng thái lỗi (4 states ở demo.md).
- [ ] Có cách chuyển sang người thật khi AI không nên tự xử lý — **không áp dụng cho F-03** (xem note ở mục 2; F-01 dự phòng mới cần escalation).
- [x] Câu chữ trong giao diện ngắn, không đổ hết trách nhiệm cho người dùng (tránh wording "AI có thể sai, bạn tự kiểm tra" — thay bằng "Có 2 cách hiểu, bạn muốn chọn cái nào?").

**Người phụ trách**: Dương (đề xuất — vì Dương đã đề propose F-03 trong diverge).
