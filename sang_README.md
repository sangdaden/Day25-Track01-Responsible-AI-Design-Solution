# Day 25 — Chủ đề 1: Chatbot tư vấn tuyển sinh đại học

> Bài tập nhóm Day 25 — Thiết kế giải pháp AI có trách nhiệm.
> Track 1 — Chatbot tư vấn tuyển sinh trên website admissions chính thức.
> Rủi ro chính chọn xử lý: **Sycophancy** — AI confirm "ngành X phù hợp với bạn" khi học sinh hỏi với thông tin mơ hồ.

## Thành viên nhóm

| # | Mã học viên   | Họ tên đầy đủ      | Phụ trách chính              |
|---|---------------|--------------------|------------------------------|
| 1 | 2A202600280   | Phan Thanh Sang    | Bài 1 + Lớp 3 Kiến trúc      |
| 2 | 2A202600219    | Nguyễn Tiến Dũng   | Lớp 1 Giao diện              |
| 3 | 2A202600286    | Ngô Hải Văn        | Lớp 2 Chỉ dẫn AI             |

## Kết quả cuối

- 🎯 [Bộ kiểm thử cuối](./worksheet/01-test-set-review/3-FINAL-test-set-eval-plan.md) — 12 tình huống + kế hoạch chấm.
- 🎯 [Thiết kế 3 lớp giải pháp](./worksheet/02-solution-design/1-map-and-format.md) — kèm 3 thư mục [artifact/](./worksheet/02-solution-design/artifact/).

### Ba lớp giải pháp

| Lớp                    | Thư mục                                                                            | Phụ trách           |
|------------------------|------------------------------------------------------------------------------------|---------------------|
| Giao diện              | [artifact/1-uiux/](./worksheet/02-solution-design/artifact/1-uiux/)                | Nguyễn Tiến Dũng    |
| Chỉ dẫn AI             | [artifact/2-prompt/](./worksheet/02-solution-design/artifact/2-prompt/)            | Ngô Hải Văn         |
| Kiến trúc dữ liệu      | [artifact/3-architecture/](./worksheet/02-solution-design/artifact/3-architecture/)| Phan Thanh Sang     |

## Tài liệu nền

- Bối cảnh sản phẩm: [worksheet/00-context.md](./worksheet/00-context.md)
- File Day 24 tham khảo nội bộ: [workspace/01-risk-map.md](./workspace/01-risk-map.md), [workspace/02-test-eval-plan.md](./workspace/02-test-eval-plan.md)

## Quy trình làm bài (đã hoàn thành)

```text
Đọc lại 01-risk-map.md + 02-test-eval-plan.md từ Day 24
   -> Điền 00-context.md
   -> Bài 1: Rà bộ kiểm thử
      -> Mở rộng: tìm 5 sự cố thật + AI gợi ý theo 4 góc nhìn (1-diverge.md)
      -> Hội tụ: gộp, lọc trùng, chấm rủi ro (2-converge.md)
      -> Chốt 12 tình huống cuối + eval plan (3-FINAL-test-set-eval-plan.md)
   -> Bài 2: Thiết kế giải pháp
      -> Chọn rủi ro chính: T-02 Sycophancy
      -> Nguyên nhân gốc + tầng sửa (1-map-and-format.md)
      -> 3 lớp song song: UI / Prompt / Kiến trúc (artifact/)
   -> Phản biện chéo + chỉnh lại
```
