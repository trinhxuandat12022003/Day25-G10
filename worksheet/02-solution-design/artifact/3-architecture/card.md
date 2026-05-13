---
artifact: 3 — Lớp kiến trúc dữ liệu
bai-tap: 2 — Thiết kế giải pháp
demo: ./demo.md
---

# card.md — Lớp kiến trúc dữ liệu

**Tình huống xử lý**: F-03 — Lóng số lớn dispute 10x
Xem `../../1-map-and-format.md` Phần A.

**Câu user mẫu**: `mua macbook 3 củ rưỡi trả góp tháng này`
**Lỗi cần chặn**: Dù prompt fix tốt, model vẫn có thể output ambiguous/clear sai. Cần lớp data infra để (a) validate output trước khi commit, (b) giữ dữ liệu trong staging cho tới khi user xác nhận, (c) lưu audit log cho dispute resolution.

---

## 1. Giải pháp là gì?

Thêm 4 components vào pipeline parser:

1. **VN Slang Dictionary** — bảng quy đổi lóng tiền VN làm source-of-truth (single dictionary, version control). LLM lấy options từ đây, không tự generate.
2. **Ambiguity Detector** — middleware giữa LLM output và DB: validate JSON schema, double-check ambiguity rules (whitelist match + outlier threshold) ngoài LLM, override nếu LLM miss flag.
3. **Confirmation Queue** — staging table trong DB. Mỗi transaction `confidence=ambiguous` được hold ở queue 24h, KHÔNG commit vào bảng `expenses` cho tới khi user xác nhận. Sau 24h không xử lý → push notification, không auto-commit.
4. **Audit Log + Monitoring** — bảng append-only lưu mỗi parse event: raw_input + parsed_options + user_choice + timestamp + session_id. Telemetry track rate of ambiguity + override 10x → cảnh báo drift của LLM.

---

## 2. Vì sao sửa ở lớp kiến trúc dữ liệu?

- Nguyên nhân chính là **AI đoán khi không biết** + **thiếu confirmation step** — chỉ prompt rule không đủ vì LLM có thể bypass rule (jailbreak, hallucination, schema malformed).
- AI đang phải tự nhớ bảng quy đổi lóng — nên đưa ra ngoài thành dictionary maintained by human, version control, có thể audit và rollback khi sai.
- Cần kiểm tra dữ liệu **giữa lúc LLM output và lúc commit DB** — đây là chỗ duy nhất chặn được nếu prompt + UI đều fail.
- Cần ghi lại để nhóm biết: bao nhiêu % parse là ambiguous, user chọn option nào nhiều nhất, có drift hay không (vd nếu tuần này tự dưng 50% case ambiguous → model có vấn đề).

**Hành động phòng vệ chính**:

- [x] Ngăn lỗi bằng nguồn dữ liệu đúng (VN Slang Dict là source-of-truth)
- [x] Phát hiện khi nguồn thiếu hoặc lỗi (Ambiguity Detector validate JSON + override khi LLM miss flag)
- [x] Khắc phục bằng cách chuyển sang người thật — **adapted cho F-03**: Confirmation Queue giữ dữ liệu, KHÔNG silent save; với case dispute sau, audit log cho phép rollback theo timestamp.
- [x] Ghi lại lỗi để cải thiện sau (Audit Log + Monitoring telemetry)

---

## 3. Demo nằm ở đâu?

**File demo**: [`demo.md`](./demo.md)

Demo cần có:

- **Sơ đồ data flow ASCII** từ input → parser → detector → queue → DB, với 2 branch (clear vs ambiguous).
- **Bảng VN Slang Dictionary** (schema + sample rows).
- **Bảng Confirmation Queue** (schema + state transitions).
- **Bảng Audit Log** (schema + ví dụ log entry cho F-03 case).
- **Telemetry dashboard mock** với 3 metric chính (ambiguity_rate, dispute_rate, override_10x_rate).
- **Xử lý failure**: LLM down, JSON malformed, queue overflow, audit log lock.

---

## 4. Tác dụng phụ

**Có thể gây vấn đề gì?**

1. **Latency tăng** — pipeline có thêm 3 hop (detector validate + queue insert + audit append) → +50-150ms cho mỗi parse. Với user nhập vội, đây là tradeoff đáng cân nhắc.
2. **Storage cost** — audit log append-only → growth linearly. Sau 1 năm có thể 10-20GB cho 100k users.
3. **Dictionary maintenance** — VN slang thay đổi theo vùng/thời gian. Ai duy trì? Khi nào update? Có process review không?
4. **Queue starvation** — nếu user không xác nhận pending transaction, queue ngày càng lớn → memory issue.
5. **Privacy** — audit log lưu raw_input có thể chứa thông tin nhạy cảm (vd "quà cho bồ" — trùng F-12).

**Nhóm giảm vấn đề đó bằng cách nào?**

1. **Async pipeline** — detector + audit log chạy trong background worker, không block response. User thấy result trong <300ms; commit DB và log diễn ra trong 1-2s sau.
2. **Audit log retention 90 ngày** — sau 90 ngày, archive sang cold storage hoặc xóa raw_input (giữ metadata). Tuân thủ NĐ 13/2023 về quyền xóa dữ liệu.
3. **Dictionary owner**: 1 PM/researcher review dictionary theo quý; community contribution qua PR; mỗi update có changelog + A/B test trên 10% users trước khi roll out 100%.
4. **Queue TTL 24h** — pending transaction không xác nhận trong 24h sẽ auto-cancel (KHÔNG commit) + push notification. User mở app thấy "Bạn có 3 khoản chưa xác nhận từ hôm qua".
5. **Encrypt audit log at rest** — raw_input encrypt với key per-user; user xóa account → key destroyed → raw_input không recover được.

---

## 5. Checklist trước khi nộp

- [x] Sơ đồ cho thấy dữ liệu đi từ đâu đến đâu (input → parser → detector → queue → DB, có 2 branch clear/ambiguous).
- [x] Có bước kiểm tra nguồn trước khi AI trả lời (Ambiguity Detector chạy giữa LLM và DB).
- [x] Có cách xử lý khi không có dữ liệu (LLM down → fallback retry 1 lần → push State 4 refuse silent save).
- [ ] Có cách chuyển sang người thật với tình huống rủi ro cao — **không áp dụng cho F-03**. F-01 dự phòng mới cần escalation channel.
- [x] Có cách biết lỗi này có đang lặp lại không (Monitoring telemetry: ambiguity_rate / dispute_rate / override_10x_rate).

**Người phụ trách**: Tùng (đề xuất — vì Tùng đã làm cross-source verification + data review ở Day 24).
