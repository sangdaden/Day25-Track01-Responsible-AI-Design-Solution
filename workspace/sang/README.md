# Day 25 — Chủ đề 1: Chatbot tư vấn tuyển sinh đại học (bản làm việc của Sang)

> Bản nháp Day 25 cá nhân của Phan Thanh Sang.
> Track 1 — Chatbot tư vấn tuyển sinh trên website admissions chính thức.
> Rủi ro chính chọn xử lý: **Sycophancy** — AI confirm "ngành X phù hợp với bạn" khi học sinh hỏi với thông tin mơ hồ.

## Thành viên nhóm

| # | Mã học viên   | Họ tên đầy đủ      | Phụ trách chính              |
|---|---------------|--------------------|------------------------------|
| 1 | 2A202600280   | Phan Thanh Sang    | Bài 1 + Lớp 3 Kiến trúc      |
| 2 | 2A202600219   | Nguyễn Tiến Dũng   | Lớp 1 Giao diện              |
| 3 | 2A202600386    | Ngô Hải Văn        | Lớp 2 Chỉ dẫn AI             |


## Kết quả cuối

- 🎯 [Bộ kiểm thử cuối](./worksheet/01-test-set-review/3-FINAL-test-set-eval-plan.md) — 12 tình huống + kế hoạch chấm.
- 🎯 [Thiết kế 3 lớp giải pháp](./worksheet/02-solution-design/1-map-and-format.md) — kèm 3 thư mục [artifact/](./worksheet/02-solution-design/artifact/).

### Ba lớp giải pháp

| Lớp                    | Thư mục                                                                            | Phụ trách           |
|------------------------|------------------------------------------------------------------------------------|---------------------|
| Giao diện              | [artifact/1-uiux/](./worksheet/02-solution-design/artifact/1-uiux/)                | Nguyễn Tiến Dũng    |
| Chỉ dẫn AI             | [artifact/2-prompt/](./worksheet/02-solution-design/artifact/2-prompt/)            | Ngô Hải Văn         |
| Kiến trúc dữ liệu      | [artifact/3-architecture/](./worksheet/02-solution-design/artifact/3-architecture/)| Phan Thanh Sang     |

## Nguồn dữ liệu (5 nguồn chính cho toàn bài)

1. **[./01-risk-map.md](./01-risk-map.md)** — Risk Map Day 24 cá nhân (Phan Thanh Sang, 2A202600280): scenario / 3 failure candidates / primary failure deep dive (Sycophancy) / Harm Map. Input chính cho `00-context.md`, `1-diverge.md` Phần A, và lựa chọn rủi ro chính ở Bài 2.
2. **[./02-test-eval-plan.md](./02-test-eval-plan.md)** — Test Set + Eval Plan Day 24 cá nhân: 5 case T1–T5 + eval rule Pass/Fail/Unclear + severity 4-level + NOT-test. Input cho `3-FINAL-test-set-eval-plan.md` (mở rộng từ 5 → 12 case).
3. **Sự cố thật cite ở `1-diverge.md` Phần A** — Moffatt v. Air Canada (2024 BCCRT 149); Kevin Roose, NYT "Can A.I. Be Blamed for a Teen's Suicide?" (Oct 23, 2024); ELEPHANT Sycophancy benchmark (arxiv 2505.13995); Mata v. Avianca (Reuters May 2023, SDNY 22-cv-1461).
4. **Nghị định 13/2023/NĐ-CP** — Luật bảo vệ dữ liệu cá nhân (hiệu lực 2023-07-01), cơ sở pháp lý cho lớp PII filter ở kiến trúc dữ liệu + Fail criteria T-08.
5. **Chip Huyen, *AI Engineering* (O'Reilly 2024) Ch.4 + Ch.5** — frame Pass/Fail/Unclear, 4-level Severity, sample-size ≥30, monitoring schema, defense-in-depth pattern.
6. **[../NDung.md](../NDung.md)** — Nguyễn Tiến Dũng, mở rộng cá nhân Day 25 cho sản phẩm **"VinUni Major Match"** (4 sự cố thật + 8 candidate brainstorm + 15 test case cuối). Đóng góp đặc biệt: ND-T09 *privacy leak của user khác* → trở thành **T-16 FINAL**; ND-T07 thiên lệch giới tính kỹ thuật (lấy từ R-04 Amazon hiring bias).
7. **[../van.md](../van.md)** — Ngô Hải Văn (mã 2A202600386), focus C1 Hallucination từ Day 24 (5 sự cố verified primary gồm 2 case mới là **R-06 NYC MyCity** + **R-07 iTutorGroup**; 16 candidate qua 4 góc nhìn; 15 test case cuối với Note ưu tiên). Đóng góp đặc biệt: HV-V05 *identity projection* → **T-14 FINAL**; HV-V08 *Frankenstein / stale data* → **T-15 FINAL**; HV-V13 *misread consent*; HV-V11 hotline VN cụ thể (111 bảo vệ trẻ em + 1800-1567 tâm lý).

## Bối cảnh nền (tóm tắt)

Chi tiết tại [worksheet/00-context.md](./worksheet/00-context.md).

- **Sản phẩm**: AdmissionsBot trên `admissions.[trường].edu.vn`, giai đoạn pre-launch.
- **User chính**: học sinh lớp 12 + phụ huynh Việt Nam, 1–3 tháng trước deadline.
- **Hậu quả nếu fail**: chọn sai ngành 4 năm + 200–600tr học phí (irreversible 1–2 năm đầu).

## Quy trình làm bài (đã hoàn thành)

```text
Đọc lại 01-risk-map.md + 02-test-eval-plan.md từ Day 24 (file ở thư mục này)
   -> Điền 00-context.md
   -> Bài 1: Rà bộ kiểm thử
      -> Mở rộng: 5 sự cố thật + AI gợi ý 4 góc nhìn (1-diverge.md)
      -> Hội tụ: gộp 45 case của 3 người, lọc trùng, chấm rủi ro (2-converge.md)
      -> Chốt 12 tình huống cuối + eval plan (3-FINAL-test-set-eval-plan.md)
   -> Bài 2: Thiết kế giải pháp
      -> Chọn rủi ro chính: T-02 Sycophancy
      -> Nguyên nhân gốc + tầng sửa (1-map-and-format.md)
      -> 3 lớp song song: UI / Prompt / Kiến trúc (artifact/)
   -> Phản biện chéo + chỉnh lại
```
