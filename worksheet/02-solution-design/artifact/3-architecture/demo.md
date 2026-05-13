---
artifact: 3 — Demo kiến trúc dữ liệu
format: ASCII data flow + bảng components + telemetry mock
---

# demo.md — Demo kiến trúc dữ liệu cho F-03

Data flow + components để (a) validate LLM output, (b) hold dữ liệu trong staging cho tới khi user xác nhận, (c) lưu audit log cho dispute resolution.

---

## 1. Sơ đồ data flow

```text
                  ┌───────────────────────────────────────────┐
                  │            USER INPUT (text/voice)         │
                  │   "mua macbook 3 củ rưỡi trả góp tháng này"│
                  └─────────────────────┬─────────────────────┘
                                        │
                                        ▼
                  ┌───────────────────────────────────────────┐
                  │   LLM PARSER (System prompt từ 2-prompt)  │
                  │   - Input: raw text                       │
                  │   - Output: JSON theo schema              │
                  └─────────────────────┬─────────────────────┘
                                        │
                                        ▼
                  ┌───────────────────────────────────────────┐
                  │   AMBIGUITY DETECTOR (middleware)         │
                  │   1. Validate JSON schema (Zod/Pydantic)  │
                  │   2. Cross-check whitelist từ Slang Dict  │
                  │   3. Apply outlier rule (≥5tr)            │
                  │   4. Apply pressure rule (cue detection)  │
                  │   5. Override LLM nếu miss flag           │
                  └────────┬─────────────────────┬────────────┘
                           │                     │
                  confidence=clear      confidence=ambiguous
                           │                     │
                           ▼                     ▼
              ┌─────────────────────┐  ┌─────────────────────┐
              │  DIRECT CONFIRM     │  │  CONFIRMATION QUEUE │
              │  (UIUX State 1)     │  │  (staging table)    │
              │  1-tap save         │  │  Hold 24h TTL       │
              └──────────┬──────────┘  └──────────┬──────────┘
                         │                        │
                         │             ┌──────────┴──────────┐
                         │             │                     │
                         │     user xác nhận           24h timeout
                         │             │                     │
                         │             ▼                     ▼
                         │   ┌─────────────────┐   ┌─────────────────┐
                         │   │ Commit chosen   │   │ AUTO-CANCEL     │
                         │   │ option to DB    │   │ + push notify   │
                         │   └────────┬────────┘   └─────────────────┘
                         │            │
                         └────────────┴────────┐
                                               │
                                               ▼
                                ┌──────────────────────────┐
                                │       expenses DB        │
                                │  (canonical source)      │
                                └──────────────┬───────────┘
                                               │
                                               │ (parallel async)
                                               ▼
                                ┌──────────────────────────┐
                                │       AUDIT LOG          │
                                │  append-only, 90d        │
                                │  encrypted at rest       │
                                └──────────────┬───────────┘
                                               │
                                               ▼
                                ┌──────────────────────────┐
                                │   MONITORING / TELEMETRY │
                                │  - ambiguity_rate        │
                                │  - dispute_rate          │
                                │  - override_10x_rate     │
                                └──────────────────────────┘

DICTIONARY (read-only, version-controlled):
                                ┌──────────────────────────┐
                                │   VN SLANG DICTIONARY    │
                                │  (source of truth cho    │
                                │   options + alt-meaning) │
                                └──────────────────────────┘
                                        ▲          ▲
                                        │          │
                          (LLM Parser đọc)  (Detector cross-check)
```

---

## 2. Thành phần chính

| Thành phần | Nhận gì? | Làm gì? | Trả ra gì? |
|---|---|---|---|
| **LLM Parser** | Raw text từ user | Apply system prompt + bảng quy đổi + 5 rule | JSON theo schema (clear/ambiguous) |
| **Ambiguity Detector** | JSON từ LLM | (1) Validate schema, (2) cross-check whitelist với Slang Dict, (3) apply outlier ≥5tr, (4) override LLM nếu LLM miss flag (vd model nói "clear" nhưng raw_input có "củ" → detector force ambiguous) | JSON đã verified + flag corrections |
| **VN Slang Dictionary** | Slang token (cành/củ/chai/lít/xị/loét/tờ đỏ) | Lookup giá trị + alt meaning + region tag | `{value: 1000000, alt: "cây/quả", region: "all"}` |
| **Confirmation Queue** | Transaction confidence=ambiguous | Insert vào staging, set TTL 24h, gửi notification | pending_transaction_id |
| **Expense DB** | Transaction đã user xác nhận | Commit canonical record | success/fail |
| **Audit Log** | Mọi parse event (clear + ambiguous + override + cancel) | Append-only, encrypt raw_input | log_id |
| **Monitoring** | Audit log stream | Aggregate metrics theo giờ/ngày | dashboard + alert khi rate >threshold |

---

## 3. Schema chi tiết

### VN Slang Dictionary

```text
TABLE: slang_money
─────────────────────────────────────────────────────────────────────
slang        | value_vnd   | alt_meaning           | region    | active
─────────────────────────────────────────────────────────────────────
cành         | 100000      | bông hoa              | all       | true
củ           | 1000000     | cây/quả/đếm số lượng  | all       | true
chai         | 1000000     | chai chứa chất lỏng   | south     | true
lít          | 100000      | đơn vị 1L thể tích    | all       | true
xị           | 100000      | chai rượu nhỏ         | south     | true
loét         | 100000      | (vùng/cũ — verify)    | rare      | true
tờ đỏ        | 200000      | (mệnh giá tiền VN)    | all       | true
k            | 1000        | đơn vị nghìn          | all       | true   (clear)
nghìn        | 1000        | đơn vị nghìn          | all       | true   (clear)
triệu        | 1000000     | đơn vị triệu          | all       | true   (clear)
tỷ           | 1000000000  | đơn vị tỷ             | all       | true   (clear)
─────────────────────────────────────────────────────────────────────
version: v1.2.0  |  maintainer: research@expense.app  |  updated: 2026-05-10
```

### Confirmation Queue

```text
TABLE: pending_confirmations
─────────────────────────────────────────────────────────────────────
id           | user_id  | raw_input                          | options_json                   | created_at | ttl_expires_at | status
─────────────────────────────────────────────────────────────────────
pc_8a3f      | u_2901   | "mua macbook 3 củ rưỡi trả góp"   | [{3.5tr},{35tr,hint:macbook}]  | 14:23      | 14:23+24h      | pending
pc_8a40      | u_2901   | "ghi 2 lít xăng"                   | [{50k,2L xăng},{200k,2L tiền}] | 14:25      | 14:25+24h      | resolved (user chose 50k @ 14:25:12)
─────────────────────────────────────────────────────────────────────

State transitions:
  pending → resolved   (user chọn option)
  pending → cancelled  (user bấm "Huỷ")
  pending → expired    (24h timeout — auto-cancel, push notify)
```

### Audit Log

```text
TABLE: parse_audit (append-only, encrypted raw_input)
─────────────────────────────────────────────────────────────────────
log_id    | user_id  | ts          | raw_input (encrypted)        | llm_output_json    | detector_corrections          | user_choice         | final_amount
─────────────────────────────────────────────────────────────────────
al_001    | u_2901   | 14:23:05    | <enc: "macbook 3 củ rưỡi"> | {confidence:amb,...} | none                          | option_idx=1        | 35000000
al_002    | u_2901   | 14:25:12    | <enc: "2 lít xăng">         | {confidence:clear,500000} | OVERRIDE→ambiguous (whitelist "lít") | option_idx=0  | 50000
─────────────────────────────────────────────────────────────────────
retention: 90 days  |  encryption: AES-256 per-user key
```

### Monitoring metrics

```text
DASHBOARD: parse_health
─────────────────────────────────────────────────────────────────────
Metric                  | Window | Threshold       | Current  | Status
─────────────────────────────────────────────────────────────────────
ambiguity_rate          | 1h     | <30%            | 18%      | OK
ambiguity_rate          | 24h    | <25%            | 22%      | OK
dispute_rate            | 7d     | <2%             | 0.4%     | OK
override_10x_rate       | 24h    | <5% per user    | 1.2%     | OK
llm_json_malformed_rate | 1h     | <1%             | 0.3%     | OK
queue_overflow_size     | live   | <1000 pending   | 234      | OK
─────────────────────────────────────────────────────────────────────
ALERT triggers:
  - ambiguity_rate >40% trong 1h → có thể LLM/dict bị drift
  - dispute_rate >5% trong 7d → user đang sửa khoản đã save → UI/UX failed
  - override_10x_rate >10% per user → user thường xuyên chọn option ×10 → có thể default option đang sai
```

---

## 4. Khi hệ thống gặp vấn đề

| Khi nào lỗi xảy ra? | Hệ thống làm gì? | Người dùng thấy gì? |
|---|---|---|
| LLM down hoặc timeout (>5s) | Retry 1 lần với prompt rút gọn; nếu fail tiếp → fallback rule-based parser (regex slang dict) | UI hiển thị card State 4 ("Đang ghi tạm — chưa lưu"); nói "Mạng chậm, mình giữ nháp nhé" |
| LLM trả JSON malformed | Detector validate fail → log lỗi + retry 1 lần với system prompt thêm "OUTPUT MUST BE VALID JSON" | UI State 4; cùng UX với LLM down |
| Detector phát hiện LLM miss flag (vd có "củ" mà LLM nói confidence=clear) | Override sang ambiguous + log entry với `detector_corrections="OVERRIDE→ambiguous"` | UI State 2 (ambiguous card); user không biết có override (transparent fix) |
| Queue overflow (>5000 pending per user) | Reject parse mới, push notification "Bạn có quá nhiều khoản chưa xác nhận — xử lý các khoản cũ trước" | Toast cảnh báo + link tới màn hình pending |
| Audit log write fail | Block commit DB (KHÔNG commit transaction nếu không log được — audit là điều kiện cần) + alert ops team | UI State 4 + thông báo "Mình đang gặp lỗi lưu lịch sử, thử lại sau 1 phút" |
| Dictionary version mismatch (vd LLM dùng dict cũ) | Detector dùng dict version mới hơn override; log entry ghi nhận version mismatch | Không thấy gì (transparent) |
| User dispute (sau 30 ngày phát hiện sai) | Tra audit log theo log_id; show raw_input + options + user_choice + timestamp để user đối chứng | Màn hình "Lịch sử khoản này" với full audit trail |

---

## 5. Kiểm tra nhanh

- [x] Sơ đồ không chỉ là "AI trả lời tốt hơn", mà có bước kiểm tra cụ thể (Ambiguity Detector validate + override).
- [x] Có cách xử lý khi thiếu dữ liệu (LLM down → fallback regex parser; JSON malformed → retry + State 4).
- [ ] Có cách chuyển sang người thật — không áp dụng cho F-03 (F-01 dự phòng mới cần escalation).
- [x] Có cách theo dõi để lần sau sửa tốt hơn (Monitoring 6 metrics + alert thresholds).

---

## 6. Tradeoffs và open questions

| Tradeoff | Decision đề xuất | Cần verify |
|---|---|---|
| Async vs sync detector | Async để giảm latency (<300ms response) | Có race condition không nếu user nhanh tap confirm trước khi detector chạy xong? |
| Dictionary as DB table vs config file | DB table — cho phép update không cần redeploy | Cần ACL: chỉ research role được edit |
| Queue trong DB (Postgres) vs message queue (Redis) | Postgres staging table — đơn giản, không thêm infra | Nếu scale >100k users, có thể migrate sang Redis |
| Audit log retention 90d vs 365d | 90d (tuân thủ NĐ 13 + cost) | Có user nào dispute sau 90d không? Cần research |
| Encrypt raw_input per-user vs per-app key | Per-user — user xóa account → recover được key | Performance impact với 10M+ rows? |
