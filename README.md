# Synchronous Data RAM - RISC-V RV32I

[![Language](https://img.shields.io/badge/Language-SystemVerilog-blue.svg)](https://en.wikipedia.org/wiki/SystemVerilog)
[![Standard](https://img.shields.io/badge/Standard-IEEE%201800--2012%2F2017-brightgreen.svg)]()
[![Target ISA](https://img.shields.io/badge/ISA-RISC--V%20RV32I-red.svg)](https://riscv.org/)
[![Tool](https://img.shields.io/badge/Verified%20with-Vivado%202022.2-orange.svg)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

Bộ nhớ dữ liệu **Data RAM (`data_ram`)** đóng vai trò là bộ nhớ chính trong tầng truy xuất bộ nhớ (**Memory Access - MEM stage**) của kiến trúc Harvard trong bộ xử lý RISC-V RV32I. Thiết kế hỗ trợ ghi đồng bộ theo xung nhịp và đọc tổ hợp bất đồng bộ cho các lệnh `LW` và `SW`.

---

## 📌 Đặc tả Chức năng & Sơ đồ Khối

```
                       +------------------------+
      addr [31:0] ---->|                        |
write_data [31:0] ---->|        data_ram        |-----> read_data [31:0]
  MemWrite ----------->|   DEPTH = 256 words    |
       clk ----------->|                        |
     rst_n ----------->|                        |
                       +------------------------+
```

### ⚡ Nguyên lý Hoạt động:
1. **Ghi đồng bộ (Synchronous Write):**
   - Thao tác ghi dữ liệu chỉ diễn ra tại sườn dương `posedge clk`.
   - Điều kiện ghi hợp lệ: Tín hiệu `MemWrite == 1` và địa chỉ căn chỉnh đúng word (`addr[1:0] == 2'b00`).
2. **Đọc tổ hợp (Combinational Read):**
   - Dữ liệu `read_data` được xuất ra ngay lập tức khi địa chỉ `addr` hợp lệ (`addr[1:0] == 2'b00` và `addr[31:2] < DEPTH`).
   - Địa chỉ không căn lề hoặc ngoài phạm vi bộ nhớ sẽ trả về `32'b0`.
3. **Reset Toàn phần (Full Array Reset):**
   - Tín hiệu `rst_n` tích cực mức thấp sẽ xóa toàn bộ nội dung của các ô nhớ trong RAM về `0x00000000`.

---

## 🔌 Đặc tả Cổng Giao tiếp & Tham số

### Parameter:
| Tên Parameter | Kiểu dữ liệu | Giá trị mặc định | Ý nghĩa |
| :--- | :---: | :---: | :--- |
| `DEPTH` | `int` | `256` | Số lượng từ nhớ 32-bit (256 words = 1 KB) |

### Ports:
| Tên cổng | Hướng (Direction) | Độ rộng bit | Ý nghĩa |
| :--- | :---: | :---: | :--- |
| `clk` | Input | `1` | Xung nhịp hệ thống (ghi đồng bộ tại posedge) |
| `rst_n` | Input | `1` | Tín hiệu Reset tích cực mức thấp |
| `addr` | Input | `[31:0]` | Địa chỉ byte bộ nhớ tính toán từ ALU |
| `write_data` | Input | `[31:0]` | Dữ liệu cần ghi vào RAM (từ thanh ghi `rs2`) |
| `MemWrite` | Input | `1` | Tín hiệu cho phép ghi RAM từ Control Unit |
| `read_data` | Output | `[31:0]` | Dữ liệu đọc từ RAM đưa về Write-Back |

---

## 🧪 Kiểm chứng & Mô phỏng (Verification)

Testbench `testbench/tb_data_ram.sv` kiểm tra đầy đủ:
- Reset toàn bộ RAM về 0.
- Ghi và đọc kiểm tra tại nhiều địa chỉ word liên tiếp.
- Kiểm tra tính bất biến của dữ liệu khi `MemWrite = 0`.
- Kiểm tra cơ chế từ chối ghi khi địa chỉ không căn lề (`misaligned address`).

### Lệnh chạy mô phỏng:

```bash
xvlog -sv rtl/data_ram.sv testbench/tb_data_ram.sv
xelab data_ram_tb -s ram_sim
xsim ram_sim -R
```

---

## 📂 Cấu trúc Thư mục Repo

```
.
├── rtl/
│   └── data_ram.sv        # RTL Data RAM
├── testbench/
│   └── tb_data_ram.sv     # Self-checking testbench
├── .gitignore
└── README.md
```

---

## 👨‍💻 Thông tin Tác giả & Đồ án

- **Sinh viên thực hiện:** Nguyễn Thành Trung
- **Học phần:** Đồ án Môn học 2 (Capstone Project II) – Ngành Kỹ thuật Máy tính
- **Tên đề tài:** Thiết kế, kiểm chứng và triển khai FPGA lõi vi xử lý RISC-V RV32I 32-bit pipeline 5 tầng ở mức RTL bằng SystemVerilog
- **GitHub cá nhân:** [@thanhchun2005-blip](https://github.com/thanhchun2005-blip)
- **Email:** thanhchun2005@gmail.com
