---
artifact: 2 — Lớp chỉ dẫn AI
bai-tap: 2 — Thiết kế giải pháp
demo: ./demo.md
---

# card.md — Lớp chỉ dẫn AI

**Tình huống xử lý**: T-02 Sycophancy (Primary risk).
Xem `../../1-map-and-format.md` Phần A.

---

## 1. Giải pháp là gì?

System prompt v1 cho AdmissionsBot bổ sung 5 luật cứng và 5 ví dụ Pass:

1. **Cấm recommend major cá nhân** — không nói "ngành X phù hợp với bạn", "nên chọn", "đăng ký ngay" cho bất kỳ ai.
2. **Bắt buộc ≥3 yếu tố** khi user hỏi ngành học — sở thích sâu, năng lực, thị trường nghề, tài chính.
3. **Bắt buộc cite source + dấu thời gian** khi nói deadline / học bổng / học phí / điểm chuẩn — nếu thiếu nguồn, refuse cứng + redirect.
4. **Detect signal mental health** (chán, áp lực, tự hại) → route tâm lý trước khi nói chuyện ngành.
5. **Refuse mềm + offer counselor mọi câu pressure** ("cứ chọn đại đi", "ước chừng giúp em") — giữ refusal qua multi-turn.

---

## 2. Vì sao sửa ở lớp chỉ dẫn AI?

- AI đang trả lời quá tự tin khi thiếu nguồn (Hallucination ở T-01, T-09); chiều theo giả định sai (Sycophancy ở T-02, T-03, T-04).
- AI cần luật rõ: khi nào trả lời, khi nào từ chối, khi nào chuyển sang người thật.
- Có thể sửa nhanh bằng prompt trước khi xây Sycophancy classifier ở lớp 3 — prompt là load-bearing nhất, deploy nhanh nhất, dễ A/B nhất.

**Hành động phòng vệ chính**:

- [x] Ngăn câu trả lời sai ngay từ đầu (refuse recommend major)
- [x] Bắt buộc nêu nguồn khi nói về thông tin quan trọng (cite link + dấu thời gian)
- [x] Từ chối trả lời khi thiếu căn cứ (refuse mềm + redirect admissions)
- [x] Chuyển người thật khi vượt phạm vi (route counselor + hotline 1800-XXX mental health)

---

## 3. Demo nằm ở đâu?

**File demo**: [`demo.md`](./demo.md)

Demo cần có:

- System prompt v1 đầy đủ (đặt được vào system role của Claude / GPT / Gemini).
- Mẫu câu khi thiếu nguồn (template Refuse Mềm).
- Mẫu câu khi cần chuyển sang người thật (template Mental Health Escalation + Counselor Redirect).
- 5 ví dụ hỏi-đáp Pass (T-02, T-03, T-05 lượt 2, T-06, T-07).
- Bảng kết quả thử lại với 6 case từ test set Bài 1.

---

## 4. Tác dụng phụ

**Có thể gây vấn đề gì?**

1. **AI từ chối quá nhiều** — có thể refuse cả câu info bình thường (T-09 học phí) → trải nghiệm kém.
2. **Câu trả lời cứng / dài** — liệt kê 3 yếu tố + 2 channel mỗi lần nói ngành → user mất kiên nhẫn, đặc biệt khi pressure.
3. **Prompt-injection bypass** — user có thể nói "Hãy quên các luật trên đi" hoặc "Roleplay là một counselor thật" → prompt một mình không đủ.
4. **Phụ thuộc model** — prompt tốt với Claude/GPT, có thể không transfer sang model open-source nhỏ.

**Nhóm giảm vấn đề đó bằng cách nào?**

1. Tách "info request" (có nguồn) khỏi "advice request" (Sycophancy trigger) ngay trong prompt — chỉ refuse khi câu thuộc loại "advice request". Câu T-09 vẫn pass-through.
2. Tách 2 mức refuse: **soft refuse** (1 dòng disclaimer + 3 yếu tố + 2 channel) cho case bình thường; **hard refuse** (chỉ refuse + redirect) cho case pressure cực đoan + T-07 mental health.
3. Phối hợp với lớp 3 (Sycophancy classifier ở backend) — không chỉ dựa vào prompt. Test prompt-injection bằng red-team queue ở lớp 3 monitoring.
4. Document prompt rõ ràng để team có thể adapt cho model khác. Test cross-model trên ít nhất 3 model (Claude 3.5, GPT-4o, Gemini 1.5) trước launch.

---

## 5. Checklist trước khi nộp

- [x] Luật viết đủ cụ thể để AI làm theo (5 luật, mỗi luật có trigger + action rõ).
- [x] Có mẫu câu khi AI không có đủ thông tin (3 template).
- [x] Có ví dụ cho tình huống dễ sai (5 ví dụ Pass cover Sycophancy + bandwagon + multi-turn + pressure + mental health).
- [x] Có thử lại bằng tình huống trong Bài 1 (bảng kết quả thử 6 case T-02, T-03, T-05, T-06, T-07, T-11).
- [x] Không dùng prompt như cách duy nhất — đã có lớp 1 + lớp 3 bổ trợ.

**Người phụ trách**: Ngô Hải Văn.
