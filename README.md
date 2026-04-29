# BÀI KIỂM TRA SỐ 2 - HỆ QUẢN TRỊ CƠ SỞ DỮ LIỆU SQL SERVER


**1**. Thông tin sinh viên

**Sinh viên thực hiện:** Nguyễn Văn Sang  

**Mã sinh viên:** K235480106060   

**Lớp:** K59.KMT.K01  

**Khoa:** Điện tử  

**Chủ đề:** Quản lý Rạp Phim 

**2**. Yêu Cầu Đề Bài
Đề tài: Quản lý rạp chiếu phim (Cinema Management System)

Thực hiện xây dựng một hệ thống quản lý cơ sở dữ liệu hoàn chỉnh trên SQL Server, đáp ứng các tiêu chuẩn kỹ thuật sau:

- Tính trực quan: Toàn bộ quá trình thực thi từ khởi tạo đến truy vấn phải được minh họa bằng ảnh chụp màn hình (screenshot), kèm theo giải thích chi tiết về mục đích và kết quả.

- Cấu trúc nộp bài: Bài làm được lưu trữ công khai trên Repository GitHub, bao gồm tệp README.md (báo cáo) và tệp baikiemtra2.sql (mã nguồn).

- Tiêu chuẩn đánh giá: * Logic xử lý dữ liệu chính xác theo nghiệp vụ rạp phim.

- Tuân thủ quy tắc đặt tên chuẩn (PascalCase cho đối tượng và camelCase cho biến).

- Lịch sử Commit trên GitHub thể hiện được quá trình làm việc nghiêm túc.

**3**. Giới Thiệu Hệ Thống Quản Lý Rạp Chiếu Phim

Hệ thống QuanLyRapPhim_K235480106060 được thiết kế nhằm hiện đại hóa quy trình quản lý tại các cụm rạp, giúp tối ưu hóa việc điều phối suất chiếu, quản lý thông tin khách hàng và nâng cao hiệu quả doanh thu. Hệ thống tập trung giải quyết các bài toán thực tế thông qua 5 giai đoạn phát triển:

- Giai đoạn 1 - Thiết kế cấu trúc: Xây dựng lược đồ quan hệ giữa Khách hàng, Phim và Vé xem phim. Đảm bảo tính toàn vẹn dữ liệu thông qua các ràng buộc khóa chính (PK), khóa ngoại (FK) và các ràng buộc kiểm tra (Check Constraints) như giới hạn độ tuổi xem phim hoặc giá vé.

- Giai đoạn 2 - Đóng gói logic (Function): Triển khai các hàm tự định nghĩa (UDF) để xử lý các phép tính lặp lại như: tính giá vé sau khi áp dụng điểm tích lũy, phân loại doanh thu phim hoặc lọc danh sách phim theo yêu cầu.

- Giai đoạn 3 - Nghiệp vụ hóa (Stored Procedure): Xây dựng các thủ tục lưu trữ để thực hiện các giao dịch bán vé phức tạp, kiểm tra điều kiện xem phim của khách hàng và xuất báo cáo thống kê chuyên sâu.

- Giai đoạn 4 - Tự động hóa (Trigger): Sử dụng các bộ kích hoạt tự động để duy trì dữ liệu thời gian thực, ví dụ như tự động cập nhật hạng thành viên và điểm thưởng ngay khi khách hàng hoàn tất giao dịch mua vé.

- Giai đoạn 5 - Kiểm soát luồng dữ liệu (Cursor): Sử dụng con trỏ để duyệt và xử lý các kịch bản cá nhân hóa (như gửi thông báo sinh nhật, thông báo khuyến mãi riêng biệt) và so sánh hiệu năng để tối ưu hóa hệ thống.

Hệ thống không chỉ là một kho lưu trữ thông tin đơn thuần mà còn là một công cụ hỗ trợ ra quyết định thông qua các báo cáo doanh thu và đánh giá hiệu quả hoạt động của từng bộ phim theo thời gian thực.

---

# Phần 1: Thiết kế và Khởi tạo Cấu trúc Dữ liệu

**1**. Tạo cơ sở dữ liệu quản lý rạp phim

1Tạo database

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2a333009-c396-42f4-97d8-b3ba53685563" />
<i>Hình 1: Tạo database QuanLyRapPhim_K235480106060 thành công</i></p>

2 Tạo bảng Phim

- Chức năng: Lưu trữ toàn bộ danh mục các bộ phim có trong rạp. Đây là dữ liệu gốc để các bảng khác tham chiếu tới.
- Khóa chính (Primary Key): MovieID. Dùng kiểu số nguyên, tự động tăng (IDENTITY). Đảm bảo mỗi bộ phim có một mã định danh duy nhất.
- Các ràng buộc (Constraints):

    - NOT NULL: Áp dụng cho TenPhim và GiaVeCoBan (Bắt buộc phải có tên và giá).

    - CHECK Constraint: * DoTuoiQuyDinh >= 0: Ngăn nhập tuổi âm.

    - ThoiLuong > 0: Phim phải có thời gian chiếu thực tế.

    - GiaVeCoBan > 0: Giá vé không thể bằng 0 hoặc âm.

```
CREATE TABLE Phim (

    MovieID INT PRIMARY KEY IDENTITY(1,1), 
    TenPhim NVARCHAR(200) NOT NULL,
    TheLoai NVARCHAR(50),
    DoTuoiQuyDinh INT DEFAULT 0 
        CONSTRAINT CK_Phim_DoTuoi CHECK (DoTuoiQuyDinh >= 0),
    ThoiLuong INT 
        CONSTRAINT CK_Phim_ThoiLuong CHECK (ThoiLuong > 0),
    GiaVeCoBan DECIMAL(10, 2) NOT NULL
        CONSTRAINT CK_Phim_GiaVe CHECK (GiaVeCoBan > 0)
);
GO
```

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e5c5ad20-fa8d-48e7-82c9-7601599b7c50" />
<i>Hình 2: Tạo bảng Phim</i></p>

3 Tạo bảng Khách Hàng

- Chức năng: Lưu trữ thông tin cá nhân, điểm tích lũy và hạng thành viên. Dữ liệu này dùng để thực hiện các chương trình ưu đãi và gửi thông báo.


- Khóa chính (Primary Key): CustomerID. Tự động tăng, dùng để phân biệt các khách hàng trùng tên nhau.

- Khóa duy nhất (Unique Key): Email. Đảm bảo mỗi email chỉ đăng ký được một tài khoản duy nhất, tránh gian lận điểm thưởng.

- Các ràng buộc (Constraints):

  - DEFAULT: * DiemTichLuy = 0: Khách mới luôn bắt đầu từ 0 điểm.

  - HangThanhVien = 'Standard': Hạng mặc định ban đầu.

  - CHECK Constraint: DiemTichLuy >= 0 (Điểm không được âm).

```
CREATE TABLE KhachHang (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    HoTen NVARCHAR(100) NOT NULL,
    NgaySinh DATE,
    Email VARCHAR(100) 
        CONSTRAINT UQ_KhachHang_Email UNIQUE,
    DiemTichLuy INT DEFAULT 0 
        CONSTRAINT CK_KhachHang_Diem CHECK (DiemTichLuy >= 0),
    HangThanhVien NVARCHAR(50) DEFAULT 'Standard'
);
GO

```
<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/dbbb4922-2656-44be-9a0c-bea479643452" />
<i>Hình 3: Tạo bảng KhachHang </i> </p>

4 Tạo bảng VeXemPhim

- Chức năng: Ghi lại lịch sử giao dịch bán vé. Đây là bảng quan trọng nhất để tính doanh thu và theo dõi lịch chiếu.


- Khóa chính (Primary Key): TicketID. Mã số duy nhất cho mỗi tờ vé bán ra.

- Khóa ngoại 1 (Foreign Key): MovieID nối tới bảng Phim.

  - Hành vi: ON DELETE CASCADE (Nếu xóa phim, các vé của phim đó tự động bị xóa theo).

- Khóa ngoại 2 (Foreign Key): CustomerID nối tới bảng KhachHang.

  - Hành vi: ON DELETE NO ACTION (Không cho phép xóa khách hàng nếu họ đã từng mua vé, nhằm giữ lại lịch sử kế toán).

- Các ràng buộc (Constraints):

  - DEFAULT: NgayBan = GETDATE() (Tự động lấy giờ hiện tại của hệ thống khi xuất vé).

  - NOT NULL: Các trường như SuatChieu và GiaThanhToan bắt buộc phải có dữ liệu.
  
```
CREATE TABLE VeXemPhim (
    TicketID INT PRIMARY KEY IDENTITY(1,1),
    MovieID INT NOT NULL 
        CONSTRAINT FK_Ve_Phim FOREIGN KEY REFERENCES Phim(MovieID) 
        ON DELETE CASCADE,
    CustomerID INT NOT NULL 
        CONSTRAINT FK_Ve_KhachHang FOREIGN KEY REFERENCES KhachHang(CustomerID)
        ON DELETE NO ACTION,
    NgayBan DATETIME DEFAULT GETDATE(),
    SuatChieu DATETIME NOT NULL,
    GiaThanhToan DECIMAL(10, 2) NOT NULL
);
GO
```
<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be952da7-8b35-45ac-965c-5ba2a3ab61c3" />
 <i>Hình 4: Tạo bảng VeXemPhim thành công</i></p>
 
5 Chèn dữ liệu mẫu vào bảng vừa  tạo

```
INSERT INTO Phim (TenPhim, TheLoai, DoTuoiQuyDinh, ThoiLuong, GiaVeCoBan)
VALUES 
(N'Doraemon: Bản Tình Ca Công Đảo', N'Hoạt hình', 0, 105, 75000),
(N'Lật Mặt 7: Một Điều Ước', N'Gia đình', 13, 138, 90000),
(N'Hành Tinh Khỉ: Vương Quốc Mới', N'Hành động', 13, 145, 95000),
(N'Vây Hãm: Kẻ Trừng Phạt', N'Hành động', 16, 109, 100000),
(N'Ma Da', N'Kinh dị', 18, 110, 110000),
(N'Deadpool & Wolverine', N'Hành động', 18, 127, 120000),
(N'Inside Out 2', N'Hoạt hình', 0, 96, 80000),
(N'Móng Vuốt', N'Giật gân', 16, 95, 85000);

INSERT INTO KhachHang (HoTen, NgaySinh, Email, DiemTichLuy, HangThanhVien)
VALUES 
(N'Nguyễn Văn Sơn', '2004-05-20', 'son.tnut@gmail.com', 150, 'Member'),
(N'Lê Văn Tiến', '2004-10-15', 'tien.tnut@gmail.com', 50, 'Standard'),
(N'Trần Thị Nhung', '1995-02-28', 'nhung.tc@gmail.com', 600, 'VIP'),
(N'Phạm Minh Hoàng', '2012-08-10', 'hoang.pm@gmail.com', 20, 'Standard'),
(N'Nguyễn Thị Mai', '2000-12-05', 'mai.nt@gmail.com', 120, 'Member'),
(N'Vũ Đức Anh', '1990-03-15', 'anh.vd@gmail.com', 1100, 'VIP'),
(N'Đặng Thu Thảo', '2006-07-22', 'thao.dt@gmail.com', 0, 'Standard'),
(N'Bùi Quang Huy', '1988-11-30', 'huy.bq@gmail.com', 450, 'Member'),
(N'Lý Hải Nam', '2005-01-12', 'nam.lh@gmail.com', 300, 'Member'),
(N'Chu Bảo Ngọc', '2001-09-09', 'ngoc.cb@gmail.com', 850, 'VIP'),
(N'Hoàng Văn Thái', '1998-04-05', 'thai.hv@gmail.com', 45, 'Standard'),
(N'Đỗ Kim Chi', '2003-11-20', 'chi.dk@gmail.com', 210, 'Member'),
(N'Trịnh Gia Bảo', '2015-06-18', 'bao.tg@gmail.com', 10, 'Standard'),
(N'Vương Lệ Quyên', '1997-12-30', 'quyen.vl@gmail.com', 550, 'VIP'),
(N'Phan Thanh Tùng', '1992-05-14', 'tung.pt@gmail.com', 720, 'VIP'),
(N'Tạ Minh Tú', '2004-02-25', 'tu.tm@gmail.com', 180, 'Member'),
(N'Đoàn Thúy Vi', '2000-08-08', 'vi.dt@gmail.com', 95, 'Standard'),
(N'Lương Thế Thành', '1985-10-10', 'thanh.lt@gmail.com', 1500, 'VIP'),
(N'Ngô Bảo Châu', '2004-03-03', 'chau.nb@gmail.com', 320, 'Member'),
(N'Dương Quốc Anh', '2010-01-01', 'anh.dq@gmail.com', 60, 'Standard'),
(N'Trương Mỹ Linh', '1994-07-15', 'linh.tm@gmail.com', 480, 'Member'),
(N'Nguyễn Hồng Đăng', '1989-12-24', 'dang.nh@gmail.com', 1200, 'VIP'),
(N'Lâm Khánh Chi', '1996-06-06', 'chi.lk@gmail.com', 25, 'Standard'),
(N'Hà Việt Dũng', '2002-04-18', 'dung.hv@gmail.com', 110, 'Member'),
(N'Võ Hạ Trâm', '1991-02-12', 'tram.vh@gmail.com', 900, 'VIP'),
(N'Đinh Tiến Đạt', '2004-11-11', 'dat.dt@gmail.com', 35, 'Standard'),
(N'Quách Ngọc Ngoan', '1986-09-25', 'ngoan.qn@gmail.com', 650, 'VIP'),
(N'Sơn Tùng MTP', '1994-07-05', 'tung.mtp@gmail.com', 2000, 'VIP'),
(N'Phương Mỹ Chi', '2003-01-13', 'chi.pm@gmail.com', 400, 'Member'),
(N'Isaac Phạm', '1988-06-13', 'isaac@gmail.com', 75, 'Standard');

INSERT INTO VeXemPhim (MovieID, CustomerID, NgayBan, SuatChieu, GiaThanhToan)
VALUES 
(1, 4, GETDATE(), '2025-11-20 10:00:00', 75000),
(2, 1, GETDATE(), '2025-11-20 19:00:00', 85500),
(5, 3, GETDATE(), '2025-11-20 21:00:00', 99000),
(6, 6, GETDATE(), '2025-11-21 20:00:00', 108000),
(3, 10, GETDATE(), '2025-11-21 18:00:00', 85500),
(8, 12, GETDATE(), '2025-11-22 15:00:00', 80750),
(7, 20, GETDATE(), '2025-11-22 09:00:00', 80000),
(2, 15, GETDATE(), '2025-11-23 19:30:00', 81000),
(4, 21, GETDATE(), '2025-11-23 21:00:00', 95000),
(6, 28, GETDATE(), '2025-11-24 20:30:00', 108000);

SELECT * FROM Phim;
SELECT * FROM KhachHang;
SELECT * FROM VeXemPhim;

SELECT v.TicketID, k.HoTen, p.TenPhim, v.SuatChieu, v.GiaThanhToan
FROM VeXemPhim v
JOIN KhachHang k ON v.CustomerID = k.CustomerID
JOIN Phim p ON v.MovieID = p.MovieID;
```

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f823dd26-022b-498c-811a-1f0a0a3854f2" />
<i>Hình 5: Chèn dữ liệu mẫu thành công</i></p>

 # Phần 2: Xây dựng Function

 **1**. Lý thuyết về Function trong SQL Server
Các loại Built-in Function (Hàm có sẵn)
SQL Server cung cấp hệ thống hàm cực kỳ phong phú, chia làm các nhóm chính:

- Aggregate Functions: Tính toán trên tập dữ liệu (SUM, AVG, COUNT, MAX, MIN).

- String Functions: Xử lý chuỗi (LEN, SUBSTRING, REPLACE, UPPER, LOWER).

- Date and Time Functions: Xử lý thời gian (GETDATE, DATEDIFF, DATEADD, YEAR).

- Mathematical Functions: Hàm toán học (ABS, ROUND, CEILING, FLOOR).

Một số System Function đặc sắc:

- COALESCE(): Trả về giá trị đầu tiên không NULL trong danh sách. Rất hữu ích khi làm báo cáo để thay thế các ô trống bằng giá trị "0" hoặc "Chưa xác định".

- DATEDIFF(): Tính khoảng cách giữa hai mốc thời gian. Trong rạp phim, hàm này cực kỳ quan trọng để tính tuổi khách hàng từ ngày sinh.

Dù hệ thống có nhiều hàm sẵn, chúng ta vẫn cần tự viết hàm riêng vì:

- Đóng gói logic nghiệp vụ: Ví dụ: Hệ thống không có sẵn hàm "Tính giá vé rạp phim sau khi trừ điểm tích lũy", bạn phải tự viết để dùng thống nhất ở mọi nơi.

- Tái sử dụng: Viết một lần, dùng được ở nhiều Store Procedure, View hoặc câu lệnh Select.

- Làm code gọn hơn: Thay vì viết một đoạn IF-ELSE dài trong câu truy vấn, bạn chỉ cần gọi tên hàm.

Các loại UDF:

- Scalar Function: Trả về một giá trị duy nhất (số, chuỗi, ngày). Dùng khi cần tính toán đơn giản.

- Inline Table-Valued Function: Trả về một bảng dữ liệu dựa trên một câu lệnh SELECT duy nhất. Hiệu năng rất cao, dùng để lọc dữ liệu nhanh.

- Multi-statement Table-Valued Function: Trả về một bảng nhưng bên trong có logic phức tạp (khai báo biến, vòng lặp, chèn dữ liệu vào biến bảng). Dùng khi cần xử lý dữ liệu qua nhiều bước trước khi trả kết quả.

**2**. Viết các loại hàm 

**Scalar Function** (Hàm trả về một giá trị)

Yêu cầu: Tính giá vé sau khi giảm giá dựa trên điểm tích lũy của khách hàng.

```
CREATE FUNCTION fn_CalculateTicketPrice (@customerId INT, @basePrice DECIMAL(10, 2))
RETURNS DECIMAL(10, 2)
AS
BEGIN
    DECLARE @finalPrice DECIMAL(10, 2), @points INT;
    SELECT @points = DiemTichLuy FROM KhachHang WHERE CustomerID = @customerId;
    SET @finalPrice = CASE 
        WHEN @points >= 500 THEN @basePrice * 0.90
        WHEN @points >= 100 THEN @basePrice * 0.95
        ELSE @basePrice
    END;
    RETURN @finalPrice;
END;
GO

SELECT HoTen, dbo.fn_CalculateTicketPrice(CustomerID, 100000) AS GiaPhaiTra FROM KhachHang;
```
Các bước xử lý dữ liệu:

- Bước 1 (Input): Nhận vào Mã khách hàng và Giá vé gốc.
- Bước 2 (Truy vấn): Kết nối vào bảng KhachHang để lấy số DiemTichLuy tương ứng với mã đó.
- Bước 3 (Kiểm tra điều kiện):
     * Nếu điểm >= 500 $\rightarrow$ Tính toán: $Giá gốc \times 0.9$.
     * Nếu điểm >= 100 $\rightarrow$ Tính toán: $Giá gốc \times 0.95$.
     * Trường hợp khác $\rightarrow$ Giữ nguyên giá gốc.
- Bước 4 (Output): Trả về một con số duy nhất là Giá vé cuối cùng

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2a61c576-a627-4974-9772-689a8c4992ac" />
<i>Hình 6: Danh sách khách hàng sau khi được giảm giá vé</i></p>

**Inline Table-Valued Function** (Hàm trả về bảng đơn giản)

Yêu cầu: Lấy danh sách các phim theo giới hạn độ tuổi cụ thể
```
CREATE FUNCTION fn_GetMoviesByAge (@ageLimit INT)
RETURNS TABLE
AS
RETURN (
    SELECT MovieID, TenPhim, TheLoai, ThoiLuong 
    FROM Phim 
    WHERE DoTuoiQuyDinh = @ageLimit
);
GO

SELECT * FROM dbo.fn_GetMoviesByAge(13);
```

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be38ca7f-871b-49a2-a2d3-a3f92b5cd20e" />
<i>Hình 7: Danh sách các phim được lấy</i></p>

Các bước xử lý dữ liệu:

- Bước 1 (Input): Nhận vào tham số Độ tuổi giới hạn (ví dụ: 13).

- Bước 2 (Xử lý): SQL Server thực thi một câu lệnh SELECT duy nhất với điều kiện WHERE DoTuoiQuyDinh = tham số.

- Bước 3 (Output): Trả về ngay lập tức một bảng ảo chứa danh sách các bộ phim thỏa mãn.

- Đặc điểm: Không có thân hàm (BEGIN...END), tốc độ truy xuất cực nhanh vì nó hoạt động giống như một View có tham số.


**Multi-statement Table-Valued Function** (Hàm trả về bảng logic phức tạp):

Yêu cầu: Thống kê chi tiết khách hàng và phân loại họ thành các nhóm "Tiềm năng", "Thân thiết", "Vip" dựa trên cả số vé đã mua và điểm tích lũy.


```

CREATE FUNCTION fn_GetCustomerAnalytics ()
RETURNS @AnalyticsTable TABLE (
    CustomerID INT,
    HoTen NVARCHAR(100),
    TotalTickets INT,
    Points INT,
    Classification NVARCHAR(50)
)
AS
BEGIN
    INSERT INTO @AnalyticsTable
    SELECT 
        k.CustomerID, 
        k.HoTen, 
        COUNT(v.TicketID), 
        k.DiemTichLuy,
        N'Chưa xác định'
    FROM KhachHang k
    LEFT JOIN VeXemPhim v ON k.CustomerID = v.CustomerID
    GROUP BY k.CustomerID, k.HoTen, k.DiemTichLuy;

    UPDATE @AnalyticsTable
    SET Classification = CASE 
        WHEN Points >= 500 AND TotalTickets >= 5 THEN N'Khách hàng VIP'
        WHEN Points >= 100 OR TotalTickets >= 2 THEN N'Khách hàng Thân thiết'
        ELSE N'Khách hàng Tiềm năng'
    END;

    RETURN;
END;
GO

SELECT * FROM dbo.fn_GetCustomerAnalytics();
```

Các bước thực hiện:

- Bước 1 (Khởi tạo): Tạo ra một biến bảng tạm (@AnalyticsTable) với các cột định sẵn.

- Bước 2 (Đổ dữ liệu): * Thực hiện JOIN giữa bảng KhachHang và VeXemPhim.

  - Sử dụng hàm COUNT để đếm số vé và lấy điểm tích lũy.

  - INSERT toàn bộ dữ liệu thô này vào bảng tạm.

- Bước 3 (Xử lý logic phức tạp): * Chạy lệnh UPDATE trên bảng tạm.

  - Sử dụng khối CASE WHEN để đánh giá: Khách có nhiều vé + nhiều điểm thì gắn nhãn "VIP", ít hơn thì gắn "Thân thiết" hoặc "Tiềm năng".

- Bước 4 (Output): Trả về toàn bộ nội dung của bảng tạm đã được cập nhật nhãn phân loại.

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/abf85277-218a-430e-8f65-09efa06ac291" />
<i>Hình 8: Trả về danh sách khách hàng thân thiết </i></p>

# Phần 3: Xây dựng Store Procedure

1. Tìm hiểu về System Stored Procedure (SP có sẵn)

- Trong SQL Server, các System SP là các thủ tục được hệ thống xây dựng sẵn để hỗ trợ quản trị và truy xuất siêu dữ liệu (metadata). Chúng thường bắt đầu bằng tiền tố sp_.

- Một số System SP đặc sắc:

  - sp_help: Dùng để xem thông tin chi tiết về một đối tượng (bảng, view, index...). Nó trả về kiểu dữ liệu của các cột, khóa chính, khóa ngoại.

    - Cách dùng: EXEC sp_help 'Tour';
  - sp_helpdb: Cung cấp thông tin về dung lượng, đường dẫn tệp tin và trạng thái của các cơ sở dữ liệu trên Server.
    
    - Cách dùng: EXEC sp_helpdb 'QuanLyTourDuLich_K235480206061';
  - sp_columns: Trả về danh sách chi tiết tất cả các cột trong một bảng nhất định.

    - Cách dùng: EXEC sp_columns 'DangKyTour';
  - sp_rename: Dùng để đổi tên một đối tượng (như bảng hoặc cột) mà không cần phải xóa đi tạo lại.

    - Cách dùng: EXEC sp_rename 'OldName', 'NewName';
      
2. Thực hành xây dựng
   
**Stored Procedure SP Thực hiện INSERT có kiểm tra điều kiện logic**

Yêu cầu: Xây dựng sp_InsertMovie để thêm một bộ phim mới. Điều kiện: Tên phim không được trùng và thời lượng phải lớn hơn 60 phút.

```
CREATE PROCEDURE sp_InsertMovie
    @tenPhim NVARCHAR(200),
    @theLoai NVARCHAR(50),
    @doTuoi INT,
    @thoiLuong INT,
    @giaVe DECIMAL(10, 2)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Phim WHERE TenPhim = @tenPhim)
    BEGIN
        PRINT N'Loi: Ten phim da ton tai!';
        RETURN;
    END

    IF @thoiLuong <= 60
    BEGIN
        PRINT N'Loi: Thoi luong phai tren 60 phut!';
        RETURN;
    END

    INSERT INTO Phim (TenPhim, TheLoai, DoTuoiQuyDinh, ThoiLuong, GiaVeCoBan)
    VALUES (@tenPhim, @theLoai, @doTuoi, @thoiLuong, @giaVe);
    
    PRINT N'Them phim thanh cong!';
END;
GO

EXEC sp_InsertMovie N'Lật Mặt 8', N'Hành động', 16, 120, 95000;
```
Luồng xử lý:
- Tiếp nhận: Nhận 5 tham số đầu vào từ người dùng (Tên, thể loại, tuổi, thời lượng, giá).
- Kiểm tra trùng lặp: Quét bảng Phim, nếu thấy TenPhim đã có thì thông báo lỗi và dừng (Return).Kiểm tra nghiệp vụ:
- Xác thực ThoiLuong, nếu $\le 60$ phút thì chặn lại vì không đúng chuẩn phim điện ảnh.
- Thực thi: Nếu vượt qua 2 bước kiểm tra, hệ thống thực hiện lệnh INSERT vào database.

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/58ddf023-d232-460e-89fe-792801f1c31f" />
<i>Hình 9: Thực hiện lệnh thành công và thêm 1 bộ phim mới</i></p>

**Store Procedure sử dụng tham số OUTPUT**

Yêu cầu: xây dựng sp_GetCustomerTotalSpent để phục vụ bộ phận kế toán và CSKH. 
Hệ thống cần tính tổng số tiền mà một khách hàng cụ thể đã chi trả cho tất cả các vé họ đã mua từ trước đến nay. Kết quả phải được trả về dưới dạng một biến để có thể tái sử dụng cho các mục đích khác (như xét nâng hạng thành viên).

```
CREATE PROCEDURE sp_GetCustomerTotalSpent
    @customerId INT,
    @totalSpent DECIMAL(10, 2) OUTPUT
AS
BEGIN
    SELECT @totalSpent = SUM(GiaThanhToan)
    FROM VeXemPhim
    WHERE CustomerID = @customerId;

    IF @totalSpent IS NULL 
        SET @totalSpent = 0;
END;
GO

DECLARE @result DECIMAL(10, 2);
EXEC sp_GetCustomerTotalSpent 1, @result OUTPUT;
SELECT @result AS TongTienDaChi;
```
Luồng xử lý:

- Khởi tạo: Nhận Mã khách hàng và chuẩn bị một biến OUTPUT để chứa kết quả trả về.

- Tính toán: Sử dụng hàm gộp SUM trên cột GiaThanhToan trong bảng VeXemPhim lọc theo mã khách hàng.

- Xử lý rỗng: Nếu khách hàng chưa mua vé bao giờ (NULL), hàm sẽ gán giá trị mặc định là 0.

- Trả kết quả: Đẩy giá trị vừa tính được ra biến @totalSpent để câu lệnh gọi thủ tục có thể sử dụng tiếp.

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5622bd35-3a4e-47ea-8044-4b3f22b695a3" />
<i>Hình 10: Kết quả của 1 khách hàng đã chi từ trước đến nay</i></p>

**Store Procedure trả về tập kết quả**

Yêu cầu: Xây dựng thủ tục sp_GetCustomerHistory để hỗ trợ nhân viên tại quầy tra cứu nhanh lịch sử xem phim của khách. 
Báo cáo trả về phải hiển thị rõ ràng tên khách hàng và tên phim (thay vì chỉ hiện mã ID), kèm theo suất chiếu và giá vé, sắp xếp theo thời gian gần nhất.
```
CREATE PROCEDURE sp_GetCustomerHistory
    @customerId INT
AS
BEGIN
    SELECT 
        v.TicketID,
        k.HoTen,
        p.TenPhim,
        p.TheLoai,
        v.SuatChieu,
        v.GiaThanhToan
    FROM VeXemPhim v
    JOIN KhachHang k ON v.CustomerID = k.CustomerID
    JOIN Phim p ON v.MovieID = p.MovieID
    WHERE k.CustomerID = @customerId
    ORDER BY v.SuatChieu DESC;
END;
GO

EXEC sp_GetCustomerHistory 1;

```
Luồng xử lý:

- Kiểm tra tồn tại: Trước khi khởi tạo, hệ thống kiểm tra trong sys.objects xem thủ tục đã tồn tại chưa. Nếu có, thực hiện DROP để làm mới bộ nhớ.

- Định vị Database: Đảm bảo phiên làm việc đang ở đúng Database rạp phim để tránh lỗi "Invalid object name".

- Kết hợp dữ liệu (JOIN): Thực hiện INNER JOIN giữa bảng VeXemPhim, KhachHang và Phim.

- Lọc & Sắp xếp: Trích xuất các bản ghi theo @customerId và sắp xếp theo suất chiếu mới nhất.

- Trả kết quả: Xuất ra tập bản ghi (Result Set) cho người dùng.
<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f17ff1ee-1752-430b-9287-1ee928533b04" />
<i>Hình 12: Tra cứu nhanh tên khách hàng và tên phim </i></p>
