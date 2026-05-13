---
title: Day 25 — Responsible AI: Solution Design
source: day25-Lab-Assignment.pdf
pages: 6
converted_at: 2026-05-13T10:07:56.250946
---

Day 25 — Responsible AI: Solution
Design
Day 25 là bài tập nhóm 2-3 người cùng chủ đề (track). Nhóm bắt đầu từ 2
file đã làm ở Day 24 ( 01-risk-map.md + 02-test-eval-plan.md ), kết
thúc bằng bộ kiểm thử cuối (10-15 tình huống) + 3 lớp giải pháp cho rủi ro
quan trọng nhất.
### 📥
Nộp bài thế nào? (đọc trước khi vào lab)
Tên repo
Cú pháp: Day25-MãNhóm
Ví dụ: Day25-G001 , Day25-G045 , Day25-A1 .
Cần nộp những file gì?
Nhóm tạo 1 repo công khai trên GitHub, push toàn bộ thư mục
worksheet/ (đầy đủ cấu trúc dưới), rồi nộp link repo qua LMS.
Day25-MãNhóm/ ← repo công khai
│
├── README.md ← Thành viên nhóm (xem
│
└── worksheet/
├── 00-context.md ← Bối cảnh sản phẩm (đã
│
├── 01-test-set-review/
│ ├── 1-diverge.md ← Trung gian: pha Mở rộ
│ ├── 2-converge.md ← Trung gian: pha Hội t
│ └── 3-FINAL-test-set-eval-plan.md 🎯 KẾT QUẢ CUỐI Bài 1
│
└── 02-solution-design/
├── 1-map-and-format.md 🎯 KẾT QUẢ CUỐI Bài 2
└── artifact/
├── 1-uiux/
│ ├── card.md
│ └── demo.{md|png|html}
├── 2-prompt/
│ ├── card.md
│ └── demo.md
└── 3-architecture/
├── card.md
└── demo.md

🎯 = file người chấm xem trước. Các file còn lại là trung gian — phải nộp
kèm để người chấm thấy nhóm đã đi qua đủ quá trình.
README đầu repo phải có gì?
Sao chép mẫu này vào README.md ở gốc repo và điền:
# Day 25 — Track [N]: [Tên track]
## Thành viên nhóm
| # | Mã học viên | Họ tên đầy đủ |
|---|-------------|---------------|
| 1 | A20-XXXXX | Nguyễn Văn A |
| 2 | A20-XXXXX | Trần Thị B |
| 3 | A20-XXXXX | Lê Văn C |
## Kết quả cuối
- 🎯 [Bộ kiểm thử cuối](./worksheet/01-test-set-review/3-FINAL-test-set-
eval-plan.md)
- 🎯 [Thiết kế 3 lớp giải pháp](./worksheet/02-solution-design/1-map-
and-format.md) + [artifact/](./worksheet/02-solution-
design/artifact/)
Các bước nộp
1. Tạo repo công khai trên GitHub theo cú pháp Day25-MãNhóm .
2. Push toàn bộ thư mục worksheet/ (đầy đủ cấu trúc trên).
3. Tạo README.md ở gốc repo theo mẫu, điền mã học viên + tên đầy đủ.
4. Một thành viên đại diện nộp link repo vào LMS Day 25.
5. Kiểm tra link mở được trước 23:59 hôm nay.
Luồng làm bài
Đọc lại 01-risk-map.md + 02-test-eval-plan.md từ Day 24
-> Điền 00-context.md
-> Bài 1: Rà bộ kiểm thử
-> Mở rộng: tìm sự cố thật + dùng AI gợi ý tình huống
-> Hội tụ: gộp, lọc trùng, chấm rủi ro
-> Chốt 10-15 tình huống cuối
-> Bài 2: Thiết kế giải pháp
-> Chọn rủi ro chính
-> Tìm nguyên nhân gốc
-> Chọn tầng sửa và định dạng demo
-> Xây 3 lớp giải pháp song song
-> Phản biện chéo với nhóm khác
-> Chỉnh lại file
-> Nộp link repo qua LMS

Bài 1 — Rà bộ kiểm thử
Mục tiêu: chọn ra 10-15 tình huống đáng kiểm thử nhất và viết kế hoạch
chấm rõ ràng.
File cuối của Bài 1:
worksheet/01-test-set-review/3-FINAL-test-set-eval-plan.md
Giai đoạn Mở rộng — 30 phút
Mỗi thành viên làm trước, sau đó mới gộp nhóm.
1. Tìm sự cố thật có nguồn.
2. Dùng AI gợi ý thêm tình huống theo 4 góc nhìn.
3. Chọn khoảng 15 tình huống tốt nhất của mỗi người.
File dùng ở giai đoạn này:
worksheet/01-test-set-review/1-diverge.md
Giai đoạn Hội tụ — 30 phút
Nhóm cùng làm.
1. Gộp toàn bộ tình huống của nhóm.
2. Lọc trùng theo kiểu lỗi.
3. Chấm điểm rủi ro: Tác động x Độ khẩn cấp.
4. Chốt 10-15 tình huống cuối.
File dùng ở giai đoạn này:
worksheet/01-test-set-review/2-converge.md
Bài 2 — Thiết kế giải pháp
Mục tiêu: chọn rủi ro quan trọng nhất từ Bài 1, rồi xây 3 lớp giải pháp cho
cùng rủi ro đó.
File cuối của Bài 2:
worksheet/02-solution-design/1-map-and-format.md
Ba lớp giải pháp:

| Lớp | Thư mục | Mục đích |
| --- | --- | --- |

| Chỉ dẫn AI | artifact/2-prompt/ | Buộc AI hỏi lại, từ chối,
hoặc dẫn nguồn khi cần |
| --- | --- | --- |

| File / Thư mục | Dùng để làm gì |
| --- | --- |

| worksheet/00-context.md | Điền bối cảnh một lần, dán vào đầu
mọi cuộc trò chuyện với AI |
| --- | --- |

| worksheet/02-solution-design/ | Làm Bài 2. Hướng dẫn chọn tầng,
demo, phản biện nằm trong
worksheet |
| --- | --- |

| Prompt mẫu | Dùng khi nào | Lưu kết quả vào |
| --- | --- | --- |

| None | prompts/02- | None | Dùng AI gợi ý tình huống | 1-diverge.md Phần B |
| --- | --- | --- | --- | --- |
| None | None | prompts/02- | D | None |
|  | brainstorm.md | None | None | None |

| None | prompts/04-solution- | Gợi ý hướng giải pháp | 1-map-and-format.md |
| --- | --- | --- | --- |
| None | None | Gợi ý hư | None |
|  | options.md | None | None |

Lớp Thư mục Mục đích
Giao diện artifact/1-uiux/ Giúp người dùng thấy
cảnh báo, nguồn,
đường chuyển người
thật
Chỉ dẫn AI artifact/2-prompt/ Buộc AI hỏi lại, từ chối,
hoặc dẫn nguồn khi cần
Kiến trúc dữ liệu artifact/3- Đảm bảo AI tra cứu
architecture/ đúng nguồn và có
phương án dự phòng
Ba lớp này bổ sung cho nhau. Một lớp có thể lọt lỗi, nhiều lớp sẽ giảm rủi ro
tốt hơn.
Tài liệu trong thư mục này
File / Thư mục Dùng để làm gì
track-bank-scenario-kit-v1.md Chọn và đọc lại bối cảnh chủ đề
worksheet/00-context.md Điền bối cảnh một lần, dán vào đầu
mọi cuộc trò chuyện với AI
worksheet/01-test-set-review/ Làm Bài 1. Hướng dẫn chi tiết nằm
ngay trong từng file worksheet
worksheet/02-solution-design/ Làm Bài 2. Hướng dẫn chọn tầng,
demo, phản biện nằm trong
worksheet
prompts/ Prompt mẫu cho từng bước
Bảng dùng prompt mẫu
Prompt mẫu Dùng khi nào Lưu kết quả vào
prompts/01-deep- Tìm sự cố thật 1-diverge.md Phần A
research.md
prompts/02- Dùng AI gợi ý tình huống 1-diverge.md Phần B
brainstorm.md
prompts/03- Lọc trùng và ưu tiên 2-converge.md
convergent-
analysis.md
prompts/04-solution- Gợi ý hướng giải pháp 1-map-and-format.md
options.md

| Prompt mẫu | Dùng khi nào | Lưu kết quả vào |
| --- | --- | --- |

| Đừng làm | Nên làm |
| --- | --- |

| Nộp mỗi file cuối | Giữ cả file trung gian |
| --- | --- |

| Chỉ làm một lớp giải pháp | Làm đủ 3 lớp: giao diện, chỉ dẫn AI,
kiến trúc |
| --- | --- |

Prompt mẫu Dùng khi nào Lưu kết quả vào
prompts/05a-* đến Dựng demo nhanh artifact/*/demo.*
05f-*
Cách dùng prompt mẫu
1. Mở Claude / ChatGPT / Gemini / Perplexity tùy bước.
2. Dán toàn bộ worksheet/00-context.md vào đầu cuộc trò chuyện.
3. Dán prompt mẫu phù hợp từ thư mục prompts/ .
4. AI tạo bản nháp.
5. Nhóm đọc lại, sửa, rồi lưu vào đúng file bài tập.
AI chỉ hỗ trợ dựng bản nháp. Nhóm vẫn chịu trách nhiệm kiểm tra nguồn,
sửa nội dung, và chốt quyết định cuối.
Checklist trước khi nộp
worksheet/00-context.md đã điền đủ.
1-diverge.md có đủ Phần A, B, C.
2-converge.md có bảng gộp, bảng lọc trùng, bảng chấm rủi ro.
3-FINAL-test-set-eval-plan.md có 10-15 tình huống cuối và kế
hoạch chấm.
1-map-and-format.md có rủi ro được chọn, nguyên nhân gốc, 3 lớp giải
pháp.
artifact/1-uiux/ , artifact/2-prompt/ , artifact/3-
architecture/ đều có card.md và demo.* .
Repo công khai, tên đúng cú pháp Day25-MãNhóm , mở được.
README đầu repo có bảng mã học viên + tên đầy đủ 2-3 thành viên.
Link repo đã nộp qua LMS trước 23:59.
Lỗi hay mắc
Đừng làm Nên làm
Bỏ qua 00-context.md Điền bối cảnh trước khi dùng AI
Nộp mỗi file cuối Giữ cả file trung gian
AI viết xong là nộp Nhóm phải đọc, sửa, kiểm chứng
Chỉ làm một lớp giải pháp Làm đủ 3 lớp: giao diện, chỉ dẫn AI,
kiến trúc

| Đừng làm | Nên làm |
| --- | --- |

| Nộp repo private | Repo phải công khai |
| --- | --- |

Đừng làm Nên làm
Demo chỉ để nhìn đẹp Demo phải giúp người khác hiểu và
phản biện
Nộp repo private Repo phải công khai
Đặt tên repo Day-25-team-final Đúng cú pháp Day25-MãNhóm

