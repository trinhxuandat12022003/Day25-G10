# Day 25 — Track 04: AI Expense Assistant

Trợ lý ghi và tổng hợp chi tiêu cá nhân với input tiếng Việt tự nhiên — phân tích rủi ro AI + thiết kế 3 lớp giải pháp cho rủi ro quan trọng nhất.

## Thành viên nhóm

| # | Mã học viên   | Họ tên đầy đủ        |
|---|---------------|----------------------|
| 1 | 2A202600326   | Trịnh Xuân Đạt       |
| 2 | 2A202600287   | Tạ Thị Thùy Dương    |
| 3 | 2A202600197   | Nguyễn Quang Tùng    |

## Kết quả cuối

- 🎯 [Bộ kiểm thử cuối — 15 tình huống F-01 → F-15](./worksheet/01-test-set-review/3-FINAL-test-set-eval-plan.md)
- 🎯 [Thiết kế 3 lớp giải pháp cho F-03 (lóng số lớn dispute 10x)](./worksheet/02-solution-design/1-map-and-format.md) + [artifact/](./worksheet/02-solution-design/artifact/)

## Cấu trúc bài nộp

```text
.
├── README.md                                        ← file này
│
└── worksheet/
    ├── 00-context.md                                ← Bối cảnh Track 04
    │
    ├── 01-test-set-review/
    │   ├── 1-diverge.md                             ← Index Mở rộng (trỏ tới 3 file cá nhân)
    │   ├── 01-diverge-Dat.md                        ← Diverge của Đạt
    │   ├── 1-diverge-Duong.md                       ← Diverge của Dương
    │   ├── 1-diverge-Tung.md                        ← Diverge của Tùng
    │   ├── 2-converge.md                            ← Hội tụ (49 → 20 cases)
    │   └── 3-FINAL-test-set-eval-plan.md            🎯 KẾT QUẢ CUỐI Bài 1
    │
    └── 02-solution-design/
        ├── 1-map-and-format.md                      🎯 KẾT QUẢ CUỐI Bài 2
        └── artifact/
            ├── 1-uiux/         (card.md + demo.md + demo.html — ASCII 4 states)
            ├── 2-prompt/       (card.md + demo.md — System prompt + JSON schema)
            └── 3-architecture/ (card.md + demo.md — ASCII data flow + 4 schema)
```

## Tóm tắt rủi ro chính được chọn

**F-03 — Lóng số lớn dispute 10x**: khi user nhập câu chứa lóng tiền VN ("củ rưỡi", "cành", "chai", "lít") đặc biệt trên giao dịch lớn, AI có xu hướng pick 1 trong 2 nghĩa hợp lý mà không hỏi lại, gây sai số 10x cho báo cáo chi tiêu (ví dụ: "macbook 3 củ rưỡi" = 3.500.000đ hay 35.000.000đ?).

**Điểm rủi ro**: 20 (Tác động 5 × Khẩn cấp 4) · Mức Nặng.

**3 lớp giải pháp**:
- **UIUX**: Confirmation card với 2-3 lựa chọn quy đổi rõ ràng + outlier warning ≥5tr.
- **Prompt**: System prompt rule-based — buộc LLM output JSON với `confidence: "ambiguous"` + `options: [...]` khi gặp whitelist lóng VN.
- **Architecture**: VN Slang Dictionary + Ambiguity Detector + Confirmation Queue 24h TTL + Audit Log + Monitoring telemetry.
