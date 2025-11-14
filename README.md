# Tên dự án: Phân tích giỏ hàng (Market Basket Analysis) bằng thuật toán Apriori để đưa ra gợi ý khi mua hàng

Dự án này thực hiện phân tích dữ liệu bán lẻ (từ file `OnlineRetail.xlsx`) để tìm ra các quy tắc kết hợp (association rules) giữa các sản phẩm.
Mục tiêu là khai phá dữ liệu để tìm ra những quy luật mua hàng mạnh mẽ, nhằm góp phần gợi ý cho khách hàng khi mua hàng.

## 1. Mục lục
1.  [Giới thiệu dự án](#1-giới-thiệu-dự-án)
2.  [Công nghệ sử dụng](#2-công-nghệ-sử-dụng)
3.  [Cài đặt (Installation)](#3-cài-đặt-installation)
4.  [Khám phá và Tiền xử lý Dữ liệu (EDA & Preprocessing)](#4-khám-phá-và-tiền-xử-lý-dữ-liệu-eda--preprocessing)
5.  [Phương pháp luận & Lựa chọn tham số](#5-phương-pháp-luận--lựa-chọn-tham-số)
6.  [Quy trình thực thi (Execution Flow) và Kết quả](#6-quy-trình-thực-thi-execution-flow-và-kết-quả)
7.  [Ứng dụng: Hàm gợi ý sản phẩm](#7-ứng-dụng-hàm-gợi-ý-sản-phẩm)
8.  [Kết luận](#8-kết-luận)

---

## 1. Giới thiệu dự án
* **Bài toán:** Phân tích hành vi mua sắm của khách hàng từ dữ liệu lịch sử giao dịch để tìm ra các sản phẩm nào thường được mua cùng nhau.
* **Mục tiêu:**
    * Áp dụng thuật toán Apriori để trích xuất các quy tắc kết hợp có ý nghĩa (với các ngưỡng Support, Confidence, Lift cụ thể).
* **Dữ liệu:** Sử dụng bộ dữ liệu `OnlineRetail.xlsx`, chứa 541,909 bản ghi giao dịch.

---

## 2. Công nghệ sử dụng
Dự án được xây dựng chủ yếu bằng Python (trong môi trường Jupyter Notebook) với các thư viện sau:
* **Pandas:** Dùng cho việc đọc, làm sạch và chuyển đổi dữ liệu.
* **Mlxtend:** Dùng để triển khai thuật toán Apriori và sinh luật kết hợp.
* **Matplotlib & Seaborn:** Dùng để trực quan hóa dữ liệu (EDA) và kết quả.
* **NetworkX:** Dùng để vẽ đồ thị mạng lưới các quy tắc.

---

## 3. Cài đặt (Installation)
Hướng dẫn các bước để thiết lập môi trường và chạy dự án này.

1.  **Clone repository về máy:**
    ```bash
    git clone https://github.com/NguyenBk22/CO3029_DATA_MINING_L01.git
    cd CO3029_DATA_MINING_L01
    ```

2.  **Tạo và kích hoạt môi trường ảo (khuyến nghị):**
    * Trên Windows:
        ```bash
        python -m venv venv
        .\venv\Scripts\activate
        ```
    * Trên macOS/Linux:
        ```bash
        python3 -m venv venv
        source venv/bin/activate
        ```

3.  **Cài đặt các thư viện cần thiết:**
    ```bash
    pip install -r requirements.txt
    ```
---

## 4. Khám phá và Tiền xử lý Dữ liệu (EDA & Preprocessing)
Quy trình này được thực hiện chi tiết trong file **`Data_processing.ipynb`**.

### 4.1. Mô tả Dữ liệu Gốc (Input)
* **Tên file:** `OnlineRetail.xlsx`
* **Mô tả:** Đây là file dữ liệu thô chứa **541,909 bản ghi** giao dịch bán lẻ.
* **Cấu trúc (8 cột):** Dữ liệu ban đầu bao gồm các trường sau:
    * `InvoiceNo`: Mã số hóa đơn. Đây là mã định danh cho mỗi giao dịch. (Lưu ý: Các mã bắt đầu bằng 'C' là các giao dịch đã bị hủy).
    * `StockCode`: Mã định danh cho từng sản phẩm.
    * `Description`: Tên/mô tả của sản phẩm.
    * `Quantity`: Số lượng của sản phẩm trong giao dịch đó.
    * `InvoiceDate`: Ngày và giờ giao dịch được thực hiện.
    * `UnitPrice`: Đơn giá của một sản phẩm.
    * `CustomerID`: Mã định danh duy nhất cho mỗi khách hàng. (Lưu ý: Một số dòng bị thiếu thông tin này).
    * `Country`: Quốc gia nơi giao dịch diễn ra (ví dụ: 'United Kingdom').

### 4.2. Khám phá dữ liệu (EDA)
* Trực quan hóa cho thấy các sản phẩm bán chạy nhất, ví dụ: "white hanging heart t-light holder".
* Phân tích sâu hơn trong `Apriori.ipynb` cho thấy giỏ hàng trung bình có 20.68 mặt hàng, và số mặt hàng phổ biến nhất (median) là 15.

### 4.3. Làm sạch (Cleaning)
Quy trình làm sạch được áp dụng để chuẩn bị dữ liệu cho phân tích:
* Loại bỏ các dòng thiếu `CustomerID` hoặc `Description`.
* Loại bỏ các hóa đơn bị hủy (những dòng có `InvoiceNo` bắt đầu bằng 'C').
* Chỉ lọc các giao dịch tại 'United Kingdom' để đảm bảo tính đồng nhất của dữ liệu.
* Chuẩn hóa mô tả sản phẩm (viết thường, xóa khoảng trắng).

### 4.4. Chuyển đổi (Transformation)
* Dữ liệu được chuyển đổi từ dạng danh sách giao dịch (dài) sang **ma trận giao dịch (transaction matrix)**.
* Trong ma trận này:
    * Mỗi hàng là một `InvoiceNo` (hóa đơn) duy nhất.
    * Mỗi cột là một `Description` (sản phẩm) duy nhất.
* Giá trị được mã hóa nhị phân (one-hot encoding): **1** nếu sản phẩm có trong giỏ, **0** nếu không có.
* Kết quả: Ma trận có kích thước (16649, 3833), tương ứng 16,649 giao dịch hợp lệ và 3,833 sản phẩm riêng biệt sau khi lọc.
* **Dữ liệu đã xử lý (Output):** Ma trận này được lưu lại thành file `cleaned_basket.csv` để sử dụng cho các bước sau.
---

## 5. Phương pháp luận & Lựa chọn tham số
* **5.1. Thuật toán Apriori:**
    * Apriori là thuật toán khai phá quy tắc kết hợp, hoạt động dựa trên 3 chỉ số chính:
        1.  **Support (Độ hỗ trợ):** Tỷ lệ các giao dịch chứa một tập sản phẩm (itemset).
        2.  **Confidence (Độ tin cậy):** Tỷ lệ các giao dịch chứa {A, B} trong số các giao dịch chứa {A}. (Đo lường P(B|A)).
        3.  **Lift (Độ nâng):** Đo lường mức độ {A} và {B} xuất hiện cùng nhau có thường xuyên hơn mức ngẫu nhiên hay không. (Lift > 1 cho thấy mối quan hệ tích cực).

* **5.2. Lựa chọn tham số (File: `test_min_support.ipynb`):**
    * Việc chọn `min_support` (ngưỡng hỗ trợ tối thiểu) rất quan trọng. Nếu quá cao, sẽ có ít luật; nếu quá thấp, số lượng luật sẽ bùng nổ và thời gian xử lý lâu.
    * Một thử nghiệm đã được chạy trên các ngưỡng support khác nhau (lọc luật với Lift > 1.0 và Confidence >= 0.5):

| Min Support (%) | Số giao dịch tối thiểu | Số tập phổ biến (Itemsets) | Số luật (Rules) | Thời gian (giây) |
|:--- |:--- |:--- |:--- |:--- |
| 2.0% | 332 | 236 | 30 | 11.72s |
| 1.5% | 249 | 441 | 67 | 25.93s |
| **1.0%** | **166** | **971** | **267** | **22.28s** |
| 0.5% | 83 | 4074 | 3730 | 55.30s |

Kết quả từ: `test_min_support.ipynb`

    Quyết định: Chọn `min_support = 0.01` (1.0%) làm ngưỡng hỗ trợ cho phân tích chính thức, vì nó cân bằng giữa số lượng luật tìm được (267 luật) và thời gian thực thi.

---

## 6. Quy trình thực thi (Execution Flow) và Kết quả
Phần này mô tả các bước chạy code của dự án và trình bày các kết quả chính.

### 6.1. Luồng thực thi (Flow chạy)
Dự án được chia thành 3 file Notebook chính, chạy theo thứ tự sau:

1.  **`Data_processing.ipynb`:**
    * **Mục đích:** Tiền xử lý dữ liệu thô.
    * **Đầu vào:** `OnlineRetail.xlsx` (File dữ liệu gốc).
    * **Đầu ra:** `cleaned_basket.csv` (Ma trận giao dịch nhị phân).

2.  **`test_min_support.ipynb` (Bước thử nghiệm):**
    * **Mục đích:** Chạy thử nghiệm Apriori với nhiều ngưỡng `min_support` (2%, 1.5%, 1%, 0.5%) để tìm ra ngưỡng tối ưu.
    * **Đầu vào:** `cleaned_basket.csv`
    * **Đầu ra:** Bảng so sánh (đã trình bày ở Mục 5.2).

3.  **`Apriori.ipynb` (Bước phân tích chính):**
    * **Mục đích:** Chạy Apriori với ngưỡng đã chọn (0.01), phân tích, trực quan hóa và sinh luật cuối cùng.
    * **Đầu vào:** `cleaned_basket.csv`
    * **Đầu ra:** `association_rules_final.csv` (File CSV chứa các luật đã lọc).

### 6.2. Kết quả phân tích (Analysis & Results)
*Thực hiện trong `Apriori.ipynb`*
* **Top 10 luật mạnh nhất (theo Lift):**

| Tiền đề (Antecedents) | Hậu đề (Consequents) | Support | Confidence | Lift |
|:--- |:--- |:--- |:--- |:--- |
| {herb marker thyme} | {herb marker rosemary} | 0.010 | 0.944 | 86.84 |
| {herb marker rosemary} | {herb marker thyme} | 0.010 | 0.934 | 86.84 |
| {regency tea plate green} | {regency tea plate roses} | 0.012 | 0.846 | 52.94 |
| {regency tea plate roses} | {regency tea plate green} | 0.012 | 0.722 | 52.94 |
| {poppy's playhouse bedroom} | {poppy's playhouse livingroom} | 0.010 | 0.650 | 51.78 |
...


* **Trực quan hóa:**
    * **Biểu đồ Scatter (Support vs Confidence, cột màu là Lift):** Cho thấy các luật có Lift cao thường có Support thấp.
    * **Đồ thị mạng lưới (Network Graph):** Trực quan hóa mối liên hệ giữa 20 luật hàng đầu, cho thấy các cụm sản phẩm như "bộ tách trà regency" (hồng, xanh, hoa hồng) và "bộ poppy's playhouse" (phòng ngủ, nhà bếp, phòng khách) có liên kết rất chặt chẽ.
    * **Heatmap (Ma trận đồng xuất hiện):** Trực quan hóa tần suất 20 sản phẩm bán chạy nhất xuất hiện cùng nhau, giúp xác định các "cặp đôi" phổ biến.

---

## 7. Ứng dụng: Hàm gợi ý sản phẩm
* Dựa trên các luật đã tìm được, file `Apriori.ipynb` xây dựng hàm `recommend_from_basket`.
* Hàm này nhận vào một danh sách các sản phẩm trong giỏ, tìm tất cả các luật mà "tiền đề" (antecedents) của luật nằm trong giỏ.
* Sau đó, hàm trả về các "hậu đề" (consequents) tương ứng (những sản phẩm chưa có trong giỏ), sắp xếp theo độ tin cậy (confidence) cao nhất.

* **Ví dụ sử dụng:**
    1.  **Khi giỏ hàng có `['pink regency teacup and saucer']`:**
        * Gợi ý hàng đầu là `('green regency teacup and saucer', 0.819)` và `('roses regency teacup and saucer', 0.777)`.
    2.  **Khi giỏ hàng có `['pink regency teacup and saucer', 'roses regency teacup and saucer']`:**
        * Gợi ý hàng đầu (và mạnh hơn) là `('green regency teacup and saucer', 0.890)`.

---

## 8. Kết luận
Dự án đã tiền xử lý thành công bộ dữ liệu bán lẻ lớn và áp dụng thuật toán Apriori (với `min_support=1.0%`) để tìm ra 267 quy tắc kết hợp có ý nghĩa. Các quy tắc này không chỉ cho thấy các mối liên hệ thú vị (như regency tea plate, poppy's playhouse) mà còn có thể được ứng dụng trực tiếp để xây dựng một hệ thống gợi ý sản phẩm đơn giản, giúp tăng doanh số bán hàng.