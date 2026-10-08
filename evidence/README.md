# Day 22 Lab Evidence & Analysis — LLMOps & Prompt Versioning

**Học viên:** Trần Văn Khánh  
**MSSV:** 2A202602413  
**Project:** `K4-L3-DAY22-TranVanKhanh-2A202602413-LLMOpsPromptVersioning`

---

## 1. Danh Sách Tệp Bằng Chứng (Evidence Files)

| File | Nội Dung / Mô Tả | Trạng Thái |
|---|---|---|
| `evidence/01_langsmith_traces.png` | Ảnh chụp màn hình 50+ traces RAG query trên LangSmith UI | Đã cập nhật |
| `evidence/02_prompt_hub.png` | Ảnh chụp 2 phiên bản prompt đã push lên LangSmith Prompt Hub | Đã cập nhật |
| `evidence/02_ab_routing_log.txt` | Console log chạy 50 câu hỏi qua A/B router (tất định MD5) | Đã hoàn thành |
| `evidence/03_ragas_scores.png` | Ảnh chụp bảng kết quả so sánh RAGAS V1 vs V2 | Đã cập nhật |
| `evidence/03_ragas_report.json` | Tệp báo cáo điểm RAGAS định lượng (Faithfulness, Answer Relevancy, Context Recall, Context Precision) | Đã hoàn thành |
| `evidence/04_pii_demo_log.txt` | Console log chạy 6 test cases PII detector validator | Đã hoàn thành |
| `evidence/04_json_demo_log.txt` | Console log chạy 5 test cases JSON formatter & repair validator | Đã hoàn thành |

---

## 2. Phân Tích So Sánh Prompt V1 vs Prompt V2 (Prompt Performance Analysis)

### Cấu tạo System Prompts
- **Prompt V1 (`tran-van-khanh-rag-prompt-v1`)**: Định hướng trả lời ngắn gọn (2–4 câu), trực diện và loại bỏ thông tin thừa.
- **Prompt V2 (`tran-van-khanh-rag-prompt-v2`)**: Định hướng vai trò chuyên gia phân tích (expert analyst), đọc kỹ tài liệu, trích xuất sự thật (facts) và trình bày câu trả lời có cấu trúc rõ ràng (3–5 câu).

### Đánh giá chất lượng bằng chỉ số RAGAS
1. **Faithfulness (Độ trung thực với ngữ cảnh)**:
   - Cả hai phiên bản đều duy trì giữ nguyên biến `{context}` trong prompt, giúp LLM đạt độ trung thực cao ($\ge 0.8$).
   - **V2** đạt điểm Faithfulness cao hơn nhờ chỉ dẫn kiểm tra nghiêm ngặt facts từ context trước khi đưa ra phản hồi.

2. **Answer Relevancy (Độ liên quan của câu trả lời)**:
   - **V1** phản hồi súc tích, đi thẳng vào trọng tâm câu hỏi nên đạt độ liên quan cao cho các câu hỏi tra cứu thông số ngắn.
   - **V2** cung cấp giải thích có cấu trúc, giúp trả lời đầy đủ ý cho các câu hỏi tổng hợp.

3. **Context Recall & Context Precision**:
   - Chỉ số phụ thuộc vào thuật toán truy xuất FAISS Vector Store với $k=3$ và kích thước chunk 500 ký tự (overlap 50 ký tự), đảm bảo độ phủ thông tin tối ưu.

---

## 3. Guardrails AI Validators

1. **PII Detector (`custom/pii-detector`)**:
   - Sử dụng Regex chuyên biệt để phát hiện 4 loại thông tin nhạy cảm: `EMAIL`, `PHONE`, `SSN`, `CREDIT_CARD`.
   - Kết hợp `OnFailAction.FIX` và `FailResult(fix_value=...)` để thay thế thông tin cá nhân bằng nhãn an toàn `[TYPE_REDACTED]` mà không làm gián đoạn luồng xử lý của ứng dụng.

2. **JSON Formatter (`custom/json-formatter`)**:
   - Tự động phát hiện và loại bỏ markdown code fences (` ```json `).
   - Chuẩn hóa dấu nháy đơn (`'`) thành dấu nháy kép (`"`), xóa dấu phẩy thừa (`trailing commas`).
   - Xử lý mượt mà cả trường hợp JSON hợp lệ, JSON bị lỗi định dạng và trường hợp hoàn toàn không phải JSON (fallback về JSON báo lỗi).
