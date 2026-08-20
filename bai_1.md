# BÀI 1: Phân tích & Lựa chọn - Giải quyết lỗi không tương thích số chiều vector

**Đáp án lựa chọn:** Phương án C

**Giải thích lý do lựa chọn:**
Phương án C là phương án tối ưu nhất dưới góc độ kỹ thuật phần mềm và hệ thống:
1. **Thiết kế cơ sở dữ liệu và quản lý hạ tầng (Dev/Prod Parity):** Nguyên tắc cốt lõi của DevOps và Software Engineering là giữ cho môi trường Local/Dev càng giống với Production càng tốt (Dev/Prod Parity). Việc giữ nguyên `vector_store` với 1536 chiều giúp cấu trúc DB đồng nhất, không cần viết script migration phức tạp, đảm bảo schema luôn đồng bộ giữa các môi trường.
2. **Khả năng so khớp ngữ nghĩa (Semantic Matching):** Retrieval-Augmented Generation (RAG) phụ thuộc hoàn toàn vào vector. Các mô hình embedding khác nhau (384 chiều vs 1536 chiều) sẽ biểu diễn không gian ngữ nghĩa hoàn toàn khác nhau. Khoảng cách Cosine Similarity và các ngưỡng (thresholds) được tinh chỉnh trên môi trường Dev (với 384 chiều) sẽ trở nên vô dụng và sai lệch khi đẩy lên Production (1536 chiều). Sử dụng mô hình 1536 chiều ở Local (như `nomic-embed-text`) hoặc chung API Key Dev đảm bảo thuật toán truy xuất (retrieval) hoạt động chính xác trên cả hai môi trường.

**Phân tích các phương án loại trừ:**
- **Phương án A (Alter Table thủ công):** Cực kỳ rủi ro. Việc thay đổi schema DB (từ 384 lên 1536) bằng lệnh ALTER trên Production có thể gây downtime, làm mất/hỏng dữ liệu (data corruption) do các vector cũ 384 chiều không thể tự động phình to thành 1536 chiều nếu không được re-embed lại. Nó phá vỡ tính nhất quán của CI/CD pipeline.
- **Phương án B (Tạo 2 bảng riêng biệt):** Mặc dù giải quyết được lỗi SQL và tránh đụng độ trực tiếp, phương án này tạo ra "nợ kỹ thuật" (Technical Debt) lớn. Việc duy trì hai cấu trúc bảng khác nhau khiến code trở nên cồng kềnh, khó bảo trì. Quan trọng hơn, như đã phân tích ở trên, logic tính toán Cosine Similarity trên bảng 384 (Local) sẽ không phản ánh đúng hành vi của hệ thống trên bảng 1536 (Prod), làm mất đi ý nghĩa của việc test tính năng trên môi trường Dev.
