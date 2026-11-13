**BÀI TẬP THỰC HÀNH TỔNG HỢP: HỆ THỐNG XỬ LÝ ĐƠN HÀNG VÀ BẢO HÀNH TỰ ĐỘNG TECHSHOP**

---

### PHẦN 1. BÁO CÁO THIẾT KẾ GIẢI PHÁP (PROMPT CHAINING)

Để giải quyết bài toán tự động hóa quy trình tiếp nhận thông tin từ khách hàng và đưa vào hệ thống ERP mà không bị lỗi cú pháp, hệ thống sử dụng mô hình **Chuỗi Prompt liên kết (Prompt Chaining)** gồm 3 bước tuần tự:

1. **Bước 1: Chuẩn hóa & Làm sạch dữ liệu thô (Text Cleaning)**
* *Nhiệm vụ:* Tiếp nhận phản hồi hỗn độn của khách hàng (có thể kèm theo cảm xúc giận dữ, lời phàn nàn, văn phong không chuẩn), lọc bỏ nhiễu và trích xuất thành ý chính.


2. **Bước 2: Phân loại & Điều hướng nghiệp vụ (Routing)**
* *Nhiệm vụ:* Đọc dữ liệu đã sạch từ Bước 1 để phân loại yêu cầu (Bảo hành, Đổi trả, hay Khiếu nại giao chậm) nhằm chọn kịch bản phản hồi phù hợp.


3. **Bước 3: Trích xuất JSON nghiêm ngặt cho Backend (Structured Output)**
* *Nhiệm vụ:* Ép toàn bộ kết quả thành chuỗi JSON thuần túy (không lời chào, không Markdown block) để nạp thẳng vào Database của hệ thống ERP.



---

### PHẦN 2. HỆ THỐNG PROMPT THỰC THI (3 BƯỚC)

#### Bước 1. Prompt làm sạch dữ liệu thô

```text
[CONTEXT] Bạn là bộ lọc tiền xử lý dữ liệu cho hệ thống CSKH TechShop.
[REQUEST] Đọc văn bản thô từ khách hàng, loại bỏ các từ ngữ cảm xúc thừa (chửi bới, than vãn) và tóm tắt lại nội dung cốt lõi gồm: Tên khách, Tên sản phẩm, Vấn đề gặp phải.
[ADJUSTMENT] Không bịa đặt thông tin. Nếu thiếu thông tin nào, điền là "Không rõ".
[TYPE OF OUTPUT] Trợ lý liệt kê ngắn gọn theo dạng gạch đầu dòng.

```

#### Bước 2. Prompt phân loại nghiệp vụ (Áp dụng Few-shot)

```text
[CONTEXT] Bạn là bộ phân loại tự động của TechShop.
[REQUEST] Đọc thông tin đã tóm tắt từ Bước 1 và phân loại vào 1 trong 3 nhóm: "BAO_HANH", "DOI_TRA", "GIAO_CHAM".

[EXAMPLES]
- Input: Lỗi bàn phím liệt nút Space -> Output: BAO_HANH
- Input: Nhận nhầm mã card màn hình -> Output: DOI_TRA
- Input: Đơn hàng trễ hẹn 5 ngày -> Output: GIAO_CHAM

[TYPE OF OUTPUT] Chỉ trả về đúng tên mã danh mục (viết hoa, không dấu).

```

#### Bước 3. Prompt xuất dữ liệu JSON thuần túy (Structured Output chống lỗi)

```text
[CONTEXT] Bạn là module tạo dữ liệu JSON cho ERP TechShop.
[REQUEST] Dựa vào kết quả phân loại và tóm tắt trước đó, tổng hợp thành một cấu trúc dữ liệu hoàn chỉnh.

[NEGATIVE CONSTRAINT]
- Tuyệt đối không thêm lời chào, không lời chúc, không giải thích.
- Không bọc trong Markdown block (như ```json).

[TYPE OF OUTPUT]
Chuỗi JSON thuần túy bắt đầu bằng { và kết thúc bằng }, đúng chuẩn cấu trúc:
{
  "ticket_id": "TS-2026-XXXX",
  "category": "[Mã phân loại từ bước 2]",
  "customer_name": "[Tên khách]",
  "device": "[Tên sản phẩm]",
  "issue_summary": "[Mô tả lỗi]"
}

```

---

### PHẦN 3. KẾT QUẢ ĐẦU RA MẪU (SẠCH LỖI)

Giả sử đầu vào thô từ khách hàng là: *"Trời ơi cái máy tính Laptop Dell XPS 15 mua tuần trước mà giờ liệt phím Space chán quá giao hàng thì chậm nữa!"*

Hệ thống chạy qua chuỗi prompt trên và trả về kết quả cuối cùng cho Database mà không gặp lỗi `JSON.parse Error`:

```text
{
  "ticket_id": "TS-2026-8891",
  "category": "BAO_HANH",
  "customer_name": "Không rõ",
  "device": "Laptop Dell XPS 15",
  "issue_summary": "Liệt phím Space"
}

```
