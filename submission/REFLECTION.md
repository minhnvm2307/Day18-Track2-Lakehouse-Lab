# Reflection: Lakehouse Anti-Patterns

Dữ liệu của team (LLM observability logs) dễ vướng vào **Small-File Problem** (vấn đề nhiều file nhỏ) nhất. 

**Lý do:**
1. **Đặc thù dữ liệu:** Dữ liệu log cuộc gọi LLM (`llm_calls_raw`) được ghi nhận liên tục theo thời gian thực (streaming hoặc micro-batch liên tục). Mỗi request ghi nhận một file riêng lẻ hoặc lượng record rất nhỏ.
2. **Hệ quả:** Nếu không tối ưu, hệ thống sẽ tạo ra hàng ngàn file Parquet kích thước chỉ vài KB trong Bronze layer. Điều này gây quá tải metadata cho Delta transaction log và làm chậm nghiêm trọng tốc độ đọc của các câu lệnh tổng hợp (dashboard Gold layer) do I/O overhead.

**Giải pháp khắc phục:**
Thực hiện định kỳ `OPTIMIZE` để gom cụm các file nhỏ thành các file lớn (target size phù hợp) kết hợp `ZORDER BY (model, user_id)` để tăng tốc độ bỏ qua file (file-pruning) khi truy vấn.
