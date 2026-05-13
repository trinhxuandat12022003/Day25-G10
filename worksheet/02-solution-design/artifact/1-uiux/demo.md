---
artifact: 1 — Demo giao diện
format: ASCII sketch — 4 states cho F-03 (lóng số lớn dispute 10x)
---

# demo.md — Demo giao diện cho F-03

Demo gồm **4 states** cho confirmation card khi user nhập câu có lóng tiền VN hoặc số mơ hồ. Mỗi state thiết kế cho 1 risk pattern khác nhau — cùng đỡ 1 rủi ro chính (sai 10x), khác nhau ở mức confidence của AI và mức urgency của user.

> 📄 **Phiên bản HTML có sẵn**: [`./demo.html`](./demo.html) — render 4 state trên browser với CSS đầy đủ, dùng cho phản biện chéo / pitch. File ASCII bên dưới là **source-of-truth**; HTML chỉ là rendering bổ sung.

---

## 1. Màn hình chính — 4 states

### State 1 — DEFAULT (số rõ, không cần confirm chi tiết)

Khi câu nhập có đơn vị rõ ("50k", "200.000đ"), không cần card mở rộng — chỉ confirmation rút gọn 1 dòng.

```text
╔═══════════════════════════════════╗
║ ✓ Đã ghi                          ║
║ 50.000đ · Cà phê · Hôm nay 8:30   ║
║                                   ║
║ [Sửa] [Xem chi tiết]              ║
╚═══════════════════════════════════╝
```

**Annotations**:
- Wording "Đã ghi" → user biết DB đã commit.
- Không hỏi gì thêm vì không có ambiguity.
- Nút "Sửa" vẫn luôn có — user có 60s undo dễ dàng.

---

### State 2 — AMBIGUOUS UNIT (case F-03: "3 củ rưỡi")

State chính cho F-03. User nhập `mua macbook 3 củ rưỡi trả góp tháng này` — AI **không** silent save, hiện card với 2-3 lựa chọn.

```text
╔═══════════════════════════════════╗
║ Bạn vừa nói:                      ║
║ "macbook 3 củ rưỡi trả góp"       ║
║ ──────────────────────────────── ║
║ ⚠ "Củ rưỡi" có nhiều cách hiểu   ║
║                                   ║
║ ○ 3.500.000đ  (3,5 triệu)         ║
║ ● 35.000.000đ (35 triệu)          ║
║   Gợi ý theo ngữ cảnh "macbook"  ║
║ ○ Đây là đơn vị khác — sửa số    ║
║                                   ║
║ Hạng mục:                         ║
║ [Đồ điện tử ▼]                    ║
║ Phương thức:                      ║
║ [Trả góp ▼]                       ║
║ ──────────────────────────────── ║
║ [Xác nhận] [Sửa lại] [Huỷ]       ║
╚═══════════════════════════════════╝
```

**Annotations**:
- **Raw input ở đầu** → user thấy lại câu mình vừa nói, đối chiếu được. Không phải "AI hiểu là...".
- **3 option radio** — option ② có tag "Gợi ý theo ngữ cảnh" nhưng vẫn là radio ngang hàng (○), không phải tick mặc định. Tránh anchor bias.
- **Hạng mục + Phương thức pre-fill** từ keyword ("macbook" → Đồ điện tử, "trả góp" → Trả góp) — giảm friction cho phần không ambiguous.
- **Nút Xác nhận disabled** đến khi user chọn 1 radio — không cho ấn nhanh skip.

---

### State 3 — OUTLIER WARNING (giao dịch lớn >5tr)

Sau khi user chọn `35.000.000đ` ở State 2, thêm 1 bước outlier — vì giao dịch ≥5tr là bất thường so với chi tiêu trung bình tháng.

```text
╔═══════════════════════════════════╗
║ ⚠ Xác nhận giao dịch lớn          ║
║ ──────────────────────────────── ║
║ Số tiền:    35.000.000đ           ║
║ Hạng mục:   Đồ điện tử            ║
║ Phương thức: Trả góp              ║
║ Thời gian:  Hôm nay 14:23         ║
║                                   ║
║ Trung bình ngày bạn chi: 230.000đ ║
║ Khoản này gấp ~150x trung bình.   ║
║                                   ║
║ ☐ Tôi đã kiểm tra số tiền đúng    ║
║                                   ║
║ [Xác nhận ghi] [Sửa]              ║
╚═══════════════════════════════════╝
```

**Annotations**:
- **Checkbox bắt buộc tick** trước khi "Xác nhận ghi" sáng — friction đủ để user dừng 1 giây.
- **So sánh "gấp ~150x trung bình"** → cung cấp anchor để user kiểm tra. Không nói "AI nghi ngờ", chỉ trình bày số liệu.
- **Audit auto-snapshot**: khi user tick + xác nhận, app chụp full state này lưu vào audit log (xem 3-architecture/demo.md) → dispute resolution sau này có evidence.

---

### State 4 — REFUSE SILENT SAVE (user ép "ghi nhanh đang lái xe")

Áp dụng cho F-04 (pressure trap) — neo cùng family với F-03. User input `nhanh lên đang lái xe ghi đại 3 củ rưỡi đỗ phải hỏi`. AI vẫn show card rút gọn, **không** save thẳng.

```text
╔═══════════════════════════════════╗
║ Đang ghi tạm — chưa lưu           ║
║ ──────────────────────────────── ║
║ "3 củ rưỡi" — có 2 cách hiểu      ║
║                                   ║
║ ⓵ 3.500.000đ                      ║
║ ⓶ 35.000.000đ                     ║
║                                   ║
║ ● Mình giữ nháp 5 phút.           ║
║   Khi dừng xe, bấm 1 nút để chọn.║
║                                   ║
║ [Để nháp 5 phút] [Chọn ⓵ chọn ⓶] ║
╚═══════════════════════════════════╝
```

**Annotations**:
- **"Đang ghi tạm — chưa lưu"** → user vẫn nghe được tóm tắt giọng (voice playback) khi đang lái xe, không cần nhìn màn hình ngay.
- **Lựa chọn voice command 1-touch**: user nói "một" hoặc "hai" để chọn — không cần thả vô lăng.
- **Mặc định "Để nháp 5 phút"** → nếu user không phản hồi, hết 5 phút app gửi push notification để user xử lý sau, **không** auto-commit option nào.
- Wording không đổ lỗi user ("đừng nguy hiểm khi lái xe") — chỉ cho safer default.

---

## 2. Trạng thái cần minh họa

| Trạng thái | Người dùng thấy gì? | Người dùng làm gì tiếp? |
|---|---|---|
| Default (số rõ) | Confirmation rút gọn 1 dòng "Đã ghi 50k cà phê" | Có thể bấm Sửa trong 60s nếu cần |
| Ambiguous unit | Card với 2-3 lựa chọn quy đổi + gợi ý theo context | Chọn 1 radio + bấm Xác nhận |
| Outlier (>5tr) | Card so sánh với trung bình + checkbox "Tôi đã kiểm tra" | Tick checkbox + Xác nhận hoặc Sửa |
| Refuse silent save (pressure) | Card "Đang ghi tạm — chưa lưu" + voice command | Để nháp 5 phút hoặc nói "một"/"hai" |

---

## 3. Ghi chú cho từng thành phần

- **Raw input box (top)**: hiện lại nguyên văn câu user nói/gõ. Vị trí trên cùng. Vai trò: user đối chiếu được "AI parse từ câu gì", phát hiện ngay nếu speech-to-text sai.
- **Radio options** (State 2): 3 lựa chọn ngang hàng. Option có "Gợi ý theo ngữ cảnh" hiển thị tag nhỏ nhưng không pre-selected. Tránh anchor bias.
- **Comparison anchor** (State 3): "Trung bình ngày bạn chi: 230k. Khoản này gấp 150x". Lấy từ DB user của 30 ngày gần nhất. Mục đích: cung cấp anchor số liệu thay vì AI nói "tôi nghi ngờ".
- **Voice command** (State 4): tích hợp với speech-to-text — user nói "một"/"hai"/"để sau" để chọn không cần touch.
- **Audit indicator** (footer mọi state): ID phiên xác nhận + timestamp + ai xác nhận (nếu multi-user). Mục đích: dispute resolution → user đối chứng được "tôi đã xác nhận 35tr lúc 14:23".

---

## 4. Kiểm tra nhanh

- [x] Nhìn vào demo là hiểu rủi ro đang được chặn ở đâu — confirmation card chặn silent save 10x.
- [x] Có trạng thái khi AI không có đủ thông tin — State 2 (ambiguous) và State 4 (pressure).
- [ ] Có cách chuyển sang người thật — **không áp dụng cho F-03**. F-01 (dự phòng) mới có escalation flow.
- [x] Câu chữ đủ ngắn để đặt trên màn hình thật — max 6-8 chữ/dòng cho phần action, max 2 dòng cho phần thông báo.

---

## 5. User testing scenarios (đề xuất verify sau)

| Scenario | User profile | Task | Pass criteria |
|---|---|---|---|
| A — Macbook 3 củ rưỡi | User mua đồ điện tử, dùng app lần đầu | Nhập câu macbook 3 củ rưỡi | Notice 3 options trong <3s, chọn đúng 35tr theo context |
| B — Bún 2 lít | User văn phòng, ăn trưa với khách | Nhập "2 lít trà sữa hết 2 lít momo" | Phát hiện 2 nghĩa "lít" khác nhau, chọn đúng cả 2 |
| C — Pressure đang lái xe | Tài xế Grab, voice input | Nói "ghi đại 3 củ rưỡi" trong xe | Không touch màn hình; voice command 1-từ phải hoạt động |
| D — Outlier 15tr ăn sáng | User typo "15 triệu" thay vì "15k" | Nhập "ăn sáng bánh mì 15 triệu" | Flag outlier trong State 3, user sửa thành 15k trong <10s |
