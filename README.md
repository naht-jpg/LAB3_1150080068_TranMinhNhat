# LAB 3 – MÃ HÓA KHÓA CÔNG KHAI

**Họ tên:** Trần Minh Nhật
**MSSV:** 1150080068
**Lớp:** 11CNPM1
**Học phần:** An toàn và Bảo mật Thông tin

Chương trình mã hóa / giải mã và ký số bằng các thuật toán khóa công khai: **RSA (OAEP-SHA256)**, **ECC (ECIES trên secp256r1)**, **hệ mã lai RSA + AES-256-GCM** và **chữ ký số ECDSA (SHA256withECDSA)**. Giao diện Java Swing (NetBeans + Maven), lưu lịch sử thao tác trên **SQL Server**.

---

## 1. Cấu trúc thư mục

```
LAB3_1150080068_TranMinhNhat/
├── LAB3_MaHoaKhoaCongKhai/           # Project Java NetBeans (Maven)
│   ├── pom.xml
│   └── src/main/java/week_03/
│       ├── CryptoSupport.java        # Tiện ích: Base64, UTF-8, số ngẫu nhiên, AES-GCM, HKDF, tệp khóa
│       ├── FileTextIO.java           # Mở / lưu tệp .txt
│       ├── DbConfig.java             # Thông tin kết nối SQL Server (mật khẩu chỉ giữ trong bộ nhớ)
│       ├── HistoryDao.java           # Ghi / đọc bảng dbo.History
│       ├── RSACipher.java            # RSA/ECB/OAEPWithSHA-256AndMGF1Padding
│       ├── RSADemo.java              # Bài 3.1 – RSA chạy trên console
│       ├── ECCCipher.java            # ECIES: ECDH + HKDF-SHA256 + AES-256-GCM
│       ├── RSA_AES_Cipher.java       # Hệ mã lai RSA + AES-256-GCM
│       ├── ECCSignature.java         # Chữ ký số ECDSA trên secp256r1
│       └── frm_*.java / frm_*.form   # Các JFrame Form
├── SQLServer/
│   ├── 01_Tao_CSDL.sql               # Tạo CSDL LAB3_MaHoaKhoaCongKhai và bảng dbo.History
│   ├── 02_Tao_TaiKhoan_LAB.sql       # Tạo login lab3_user, cấp quyền SELECT/INSERT
│   └── 03_KiemTra_LichSu.sql         # Xem lịch sử và thống kê số lần theo thuật toán
├── 11CNPM1-Lab12_1150080068-TranMinhNhat.docx   # Báo cáo kèm ảnh minh chứng
└── README.md
```

| Form | Chức năng |
|---|---|
| `frm_Menu` | Cửa sổ chính, mở các bài |
| `frm_KetNoiSQL` | Kết nối SQL Server |
| `frm_LichSu` | Xem lịch sử thao tác |
| `frm_RSA` | Bài 3.2 – RSA giao diện Swing |
| `frm_ECC` | Bài 3.3 – Mật mã đường cong elliptic |
| `frm_RSA_AES` | Bài 3.4 – Hệ mã lai RSA + AES |
| `frm_ECC_Signature` | Bài 3.5 – Chữ ký số ECDSA |

## 2. Yêu cầu phần mềm

- JDK 17 hoặc 21
- Apache NetBeans (lần build đầu cần Internet để Maven tải thư viện `mssql-jdbc`)
- SQL Server Express + SSMS

RSA, ECC, AES và ECDSA đều dùng thư viện có sẵn của Java (`java.security`, `javax.crypto`), không cần thư viện mã hóa ngoài.

## 3. Chuẩn bị SQL Server

1. **SQL Server Configuration Manager** → Protocols for SQLEXPRESS → bật **TCP/IP**, mục IPAll đặt **TCP Port = 14333** (để trống TCP Dynamic Ports) → Restart dịch vụ SQL Server (SQLEXPRESS).
2. **SSMS** → Server Properties → Security → chọn **SQL Server and Windows Authentication mode**.
3. Kết nối SSMS bằng Windows Authentication, lần lượt chạy:
   - `SQLServer/01_Tao_CSDL.sql`
   - `SQLServer/02_Tao_TaiKhoan_LAB.sql`: script dùng mật khẩu mẫu của bài lab, chỉ dùng trên máy cá nhân. Nếu đổi mật khẩu thì nhập đúng mật khẩu mới trên form kết nối.

Mật khẩu không được ghi trong mã Java: chương trình chỉ nhận mật khẩu khi người dùng gõ trên form `frm_KetNoiSQL`.

## 4. Chạy chương trình

**Bài 3.1 (console):** nhấp phải `RSADemo.java` → **Run File** (Shift+F6). Nhấp vào cửa sổ Output, nhập kích thước khóa (`2048` hoặc `3072`), rồi nhập bản rõ.

**Các bài còn lại:**

1. NetBeans → **File → Open Project** → chọn thư mục `LAB3_MaHoaKhoaCongKhai`.
2. Nhấp phải project → **Clean and Build** (Shift+F11), chờ `BUILD SUCCESS`.
3. Nhấp phải project → **Run** (F6). Cửa sổ `frm_Menu` hiện ra.
4. Bấm **Kết nối SQL Server** → nhập Server `127.0.0.1`, Port `14333`, Database `LAB3_MaHoaKhoaCongKhai`, User `lab3_user` và mật khẩu → **Test Connection**.
5. Mở từng bài: **Generate Keys** → Encrypt / Decrypt (hoặc Sign / Verify). Có thể **Save Keys** / **Load Keys** để lưu và nạp lại cặp khóa.
6. Bấm **Xem lịch sử** để xem các thao tác đã lưu, hoặc chạy `SQLServer/03_KiemTra_LichSu.sql` trong SSMS.

## 5. Dữ liệu kiểm thử

| Bài | Dữ liệu thử | Kết quả |
|---|---|---|
| 3.1 RSA (console) | `HELLO RSA`, khóa 2048 và 3072 bit | Giải mã đúng; bản mã 344 / 512 ký tự Base64; bản rõ quá 190 byte (khóa 2048) bị báo lỗi |
| 3.2 RSA (Swing) | `HELLO RSA` | Giải mã đúng; cùng bản rõ mỗi lần mã hóa ra bản mã khác (OAEP ngẫu nhiên) |
| 3.3 ECC | `HELLO ECC` | Giải mã đúng; dùng khóa sai → `AEADBadTagException: Tag mismatch` |
| 3.4 RSA + AES | Văn bản dài trên 1 000 ký tự | Giải mã đúng toàn văn; sửa 1 ký tự bản mã → giải mã thất bại |
| 3.5 ECDSA | `THONG DIEP A` | `VALID`; sửa thông điệp hoặc đổi khóa → `INVALID` |
| SQL Server | Một lần Encrypt thành công | `dbo.History` có bản ghi tương ứng |

Ảnh minh chứng nằm trong báo cáo Word.

> Tệp khóa (`ecc_keys.txt`, `ecdsa_keys.txt`, …) chứa khóa riêng nên **không** được đưa lên repo. Các khóa xuất hiện trong ảnh minh chứng chỉ là khóa thử nghiệm.

## 6. Lỗi thường gặp

| Lỗi | Cách sửa |
|---|---|
| `Login failed for user 'lab3_user'` | Kiểm tra mật khẩu, bật Mixed Mode rồi Restart dịch vụ |
| `The TCP/IP connection ... port 14333 has failed` | Bật TCP/IP, đặt cổng 14333, Restart dịch vụ |
| `No suitable driver found for jdbc:sqlserver` | Kiểm tra `pom.xml` có `mssql-jdbc`, Clean and Build lại |
| `The INSERT/SELECT permission was denied` | Chạy lại phần GRANT trong `02_Tao_TaiKhoan_LAB.sql` |
| `release version 17 not supported` | Tools → Java Platforms thêm JDK 17/21, chọn lại trong Project Properties |
| Bản rõ vượt giới hạn 190 byte | RSA chỉ mã hóa dữ liệu ngắn; dùng bài 3.4 (RSA + AES) |
| `AEADBadTagException: Tag mismatch` | Sai khóa riêng hoặc bản mã bị sửa; Load Keys đúng tệp khóa đã dùng khi mã hóa |
