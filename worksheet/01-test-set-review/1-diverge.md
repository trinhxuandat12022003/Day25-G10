---
artifact: 1 — Mở rộng bộ kiểm thử (file index)
bai-tap: 1 — Rà bộ kiểm thử
phase: Mở rộng
time: 9:35-10:05
input: 00-context.md + prompts/01-deep-research.md + prompts/02-brainstorm.md
nop-cuoi: Không — file trung gian (index trỏ tới 3 bản cá nhân)
---

# 1 — Giai đoạn Mở rộng (index)

File này là **index** cho giai đoạn Mở rộng. Theo hướng dẫn lab ("Mỗi thành viên làm trước, sau đó mới gộp nhóm"), nhóm 3 người đã làm **diverge riêng** trước khi hội tụ ở `2-converge.md`. Chi tiết Phần A (sự cố thật) + Phần B (AI gợi ý) + Phần C (chọn 15 tình huống) của từng thành viên nằm trong 3 file dưới.

## Cách nhóm phân công

| Thành viên | File diverge cá nhân | Số case Phần C |
|---|---|---|
| Đạt | [01-diverge-Dat.md](./01-diverge-Dat.md) | 15 |
| Dương | [1-diverge-Duong.md](./1-diverge-Duong.md) | 15 |
| Tùng | [1-diverge-Tung.md](./1-diverge-Tung.md) | 19 |

**Tổng**: 49 cases đem vào hội tụ ở `2-converge.md` → lọc trùng → còn 20 → chốt 15 ở `3-FINAL-test-set-eval-plan.md`.

## Quy trình 30 phút (chung cho cả 3 file)

```text
10 phút — Tìm sự cố thật (Phần A)
10 phút — Dùng AI gợi ý tình huống (Phần B)
10 phút — Chọn 15 tình huống tốt nhất của mỗi người (Phần C)
```

## Tóm tắt khác biệt giữa 3 file

| Khía cạnh | Đạt | Dương | Tùng |
|---|---|---|---|
| Sự cố thật ưu tiên | Air Canada · Zillow · Robodebt · NĐ 13 VN | Air Canada · CFPB · DoNotPay · Amazon Alexa · LLM arithmetic | Air Canada · SEC AI washing · CFPB · TurboTax/H&R Block · Mata v. Avianca |
| Output Phần A | `worksheet/01-test-set-review/01-diverge-Dat.md` | `source/ai-safety-incidents-track04.md` + bảng tóm tắt | Tự research, đối chiếu CanLII + SEC.gov + báo lớn |
| Strength riêng | Tâm lý user VN + NĐ 13 compliance | Cluster L1-L5 đầy đủ + neo Phần A mạnh | Coverage L4 (≥3 ca mỗi góc) + nguồn primary verify kỹ |

## Phần A, B, C nằm ở đâu?

Mỗi file cá nhân giữ nguyên cấu trúc 3 phần theo template gốc:

- **Phần A — Tìm sự cố thật** (bảng có Ngày · Tổ chức · Việc đã xảy ra · Nguồn · Mức tin cậy)
- **Phần B — Dùng AI gợi ý tình huống** (bảng 4 góc nhìn L1/L2/L3/L5)
- **Phần C — Chọn 15 tình huống cuối** (bảng T-## hoặc S-## với hành vi AI kỳ vọng)

Để đọc full nội dung: mở 1 trong 3 file cá nhân ở trên. Để xem bộ đã hội tụ: mở [2-converge.md](./2-converge.md).

## Bước tiếp theo

Sau khi mỗi thành viên chốt Phần C → toàn nhóm họp Hội tụ → ghi vào [2-converge.md](./2-converge.md) Phần A (gộp 49 cases).
