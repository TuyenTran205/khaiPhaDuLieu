# ĐÁP ÁN CHI TIẾT BÀI TẬP VỀ LUẬT KẾT HỢP
> Môn: **Khai phá dữ liệu (Data Mining)**  
> Dựa trên tài liệu đề bài `Bài tập về luật kết hợp.pdf` và các bài chữa mẫu trên lớp.  
> Các thuật toán sử dụng: **Apriori**, **FP-growth**, **Khai phá luật kết hợp (Association Rules)**.

---

## MỤC LỤC
1. [Khung lý thuyết & công thức chung](#khung-ly-thuyet)
2. [Bài 1](#bai-1) — CSDL 6 giao dịch {ACDI, ACI, CEI, ABDE, BDI, ABDE} (minsup = 0.3, minconf = 1.0)
3. [Bài 2](#bai-2) — Bài tập tập mặt hàng I={i1..i8}, O={o1..o6} (minsup = 0.3, minconf = 1.0)
4. [Bài 3](#bai-3) — Đảo thứ tự FP-growth trước, Apriori sau
5. [Bài 4](#bai-4) — CSDL nhị phân 6 giao dịch D, E, T, H, I, M (minsup = 60%, minconf = 100%)
6. [Bài 5](#bai-5) — CSDL nhị phân 10 giao dịch (Lạc, Vừng, Đỗ, Bánh mì, Bơ, Gạo, Lúa mì)
7. [Bài 6](#bai-6) — CSDL 5 giao dịch có MinSup = 3, MinConf = 1
8. [Bài 7](#bai-7) — CSDL 5 giao dịch A..N (minsup = 60%, minconf = 100%)
9. [Bài 8](#bai-8) — Hướng dẫn thực hành trên tập dữ liệu UCI

---

<a name="khung-ly-thuyet"></a>
## 1. KHUNG LÝ THUYẾT & CÔNG THỨC CHUNG

### 1.1. Các đại lượng cơ bản
- Cho CSDL giao dịch $D$ gồm $|D| = N$ giao dịch và tập hạng mục (itemset) $I$.
- **Độ hỗ trợ (Support count / Count):** $\text{count}(X) = \sigma(X)$ là số giao dịch trong $D$ chứa tập $X$.
- **Độ hỗ trợ tương đối (Relative Support):**
  $$\text{supp}(X) = \frac{\text{count}(X)}{N}$$
- **Ngưỡng độ hỗ trợ tối thiểu:**
  $$\text{mincount} = \lceil \text{minsup} \times N \rceil$$
  *Quy tắc chuẩn:* Tập $X$ là tập phổ biến khi và chỉ khi $\text{count}(X) \ge \text{mincount}$. Nếu phép nhân ra số lẻ (ví dụ $1.8$ hay $3.6$), trong thực hành trên lớp ta lấy cận trên nguyên nhỏ nhất thoả mãn ($\ge 1.8 \Rightarrow \text{count} \ge 2$; $\ge 3.6 \Rightarrow \text{count} \ge 4$).

- **Độ tin cậy của luật kết hợp $R: A \Rightarrow B$** ($A \cap B = \emptyset, A \neq \emptyset, B \neq \emptyset$):
  $$\text{conf}(R) = \frac{\text{supp}(A \cup B)}{\text{supp}(A)} = \frac{\text{count}(A \cup B)}{\text{count}(A)}$$
  Luật $R$ hợp lệ khi $\text{conf}(R) \ge \text{minconf}$.  
  Đặc biệt khi $\text{minconf} = 1.0$ (100%), luật hợp lệ khi và chỉ khi:
  $$\text{count}(A \cup B) = \text{count}(A)$$

### 1.2. Phân loại tập phổ biến
- **Tập phổ biến (Frequent Itemset):** Có $\text{supp}(X) \ge \text{minsup}$.
- **Tập phổ biến đóng (Closed Frequent Itemset):** Là tập phổ biến và **không tồn tại** tập cha thực sự $Y \supset X$ sao cho $\text{count}(Y) = \text{count}(X)$.
- **Tập phổ biến tối đại (Maximal Frequent Itemset):** Là tập phổ biến và **không tồn tại** tập cha thực sự $Y \supset X$ nào cũng là tập phổ biến.
  $$\text{Maximal} \subseteq \text{Closed} \subseteq \text{Frequent}$$

---

<a name="bai-1"></a>
## BÀI 1

### Đề bài
Cho Cơ sở dữ liệu giao dịch:

| TID | Nội dung giao dịch |
|:---:|:---|
| 100 | A, C, D, I |
| 200 | A, C, I |
| 300 | C, E, I |
| 400 | A, B, D, E |
| 500 | B, D, I |
| 600 | A, B, D, E |

Ngưỡng: $\text{minsup} = 0.3$, $\text{minconf} = 1.0$.

### Chuẩn bị
- Số giao dịch $N = 6$.
- Ngưỡng hỗ trợ đếm: $\text{mincount} = 0.3 \times 6 = 1.8 \Rightarrow \mathbf{\text{count} \ge 2}$.
- Ngưỡng độ tin cậy: $\text{minconf} = 1.0$ (100%).

---

### a) Thuật toán Apriori tìm tất cả các tập phổ biến

#### Bước 1: Khởi tạo $C_1$ và tìm $L_1$
Đếm tần số xuất hiện của từng mặt hàng đơn lẻ qua 1 lần quét CSDL:

| Item | Giao dịch chứa | Count | Đạt ngưỡng ($\ge 2$)? |
|:---:|:---|:---:|:---:|
| A | 100, 200, 400, 600 | 4 | Có $\to L_1$ |
| B | 400, 500, 600 | 3 | Có $\to L_1$ |
| C | 100, 200, 300 | 3 | Có $\to L_1$ |
| D | 100, 400, 500, 600 | 4 | Có $\to L_1$ |
| E | 300, 400, 600 | 3 | Có $\to L_1$ |
| I | 100, 200, 300, 500 | 4 | Có $\to L_1$ |

$$\mathbf{L_1 = \{ \{A\}:4,\; \{B\}:3,\; \{C\}:3,\; \{D\}:4,\; \{E\}:3,\; \{I\}:4 \}}$$

#### Bước 2: Sinh ứng viên $C_2$ từ $L_1$ và tìm $L_2$
Tổ hợp chập 2 từ 6 phần tử của $L_1$ có $\binom{6}{2} = 15$ cặp. Vì mọi tập con 1 phần tử đều thuộc $L_1$ nên không có ứng viên nào bị tỉa trước khi đếm. Quét lại CSDL:

| Ứng viên $C_2$ | Giao dịch chứa | Count | Kết luận |
|:---|:---|:---:|:---:|
| {A, B} | 400, 600 | 2 | Đạt $\to L_2$ |
| {A, C} | 100, 200 | 2 | Đạt $\to L_2$ |
| {A, D} | 100, 400, 600 | 3 | Đạt $\to L_2$ |
| {A, E} | 400, 600 | 2 | Đạt $\to L_2$ |
| {A, I} | 100, 200 | 2 | Đạt $\to L_2$ |
| {B, C} | *(không có)* | 0 | Loại |
| {B, D} | 400, 500, 600 | 3 | Đạt $\to L_2$ |
| {B, E} | 400, 600 | 2 | Đạt $\to L_2$ |
| {B, I} | 500 | 1 | Loại |
| {C, D} | 100 | 1 | Loại |
| {C, E} | 300 | 1 | Loại |
| {C, I} | 100, 200, 300 | 3 | Đạt $\to L_2$ |
| {D, E} | 400, 600 | 2 | Đạt $\to L_2$ |
| {D, I} | 100, 500 | 2 | Đạt $\to L_2$ |
| {E, I} | 300 | 1 | Loại |

$$\mathbf{L_2 = \{ \{A,D\}:3, \{B,D\}:3, \{C,I\}:3, \{A,B\}:2, \{A,C\}:2, \{A,E\}:2, \{A,I\}:2, \{B,E\}:2, \{D,E\}:2, \{D,I\}:2 \}}$$

#### Bước 3: Sinh ứng viên $C_3$ từ $L_2$ và tìm $L_3$
Ghép các cặp trong $L_2$ có cùng tiền tố và kiểm tra điều kiện tỉa (mọi tập con 2 phần tử phải nằm trong $L_2$):
- Ghép {A,B} với {A,D} $\to$ {A,B,D}: các tập con {A,B}, {A,D}, {B,D} đều $\in L_2 \implies$ giữ.
- Ghép {A,B} với {A,E} $\to$ {A,B,E}: các tập con {A,B}, {A,E}, {B,E} đều $\in L_2 \implies$ giữ.
- Ghép {A,C} với {A,I} $\to$ {A,C,I}: các tập con {A,C}, {A,I}, {C,I} đều $\in L_2 \implies$ giữ.
- Ghép {A,D} với {A,E} $\to$ {A,D,E}: các tập con {A,D}, {A,E}, {D,E} đều $\in L_2 \implies$ giữ.
- Ghép {B,D} với {B,E} $\to$ {B,D,E}: các tập con {B,D}, {B,E}, {D,E} đều $\in L_2 \implies$ giữ.
- Các tổ hợp khác như {A,D,I} bị tỉa vì {A,I} $\in L_2, \{A,D\} \in L_2$ nhưng $\{D,I\} \in L_2$ (tuy nhiên count thực tế của {A,D,I} chỉ xuất hiện ở TID 100 = 1 $\to$ loại).

Đếm trong CSDL:
| Ứng viên $C_3$ | Giao dịch chứa | Count | Kết luận |
|:---|:---|:---:|:---:|
| {A, B, D} | 400, 600 | 2 | Đạt $\to L_3$ |
| {A, B, E} | 400, 600 | 2 | Đạt $\to L_3$ |
| {A, C, I} | 100, 200 | 2 | Đạt $\to L_3$ |
| {A, D, E} | 400, 600 | 2 | Đạt $\to L_3$ |
| {B, D, E} | 400, 600 | 2 | Đạt $\to L_3$ |
| {A, D, I} | 100 | 1 | Loại |

$$\mathbf{L_3 = \{ \{A,B,D\}:2, \{A,B,E\}:2, \{A,C,I\}:2, \{A,D,E\}:2, \{B,D,E\}:2 \}}$$

#### Bước 4: Sinh ứng viên $C_4$ từ $L_3$ và tìm $L_4$
Ghép các tập 3 phần tử:
- {A,B,D} ghép {A,B,E} $\to$ {A,B,D,E}.
- Kiểm tra tập con: {A,B,D}, {A,B,E}, {A,D,E}, {B,D,E} — cả 4 tập con 3 phần tử đều thuộc $L_3 \implies$ giữ làm ứng viên duy nhất của $C_4$.

Đếm trong CSDL:
- {A, B, D, E} xuất hiện ở giao dịch 400 và 600 $\to \text{count} = 2 \ge 2 \implies \mathbf{L_4 = \{ \{A,B,D,E\}: 2 \}}$.

#### Bước 5: Sinh $C_5$
Không thể ghép để tạo tập 5 phần tử $\implies C_5 = \emptyset$. Dừng thuật toán.

#### Kết luận câu 1a
Tập hợp tất cả các tập phổ biến $L = L_1 \cup L_2 \cup L_3 \cup L_4$ gồm **22 tập phổ biến**:
- **1-itemset (6):** {A}:4, {B}:3, {C}:3, {D}:4, {E}:3, {I}:4
- **2-itemset (10):** {A,D}:3, {B,D}:3, {C,I}:3, {A,B}:2, {A,C}:2, {A,E}:2, {A,I}:2, {B,E}:2, {D,E}:2, {D,I}:2
- **3-itemset (5):** {A,B,D}:2, {A,B,E}:2, {A,C,I}:2, {A,D,E}:2, {B,D,E}:2
- **4-itemset (1):** {A,B,D,E}:2

---

### b) Thuật toán FP-growth tìm tất cả các tập phổ biến

#### Bước 1: Lập bảng tiêu đề (Header Table) sắp xếp giảm dần theo tần số
Tần số từ $L_1$: $A:4, D:4, I:4, B:3, C:3, E:3$.  
Thứ tự ưu tiên chuẩn (giảm dần theo count): **$A \succ D \succ I \succ B \succ C \succ E$**.

#### Bước 2: Sắp xếp lại từng giao dịch theo thứ tự trên
- TID 100: $\{A, C, D, I\} \to \mathbf{A, D, I, C}$
- TID 200: $\{A, C, I\} \to \mathbf{A, I, C}$
- TID 300: $\{C, E, I\} \to \mathbf{I, C, E}$
- TID 400: $\{A, B, D, E\} \to \mathbf{A, D, B, E}$
- TID 500: $\{B, D, I\} \to \mathbf{D, I, B}$
- TID 600: $\{A, B, D, E\} \to \mathbf{A, D, B, E}$

#### Bước 3: Dựng cây FP-tree gốc
Chèn lần lượt 6 chuỗi trên vào cây bắt đầu từ gốc `null`:
```text
                  null
               /    |    \
             A:4   I:1   D:1
            /   \    |     |
          D:3   I:1  C:1   I:1
         /   \    |   |     |
       I:1   B:2  C:1 E:1   B:1
        |     |
       C:1   E:2
```

#### Bước 4: Khai phá FP-tree (duyệt từ dưới lên theo thứ tự header ngược: E, C, B, I, D, A)

| Item | Cơ sở mẫu điều kiện (Conditional Pattern Base) | FP-tree điều kiện | Tập phổ biến sinh ra |
|:---:|:---|:---|:---|
| **E** | {A, D, B}: 2<br>{I, C}: 1 | Đường đơn $\langle A:2, D:2, B:2 \rangle$ (vì I, C chỉ có count=1 bị loại) | {E}:3<br>{A,E}:2, {D,E}:2, {B,E}:2<br>{A,D,E}:2, {A,B,E}:2, {B,D,E}:2<br>{A,B,D,E}:2 |
| **C** | {A, D, I}: 1<br>{A, I}: 1<br>{I}: 1 | $\langle I:3, A:2 \rangle$ (nhánh I có count=3 $\ge 2$, A có count=2 $\ge 2$, D count=1 loại) | {C}:3<br>{I,C}:3, {A,C}:2<br>{A,C,I}:2 |
| **B** | {A, D}: 2<br>{D, I}: 1 | $\langle D:3, A:2 \rangle$ (D count=3, A count=2, I count=1 loại) | {B}:3<br>{D,B}:3, {A,B}:2<br>{A,D,B}:2 |
| **I** | {A, D}: 1<br>{A}: 1<br>{D}: 1 | $A:2, D:2$ nhưng không cùng trên một đường đi đơn $\ge 2$ | {I}:4<br>{A,I}:2, {D,I}:2 |
| **D** | {A}: 3 | $\langle A:3 \rangle$ | {D}:4<br>{A,D}:3 |
| **A** | $\emptyset$ | $\emptyset$ | {A}:4 |

**Kết luận:** Tập phổ biến tìm được qua FP-growth **hoàn toàn trùng khớp** với Apriori (22 tập).

---

### c) Tìm tất cả các luật kết hợp từ tập phổ biến có nhiều phần tử nhất

Tập phổ biến có nhiều phần tử nhất là tập 4 phần tử: **$X = \{A, B, D, E\}$** với $\text{count}(X) = 2$.  
Vì $\text{minconf} = 1.0$, luật $R: s \Rightarrow (X \setminus s)$ hợp lệ khi và chỉ khi:
$$\text{conf}(R) = \frac{\text{count}(X)}{\text{count}(s)} = \frac{2}{\text{count}(s)} \ge 1.0 \iff \mathbf{\text{count}(s) \le 2}$$
Mà $s \subset X$ là tập con thực sự không rỗng của $X$ nên $\text{count}(s) \ge \text{count}(X) = 2$. Do đó điều kiện tương đương là **$\text{count}(s) = 2$**.

Liệt kê tất cả $2^4 - 2 = 14$ luật sinh từ $\{A, B, D, E\}$:

| STT | Vế trái $s$ | Vế phải $X \setminus s$ | $\text{count}(s)$ | $\text{conf} = \frac{2}{\text{count}(s)}$ | Đạt minconf=1.0? |
|:---:|:---|:---|:---:|:---:|:---:|
| 1 | {A} | {B, D, E} | 4 | $2/4 = 0.50$ | Không |
| 2 | {B} | {A, D, E} | 3 | $2/3 = 0.67$ | Không |
| 3 | {D} | {A, B, E} | 4 | $2/4 = 0.50$ | Không |
| 4 | {E} | {A, B, D} | 3 | $2/3 = 0.67$ | Không |
| 5 | **{A, B}** | **{D, E}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 6 | {A, D} | {B, E} | 3 | $2/3 = 0.67$ | Không |
| 7 | **{A, E}** | **{B, D}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 8 | {B, D} | {A, E} | 3 | $2/3 = 0.67$ | Không |
| 9 | **{B, E}** | **{A, D}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 10 | **{D, E}** | **{A, B}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 11 | **{A, B, D}** | **{E}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 12 | **{A, B, E}** | **{D}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 13 | **{A, D, E}** | **{B}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |
| 14 | **{B, D, E}** | **{A}** | 2 | **$2/2 = 1.00$** | **HỢP LỆ** |

**Kết luận:** Có **8 luật kết hợp hợp lệ** đạt $\text{conf} = 100\%$:
1. $\{A, B\} \Rightarrow \{D, E\}$
2. $\{A, E\} \Rightarrow \{B, D\}$
3. $\{B, E\} \Rightarrow \{A, D\}$
4. $\{D, E\} \Rightarrow \{A, B\}$
5. $\{A, B, D\} \Rightarrow \{E\}$
6. $\{A, B, E\} \Rightarrow \{D\}$
7. $\{A, D, E\} \Rightarrow \{B\}$
8. $\{B, D, E\} \Rightarrow \{A\}$

---

<a name="bai-2"></a>
## BÀI 2

### Đề bài
Cho tập mặt hàng $I = \{i_1, i_2, i_3, i_4, i_5, i_6, i_7, i_8\}$ và tập giao tác $O = \{o_1, o_2, o_3, o_4, o_5, o_6\}$:
- $o_1 = \{i_1, i_7, i_8\}$
- $o_2 = \{i_1, i_2, i_6, i_7, i_8\}$
- $o_3 = \{i_1, i_2, i_6, i_7\}$
- $o_4 = \{i_1, i_7, i_8\}$
- $o_5 = \{i_3, i_4, i_5, i_6, i_8\}$
- $o_6 = \{i_1, i_4, i_5\}$

Ngưỡng: $\text{minsup} = 0.3$, $\text{minconf} = 1.0$.

---

### a. Tìm ngữ cảnh khai thác dữ liệu (Formal Context) tạo từ I, O
Ngữ cảnh khai thác dữ liệu là một bộ ba $(O, I, R)$ trong đó $R \subseteq O \times I$ biểu diễn giao dịch $o$ chứa mặt hàng $i$. Thể hiện dưới dạng bảng nhị phân:

| Giao tác \ Mặt hàng | $i_1$ | $i_2$ | $i_3$ | $i_4$ | $i_5$ | $i_6$ | $i_7$ | $i_8$ |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| $o_1$ | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 1 |
| $o_2$ | 1 | 1 | 0 | 0 | 0 | 1 | 1 | 1 |
| $o_3$ | 1 | 1 | 0 | 0 | 0 | 1 | 1 | 0 |
| $o_4$ | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 1 |
| $o_5$ | 0 | 0 | 1 | 1 | 1 | 1 | 0 | 1 |
| $o_6$ | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 0 |

---

### b. Tìm tất cả các tập phổ biến theo ngưỡng minsupp = 0.3
- $N = 6 \implies \text{mincount} = \lceil 0.3 \times 6 \rceil = \mathbf{2}$.

**Lần lặp 1:**
- Đếm $C_1$: $i_1:5, i_2:2, i_3:1, i_4:2, i_5:2, i_6:3, i_7:4, i_8:4$.
- Bỏ $i_3$ (count=1 < 2).
$$\mathbf{L_1 = \{ \{i_1\}:5, \{i_7\}:4, \{i_8\}:4, \{i_6\}:3, \{i_2\}:2, \{i_4\}:2, \{i_5\}:2 \}}$$ (7 tập)

**Lần lặp 2 ($C_2 \to L_2$):**
Đếm các cặp trong CSDL, chỉ giữ cặp có count $\ge 2$:
$$\mathbf{L_2 = \begin{cases}
\{i_1, i_7\}: 4, & \{i_1, i_8\}: 3, & \{i_7, i_8\}: 3, \\
\{i_1, i_2\}: 2, & \{i_1, i_6\}: 2, & \{i_2, i_6\}: 2, \\
\{i_2, i_7\}: 2, & \{i_4, i_5\}: 2, & \{i_6, i_7\}: 2, & \{i_6, i_8\}: 2
\end{cases}} \quad \text{(10 tập)}$$
*(Các cặp khác như $\{i_1, i_4\}:1, \{i_1, i_5\}:1, \{i_4, i_6\}:1, \{i_4, i_8\}:1, \{i_5, i_6\}:1, \{i_5, i_8\}:1$ đều bị loại).*

**Lần lặp 3 ($C_3 \to L_3$):**
Sinh các bộ 3 và kiểm tra:
- $\{i_1, i_7, i_8\}$: xuất hiện ở $o_1, o_2, o_4 \implies \text{count} = 3 \ge 2 \to \text{Đạt}$
- $\{i_1, i_2, i_6\}$: xuất hiện ở $o_2, o_3 \implies \text{count} = 2 \ge 2 \to \text{Đạt}$
- $\{i_1, i_2, i_7\}$: xuất hiện ở $o_2, o_3 \implies \text{count} = 2 \ge 2 \to \text{Đạt}$
- $\{i_1, i_6, i_7\}$: xuất hiện ở $o_2, o_3 \implies \text{count} = 2 \ge 2 \to \text{Đạt}$
- $\{i_2, i_6, i_7\}$: xuất hiện ở $o_2, o_3 \implies \text{count} = 2 \ge 2 \to \text{Đạt}$
- Các bộ 3 chứa $i_8$ với $\{i_2, i_6\}$ chỉ có $o_2 \implies \text{count} = 1 \to \text{Loại}$.
$$\mathbf{L_3 = \{ \{i_1, i_7, i_8\}: 3, \{i_1, i_2, i_6\}: 2, \{i_1, i_2, i_7\}: 2, \{i_1, i_6, i_7\}: 2, \{i_2, i_6, i_7\}: 2 \}} \quad \text{(5 tập)}$$

**Lần lặp 4 ($C_4 \to L_4$):**
Ghép các tập 3 phần tử tạo thành $\{i_1, i_2, i_6, i_7\}$:
- Kiểm tra tập con: cả 4 tập con 3 phần tử đều thuộc $L_3$.
- Đếm trong CSDL: xuất hiện ở $o_2, o_3 \implies \text{count} = 2 \ge 2 \to \text{Đạt}$.
$$\mathbf{L_4 = \{ \{i_1, i_2, i_6, i_7\}: 2 \}} \quad \text{(1 tập)}$$

**Lần lặp 5:** $C_5 = \emptyset \implies$ Dừng.

**Tổng số tập phổ biến:** $7 + 10 + 5 + 1 = \mathbf{23 \text{ tập}}$.

---

### c. Tìm tất cả các tập phổ biến tối đại (Maximal Frequent Itemsets)
Tập phổ biến tối đại là tập phổ biến mà không có tập cha nào cũng phổ biến:
1. **$\{i_1, i_2, i_6, i_7\}$ (count = 2):** Là tập 4 phần tử duy nhất, không có tập cha phổ biến $\implies$ **Tối đại**.  
   *(Nó bao hàm các tập $\{i_1, i_2, i_6\}, \{i_1, i_2, i_7\}, \{i_1, i_6, i_7\}, \{i_2, i_6, i_7\}$ và các tập 2, 1 phần tử tương ứng).*
2. **$\{i_1, i_7, i_8\}$ (count = 3):** Không nằm trong $\{i_1, i_2, i_6, i_7\}$ vì chứa $i_8$. Không có tập cha 4 phần tử nào chứa nó phổ biến $\implies$ **Tối đại**.
3. **$\{i_4, i_5\}$ (count = 2):** Không thể ghép thêm phần tử nào để tạo thành tập phổ biến 3 phần tử $\implies$ **Tối đại**.
4. **$\{i_6, i_8\}$ (count = 2):** Không nằm trong $\{i_1, i_2, i_6, i_7\}$ (vì chứa $i_8$) và không nằm trong $\{i_1, i_7, i_8\}$ (vì chứa $i_6$). Tập $\{i_1, i_6, i_8\}$ chỉ có count=1 $\implies$ **Tối đại**.

$$\mathbf{\text{Maximal} = \{ \{i_1, i_2, i_6, i_7\}, \{i_1, i_7, i_8\}, \{i_4, i_5\}, \{i_6, i_8\} \}}$$

---

### d. Tìm tất cả các luật kết hợp hợp lệ từ các tập phổ biến tối đại của câu 2c (minconf = 1.0)
Một luật $s \Rightarrow (M \setminus s)$ hợp lệ khi $\text{conf} = \frac{\text{count}(M)}{\text{count}(s)} = 1 \iff \mathbf{\text{count}(s) = \text{count}(M)}$.

#### Nhóm 1: Từ tập $\{i_1, i_2, i_6, i_7\}$ ($\text{count} = 2$)
Cần tìm các tập con $s \subset \{i_1, i_2, i_6, i_7\}$ có $\text{count}(s) = 2$:
- Các tập con có count = 2:
  - 1 phần tử: $\{i_2\}$ (count=2).
  - 2 phần tử: $\{i_1, i_2\}, \{i_1, i_6\}, \{i_2, i_6\}, \{i_2, i_7\}, \{i_6, i_7\}$ (đều có count=2). *(Loại $\{i_1, i_7\}$ vì count=4 $\to$ conf = 2/4 = 0.5).*
  - 3 phần tử: $\{i_1, i_2, i_6\}, \{i_1, i_2, i_7\}, \{i_1, i_6, i_7\}, \{i_2, i_6, i_7\}$ (đều có count=2).

Các luật hợp lệ đạt $\text{conf} = 1.0$:
1. $\{i_2\} \Rightarrow \{i_1, i_6, i_7\}$
2. $\{i_1, i_2\} \Rightarrow \{i_6, i_7\}$
3. $\{i_1, i_6\} \Rightarrow \{i_2, i_7\}$
4. $\{i_2, i_6\} \Rightarrow \{i_1, i_7\}$
5. $\{i_2, i_7\} \Rightarrow \{i_1, i_6\}$
6. $\{i_6, i_7\} \Rightarrow \{i_1, i_2\}$
7. $\{i_1, i_2, i_6\} \Rightarrow \{i_7\}$
8. $\{i_1, i_2, i_7\} \Rightarrow \{i_6\}$
9. $\{i_1, i_6, i_7\} \Rightarrow \{i_2\}$
10. $\{i_2, i_6, i_7\} \Rightarrow \{i_1\}$

#### Nhóm 2: Từ tập $\{i_1, i_7, i_8\}$ ($\text{count} = 3$)
Cần $\text{count}(s) = 3$:
- $\{i_1\}: 5 \implies \text{conf} = 3/5 = 0.6$ (loại).
- $\{i_7\}: 4, \{i_8\}: 4 \implies \text{conf} = 3/4 = 0.75$ (loại).
- $\{i_1, i_7\}: 4 \implies \text{conf} = 3/4 = 0.75$ (loại).
- $\{i_1, i_8\}: 3 \implies \text{conf} = 3/3 = \mathbf{1.0}$ (HỢP LỆ).
- $\{i_7, i_8\}: 3 \implies \text{conf} = 3/3 = \mathbf{1.0}$ (HỢP LỆ).

Các luật hợp lệ:
11. $\{i_1, i_8\} \Rightarrow \{i_7\}$
12. $\{i_7, i_8\} \Rightarrow \{i_1\}$

#### Nhóm 3: Từ tập $\{i_4, i_5\}$ ($\text{count} = 2$)
- $\text{count}(i_4) = 2 \implies \text{conf} = 2/2 = \mathbf{1.0}$.
- $\text{count}(i_5) = 2 \implies \text{conf} = 2/2 = \mathbf{1.0}$.

Các luật hợp lệ:
13. $\{i_4\} \Rightarrow \{i_5\}$
14. $\{i_5\} \Rightarrow \{i_4\}$

#### Nhóm 4: Từ tập $\{i_6, i_8\}$ ($\text{count} = 2$)
- $\text{count}(i_6) = 3 \implies \text{conf} = 2/3 = 0.67$ (loại).
- $\text{count}(i_8) = 4 \implies \text{conf} = 2/4 = 0.50$ (loại).
- $\implies$ Không có luật nào đạt $\text{conf} = 1.0$.

**Tổng kết:** Có **14 luật kết hợp hợp lệ** sinh ra từ các tập phổ biến tối đại.

---

### e. Suy nghĩ về thuật toán tìm tập phổ biến tối đại
- **Vấn đề của Apriori thông thường:** Để tìm được tập phổ biến kích thước $k$, Apriori phải sinh và kiểm tra toàn bộ $2^k - 2$ tập con của nó ở các mức $L_1, L_2, \dots, L_{k-1}$. Khi $k$ lớn, chi phí sinh ứng viên và duyệt CSDL bùng nổ theo hàm mũ.
- **Ý tưởng cải tiến (Thuật toán tìm tập tối đại chuyên dụng, ví dụ MaxMiner, Pincer-Search, GenMax):**
  1. **Tìm kiếm kết hợp từ trên xuống (Top-down) và từ dưới lên (Bottom-up):** Không chỉ duyệt từ $k=1$ lên, mà song song kiểm tra các tập kích thước lớn từ trên xuống. Ngay khi phát hiện một tập lớn là phổ biến, lập tức **loại bỏ (tỉa) toàn bộ tập con của nó** khỏi không gian tìm kiếm vì chắc chắn chúng không thể là tối đại.
  2. **Tỉa dựa trên chặn trên (Lookahead Pruning):** Tại mỗi nút duyệt, dự đoán chặn trên của support của các nhánh con. Nếu chặn trên < minsup thì cắt bỏ toàn bộ nhánh.
  3. **Biểu diễn dữ liệu dạng dọc (Vertical / Bitset / Diffset):** Dùng TID-list để tính giao (intersection) nhanh bằng các phép toán bitwise AND thay vì quét lại toàn bộ bảng CSDL.
  4. **Ưu điểm:** Giảm đáng kể kích thước tập kết quả (trong bài này từ 23 tập phổ biến chỉ còn 4 tập tối đại), tiết kiệm bộ nhớ và thời gian tính toán mà vẫn lưu giữ được biên (boundary) của không gian tập phổ biến.

---

<a name="bai-3"></a>
## BÀI 3

### Đề bài
Dữ liệu giống Bài 1:

| TID | Giao dịch |
|:---:|:---|
| 100 | ACDI |
| 200 | ACI |
| 300 | CEI |
| 400 | ABDE |
| 500 | BDI |
| 600 | ABDE |

$\text{minsup} = 0.3$, $\text{minconf} = 1.0$.
- Câu a: Dùng FP-growth tìm tập phổ biến.
- Câu b: Dùng Apriori tìm tập phổ biến.
- Câu c: Tìm luật kết hợp từ tập phổ biến có nhiều phần tử nhất.

### Lời giải
Vì bộ dữ liệu và ngưỡng **hoàn toàn trùng lặp với Bài 1**, lời giải chi tiết của Bài 3 được thực hiện theo đúng các bước:
1. **Câu a (FP-growth):** Trình bày toàn bộ **Mục b của Bài 1** (Header table $\to$ FP-tree $\to$ bảng Conditional Pattern Base $\to$ kết luận 22 tập phổ biến).
2. **Câu b (Apriori):** Trình bày toàn bộ **Mục a của Bài 1** ($C_1 \to L_1 \to C_2 \to L_2 \to C_3 \to L_3 \to C_4 \to L_4$).
3. **Câu c (Luật kết hợp):** Trình bày toàn bộ **Mục c của Bài 1** (8 luật kết hợp từ tập $\{A, B, D, E\}$).

---

<a name="bai-4"></a>
## BÀI 4

### Đề bài
Cho CSDL giao dịch dạng nhị phân:

| TID | D | E | T | H | I | M |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| T100 | 1 | 0 | 1 | 1 | 0 | 1 |
| T200 | 1 | 0 | 1 | 0 | 0 | 1 |
| T300 | 0 | 0 | 1 | 0 | 1 | 1 |
| T400 | 1 | 1 | 0 | 1 | 1 | 0 |
| T500 | 0 | 1 | 0 | 1 | 0 | 1 |
| T600 | 1 | 1 | 0 | 1 | 1 | 1 |

Ngưỡng: $\text{minsup} = 60\%$, $\text{minconf} = 100\%$.

### Bước chuẩn bị: Đổi bảng nhị phân sang dạng tập giao tác
- T100 = {D, T, H, M}
- T200 = {D, T, M}
- T300 = {T, I, M}
- T400 = {D, E, H, I}
- T500 = {E, H, M}
- T600 = {D, E, H, I, M}

Số giao dịch $N = 6$.  
$\text{mincount} = 60\% \times 6 = 3.6 \implies \mathbf{\text{count} \ge 4}$.  
$\text{minconf} = 100\% = 1.0$.

---

### a) Thuật toán Apriori
1. **Tìm $L_1$:**
   - Đếm tần số xuất hiện của từng mặt hàng:
     - D: xuất hiện ở T100, T200, T400, T600 $\implies \text{count} = 4 \ge 4 \to \text{Đạt}$
     - E: xuất hiện ở T400, T500, T600 $\implies \text{count} = 3 < 4 \to \text{Loại}$
     - T: xuất hiện ở T100, T200, T300 $\implies \text{count} = 3 < 4 \to \text{Loại}$
     - H: xuất hiện ở T100, T400, T500, T600 $\implies \text{count} = 4 \ge 4 \to \text{Đạt}$
     - I: xuất hiện ở T300, T400, T600 $\implies \text{count} = 3 < 4 \to \text{Loại}$
     - M: xuất hiện ở T100, T200, T300, T500, T600 $\implies \text{count} = 5 \ge 4 \to \text{Đạt}$

   $$\mathbf{L_1 = \{ \{M\}: 5, \{D\}: 4, \{H\}: 4 \}}$$

2. **Sinh $C_2$ và tìm $L_2$:**
   - Tổ hợp 2 phần tử từ $L_1$: $\{D, H\}, \{D, M\}, \{H, M\}$.
   - Đếm support trong CSDL:
     - $\{D, H\}$: xuất hiện ở T100, T400, T600 $\implies \text{count} = 3 < 4 \to \text{Loại}$
     - $\{D, M\}$: xuất hiện ở T100, T200, T600 $\implies \text{count} = 3 < 4 \to \text{Loại}$
     - $\{H, M\}$: xuất hiện ở T100, T500, T600 $\implies \text{count} = 3 < 4 \to \text{Loại}$

   $$\mathbf{L_2 = \emptyset}$$

3. **Dừng thuật toán.**  
   **Kết luận câu 4a:** Chỉ có **3 tập phổ biến** gồm 1 phần tử:
   $$\mathbf{L = \{ \{M\}: 5, \{D\}: 4, \{H\}: 4 \}}$$

---

### b) Thuật toán FP-growth
1. **Header Table (sắp giảm dần):** $M:5, D:4, H:4$.
2. **Chiếu các giao dịch theo Header Table (loại bỏ E, T, I):**
   - T100: $\{M, D, H\} \to M, D, H$
   - T200: $\{M, D\} \to M, D$
   - T300: $\{M\} \to M$
   - T400: $\{D, H\} \to D, H$
   - T500: $\{M, H\} \to M, H$
   - T600: $\{M, D, H\} \to M, D, H$
3. **Cây FP-tree gốc:**
   ```text
            null
           /    \
         M:5    D:1
        /   \     \
      D:3   H:1   H:1
       |
      H:2
   ```
4. **Khai phá:**
   - Xét **H** (support = 4):
     - Mẫu điều kiện: $\{M, D\}: 2, \{M\}: 1, \{D\}: 1$.
     - Tổng tần số trong cơ sở mẫu điều kiện: $M$ xuất hiện $2+1 = 3 < 4$, $D$ xuất hiện $2+1 = 3 < 4$.
     - FP-tree điều kiện rỗng $\implies$ Không sinh tập phổ biến 2 phần tử mới.
   - Xét **D** (support = 4):
     - Mẫu điều kiện: $\{M\}: 3$. Tần số $M = 3 < 4 \implies$ Không sinh tập mới.
   - Xét **M** (support = 5):
     - Không có mẫu điều kiện.

   **Kết luận câu 4b:** Tập phổ biến gồm: $\mathbf{\{M\}: 5, \{D\}: 4, \{H\}: 4}$.

---

### c) Tìm tất cả các luật kết hợp từ tập phổ biến có nhiều phần tử nhất
- Tập phổ biến có nhiều phần tử nhất chỉ có **1 phần tử** (kích thước $k = 1$, gồm $\{M\}, \{D\}, \{H\}$).
- Định nghĩa luật kết hợp $A \Rightarrow B$ yêu cầu $A \neq \emptyset$ và $B \neq \emptyset$, tức là tập sinh luật phải có ít nhất 2 phần tử ($k \ge 2$).
- **Kết luận:** **Không tồn tại luật kết hợp nào** thỏa mãn ngưỡng đề bài.

---

<a name="bai-5"></a>
## BÀI 5

### Đề bài
Cho CSDL nhị phân 10 giao dịch:

| STT | Lạc | Vừng | Đỗ | Bánh mì | Bơ | Gạo | Lúa mì |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | Không | Có | Không | Có | Có | Không | Không |
| 2 | Có | Không | Có | Có | Có | Không | Không |
| 3 | Không | Có | Không | Có | Có | Không | Không |
| 4 | Có | Có | Không | Có | Có | Không | Không |
| 5 | Không | Không | Có | Không | Không | Không | Không |
| 6 | Không | Không | Không | Không | Có | Không | Không |
| 7 | Không | Không | Không | Có | Không | Không | Không |
| 8 | Không | Không | Không | Không | Không | Không | Có |
| 9 | Không | Không | Không | Không | Không | Có | Có |
| 10| Không | Không | Không | Không | Không | Có | Không |

> *Ghi chú quan trọng:* Đề bài 5 trong file PDF **bị khuyết ngưỡng minsup và minconf**. Dưới đây giải chi tiết theo ngưỡng tiêu chuẩn của toàn bộ chuyên đề: **$\text{minsup} = 0.3$ (30%), $\text{minconf} = 1.0$ (100%)** (và cung cấp thêm kết quả đối chiếu nhanh cho trường hợp $\text{minsup} = 0.4$).

### Bước chuẩn bị: Chuyển đổi dữ liệu sang dạng giao tác
Quy ước: 1 = Có, 0 = Không.
- T1 = {Bánh mì, Bơ, Vừng}
- T2 = {Bánh mì, Bơ, Đỗ, Lạc}
- T3 = {Bánh mì, Bơ, Vừng}
- T4 = {Bánh mì, Bơ, Lạc, Vừng}
- T5 = {Đỗ}
- T6 = {Bơ}
- T7 = {Bánh mì}
- T8 = {Lúa mì}
- T9 = {Gạo, Lúa mì}
- T10 = {Gạo}

Tổng số giao dịch $N = 10$.  
Với $\text{minsup} = 0.3 \implies \text{mincount} = 0.3 \times 10 = \mathbf{3}$.  
$\text{minconf} = 1.0$ (100%).

---

### a) Thuật toán Apriori (với minsup = 0.3)
1. **Tìm $L_1$:**
   Đếm số lần xuất hiện của từng mặt hàng:
   - Bánh mì: 5 (T1, T2, T3, T4, T7) $\ge 3 \to \text{Đạt}$
   - Bơ: 5 (T1, T2, T3, T4, T6) $\ge 3 \to \text{Đạt}$
   - Vừng: 3 (T1, T3, T4) $\ge 3 \to \text{Đạt}$
   - Lạc: 2 (T2, T4) $< 3 \to \text{Loại}$
   - Đỗ: 2 (T2, T5) $< 3 \to \text{Loại}$
   - Gạo: 2 (T9, T10) $< 3 \to \text{Loại}$
   - Lúa mì: 2 (T8, T9) $< 3 \to \text{Loại}$

   $$\mathbf{L_1 = \{ \{\text{Bánh mì}\}: 5, \{\text{Bơ}\}: 5, \{\text{Vừng}\}: 3 \}}$$

2. **Tìm $L_2$:**
   Các cặp ứng viên từ $L_1$:
   - {Bánh mì, Bơ}: xuất hiện ở T1, T2, T3, T4 $\implies \text{count} = 4 \ge 3 \to \text{Đạt}$
   - {Bánh mì, Vừng}: xuất hiện ở T1, T3, T4 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {Bơ, Vừng}: xuất hiện ở T1, T3, T4 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$

   $$\mathbf{L_2 = \{ \{\text{Bánh mì, Bơ}\}: 4, \{\text{Bánh mì, Vừng}\}: 3, \{\text{Bơ, Vừng}\}: 3 \}}$$

3. **Tìm $L_3$:**
   Ứng viên duy nhất: {Bánh mì, Bơ, Vừng}.
   - Xuất hiện ở T1, T3, T4 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$.
   $$\mathbf{L_3 = \{ \{\text{Bánh mì, Bơ, Vừng}\}: 3 \}}$$

4. **Dừng thuật toán.** Tổng cộng có **7 tập phổ biến**.

*(Nếu xét $\text{minsup} = 0.4 \implies \text{count} \ge 4$: khi đó $L_1 = \{\text{Bánh mì}:5, \text{Bơ}:5\}$, $L_2 = \{\{\text{Bánh mì, Bơ}\}:4\}$, $L_3 = \emptyset$).*

---

### b) Thuật toán FP-growth (với minsup = 0.3)
1. **Header Table:** Bánh mì: 5, Bơ: 5, Vừng: 3.
2. **Chiếu giao dịch:**
   - T1, T3, T4: Bánh mì, Bơ, Vừng (3 lần)
   - T2: Bánh mì, Bơ (1 lần)
   - T6: Bơ (1 lần)
   - T7: Bánh mì (1 lần)
3. **Cây FP-tree:**
   ```text
            null
           /    \
     Bánh mì:5   Bơ:1
        |
       Bơ:4
        |
      Vừng:3
   ```
4. **Khai phá:**
   - **Vừng (count=3):** Nhánh tiền tố là đường đơn $\langle \text{Bánh mì}:3, \text{Bơ}:3 \rangle$.  
     $\implies$ Sinh ra: **{Bánh mì, Vừng}:3, {Bơ, Vừng}:3, {Bánh mì, Bơ, Vừng}:3**.
   - **Bơ (count=5):** Nhánh tiền tố là đường đơn $\langle \text{Bánh mì}:4 \rangle$.  
     $\implies$ Sinh ra: **{Bánh mì, Bơ}:4**.
   - **Bánh mì (count=5):** Không có tiền tố.

Kết quả hoàn toàn khớp với Apriori (7 tập phổ biến).

---

### c) Tìm tất cả các luật kết hợp từ tập phổ biến có nhiều phần tử nhất
Tập nhiều phần tử nhất: **$X = \{\text{Bánh mì, Bơ, Vừng}\}$** với $\text{count}(X) = 3$.  
Ngưỡng $\text{minconf} = 1.0 \implies$ chỉ nhận các luật có vế trái $s$ thỏa mãn $\text{count}(s) = \text{count}(X) = 3$.

Xét tất cả 6 luật:
1. $\{\text{Bánh mì}\} \Rightarrow \{\text{Bơ, Vừng}\}$: $\text{conf} = 3/5 = 0.60$ (Loại)
2. $\{\text{Bơ}\} \Rightarrow \{\text{Bánh mì, Vừng}\}$: $\text{conf} = 3/5 = 0.60$ (Loại)
3. **$\{\text{Vừng}\} \Rightarrow \{\text{Bánh mì, Bơ}\}$:** $\text{conf} = 3/3 = \mathbf{1.00}$ $\implies$ **HỢP LỆ**
4. $\{\text{Bánh mì, Bơ}\} \Rightarrow \{\text{Vừng}\}$: $\text{conf} = 3/4 = 0.75$ (Loại)
5. **$\{\text{Bánh mì, Vừng}\} \Rightarrow \{\text{Bơ}\}$:** $\text{conf} = 3/3 = \mathbf{1.00}$ $\implies$ **HỢP LỆ**
6. **$\{\text{Bơ, Vừng}\} \Rightarrow \{\text{Bánh mì}\}$:** $\text{conf} = 3/3 = \mathbf{1.00}$ $\implies$ **HỢP LỆ**

**Kết luận:** Có **3 luật kết hợp hợp lệ** đạt độ tin cậy 100%:
- Vừng $\Rightarrow$ Bánh mì, Bơ
- Bánh mì, Vừng $\Rightarrow$ Bơ
- Bơ, Vừng $\Rightarrow$ Bánh mì

---

<a name="bai-6"></a>
## BÀI 6

### Đề bài
Cho CSDL sau với ngưỡng $\text{MinSup} = 3$ (ngưỡng đếm trực tiếp, không nhân $N$), $\text{MinConf} = 1.0$:

| TID | Nội dung giao dịch |
|:---:|:---|
| 100 | f, a, b, d, g, i, m, p |
| 200 | a, b, c, f, l, m, o |
| 300 | a, a, h, j, o *(lưu ý: mặt hàng 'a' lặp lại $\to$ coi là tập {a, h, j, o})* |
| 400 | b, c, k, s, p |
| 500 | a, f, b, c, l, p, m, n |

---

### a) Tìm tập phổ biến bằng thuật toán Apriori ($\text{mincount} = 3$)
1. **Tìm $L_1$:**
   Đếm tần số từng mặt hàng (mỗi giao dịch chỉ tính 1 lần):
   - a: 4 (T100, T200, T300, T500) $\ge 3 \to \text{Đạt}$
   - b: 4 (T100, T200, T400, T500) $\ge 3 \to \text{Đạt}$
   - c: 3 (T200, T400, T500) $\ge 3 \to \text{Đạt}$
   - f: 3 (T100, T200, T500) $\ge 3 \to \text{Đạt}$
   - m: 3 (T100, T200, T500) $\ge 3 \to \text{Đạt}$
   - p: 3 (T100, T400, T500) $\ge 3 \to \text{Đạt}$
   - Các mặt hàng $d, g, i, h, j, o, k, s, l, n$ đều có count $\le 2 < 3 \implies$ Loại.

   $$\mathbf{L_1 = \{ \{a\}: 4, \{b\}: 4, \{c\}: 3, \{f\}: 3, \{m\}: 3, \{p\}: 3 \}}$$

2. **Tìm $L_2$:**
   Tổ hợp các cặp từ $L_1$ và đếm trong CSDL:
   - {a, b}: xuất hiện ở T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {a, c}: T200, T500 $\implies 2 < 3 \to$ Loại
   - {a, f}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {a, m}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {a, p}: T100, T500 $\implies 2 < 3 \to$ Loại
   - {b, c}: T200, T400, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {b, f}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {b, m}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {b, p}: T100, T400, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {c, f}: T200, T500 $\implies 2 < 3 \to$ Loại
   - {c, m}: T200, T500 $\implies 2 < 3 \to$ Loại
   - {c, p}: T400, T500 $\implies 2 < 3 \to$ Loại
   - {f, m}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - {f, p}: T100, T500 $\implies 2 < 3 \to$ Loại
   - {m, p}: T100, T500 $\implies 2 < 3 \to$ Loại

   $$\mathbf{L_2 = \{ \{a,b\}:3, \{a,f\}:3, \{a,m\}:3, \{b,c\}:3, \{b,f\}:3, \{b,m\}:3, \{b,p\}:3, \{f,m\}:3 \}} \quad \text{(8 tập)}$$

3. **Tìm $L_3$:**
   Ghép các cặp trong $L_2$:
   - Nhóm $\{a, b, f, m\}$: mọi tập con 2 phần tử ({a,b}, {a,f}, {a,m}, {b,f}, {b,m}, {f,m}) đều thuộc $L_2$!
   - Đếm trên CSDL:
     - {a, b, f}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
     - {a, b, m}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
     - {a, f, m}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
     - {b, f, m}: T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$
   - Các bộ 3 chứa $c$ hoặc $p$: ví dụ $\{b, c, p\}$ chỉ có ở T400, T500 (count=2 < 3 $\to$ Loại).

   $$\mathbf{L_3 = \{ \{a,b,f\}: 3, \{a,b,m\}: 3, \{a,f,m\}: 3, \{b,f,m\}: 3 \}} \quad \text{(4 tập)}$$

4. **Tìm $L_4$:**
   Ứng viên duy nhất: $\{a, b, f, m\}$.
   - Cả 4 tập con 3 phần tử đều thuộc $L_3$.
   - Xuất hiện ở T100, T200, T500 $\implies \text{count} = 3 \ge 3 \to \text{Đạt}$.
   $$\mathbf{L_4 = \{ \{a, b, f, m\}: 3 \}}$$

5. **Dừng.** Tổng cộng có **19 tập phổ biến**:
   - 1 phần tử (6): {a}:4, {b}:4, {c}:3, {f}:3, {m}:3, {p}:3
   - 2 phần tử (8): {a,b}:3, {a,f}:3, {a,m}:3, {b,c}:3, {b,f}:3, {b,m}:3, {b,p}:3, {f,m}:3
   - 3 phần tử (4): {a,b,f}:3, {a,b,m}:3, {a,f,m}:3, {b,f,m}:3
   - 4 phần tử (1): {a,b,f,m}:3

---

### b) Tìm tập phổ biến bằng thuật toán FP-growth
1. **Header Table (sắp giảm dần):** $a:4, b:4, c:3, f:3, m:3, p:3$.
2. **Chiếu giao dịch:**
   - T100: $a, b, f, m, p$
   - T200: $a, b, c, f, m$
   - T300: $a$
   - T400: $b, c, p$
   - T500: $a, b, c, f, m, p$
3. **Cây FP-tree:**
   ```text
            null
           /    \
         a:4    b:1
          |      |
         b:3    c:1
        /   \    |
      f:1   c:2 p:1
       |     |
      m:1   f:2
       |     |
      p:1   m:2
             |
            p:1
   ```
4. **Khai phá các nhánh:**
   - **p (count=3):** Các nhánh tiền tố $\{a,b,f,m\}:1, \{a,b,c,f,m\}:1, \{b,c\}:1$. Gộp lại: $b:3$ (đạt), $a:2, c:2, f:2, m:2$ (loại).  
     $\implies$ Sinh ra: **{b, p}: 3**.
   - **m (count=3):** Tiền tố $\{a,b,f\}:1, \{a,b,c,f\}:2$. Gộp lại: $a:3, b:3, f:3$ (đạt), $c:2$ (loại).  
     $\implies$ Nhánh điều kiện $\langle a:3, b:3, f:3 \rangle$ sinh ra: **{a,m}:3, {b,m}:3, {f,m}:3, {a,b,m}:3, {a,f,m}:3, {b,f,m}:3, {a,b,f,m}:3**.
   - **f (count=3):** Tiền tố $\{a,b\}:1, \{a,b,c\}:2$. Gộp lại: $a:3, b:3$ (đạt), $c:2$ (loại).  
     $\implies$ Sinh ra: **{a,f}:3, {b,f}:3, {a,b,f}:3**.
   - **c (count=3):** Tiền tố $\{a,b\}:2, \{b\}:1$. Gộp lại: $b:3$ (đạt), $a:2$ (loại).  
     $\implies$ Sinh ra: **{b,c}: 3**.
   - **b (count=4):** Tiền tố $\{a\}:3$.  
     $\implies$ Sinh ra: **{a,b}: 3**.
   - **a (count=4):** Không có tiền tố.

Kết quả hoàn toàn khớp với câu 6a.

---

### c) Tìm các luật kết hợp có ngưỡng MinSup = 3 và MinConf = 1.0
Để luật $A \Rightarrow B$ có $\text{conf} = 1.0$, ta cần $\text{count}(A \cup B) = \text{count}(A) = 3$.  
Nghĩa là vế trái $A$ phải là một tập phổ biến có **$\text{count}(A) = 3$**.

Duyệt trên toàn bộ các tập phổ biến $\ge 2$ phần tử có count = 3:
- Từ các tập 2 phần tử có count = 3:
  - $\{a, f\} \Rightarrow \text{conf}(f \to a) = 3/3 = 1.0 \implies \mathbf{f \Rightarrow a}$
  - $\{a, m\} \implies \mathbf{m \Rightarrow a}$
  - $\{b, c\} \implies \mathbf{c \Rightarrow b}$
  - $\{b, f\} \implies \mathbf{f \Rightarrow b}$
  - $\{b, m\} \implies \mathbf{m \Rightarrow b}$
  - $\{b, p\} \implies \mathbf{p \Rightarrow b}$
  - $\{f, m\} \implies \mathbf{f \Rightarrow m, m \Rightarrow f}$
  - $\{a, b\} \implies \text{conf} = 3/4$ (loại cả 2 chiều)

- Từ các tập 3 phần tử $\{a,b,f\}, \{a,b,m\}, \{a,f,m\}, \{b,f,m\}$:
  - Bất kỳ vế trái nào có count = 3 đều tạo thành luật hợp lệ:
  - Ví dụ: $\{a, f\} \Rightarrow b$, $\{b, f\} \Rightarrow a$, $\{f\} \Rightarrow \{a, b\}$, $\{m\} \Rightarrow \{a, b\}$, v.v.

- Từ tập 4 phần tử $\{a, b, f, m\}$ (xem chi tiết câu 6d).

---

### d) Tìm tất cả các luật kết hợp từ tập phổ biến có nhiều phần tử nhất với MinConf = 1.0
Tập phổ biến nhiều phần tử nhất là: **$X = \{a, b, f, m\}$** với $\text{count}(X) = 3$.  
Điều kiện luật hợp lệ: $\text{count}(s) = 3$ với mọi $s \subset X, s \neq \emptyset$.

Đánh giá 14 tập con thực sự của $\{a, b, f, m\}$:
- Tập con 1 phần tử:
  - $\text{count}(a) = 4 \implies \text{conf} = 3/4 = 0.75$ (Loại)
  - $\text{count}(b) = 4 \implies \text{conf} = 3/4 = 0.75$ (Loại)
  - $\text{count}(f) = 3 \implies \text{conf} = 3/3 = \mathbf{1.00} \implies \mathbf{\{f\} \Rightarrow \{a, b, m\}}$
  - $\text{count}(m) = 3 \implies \text{conf} = 3/3 = \mathbf{1.00} \implies \mathbf{\{m\} \Rightarrow \{a, b, f\}}$
- Tập con 2 phần tử (cả 6 cặp đều có count = 3):
  - $\mathbf{\{a, b\} \Rightarrow \{f, m\}}$
  - $\mathbf{\{a, f\} \Rightarrow \{b, m\}}$
  - $\mathbf{\{a, m\} \Rightarrow \{b, f\}}$
  - $\mathbf{\{b, f\} \Rightarrow \{a, m\}}$
  - $\mathbf{\{b, m\} \Rightarrow \{a, f\}}$
  - $\mathbf{\{f, m\} \Rightarrow \{a, b\}}$
- Tập con 3 phần tử (cả 4 bộ 3 đều có count = 3):
  - $\mathbf{\{a, b, f\} \Rightarrow \{m\}}$
  - $\mathbf{\{a, b, m\} \Rightarrow \{f\}}$
  - $\mathbf{\{a, f, m\} \Rightarrow \{b\}}$
  - $\mathbf{\{b, f, m\} \Rightarrow \{a\}}$

**Kết luận:** Có **12 luật kết hợp hợp lệ** đạt độ tin cậy 100% từ tập $\{a, b, f, m\}$.

---

<a name="bai-7"></a>
## BÀI 7

### Đề bài
Cho CSDL và $\text{minsup} = 60\%$, $\text{minconf} = 100\%$:

| TID | A | B | C | D | E | F | G | H | K | M | N |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 10 | 0 | 1 | 0 | 0 | 1 | 1 | 1 | 1 | 0 | 0 | 0 |
| 20 | 0 | 0 | 0 | 1 | 0 | 1 | 0 | 0 | 1 | 1 | 0 |
| 30 | 1 | 0 | 1 | 1 | 0 | 1 | 0 | 1 | 0 | 1 | 1 |
| 40 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 1 |
| 50 | 1 | 1 | 0 | 0 | 1 | 0 | 1 | 1 | 0 | 1 | 1 |

### Bước chuẩn bị
- Danh sách giao dịch:
  - TID 10: {B, E, F, G, H}
  - TID 20: {D, F, K, M}
  - TID 30: {A, C, D, F, H, M, N}
  - TID 40: {A, B, H, K, M, N}
  - TID 50: {A, B, E, G, H, M, N}
- Số giao dịch $N = 5$.
- $\text{mincount} = 60\% \times 5 = \mathbf{3.0}$.
- $\text{minconf} = 100\% = 1.0$.

---

### a) Thuật toán Apriori tìm tập phổ biến

#### Lần lặp 1: Tìm $L_1$
| Item | TID chứa | Count | Kết luận ($\ge 3$) |
|:---:|:---|:---:|:---:|
| A | 30, 40, 50 | 3 | Đạt $\to L_1$ |
| B | 10, 40, 50 | 3 | Đạt $\to L_1$ |
| C | 30 | 1 | Loại |
| D | 20, 30 | 2 | Loại |
| E | 10, 50 | 2 | Loại |
| F | 10, 20, 30 | 3 | Đạt $\to L_1$ |
| G | 10, 50 | 2 | Loại |
| H | 10, 30, 40, 50 | 4 | Đạt $\to L_1$ |
| K | 20, 40 | 2 | Loại |
| M | 20, 30, 40, 50 | 4 | Đạt $\to L_1$ |
| N | 30, 40, 50 | 3 | Đạt $\to L_1$ |

$$\mathbf{L_1 = \{ \{H\}: 4, \{M\}: 4, \{A\}: 3, \{B\}: 3, \{F\}: 3, \{N\}: 3 \}}$$

#### Lần lặp 2: Sinh $C_2$ và tìm $L_2$
Tổ hợp các cặp từ 6 phần tử $\{A, B, F, H, M, N\}$:
- {A, B}: TID 40, 50 $\implies$ count = 2 (Loại)
- {A, F}: TID 30 $\implies$ count = 1 (Loại)
- **{A, H}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$
- **{A, M}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$
- **{A, N}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$
- {B, F}: TID 10 $\implies$ count = 1 (Loại)
- **{B, H}:** TID 10, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$
- {B, M}: TID 40, 50 $\implies$ count = 2 (Loại)
- {B, N}: TID 40, 50 $\implies$ count = 2 (Loại)
- {F, H}: TID 10, 30 $\implies$ count = 2 (Loại)
- {F, M}: TID 20, 30 $\implies$ count = 2 (Loại)
- {F, N}: TID 30 $\implies$ count = 1 (Loại)
- **{H, M}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$
- **{H, N}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$
- **{M, N}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_2}$

$$\mathbf{L_2 = \{ \{A,H\}:3, \{A,M\}:3, \{A,N\}:3, \{B,H\}:3, \{H,M\}:3, \{H,N\}:3, \{M,N\}:3 \}} \quad \text{(7 tập)}$$

#### Lần lặp 3: Sinh $C_3$ và tìm $L_3$
Các bộ 3 tạo từ $\{A, H, M, N\}$ có mọi tập con 2 phần tử thuộc $L_2$:
- **{A, H, M}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_3}$
- **{A, H, N}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_3}$
- **{A, M, N}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_3}$
- **{H, M, N}:** TID 30, 40, 50 $\implies$ **count = 3** $\to \mathbf{L_3}$

*(Các bộ 3 chứa B như $\{A, B, H\}$ bị loại vì $\{A, B\} \notin L_2$).*

$$\mathbf{L_3 = \{ \{A,H,M\}:3, \{A,H,N\}:3, \{A,M,N\}:3, \{H,M,N\}:3 \}} \quad \text{(4 tập)}$$

#### Lần lặp 4: Sinh $C_4$ và tìm $L_4$
Ghép 4 tập trong $L_3$ thành ứng viên duy nhất: **$\{A, H, M, N\}$**.
- Cả 4 tập con 3 phần tử đều thuộc $L_3$.
- Đếm trên CSDL: xuất hiện ở TID 30, 40, 50 $\implies$ **count = 3 $\ge 3$**.
$$\mathbf{L_4 = \{ \{A, H, M, N\}: 3 \}}$$

#### Lần lặp 5:
$C_5 = \emptyset \implies$ Dừng.

**Tổng kết:** Có **18 tập phổ biến**:
- 1 phần tử (6): {H}:4, {M}:4, {A}:3, {B}:3, {F}:3, {N}:3
- 2 phần tử (7): {A,H}:3, {A,M}:3, {A,N}:3, {B,H}:3, {H,M}:3, {H,N}:3, {M,N}:3
- 3 phần tử (4): {A,H,M}:3, {A,H,N}:3, {A,M,N}:3, {H,M,N}:3
- 4 phần tử (1): {A,H,M,N}:3

---

### b) Thuật toán FP-growth
1. **Header Table (sắp giảm dần theo tần số):**  
   $$H: 4 \succ M: 4 \succ A: 3 \succ B: 3 \succ F: 3 \succ N: 3$$
2. **Chiếu giao dịch theo Header Table:**
   - TID 10: $\{H, B, F\}$
   - TID 20: $\{M, F\}$
   - TID 30: $\{H, M, A, F, N\}$
   - TID 40: $\{H, M, A, B, N\}$
   - TID 50: $\{H, M, A, B, N\}$
3. **Cây FP-tree gốc:**
   ```text
                  null
                 /    \
               H:4    M:1
              /   \     \
            B:1   M:3   F:1
             |     |
            F:1   A:3
                 /   \
               F:1   B:2
                |     |
               N:1   N:2
   ```
4. **Khai phá các nhánh (xét ngược từ $N$ lên $H$):**
   - **N (count = 3):**
     - Mẫu điều kiện: $\{H, M, A, F\}: 1$, $\{H, M, A, B\}: 2$.
     - Gộp lại: $H:3, M:3, A:3$ (đạt), $B:2, F:1$ (loại).
     - FP-tree điều kiện: đường đơn $\langle H:3, M:3, A:3 \rangle$.
     - $\implies$ Sinh ra các tập phổ biến chứa N:  
       **{H,N}:3, {M,N}:3, {A,N}:3, {H,M,N}:3, {H,A,N}:3, {M,A,N}:3, {H,M,A,N}:3**.
   - **F (count = 3):**
     - Mẫu điều kiện: $\{H, B\}: 1, \{M\}: 1, \{H, M, A\}: 1$.
     - Mỗi mặt hàng chỉ có count $\le 2 < 3 \implies$ Không sinh tập mới.
   - **B (count = 3):**
     - Mẫu điều kiện: $\{H\}: 1, \{H, M, A\}: 2$.
     - $H:3$ (đạt), $M:2, A:2$ (loại).
     - $\implies$ Sinh ra: **{H, B}: 3**.
   - **A (count = 3):**
     - Mẫu điều kiện: $\{H, M\}: 3$.
     - FP-tree điều kiện: đường đơn $\langle H:3, M:3 \rangle$.
     - $\implies$ Sinh ra: **{H, A}: 3, {M, A}: 3, {H, M, A}: 3**.
   - **M (count = 4):**
     - Mẫu điều kiện: $\{H\}: 3$.
     - $\implies$ Sinh ra: **{H, M}: 3**.
   - **H (count = 4):** Không có tiền tố.

Kết quả hoàn toàn trùng khớp với Apriori (18 tập).

---

### c) Tìm tất cả các luật kết hợp từ tập phổ biến có nhiều phần tử nhất
Tập có nhiều phần tử nhất: **$X = \{A, H, M, N\}$** với $\text{count}(X) = 3$.  
Ngưỡng $\text{minconf} = 1.0 \implies \text{count}(s) = \text{count}(X) = 3$.

Đánh giá 14 tập con thực sự của $\{A, H, M, N\}$:
- Tập con 1 phần tử:
  - $\text{count}(A) = 3 \implies \mathbf{\{A\} \Rightarrow \{H, M, N\}}$ (conf = 3/3 = 1.0)
  - $\text{count}(H) = 4 \implies \text{conf} = 3/4 = 0.75$ (Loại)
  - $\text{count}(M) = 4 \implies \text{conf} = 3/4 = 0.75$ (Loại)
  - $\text{count}(N) = 3 \implies \mathbf{\{N\} \Rightarrow \{A, H, M\}}$ (conf = 3/3 = 1.0)
- Tập con 2 phần tử (cả 6 cặp đều có count = 3 $\implies$ đều sinh luật hợp lệ):
  - $\mathbf{\{A, H\} \Rightarrow \{M, N\}}$
  - $\mathbf{\{A, M\} \Rightarrow \{H, N\}}$
  - $\mathbf{\{A, N\} \Rightarrow \{H, M\}}$
  - $\mathbf{\{H, M\} \Rightarrow \{A, N\}}$
  - $\mathbf{\{H, N\} \Rightarrow \{A, M\}}$
  - $\mathbf{\{M, N\} \Rightarrow \{A, H\}}$
- Tập con 3 phần tử (cả 4 bộ 3 đều có count = 3 $\implies$ đều sinh luật hợp lệ):
  - $\mathbf{\{A, H, M\} \Rightarrow \{N\}}$
  - $\mathbf{\{A, H, N\} \Rightarrow \{M\}}$
  - $\mathbf{\{A, M, N\} \Rightarrow \{H\}}$
  - $\mathbf{\{H, M, N\} \Rightarrow \{A\}}$

**Kết luận:** Có **12 luật kết hợp hợp lệ** đạt độ tin cậy 100% từ tập $\{A, H, M, N\}$.

---

<a name="bai-8"></a>
## BÀI 8

### Đề bài
> *Sinh viên làm thực hành trên các bộ dữ liệu lớn lấy từ UCI.*

### Hướng dẫn các bước thực hiện chi tiết

#### 1. Chọn bộ dữ liệu từ UCI Machine Learning Repository
Hai bộ dữ liệu kinh điển và phù hợp nhất cho bài toán khai phá luật kết hợp:
1. **Online Retail Data Set:** Dữ liệu giao dịch thực tế của một cửa hàng bán lẻ trực tuyến tại Anh (gồm hơn 500,000 dòng giao dịch).
2. **Groceries Market Basket Dataset:** Dữ liệu mua hàng siêu thị dạng POS phổ biến cho bài toán Market Basket Analysis.

#### 2. Quy trình tiền xử lý dữ liệu
1. **Làm sạch:** Loại bỏ các bản ghi trùng lặp, giao dịch trả hàng (Invoice bắt đầu bằng 'C'), các hóa đơn thiếu CustomerID hoặc Description.
2. **Chuyển đổi sang ma trận One-hot (hoặc Transaction Encoder):**
   - Gom nhóm theo `InvoiceNo` và tạo bảng nhị phân: dòng là Hóa đơn, cột là Mặt hàng, giá trị 1/0 (True/False).

#### 3. Mã nguồn mẫu bằng Python (`mlxtend`)

```python
import pandas as pd
from mlxtend.frequent_patterns import apriori, fpgrowth, association_rules
import time

# 1. Đọc và tiền xử lý dữ liệu (ví dụ với file retail.csv)
# df = pd.read_csv('Online_Retail.csv')
# basket = (df.groupby(['InvoiceNo', 'Description'])['Quantity']
#             .sum().unstack().reset_index().fillna(0)
#             .set_index('InvoiceNo'))
# basket_sets = basket.map(lambda x: 1 if x >= 1 else 0)

# Dữ liệu giả lập minh họa chạy thử:
dataset = [
    ['Milk', 'Onion', 'Nutmeg', 'Kidney Beans', 'Eggs', 'Yogurt'],
    ['Dill', 'Onion', 'Nutmeg', 'Kidney Beans', 'Eggs', 'Yogurt'],
    ['Milk', 'Apple', 'Kidney Beans', 'Eggs'],
    ['Milk', 'Unicorn', 'Corn', 'Kidney Beans', 'Yogurt'],
    ['Corn', 'Onion', 'Onion', 'Kidney Beans', 'Ice cream', 'Eggs']
]

from mlxtend.preprocessing import TransactionEncoder
te = TransactionEncoder()
te_ary = te.fit(dataset).transform(dataset)
df_trans = pd.DataFrame(te_ary, columns=te.columns_)

minsup = 0.6
minconf = 0.7

# 2. Chạy và đo thời gian Apriori
t0 = time.time()
freq_apriori = apriori(df_trans, min_support=minsup, use_colnames=True)
t_apriori = time.time() - t0
print(f"Apriori chạy trong {t_apriori:.4f}s - Số tập phổ biến: {len(freq_apriori)}")

# 3. Chạy và đo thời gian FP-growth
t0 = time.time()
freq_fpgrowth = fpgrowth(df_trans, min_support=minsup, use_colnames=True)
t_fpgrowth = time.time() - t0
print(f"FP-growth chạy trong {t_fpgrowth:.4f}s - Số tập phổ biến: {len(freq_fpgrowth)}")

# 4. Sinh luật kết hợp
rules = association_rules(freq_fpgrowth, metric="confidence", min_threshold=minconf)
print("\nDanh sách các luật kết hợp tiêu biểu:")
print(rules[['antecedents', 'consequents', 'support', 'confidence', 'lift']].head(10))
```

#### 4. Nhận xét và báo cáo kết quả
- **Tính đúng đắn:** Cả hai thuật toán Apriori và FP-growth luôn trả về **chính xác cùng một tập hợp các tập phổ biến** (số lượng và support của từng tập là như nhau).
- **Hiệu năng:**
  - Trên các bộ dữ liệu lớn với nhiều giao dịch dài, **FP-growth chạy nhanh hơn Apriori từ 5 đến vài chục lần** vì FP-growth chỉ cần 2 lần quét CSDL và nén toàn bộ vào cây nhớ, không phải sinh ứng viên khổng lồ theo hàm mũ như Apriori.
  - Khi `minsup` giảm xuống mức thấp, Apriori có thể bị tràn bộ nhớ hoặc chạy rất lâu, trong khi FP-growth vẫn giữ được hiệu năng ổn định.
- **Ý nghĩa chỉ số Lift:**
  - $\text{Lift}(A \Rightarrow B) = \frac{\text{conf}(A \Rightarrow B)}{\text{supp}(B)}$.
  - $\text{Lift} > 1$: $A$ và $B$ có tương quan thuận (người mua $A$ rất có xu hướng mua thêm $B$).
  - $\text{Lift} = 1$: $A$ và $B$ độc lập.
  - $\text{Lift} < 1$: $A$ và $B$ tương quan nghịch.

---

## BẢNG TỔNG HỢP KẾT QUẢ ĐỐI CHIẾU CÁC BÀI TẬP

| Bài | Số GD ($N$) | Ngưỡng $\text{minsup}$ | Ngưỡng $\text{mincount}$ | Số tập phổ biến ($|L|$) | Tập phổ biến tối đại tiêu biểu | Số luật từ tập lớn nhất ($\text{minconf}=1.0$) |
|:---:|:---:|:---:|:---:|:---:|:---|:---:|
| **1** | 6 | 0.3 | 2 | **22** | {A, B, D, E}, {A, C, I}, {D, I} | **8 luật** |
| **2** | 6 | 0.3 | 2 | **23** | $\{i_1, i_2, i_6, i_7\}, \{i_1, i_7, i_8\}, \{i_4, i_5\}, \{i_6, i_8\}$ | **10 luật** (từ tập 4 pt) |
| **3** | 6 | 0.3 | 2 | **22** | *(Trùng Bài 1)* | **8 luật** |
| **4** | 6 | 60% | 4 | **3** | {M}, {D}, {H} | **0 luật** |
| **5** | 10 | 0.3 | 3 | **7** | {Bánh mì, Bơ, Vừng} | **3 luật** |
| **6** | 5 | $\text{count}=3$ | 3 | **19** | {a, b, f, m}, {b, c}, {b, p} | **12 luật** |
| **7** | 5 | 60% | 3 | **18** | {A, H, M, N}, {B, H}, {F} | **12 luật** |
