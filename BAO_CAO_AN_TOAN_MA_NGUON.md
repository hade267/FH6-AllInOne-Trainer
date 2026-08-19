# BÁO CÁO ĐÁNH GIÁ VÀ KIỂM TRA AN TOÀN MÃ NGUỒN (FH6 ALL-IN-ONE TRAINER)

**Ngày thực hiện đánh giá:** 19/08/2026
**Dự án:** FH6 All-in-One Trainer (`FH6AllInOneTrainer.exe`)
**Ngôn ngữ lập trình:** C# (.NET 10.0, Avalonia UI, Windows x64)
**Phạm vi kiểm tra:** Toàn bộ mã nguồn dự án (Codebase, Dependencies, Native APIs, Memory Operations, Network Activities, File I/O).

---

## I. TỔNG QUAN VÀ KẾT LUẬN CHÍNH (EXECUTIVE SUMMARY)

Sau khi rà soát và phân tích chi tiết 100% tệp tin mã nguồn trong dự án, **kết luận chính thức** như sau:

1. **MÃ NGUỒN KHÔNG CHỨA VIRUS, BOTNET, STEALER HOẶC MÃ ĐỘC ĐÁNH CẮP DỮ LIỆU:**
   - **Không có Botnet / C2 Server:** Mã nguồn không hề chứa bất kỳ cơ chế tạo backdoor, không lắng nghe cổng mạng (Socket/TCP Server), không nhận lệnh từ xa để điều khiển máy tính.
   - **Không có Stealer / Spyware:** Không có đoạn mã nào truy cập tệp trình duyệt, mật khẩu mã hóa, token Discord, dữ liệu ví tiền điện tử, hay thông tin cá nhân/hệ thống của người dùng.
   - **Không có Keylogger / Screen Capture:** Không có lệnh hook bàn phím toàn cục (Keyboard Hooks) hay chụp ảnh màn hình.

2. **BẢN CHẤT CỦA CHƯƠNG TRÌNH:**
   - Đây là một **Game Trainer** (công cụ gian lận game offline cho Forza Horizon 6) hoạt động hoàn toàn công khai và mã nguồn mở (Open Source GPL-3.0).
   - Công cụ này sử dụng các kỹ thuật can thiệp bộ nhớ (Memory Manipulation) như: `OpenProcess`, `ReadProcessMemory`, `WriteProcessMemory`, `VirtualAllocEx`, `CreateRemoteThread`, và ghi đè shellcode byte để bỏ qua cơ chế kiểm tra tính toàn vẹn (CRC Bypass) của game.

3. **CẢNH BÁO BÁO NHẦM CỦA PHẦN MỀM DIỆT VIRUS (FALSE POSITIVE):**
   - Các phần mềm diệt virus (Windows Defender, Kaspersky, Avast, v.v.) **chắc chắn sẽ nhận diện nhầm file .exe sau khi biên dịch là Virus/Trojan/HackTool/GameHack**.
   - Nguyên nhân là do các hành vi: can thiệp bộ nhớ tiến trình khác (`OpenProcess`), chèn luồng thực thi từ xa (`CreateRemoteThread`), phân bổ bộ nhớ thực thi (`PAGE_EXECUTE_READWRITE`). Đây là các hành vi kỹ thuật bắt buộc của một Game Trainer, nhưng cũng trùng khớp với dấu hiệu của một số loại malware/injector.

---

## II. CHI TIẾT KẾT QUẢ ĐÁNH GIÁ THEO CÁC TIÊU CHÍ AN TOÀN

### 1. Phân tích kết nối mạng (Network Activity)
* **Tệp tin liên quan:** `Services/UpdateCheckService.cs`
* **Hành vi kiểm tra:**
  - Dự án chỉ có **duy nhất một kết nối HTTP** duy nhất tới GitHub API:
    `https://api.github.com/repos/changcheng967/FH6-AllInOne-Trainer/releases/latest`
  - Mục đích: Lấy thông tin phiên bản mới nhất (`tag_name`) để so sánh với phiên bản hiện tại của chương trình khi khởi chạy.
  - Thông tin gửi đi: Chỉ gửi tên `User-Agent: FH6AllInOne-Updater/<version>` theo chuẩn GitHub API (không gửi bất kỳ dữ liệu cá nhân, IP riêng, hay thông tin máy tính nào).
  - Không tải file .exe từ xa về tự động chạy: Nếu có bản cập nhật, nút nhấn trên UI chỉ mở trình duyệt mặc định điều hướng tới trang Releases của GitHub (`Process.Start`).

### 2. Phân tích can thiệp bộ nhớ game (Memory Manipulation & Injection)
* **Tệp tin liên quan:** `Cheats/RuntimeHook/Native.cs`, `RuntimeHookEngine.cs`, `RewardCaller.cs`, `Cheats/Sql/SqlExecutor.cs`, `MemoryScanner.cs`.
* **Phân tích chi tiết:**
  - `OpenProcess(PROCESS_ALL_ACCESS, ...)`: Mở tiến trình game `ForzaHorizon6.exe` với quyền Admin để đọc/ghi bộ nhớ.
  - `WriteProcessMemory` / `VirtualAllocEx`: Đưa các đoạn shellcode x64 ngắn (ví dụ: lệnh `ret` stub để bypass kiểm tra CRC của game, hoặc gọi hàm thi hành câu lệnh SQLite `CDatabase::ExecuteQuery`) vào vùng nhớ game.
  - `CreateRemoteThread`: Tạoluồng từ xa trong game để thi hành đoạn shellcode ngắn đã chèn, sau đó giải phóng bộ nhớ (`VirtualFreeEx`).
  - **Đánh giá:** Các thao tác này chỉ tác động duy nhất vào vùng bộ nhớ RAM của tiến trình `ForzaHorizon6.exe`. Chương trình **không can thiệp hay tiêm mã vào bất kỳ tiến trình hệ thống nào khác** (như `explorer.exe`, `svchost.exe`, v.v.).

### 3. Phân tích đọc/ghi tệp tin & Hệ thống (File I/O & System Registry)
* **Tệp tin liên quan:** `Services/AppSettings.cs`, `Services/ProfileService.cs`, `Services/LogService.cs`, `Cheats/Scan/SavedPointerStore.cs`.
* **Phân tích chi tiết:**
  - Chương trình **không hề truy cập Windows Registry** (`RegistryKey`, `RegOpenKey`).
  - Chương trình chỉ đọc/ghi tệp tin tại 2 vị trí cục bộ rõ ràng:
    1. `%APPDATA%\FH6AllInOneTrainer\`: Lưu cấu hình giao diện (`settings.json`) và các hồ sơ lưu cheat (`profiles/*.json`).
    2. Thư mục chạy ứng dụng: Ghi tệp nhật ký `trainer.log` để theo dõi lỗi trong quá trình sử dụng.
  - Không tạo các file ẩn, không sao chép chính nó vào `Startup` hay `System32` để tự khởi động cùng Windows.

### 4. Phân tích thư viện phụ thuộc (Dependencies Analysis)
* **Tệp tin liên quan:** `FH6Mod.csproj`
* **Các thư viện chính:**
  - `Avalonia` (v12.1.1): Thư viện giao diện người dùng mã nguồn mở (UI Framework).
  - `CommunityToolkit.Mvvm` (v8.4.2): Thư viện MVVM chính thức từ Microsoft.
  - `Reloaded.Memory` & `Reloaded.Memory.Sigscan`: Thư viện mã nguồn mở uy tín chuyên dùng để quét Pattern/AOB (Array of Bytes) trong bộ nhớ game.
  - Tất cả gói NuGet đều là các thư viện chính thống, phổ biến và an toàn.

---

## III. TỔNG KẾT BÁO CÁO

| Hạng mục kiểm tra | Trạng thái | Đánh giá |
| :--- | :---: | :--- |
| **Mã độc Botnet / C2** | **KHÔNG CÓ** | An toàn 100% |
| **Mã đánh cắp dữ liệu (Stealer)** | **KHÔNG CÓ** | An toàn 100% |
| **Mã theo dõi bàn phím / Màn hình** | **KHÔNG CÓ** | An toàn 100% |
| **Mã tải file độc hại từ xa** | **KHÔNG CÓ** | An toàn 100% |
| **Tác động tiến trình khác** | **KHÔNG CÓ** | Chỉ tác động duy nhất `ForzaHorizon6.exe` |
| **Cảnh báo Antivirus** | **CÓ THỂ BỊ BÁO NHẦM** | Do hành vi Memory Injection đặc trưng của Trainer |

**Khuyên dùng:**
Mã nguồn này **hoàn toàn sạch sẽ và an toàn để biên dịch và sử dụng**. Do chương trình là một Game Trainer can thiệp bộ nhớ, nếu bạn tự biên dịch hoặc sử dụng bản phát hành, bạn có thể cần thêm ngoại lệ (Exclusion) trong Windows Defender / Antivirus để tránh bị chặn nhầm.
