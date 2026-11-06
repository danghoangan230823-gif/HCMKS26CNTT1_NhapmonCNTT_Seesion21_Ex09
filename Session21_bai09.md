# Bài 9: Prompt 3 thành phần giải mã lỗi trạm sạc VoltCharge

## Phần 1: Thiết kế Prompt 3 thành phần

### 1. Vì sao câu hỏi cụt ngủn khiến AI trả lời sai?

Câu "AI ơi phần mềm trạm sạc của tôi bị hỏng rồi" thiếu hoàn toàn dữ kiện:
- **Không có ngữ cảnh:** AI không biết thiết bị, hệ điều hành, tác vụ đang làm.
- **Không có lỗi nguyên văn:** AI không có bằng chứng để phân tích.
- **Không có yêu cầu rõ ràng:** không nói cần giải thích hay cần cách sửa.

AI vẫn phải trả lời nên **đoán mò** theo những tình huống phổ biến nhất ("trạm sạc hỏng" nghĩa là kiểm tra phần cứng, dây điện). Đó là **ảo giác AI (Hallucination)**: câu trả lời nghe hợp lý nhưng không dựa trên dữ liệu thật, và ở đây còn sai bản chất (lỗi phần mềm, không phải phần cứng).

### 2. Prompt 3 thành phần hoàn chỉnh

```text
[Context]
Tôi là kỹ thuật viên trực ca đêm tại Kiosk sạc xe điện VoltCharge (trụ sạc nhanh số 04).
Máy điều khiển chạy Windows 11. Trụ đột ngột ngừng cấp điện cho xe taxi điện,
màn hình Terminal hiện thông báo lỗi bên dưới. Tôi là người mới, không giỏi tiếng Anh chuyên ngành.

[Raw Error]
[TERMINAL ERROR LOG - VOLTCHARGE CHARGER #04]
2026-09-21 21:15:30 [FATAL] ControllerDaemon failed to start.
System.UnauthorizedAccessException: Access to the path 'C:\Program Files\VoltCharge\data\session.log' is denied.
  at System.IO.FileStream.ValidateFileHandle(SafeFileHandle handle)
  at VoltCharge.Core.SessionManager.Initialize()
InnerException: None. Process terminated with exit code 5 (0x00000005).
Suggestion: Run the application with elevated administrator privileges or verify NTFS file permissions.

[Instruction]
1. Giải thích lỗi bằng tiếng Việt bình dân, dễ hiểu cho người mới.
2. Chỉ ra nguyên nhân gốc rễ dựa trên log trên (không đoán ngoài log).
3. Đưa ra đúng 3 bước khắc phục cụ thể trên Windows 11.
4. Nếu thiếu thông tin, hãy hỏi lại thay vì tự suy đoán.
```

## Phần 2: Giải mã lỗi và kịch bản khắc phục

### 3. Giải thích lỗi bằng tiếng Việt đời thường

**Dịch nhanh:**
- `ControllerDaemon failed to start`: chương trình điều khiển trụ sạc **không khởi động được** (lỗi nghiêm trọng).
- `UnauthorizedAccessException ... Access ... is denied`: **bị từ chối quyền truy cập** vào đường dẫn `C:\Program Files\VoltCharge\data\session.log`.
- `exit code 5`: mã lỗi 5 của Windows nghĩa là **"Access is denied"** (không đủ quyền).
- `Suggestion`: gợi ý chạy bằng **quyền Quản trị viên** hoặc kiểm tra quyền file NTFS.

**Nguyên nhân gốc rễ:** phần mềm cố **ghi file nhật ký phiên sạc** (`session.log`) vào thư mục `Program Files`, là thư mục được Windows bảo vệ. Tài khoản đang chạy phần mềm **không có quyền ghi** nên chương trình bị dừng ngay lúc khởi tạo, và trụ sạc ngừng hoạt động.

**Ví dụ dễ hiểu:** giống nhân viên cố ghi sổ vào két sắt của quản lý nhưng không có chìa khóa. Đây là **lỗi phần mềm/phân quyền**, hoàn toàn **không phải hỏng phần cứng**, nên không cần tháo vỏ trụ sạc.

### 4. Ba bước xử lý trên Windows 11

1. **Chạy thử bằng quyền Quản trị viên:** chuột phải vào biểu tượng ứng dụng VoltCharge → **Run as administrator** → chọn **Yes**.
2. **Cấp quyền ghi cho thư mục `data`:**
   - Mở `C:\Program Files\VoltCharge\`, chuột phải thư mục **data** → **Properties** → tab **Security** → **Edit**.
   - Chọn tài khoản chạy dịch vụ (ví dụ nhóm **Users**) → tích **Allow** cho **Modify** và **Write** → **Apply** → **OK**.
3. **Khởi động lại và kiểm tra:** khởi động lại ứng dụng/dịch vụ, mở lại log để xác nhận không còn lỗi, rồi thử sạc xe.

*(Tùy chọn, dòng lệnh, chạy trong CMD quyền Admin; nên kiểm tra tên tài khoản trước khi chạy:)*
```cmd
icacls "C:\Program Files\VoltCharge\data" /grant Users:(OI)(CI)M
```

**Khuyến nghị lâu dài:** không nên mở rộng quyền ghi bừa bãi vào `Program Files`; nên cấu hình để phần mềm ghi log ở thư mục dữ liệu riêng (như `C:\ProgramData\VoltCharge`) hoặc chạy dưới **tài khoản dịch vụ chuyên dụng** với quyền tối thiểu, theo nhà cung cấp phần mềm.

### 5. Bẫy an ninh: chuỗi `API_KEY: volt_secret_token_889922`

**Xử lý trước khi dán vào ChatGPT/Gemini:** **che hoặc thay bằng giá trị giả**, ví dụ `API_KEY: [REDACTED]` hoặc `API_KEY: xxxxxxxx`. Rà soát cả mật khẩu DB, token, mã OTP, IP/tên máy nội bộ.

**Lý do:**
- Nội dung chat được gửi tới máy chủ bên thứ ba; có thể bị **lưu lại** hoặc **lộ qua lịch sử chat/tài khoản bị chiếm**.
- AI **không cần** khóa bí mật để giải thích lỗi; thông tin còn lại đã đủ.
- Lộ API Key cho phép kẻ khác **giả mạo hệ thống** và truy cập trái phép dữ liệu/dịch vụ.
- **Nếu lỡ dán khóa thật:** thu hồi và **tạo khóa mới (rotate)** ngay.

## Phần bổ sung: 3 nguyên tắc an toàn khi dùng AI
1. **Không copy-paste mù quáng** lệnh AI đưa ra; đọc hiểu và kiểm tra trước khi chạy (đặc biệt với lệnh quản trị).
2. **Cung cấp đủ ngữ cảnh** (3 thành phần) để tránh ảo giác.
3. **Tuyệt đối không gửi Secret**, mật khẩu, API Key, mã OTP lên AI.
