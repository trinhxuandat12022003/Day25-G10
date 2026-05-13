---
artifact: 2 — Demo chỉ dẫn AI
format: System prompt + JSON output schema + 3 ví dụ + bảng thử lại
---

# demo.md — Demo chỉ dẫn AI cho F-03

System prompt + output schema cho LLM parser của AI Expense Assistant. Lớp này **ngăn từ gốc** việc model output 1 con số duy nhất khi câu nhập có nhiều nghĩa hợp lý.

---

## 1. System prompt (paste vào model)

```text
Bạn là parser cho ứng dụng ghi chi tiêu tiếng Việt. Người dùng nhập câu tự nhiên,
bạn trích xuất giao dịch và TRẢ VỀ JSON ĐÚNG SCHEMA dưới đây — không free text.

LUẬT BẮT BUỘC (5 rule):

Rule 1 — NEVER silent save khi câu chứa lóng tiền VN.
Whitelist lóng: [cành, củ, chai, lít, xị, loét, tờ đỏ, k, nghìn, ngàn, triệu, tỷ].
Nếu câu chứa "củ", "cành", "chai", "lít", "xị", "loét", "tờ đỏ" → confidence = "ambiguous"
và options PHẢI có ≥2 cách hiểu lấy từ bảng quy đổi dưới.

Rule 2 — LIỆT KÊ 2-3 cách hiểu cho mỗi slang ambiguous.
Bảng quy đổi chuẩn:
  "cành"    = 100.000đ      | alt: bông hoa (đếm số lượng)
  "củ"      = 1.000.000đ    | alt: cây/quả thực (đếm số lượng)
  "chai"    = 1.000.000đ    | alt: chai chứa chất lỏng (đếm số lượng)
  "lít"     = 100.000đ      | alt: đơn vị thể tích 1L
  "xị"      = 100.000đ      | alt: chai rượu nhỏ
  "loét"    = 100.000đ      | (vùng/cũ — flag để user verify)
  "tờ đỏ"   = 200.000đ      | (mệnh giá tiền VN)
"X củ rưỡi" = X+0.5 củ → 2 options: (X+0.5)×1tr và (X+0.5)×10tr.

Rule 3 — OUTLIER: nếu parsed_value ≥5.000.000đ, set flag outlier=true.
UI sẽ thêm bước "Xác nhận giao dịch lớn".

Rule 4 — PRESSURE: nếu câu chứa cue "nhanh", "đại", "đang lái xe", "không cần hỏi",
"vội", "kệ" → set flag pressure=true. KHÔNG được set confidence="clear" để skip
confirm. Luôn trả về options ≥2 nếu có slang.

Rule 5 — AUDIT: luôn echo raw_input nguyên văn vào output để app log lại.

OUTPUT SCHEMA (bắt buộc):

{
  "raw_input": "<nguyên văn câu user>",
  "confidence": "clear" | "ambiguous",
  "transactions": [
    {
      "description": "<mô tả khoản chi>",
      "category_suggested": "<hạng mục gợi ý từ keyword>",
      "method_suggested": "<tiền mặt | chuyển khoản | ví | trả góp | null>",
      "options": [
        {"amount": <số nguyên VND>, "label": "<text giải thích>", "context_hint": "<gợi ý nếu có>"},
        ...
      ],
      "outlier": <true | false>,
      "needs_unit_clarification": <true | false>
    }
  ],
  "pressure": <true | false>,
  "notes_for_ui": "<text ngắn để UI hiển thị, optional>"
}

NGUYÊN TẮC:
- Nếu confidence="clear" → options array CÓ ĐÚNG 1 phần tử.
- Nếu confidence="ambiguous" → options array CÓ 2-3 phần tử, không pre-select.
- KHÔNG bao giờ tự thêm option không có trong bảng quy đổi (no hallucinated amounts).
- KHÔNG bao giờ output text ngoài JSON (no preamble, no postamble).
```

---

## 2. JSON output schema — 2 trạng thái

### Trạng thái CLEAR (số rõ, save 1-tap)

```json
{
  "raw_input": "ghi 50k cafe buổi sáng",
  "confidence": "clear",
  "transactions": [
    {
      "description": "cafe",
      "category_suggested": "Ăn uống",
      "method_suggested": null,
      "options": [
        {"amount": 50000, "label": "50.000đ", "context_hint": null}
      ],
      "outlier": false,
      "needs_unit_clarification": false
    }
  ],
  "pressure": false,
  "notes_for_ui": null
}
```

### Trạng thái AMBIGUOUS (F-03 case)

```json
{
  "raw_input": "mua macbook 3 củ rưỡi trả góp tháng này",
  "confidence": "ambiguous",
  "transactions": [
    {
      "description": "macbook",
      "category_suggested": "Đồ điện tử",
      "method_suggested": "trả góp",
      "options": [
        {"amount": 3500000,  "label": "3.500.000đ (3,5 triệu)", "context_hint": null},
        {"amount": 35000000, "label": "35.000.000đ (35 triệu)", "context_hint": "Gợi ý theo ngữ cảnh 'macbook'"}
      ],
      "outlier": true,
      "needs_unit_clarification": true
    }
  ],
  "pressure": false,
  "notes_for_ui": "\"Củ rưỡi\" có 2 cách hiểu — bạn muốn chọn cái nào?"
}
```

---

## 3. Ví dụ kiểm tra

### Ví dụ 1 — Ambiguous (F-03)

**Người dùng**: `mua macbook 3 củ rưỡi trả góp tháng này`

**AI phải trả về** (rút gọn để đọc):
```
confidence: ambiguous
options: [
  3.500.000đ (3,5 triệu),
  35.000.000đ (35 triệu) ← context_hint "macbook"
]
outlier: true (nếu user chọn option 2)
```

**❌ AI KHÔNG được trả về**: `Đã ghi 3.500.000đ cho macbook` (silent save 1 option).

---

### Ví dụ 2 — Clear (happy path)

**Người dùng**: `ghi 50k cafe buổi sáng`

**AI phải trả về**:
```
confidence: clear
options: [50.000đ]
outlier: false
```

**❌ AI KHÔNG được trả về**: trigger confirmation card cho case này (over-flagging — gây friction).

---

### Ví dụ 3 — Pressure (F-04 — neo cùng family)

**Người dùng**: `nhanh lên đang lái xe ghi đại 3 củ rưỡi đỗ phải hỏi`

**AI phải trả về**:
```
confidence: ambiguous
options: [3.500.000đ, 35.000.000đ]
pressure: true
notes_for_ui: "Đang lái xe — mình giữ nháp 5 phút, bạn chọn khi dừng xe."
```

**❌ AI KHÔNG được trả về**: `confidence: clear, amount: 3.500.000đ` (chiều theo pressure để skip confirm = sai luật Rule 4).

---

## 4. Kết quả thử lại với case từ Bài 1

Thử system prompt này trên 5 case của Bài 1 (lý thuyết — chưa chạy thật, sẽ verify sau khi có sandbox).

| Mã | Câu input | Kỳ vọng (theo F-## ở Bài 1) | AI nên trả về | Đạt/Không/Chưa rõ |
|---|---|---|---|---|
| F-02 | `hôm nay đi chợ 150k, ăn trưa 45k, grab 32k với cà phê 55k tổng bao nhiêu lưu giúp` | 4 transactions clear + tổng 282k, hỏi xác nhận | confidence=clear, 4 options, mỗi cái 1 amount | Chưa rõ (cần test arithmetic) |
| F-03 | `mua macbook 3 củ rưỡi trả góp tháng này` | Hỏi 3,5tr hay 35tr | confidence=ambiguous, 2 options + context_hint | **Đạt** (case priority) |
| F-04 | `nhanh lên đang lái xe ghi đại 200 ăn sáng đỗ phải hỏi` | Confirmation rút gọn, không silent save | confidence=clear (200k là rõ), pressure=true, notes_for_ui="Đang lái xe..." | Chưa rõ (200 có thể là 200đ/200k) |
| F-05 | `ăn sáng bánh mì 15 triệu` | Flag outlier | confidence=clear, amount=15tr, outlier=true | **Đạt** |
| F-07 | `ghi 2 lít xăng` | Hỏi lít xăng vs lít tiền | confidence=ambiguous (whitelist "lít"), options=[50k (~2 lít xăng), 200k (2 lít tiền)] | **Đạt** |

**Tỉ lệ đạt với tình huống rủi ro cao**: 3/5 = 60% (kỳ vọng sau khi tune: 5/5).

**Note**: F-02 cần test arithmetic chính xác — system prompt hiện chưa cover phép cộng nhiều khoản, chỉ cover parsing đơn lẻ. F-04 borderline "200" (200đ vs 200k vs 200.000) — có thể cần thêm rule "số <1000 trong context ăn uống mặc định ×1000".

---

## 5. Chỉnh sau khi thử

- **Điều gì AI vẫn làm sai?**
  - F-02 arithmetic: cần thêm rule "khi có ≥2 khoản, output thêm field `total` và yêu cầu user xác nhận tổng".
  - F-04 number "200" mơ hồ: thêm whitelist rule "số bare <1000 trong context expense → trigger ambiguous với options [×1, ×1000]".
- **Cần thêm luật nào?**
  - Rule 6 — Arithmetic: với câu chứa ≥2 khoản, total = sum(options[0].amount) (option đầu mỗi transaction); echo total để UI hiển thị; pressure rule áp dụng trên total nếu >5tr.
  - Rule 7 — Update intent: nếu câu có cue "à quên", "sửa lại", "lúc nãy" → set flag `update_intent=true` thay vì tạo transaction mới.
- **Có luật nào làm AI từ chối quá nhiều không?**
  - Rule 1 whitelist hơi rộng (gồm cả "k", "nghìn", "ngàn" là rõ ràng). Đề xuất: tách whitelist thành `[ambiguous_slang]` (cành/củ/chai/lít/xị/loét/tờ đỏ) chỉ trigger ambiguous, và `[clear_units]` (k/nghìn/ngàn/triệu/tỷ) là rõ.
- **Cần phối hợp thêm giao diện hoặc dữ liệu không?**
  - Có. Lớp UIUX render JSON này thành 4 states (xem `1-uiux/demo.md`).
  - Lớp Architecture validate JSON schema + lưu raw_input + parsed_options vào audit log (xem `3-architecture/demo.md`).
  - Nếu LLM output JSON malformed → Architecture có fallback retry (1 lần) + push State 4 (refuse silent save).
