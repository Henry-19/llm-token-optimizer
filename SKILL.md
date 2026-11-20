---
name: llm-token-optimizer
description: >-
  Comprehensive guide and operational runbook for optimizing LLM token consumption, latency, and cost
  while maximizing model productivity and reasoning quality. Use this skill when designing prompts,
  implementing RAG/agent workflows, integrating semantic caching, configuring structured outputs,
  or routing tasks across model tiers.
---

# LLM Token & Efficiency Optimizer (SKILL.md)

Skill này cung cấp kiến trúc và quy trình chuẩn hóa để tối ưu hóa token, giảm chi phí API và tăng tốc độ xử lý của mô hình ngôn ngữ lớn (LLM), đồng thời khắc phục triệt để các hạn chế cố hữu của các phương pháp tối ưu đơn lẻ.

---

## 1. Kiến trúc Tối ưu hóa 4 Tầng (4-Layer Optimization Pipeline)

Quy trình xử lý một yêu cầu (Prompt/Request) qua 4 chốt chặn tối ưu:

```text
[User / System Request]
          │
          ▼
┌─────────────────────────┐
│ Layer 1: Semantic Cache │ ─── (Hit: Similarity >= Threshold & Valid TTL) ───► Trả về ngay (0 Token)
└─────────────────────────┘
          │ (Miss)
          ▼
┌─────────────────────────┐
│ Layer 2: Model Routing  │ ─── Đánh giá độ phức tạp / độ rủi ro tác vụ
└─────────────────────────┘
          │
          ├── Dễ / Phổ thông ────────► Gán cho Fast Tier (Flash / Mini / 8B)
          └── Khó / Suy luận sâu ────► Gán cho Strong Tier (Pro / Sonnet / 70B)
          │
          ▼
┌─────────────────────────┐
│ Layer 3: Context & Comp │ ─── Nén tài liệu RAG, trích xuất thông tin trọng yếu,
└─────────────────────────┘     loại bỏ token rác / từ nối không mang giá trị ngữ nghĩa
          │
          ▼
┌─────────────────────────┐
│ Layer 4: Structured Out │ ─── Ràng buộc JSON / Schema chặt chẽ, triệt tiêu văn phong rườm rà
└─────────────────────────┘
          │
          ▼
[High-Quality Minimal Token Output]
```

---

## 2. Tổng hợp Ưu điểm & Giải pháp Khắc phục Hạn chế

| Phương pháp | Ưu điểm cốt lõi | Hạn chế cố hữu | Giải pháp khắc phục trong Skill này |
| :--- | :--- | :--- | :--- |
| **Prompt Compression** *(LLMLingua)* | Giảm 50% - 80% token input context, mở rộng khả năng tiếp nhận context dài. | Rủi ro cắt nhầm số liệu quan trọng, mã code, thực thể pháp lý hoặc logic phủ định. | **Preservation Masking:** Khóa cứng (Pin/Protect) các thực thể (NER, con số, công thức, từ khóa điều kiện `NOT`, `ONLY`, `BEFORE`) không cho phép nén. Chỉ nén văn bản mô tả nền. |
| **Semantic Caching** *(GPTCache)* | Tiết kiệm 100% token khi có câu hỏi tương tự; độ trễ phản hồi < 20ms. | Trả về dữ liệu cũ (Stale Data), hoặc match nhầm câu hỏi có từ phủ định (False Positives). | **Strict Filtering + TTL:** Thiết lập TTL theo độ biến động của dữ liệu. Bắt buộc kiểm tra từ khóa phân cực (như `không`, `chưa`, `trừ`) trước khi chấp nhận Cache Hit. |
| **Structured Output** *(Outlines/Guidance)* | Không tốn token cho lời dẫn rườm rà; triệt tiêu lỗi cú pháp JSON; 0 retry parse. | Có thể rơi vào vòng lặp vô hạn (Infinite Loop) nếu schema mâu thuẫn với khả năng mô hình. | **Graceful Timeout & Schema Fallback:** Đặt `max_tokens` chặt chẽ; định nghĩa schema đơn giản hóa mức tối đa, bổ sung cơ chế fallback về raw text nếu timeout. |
| **Model Routing** *(RouteLLM/LiteLLM)* | Cắt giảm tới 70-85% chi phí API bằng cách chuyển việc dễ sang mô hình rẻ. | Router phân loại sai câu hỏi phức tạp thành đơn giản, gây suy giảm chất lượng câu trả lời. | **Dual-Threshold & Escalation Policy:** Nếu điểm độ khó rơi vào vùng xám (Ambiguous Zone, 0.4 - 0.6), tự động ưu tiên model mạnh hơn. Nếu model nhỏ trả về kết quả lỗi hoặc không tự tin, tự động retry sang model lớn. |

---

## 3. Quy chuẩn Prompt Engineering Tiết kiệm Token (Token-Saving Rules)

Khi tạo prompt hoặc hướng dẫn agent, tuân thủ các quy tắc sau:

1. **Loại bỏ Prompt Politeness & Fluff:**
   * ❌ *Tránh:* "Xin vui lòng hãy giúp tôi đọc kỹ đoạn văn dưới đây và nếu có thể hãy cho tôi biết..."
   * ✅ *Dùng:* "Nhiệm vụ: Trích xuất các lỗi bảo mật trong đoạn mã sau. Định dạng: gạch đầu dòng ngắn gọn."

2. **Sử dụng Markdown / Delimiter Ngắn:**
   * Sử dụng dấu ba nháy (```) hoặc thẻ XML ngắn (`<ctx>`, `<req>`) thay vì các đoạn văn dẫn giải dài dòng.

3. **Chỉ thị Output Không Lời Thoại:**
   * Luôn gắn cờ: `Return raw JSON only. No markdown formatting, no explanations, no prefix/suffix.`

4. **Kỹ thuật Few-Shot Tối Giản:**
   * Thay vì đưa ra 5 ví dụ dài, chỉ cần đưa 1-2 ví dụ đại diện ngắn gọn nhất, tập trung vào edge-case.

---

## 4. Runbook Triển khai Mẫu (Python Workflow)

### A. Tầng Nén Context Bảo toàn Dữ liệu (Context Compressor)
```python
def compress_context_safely(long_text: str, protected_terms: list[str]) -> str:
    """
    Nén ngữ cảnh nhưng bảo vệ nguyên vẹn các thực thể quan trọng (số liệu, mã định danh).
    """
    # 1. Tách và gắn tag bảo vệ cho các từ khóa nhạy cảm
    placeholders = {}
    for i, term in enumerate(protected_terms):
        tag = f"__PROTECTED_{i}__"
        placeholders[tag] = term
        long_text = long_text.replace(term, tag)

    # 2. Áp dụng logic nén (hoặc gọi mô hình nén nhỏ / regex rút gọn)
    compressed = " ".join([line.strip() for line in long_text.splitlines() if line.strip()])

    # 3. Phục hồi lại các thực thể quan trọng
    for tag, original in placeholders.items():
        compressed = compressed.replace(tag, original)
        
    return compressed
```

### B. Tầng Định tuyến Tác vụ (Task Router)
```python
def route_model(prompt: str, task_type: str) -> str:
    """
    Quyết định model dựa trên loại tác vụ và độ dài yêu cầu.
    """
    # Tác vụ cần suy luận logic nhiều bước hoặc tạo mã nguồn phức tạp
    HEAVY_TASKS = {"architecture_design", "security_audit", "complex_reasoning"}
    
    if task_type in HEAVY_TASKS or len(prompt) > 8000:
        return "pro-tier"  # e.g., Claude 3.5 Sonnet / Gemini Pro
    
    # Tác vụ phân loại, trích xuất dữ liệu, dịch thuật, tóm tắt cơ bản
    return "flash-tier"    # e.g., Gemini Flash / GPT-4o-mini
```

---

## 5. Checklist Tối ưu hóa Trước khi Gửi Request

Trước khi gửi bất kỳ prompt nào đến LLM:
- [ ] Câu hỏi này đã có trong Cache chưa? Có kiểm tra keyword phủ định chưa?
- [ ] Tác vụ này có thực sự cần mô hình cao cấp nhất không, hay model Flash đã đáp ứng đủ?
- [ ] Đoạn tài liệu RAG đưa vào đã được lọc bỏ các phần dư thừa (HTML tags, CSS, khoảng trắng thừa) chưa?
- [ ] Đã có chỉ thị ép buộc output không sinh văn bản dẫn dắt (Zero fluff) chưa?
- [ ] Đã đặt giới hạn `max_output_tokens` để tránh trường hợp mô hình hallucinate sinh văn bản vô tận chưa?
