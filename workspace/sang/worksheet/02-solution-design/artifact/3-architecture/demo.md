---
artifact: 3 — Demo kiến trúc dữ liệu
format: ASCII data-flow + bảng thành phần + bảng fallback
---

# demo.md — Demo kiến trúc dữ liệu

ASCII data-flow + 3 bảng (5 thành phần / 4 fallback / monitoring log).

---

## 1. Sơ đồ cách hệ thống xử lý

```text
                     ┌──────────────────────┐
                     │   User gõ câu hỏi    │
                     │  (chatbot bubble UI) │
                     └──────────┬───────────┘
                                │
                                ▼
                  ┌─────────────────────────────┐
                  │   ① PII FILTER              │
                  │   Scan CMND / SĐT / học bạ   │
                  └─────────────┬───────────────┘
                                │ (sạch)
                                ▼
                  ┌─────────────────────────────┐
                  │   ② INTENT CLASSIFIER       │
                  │   Phân 4 nhóm:               │
                  │   • info-request             │
                  │   • advice-request (Syco.)   │
                  │   • mental-health-signal     │
                  │   • out-of-scope             │
                  └─────────────┬───────────────┘
                                │
            ┌───────────────────┼──────────────────┬──────────────────┐
            │                   │                  │                  │
            ▼                   ▼                  ▼                  ▼
     ┌───────────┐      ┌───────────────┐  ┌──────────────┐  ┌──────────────┐
     │ info-req  │      │ advice-req    │  │ mental-health│  │ out-of-scope │
     └─────┬─────┘      └───────┬───────┘  └──────┬───────┘  └──────┬───────┘
           │                    │                 │                 │
           ▼                    ▼                 ▼                 ▼
    ┌──────────────┐     ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │ ③ RAG        │     │ Skip RAG     │  │ ⚡ EMERGENCY │  │ Refuse polite│
    │ admissions   │     │ (model có    │  │ Auto-create  │  │ + redirect   │
    │ +career data │     │ system prompt│  │ counselor    │  │ scope        │
    │              │     │ luật cứng)   │  │ ticket Tier 3│  │              │
    └──────┬───────┘     └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
           │                    │                 │                 │
           │   ┌───────────────┴────────┐         │                 │
           │   │                        │         │                 │
           ▼   ▼                        │         │                 │
       ┌─────────────────┐              │         │                 │
       │  ④ LLM CALL     │◀─────────────┘         │                 │
       │  (Claude/GPT)   │                        │                 │
       │  + system prompt│                        │                 │
       │  + RAG context  │                        │                 │
       └────────┬────────┘                        │                 │
                │ raw response                    │                 │
                ▼                                 │                 │
       ┌─────────────────┐                        │                 │
       │ ⑤ SYCOPHANCY    │                        │                 │
       │    GUARD        │                        │                 │
       │ Match commitment│                        │                 │
       │ wording vs      │                        │                 │
       │ intent?         │                        │                 │
       └────┬───────┬────┘                        │                 │
            │ pass  │ fail (Sycophancy detected)  │                 │
            │       │                             │                 │
            │       ▼                             │                 │
            │  ┌────────────────┐                 │                 │
            │  │ Re-prompt LLM  │                 │                 │
            │  │ + reminder     │                 │                 │
            │  │ luật cứng      │                 │                 │
            │  └───────┬────────┘                 │                 │
            │          │                          │                 │
            └────┬─────┘                          │                 │
                 │                                │                 │
                 ▼                                ▼                 ▼
         ┌─────────────────────────────────────────────────────────────┐
         │  ⑥ RESPONSE FORMATTER                                       │
         │  • Gắn badge "Đã xác minh" hoặc "Gợi ý AI"                  │
         │  • Gắn nút "Đặt lịch counselor" nếu trigger                  │
         │  • Format multi-line cho UI bubble                            │
         └────────────────────────┬─────────────────────────────────────┘
                                  │
                                  ▼
         ┌─────────────────────────────────────────────────────────────┐
         │  ⑦ MONITORING LOG (async)                                   │
         │  Lưu: trace_id, user_id (hashed), intent, RAG hit/miss,     │
         │  Sycophancy detected?, multi-turn push count, PII flag,     │
         │  escalation triggered?                                        │
         └────────────────────────┬─────────────────────────────────────┘
                                  │
                                  ▼
                  ┌────────────────────────────────┐
                  │ User thấy response trên bubble │
                  └────────────────────────────────┘
```

Đặc điểm:

- **PII filter trước** mọi pipeline khác — bảo mật ưu tiên tuyệt đối.
- **Mental-health-signal** route thẳng sang emergency, KHÔNG đi qua RAG / LLM bình thường.
- **Sycophancy guard** ở giữa LLM và UI — defense in depth khi prompt vẫn bị bypass.
- **Monitoring log async** không chặn response, nhưng đảm bảo team review được sau.

---

## 2. Bảng 5 thành phần chính

| # | Thành phần          | Nhận gì?                                        | Làm gì?                                                                                              | Trả ra gì?                                                                       | Stack đề xuất                                  |
|---|---------------------|--------------------------------------------------|-------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|------------------------------------------------|
| 1 | PII filter          | Câu user thô                                    | Regex + classifier nhận diện CMND / SĐT / thu nhập / học bạ paste. Mask token PII trước khi đi tiếp.   | Câu user đã mask + flag PII (bool) + emergency PII alert nếu match               | Microsoft Presidio + custom rule VN            |
| 2 | Intent classifier   | Câu user đã mask                                | Phân 4 intent. Có thể nhiều intent cùng lúc (vd: pressure + mental-health).                            | Intent labels + confidence score                                                  | Haiku 4.5 với few-shot                          |
| 3 | RAG curriculum + career | Intent = info-request + chủ đề (deadline / học bổng / học phí / curriculum) | Tra Vector DB (Pinecone/Weaviate) chứa admissions content + báo cáo TopDev cache.                      | Snippet + nguồn URL + dấu thời gian. Hoặc `null` nếu miss.                        | Pinecone + content team manual upload          |
| 4 | LLM call            | Câu user + system prompt v1 (lớp 2) + RAG context | Sinh response theo system prompt v1                                                                   | Raw response text                                                                | Claude 3.5 Sonnet (primary) + GPT-4o (fallback)|
| 5 | Sycophancy guard    | Raw response + intent label                     | Regex match wording commitment ("phù hợp", "nên chọn", "đăng ký ngay", "rất hợp", "bạn có khả năng thành công") + check intent = advice-request. | Pass-through hoặc trigger re-prompt với reminder.                                | Custom Python service + Hyperscan regex        |
| 6 | Response formatter  | Response sau guard + intent + RAG result        | Gắn badge / disclaimer / nút action theo state. Format Markdown cho UI bubble.                         | JSON `{ text, badge, buttons[], emergency_banner? }` cho UI                       | Next.js API route                              |
| 7 | Monitoring log      | Tất cả output từng bước                         | Lưu async vào Postgres + ELK. Có dashboard real-time. Weekly aggregation cho team review.               | Log row + alert nếu Sycophancy guard trigger rate >5%/h.                          | Postgres + Grafana + PagerDuty                 |

(Bổ sung: ⚡ Emergency route cho mental-health đi qua thành phần riêng — không thuộc 5 thành phần chính ở trên vì nó skip LLM hoàn toàn.)

---

## 3. Bảng 4 trường hợp fallback (khi hệ thống gặp vấn đề)

| Khi nào lỗi xảy ra?                                | Hệ thống làm gì?                                                                                                | Người dùng thấy gì?                                                                                                          | Log gì?                                            |
|----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------|
| **Nguồn chính thức không có dữ liệu** (RAG miss)   | RAG return null → System prompt nhận `[RAG_NULL]` context → AI dùng template refuse mềm + redirect admissions.   | "Mình không có dữ liệu chính thức trong hệ thống cho [chủ đề]. Bạn xem [link admissions] hoặc đặt lịch counselor."           | `rag_miss=true`, topic, user_id_hashed.            |
| **Nguồn bị lỗi hoặc quá chậm** (RAG timeout >2s)   | Cancel RAG, model tiếp tục với system prompt nhưng KHÔNG cite nguồn. Banner phía UI: "Đang tra cứu nguồn — kết quả có thể chưa cập nhật". | Banner vàng "Đang tra cứu nguồn — vui lòng kiểm tra lại tại [link]"; AI vẫn refuse mềm các câu cần nguồn.                    | `rag_timeout=true`, latency_ms.                    |
| **Câu hỏi vượt phạm vi AI** (intent = out-of-scope) | Skip LLM call; trả response polite refuse + redirect.                                                            | "Câu hỏi này mình không hỗ trợ — bạn tham khảo [kênh phù hợp]. Mình quay lại các câu về tuyển sinh nhé?"                     | `out_of_scope=true`, topic.                        |
| **Lỗi này lặp lại nhiều lần** (alert >5% Sycophancy guard trigger/h, hoặc cùng 1 user push back 3 lần) | PagerDuty báo team admissions; weekly review có sẵn report top-10 conversation push back nhiều nhất. Cập nhật prompt + RAG cho vòng sau. | User không thấy gì khác. Phía vận hành: alert + dashboard.                                                                   | Aggregated stats; conversation full trace.         |

Bonus — fallback cho 2 case không trong scope cốt lõi:

| Case                                | Hệ thống làm gì?                                                                                | Người dùng thấy gì?                                                                                          |
|-------------------------------------|--------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| **PII paste** (T-08)                | PII filter mask + trả response cảnh báo bảo mật trực tiếp (KHÔNG đi qua LLM).                    | "Bạn không nên paste [CMND/thu nhập] vào chat. Mình không lưu. Gửi qua form bảo mật: [link]."                |
| **Mental-health signal** (T-07)     | Skip LLM bình thường; route emergency template + auto-create ticket Tier 3 (counselor mental-load).| Banner emergency 1800-XXX + counselor mental-load + career assessment + KHÔNG chọn ngành thay.               |

---

## 4. Monitoring log — bảng cấu trúc

Mỗi conversation lưu 1 row + N row con (1 row/turn):

```text
conversations
─────────────────────────────────────────────────────────
trace_id            UUID
user_id_hashed      str (không lưu PII)
started_at          timestamp
ended_at            timestamp
turn_count          int
intent_primary      enum (info-request, advice-request, mental-health, out-of-scope)
sycophancy_attempts int   ← số lượt user push back sau refuse
pii_detected        bool
escalation_tier     int (0=none, 1=counselor 24h, 2=counselor chat, 3=mental-load)
prompt_inject_attempt bool
```

```text
turns
─────────────────────────────────────────────────────────
trace_id            UUID (FK)
turn_number         int
user_input_masked   text  ← PII đã mask
intent_label        str
rag_hit             bool
rag_source_url      text (nullable)
rag_timestamp       date (nullable)
llm_model           str
llm_latency_ms      int
sycophancy_guard    enum (pass, blocked_and_reprompted, blocked_and_refused)
final_response      text
badge_shown         str (xanh / vàng / emergency)
counselor_ticket_id text (nullable)
```

### Dashboard cần có (Grafana)

| Panel                                      | Metric                                                  | Threshold alert      |
|--------------------------------------------|---------------------------------------------------------|----------------------|
| Sycophancy guard trigger rate             | trigger / total responses                               | >5% /h               |
| RAG hit rate                              | hit / total info-request                                | <60% /day            |
| Multi-turn push back                       | conversation có `sycophancy_attempts ≥3`                | top-10 daily review  |
| Mental-health escalation                   | count / day                                             | >0 → review từng case|
| PII paste detected                         | count / day                                             | >2 → audit form bảo mật |
| Counselor ticket auto-created              | count by tier                                           | overload alert       |
| Latency p95                                | end-to-end p95 ms                                       | >1500ms              |
| Prompt-injection attempt                   | count / day                                             | >5 → red-team review |

### Weekly review checklist (team admissions, 1h Mon AM)

- [ ] Top-10 conversation có `sycophancy_attempts ≥3` → đọc full transcript → có cần cập nhật prompt không?
- [ ] Top-10 RAG miss topic → content team có cần thêm document gì?
- [ ] Tất cả `mental-health=true` → counselor mental-load có follow up chưa?
- [ ] Prompt-injection attempt → có cần thêm guard rule không?
- [ ] PII paste detected → form bảo mật có dễ tìm không, có cần redesign UX không?

---

## 5. Kiểm tra nhanh

- [x] Sơ đồ không chỉ là "AI trả lời tốt hơn", mà có 7 bước cụ thể (PII filter / intent / RAG / LLM / Sycophancy guard / formatter / log).
- [x] Có cách xử lý khi thiếu dữ liệu (RAG null → refuse template).
- [x] Có cách chuyển sang người thật (escalation queue auto-create ticket 3 Tier).
- [x] Có cách theo dõi để lần sau sửa tốt hơn (monitoring log + weekly review + dashboard).
- [x] Có defense in depth: Sycophancy guard catch lỗi khi prompt bị bypass.
- [x] Có ownership rõ: content team update nguồn / admissions team review weekly / engineering on-call cho latency + alert.
