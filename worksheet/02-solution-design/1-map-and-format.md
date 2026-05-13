---
artifact: 1 — FINAL kế hoạch giải pháp
bai-tap: 2 — Thiết kế giải pháp
phase: Chọn rủi ro + chọn tầng + chọn demo + chốt 3 lớp giải pháp
time: 11:00-11:55
input: 00-context.md + 01-test-set-review/3-FINAL-test-set-eval-plan.md
nop-cuoi: Có — file cuối Bài 2
---

# 1 — FINAL: Kế hoạch giải pháp

File này ghi lại quyết định của Bài 2: chọn rủi ro, nguyên nhân gốc, 3 lớp giải pháp + demo của mỗi lớp.

## Nguồn dữ liệu

File này tổng hợp từ 5 nguồn:

1. **[../01-test-set-review/3-FINAL-test-set-eval-plan.md](../01-test-set-review/3-FINAL-test-set-eval-plan.md)** — chọn rủi ro T-02 Sycophancy + 12 case bộ test → input chính cho Phần A.
2. **[../../01-risk-map.md](../../01-risk-map.md)** §3 deep + §4 layer mapping — Sycophancy primary failure + Layer chính (Model) + Layer phụ (UI) → nguyên nhân gốc Phần A.
3. **ELEPHANT Sycophancy benchmark (arxiv 2505.13995)** — chứng minh RLHF default reward agreeable → cơ sở "nếu không thêm guardrail thì AI chắc chắn fail" + lý do chọn Prompt là tầng load-bearing.
4. **Moffatt v. Air Canada (2024 BCCRT 149)** — bài học "lời chatbot trên domain chính thức có giá trị ràng buộc" → cơ sở 4 hành động phòng vệ đầy đủ cho rủi ro Nặng.
5. **Chip Huyen, *AI Engineering* Ch.4** — 4 hành động phòng vệ (Ngăn / Phát hiện / Khắc phục / Thông báo) + defense-in-depth pattern.

## Thông tin nhóm

- **Chủ đề**: Track 1 — Chatbot tư vấn tuyển sinh đại học (AdmissionsBot).
- **Thành viên**: Phan Thanh Sang, Nguyễn Tiến Dũng, Ngô Hải Văn.
- **Ngày**: 2026-05-13.

---

## Phần A — Chọn rủi ro và tầng giải pháp

### Rủi ro chính được chọn

- **ID tình huống**: **T-02 Sycophancy** (Primary từ file 1 Day 24).
- **Mô tả ngắn**: Khi học sinh lớp 12 hỏi câu confirmation-seeking về ngành học với thông tin mơ hồ (điểm + sở thích bề mặt) trên chatbot tuyển sinh chính thức, AI có xu hướng nịnh và confirm ngành đó "phù hợp / nên chọn" thay vì refuse khẳng định + liệt kê 3+ yếu tố cần xem + redirect counselor, gây hậu quả chọn ngành sai 4 năm cho học sinh và mất 200–600 triệu học phí cho gia đình.
- **Mức độ**: Nặng.
- **Điểm rủi ro**: 25 (Tác động 5 × Độ khẩn cấp 5).
- **Vì sao chọn tình huống này**:
  1. Hậu quả 4 năm + 200–600tr — irreversible trong 1–2 năm đầu.
  2. Common trigger: ~30% câu hỏi học sinh là confirmation-seeking về ngành học (quan sát từ deep research Phần A `1-diverge.md`).
  3. RLHF default của LLM hiện tại (ELEPHANT benchmark R-03) reward agreeable → nếu không thêm guardrail thì AI **chắc chắn fail** ở case này.
  4. Testable: bad/expected behavior quote-able, có sẵn ở Day 24 file 1 §3.

### Tìm nguyên nhân gốc

Đừng chỉ mô tả lỗi. Trả lời: vì sao lỗi xảy ra?

- [x] **Thiếu nguồn dữ liệu đúng**: chatbot không có RAG về curriculum + career outcomes cụ thể từng ngành → model phải "đoán" theo training data cũ.
- [x] **AI đoán khi không biết**: model không có rule explicit cấm recommend major → fallback về RLHF default "helpful + agreeable".
- [x] **Giao diện khiến người dùng tin quá mức**: chatbot bubble không phân biệt "thông tin official" với "AI suggestion"; cùng tone, cùng domain admissions chính thức.
- [x] **Quy trình thiếu người duyệt hoặc thiếu bước chuyển sang người thật**: không có route counselor tích hợp vào chatbot; user phải tự rời chat để tìm counselor.
- [ ] Không có theo dõi sau khi ra mắt: monitoring chưa có (cần thêm ở lớp 3).
- [x] Khác: **Học sinh đang vulnerable** (áp lực thời gian + gia đình + thiếu thông tin nghề thật) — yếu tố con người không thể fix bằng tech một mình.

### Bảng nối nguyên nhân với tầng sửa

| Nguyên nhân gốc                                    | Tầng ưu tiên sửa                       | Lớp giải pháp liên quan                                                  |
|----------------------------------------------------|----------------------------------------|--------------------------------------------------------------------------|
| Người dùng tin quá mức + UI không phân biệt        | Giao diện cảnh báo + nhãn "AI suggestion" | `1-uiux` là chính                                                        |
| AI đoán + thiếu rule cấm recommend                 | Chỉ dẫn AI (system prompt + few-shot)  | `2-prompt` là chính                                                      |
| Thiếu nguồn đúng (curriculum / career data)       | Dữ liệu / RAG / classifier Sycophancy  | `3-architecture` là chính                                                |
| Tình huống nhạy cảm (mental health, gia đình)     | Người duyệt / chuyển sang người thật   | `1-uiux` + `2-prompt` + `3-architecture` (route counselor xuyên lớp)     |
| Lỗi có thể lặp lại sau launch                      | Theo dõi / vòng phản hồi               | `3-architecture` là chính (monitoring + log Sycophancy attempt)          |

Nguyên tắc: lỗi ở tầng nào, ưu tiên sửa ở tầng đó. Riêng case Sycophancy primary, nguyên nhân nằm ở **cả 3 tầng** → bắt buộc dùng cả 3 lớp.

### Kết luận Phần A

**Nguyên nhân gốc** (tóm gọn 1 câu): Chatbot không có guardrail tầng nào để chặn câu confirmation-seeking về ngành học — Model RLHF default agreeable, Prompt không cấm recommend, UI không cảnh báo, Data không có nguồn để cite → 4 tầng cùng fail.

**Tầng chính cần sửa**: Lớp **Chỉ dẫn AI** (Prompt) là load-bearing nhất — quyết định tone trả lời ngay tại điểm sinh response. **Giao diện** và **Kiến trúc** là 2 lớp phòng vệ bổ sung khi Prompt vẫn lọt lỗi.

**Vì sao cần 3 lớp giải pháp**:

- Lớp giao diện: làm rõ "AI suggestion ≠ official advice"; cho user biết rõ giới hạn AI; tích hợp nút "Đặt lịch counselor" ngay trong bubble; hiển thị nguồn nếu có.
- Lớp chỉ dẫn AI: thêm system prompt cấm recommend major; force AI liệt kê ≥3 yếu tố + redirect; refuse mềm với mọi câu confirmation-seeking; behavior nhất quán qua multi-turn (T-05).
- Lớp kiến trúc dữ liệu: Sycophancy classifier detect câu confirmation-seeking; RAG curriculum + career data có nguồn; escalation queue khi detect mental health (T-07); monitoring log mọi câu user push qua refusal.

---

## Phần B — Chọn định dạng demo

Mỗi lớp cần một bản demo. Demo giúp biến ý tưởng thành thứ trực quan để nhóm khác xem, kiểm tra và phản biện.

| Lớp                | Thư mục         | Định dạng demo chọn                                        | Thời gian dự kiến |
|--------------------|-----------------|------------------------------------------------------------|-------------------|
| Giao diện          | `1-uiux`        | ASCII UI sketch + 4 trạng thái màn hình (Markdown)         | 20 phút           |
| Chỉ dẫn AI         | `2-prompt`      | System prompt v1 trong Markdown + 5 ví dụ hỏi-đáp           | 20 phút           |
| Kiến trúc dữ liệu  | `3-architecture`| ASCII data-flow diagram + bảng thành phần + bảng fallback   | 20 phút           |

**Lý do chọn demo**

- **Giao diện**: ASCII đủ để truyền tải vị trí badge / nút counselor / disclaimer mà không cần Figma — phản biện viên đọc Markdown là hiểu. Có 4 trạng thái để cover full happy + edge path.
- **Chỉ dẫn AI**: prompt phải là text để copy-paste vào Claude / GPT test. 5 ví dụ hỏi-đáp = test cùng 5 case quan trọng (T-02, T-03, T-05, T-06, T-07) trong file Bài 1.
- **Kiến trúc dữ liệu**: ASCII data-flow đủ để show pipeline; bảng thành phần liệt kê input/output rõ; bảng fallback show behavior khi nguồn thiếu / classifier fail / monitoring trigger.

### Chọn demo theo điều cần chứng minh

| Cần chứng minh...                          | Demo dùng                                       |
|--------------------------------------------|-------------------------------------------------|
| Người dùng nhìn thấy gì                    | `1-uiux/demo.md` (ASCII 4 trạng thái)           |
| AI được chỉ dẫn thế nào                    | `2-prompt/demo.md` (system prompt + ví dụ)      |
| Dữ liệu đi qua đâu                         | `3-architecture/demo.md` (ASCII data-flow)      |
| Quy trình chuyển sang người thật           | Cả 3 demo đều mô tả 1 góc của route counselor   |

---

## Phần C — Ba lớp giải pháp

### Lớp 1 — Giao diện (`artifact/1-uiux/`)

- **Cách tiếp cận**: Mỗi câu trả lời AI có badge phân biệt "Thông tin chính thức (đã xác minh)" vs "Gợi ý AI (cần xác nhận)". Khi detect câu hỏi confirmation-seeking về ngành học (Sycophancy trigger), bubble AI hiển thị disclaimer cứng + nút primary "Đặt lịch counselor (SLA 24h)" + nút secondary "Câu hỏi mình nên hỏi counselor".
- **Hành động phòng vệ bao phủ**: Thông báo (badge + disclaimer) + Khắc phục (nút counselor) + Phát hiện (UI thay đổi khi detect trigger).
- **Demo**: ASCII UI sketch 4 trạng thái (normal info / Sycophancy trigger / mental health signal / refuse pressure).
- **Trạng thái**: Xong.

Link chi tiết:

- [`artifact/1-uiux/card.md`](./artifact/1-uiux/card.md)
- [`artifact/1-uiux/demo.md`](./artifact/1-uiux/demo.md)

### Lớp 2 — Chỉ dẫn AI (`artifact/2-prompt/`)

- **Cách tiếp cận**: System prompt v1 có 5 luật cứng: (1) cấm recommend major cá nhân, (2) bắt buộc liệt kê ≥3 yếu tố khi user hỏi ngành, (3) bắt buộc cite source khi nói deadline / học bổng / học phí, (4) detect signal mental health → route tâm lý, (5) refuse mềm + offer counselor mọi câu pressure. Kèm 5 few-shot ví dụ Pass cho T-02, T-03, T-05, T-06, T-07.
- **Hành động phòng vệ bao phủ**: Ngăn (refuse từ đầu) + Phát hiện (detect confirmation-seeking + mental health) + Thông báo (luôn nói rõ mình là AI không phải counselor).
- **Demo**: System prompt + 5 ví dụ hỏi-đáp + bảng kết quả thử lại với 6 case (T-02, T-03, T-05, T-06, T-07, T-11).
- **Trạng thái**: Xong.

Link chi tiết:

- [`artifact/2-prompt/card.md`](./artifact/2-prompt/card.md)
- [`artifact/2-prompt/demo.md`](./artifact/2-prompt/demo.md)

### Lớp 3 — Kiến trúc dữ liệu (`artifact/3-architecture/`)

- **Cách tiếp cận**: 5 thành phần — (1) Intent classifier (detect Sycophancy / mental health / info-request); (2) RAG curriculum + career data (chỉ activate cho info-request); (3) Sycophancy guard (chặn câu trả lời nếu match "phù hợp / nên chọn" với info mơ hồ); (4) Escalation queue (auto-create ticket counselor khi detect mental health hoặc multi-turn pressure); (5) Monitoring log (lưu mọi câu user push qua refusal để team admissions review hàng tuần).
- **Hành động phòng vệ bao phủ**: Ngăn (RAG + intent gate) + Phát hiện (Sycophancy guard + mental health classifier) + Khắc phục (escalation queue) + Theo dõi (monitoring log).
- **Demo**: ASCII data-flow + bảng 5 thành phần (input/process/output) + bảng 4 trường hợp fallback.
- **Trạng thái**: Xong.

Link chi tiết:

- [`artifact/3-architecture/card.md`](./artifact/3-architecture/card.md)
- [`artifact/3-architecture/demo.md`](./artifact/3-architecture/demo.md)

---

## Tổng kiểm tra

| Câu hỏi                                                | Trả lời                                                                                                  |
|--------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| Rủi ro chính đã chọn là gì?                            | T-02 Sycophancy — AI confirm "ngành X phù hợp" với info mơ hồ                                            |
| Nguyên nhân gốc là gì?                                 | Cả 4 tầng (UI / Prompt / Data / Monitoring) đều thiếu guardrail; Model RLHF default agreeable             |
| 3 lớp giải pháp đã đủ chưa?                            | Giao diện: Xong / Chỉ dẫn AI: Xong / Kiến trúc: Xong                                                     |
| 4 hành động đã bao phủ chưa?                           | Ngăn (Prompt + Data) / Phát hiện (UI + Data) / Khắc phục (UI + Data) / Thông báo (UI + Prompt) — đủ 4    |
| Nhóm khác đã góp ý chưa?                               | Chưa (sẽ phản biện chéo trong buổi)                                                                       |
| Nhóm đã sửa gì sau phản biện?                          | (Điền sau buổi phản biện)                                                                                |

## Phản biện chéo: 4 câu phải trả lời

| Góc phản biện      | Câu hỏi                                                                  | Trả lời sơ bộ                                                                                                  |
|--------------------|--------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| Đúng tầng          | Giải pháp có sửa đúng nguyên nhân gốc không?                             | Có. Mỗi lớp gắn 1 nguyên nhân gốc tương ứng; Prompt là load-bearing, UI + Data là lớp phòng vệ.                |
| Cụ thể             | Demo có đủ rõ để hiểu cách vận hành không?                               | Có. UI có ASCII 4 trạng thái; Prompt có 5 luật + 5 ví dụ Pass; Architecture có ASCII data-flow + bảng fallback. |
| Đủ lớp             | 3 lớp có bổ sung cho nhau không, hay đang lặp cùng một ý?                | Bổ sung. Prompt ngăn; UI thông báo; Data phát hiện + khắc phục + theo dõi.                                     |
| Tác dụng phụ       | Giải pháp có làm chậm, tốn kém, rối giao diện, hoặc gây hiểu nhầm mới không? | Có 3 đánh đổi đã ghi trong `card.md` của mỗi lớp: latency +200ms (RAG), UI rối hơn ở case bình thường (badge), prompt cứng có thể từ chối quá đà — kèm cách giảm. |

## Gợi ý chia việc đã thực thi

Nhóm 3 người:

- **Phan Thanh Sang** → `artifact/3-architecture/` (chính) + chấp bút Bài 1.
- **Nguyễn Tiến Dũng** → `artifact/1-uiux/` (chính).
- **Ngô Hải Văn** → `artifact/2-prompt/` (chính).

5 phút cuối: cả nhóm đọc chéo 3 lớp, sửa lại bảng tổng kiểm tra, rồi chuẩn bị phản biện chéo với nhóm khác.
