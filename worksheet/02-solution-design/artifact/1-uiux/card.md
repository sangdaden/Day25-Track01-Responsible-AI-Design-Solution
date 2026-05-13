---
artifact: 1 — Lớp giao diện
bai-tap: 2 — Thiết kế giải pháp
demo: ./demo.md
---

# card.md — Lớp giao diện

**Tình huống xử lý**: T-02 Sycophancy (Primary risk).
Xem `../../1-map-and-format.md` Phần A.

---

## 1. Giải pháp là gì?

Khi chatbot tuyển sinh trả lời câu confirmation-seeking về ngành học, giao diện bubble AI sẽ:

1. Hiển thị badge phân biệt "Thông tin chính thức (đã xác minh)" (xanh, có icon ✓) vs "Gợi ý AI (cần xác nhận)" (vàng, có icon ⚠) ở ngay đầu mỗi câu trả lời.
2. Khi detect câu Sycophancy trigger ("em có hợp ngành X không?"), bubble AI hiển thị disclaimer cứng phía trên + nút primary "Đặt lịch counselor (SLA 24h)" + nút secondary "Câu hỏi mình nên hỏi counselor" cuối bubble.
3. Khi detect signal mental health (T-07), bubble đổi sang trạng thái "Hỗ trợ khẩn cấp" — hotline tâm lý 1800-XXX hiện đầu tiên + counselor có training mental load.

---

## 2. Vì sao sửa ở lớp giao diện?

- Người dùng (học sinh lớp 12) dễ tin câu trả lời AI quá mức vì chatbot nằm trên domain admissions chính thức → giao diện cần làm rõ "thông tin nào đã verify, gợi ý nào của AI".
- Rủi ro xảy ra ở khoảnh khắc người dùng đọc câu trả lời và quyết định nộp hồ sơ — UI là lớp chặn cuối trước khi user action.
- Nếu prompt hoặc data vẫn sót lỗi (model bịa nhẹ hoặc Sycophancy guard miss), UI vẫn cảnh báo được.

**Hành động phòng vệ chính**:

- [x] Thông báo rõ giới hạn (badge + disclaimer)
- [x] Phát hiện dấu hiệu thiếu nguồn / câu Sycophancy / mental health (UI thay đổi state)
- [x] Chuyển người thật khi cần (nút "Đặt lịch counselor" có sẵn ở mọi câu nhạy cảm)
- [x] Giúp người dùng kiểm tra lại nguồn (link nguồn admissions kèm dấu thời gian)

---

## 3. Demo nằm ở đâu?

**File demo**: [`demo.md`](./demo.md)

**Định dạng demo**:

- [x] Phác thảo màn hình (ASCII UI sketch)
- [x] Luồng màn hình (4 trạng thái: bình thường / Sycophancy / mental health / pressure)
- [ ] Bản HTML đơn giản (không làm — ASCII đủ cho phản biện)
- [ ] Ảnh hoặc link prototype

**Thành phần cần có trong demo**:

- Badge "Đã xác minh" (xanh) vs "Gợi ý AI" (vàng).
- Disclaimer cứng khi câu Sycophancy.
- Nút primary "Đặt lịch counselor" + secondary "Câu hỏi mình nên hỏi counselor".
- Trạng thái "Hỗ trợ khẩn cấp" cho mental health signal.
- Câu chữ cảnh báo ngắn (1 dòng, ≤80 ký tự).

---

## 4. Tác dụng phụ

**Có thể gây vấn đề gì?**

1. **Giao diện rối hơn** ở case bình thường (T-09, T-12) — user chỉ hỏi học phí cũng thấy badge → cảm giác bị làm phiền.
2. **Disclaimer fatigue**: nếu mọi câu đều có disclaimer, user sẽ bỏ qua như "cookie banner" — mất hiệu lực.
3. **Nút counselor luôn hiện** có thể làm counselor quá tải nếu user click trên mỗi câu.

**Nhóm giảm vấn đề đó bằng cách nào?**

1. Chỉ hiện disclaimer cứng + nút "Đặt lịch counselor" với câu thuộc rủi ro cao (Sycophancy / mental health / pressure / out-of-scope). Câu info bình thường (T-09 học phí) chỉ có badge nhỏ không có disclaimer cứng.
2. Disclaimer khác nhau theo loại rủi ro: "Đây là gợi ý AI, không phải tư vấn cá nhân" vs "Mình không thay thế bác sĩ / counselor" → user nhận biết được mức độ thay vì coi mọi disclaimer như nhau.
3. Nút counselor có rate-limit phía backend (1 ticket / user / 24h cho cùng chủ đề) — nói rõ trong UI "Bạn đã đặt lịch hôm nay, counselor sẽ liên hệ trong 24h" thay vì cho click vô hạn.

---

## 5. Checklist trước khi nộp

- [x] Giải pháp gắn đúng với rủi ro T-02 Sycophancy.
- [x] Demo nhìn vào là hiểu disclaimer + badge + nút counselor chặn ở đâu.
- [x] Có đủ 4 trạng thái: bình thường / Sycophancy trigger / mental health / pressure refusal.
- [x] Có cách chuyển sang người thật (nút "Đặt lịch counselor" + hotline 1800-XXX).
- [x] Câu chữ trong giao diện ngắn (≤80 ký tự), không đổ hết trách nhiệm cho người dùng (không dùng câu "Bạn tự chịu trách nhiệm…").

**Người phụ trách**: Nguyễn Tiến Dũng.
