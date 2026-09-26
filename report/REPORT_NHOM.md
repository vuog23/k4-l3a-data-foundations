# Báo cáo nhóm — Lab 7: Embedding & Vector Store (K4-L3A)

**Nhóm:** [bổ sung tên nhóm]
**Thành viên:** [bổ sung họ tên]
**Ngày:** 2026-09-26

> Bản nháp này được điền từ nội dung đang có trong repository. Hai tài liệu trong `data/university/` tự ghi là dữ liệu khởi động/template và dùng URL `example.edu`; cần thay bằng nguồn chính thức trước khi xem kết quả là benchmark thực tế. Repository hiện có 2 tài liệu đại học, chưa đạt yêu cầu 5–10 tài liệu.

## 1. Lựa chọn tài liệu

### Chủ đề và lý do

**Chủ đề:** Dịch vụ và thủ tục học vụ đại học: đăng ký học phần và dịch vụ thư viện.
Chủ đề phù hợp với K4-L3A và tạo được câu hỏi truy xuất có câu trả lời kiểm chứng trực tiếp trong tài liệu. Tập hiện tại còn quá nhỏ và nội dung thư viện có ghi rõ cần bổ sung quy định chính thức về thời hạn mượn, gia hạn và quá hạn.

### Kiểm kê dữ liệu hiện có

| # | Tài liệu | Nguồn trong metadata | Ngày / phiên bản | Số ký tự tệp | Metadata |
|---|---|---|---|---:|---|
| 1 | `course-registration.md` — Đăng ký học phần | `https://example.edu/hoc-vu/dang-ky-hoc-phan` (URL mẫu, chưa xác minh) | 2026-08-02 / 2026.1 (mẫu) | 939 | `audience=student`, `department=academic-affairs`, `language=vi`, `doc_id` |
| 2 | `library-services.md` — Dịch vụ thư viện | `https://example.edu/thu-vien/dich-vu` (URL mẫu, chưa xác minh) | 2026-08-02 / 2026.1 (mẫu) | 754 | `audience=all`, `department=library`, `language=vi`, `doc_id` |

**Quản trị dữ liệu:** Chưa thể đánh dấu đạt. Cả hai nguồn đều là URL mẫu, corpus chưa đủ 5–10 tài liệu. Trước khi nộp, thay URL/ngày/phiên bản bằng thông tin nguồn công khai thật, bổ sung tối thiểu 3 tài liệu và xác nhận quyền sử dụng.

### Metadata đề xuất

| Trường | Kiểu / ví dụ | Giá trị cho truy xuất |
|---|---|---|
| `audience` | enum: `student`, `faculty`, `staff`, `all` | Lọc đúng nhóm người dùng; benchmark có câu lọc `student` |
| `department` | chuỗi: `library`, `academic-affairs` | Thu hẹp theo đơn vị phụ trách |
| `category` | chuỗi: `registration`, `borrowing` | Phân loại nghiệp vụ khi hỏi theo chủ đề |
| `language` | chuỗi: `vi` | Lọc ngôn ngữ corpus |
| `source_url` | URL | Truy vết nguồn và kiểm chứng câu trả lời |
| `retrieved_at` | ngày ISO | Đánh giá độ mới |
| `document_version` | chuỗi/ngày hiệu lực | Phân biệt phiên bản chính sách |

## 2. Thiết kế chiến lược

### Baseline

Chưa có kết quả chạy `ChunkingStrategyComparator().compare()` được lưu trong repository. Hãy chạy trên 2–3 tài liệu đã xác minh và điền các số liệu bên dưới; không nên suy ra điểm truy xuất từ số chunk.

| Tài liệu | Chiến lược | Số chunk | Độ dài TB | Nhận xét ngữ cảnh |
|---|---|---:|---:|---|
| [điền] | FixedSizeChunker (`fixed_size`) | — | — | — |
| [điền] | SentenceChunker (`by_sentences`) | — | — | — |
| [điền] | RecursiveChunker (`recursive`) | — | — | — |

### Chiến lược từng thành viên

**Thành viên 1 — [tên]**: [chiến lược, tham số, lý do phù hợp; nếu custom, dán mã nguồn].
**Thành viên 2 — [tên]**: [chiến lược, tham số, lý do phù hợp; nếu custom, dán mã nguồn].
**Thành viên 3 — [tên]**: [chiến lược, tham số, lý do phù hợp; nếu custom, dán mã nguồn].

### So sánh trong nhóm

| Thành viên | Chiến lược | Điểm truy xuất /10 | Điểm mạnh | Điểm yếu |
|---|---|---:|---|---|
| [tên] | [điền] | — | — | — |
| [tên] | [điền] | — | — | — |
| [tên] | [điền] | — | — | — |

Chưa thể kết luận chiến lược tốt nhất khi chưa có kết quả chạy chung. Với corpus ngắn có cấu trúc tiêu đề/đoạn như hiện tại, recursive chunking có khả năng giữ ranh giới mục tốt; cần xác nhận bằng top-3 retrieval trên các câu hỏi chung.

## 3. Benchmark câu hỏi và chất lượng truy xuất

Các câu trả lời dưới đây chỉ dựa trên câu chữ có trong corpus khởi động hiện tại. Chúng không phải quy định chính thức của một trường.

| # | Câu hỏi | Câu trả lời chuẩn | Tài liệu/đoạn liên quan |
|---|---|---|---|
| 1 | Sinh viên đăng ký học phần ở đâu và theo lịch nào? | Trên cổng học vụ, theo lịch của từng học kỳ. | `course-registration.md`, đoạn 1 |
| 2 | Sinh viên cần kiểm tra gì trước khi xác nhận đăng ký học phần? | Kiểm tra học phần tiên quyết nếu học phần đó yêu cầu. | `course-registration.md`, đoạn 1 |
| 3 | Sinh viên xử lý lỗi trùng lịch và yêu cầu ngoại lệ như thế nào? | Điều chỉnh lớp trước hạn điều chỉnh được công bố; yêu cầu ngoại lệ gửi qua kênh hỗ trợ học vụ chính thức. | `course-registration.md`, đoạn 2 |
| 4 | Cần mang gì để sử dụng dịch vụ mượn tài liệu? | Thẻ định danh hợp lệ. | `library-services.md`, đoạn 1 |
| 5 | Chỉ tìm tài liệu có `audience=student`: sinh viên đăng ký học phần ở đâu? | Sinh viên đăng ký học phần trên cổng học vụ. | `course-registration.md`, đoạn 1; front matter `audience=student` |

> Câu 5 là câu cần filter metadata; trong corpus hiện tại chỉ tài liệu đăng ký học phần có `audience=student`.

### Kết quả chạy retrieval

Chưa có log chạy 5 câu hỏi bằng các chiến lược của thành viên. Điền sau khi chạy: chiến lược tốt nhất theo từng câu, top-3 có chứa chunk liên quan hay không, và câu trả lời agent. Không tự chấm điểm khi chưa có log.

| # | Chiến lược tốt nhất | Chunk liên quan trong top-3? | Ghi chú / câu trả lời agent |
|---|---|---|---|
| 1 | — | Chưa chạy | — |
| 2 | — | Chưa chạy | — |
| 3 | — | Chưa chạy | — |
| 4 | — | Chưa chạy | — |
| 5 | — | Chưa chạy | — |

**Metadata:** Filter có thể loại bỏ tài liệu sai đối tượng trước khi xếp hạng. Tuy nhiên corpus hiện có một tài liệu `student` và một tài liệu `all`, nên cần thêm tài liệu và kiểm tra câu hỏi cùng filter trước khi kết luận filter cải thiện kết quả.

## 4. Demo và bài học nhóm

- So sánh ba ranh giới chunk: kích thước ký tự, nhóm câu, và phân tách đệ quy theo đoạn/câu/từ.
- Demo câu hỏi trùng lịch và câu hỏi cần lọc theo audience; chỉ trình diễn trên nguồn chính thức sau khi thay dữ liệu mẫu.
- Hiện chưa có kết quả nhóm để báo cáo bài học thực nghiệm. Dự kiến cần đối chiếu tính mạch lạc của chunk, khả năng tìm đúng đoạn, và độ chính xác của câu trả lời.

**Nếu làm lại:** Mở rộng corpus lên 5–10 trang chính thức, chuẩn hóa front matter metadata, tạo câu hỏi có đáp án cụ thể từ từng nguồn, rồi chạy cùng benchmark và lưu top-3 cho mỗi chiến lược.

## Tự đánh giá nhóm

Chưa chấm điểm: cần hoàn thành corpus, thí nghiệm retrieval và demo trước khi tự đánh giá các hạng mục.
