# Reflection & Action Plan
**Họ và tên:** Phạm Tuấn Anh

## Phần 1: Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Threshold tạo ra các đoạn văn mạch lạc hơn thay vì bị cắt cứng nhắc bằng paragraph. |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | RRF giải quyết được việc cân bằng giữa tìm kiếm từ khóa tiếng Việt chính xác và ngữ nghĩa của câu hỏi. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Reranking giúp kéo các kết quả thực sự liên quan lên đầu, nhưng làm tăng latency đôi chút. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | Giúp tự động hóa việc đánh giá mà không cần giám sát con người, chỉ ra điểm yếu trong từng module. |
| Contextual embeddings | M5 | `contextual_prepend()` | Giảm retrieval failure do cung cấp thêm ngữ cảnh toàn cục (document title) vào từng chunk cô lập. |

## Phần 2: Khó khăn & giải quyết

*   **Lỗi gặp phải:** Không thể chạy được `underthesea` trên một số môi trường, tokenizer bị dính ký tự `_` làm BM25 không khớp từ khóa.
*   **Cách debug:** Thêm khối `try/except` để fallback, đồng thời thực hiện thao tác `replace("_", " ")` trước khi đưa vào token array của BM25.
*   **Kiến thức thiếu:** Sự kết hợp RRF (Reciprocal Rank Fusion) và điểm số BM25 so với Dense embeddings vốn khác ranh giới.
*   **Cách bổ sung:** Đọc công thức tính của RRF, nhận ra không phụ thuộc vào giá trị điểm tuyệt đối mà chỉ tính dựa trên thứ hạng (rank) của từng hệ thống.

## Phần 3: Action Plan cho project

### Project: Hệ thống Q&A nội bộ

#### Hiện tại
- RAG pipeline hiện tại: Naive RAG, chỉ dùng RecursiveCharacterTextSplitter và FAISS vector search.
- Known issues: LLM hay trả lời sai do lấy nhầm chunk không liên quan; không hỗ trợ tìm keyword chính xác (mã số quy định, từ viết tắt).

#### Plan áp dụng
1. [x] **Chunking strategy:** Hierarchical Chunking (Parent-Child). Giúp tăng độ chính xác lúc search (child) nhưng vẫn có đủ context để LLM tổng hợp (parent).
2. [x] **Search:** Hybrid Search (BM25 + BGE-m3). Tận dụng tiếng Việt và bắt mã số quy định chuẩn hơn.
3. [x] **Reranking:** Cross-encoder (bge-reranker-v2-m3) để đẩy chunk chính xác lên top 3.
4. [x] **Evaluation:** RAGAS (4 metrics cốt lõi) kết hợp lưu test set từ lịch sử user query.
5. [x] **Enrichment:** Sử dụng Combined Single-Call (Contextual Prepend + Metadata extraction) để tiết kiệm token mà vẫn bổ sung đầy đủ ngữ cảnh.

#### Timeline
- Tuần 1: Nâng cấp Search lên Hybrid và thêm Reranker vào backend.
- Tuần 2: Implement quy trình đánh giá RAGAS tự động chạy hàng đêm.
- Tuần 3: Tối ưu dữ liệu đầu vào bằng Enrichment LLM pipeline.
