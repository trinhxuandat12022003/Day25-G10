---
artifact: 1 — FINAL kế hoạch giải pháp
bai-tap: 2 — Thiết kế giải pháp
phase: Chọn rủi ro + chọn tầng + chọn demo + chốt 3 lớp giải pháp
time: 11:00-11:55
input: 00-context.md + 01-test-set-review/3-FINAL-test-set-eval-plan.md
nop-cuoi: Có — file cuối Bài 2
---

# 1 — FINAL: Kế hoạch giải pháp

File này ghi lại quyết định chính của Bài 2:

- Rủi ro nào được chọn.
- Vì sao rủi ro đó quan trọng.
- Nguyên nhân gốc là gì.
- Nhóm sẽ xây 3 lớp giải pháp nào.
- Mỗi lớp dùng demo gì.

Lý do cần 3 lớp: một giải pháp đơn lẻ dễ lọt lỗi. Với rủi ro nặng, nhóm cần nhiều lớp cùng đỡ: lớp này ngăn, lớp kia phát hiện, lớp khác khắc phục hoặc thông báo cho người dùng.

Ba lớp giải pháp nằm trong thư mục `artifact/`:

| Lớp | Thư mục | Vai trò |
|---|---|---|
| Giao diện | `artifact/1-uiux/` | Cảnh báo, dẫn nguồn, nút chuyển sang người thật |
| Chỉ dẫn AI | `artifact/2-prompt/` | Hỏi lại, từ chối, bắt buộc dẫn nguồn |
| Kiến trúc dữ liệu | `artifact/3-architecture/` | Tra cứu nguồn đúng, lưu tạm dữ liệu, xử lý khi thiếu nguồn, giám sát |

Ba lớp này bổ sung cho nhau. Nếu một lớp lọt lỗi, lớp khác vẫn có thể chặn hoặc giảm hại.

## Thông tin nhóm

- **Chủ đề**: Track 04 — AI Expense Assistant tiếng Việt
- **Thành viên**: Dương, Đạt, Tùng
- **Ngày**: 2026-05-13

---

## Phần A — Chọn rủi ro và tầng giải pháp

### Rủi ro chính được chọn

- **ID tình huống**: F-03 — Lóng số lớn dispute 10x (`mua macbook 3 củ rưỡi trả góp tháng này`)
- **Mô tả ngắn**: Khi user nhập câu chứa lóng tiền VN ("củ rưỡi", "cành", "chai", "lít") đặc biệt trên giao dịch lớn, AI có xu hướng pick 1 trong 2 nghĩa hợp lý mà không hỏi lại, gây sai số 10x cho báo cáo chi tiêu và mất niềm tin vào app khi user phát hiện sao kê ngân hàng không khớp.
- **Mức độ**: Nặng
- **Điểm rủi ro**: 20 (Tác động 5 × Khẩn cấp 4)
- **Vì sao chọn tình huống này**:
  1. Neo trực tiếp 2 sự cố thật mạnh nhất Phần A: R-01 Air Canada (trách nhiệm AI khi đưa thông tin tài chính sai) + R-04 LLM arithmetic errors (hallucination rate 3-27%).
  2. Đặc thù tiếng Việt — không có precedent quốc tế nào cover lóng "củ/cành/chai/lít/xị" → cả 3 thành viên đều propose ở Bài 1 (Dương T-03, Đạt L1-C1, Tùng S-02/S-03).
  3. Family lớn — 1 solution thiết kế cho F-03 cũng đỡ được F-04 (pressure), F-05 (outlier), F-07 (ambiguous lít), F-11 (hiếu hỉ category) — ROI design cao nhất trong 15 case.

### Tìm nguyên nhân gốc

Đừng chỉ mô tả lỗi. Hãy trả lời: vì sao lỗi xảy ra?

- [x] Thiếu nguồn dữ liệu đúng — không có VN Slang Dictionary làm source-of-truth; LLM tự nhớ bảng quy đổi, không version control.
- [x] AI đoán khi không biết — "củ rưỡi" có 2 nghĩa hợp lý (3,5tr hoặc 35tr), LLM pick nghĩa phổ biến nhất thay vì flag uncertainty và liệt kê.
- [x] Giao diện khiến người dùng tin quá mức — silent save 1 số duy nhất; user vội, không tự cộng lại; cognitive offloading mạnh.
- [ ] Quy trình thiếu người duyệt hoặc thiếu bước chuyển sang người thật — không áp dụng cho F-03 (không cần escalate; F-01 dự phòng mới cần).
- [x] Không có theo dõi sau khi ra mắt — không có metric ambiguity_rate / dispute_rate / override_10x_rate, không biết model có drift không.
- [ ] Khác.

### Bảng nối nguyên nhân với tầng sửa

| Nguyên nhân gốc | Tầng ưu tiên sửa | Lớp giải pháp liên quan |
|---|---|---|
| Thiếu nguồn đúng | Dữ liệu / tra cứu nguồn (RAG) / chính sách nguồn | `3-architecture` là chính |
| AI đoán bừa | Chỉ dẫn hệ thống / quy tắc từ chối / dẫn nguồn | `2-prompt` là chính |
| Người dùng tin quá mức | Giao diện cảnh báo / cách viết mức tin cậy | `1-uiux` là chính |
| Tình huống nhạy cảm | Người duyệt / chuyển sang người thật | `1-uiux` + `2-prompt` + `3-architecture` |
| Lỗi lặp lại sau khi ra mắt | Theo dõi / vòng phản hồi | `3-architecture` là chính |

Nguyên tắc: lỗi ở tầng nào, ưu tiên sửa ở tầng đó. Đừng chỉ thêm cảnh báo giao diện nếu nguyên nhân gốc là thiếu nguồn dữ liệu hoặc AI đoán khi không biết.

### 10 tầng giải pháp tham khảo

Không bắt buộc dùng đủ 10 tầng. Bảng này giúp nhóm chọn đúng hướng sửa.

| Tầng | Khi nào dùng |
|---|---|
| Giao diện | Người dùng tin AI quá mức, thiếu cảnh báo, thiếu nguồn, thiếu nút chuyển sang người thật |
| Chỉ dẫn AI | AI đoán khi không biết, không hỏi lại, không từ chối |
| Quy trình xử lý | Cần phân loại ý định, chuyển đúng nơi xử lý, có cách xử lý khi AI không nên trả lời |
| Dữ liệu / tra cứu nguồn (RAG) | Thiếu nguồn đúng, nguồn cũ, AI không dựa vào nguồn đáng tin cậy |
| Theo dõi | Lỗi lặp lại sau khi ra mắt nhưng không ai thấy |
| Chính sách / thông báo giới hạn | Người dùng không biết giới hạn của AI |
| Người duyệt / phê duyệt | Tình huống pháp lý, y tế, tài chính, tuyển dụng, hoặc tác động lớn |
| Vai trò trách nhiệm | Có cảnh báo nhưng không ai chịu trách nhiệm xử lý |
| Vòng phản hồi | Cần người dùng / người rà báo lỗi để cập nhật hệ thống |
| Kiến trúc lai | LLM một mình không đủ, cần rule, classifier, hoặc nhiều bước kiểm tra |

### 4 hành động phòng vệ

Mỗi lớp nên làm ít nhất một việc:

- **Ngăn**: giảm khả năng lỗi xảy ra từ đầu.
- **Phát hiện**: nhận ra lỗi hoặc tín hiệu nguy hiểm.
- **Khắc phục**: chuyển sang người thật, dùng câu trả lời dự phòng, hoặc dừng trả lời.
- **Thông báo**: giúp người dùng hiểu mức tin cậy và rủi ro.

Gợi ý theo mức rủi ro:

| Mức rủi ro | Nên có |
|---|---|
| Nhẹ | Ít nhất 1 hành động |
| Vừa | Ít nhất 2 hành động |
| Nặng | Ít nhất 3 hành động |
| Rất nặng / không đảo ngược được | Cố gắng đủ 4 hành động + có người chịu trách nhiệm |

### Kết luận Phần A

**Nguyên nhân gốc**: 3 cause cùng có — (1) AI đoán khi không biết, (2) thiếu bước confirmation, (3) người dùng tin AI quá mức (cognitive offloading). Vì cả 3 cause cùng xuất hiện, không lớp nào đơn lẻ đỡ được — phải sửa ở cả 3 tầng.

**Tầng chính cần sửa**: Cả 3 tầng đều là tầng chính. UIUX chặn cuối, Prompt ngăn từ gốc, Architecture validate và lưu evidence. Bỏ bất kỳ lớp nào → còn lỗ hổng.

**Vì sao cần 3 lớp giải pháp**:

- **Lớp giao diện** (`1-uiux/`): chặn cuối trước khi DB ghi. Render confirmation card với 2-3 lựa chọn quy đổi → user phải tap chọn, không silent save. 4 states phục vụ 4 risk pattern (clear / ambiguous / outlier / pressure).
- **Lớp chỉ dẫn AI** (`2-prompt/`): ngăn từ gốc — buộc LLM output JSON structured với `confidence: "ambiguous"` + `options: [...]` khi gặp whitelist lóng VN. Không cho model output 1 con số duy nhất khi câu nhập có nhiều nghĩa hợp lý.
- **Lớp kiến trúc dữ liệu** (`3-architecture/`): backup nếu prompt fail. Ambiguity Detector validate JSON + cross-check với VN Slang Dictionary + override LLM khi miss flag. Confirmation Queue giữ dữ liệu trong staging 24h, không commit DB cho tới khi user xác nhận. Audit Log lưu raw_input + parsed_options + user_choice cho dispute resolution.

---

## Phần B — Chọn định dạng demo

Mỗi lớp cần một bản demo. Demo giúp biến ý tưởng thành thứ trực quan để nhóm khác xem, kiểm tra và phản biện.

| Lớp | Thư mục | Định dạng demo chọn | Thời gian dự kiến |
|---|---|---|---|
| Giao diện | `1-uiux` | ASCII sketch — 4 states (theo `prompts/05a-ascii-ui-sketch.md`) | 20 phút |
| Chỉ dẫn AI | `2-prompt` | Markdown — system prompt + JSON schema + 3 ví dụ + bảng thử lại | 25 phút |
| Kiến trúc dữ liệu | `3-architecture` | ASCII data flow + bảng schema (theo `prompts/05e-ascii-architecture.md`) | 25 phút |

**Lý do chọn demo**

- **Giao diện — ASCII**: review trong văn bản (Markdown), không cần tool ngoài (Figma/Excalidraw); render được cả trên GitHub và VSCode preview; đủ rõ để demo 4 states confirmation card với nội dung tiếng Việt thuần.
- **Chỉ dẫn AI — Markdown**: prompt phải copy-paste được vào LLM để test, không phù hợp với sketch. JSON schema + 3 ví dụ ở dạng code block dễ reproduce.
- **Kiến trúc dữ liệu — ASCII**: data flow + bảng schema thể hiện tốt trong ASCII (5 components + 4 schema bảng); không cần Mermaid vì phần lớn là table chứ không phải flow phức tạp.

Gợi ý: có thể dùng AI để dựng nhanh bản nháp demo, nhưng nhóm phải đọc lại và sửa.

### Chọn demo theo điều cần chứng minh

| Nếu cần chứng minh... | Demo phù hợp |
|---|---|
| Người dùng nhìn thấy gì | Sketch, Figma, HTML, ASCII UI |
| AI được chỉ dẫn thế nào | Bản prompt trong Markdown, ví dụ trả lời |
| Dữ liệu đi qua đâu | Sơ đồ hộp-mũi tên, ASCII, Mermaid |
| Quy trình chuyển sang người thật | Sơ đồ quy trình |

---

## Phần C — Ba lớp giải pháp

Ghi tóm tắt ở đây. Chi tiết nằm trong `card.md` và `demo.*` của từng thư mục.

### Lớp 1 — Giao diện (`artifact/1-uiux/`)

- **Cách tiếp cận**: Confirmation card với 2-3 lựa chọn quy đổi rõ ràng khi gặp lóng VN; outlier warning cho giao dịch >5tr; voice command + "để nháp 5 phút" cho user đang lái xe.
- **Hành động phòng vệ bao phủ**: Thông báo (badge "có 2 cách hiểu") + Phát hiện (outlier ≥5tr) + Khắc phục (cho user chọn lại).
- **Demo**: ASCII 4 states — Default · Ambiguous · Outlier · Refuse Silent Save.
- **Trạng thái**: Xong (chờ phản biện chéo).

Link chi tiết:

- [`artifact/1-uiux/card.md`](artifact/1-uiux/card.md)
- [`artifact/1-uiux/demo.md`](artifact/1-uiux/demo.md)

### Lớp 2 — Chỉ dẫn AI (`artifact/2-prompt/`)

- **Cách tiếp cận**: System prompt rule-based — 5 rule (whitelist lóng VN, structured JSON output, outlier ≥5tr, pressure cue, audit echo). LLM buộc trả `confidence: "ambiguous"` + `options: [...]` thay vì pick 1 số.
- **Hành động phòng vệ bao phủ**: Ngăn (rule "never silent save khi có slang") + Hỏi lại (liệt kê 2-3 cách hiểu) + Dẫn nguồn (echo raw_input vào output cho audit).
- **Demo**: System prompt đầy đủ + JSON schema 2 trạng thái + 3 ví dụ (clear/ambiguous/pressure) + bảng thử lại 5 case từ Bài 1.
- **Trạng thái**: Xong (chờ test thực với sandbox LLM).

Link chi tiết:

- [`artifact/2-prompt/card.md`](artifact/2-prompt/card.md)
- [`artifact/2-prompt/demo.md`](artifact/2-prompt/demo.md)

### Lớp 3 — Kiến trúc dữ liệu (`artifact/3-architecture/`)

- **Cách tiếp cận**: 4 components mới — VN Slang Dictionary (source of truth) · Ambiguity Detector (validate + override LLM khi miss flag) · Confirmation Queue (staging 24h TTL, không silent commit) · Audit Log + Monitoring (telemetry rate of ambiguity / dispute / override_10x).
- **Hành động phòng vệ bao phủ**: Ngăn (queue chặn silent commit) + Phát hiện (Detector override LLM) + Khắc phục (rollback từ audit log) + Theo dõi (monitoring metrics).
- **Demo**: ASCII data flow + 4 bảng schema (Slang Dict / Queue / Audit Log / Monitoring) + bảng failure handling.
- **Trạng thái**: Xong (chờ ops review về infra cost).

Link chi tiết:

- [`artifact/3-architecture/card.md`](artifact/3-architecture/card.md)
- [`artifact/3-architecture/demo.md`](artifact/3-architecture/demo.md)

---

## Tổng kiểm tra

| Câu hỏi | Trả lời |
|---|---|
| Rủi ro chính đã chọn là gì? | **F-03** — Lóng số lớn dispute 10x (`mua macbook 3 củ rưỡi trả góp tháng này`) |
| Nguyên nhân gốc là gì? | 3 cause: (1) AI đoán khi không biết, (2) thiếu confirmation step, (3) người dùng tin AI quá mức |
| 3 lớp giải pháp đã đủ chưa? | Giao diện: ✓ Xong / Chỉ dẫn AI: ✓ Xong / Kiến trúc: ✓ Xong |
| 4 hành động đã bao phủ chưa? | Ngăn: ✓ (Prompt + Arch) / Phát hiện: ✓ (UIUX + Prompt + Arch) / Khắc phục: ✓ (UIUX + Arch) / Thông báo: ✓ (UIUX + Prompt) |
| Nhóm khác đã góp ý chưa? | Chưa — chờ phản biện chéo (cross-team review) |
| Nhóm đã sửa gì sau phản biện? | Pending — sẽ ghi vào đây sau buổi review |

### Ma trận 3 lớp × 4 hành động phòng vệ

| Hành động | UIUX | Prompt | Architecture |
|---|---|---|---|
| **Ngăn** | — | ✓ (rule "never silent save khi có slang") | ✓ (Confirmation Queue chặn commit DB) |
| **Phát hiện** | ✓ (outlier warning State 3) | ✓ (slang whitelist + outlier rule) | ✓ (Ambiguity Detector override LLM) |
| **Khắc phục** | ✓ (3 radio options cho user chọn lại) | — | ✓ (rollback từ audit log; queue auto-cancel 24h) |
| **Thông báo** | ✓ (raw_input + badge "có 2 cách hiểu") | ✓ (liệt kê 2-3 options trong JSON) | — |

→ Đủ 4 hành động (yêu cầu cho mức Nặng) + có lớp dự phòng khi 1 lớp fail.

## Phản biện chéo: 4 câu phải trả lời

Khi nhóm khác góp ý, hoặc khi nhóm tự rà lại, dùng 4 câu này:

| Góc phản biện | Câu hỏi |
|---|---|
| Đúng tầng | Giải pháp có sửa đúng nguyên nhân gốc không? |
| Cụ thể | Demo có đủ rõ để hiểu cách vận hành không? |
| Đủ lớp | 3 lớp có bổ sung cho nhau không, hay đang lặp cùng một ý? |
| Tác dụng phụ | Giải pháp có làm chậm, tốn kém, rối giao diện, hoặc gây hiểu nhầm mới không? |

Ghi góp ý cụ thể vào `card.md` hoặc phần tổng kiểm tra. Không ghi chung chung "ổn" hoặc "chưa ổn".

### Tự rà của nhóm (trước khi cross-team review)

| Góc phản biện | Trả lời |
|---|---|
| **Đúng tầng** | ✓ Có. F-03 có 3 root cause cùng lúc (AI đoán + thiếu confirm + user tin AI), nên 3 lớp đánh đúng 3 cause: Prompt (AI đoán) · UIUX (user tin AI) · Architecture (thiếu confirm). Nếu chỉ chọn 1 lớp, 2 cause còn lại không có gì đỡ. |
| **Cụ thể** | ✓ Có. UIUX có 4 state ASCII với raw_input + radio options + outlier comparison. Prompt có JSON schema đầy đủ + 3 ví dụ input/output. Architecture có data flow + 4 bảng schema + bảng failure handling. Mỗi demo có thể implement được bởi developer mà không cần hỏi lại. |
| **Đủ lớp** | ✓ Có. Ma trận 3×4 ở Tổng kiểm tra: 4 hành động phòng vệ đều có ≥2 lớp cover; không hành động nào chỉ 1 lớp single-point-of-failure. Lớp Prompt ngăn từ gốc; nếu LLM bypass rule, Architecture Detector override; nếu Detector miss, UIUX confirmation card vẫn chặn cuối. |
| **Tác dụng phụ** | ⚠ Có tradeoff đã document: (1) UIUX gây confirmation friction → giảm bằng "chỉ confirm khi thật sự mơ hồ" (whitelist explicit). (2) Prompt có latency JSON output → async pipeline. (3) Architecture có storage cost audit log → retention 90 ngày + encrypt per-user. (4) Privacy raw_input có thể chứa thông tin nhạy cảm ("quà cho bồ" F-12) → encrypt at rest, user xóa account → key destroyed. |

### Open issues cần verify với team / user research

1. **User research** — UI có 4 state, có thể user mới confused. Cần test 4 scenario (Macbook 3 củ rưỡi · 2 lít xăng · pressure lái xe · 15tr ăn sáng outlier) với 5-10 user thật để measure click time + accuracy.
2. **LLM sandbox test** — system prompt chưa chạy trên model thật. Cần test trên Claude/GPT-4 với 15 case F-01→F-15 từ Bài 1 để đo tỉ lệ JSON malformed + over-flagging.
3. **Infra cost** — audit log + queue ước lượng 10-20GB sau 1 năm với 100k users. Cần ops review về storage + encryption key management.
4. **Dictionary maintenance** — VN slang thay đổi theo vùng/thời gian. Cần PM/researcher commit owner cho dictionary + process review hàng quý.

## Gợi ý chia việc

Nhóm 3 người:

- Thành viên A: `artifact/1-uiux/`
- Thành viên B: `artifact/2-prompt/`
- Thành viên C: `artifact/3-architecture/`

Nhóm 2 người:

- Một người phụ trách 2 lớp.
- Người còn lại phụ trách 1 lớp và rà lại 2 lớp kia.

5 phút cuối: cả nhóm đọc chéo 3 lớp, sửa lại bảng tổng kiểm tra, rồi chuẩn bị phản biện chéo.
