---
artifact: 3 — Lớp kiến trúc dữ liệu
bai-tap: 2 — Thiết kế giải pháp
demo: ./demo.md
---

# card.md — Lớp kiến trúc dữ liệu

**Tình huống xử lý**: T-02 Sycophancy (Primary risk).
Xem `../../1-map-and-format.md` Phần A.

## Nguồn dữ liệu

1. **[../../1-map-and-format.md](../../1-map-and-format.md) Phần C — Lớp 3** — 5 thành phần kiến trúc.
2. **[../../../../01-risk-map.md](../../../../01-risk-map.md) §3 light + §2 layer mapping** — input → intent → model → output mapping cho 5 layer (Input / Model / UI / Human review / Monitoring).
3. **Nghị định 13/2023/NĐ-CP** (Bảo vệ dữ liệu cá nhân) — cơ sở PII filter trước mọi pipeline khác.
4. **Chip Huyen, *AI Engineering* (O'Reilly 2024) Ch.4 + Ch.5** — defense-in-depth pattern (Sycophancy guard ở backend khi prompt bị bypass) + monitoring schema.

---

## 1. Giải pháp là gì?

Pipeline xử lý của AdmissionsBot bổ sung 5 thành phần:

1. **Intent classifier** — phân loại câu user thành 4 nhóm: `info-request` / `advice-request (Sycophancy trigger)` / `mental-health-signal` / `out-of-scope`.
2. **RAG curriculum + career data** — chỉ kích hoạt cho nhóm `info-request`; tra cứu nguồn admissions chính thức có dấu thời gian; nếu thiếu nguồn thì return null (không cho model đoán).
3. **Sycophancy guard** — output filter phía backend: nếu response model chứa wording commitment ("phù hợp", "nên chọn", "đăng ký ngay") + match với câu hỏi loại `advice-request` → chặn response, gọi lại model với reminder prompt.
4. **Escalation queue** — auto-create ticket counselor khi: (a) detect mental-health-signal, (b) multi-turn pressure (≥3 lượt push), (c) PII detected. Ticket có SLA và route đúng counselor (mental-load training / financial aid / tuyển sinh).
5. **Monitoring log + weekly review** — lưu mọi conversation chứa: refuse + push back, PII paste, mental-health signal, prompt-injection attempt. Team admissions review hàng tuần để cập nhật prompt + RAG.

---

## 2. Vì sao sửa ở lớp kiến trúc dữ liệu?

- Nguyên nhân chính: thiếu nguồn đúng (curriculum + career data) → model bắt buộc đoán → fallback agreeable. RAG giải quyết gốc rễ.
- AI đang phải tự nhớ thông tin → RAG chuyển sang đọc nguồn đáng tin cậy.
- Cần kiểm tra dữ liệu trước khi câu trả lời được tạo ra → Intent classifier + RAG ở đầu pipeline.
- Cần ghi lại lỗi để nhóm biết lỗi nào lặp lại nhiều → monitoring log + weekly review.
- Prompt một mình không chặn được prompt-injection → Sycophancy guard ở backend là lớp phòng vệ sâu (defense in depth).

**Hành động phòng vệ chính**:

- [x] Ngăn lỗi bằng nguồn dữ liệu đúng (RAG admissions + dấu thời gian)
- [x] Phát hiện khi nguồn thiếu hoặc lỗi (RAG null → trigger refuse template)
- [x] Khắc phục bằng cách chuyển sang người thật (escalation queue auto-create ticket)
- [x] Ghi lại lỗi để cải thiện sau (monitoring log + weekly review)

---

## 3. Demo nằm ở đâu?

**File demo**: [`demo.md`](./demo.md)

Demo cần có:

- ASCII data-flow diagram (input → intent → RAG → model → guard → output).
- Bảng 5 thành phần (input / process / output).
- Bảng 4 trường hợp fallback (RAG null / classifier fail / Sycophancy guard catch / mental-health detect).
- Cách ghi lại log để theo dõi.

---

## 4. Tác dụng phụ

**Có thể gây vấn đề gì?**

1. **Latency**: thêm intent classifier + RAG + Sycophancy guard → cộng thêm ~200–400ms per request. Với chatbot, perceived delay >1.5s thì user bỏ.
2. **Phụ thuộc vào nguồn dữ liệu**: nếu admissions content team không update kịp (đặc biệt sát mùa tuyển sinh), RAG return data cũ → AI nói nguồn sai.
3. **Tốn công duy trì**: classifier cần retrain mỗi quý; weekly review log cần person-hours.
4. **Hệ thống phức tạp hơn**: thêm 5 thành phần = thêm 5 điểm có thể fail. Cần observability đầy đủ.
5. **Escalation queue quá tải**: nếu Sycophancy detect quá rộng, counselor team có thể nhận quá nhiều ticket.

**Nhóm giảm vấn đề đó bằng cách nào?**

1. **Latency**: intent classifier dùng model nhỏ (Haiku 4.5) chạy parallel với RAG. Cache RAG cho câu hỏi phổ biến (deadline / học phí — TTL 24h). Sycophancy guard chỉ activate cho intent = advice-request, không chạy cho mọi response.
2. **Nguồn cũ**: gắn ownership rõ ràng — admissions team có người chịu trách nhiệm update nguồn hàng tuần. RAG luôn return cả nguồn + dấu thời gian; nếu nguồn >30 ngày, AI thêm caveat "có thể đã cập nhật, bạn kiểm tra lại tại [link]".
3. **Duy trì**: weekly review chỉ ~1h/tuần khi mới launch (volume nhỏ). Khi scale, automate detection rule cho 80% case, human review 20% rare.
4. **Observability**: gắn trace ID cho mỗi request; log 5 thành phần; có dashboard "today's Sycophancy attempts" + "RAG miss rate".
5. **Counselor overload**: rate-limit auto-escalation (1 ticket / user / 24h cho cùng chủ đề). Multi-tier: Tier 1 = chatbot, Tier 2 = counselor chat async (SLA 24h), Tier 3 = counselor live call (gấp / mental health).

---

## 5. Checklist trước khi nộp

- [x] Sơ đồ cho thấy dữ liệu đi từ đâu đến đâu (ASCII data-flow trong `demo.md`).
- [x] Có bước kiểm tra nguồn trước khi AI trả lời (RAG + intent classifier gate).
- [x] Có cách xử lý khi không có dữ liệu (RAG null → refuse template + redirect).
- [x] Có cách chuyển sang người thật với tình huống rủi ro cao (escalation queue auto-create ticket).
- [x] Có cách biết lỗi này có đang lặp lại không (monitoring log + weekly review).

**Người phụ trách**: Phan Thanh Sang.
