# LAB 3: MẬT MÃ KHÓA CÔNG KHAI & CHỮ KÝ SỐ (JAVA SWING + SQL SERVER)

Dự án thực hành **Lab 3 – Mã hóa khóa công khai** được xây dựng bằng ngôn ngữ **Java (JDK 17)** trên nền tảng **Apache Maven**, kết hợp giao diện đồ họa **Java Swing** (viết bằng Java Class thuần, không sử dụng mã khóa Guarded Code của NetBeans) và lưu vết lịch sử thao tác vào **Microsoft SQL Server**.

---

## 1. Công nghệ & Môi trường phát triển

* **Ngôn ngữ & Nền tảng:** Java SE 17 (sử dụng thư viện bảo mật chuẩn `java.security` và `javax.crypto` tích hợp sẵn trong JDK).
* **Quản lý dự án & Thư viện:** Apache Maven (`mssql-jdbc` phiên bản `12.8.1.jre11`).
* **IDE khuyến nghị:** Apache NetBeans.
* **Cơ sở dữ liệu:** Microsoft SQL Server (kết nối qua giao thức TCP/IP cổng `14333`, xác thực Mixed Mode).

---

## 2. Các thuật toán và chức năng chính

1. **Bài 3.1 – Mật mã RSA (Console):**
   * Sinh cặp khóa RSA `2048-bit` hoặc `3072-bit`.
   * Mã hóa và giải mã thông điệp ngắn với cơ chế đệm **RSA/ECB/OAEPWithSHA-256AndMGF1Padding** (giới hạn kích thước bản rõ $k/8 - 66$ byte: tối đa $190$ byte với khóa $2048$-bit và $318$ byte với khóa $3072$-bit).
2. **Bài 3.2 – Mật mã RSA (Giao diện Swing):**
   * Trực quan hóa quá trình tạo khóa, đo thời gian sinh khóa, mã hóa/giải mã và kiểm chứng tính ngẫu nhiên của đệm OAEP (cùng một bản rõ cho ra các bản mã Base64 khác nhau).
3. **Bài 3.3 – Mật mã đường cong Elliptic (ECC – `secp256r1`):**
   * Cài đặt lược đồ mã hóa lai **ECIES**: thỏa thuận khóa **ECDH** trên đường cong `secp256r1` với cặp khóa tạm thời (*ephemeral key pair*), dẫn xuất khóa phiên bằng **HKDF-SHA256** (RFC 5869) và mã hóa dữ liệu bằng **AES-256-GCM**.
   * Hỗ trợ xuất/nhập cặp khóa công khai và khóa riêng ra tệp `.txt`.
4. **Bài 3.4 – Hệ mã lai RSA + AES-GCM (Văn bản dài):**
   * Kết hợp tốc độ của mật mã đối xứng và khả năng trao đổi khóa của mật mã bất đối xứng: sinh khóa phiên ngẫu nhiên $32$ byte để mã hóa văn bản dài bằng **AES-256-GCM**, sau đó bọc khóa phiên này bằng **RSA-2048-OAEP**.
   * Tích hợp nút **Sửa 1 ký tự bản mã** để kiểm chứng khả năng phát hiện giả mạo dữ liệu (`AEADBadTagException`) của chế độ GCM.
5. **Bài 3.5 – Chữ ký số ECDSA (`SHA256withECDSA`):**
   * Ký số thông điệp bằng khóa riêng trên đường cong `secp256r1` và xác minh tính toàn vẹn, xác thực nguồn gốc bằng khóa công khai (`VALID` / `INVALID`).
6. **Quản lý kết nối & Lịch sử SQL Server:**
   * Tự động ghi nhận nhật ký các thao tác (`ENCRYPT`, `DECRYPT`, `SIGN`, `VERIFY`), độ dài dữ liệu đầu vào và trạng thái thành công/thất bại vào bảng `dbo.History`.

---

## 3. Cấu trúc mã nguồn (`package week_03`)

```text
LAB3_MaHoaKhoaCongKhai/
├── pom.xml                         # Cấu hình Maven và thư viện mssql-jdbc
└── src/main/java/week_03/
    ├── CryptoSupport.java          # Tiện ích Base64, UTF-8, AES-GCM, HKDF-SHA256, xử lý tệp khóa
    ├── FileTextIO.java             # Hộp thoại mở/lưu tệp văn bản và tệp khóa (.txt)
    ├── DbConfig.java               # Quản lý thông tin và mở kết nối JDBC tới SQL Server
    ├── HistoryDao.java             # Thao tác INSERT và SELECT dữ liệu bảng dbo.History
    ├── RSACipher.java              # Cài đặt thuật toán RSA-OAEP (SHA-256)
    ├── RSADemo.java                # Chương trình Console cho Bài 3.1
    ├── frm_RSA.java                # Giao diện Swing cho Bài 3.2
    ├── ECCCipher.java              # Cài đặt thuật toán ECIES (ECDH secp256r1 + HKDF + AES-GCM)
    ├── frm_ECC.java                # Giao diện Swing cho Bài 3.3
    ├── RSA_AES_Cipher.java         # Cài đặt hệ mã lai RSA-2048 + AES-256-GCM
    ├── frm_RSA_AES.java            # Giao diện Swing cho Bài 3.4
    ├── ECCSignature.java           # Cài đặt chữ ký số SHA256withECDSA
    ├── frm_ECC_Signature.java      # Giao diện Swing cho Bài 3.5
    ├── frm_KetNoiSQL.java          # Giao diện cấu hình và kiểm tra kết nối SQL Server
    ├── frm_LichSu.java             # Giao diện xem danh sách 50 thao tác gần nhất
    └── frm_Menu.java               # Cửa sổ điều hướng trung tâm của toàn bộ Lab 3
```

---

## 4. Hướng dẫn cài đặt Cơ sở dữ liệu (SQL Server)

1. Mở **SQL Server Management Studio (SSMS)** và đăng nhập vào instance cục bộ (ví dụ: `DESKTOP-AHL97Q6\SQL`).
2. Đảm bảo dịch vụ SQL Server đã bật giao thức **TCP/IP** trên cổng **`14333`** và chế độ xác thực **SQL Server and Windows Authentication mode**.
3. Chạy đoạn script SQL sau để khởi tạo cơ sở dữ liệu `LAB3_MaHoaKhoaCongKhai`, bảng `dbo.History` và tài khoản `lab3_user`:

```sql
USE master;
GO
IF DB_ID(N'LAB3_MaHoaKhoaCongKhai') IS NULL
BEGIN
    CREATE DATABASE LAB3_MaHoaKhoaCongKhai;
END;
GO

USE LAB3_MaHoaKhoaCongKhai;
GO
IF OBJECT_ID(N'dbo.History', N'U') IS NULL
BEGIN
    CREATE TABLE dbo.History
    (
        Id BIGINT IDENTITY(1,1) NOT NULL PRIMARY KEY,
        Algorithm NVARCHAR(30) NOT NULL,
        Operation NVARCHAR(12) NOT NULL,
        InputLength INT NOT NULL,
        Success BIT NOT NULL,
        CreatedAt DATETIME2(0) NOT NULL
            CONSTRAINT DF_Lab3_History_CreatedAt DEFAULT SYSDATETIME(),
        CONSTRAINT CK_Lab3_InputLength CHECK (InputLength >= 0),
        CONSTRAINT CK_Lab3_Operation CHECK (Operation IN
            (N'ENCRYPT', N'DECRYPT', N'SIGN', N'VERIFY'))
    );
END;
GO

USE master;
GO
IF SUSER_ID(N'lab3_user') IS NULL
BEGIN
    CREATE LOGIN lab3_user WITH PASSWORD = 'LabUser@2026', CHECK_POLICY = ON;
END;
GO

USE LAB3_MaHoaKhoaCongKhai;
GO
IF DATABASE_PRINCIPAL_ID(N'lab3_user') IS NULL
BEGIN
    CREATE USER lab3_user FOR LOGIN lab3_user;
END;
GO
GRANT SELECT, INSERT ON OBJECT::dbo.History TO lab3_user;
GO
```

---

## 5. Hướng dẫn biên dịch và chạy chương trình

1. **Biên dịch dự án:**
   * Mở dự án `LAB3_MaHoaKhoaCongKhai` trong NetBeans.
   * Nhấp chuột phải vào tên dự án $\rightarrow$ chọn **Clean and Build** (`Shift + F11`) để Maven tải driver `mssql-jdbc` và biên dịch mã nguồn.
2. **Chạy chương trình chính (Giao diện Swing):**
   * Nhấn phím **`F6`** (hoặc nhấp phải `frm_Menu.java` $\rightarrow$ **Run File**).
   * Bấm **Kết nối SQL Server**, nhập mật khẩu **`LabUser@2026`** (các thông số mặc định: Server `127.0.0.1`, Port `14333`, Database `LAB3_MaHoaKhoaCongKhai`, User `lab3_user`) và nhấn **Test Connection**.
   * Mở lần lượt các chức năng **3.2 RSA**, **3.3 ECC**, **3.4 RSA + AES**, **3.5 Chữ ký ECDSA** để thực nghiệm và theo dõi kết quả tại mục **Xem lịch sử**.
3. **Chạy bài thực hành Console (Bài 3.1):**
   * Nhấp chuột phải vào tệp `RSADemo.java` $\rightarrow$ chọn **Run File** (`Shift + F6`), sau đó nhập kích thước khóa (`2048` hoặc `3072`) và bản rõ trực tiếp trên cửa sổ **Output** của NetBeans.