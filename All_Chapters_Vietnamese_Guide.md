# 📘 HƯỚNG DẪN POWER BI CHO TÀI CHÍNH
## *"Làm trước → Học sau"* — Phiên bản tiếng Việt, ví dụ ABC Coffee

> **Dành cho:** Kế toán, Kiểm toán, Financial Analyst muốn tự xây báo cáo tài chính trên Power BI mà **không cần background IT sâu**.
>
> **Phong cách:** Mỗi chương đều có **2 phần rõ ràng**
> - 🛠 **LÀM TRƯỚC** — bạn click chuột theo từng bước, ra sản phẩm ngay
> - 💡 **HỌC SAU** — bạn hiểu **vì sao** lại làm thế, lý thuyết đằng sau
>
> **Ví dụ xuyên suốt:** Công ty **ABC Coffee** — chuỗi quán cà phê nhỏ có 3 cửa hàng, dùng để minh họa cho mọi DAX/Power Query.

---

## 📑 MỤC LỤC

| # | Chương | Trọng tâm |
|---|--------|-----------|
| 1 | Xác định nhu cầu báo cáo | Phạm vi dự án, stakeholder, MoSCoW |
| 2 | Mô hình cơ sở cho báo cáo tài chính | Star schema, Calendar, CoA, GL |
| 3 | Tạo Trial Balance & Báo cáo thu nhập | SUM, SUMX, Matrix visual |
| 4 | Các measure DAX thường gặp | CALCULATE, time intelligence, FX |
| 5 | Xử lý bút toán thủ công | Journal header/lines, role-playing dim |
| 6 | Tinh gọn với Power Query | ETL, Merge, Pivot/Unpivot |
| 7 | Ngân sách & Mục tiêu | Budget vs Actual, variance | *(đã dịch ở session trước)* |
| 8 | Quản lý dữ liệu tồn kho | Snapshot vs transactional, Stock Turn |
| 9 | Bộ báo cáo tài chính | 5 nguyên tắc thiết kế, P&L report |
| 10 | Kiểm thử & tinh chỉnh dữ liệu | Reconciliation, DAX typo |
| 11 | Báo cáo in được với Paginated | Power BI Report Builder |
| 12 | Báo cáo tài chính trong thế giới AI | Copilot, Fabric, rủi ro |
| 13 | Vận hành & bảo trì | Workspaces, deployment, RLS, refresh |

---

# CHƯƠNG 1: XÁC ĐỊNH NHU CẦU BÁO CÁO CỦA BẠN

## 🎯 Mục tiêu chương
- Trả lời được: *"Dự án Power BI này giải quyết vấn đề gì?"*
- Tránh xây dự báo cáo **không ai dùng** — bẫy phổ biến nhất của dân tài chính khi mới làm BI.

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (3 bước)

### Bước 1.1 — Mở file Excel, viết ra **5 câu hỏi** mà sếp hỏi bạn nhiều nhất
> *ABC Coffee — Cửa hàng 1, 2, 3 đang hoạt động. Sếp bạn là CEO.*

Ví dụ:
1. Doanh thu **tháng này** của **từng cửa hàng** là bao nhiêu?
2. Lợi nhuận gộp tháng này so với **cùng kỳ năm ngoái**?
3. Chi phí mặt bằng cửa hàng 2 chiếm bao nhiêu % doanh thu?
4. Dòng tiền cuối tháng dự kiến còn bao nhiêu?
5. Sản phẩm nào bán chạy nhất ở cửa hàng 1?

➡ **Đây chính là backlog** cho dự án Power BI của bạn.

### Bước 1.2 — Phân loại theo **MoSCoW**
Mở Excel, thêm 2 cột: `MUST / SHOULD / COULD / WON'T`:

| Câu hỏi | Phân loại | Lý do |
|---|---|---|
| Doanh thu theo cửa hàng | **MUST** | Báo cáo doanh thu hàng tháng — bắt buộc |
| Lợi nhuận gộp YoY | **MUST** | CEO cần đánh giá tăng trưởng |
| Chi phí mặt bằng / doanh thu | SHOULD | Có thì tốt, không có thì vẫn OK tháng đầu |
| Dòng tiền dự kiến | SHOULD | Làm thủ công Excel vẫn được |
| Sản phẩm bán chạy | COULD | Làm sau khi xong MUST |

➡ **Quy tắc vàng:** MVP (Minimum Viable Product) chỉ chứa **MUST** — bằng không dự án kéo dài vô tận.

### Bước 1.3 — Vẽ sơ đồ **stakeholder**
```
                  ┌─────────────┐
                  │    CEO      │  ← Consumer (xem báo cáo)
                  └──────┬──────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼─────┐    ┌─────▼────┐    ┌─────▼────┐
   │ Kế toán  │    │ QL cửa   │    │ Phân     │
   │ trưởng   │    │ hàng     │    │ tích KD  │
   └────┬─────┘    └────┬─────┘    └─────┬────┘
        │               │               │
        │          ┌────▼─────┐         │
        │          │ Barista  │         │
        │          │ (nhập liệu)       │
        │          └──────────┘         │
        │                                │
   Creator/                          Creator
   Developer                         (tự tạo
   (build                           báo cáo
   dataset)                          phụ)
```

Trong Power BI, mỗi nhóm này cần **mức độ truy cập khác nhau** (xem Ch 13 về RLS).

---

## 💡 PHẦN 2 — HỌC SAU

### 1.1 Các "vai" trong một dự án BI
| Vai | Ví dụ ABC Coffee | Quyền trong Power BI |
|---|---|---|
| **Consumer** | CEO xem dashboard | Xem báo cáo trên app/web |
| **Creator** | Phân tích KD tự tạo báo cáo phụ | Tạo báo cáo trên semantic model có sẵn |
| **Developer** | Bạn — kế toán trưởng | Build model, DAX, dataflow |
| **Admin** | IT | Quản lý workspace, gateway, license |

### 1.2 Self-service — 3 cấp độ trưởng thành
```
Cấp 1: Tự tạo file Excel riêng lẻ
         ↓
Cấp 2: Chia sẻ Excel qua email/SharePoint
         ↓
Cấp 3: Một semantic model chuẩn (Power BI),
        nhiều báo cáo phái sinh
```
➡ ABC Coffee **đang ở cấp 2**, mục tiêu 6 tháng tới là **cấp 3**.

### 1.3 Nguồn dữ liệu — checklist cần liệt kê
- [ ] Sổ cái (General Ledger) từ phần mềm kế toán (MISA, FAST, SAP…)
- [ ] Sổ phụ: phải thu, phải trả, tồn kho
- [ ] File Excel/Google Sheet phòng KD nhập hàng ngày
- [ ] CRM (lịch sử khách hàng)
- [ ] POS (doanh thu từng cửa hàng)
- [ ] Ngân hàng (sổ phụ ngân hàng)

### 1.4 Lỗi thường gặp
- ❌ Xây hết 100% yêu cầu → 6 tháng mới ra báo cáo đầu tiên
- ❌ Không hỏi stakeholder → làm ra KPI họ không quan tâm
- ❌ Bỏ qua RLS (Row-Level Security) → kế toán cửa hàng 2 thấy được dữ liệu cửa hàng 3
- ✅ MVP 4-6 tuần, MUST trước, lặp lại mỗi sprint 2 tuần

---

## ✅ Checklist thực hành Chương 1
- [ ] Tôi đã viết ra ≥5 câu hỏi của sếp
- [ ] Đã phân loại MoSCoW
- [ ] Đã vẽ sơ đồ stakeholder
- [ ] Đã liệt kê nguồn dữ liệu

---

# CHƯƠNG 2: MÔ HÌNH CƠ SỞ CHO BÁO CÁO TÀI CHÍNH

## 🎯 Mục tiêu
Xây dựng **"trái tim"** của mọi báo cáo tài chính: **star schema** gồm Calendar, Chart of Accounts, GL Transactions.

---

## 🛠 PHẦN 1 — LÀM TRƯỚC

### Bước 2.1 — Tạo bảng **Calendar (Lịch)** cho ABC Coffee
Trong Power BI Desktop → **Modeling** → **New Table**, gõ:

```dax
Calendar = 
ADDCOLUMNS(
    CALENDAR(DATE(2024,1,1), DATE(2026,12,31)),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month", MONTH([Date]),
    "MonthName", FORMAT([Date], "mmm"),
    "Day", DAY([Date]),
    "WeekNum", WEEKNUM([Date]),
    "YearMonth", FORMAT([Date], "YYYY-MM")
)
```

> Đối với công ty Việt Nam cuối năm tài chính = 31/12 → dùng hàm trên là đủ. Nếu làm cho cty Mỹ fiscal year tháng 10, đọc phần "4-4-5 calendar" bên dưới.

### Bước 2.2 — Tạo bảng **ChartOfAccounts (CoA)** — Hệ thống tài khoản
Import từ Excel `DanhMucTaiKhoan.xlsx`:

| AccountKey | AccountCode | AccountName | AccountCategory | AccountType | Level |
|---|---|---|---|---|---|
| 1 | 1000 | Tiền mặt | Tài sản | 1 | 1 |
| 2 | 1100 | Tiền gửi NH | Tài sản | 1 | 1 |
| 3 | 4000 | Doanh thu bán hàng | Doanh thu | 4 | 1 |
| 4 | 5000 | Giá vốn hàng bán | Giá vốn | 5 | 1 |
| 5 | 6000 | Chi phí lương | Chi phí | 6 | 1 |
| 6 | 7000 | Chi phí thuê MB | Chi phí | 6 | 1 |

**Mẹo:** Giữ **AccountKey** là số nguyên (1, 2, 3…) làm primary key — **không dùng AccountCode** vì có thể chứa ký tự "-".

### Bước 2.3 — Import **GLTransactions** — Sổ cái

| TransID | Date | AccountKey | Debit | Credit | Description | StoreKey |
|---|---|---|---|---|---|---|
| 1 | 2025-01-02 | 1 | 5,000,000 | 0 | Bán hàng ngày 02/01 | 1 |
| 2 | 2025-01-02 | 3 | 0 | 8,000,000 | Doanh thu bán cà phê | 1 |
| 3 | 2025-01-03 | 5 | 12,000,000 | 0 | Trả lương NV tháng 12 | 1 |
| 4 | 2025-01-05 | 2 | 8,000,000 | 0 | Chuyển khoản NH | 1 |
| 5 | 2025-01-05 | 1 | 0 | 8,000,000 | Rút tiền mặt gửi NH | 1 |

### Bước 2.4 — Kéo relationship trong Model view
```
   ┌──────────────┐ 1      N ┌────────────────┐
   │  Calendar    │─────────│ GLTransactions │
   │  (Date*)     │         │ (Date)         │
   └──────────────┘         └────────────────┘
                                     │ N
                                     │
                                     │ 1
                              ┌──────▼────────┐
                              │ ChartOfAccounts│
                              │ (AccountKey*) │
                              └───────────────┘
```
- Click kéo từ `Calendar[Date]` → `GLTransactions[Date]`: cardinality **1:N**, cross filter **Single**
- Click kéo từ `ChartOfAccounts[AccountKey]` → `GLTransactions[AccountKey]`: cardinality **1:N**

### Bước 2.5 — Tạo cột **TotalAmount** (Số tiền ròng — dấu âm cho bên Có)
```dax
TotalAmount = GLTransactions[Debit] - GLTransactions[Credit]
```

---

## 💡 PHẦN 2 — HỌC SAU

### 2.1 Star schema vs Snowflake
```
Star (khuyên dùng)              Snowflake (tránh)
  Dim  ─┐   ┌─ Dim               Dim  ─→  SubDim
         │   │                          
       Fact  Fact                      Fact
```
- **Star** = 1 fact table, nhiều dim phẳng → DAX nhanh, dễ hiểu
- **Snowflake** = dim có dim-con → phải JOIN nhiều, chậm hơn

### 2.2 Totaling Accounts (Tài khoản tổng hợp) — cách mạnh hơn Hierarchy
Thay vì xếp cấp cha-con bằng `ParentID`, hãy tạo **1 row tổng hợp riêng**:

| AccountKey | AccountName | Sign |
|---|---|---|
| 1 | 1000 - Tiền mặt | 1 |
| 2 | 1100 - Tiền gửi NH | 1 |
| 100 | **Tổng tiền & tương đương tiền** | 1 |
| 3 | 4000 - Doanh thu bán hàng | -1 |

Trong GL Transactions, thêm cột `SignedAmount = IF(RELATED(ChartOfAccounts[Sign])=1, [TotalAmount], -[TotalAmount])`. Khi SUM(SignedAmount) → chi phí có dấu âm, doanh thu có dấu dương → hiển thị matrix đẹp hơn.

### 2.3 Multi-company: Approach 1 vs Approach 2
**Approach 1 — Cột Company trong CoA** (cho tập đoàn nhỏ)
```
ChartOfAccounts:  |  AccountKey  |  AccountCode  |  Company  |  AccountName
                  |  1           |  1000-VN      |  VN       |  Tiền mặt VN
                  |  2           |  1000-US      |  US       |  Cash US
GLTransactions:   |  TransID  |  AccountKey  |  Amount
```

**Approach 2 — Mỗi công ty 1 dòng phát sinh riêng** (chuẩn hơn, dùng cho SAP/Oracle)
```
GLTransactions:   |  TransID  |  Company  |  AccountKey  |  Amount
```
➡ ABC Coffee: chỉ có 1 công ty → **Approach 1**, đơn giản.

### 2.4 Foreign Exchange (Quy đổi ngoại tệ)
- Tạo bảng `FxRates(Date, Currency, RateToVND)`
- Bước 1: Spot rate tại **ngày giao dịch** → dùng `LOOKUPVALUE`
- Bước 2: Average rate trong **kỳ báo cáo** → dùng `AVERAGEX`

(Xem chi tiết ở **Chương 4**.)

### 2.5 Lỗi thường gặp
- ❌ Dùng `AccountCode` (text) làm primary key → chậm
- ❌ Snowflake schema → DAX chậm
- ❌ Quên relationship `Calendar → GLTransactions` → time intelligence **không hoạt động**
- ✅ Luôn có **Date dimension table** với cột Date duy nhất liên tục

---

## ✅ Checklist Chương 2
- [ ] Đã tạo bảng `Calendar` bằng `CALENDAR` + `ADDCOLUMNS`
- [ ] Đã có `ChartOfAccounts` với AccountKey
- [ ] Đã import `GLTransactions` (Debit/Credit tách cột)
- [ ] Đã kéo 2 relationship trong Model view
- [ ] Đã tạo cột `TotalAmount = Debit - Credit`

---

# CHƯƠNG 3: TẠO TRIAL BALANCE & BÁO CÁO THU NHẬP

## 🎯 Mục tiêu
Biến **raw GL transactions** thành:
1. **Trial Balance** (Bảng cân đối số phát sinh) — tổng Debit = tổng Credit
2. **Income Statement** (Báo cáo thu nhập) — Doanh thu - Chi phí = Lợi nhuận ròng

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 3.1 — Tạo bảng `ListOfMeasures` (Best practice)
Tại sao cần? Để gom **tất cả measure ở 1 bảng**, dễ quản lý.

**Modeling → New Table:**
```dax
ListOfMeasures = DATATABLE(
    "MeasureName", STRING,
    "Description", STRING,
    {
        {"Total Amount", "Tổng phát sinh (Debit - Credit)"},
        {"Total Debit", "Tổng bên Nợ"},
        {"Total Credit", "Tổng bên Có"},
        {"Sales Revenue", "Doanh thu bán hàng"},
        {"COGS", "Giá vốn hàng bán"},
        {"Net Income", "Lợi nhuận ròng"}
    }
)
```

### Bước 3.2 — Các measure cơ bản

```dax
Total Amount = SUM(GLTransactions[TotalAmount])

Total Debit = SUM(GLTransactions[Debit])

Total Credit = SUM(GLTransactions[Credit])
```

### Bước 3.3 — Measure **Sales Revenue** (Doanh thu)
```dax
Sales Revenue = 
CALCULATE(
    [Total Amount],
    ChartOfAccounts[AccountCategory] = "Doanh thu"
)
```
> 📖 *Giải thích từng dòng:*
> - `CALCULATE(...)` = thay đổi ngữ cảnh filter
> - `[Total Amount]` = measure ở trên
> - `ChartOfAccounts[AccountCategory] = "Doanh thu"` = thêm filter: chỉ lấy account thuộc loại "Doanh thu"

### Bước 3.4 — Measure **COGS** (Giá vốn)
```dax
COGS = 
CALCULATE(
    [Total Amount],
    ChartOfAccounts[AccountCategory] = "Giá vốn"
)
```

### Bước 3.3 — Measure **Net Income** (Lợi nhuận ròng)
```dax
Net Income = 
[Sales Revenue] - [COGS] - 
CALCULATE([Total Amount], ChartOfAccounts[AccountCategory] = "Chi phí")
```

### Bước 3.5 — Dựng Trial Balance trong **Matrix visual**
1. Kéo `AccountCode` vào **Rows**
2. Kéo `AccountName` vào **Rows** (dưới `AccountCode`)
3. Kéo 3 measure vào **Values**: `Total Debit`, `Total Credit`, `Total Amount`
4. Bật **Subtotals** = On

Kết quả cho ABC Coffee (tháng 1/2025):

| AccountCode | AccountName | Total Debit | Total Credit | Total Amount |
|---|---|---:|---:|---:|
| 1000 | Tiền mặt | 5,000,000 | 8,000,000 | -3,000,000 |
| 1100 | Tiền gửi NH | 8,000,000 | 0 | 8,000,000 |
| 4000 | Doanh thu bán hàng | 0 | 8,000,000 | -8,000,000 |
| 5000 | Giá vốn hàng bán | 4,000,000 | 0 | 4,000,000 |
| 6000 | Chi phí lương | 12,000,000 | 0 | 12,000,000 |
| **Total** | | **29,000,000** | **16,000,000** | **13,000,000** |

### Bước 3.6 — Dựng **Income Statement** (Báo cáo thu nhập)
1. **Visualizations → Matrix**
2. Rows: `AccountCategory` (theo thứ tự Doanh thu → Giá vốn → Chi phí)
3. Values: `Sales Revenue`, `COGS`, `Net Income`

```
ABC Coffee — Income Statement
Tháng 1/2025
─────────────────────────────────────
  Doanh thu bán hàng           8,000,000
  Giá vốn hàng bán            (4,000,000)
  ─────────────────────────────────
  Lợi nhuận gộp                4,000,000
  Chi phí hoạt động           
    Chi phí lương            (12,000,000)
  ─────────────────────────────────
  LỢI NHUẬN RÒNG             (8,000,000)
```

> ⚠️ *Lưu ý:* Cty lỗ tháng đầu là bình thường vì trả lương cả tháng trước + tháng này.

### Bước 3.7 — Thêm **Shape map / đường kẻ** cho đẹp
- **Format → Row dividers → On**
- **Format → Subtotal divider = Top + Bottom, Bold**

---

## 💡 PHẦN 2 — HỌC SAU

### 3.1 SUM vs SUMX — khác nhau khi nào?

| Hàm | Dùng khi | Ví dụ |
|---|---|---|
| `SUM(cột)` | Tổng đơn giản 1 cột số | `SUM(GLTransactions[Debit])` |
| `SUMX(bảng, biểu thức)` | Tính **trên từng dòng** rồi mới cộng | `SUMX(Products, [Qty] * [Price])` |

**Ví dụ ABC Coffee:** Bảng `Sales` có cột `Qty` và `UnitPrice`. Tính doanh thu:
```dax
Total Sales = SUMX(Sales, Sales[Qty] * Sales[UnitPrice])
```

### 3.2 Calculated Column vs Measure

| | Calculated Column | Measure |
|---|---|---|
| Tính khi nào | Khi refresh (1 lần) | Mỗi lần user tương tác |
| Lưu ở đâu | Trong model (tốn RAM) | Không lưu (tính lại) |
| Dùng cho | Filter, slicer, axis | KPI, số trên card |
| Ví dụ | `[Profit] = [Revenue] - [Cost]` | `[Total Revenue] = SUM(Sales[Revenue])` |

➡ **Quy tắc:** Mặc định dùng **Measure**. Chỉ tạo Calculated Column khi cần filter theo nó.

### 3.3 Lỗi thường gặp
- ❌ Dùng `SUMX` cho 1 cột đơn → chậm hơn `SUM`
- ❌ Tạo 100 calculated columns → model phình to
- ❌ Quên `CALCULATE` → filter bị bỏ qua, kết quả sai
- ❌ Matrix không bật Subtotals → người xem không thấy tổng
- ✅ `Total Amount = Debit - Credit` thay vì 2 measure riêng → nhanh hơn
- ✅ Gom measures vào bảng `ListOfMeasures` → tìm dễ

---

## ✅ Checklist Chương 3
- [ ] Tạo bảng `ListOfMeasures`
- [ ] Có 6 measure: Total Amount, Total Debit, Total Credit, Sales Revenue, COGS, Net Income
- [ ] Matrix Trial Balance hiển thị đúng tổng phát sinh
- [ ] Income Statement hiển thị đúng, có dòng Lợi nhuận ròng

---

# CHƯƠNG 4: CÁC MEASURE DAX THƯỜNG GẶP

## 🎯 Mục tiêu
Thành thạo 4 nhóm measure "mang tiền về cho sếp":
1. **Margin & %** (Tỷ suất)
2. **Time intelligence** (So sánh theo thời gian)
3. **FX conversion** (Quy đổi ngoại tệ)
4. **Variance** (Chênh lệch — sẽ dùng nhiều ở Ch 7)

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 4.1 — Measure **Gross Margin %** (Biên lợi nhuận gộp)
```dax
Gross Margin % = 
DIVIDE(
    [Sales Revenue] - [COGS],
    [Sales Revenue]
)
```
> 📖 *Giải thích:*
> - `DIVIDE(a, b)` = `a/b` nhưng an toàn — nếu `b=0` trả về BLANK() thay vì lỗi `#DIV/0!`
> - Cách cũ: `IFERROR([Sales] / [Sales] - [COGS], 0)` → dài hơn, chậm hơn

Với ABC Coffee tháng 1:
- Sales = 8,000,000; COGS = 4,000,000
- Gross Margin = 4,000,000 / 8,000,000 = **50%**

### Bước 4.2 — Time Intelligence cơ bản
Trước tiên **bắt buộc phải có**:
- Bảng `Calendar` (đã làm Ch 2) ✓
- Relationship `Calendar[Date]` → `GLTransactions[Date]` ✓
- Mark `Calendar` là **Date table** (Modeling → Mark as date table)

**MTD (Month-to-Date — Từ đầu tháng đến hiện tại):**
```dax
Sales MTD = 
TOTALMTD([Sales Revenue], Calendar[Date])
```

**QTD, YTD** — tương tự:
```dax
Sales QTD = TOTALQTD([Sales Revenue], Calendar[Date])
Sales YTD = TOTALYTD([Sales Revenue], Calendar[Date], "12/31")
```

> 💡 Tham số cuối `"12/31"` là ngày cuối năm tài chính. VN = 31/12, Mỹ có thể là 31/10.

**Prior Year (Cùng kỳ năm trước):**
```dax
Sales PY = 
CALCULATE(
    [Sales Revenue],
    SAMEPERIODLASTYEAR(Calendar[Date])
)
```

### Bước 4.3 — Measure **YoY Growth %**
```dax
Sales YoY % = 
DIVIDE(
    [Sales Revenue] - [Sales PY],
    [Sales PY]
)
```

### Bước 4.4 — FX Conversion với 2 loại tỷ giá

**Bảng FxRates:**
| Date | Currency | SpotRate | AvgRate |
|---|---|---|---|
| 2025-01-31 | USD | 25,000 | 24,500 |
| 2025-02-28 | USD | 25,200 | 24,800 |

**a) Spot rate tại ngày giao dịch (dùng cho balance sheet):**
```dax
Amount in VND (Spot) = 
SUMX(
    GLTransactions,
    GLTransactions[TotalAmount] * 
    LOOKUPVALUE(
        FxRates[SpotRate],
        FxRates[Date], GLTransactions[Date],
        FxRates[Currency], GLTransactions[Currency]
    )
)
```
> 📖 *LOOKUPVALUE = tra cứu giá trị trong bảng khác theo nhiều điều kiện (giống VLOOKUP nhiều key).*

**b) Period average rate (dùng cho P&L):**
```dax
Amount in VND (Avg) = 
SUMX(
    GLTransactions,
    GLTransactions[TotalAmount] * 
    AVERAGEX(
        FILTER(
            FxRates,
            FxRates[Currency] = GLTransactions[Currency] &&
            FxRates[Date] <= GLTransactions[Date] &&
            FxRates[Date] > EDATE(GLTransactions[Date], -1)
        ),
        FxRates[AvgRate]
    )
)
```
> 📖 *AVERAGEX lấy trung bình tỷ giá trong tháng giao dịch.*

### Bước 4.5 — Dynamic rate (User chọn loại tỷ giá)
```dax
Selected FX Measure = 
VAR SelectedRateType = SELECTEDVALUE(FxSelector[RateType], "Spot")
RETURN
SWITCH(
    SelectedRateType,
    "Spot", [Amount in VND (Spot)],
    "Average", [Amount in VND (Avg)]
)
```

---

## 💡 PHẦN 2 — HỌC SAU

### 4.1 CALCULATE — "dao mổ" của DAX
```dax
CALCULATE( <biểu thức>, <filter1>, <filter2>, ... )
```
- **Bỏ filter**: `CALCULATE([Sales], ALL(Products))` — bỏ filter từ slicer
- **Thay filter**: `CALCULATE([Sales], Stores[Store] = "Store 1")`
- **Thêm filter**: `CALCULATE([Sales], Calendar[Year] = 2025)`

### 4.2 4 hàm time intelligence hay dùng
| Hàm | Ý nghĩa | Ví dụ |
|---|---|---|
| `TOTALMTD` | Tổng từ đầu tháng | Doanh thu từ 1/1 đến nay |
| `SAMEPERIODLASTYEAR` | Cùng kỳ năm trước | Tháng 1/2024 |
| `PARALLELPERIOD` | Lùi về N kỳ | Tháng trước, năm trước |
| `DATEADD` | Cộng/trừ ngày | 30 ngày gần nhất |

**PARALLELPERIOD vs DATEADD:**
```dax
Sales Last Month = CALCULATE([Sales], PARALLELPERIOD(Calendar[Date], -1, MONTH))
Sales 30 Days = CALCULATE([Sales], DATEADD(Calendar[Date], -30, DAY))
```
- `PARALLELPERIOD` = "nguyên kỳ" (cả tháng, cả năm)
- `DATEADD` = "dịch chuyển" theo ngày

### 4.3 Best practice: dùng `VAR` để code dễ đọc
```dax
Gross Margin % (Clean) = 
VAR Revenue = [Sales Revenue]
VAR Cost = [COGS]
VAR Profit = Revenue - Cost
RETURN DIVIDE(Profit, Revenue)
```
➡ Dễ debug hơn — click vào VAR thấy giá trị ngay trong Power BI.

### 4.4 Lỗi thường gặp
- ❌ Quên mark `Calendar` là **Date table** → time intelligence báo lỗi đỏ
- ❌ `CALCULATE([Sales], Calendar[Year] = 2025)` nhưng `Year` là text → không hoạt động
- ❌ `PARALLELPERIOD` cho ngày (Day) → phải dùng `DATEADD`
- ❌ Chia tỷ giá spot cho P&L → kết quả bị volatility cao
- ✅ Dùng `DIVIDE` thay vì `/`
- ✅ Dùng `VAR` cho mọi measure phức tạp

---

## ✅ Checklist Chương 4
- [ ] Có measure `Gross Margin %`
- [ ] Có MTD/QTD/YTD/PY
- [ ] Có measure FX spot và FX average
- [ ] Có measure động cho phép user chọn loại tỷ giá

---

# CHƯƠNG 5: XỬ LÝ BÚT TOÁN THỦ CÔNG (JOURNAL ENTRIES)

## 🎯 Mục tiêu
- Phân biệt bút toán **nên** và **không nên** đưa vào model
- Thiết kế schema cho phép theo dõi **ai tạo, duyệt, hạch toán** bút toán
- Tính được **thời gian trung bình** từ tạo → hạch toán

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 5.1 — Quyết định bút toán nào đưa vào
**Bảng kiểm:**

| Loại bút toán | Đưa vào Power BI? | Lý do |
|---|---|---|
| Điều chỉnh cuối tháng (accrual) | ✅ Có | Quan trọng cho báo cáo |
| Phân bổ trước (prepaid amortization) | ✅ Có | Ảnh hưởng nhiều kỳ |
| Sai sót phát hiện sau | ✅ Có | Audit trail |
| Điều chỉnh tỷ giá tự động | ❌ Không | Đã có FxRate rồi |
| Bút toán đóng/mở năm tự động | ❌ Không | Hệ thống đã xử lý |

### Bước 5.2 — Schema Journal cho ABC Coffee

**Bảng JournalHeader (Header bút toán):**
| JournalID | JournalDate | Description | CreatedByUserID | ApprovedByUserID | PostedByUserID | PostedDate | Status |
|---|---|---|---|---|---|---|---|
| 1 | 2025-01-15 | Chi phí thuê MB tháng 1 | 1 | 2 | 3 | 2025-01-20 | Posted |
| 2 | 2025-01-28 | Phân bổ CP bảo hiểm | 1 | 2 | 3 | 2025-02-01 | Posted |
| 3 | 2025-01-30 | Điều chỉnh kiểm kê | 1 | NULL | NULL | NULL | Pending |

**Bảng JournalLines (Dòng bút toán):**
| LineID | JournalID | AccountKey | Debit | Credit |
|---|---|---|---|---|
| 1 | 1 | 6 | 30,000,000 | 0 |
| 2 | 1 | 1 | 0 | 30,000,000 |
| 3 | 2 | 5 | 5,000,000 | 0 |
| 4 | 2 | 1 | 0 | 5,000,000 |
| 5 | 3 | 4 | 2,000,000 | 0 |
| 6 | 3 | 1 | 0 | 2,000,000 |

**Bảng Users:**
| UserID | UserName | Role | Email |
|---|---|---|---|
| 1 | Nguyen Van A | Accountant | a@abccoffee.vn |
| 2 | Tran Thi B | Manager | b@abccoffee.vn |
| 3 | Le Van C | CFO | c@abccoffee.vn |

### Bước 5.3 — Tạo **Role-Playing Dimensions** (3 bản sao Users)
Vì 1 bút toán có **3 người** (Created / Approved / Posted), cần 3 relationship từ `JournalHeader` → `Users`.

Trong Power BI:
1. Mở Model view
2. Tạo **3 bản sao** `Users` bằng cách vào Modeling → New Table:
```dax
Users_Created = Users
Users_Approved = Users
Users_Posted = Users
```
3. Kéo relationship:
   - `JournalHeader[CreatedByUserID]` → `Users_Created[UserID]` (Active)
   - `JournalHeader[ApprovedByUserID]` → `Users_Approved[UserID]` (Inactive)
   - `JournalHeader[PostedByUserID]` → `Users_Posted[UserID]` (Inactive)

### Bước 5.4 — Measure **Avg Days to Post** (Thời gian hạch toán trung bình)
```dax
Avg Days to Post = 
AVERAGEX(
    JournalHeader,
    DATEDIFF(
        JournalHeader[JournalDate],
        JournalHeader[PostedDate],
        DAY
    )
)
```
> Với ABC Coffee:
> - J1: 20/1 - 15/1 = 5 ngày
> - J2: 1/2 - 28/1 = 4 ngày
> - Trung bình = **4.5 ngày**

### Bước 5.5 — Measure **# of Journal Lines per Journal** (Số dòng/but toan)
```dax
Avg Lines per Journal = 
DIVIDE(
    COUNTROWS(JournalLines),
    DISTINCTCOUNT(JournalHeader[JournalID])
)
```
> ABC Coffee: 6 dòng / 3 but toan = **2 dòng/but toan**

### Bước 5.6 — Measure **# of Pending Journals** (Bút toán chờ duyệt)
```dax
Pending Journals = 
CALCULATE(
    DISTINCTCOUNT(JournalHeader[JournalID]),
    JournalHeader[Status] = "Pending"
)
```

---

## 💡 PHẦN 2 — HỌC SAU

### 5.1 Khi nào nên dùng bút toán thủ công?
- ✅ Khi **hệ thống ERP chưa hỗ trợ** (vd: phân bổ chi phí trả trước)
- ✅ Khi cần **ghi nhận nhanh** cuối tháng (accruals)
- ❌ **Không** lạm dụng để "làm đẹp" số liệu

### 5.2 Tại sao cần 3 bản sao Users?
DAX/Power BI chỉ cho phép **1 active relationship** giữa 2 bảng. Nếu có 3 cột User (Created/Approved/Posted) trong JournalHeader, phải tạo 3 bản sao → mỗi bản chỉ có 1 relationship Active.

Khi measure cần dùng bản nào, dùng `USERELATIONSHIP`:
```dax
Approved by Manager = 
CALCULATE(
    DISTINCTCOUNT(JournalHeader[JournalID]),
    USERELATIONSHIP(JournalHeader[ApprovedByUserID], Users_Approved[UserID])
)
```

### 5.3 Lỗi thường gặp
- ❌ Tạo 1 bảng `Users` rồi cố kéo 3 relationship → DAX báo lỗi ambiguous
- ❌ Quên `USERELATIONSHIP` → kết quả = Created By (mặc định Active)
- ❌ Không có cột `Status` → không lọc được bút toán Pending
- ✅ Dùng `AVERAGEX` thay vì `AVERAGE` trên cột tính sẵn → nhanh hơn
- ✅ Tạo **calculated column** `[DaysToPost] = DATEDIFF(...)` nếu cần filter cột này trong slicer

---

## ✅ Checklist Chương 5
- [ ] Xác định được bút toán nào đưa vào model
- [ ] Có 3 bảng: JournalHeader, JournalLines, Users
- [ ] Tạo 3 bản sao Users cho role-playing
- [ ] Có measure Avg Days to Post, Pending Journals

---

# CHƯƠNG 6: TINH GỌN VỚI POWER QUERY

## 🎯 Mục tiêu
- Hiểu **Power Query = công cụ ETL** (Extract-Transform-Load) của Power BI
- Biết 7 thao tác hay dùng nhất: Choose Columns, Remove Rows, Group By, Change Type, Custom Column, Merge, Pivot/Unpivot
- Sửa được **số dư đầu kỳ** sai từ hệ thống nguồn

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 6.1 — Mở Power Query Editor
Trong Power BI Desktop → tab **Home** → **Transform Data** → mở Power Query Editor.

### Bước 6.2 — Choose Columns (Chỉ giữ cột cần)
Ví dụ file `GL_Raw.xlsx` có 30 cột, bạn chỉ cần 7.

1. Chọn bảng GL_Raw
2. **Home → Choose Columns → Choose Columns**
3. Tick: `TransID`, `Date`, `AccountCode`, `Debit`, `Credit`, `Description`, `Store`
4. OK

> 💡 *Mẹo:* Sau khi chọn xong, **mỗi bước sẽ xuất hiện ở panel bên phải (Applied Steps)**. Click vào đó để undo/redo.

### Bước 6.3 — Change Data Type
Cột `Date` đang bị Power Query hiểu là `Any`:
- Click chuột phải cột `Date` → **Change Type → Date**
- Cột `Debit`, `Credit` → **Change Type → Decimal Number**
- Cột `AccountCode` → **Change Type → Text** (rất quan trọng — để mã `1000` không bị cắt số 0)

### Bước 6.4 — Remove Rows (Xóa dòng)
Ví dụ file Excel có 3 dòng header/footer không phải dữ liệu:
- **Home → Remove Rows → Remove Top Rows → 3**

Hoặc xóa dòng theo điều kiện (vd: xóa dòng có `Status = "Test"`):
- **Home → Remove Rows → Remove Blank Rows** (nếu có dòng trống)
- Hoặc dùng **Filter** trên cột → bỏ tick giá trị muốn xóa

### Bước 6.5 — Group By (Gộp dữ liệu)
Tính tổng doanh thu theo tháng cho ABC Coffee:

1. **Transform → Group By**
2. Group by: `YearMonth` (cột bạn tạo từ Calendar)
3. New column name: `Total Sales`
4. Operation: `Sum`
5. Column: `Debit - Credit` (cần tạo trước bằng Custom Column)

Kết quả:

| YearMonth | Total Sales |
|---|---:|
| 2025-01 | 13,000,000 |
| 2025-02 | 15,500,000 |
| 2025-03 | 18,200,000 |

### Bước 6.6 — Conditional Column (Cột điều kiện)
Thêm cột `TransactionType` dựa vào Debit/Credit:

1. **Add Column → Conditional Column**
2. New column name: `TransactionType`
3. If: `[Debit] > 0` → Then: `"Debit"` → Else: `"Credit"`

Hoặc Custom Column với M code:
```m
= Table.AddColumn(Source, "SignedAmount", each [Debit] - [Credit], type number)
```

### Bước 6.7 — Merge Queries (JOIN)
Kết nối `GLTransactions` với `ChartOfAccounts`:

1. Chọn query `GLTransactions`
2. **Home → Merge Queries → Merge Queries**
3. Chọn bảng: `ChartOfAccounts`
4. Click cột `AccountCode` ở GLTransactions, giữ Shift, click `AccountCode` ở ChartOfAccounts
5. Join kind: **Left Outer** (giữ tất cả dòng GL, lấy thông tin từ CoA)
6. OK → cột mới `ChartOfAccounts` xuất hiện (chứa nested table)
7. Click nút **↔ mở rộng** → tick `AccountName`, `AccountCategory`, `AccountType`

### Bước 6.8 — Pivot / Unpivot
**Unpivot** (chuyển từ wide → long):
File Excel ABC Coffee có cấu trúc:
| Account | Jan-2025 | Feb-2025 | Mar-2025 |
|---|---:|---:|---:|
| Doanh thu | 8M | 10M | 12M |
| COGS | 4M | 5M | 6M |

Muốn chuyển thành:
| Account | Month | Amount |
|---|---:|---:|
| Doanh thu | Jan-2025 | 8M |
| Doanh thu | Feb-2025 | 10M |
| ... | ... | ... |

1. Chọn cột `Account`
2. **Transform → Unpivot Other Columns**
3. Cột tháng + cột giá trị tự động xuất hiện

**Pivot** (làm ngược lại): **Transform → Pivot Column** → chọn cột Month, Values = Sum of Amount

### Bước 6.9 — Sửa **Opening Balance** sai
Hệ thống nguồn đôi khi có số dư đầu kỳ sai. Cách sửa:

1. Lọc ra các dòng có `AccountCode = "1000"` (Tiền mặt)
2. **Filter Rows** chỉ giữ dòng `Date = "2025-01-01"`
3. **Group By** theo `AccountCode` → Sum `Debit - Credit` → gọi `CalculatedOpening`
4. Join với bảng OpeningBalance chuẩn
5. Tính `Corrected = CalculatedOpening + Adjustment`
6. Append vào bảng GL

Code M minh họa:
```m
= Table.SelectRows(GLTransactions, each [AccountCode] = "1000" and [Date] = #date(2025,1,1))
```

---

## 💡 PHẦN 2 — HỌC SAU

### 6.1 ETL — Extract / Transform / Load là gì?
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  EXTRACT     │ ──→ │  TRANSFORM   │ ──→ │     LOAD     │
│  (Lấy data)  │     │  (Làm sạch)  │     │ (Vào model)  │
└──────────────┘     └──────────────┘     └──────────────┘
   Từ Excel,           Power Query          Power BI
   SQL, CSV,           Editor                Dataset
   API, ...
```

### 6.2 Power Query vs DAX — khi nào dùng cái nào?
| | Power Query (M) | DAX |
|---|---|---|
| Chạy khi nào | Khi **refresh** (1 lần) | Mỗi lần user xem |
| Dùng cho | Làm sạch, đổi kiểu, merge | Tính toán, measure |
| Tốn tài nguyên | RAM (lưu kết quả) | CPU (tính lại) |
| Ví dụ | "Đổi text 'NV' thành 'Nhân viên'" | "Tính tổng doanh thu theo tháng" |

➡ **Quy tắc:** Mọi thứ có thể làm trong Power Query → **làm trong Power Query** (chạy 1 lần, dùng nhiều lần).

### 6.3 Merge vs Append
- **Merge** = nối **cột** (giống JOIN trong SQL): có `ChartOfAccounts` mở rộng
- **Append** = nối **dòng** (giống UNION): gộp GL 2024 + GL 2025

### 6.4 AI trong Power Query
Từ bản Power BI mới, có **AI Insights**:
- **Transform → Extract → From AI Insights** (cần license)
- **Column quality**, **Column distribution**, **Column profile** (built-in): Power Query tự phát hiện % null, % error, min/max
- **Copilot** (preview): tự generate M code từ prompt tiếng Anh

### 6.5 Lỗi thường gặp
- ❌ Đổi `AccountCode` từ Text → Number → mất số 0 đầu (`0100` → `100`)
- ❌ Quên `Change Type` cho cột Date → DAX `YEAR()` trả về lỗi
- ❌ Merge mà chọn sai Join Kind → mất dòng (Inner) hoặc duplicate (Cartesian)
- ❌ Sửa data trong Power Query thay vì **sửa ở source** → tạo "nghĩa vụ" maintain
- ✅ Luôn **profile data** trước khi transform (View → Column Profile)
- ✅ **Fix at source** — sửa ở hệ thống gốc, không sửa trong Power Query

---

## ✅ Checklist Chương 6
- [ ] Biết mở Power Query Editor
- [ ] Thực hiện được: Choose Columns, Change Type, Remove Rows
- [ ] Group By theo tháng ra được tổng doanh thu
- [ ] Merge GLTransactions với ChartOfAccounts
- [ ] Unpivot file wide → long

---

# CHƯƠNG 8: QUẢN LÝ DỮ LIỆU TỒN KHO

## 🎯 Mục tiêu
- Phân biệt **snapshot** vs **transactional** inventory
- Tính **Stock Quantity** và **Stock Value** chính xác tại 1 thời điểm
- Tính **Stock Turn** (Vòng quay tồn kho)

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 8.1 — Chọn mô hình dữ liệu tồn kho
| Mô hình | Khi nào dùng | Ưu | Nhược |
|---|---|---|---|
| **Snapshot** | Hệ thống có ảnh chụp cuối ngày | Dễ tính | Cần lưu trữ lịch sử |
| **Transactional** | Có dòng nhập/xuất chi tiết | Tái dựng được mọi thời điểm | Cần SUMX nặng |

ABC Coffee dùng **Transactional** — POS ghi mỗi lần bán + nhập hàng.

### Bước 8.2 — Schema InventoryTransactions

| TransID | Date | ProductKey | StatusType | Quantity | UnitCost |
|---|---|---|---|---:|---:|
| 1 | 2025-01-05 | P001 | Purchased | 100 | 25,000 |
| 2 | 2025-01-10 | P001 | Sold | 10 | NULL |
| 3 | 2025-01-15 | P001 | Sold | 15 | NULL |
| 4 | 2025-01-20 | P001 | Purchased | 50 | 26,000 |

**StatusType có 4 giá trị:**
- `Purchased` (Nhập mua)
- `Sold` (Bán ra)
- `Ordered` (Đã đặt hàng — chưa nhận)
- `Transferred` (Chuyển giữa các cửa hàng)

### Bước 8.3 — Measure **Stock Quantity** (Tồn kho cuối kỳ)
```dax
Stock Quantity = 
CALCULATE(
    SUM(InventoryTransactions[Quantity]),
    InventoryTransactions[StatusType] IN {"Purchased", "Transferred In"},
    InventoryTransactions[Date] <= MAX(Calendar[Date])
) - 
CALCULATE(
    SUM(InventoryTransactions[Quantity]),
    InventoryTransactions[StatusType] IN {"Sold", "Transferred Out"},
    InventoryTransactions[Date] <= MAX(Calendar[Date])
)
```

> 📖 *Giải thích:*
> - Tồn kho = Tổng Nhập - Tổng Xuất
> - `MAX(Calendar[Date])` = ngày cuối cùng trong context (tháng, quý, năm)
> - `IN {"A", "B"}` = hoặc A, hoặc B

Tính cho P001 đến 31/01/2025:
- Nhập: 100 + 50 = 150
- Xuất: 10 + 15 = 25
- **Tồn = 150 - 25 = 125**

### Bước 8.4 — Measure **Stock Value** (Giá trị tồn kho)
Dùng **FIFO** (nhập trước xuất trước) — phương pháp phổ biến nhất:

```dax
Stock Value (FIFO) = 
SUMX(
    VALUES(InventoryTransactions[ProductKey]),
    VAR Remaining = [Stock Quantity]
    VAR Purchases = 
        FILTER(
            InventoryTransactions,
            InventoryTransactions[ProductKey] = EARLIER(InventoryTransactions[ProductKey]) &&
            InventoryTransactions[StatusType] = "Purchased" &&
            InventoryTransactions[Date] <= MAX(Calendar[Date])
        )
    RETURN
    SUMX(
        Purchases,
        ...
    )
)
```

➡ **Cảnh báo:** FIFO chính xác cần **rất nhiều bộ nhớ**. ABC Coffee nên dùng **Weighted Average** cho đơn giản:

```dax
Stock Value (Avg Cost) = 
AVERAGEX(
    FILTER(
        InventoryTransactions,
        InventoryTransactions[ProductKey] = SELECTEDVALUE(Products[ProductKey]) &&
        InventoryTransactions[StatusType] = "Purchased" &&
        InventoryTransactions[Date] <= MAX(Calendar[Date])
    ),
    InventoryTransactions[UnitCost] * InventoryTransactions[Quantity]
) * [Stock Quantity]
```

### Bước 8.5 — Measure **Stock Turn** (Vòng quay tồn kho)
```dax
Stock Turn = 
DIVIDE(
    [COGS],
    [Average Inventory Value]
)
```
> 📖 *Stock Turn = COGS / Tồn kho trung bình*
> - Stock Turn cao → bán nhanh (tốt, trừ khi hết hàng)
> - Stock Turn thấp → tồn kho ứ đọng (xấu)

Ví dụ ABC Coffee năm 2024:
- COGS = 600,000,000
- Avg Inventory = 50,000,000
- **Stock Turn = 12 lần/năm** → ~30 ngày/lần (tốt cho cà phê tươi)

### Bước 8.6 — Measure **Days Inventory Outstanding** (Số ngày tồn kho)
```dax
DIO = DIVIDE(365, [Stock Turn])
```
> ABC Coffee: 365 / 12 = **30 ngày**

### Bước 8.7 — Measure **# of Days Since Last Purchase**
```dax
Days Since Last Purchase = 
DATEDIFF(
    CALCULATE(
        MAX(InventoryTransactions[Date]),
        InventoryTransactions[StatusType] = "Purchased"
    ),
    TODAY(),
    DAY
)
```

### Bước 8.8 — Measure **# of Stock-Out Incidents** (Số lần hết hàng)
```dax
Stock Out Days = 
CALCULATE(
    COUNTROWS(Calendar),
    FILTER(
        Calendar,
        [Stock Quantity] = 0
    )
)
```

---

## 💡 PHẦN 2 — HỌC SAU

### 8.1 Snapshot vs Transactional — sâu hơn
**Snapshot** (ảnh chụp):
```
Date        Product  Qty  Value
2025-01-01  P001     0    0
2025-01-02  P001     100  2,500,000
2025-01-03  P001     85   2,125,000
```
- Cột `Qty` đã là **tồn cuối ngày** → đọc measure rất nhanh
- Nhưng tốn dung lượng (1 dòng/ngày/sản phẩm)

**Transactional** (dòng chảy):
```
Date        Product  Status    Qty
2025-01-02  P001     Purchased 100
2025-01-10  P001     Sold      10
```
- Cần tính `SUM` + `FILTER` theo ngày → DAX nặng hơn
- Nhưng chỉ tốn dung lượng cho **giao dịch**

### 8.2 Tại sao `MAX(Calendar[Date])` quan trọng?
Trong Matrix visual:
- User chọn tháng 3/2025 → `MAX(Calendar[Date])` = 31/3/2025
- DAX chỉ tính giao dịch **trước 31/3** → tồn kho cuối tháng 3
- User chọn cả năm → `MAX(Calendar[Date])` = 31/12/2025
- DAX tính tổng tất cả → tồn cuối năm

### 8.3 Lỗi thường gặp
- ❌ Dùng `Quantity` cho cả nhập lẫn xuất (không có cột StatusType) → không phân biệt được
- ❌ Tính Stock Quantity không có `MAX(Calendar[Date])` → tổng tất cả thời gian
- ❌ Dùng `LOOKUPVALUE` cho giá xuất (FIFO) → chậm với dataset lớn
- ✅ Lưu **`StatusType`** rõ ràng (không dùng số dương/âm)
- ✅ Tạo **bảng DateRange** riêng cho stockout analysis

---

## ✅ Checklist Chương 8
- [ ] Phân biệt được snapshot vs transactional
- [ ] Schema InventoryTransactions có cột StatusType
- [ ] Measure Stock Quantity chính xác tại 1 ngày
- [ ] Measure Stock Turn, DIO

---

# CHƯƠNG 9: TẠO BỘ BÁO CÁO TÀI CHÍNH

## 🎯 Mục tiêu
- Áp dụng **5 nguyên tắc thiết kế** báo cáo tài chính
- Chọn **đúng loại visual** cho từng câu hỏi kinh doanh
- Xây dựng **P&L Report** hoàn chỉnh cho ABC Coffee

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 9.1 — Nguyên tắc 1: **Accuracy** (Chính xác)
✅ DAX đã verified với Trial Balance (Ch 3)
✅ Mọi số trên báo cáo = số trên sổ cái

### Bước 9.2 — Nguyên tắc 2: **Security** (Bảo mật)
- RLS cho phép kế toán từng cửa hàng chỉ thấy cửa hàng mình (xem Ch 13)
- Không show số liệu thô, chỉ show báo cáo tổng hợp cho viewer

### Bước 9.3 — Nguyên tắc 3: **Responsiveness** (Phản hồi nhanh)
- Báo cáo tải trong **< 3 giây** (test trên mobile)
- Tránh dùng quá nhiều visual trên 1 page (< 8)

### Bước 9.4 — Nguyên tắc 4: **Clear Purpose** (Mục đích rõ ràng)
Mỗi page trả lời **1 câu hỏi duy nhất**:
- Page 1: "Doanh thu tháng này bao nhiêu?" → KPI card + trend line
- Page 2: "Lợi nhuận gộp theo cửa hàng?" → Bar chart
- Page 3: "Chi phí nào tăng mạnh?" → Waterfall chart

### Bước 9.5 — Nguyên tắc 5: **Consistent Layout** (Bố cục nhất quán)
- Tất cả page có **cùng header** (logo ABC Coffee, period filter, user name)
- Cùng **font**, **color palette** (xanh dương = tốt, đỏ = cảnh báo)
- Cùng **slicer position** (góc trên bên phải)

### Bước 9.6 — Chọn **đúng visual** cho từng câu hỏi

| Câu hỏi | Visual nên dùng | Visual **tránh** |
|---|---|---|
| "Doanh thu tháng này bao nhiêu?" | **KPI Card** | Pie chart (chỉ hiển thị 1 giá trị) |
| "Doanh thu thay đổi theo TG?" | **Line chart** | Bar chart (khó thấy trend) |
| "So sánh 3 cửa hàng?" | **Clustered bar** | Pie chart (quá nhiều lát) |
| "Doanh thu đóng góp %?" | **100% stacked bar** | Pie chart (nhiều hơn 5 lát) |
| "Chi phí nào lớn nhất?" | **Sorted bar chart** | Treemap (khó so sánh chính xác) |
| "Chi phí tăng/giảm?" | **Waterfall chart** | Bar chart (khó thấy dòng chảy) |

### Bước 9.7 — Xây **P&L Report** cho ABC Coffee

1. **Tạo Page mới** → đặt tên "P&L Statement"
2. **Add Matrix visual**:
   - Rows: `ChartOfAccounts[AccountCategory]` (sort theo custom order)
   - Values: `Sales Revenue`, `COGS`, `Total Expenses`, `Net Income`
3. **Sort AccountCategory** theo thứ tự logic:
   - Sort by cột `SortOrder` trong bảng ChartOfAccounts (1=Doanh thu, 2=Giá vốn, 3=Chi phí, 4=Lợi nhuận)
4. **Add Subtotal**:
   - Lợi nhuận gộp (Gross Profit) = Sales - COGS
   - Lợi nhuận hoạt động (Operating Profit) = Gross Profit - Operating Expenses
   - Lợi nhuận ròng (Net Income) = Operating Profit - Other
5. **Add Card visuals** cho KPI:
   - Top-left: Net Income tháng này
   - Top-center: Net Income YoY %
   - Top-right: Gross Margin %
6. **Add Line chart** phía dưới: Net Income theo 12 tháng

```
┌────────────────────────────────────────────┐
│  ABC Coffee — P&L Statement               │
│  Tháng 3/2025 (YTD)                        │
├──────────────┬──────────────┬──────────────┤
│ Lợi nhuận    │ YoY Growth   │ Gross Margin │
│ 450M VND     │ +12.5% ↑     │ 55%          │
├──────────────┴──────────────┴──────────────┤
│ AccountCategory   │ Amount  │ % of Sales │
│ Doanh thu         │ 2,400M  │ 100.0%     │
│ Giá vốn           │(1,080M) │  45.0%     │
│ ─ Lợi nhuận gộp ─ │ 1,320M  │  55.0%     │
│ Chi phí lương     │  (480M) │  20.0%     │
│ Chi phí thuê MB   │  (240M) │  10.0%     │
│ Chi phí khác      │  (150M) │   6.3%     │
│ ─ LỢI NHUẬN RÒNG │   450M  │  18.7%     │
├────────────────────────────────────────────┤
│ Net Income Trend (12 tháng)                │
│         ___/\___                           │
│        /        \___                       │
│   ____/             \______                │
└────────────────────────────────────────────┘
```

### Bước 9.8 — Thêm **drill-through** (Click drill xuống chi tiết)
1. Click chuột phải vào cell "Chi phí lương 480M" trên P&L
2. **Drill through → Chi tiết chi phí lương**
3. Tạo page "Chi tiết lương" với breakdown theo: cửa hàng, phòng ban, tháng

### Bước 9.9 — Conditional formatting cho Matrix
- Click Matrix → **Format → Cell elements → Background color**
- Rule: `Value < 0` → màu đỏ nhạt, `Value > 0` → màu xanh nhạt
- **Data bars**: thêm thanh bar ngang trong ô số

### Bước 9.10 — Slicer & Bookmark cho UX
- **Slicer** cho: `YearMonth`, `Store`, `AccountCategory`
- **Bookmark** cho: "View 2024 YTD", "View 2025 Q1"
- **Button** để switch giữa các bookmark

---

## 💡 PHẦN 2 — HỌC SAU

### 9.1 Tại sao **tránh** Pie chart?
- Não người **khó so sánh góc** (so với so sánh chiều dài)
- Khi có > 5 lát → không đọc được
- Thay thế bằng:
  - **Sorted bar chart** (dễ so sánh)
  - **Donut chart** (nếu < 5 lát)
  - **Treemap** (nếu có hierarchy)
  - **100% stacked bar** (nếu so sánh qua nhiều kỳ)

### 9.2 Format số tiền cho đẹp
- **Format → Measure → Decimal places = 0** (không hiển thị ,00)
- **Display units = Auto** hoặc "Millions" (cho doanh thu lớn)
- **Custom format string**: `#,##0.0,,"M"` → hiển thị "1.2M" thay vì "1,200,000"

### 9.3 Performance — 7 mẹo tăng tốc báo cáo
1. **Giảm số visual** trên 1 page (< 8)
2. **Dùng slicer ít field** (1-3 fields)
3. **Tắt auto date** (File → Options → Time intelligence → Off)
4. **Reduce rows** trong Matrix (< 10,000)
5. **Tránh measure phức tạp** trong visual axis
6. **Aggregate trong Power Query** trước khi load vào model
7. **Dùng Import mode** thay vì DirectQuery (nếu data không quá lớn)

### 9.4 Lỗi thường gặp
- ❌ 1 page chứa 20 visual → load chậm
- ❌ Pie chart với 15 cửa hàng → không đọc được
- ❌ Không có slicer period → user phải chỉnh từng measure
- ❌ Matrix không có Subtotals → người xem tự cộng
- ❌ Conditional format quá nhiều màu → rối mắt
- ✅ Tối đa **8 visual/page**, **1 chủ đề/page**
- ✅ Dùng **bookmark** cho các view khác nhau
- ✅ Có **"Last Refreshed"** label ở góc → user biết data cũ hay mới

---

## ✅ Checklist Chương 9
- [ ] Có 5 nguyên tắc thiết kế trong đầu
- [ ] Biết chọn visual phù hợp
- [ ] P&L report có Matrix + KPI cards + Trend line
- [ ] Có Slicer cho period + store
- [ ] Có drill-through từ P&L xuống chi tiết

---

# CHƯƠNG 10: KIỂM THỬ & TINH CHỈNH DỮ LIỆU

## 🎯 Mục tiêu
- 4 loại kiểm thử: Trial Balance check, Subtotal reconciliation, Transaction-level reconciliation, Non-functional test
- Phát hiện **DAX typo** và sửa nhanh
- Biết khi nào **fix at source** thay vì fix in Power Query

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 10.1 — Test 1: **Trial Balance Check** (Tổng Nợ = Tổng Có)
Tạo measure:
```dax
TB Check = 
IF(
    [Total Debit] = [Total Credit],
    "✓ Balanced",
    "✗ Difference: " & FORMAT([Total Debit] - [Total Credit], "#,##0")
)
```

Hiển thị trên **Card visual** ở đầu mỗi báo cáo.

Nếu lệch → **không bao giờ xuất bản báo cáo** cho user.

### Bước 10.2 — Test 2: **Subtotal Reconciliation** (Đối chiếu chéo)
So sánh **Sales Revenue từ Matrix visual** với **Sales Revenue từ phần mềm ERP**:
- Mở báo cáo "Sales by Customer" trong ERP → SUM(Sales) = X
- Mở Power BI Matrix cùng filter → SUM(Sales Revenue) = Y
- Nếu |X - Y| > 0 → **bug**

Tự động hóa bằng measure:
```dax
Reconciliation Diff = 
[Sales Revenue] - [ERP Sales Total]
```

### Bước 10.3 — Test 3: **Transaction-Level Reconciliation** (Đối chiếu từng dòng)
**Vấn đề:** Power BI có hàng triệu dòng, không thể check từng dòng bằng mắt.

**Cách làm:**
1. Trong Power BI: **Visualizations → Table visual** → kéo tất cả cột cần check
2. **Home → Export → Export data → CSV** (max 30,000 rows cho Pro, 150,000 cho Premium)
3. Mở CSV bằng Excel, dùng **VLOOKUP/XLOOKUP** để so với ERP
4. Đánh dấu dòng lệch

**Mẹo:** Chỉ export theo **1 kỳ / 1 cửa hàng** để dễ check:
```dax
Sales for Reconciliation = 
CALCULATE(
    [Sales Revenue],
    Calendar[YearMonth] = "2025-01",
    Stores[Store] = "Store 1"
)
```

### Bước 10.4 — Test 4: **Non-Functional Testing** (Test phi chức năng)
| Tiêu chí | Ngưỡng chấp nhận | Cách test |
|---|---|---|
| Thời gian load page | < 3s | Mở Performance Analyzer (View tab) |
| Thời gian refresh dataset | < 30 phút | Xem Scheduled refresh history |
| Accessibility | Đạt WCAG AA | Power BI Accessibility checker |
| Số visual/page | ≤ 8 | Tự đếm |
| Mobile render | Hiển thị đúng | Power BI Mobile app |

### Bước 10.5 — Phát hiện **DAX typo** (Lỗi chính tả)
Lỗi phổ biến nhất:
```dax
-- ❌ SAI: viết sai tên cột
[Total Amont]    -- thiếu chữ 'u'
[Total Amount]   -- ✅ ĐÚNG

-- ❌ SAI: viết sai tên bảng  
CALCULATE([Sales], GL_Transactions[Date] = ...)  -- sai tên bảng
CALCULATE([Sales], GLTransactions[Date] = ...)   -- ✅ ĐÚNG
```

**Cách phát hiện:**
- Có underline đỏ nguệch dưới tên hàm → sai
- Kết quả trả về BLANK nhưng không lỗi → thường do filter sai

### Bước 10.6 — Fix **opening balance** sai
Hệ thống nguồn đôi khi có số dư đầu kỳ sai. **KHÔNG sửa trong Power Query** vì:
- ❌ Tạo "nghĩa vụ" maintain mỗi lần refresh
- ❌ Audit trail bị mất
- ✅ **Sửa ở hệ thống gốc** (ERP), sau đó Power BI tự động đúng

Nếu bắt buộc sửa trong Power Query (vd: ERP không cho sửa):
1. **Filter Rows** chỉ giữ dòng opening balance
2. **Group By** theo AccountCode → Sum
3. **Merge** với bảng `ManualAdjustments`
4. **Append** lại

### Bước 10.7 — Test **Edge Cases** (Trường hợp đặc biệt)
| Edge case | Kết quả mong đợi |
|---|---|
| Chia cho 0 | DIVIDE trả về BLANK (không lỗi) |
| Lọc ngày không có dữ liệu | Trả về BLANK, không hiển thị "0" |
| User chưa login (RLS) | Không thấy data nào |
| Filter conflict (nhiều slicer) | Áp dụng AND logic |

---

## 💡 PHẦN 2 — HỌC SAU

### 10.1 Tại sao **fix at source** quan trọng?
> "If it's wrong in the source, it will be wrong in Power BI. **Always fix at the source.**"
> — Packt Publishing

- Power Query transformation chỉ là **"vá" tạm thời**
- Mỗi lần refresh → phải chạy lại transformation
- Nếu user tạo báo cáo phụ → họ không biết đang dùng data đã sửa → báo cáo khác nhau

### 10.2 Power BI Performance Analyzer
**View → Performance Analyzer** → Start → tương tác với báo cáo → Stop

Kết quả hiển thị:
- DAX query time (thường < 100ms)
- Visual display time (thường < 500ms)
- Other (thường < 200ms)

➡ Nếu **DAX query time > 1s** → cần tối ưu measure.

### 10.3 Khi nào KHÔNG nên test bằng Power BI
- **Stress test** với 10M rows → dùng SQL/DAX Studio
- **Unit test** DAX measure → dùng DAX Studio Tabular Editor
- **End-to-end test** với data mới → dùng Power BI Deployment Pipelines

### 10.4 Lỗi thường gặp
- ❌ Bỏ qua TB Check → báo cáo lệch mà không biết
- ❌ Export 100k rows trên Pro license → bị cắt 30k
- ❌ Sửa data trong Power Query thay vì sửa ở source → maintain nightmare
- ❌ Không test trên mobile → user khiếu nại
- ✅ Luôn có **TB Check** card trên mỗi report
- ✅ Dùng **Tabular Editor** để audit DAX

---

## ✅ Checklist Chương 10
- [ ] Có measure TB Check trên mỗi báo cáo
- [ ] Có quy trình export + reconcile với ERP
- [ ] Biết cách mở Performance Analyzer
- [ ] Luôn fix at source, không sửa trong Power Query trừ khi bất khả kháng

---

# CHƯƠNG 11: BÁO CÁO IN ĐƯỢC VỚI POWER BI PAGINATED

## 🎯 Mục tiêu
- Phân biệt **Interactive report** vs **Paginated report**
- Dựng Income Statement dạng **PDF in được** qua Power BI Report Builder
- Biết dùng **Parameters** để user chọn period, store

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 11.1 — Khi nào cần Paginated report?
| Tình huống | Dùng gì |
|---|---|
| CEO xem trên iPad, muốn click filter | **Interactive report** (Power BI Desktop) |
| In báo cáo P&L ra PDF gửi ngân hàng | **Paginated report** (Report Builder) |
| Báo cáo tuân thủ pháp lý (PDF 100 trang) | **Paginated report** |
| Dashboard KPI real-time | **Interactive report** |

➡ ABC Coffee cần **cả 2**:
- Interactive cho CEO xem nhanh
- Paginated cho báo cáo gửi ngân hàng hàng quý

### Bước 11.2 — Cài Power BI Report Builder
1. Tải từ: https://aka.ms/pbireportbuilder (miễn phí)
2. Cài đặt → mở lên

### Bước 11.3 — Kết nối với Power BI Dataset
1. **File → New → Blank Report**
2. Trong Report Data panel bên trái → **Data Sources → New Data Source**
3. Chọn **Microsoft SQL Server Analysis Services**
4. Connection string: `powerbi://api.powerbi.com/v1.0/myorg/ABC Coffee Workspace`
5. Chọn dataset `ABC_Coffee_Model`

### Bước 11.4 — Tạo **Dataset với DAX**
1. **Datasets → New Dataset → Use a dataset embedded in my report** (KHÔNG dùng, vì ta dùng Power BI dataset)
2. Hoặc **New → Dataset → Query Designer** → chọn measure, dimension, filter
3. Kéo `ChartOfAccounts[AccountCategory]` vào filter
4. Kéo measure `Sales Revenue`, `COGS`, `Net Income` vào Values

### Bước 11.5 — Dựng **P&L Paginated Report**
1. **Insert → Matrix** (không phải Table)
2. Kéo dataset vừa tạo vào matrix
3. **Rows**: `AccountCategory`
4. **Columns**: `Month`
5. **Values**: `Sales Revenue`, `COGS`, `Net Income`

### Bước 11.6 — Thêm **Parameters** (Tham số)
Người dùng muốn chọn **năm** và **cửa hàng** trước khi in:

1. **Report Data → Parameters → New Parameter**
2. Parameter 1: `Year`
   - Available values: query `SELECT DISTINCT Year FROM Calendar`
   - Default: 2025
3. Parameter 2: `Store`
   - Available values: query `SELECT StoreName FROM Stores`
   - Default: All
4. Trong dataset Query Designer → thêm filter:
   ```
   Calendar[Year] = @Year
   AND Stores[StoreName] = @Store
   ```

Khi user mở report → form popup hỏi Year + Store → in ra đúng kỳ.

### Bước 11.7 — Format trang in
- **Report → Report Properties → Page Setup**:
  - Orientation: Portrait
  - Size: A4
  - Margins: 2cm mỗi cạnh
- **Insert → Page Header**: Logo ABC Coffee + "Income Statement" + "Period: @Year"
- **Insert → Page Footer**: "Page X of Y" + "Generated: @ExecutionTime"

### Bước 11.8 — **Group** để hiển thị Subtotal
1. Click chuột phải vào group `AccountCategory` → **Add Group → Parent Group**
2. Group by: `AccountType` (Doanh thu / Giá vốn / Chi phí)
3. Subtotal hiển thị: tổng Doanh thu, tổng Giá vốn, **Lợi nhuận gộp**

### Bước 11.9 — Publish lên Power BI Service
1. **File → Save As** → `.rdl` file
2. Vào Power BI Service → workspace `ABC Coffee`
3. **New → Paginated Report → Upload** → chọn file `.rdl`
4. Khi user click → form popup → chọn Year + Store → xem/in

### Bước 11.10 — Schedule gửi email tự động
1. Trên paginated report đã publish → **... → Export → Subscribe**
2. Schedule: monthly, ngày 5 hàng tháng, 8:00 sáng
3. Recipients: ceo@abccoffee.vn, cfo@abccoffee.vn
4. Format: PDF
5. Lưu → Power BI tự động gửi PDF mỗi tháng

---

## 💡 PHẦN 2 — HỌC SAU

### 11.1 Paginated vs Interactive — sâu hơn
| | Interactive | Paginated |
|---|---|---|
| Render | Theo viewport (responsive) | Theo trang A4 (pixel-perfect) |
| Tương tác | Click, drill, filter | Chỉ parameter trước khi xem |
| Export | PDF, PowerPoint, Excel | PDF, Word, Excel, CSV, XML |
| Dùng cho | Dashboard, exploration | Báo cáo chính thức, in ấn |
| Công cụ | Power BI Desktop | Power BI Report Builder |
| Phí | Pro ($10/user/month) | Pro + Paginated workload |

### 11.2 Row Groups vs Column Groups trong Report Builder
```
                Q1 2025         Q2 2025
                Jan Feb Mar    Apr May Jun
─────────────────────────────────────────
Doanh thu (group)
  Bán cà phê    8M  9M 10M   11M 12M 13M
  Bán bánh      2M  2M  3M    3M  3M  4M
  Subtotal     10M 11M 13M   14M 15M 17M
─────────────────────────────────────────
COGS (group)
  Nguyên liệu  -4M -4M -5M   -5M -5M -6M
```
- **Column Group** = chia cột theo tháng/quý
- **Row Group** = chia hàng theo AccountCategory

### 11.3 DAX cho Paginated Report
Paginated Report dùng **MDX** hoặc **DAX** queries. Thường:
- Dùng DAX cho simple measure
- Dùng MDX cho complex filter với hierarchy

### 11.4 Lỗi thường gặp
- ❌ Paginated report quá nhiều cột (12 tháng × 3 measure) → chậm
- ❌ Parameter không có default → user phải tự chọn mỗi lần
- ❌ Page setup sai (landscape cho báo cáo dài) → bị cắt
- ❌ Không có page header/footer → báo cáo thiếu thông tin
- ✅ Dùng **Page Break** để tách các section dài
- ✅ Conditional formatting cho negative numbers (đỏ, ngoặc đơn)
- ✅ Test in trên **A4 thật** trước khi gửi cho sếp

---

## ✅ Checklist Chương 11
- [ ] Cài Power BI Report Builder
- [ ] Kết nối được với Power BI dataset
- [ ] Tạo Matrix P&L với Row + Column group
- [ ] Có Parameter cho Year + Store
- [ ] Page setup A4 portrait/landscape đúng
- [ ] Publish lên Service + schedule email

---

# CHƯƠNG 12: BÁO CÁO TÀI CHÍNH TRONG THẾ GIỚI AI

## 🎯 Mục tiêu
- Hiểu 5 loại AI: ML, Deep Learning, NLP, Computer Vision, Generative AI
- Biết cách **Copilot + Power BI** giúp gì trong tài chính
- Chuẩn bị dữ liệu cho AI (medallion architecture)
- Nhận diện **6 rủi ro** của AI trong tài chính

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 12.1 — 5 loại AI cần biết
| Loại | Làm gì | Ví dụ tài chính |
|---|---|---|
| **Machine Learning (ML)** | Học từ data dự đoán | Dự đoán doanh thu tháng sau |
| **Deep Learning (DL)** | ML với neural network nhiều layer | Phát hiện gian lận (anomaly) |
| **Natural Language Processing (NLP)** | Hiểu ngôn ngữ tự nhiên | Đọc 1000 hóa đơn, tự trích xuất |
| **Computer Vision** | Hiểu hình ảnh | Scan hóa đơn giấy → số |
| **Generative AI (Copilot, GPT)** | Tạo nội dung mới | "Tóm tắt báo cáo Q1 thành 3 dòng" |

### Bước 12.2 — **Start with business problem, NOT technology**
> ❌ Sai: "Mình dùng AI để làm gì đó cho ngầu"
> ✅ Đúng: "Mình có vấn đề X → AI có thể giải được không?"

ABC Coffee đặt câu hỏi:
1. Dự đoán doanh thu tuần sau? → ML Time Series
2. Tự động phân loại chi phí? → NLP trên description field
3. Cảnh báo khi chi phí tăng bất thường? → Anomaly Detection
4. Tự tạo email báo cáo tuần? → Generative AI (Copilot)

### Bước 12.3 — **Copilot trong Power BI** (từ 2024+)
Tính năng:
- **Tạo report từ prompt**: "Create a P&L by store for 2025" → Copilot tự dựng
- **Tạo DAX measure từ prompt**: "YoY growth of sales" → Copilot viết measure
- **Tạo summary tự động**: "Summarize this report" → 1 đoạn văn
- **Q&A**: User hỏi "What was last quarter's profit?" → Copilot tạo visual

**Bước dùng Copilot:**
1. Mở Power BI Desktop
2. Copilot button (góc phải)
3. Gõ: *"Create a P&L report by store with YoY comparison"*
4. Copilot tạo draft → bạn refine

### Bước 12.4 — **Microsoft Fabric** — Nền tảng tổng
Fabric = Power BI + Data Factory + Synapse + OneLake, tất cả trong 1.

```
┌─────────────────────────────────────────────────┐
│            Microsoft Fabric                     │
├──────────────┬──────────────┬─────────────────┤
│   OneLake    │   Power BI   │  Copilot        │
│  (Data Lake) │  (BI)        │  (AI assist)    │
├──────────────┴──────────────┴─────────────────┤
│  Lakehouse  │  Warehouse  │  Data Factory    │
│  (raw data) │  (curated)   │  (ETL)          │
├──────────────┴──────────────┴─────────────────┤
│  Bronze  →  Silver  →  Gold                    │
│  (Medallion Architecture)                      │
└─────────────────────────────────────────────────┘
```

**Medallion Architecture cho ABC Coffee:**
- **Bronze** (raw): GL_Raw_2024.csv, GL_Raw_2025.csv (giữ nguyên)
- **Silver** (cleaned): GL_Cleaned.parquet (đã chuẩn hóa date, account code)
- **Gold** (business): Sales_By_Day, PnL_Monthly (đã aggregate)

➡ User chỉ truy cập **Gold layer** qua Power BI — không bao giờ chạm Bronze.

### Bước 12.5 — Chuẩn bị data cho AI — 3 bước
1. **Simplify schema**: Bỏ cột không cần, đổi tên rõ ràng
2. **Verified answers**: Tạo bộ Q&A đã verify để Copilot học
3. **AI instructions**: Viết 1 file `AI_Instructions.txt` mô tả dataset

Ví dụ AI_Instructions.txt cho ABC Coffee:
```
Dataset: ABC_Coffee_Financial_2024_2025
Measures:
  - Sales Revenue: Total sales (exclude COGS)
  - COGS: Cost of goods sold
  - Net Income: Sales - COGS - Operating Expenses
Filters:
  - YearMonth: Format YYYY-MM
  - Store: Store 1, Store 2, Store 3
Currency: All amounts in VND
```

### Bước 12.6 — Phân biệt **Data Maturity Levels**
| Level | Mô tả | ABC Coffee đang ở |
|---|---|---|
| 1 | File Excel riêng lẻ | Đã qua |
| 2 | File Excel chia sẻ qua SharePoint | Đang ở đây |
| 3 | Power BI dataset tập trung | Mục tiêu 6 tháng |
| 4 | Data warehouse + semantic model | Mục tiêu 1 năm |
| 5 | AI-driven (Copilot tự trả lời) | Mục tiêu 2 năm |

### Bước 12.7 — **Data Planning** theo 3 khung thời gian
| Khung | Câu hỏi | Trả lời |
|---|---|---|
| **Ngắn hạn** (3-6 tháng) | Cần gì tuần này/tháng này? | MVP báo cáo Power BI |
| **Trung hạn** (6-18 tháng) | Cần gì trong 1-2 năm? | Fabric Lakehouse + Copilot |
| **Dài hạn** (18+ tháng) | Ngành tài chính sẽ thay đổi sao? | AI agents tự audit |

---

## 💡 PHẦN 2 — HỌC SAU

### 12.1 6 Rủi ro của AI trong tài chính
| Rủi ro | Ví dụ | Cách giảm |
|---|---|---|
| **Hallucination** | Copilot trả lời sai số liệu | Luôn verify với Trial Balance |
| **Garbage in, garbage out** | Data bẩn → AI học sai | Fix at source (Ch 10) |
| **Maintainability** | Code AI-generated khó maintain | Review mọi measure Copilot viết |
| **Accuracy** | 99% đúng vẫn = 1% sai | Đối chiếu mẫu manual |
| **Security** | Copilot lộ data nhạy cảm | RLS + audit log |
| **Bias** | AI học từ data thiên lệch | Đa dạng nguồn training |

### 12.2 Generative AI vs Traditional BI
| | Traditional BI | Generative AI |
|---|---|---|
| Câu hỏi | "Doanh thu Q1?" | "Tại sao doanh thu Q1 giảm?" |
| Trả lời | Con số | Con số + **giải thích + đề xuất** |
| Rủi ro | Thấp | Cao (hallucination) |
| Tốc độ | Tức thì | Tức thì (nhưng cần verify) |

### 12.3 Khi nào KHÔNG nên dùng AI
- ❌ Báo cáo **pháp lý** cần độ chính xác 100% (vẫn cần human verify)
- ❌ Quyết định **chiến lược lớn** (vd: M&A, sa thải)
- ❌ Data **nhạy cảm** chưa có governance

### 12.4 Lỗi thường gặp
- ❌ Tin Copilot 100% không verify
- ❌ Dùng AI khi data chưa clean
- ❌ Không có human-in-the-loop
- ✅ AI là **trợ lý**, quyết định cuối vẫn là người
- ✅ Verify mọi output với Trial Balance

---

## ✅ Checklist Chương 12
- [ ] Biết 5 loại AI + 1 ví dụ tài chính mỗi loại
- [ ] Đã thử Copilot trong Power BI
- [ ] Hiểu Medallion Architecture
- [ ] Liệt kê 6 rủi ro + cách giảm
- [ ] Xác định ABC Coffee ở Data Maturity level mấy

---

# CHƯƠNG 13: VẬN HÀNH & BẢO TRÌ BÁO CÁO

## 🎯 Mục tiêu
- Quản lý **workspace**, **deployment pipeline**
- Cấu hình **Row-Level Security** (RLS)
- Lên lịch **data refresh**
- Theo dõi, bảo trì, "về hưu" báo cáo

---

## 🛠 PHẦN 1 — LÀM TRƯỚC (với ABC Coffee)

### Bước 13.1 — Cấu trúc **Workspace**
| Workspace | Mục đích | License | Thành viên |
|---|---|---|---|
| **Dev** | Build, test | Pro | Developers |
| **Test** | UAT, training | Pro | Developers + 1 Manager |
| **Production** | User cuối dùng | Pro/Premium | Toàn công ty |
| **Sandbox** | Thử nghiệm | Pro | Power users |

ABC Coffee cần 3 workspaces: **Dev → Test → Production**

### Bước 13.2 — **Deployment Pipeline**
Power BI Service → workspace → **Deployment pipelines** → Create:
1. Pipeline name: `ABC_Coffee_Pipeline`
2. Stages: Dev → Test → Production
3. **Deploy from Dev to Test** (sau khi Developer test xong)
4. **Deploy from Test to Production** (sau khi Manager duyệt)

Lợi ích:
- ✅ User cuối không thấy "rác" lúc đang phát triển
- ✅ Có lịch sử deploy (audit trail)
- ✅ Rollback nhanh nếu báo cáo lỗi

### Bước 13.3 — **Row-Level Security (RLS)**
Kế toán cửa hàng 1 chỉ thấy data cửa hàng 1.

**Bước 3.1 — Tạo Role:**
1. Desktop → **Modeling → Manage Roles → Create**
2. Role name: `Store1_Accountant`
3. Table filter: `Stores[StoreName] = "Store 1"`

**Bước 3.2 — Test Role:**
- **Modeling → View as Roles** → tick `Store1_Accountant`
- Báo cáo chỉ hiển thị data cửa hàng 1

**Bước 3.3 — Publish + gán user:**
1. Publish lên Service
2. Workspace → **Security** → add email user → Role: `Store1_Accountant`

**Bước 13.3.4 — Dynamic RLS (User xem đúng cửa hàng mình quản lý)**

Tạo bảng `UserStoreMapping`:
| UserEmail | StoreName |
|---|---|
| khoa@abccoffee.vn | Store 1 |
| lan@abccoffee.vn | Store 2 |
| minh@abccoffee.vn | Store 3 |
| ceo@abccoffee.vn | (All) |

Trong Desktop:
1. Role: `Store_Accountant`
2. Table filter trên `UserStoreMapping`:
```dax
[StoreName] = USERPRINCIPALNAME()  -- hoặc LOOKUPVALUE
```

Nhưng `USERPRINCIPALNAME()` trả về email, nên cần bảng mapping. DAX tốt hơn:
```dax
[UserEmail] = USERPRINCIPALNAME()
```

➡ Khi user login → Power BI tự filter đúng cửa hàng.

### Bước 13.4 — **Data Refresh**
| License | Số lần refresh/ngày | Thời gian tối đa |
|---|---|---|
| **Pro** | 8 lần | 2 giờ/lần |
| **Premium Per User** | 48 lần | 5 giờ/lần |
| **Premium Per Capacity** | 48 lần | 5 giờ/lần |

**Source on-premises** (SQL Server tại văn phòng):
1. Cài **On-premises data gateway** trên 1 máy luôn bật
2. Power BI Service → **Manage gateways** → add data source
3. Scheduled refresh → chọn gateway

**ABC Coffee:**
- SQL Server ở văn phòng → cần gateway
- POS data từ cloud API → không cần gateway
- Schedule: refresh mỗi đêm 2:00 sáng

### Bước 13.5 — **Monitoring** với Usage Metrics
Workspace → Report → **Usage metrics** (built-in):
- Số lượt view/ngày
- User nào xem nhiều nhất
- Page nào được xem nhiều nhất
- Refresh history (success/fail)

➡ Nếu báo cáo ít view → có thể **không cần thiết** → "về hưu".

### Bước 13.6 — **Change Control** — Phân biệt Bug vs Change
| Loại | Ví dụ | Quy trình |
|---|---|---|
| **Bug** | Measure tính sai | Fix ngay, thông báo user |
| **Change** | Sếp muốn thêm KPI mới | Yêu cầu chính thức, đánh giá effort |

### Bước 13.7 — **Retirement** (Cho báo cáo "về hưu")
Khi nào nên retire?
- Không ai xem > 3 tháng
- Đã có báo cáo mới thay thế
- Data source không còn tồn tại

Quy trình:
1. Thông báo user: "Báo cáo X sẽ ngừng hỗ trợ từ 1/7"
2. Archive .pbix + .rdl
3. Remove khỏi workspace Production
4. Document trong Confluence/SharePoint

### Bước 13.8 — **Admin Portal** (cho Admin)
Power BI Service → **Settings → Admin portal**:
- **Tenant settings**: Bật/tắt export to PDF, publish to web
- **Usage metrics**: Track adoption
- **Audit logs**: Security/compliance
- **Capacity settings**: Premium capacity management

---

## 💡 PHẦN 2 — HỌC SAU

### 13.1 App vs Workspace — cái nào nên share?
| | App | Workspace |
|---|---|---|
| Ai truy cập | User cuối (read-only) | Developers/Contributors |
| Customize cho user? | Có (qua RLS) | Không |
| Branding | Logo + description custom | Mặc định workspace |
| Dùng khi | Chia sẻ cho **viewer** | Chia sẻ cho **builder** |

➡ ABC Coffee:
- Developers share **workspace** Test
- User cuối cài **app** từ Production

### 13.2 Refresh strategy
| Tần suất data thay đổi | Refresh schedule |
|---|---|
| Real-time (POS mỗi phút) | DirectQuery + Auto refresh |
| Mỗi giờ | 12 lần/ngày, mỗi 2h |
| Mỗi ngày | 1 lần lúc 2:00 sáng |
| Mỗi tuần | 1 lần Chủ nhật 23:00 |

### 13.3 Capacity planning
- **Pro** (10$/user/month): Dataset < 1GB, refresh 8/ngày
- **PPU** (20$/user/month): Dataset < 100GB, refresh 48/ngày, AI features
- **Premium per capacity** (~5,000$/tháng): Unlimited users, large dataset, dedicated resource

ABC Coffee 5 user → **PPU** (~100$/tháng) là đủ.

### 13.4 Lỗi thường gặp
- ❌ Không có Deployment Pipeline → Dev và Prod chung 1 workspace → rối
- ❌ RLS chỉ cấu hình Desktop, quên gán user trong Service
- ❌ Refresh fail vì gateway offline → user mất data
- ❌ Không archive báo cáo cũ → workspace ngày càng đầy
- ✅ Tự động **alert** khi refresh fail
- ✅ **Version control** .pbix qua OneDrive/Git
- ✅ **Documentation** trong file ReadMe.md kèm .pbix

---

## ✅ Checklist Chương 13
- [ ] Có 3 workspaces Dev/Test/Production
- [ ] Có Deployment Pipeline
- [ ] RLS hoạt động (test bằng "View as Role")
- [ ] Refresh schedule chạy đúng
- [ ] Usage metrics theo dõi adoption
- [ ] Quy trình retire báo cáo cũ

---

# 🎁 PHỤ LỤC: CHECKLIST TỔNG KẾT DỰ ÁN POWER BI

## Checklist "Go-Live" cho dự án BI tài chính

### A. Foundation (Ch 1-2)
- [ ] Scope MVP đã được CEO ký duyệt
- [ ] Star schema gồm: Calendar, CoA, GL, các dim phụ (Store, Product, Customer)
- [ ] Relationships đúng (1:N, single direction)
- [ ] Calendar marked as Date table

### B. Core DAX & Power Query (Ch 3-6)
- [ ] Trial Balance check = 0 (luôn đúng)
- [ ] P&L report có Gross Profit, Operating Profit, Net Income
- [ ] Time intelligence: MTD, QTD, YTD, PY hoạt động
- [ ] FX conversion với spot + average rate
- [ ] Power Query không có transformation thừa
- [ ] Sửa data ở source, không ở Power Query

### C. Data Quality (Ch 10)
- [ ] TB Check card trên mỗi report
- [ ] Reconciliation với ERP < 0.1% difference
- [ ] Performance < 3s/page
- [ ] Mobile render đúng
- [ ] Test edge cases (chia 0, ngày rỗng, RLS)

### D. Reports (Ch 9, 11)
- [ ] Tối đa 8 visual/page
- [ ] Visual đúng loại (pie chart chỉ < 5 lát)
- [ ] Consistent layout (font, color, slicer position)
- [ ] Paginated report cho báo cáo in được
- [ ] Parameter cho Year + Store

### E. Operations (Ch 13)
- [ ] 3 workspaces Dev/Test/Prod
- [ ] Deployment Pipeline
- [ ] RLS với dynamic UserStoreMapping
- [ ] Refresh schedule hàng ngày
- [ ] Usage metrics theo dõi adoption
- [ ] Documentation + Version control

### F. AI & Future (Ch 12)
- [ ] Đã thử Copilot trong Power BI
- [ ] Data ở Medallion Bronze → Silver → Gold
- [ ] AI_Instructions.txt cho dataset
- [ ] Roadmap AI trong 6-18 tháng

---

## 📚 BẢNG TRA CỨU NHANH DAX

| Mục đích | Công thức mẫu |
|---|---|
| Tổng đơn giản | `SUM(Table[Column])` |
| Tổng có filter | `CALCULATE(SUM(Table[Column]), Table2[Cat] = "X")` |
| Tỷ lệ % | `DIVIDE([Numerator], [Denominator])` |
| MTD | `TOTALMTD([Measure], Calendar[Date])` |
| YTD | `TOTALYTD([Measure], Calendar[Date], "12/31")` |
| Cùng kỳ năm trước | `CALCULATE([Measure], SAMEPERIODLASTYEAR(Calendar[Date]))` |
| Trung bình thời gian | `AVERAGEX(Table, DATEDIFF([Start], [End], DAY))` |
| Bỏ filter | `CALCULATE([Measure], ALL(Table))` |
| Đếm dòng có điều kiện | `COUNTROWS(FILTER(Table, Table[Col] > 0))` |
| Giá trị cuối của năm | `CALCULATE(MAX(Table[Date]), ALL(Calendar))` |

---

## 🎯 LỘ TRÌNH HỌC TIẾP

| Mức | Bạn đã biết | Nên học tiếp |
|---|---|---|
| **Beginner** (1-2 tháng) | Mở Power BI, import Excel, tạo visual đơn giản | Tabular Model, cơ bản DAX |
| **Intermediate** (3-6 tháng) | DAX cơ bản, Power Query, Matrix, Slicer | Time Intelligence, RLS, Deployment |
| **Advanced** (6-12 tháng) | Paginated Report, Performance tuning | Fabric, Copilot, AI integration |
| **Expert** (12+ tháng) | Architecture design cho enterprise | Python/R integration, custom visuals, Multi-Geo deployment |

---

## 📖 TÀI NGUYÊN THAM KHẢO

- **Sách gốc**: "Financial Modeling and Reporting with Microsoft Power BI" (Packt, 2026)
- **GitHub repo**: https://github.com/PacktPublishing/Financial-Modeling-with-Power-BI_Packt
- **Power BI Docs**: https://learn.microsoft.com/power-bi/
- **DAX Guide**: https://dax.guide
- **SQLBI** (Marco Russo, Alberto Ferrari): https://sqlbi.com
- **Cộng đồng Power BI Việt Nam**: Facebook group "Power BI Vietnam"

---

## ✨ LỜI KẾT

> **Bạn vừa hoàn thành tour 13 chương Power BI cho tài chính.** Từ "Excel thuần" đến "AI-driven financial reporting", tất cả qua **một ví dụ xuyên suốt: ABC Coffee**.

**3 điều cần nhớ nhất:**
1. **MVP trước, hoàn hảo sau** — báo cáo 80% chạy được tốt hơn 100% chưa chạy
2. **Fix at source** — đừng bao giờ vá trong Power Query khi có thể sửa ở ERP
3. **Verify mọi số liệu** — dù có AI/Copilot, Trial Balance = chân lý cuối cùng

➡ **Bước tiếp theo của bạn**: Mở Power BI Desktop, tạo file `ABC_Coffee.pbix`, bắt đầu từ **Chương 2 — Calendar table**. Chúc bạn thành công! 🚀

---

*File này được tạo tự động bởi GitHub Copilot dựa trên sách Packt Publishing 2026, với phong cách "Làm trước → Học sau" và ví dụ xuyên suốt ABC Coffee.*




