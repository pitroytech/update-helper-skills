---
name: update-helper-guidelines
description: Các nguyên tắc hành vi khi chỉnh sửa các file mã nguồn lớn, dựa trên bộ quy tắc update-helper. Giúp tránh lỗi làm hỏng định dạng, sai lệch dòng và phá hủy context window.
license: MIT
---

# Update-Helper Guidelines

Đây là bộ quy tắc hành vi thiết yếu dành cho AI (LLM) khi thao tác trên các codebase hoặc file nguồn có kích thước lớn (1,000–30,000+ dòng). 
Bộ quy tắc này được đúc kết từ cẩm nang "Universal Large File Patcher" của PitroyTech. Tuân thủ nghiêm ngặt để đảm bảo an toàn cho dự án.

**Đánh đổi:** Các nguyên tắc này đặt **Sự An Toàn** lên trên Tốc Độ. Một thao tác sai lầm có thể phá hỏng toàn bộ codebase.

## 1. Không Tự Bịa Code (Never Assume Target Content)

**Tuyệt đối không gõ tay phần mã nguồn cần thay thế. Mọi sai lệch khoảng trắng đều dẫn đến thất bại.**

Trước khi dùng lệnh thay thế (replace):
- BẮT BUỘC dùng lệnh đọc file (ví dụ: `view_file` với `StartLine` và `EndLine` cụ thể). Không bao giờ đọc toàn bộ file lớn vì sẽ tràn bộ nhớ ngữ cảnh (Context Window).
- Copy y nguyên 100% nội dung thật đang có trong file (kể cả những khoảng trắng, thụt lề vô lý nhất) để làm `TargetContent`.
- Không bao giờ đoán cấu trúc code nếu chưa đọc trực tiếp.

## 2. Khảo Sát Kỹ Trước Khi Hành Động (Read Before You Patch)

**Đọc → Hiểu → Lên Kế Hoạch → Vá Lỗi → Xác Nhận. Không bao giờ nhảy cóc.**

Trước khi viết code mới:
- Đừng vội vã vá lỗi ngay. Hãy tìm hiểu xem hàm/biến này được gọi từ đâu và trả kết quả về đâu (Trace Data Flow).
- Dùng chức năng tìm kiếm (Search/Grep) để vạch ra cấu trúc file trước khi đi sâu vào logic.
- Nếu mô tả (spec) từ người dùng không khớp với code thực tế đang có, hãy DỪNG LẠI, báo cáo sự khác biệt và hỏi ý kiến thay vì cố tình ghi đè.

## 3. Bảo Vệ Dữ Liệu Tuyệt Đối (Safety First: Backups & Encoding)

**Luôn thủ sẵn đường lui. Đừng để hỏng file rồi mới lo đi sửa.**

Khi thao tác trên file:
- **Nguyên tắc Backup 3 Lớp:** Tạo bản sao lưu `.bak` ở đầu phiên làm việc. Tạo bản sao lưu thứ hai `.bak2` trước mỗi đợt vá code phức tạp hoặc động chạm đến luồng dữ liệu chính.
- Nếu lỗi xảy ra ở đợt cập nhật lớn, khôi phục lại ngay từ file `.bak` gần nhất.
- **Cảnh báo Bảng Mã (Encoding):** Các công cụ tự động hoặc PowerShell có thể âm thầm làm mất ký hiệu BOM (Byte Order Mark) của file UTF-8. Nếu file chứa ký tự tiếng Việt hoặc ký tự đặc biệt, phải hết sức chú ý phương thức lưu file để không làm hỏng dữ liệu.

## 4. Vá Lỗi Phẫu Thuật, Sửa Từ Dưới Lên (Surgical, Bottom-Up Execution)

**Sửa ít nhất có thể. Xử lý triệt để rủi ro trượt dòng (Line Shifting).**

Khi thực hiện chỉnh sửa:
- Nếu phải sửa nhiều đoạn (không liền kề) trong cùng 1 file, hãy ưu tiên sửa từ **dưới cùng lên trên cùng** (Bottom to Top). Việc này giúp các mốc dòng code (line number) bên trên không bị sai lệch đi sau khi các dòng bên dưới đã bị thay đổi độ dài.
- Kiểm tra tính hợp lệ của cú pháp (Syntax) hoặc logic ngay lập tức sau **mỗi** một bản vá. 
- Không gộp lại sửa mù mờ một đống rồi mới chạy kiểm tra, vì khi xuất hiện lỗi bạn sẽ không thể khoanh vùng được nguyên nhân do đâu.
- Chỉ chỉnh sửa đúng những dòng được yêu cầu. Tuyệt đối không tự ý format, dọn dẹp code, hay xóa comment xung quanh nếu không nằm trong chỉ đạo của người dùng.
