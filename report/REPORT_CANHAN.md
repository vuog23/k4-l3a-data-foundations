# Báo cáo cá nhân — Lab 7: Embedding & Vector Store (K4-L3A)

**Họ tên:** [điền tên]
**Nhóm:** [điền nhóm]
**Ngày:** 2026-09-26

> Nội dung giải thích bên dưới được đối chiếu với mã trong `src/`. Các phần cần kết quả thực nghiệm (pytest, embedding similarity và 5 lần chạy retrieval) được để rõ là chưa chạy; không thay bằng số liệu giả.

## 1. Khởi động

### Cosine similarity

Cosine similarity cao nghĩa là hai vector embedding hướng gần nhau, thường biểu thị hai văn bản có nội dung hoặc ý định gần nhau dù cách dùng từ có thể khác. Ví dụ gần nhau: “Sinh viên đăng ký môn học trên cổng học vụ” và “Người học chọn học phần qua hệ thống đăng ký”; xa nhau: “Thư viện cho mượn sách” và “Cây cần ánh sáng để quang hợp”. Cosine thường phù hợp hơn Euclidean cho embedding văn bản vì nó so sánh hướng biểu diễn và ít bị ảnh hưởng bởi độ lớn vector.

### Tính số chunk

Với `chunk_size=500`, `overlap=50`, bước trượt là `500 − 50 = 450`. Theo công thức bài tập: `ceil((10,000 − 50) / 450) = ceil(22.11) = 23 chunks`.

Nếu overlap tăng lên 100, bước trượt còn 400: `ceil((10,000 − 100) / 400) = ceil(24.75) = 25 chunks`. Overlap lớn hơn giúp giữ lại ngữ cảnh ở ranh giới chunk, nhưng tạo thêm chunk trùng lặp và tăng chi phí lưu trữ/tìm kiếm.

## 2. Hướng tiếp cận của tôi

### `SentenceChunker.chunk`

Hàm bỏ qua input rỗng hoặc chỉ có khoảng trắng. Regex `(?<=[.!?])(?:\s+|\n+)` tách sau dấu kết câu, giữ dấu câu ở câu trước; sau đó gom tối đa `max_sentences_per_chunk` câu và nối bằng khoảng trắng. Cách này gọn cho văn bản có dấu chấm than/hỏi/chấm, nhưng có thể tách nhầm chữ viết tắt hoặc số thập phân.

### `RecursiveChunker.chunk` / `_split`

Hàm thử lần lượt các separator mặc định `\n\n`, `\n`, `. `, khoảng trắng và cuối cùng là tách ký tự. Đoạn dài được chia đệ quy theo separator còn lại rồi các đơn vị kề nhau được đóng gói lại nếu vẫn nằm trong `chunk_size`. Base case là đoạn rỗng (trả danh sách rỗng) hoặc đoạn không dài hơn giới hạn (trả nguyên đoạn); fallback theo ký tự xử lý trường hợp không tìm được separator phù hợp.

### `EmbeddingStore`: thêm và tìm kiếm

`add_documents` tạo embedding cho từng nội dung và lưu record gồm ID, nội dung, metadata và vector. Store thử khởi tạo ChromaDB; nếu không sẵn có thì lưu trong list trong bộ nhớ. Tìm kiếm nhúng câu hỏi, xếp hạng các vector theo dot product giảm dần và trả về top-k; backend Chroma dùng khoảng cách trả về để tạo score `1 - distance`.

### Lọc metadata và xóa

`search_with_filter` lọc metadata trước khi tìm kiếm: Chroma nhận `where`, còn bộ nhớ duyệt record và chỉ giữ record khớp mọi cặp key/value. `delete_document` tìm/xóa các chunk mang `doc_id`; backend list thay list bằng các record còn lại và trả về `True` nếu có record bị xóa.

### `KnowledgeBaseAgent.answer`

Agent lấy top-k kết quả, ghép nội dung thành context và thêm nhãn nguồn lấy từ metadata `source` hoặc `doc_id`. Prompt yêu cầu chỉ trả lời theo context và nói rõ khi context không có đáp án; prompt được chuyển cho `llm_fn`. Bản cài hiện chưa đưa metadata filter vào `answer`.

## 3. Hoàn thiện code — kết quả kiểm thử

Không chạy `pytest` trong lượt hoàn thiện báo cáo này, nên chưa có output hoặc số test pass để ghi. Hãy chạy `pytest tests/ -v` trong môi trường Python của lab và dán kết quả tại đây.

```text
Kết quả: chưa chạy trong lượt này
Số test pass: chưa xác định / 42
```

## 4. Dự đoán độ tương tự

Các cặp dưới đây là dự đoán ngữ nghĩa trước khi chạy embedding; cột điểm thực tế cần điền bằng backend đã chọn. Backend mặc định `MockEmbedder` là vector giả xác định, không phản ánh tương đồng ngôn ngữ, nên để đánh giá tiếng Việt cần dùng local multilingual hoặc một embedder thật.

| Cặp | Câu A | Câu B | Dự đoán | Điểm thực tế | Đúng? |
|---|---|---|---|---:|---|
| 1 | Sinh viên đăng ký học phần trên cổng học vụ. | Người học chọn môn qua hệ thống đăng ký. | Cao | Chưa chạy | — |
| 2 | Cần kiểm tra môn tiên quyết trước khi xác nhận đăng ký. | Thư viện cung cấp không gian học tập. | Thấp | Chưa chạy | — |
| 3 | Mang thẻ định danh hợp lệ để mượn tài liệu. | Người dùng cần thẻ thư viện khi mượn sách. | Cao | Chưa chạy | — |
| 4 | Lỗi trùng lịch cần xử lý trước hạn điều chỉnh. | Yêu cầu ngoại lệ gửi qua kênh hỗ trợ học vụ. | Trung bình/cao | Chưa chạy | — |
| 5 | Học phí được thanh toán theo học kỳ. | Chim di cư về phương nam vào mùa đông. | Thấp | Chưa chạy | — |

Chưa thể nêu kết quả bất ngờ khi chưa có điểm similarity thực tế. Khi có điểm, so sánh thứ hạng dự đoán với thứ hạng đo được; nếu dùng mock backend thì sai khác chủ yếu phản ánh giới hạn của mock chứ không phải chất lượng biểu diễn ngữ nghĩa.

## 5. Kết quả truy xuất cá nhân

Dùng cùng 5 câu hỏi trong báo cáo nhóm. Corpus hiện tại còn là dữ liệu mẫu, do đó kết quả dưới đây cần bổ sung sau khi chạy agent trên tài liệu chính thức.

| # | Câu hỏi | Top-1 chunk (tóm tắt) | Score | Liên quan? | Trả lời agent |
|---|---|---|---:|---|---|
| 1 | Sinh viên đăng ký học phần ở đâu và theo lịch nào? | Chưa chạy | — | — | — |
| 2 | Cần kiểm tra gì trước khi xác nhận đăng ký học phần? | Chưa chạy | — | — | — |
| 3 | Xử lý lỗi trùng lịch và yêu cầu ngoại lệ thế nào? | Chưa chạy | — | — | — |
| 4 | Cần mang gì để mượn tài liệu? | Chưa chạy | — | — | — |
| 5 | Chỉ tìm tài liệu có `audience=student`: sinh viên đăng ký học phần ở đâu? | Chưa chạy; chunk nguồn là `course-registration.md` | — | — | — |

**Số câu có chunk liên quan trong top-3:** chưa đo / 5.
**Điều học được qua demo:** [bổ sung sau khi xem demo của nhóm/nhóm khác; nêu một quan sát cụ thể về chunking, metadata hoặc grounding].

## Tự đánh giá cá nhân

Chưa tự chấm điểm vì các phần thực nghiệm và thông tin cá nhân còn cần hoàn tất.
