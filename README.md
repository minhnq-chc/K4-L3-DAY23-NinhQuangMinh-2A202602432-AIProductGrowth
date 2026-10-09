# Day 23 — Operating Dashboard · Đèn nào bật trước?
## Trợ lý AI Tiếp nhận và Phân tầng Triệu chứng Y tế (Vinmec Clinical Triage)

- **Học viên:** Ninh Quang Minh
- **Mã học viên (MSSV):** 2A202602432
- **Lớp:** K4 — Track 1 (L34A)
- **Sản phẩm:** Trợ lý AI Tiếp nhận và Phân tầng Triệu chứng Y tế (Vinmec Clinical Triage & Patient Intake)
- **Đối tác triển khai:** Hệ thống Y tế Vinmec (7 bệnh viện và 3 phòng khám đa khoa)
- **Ngày cập nhật:** 09/10/2026

---

## 1. Câu chốt loại mô hình (Model Statement)

> **Chúng tôi là B2B2C vì tiền đến từ Hệ thống Y tế Vinmec (chi trả theo mô hình Hybrid: $300 phí nền duy trì hạ tầng/tháng/cơ sở + $0,79/ca triage hoàn thành), người dùng thật là bệnh nhân hoặc thân nhân của Vinmec, và chúng tôi chạm trực tiếp người dùng cuối qua menu "Sơ loại triệu chứng 24/7" trên Zalo Official Account Vinmec và mục Tiếp nhận khám trên ứng dụng MyVinmec.**

### Căn cứ 3 câu hỏi thực tế (HANDBOOK §2.5):
1. **Ai trả tiền cho bạn?** Doanh nghiệp y tế — Hệ thống Y tế Vinmec (ký kết qua COO & Giám đốc Chuyên môn Lâm sàng, thanh toán từ ngân sách Vận hành & Tiếp nhận bệnh nhân).
2. **Ai dùng sản phẩm?** Bệnh nhân & Thân nhân bệnh nhân (khách hàng của người trả tiền).
3. **Bạn có chạm được người dùng cuối không?** Có, chạm trực tiếp và lưu trữ phiên hội thoại phân tầng ESI sơ bộ trên giao diện Zalo OA và ứng dụng MyVinmec.

---

## 2. Thông số kinh tế đơn vị & Mô hình tài chính (Căn cứ từ Day 22)

Các chỉ số cơ sở được trích xuất trực tiếp từ mô hình tài chính chuẩn tại `MinhNQ_Day22_model.xlsx`:

| Chỉ số | Giá trị | Căn cứ & Ý nghĩa |
| :--- | :---: | :--- |
| **Định nghĩa 1 Job** | 1 ca triage hoàn thành | Bệnh nhân nhận khuyến nghị phân tầng ESI sơ bộ, hướng dẫn đặt chuyên khoa hoặc điều dưỡng tiếp nhận an toàn. |
| **Giá bán đề xuất** | **$0,7900** (~20.540 ₫) | Bội số định giá 3,48× Cost/Job; tương đương benchmark Outcome-based y tế quốc tế (Zendesk AI $1,50; Intercom Fin $0,99). |
| **Cost / Job (COGS)** | **$0,2271** (~5.905 ₫) | Bao gồm chi phí LLM Claude Haiku 4.5 có cache ($0,0185), Infra bảo mật y tế ($0,008), chi phí nhân sự điều dưỡng trực HITL ($0,1915), retry 7% ($0,0017). |
| **Tỷ lệ Containment vận hành** | **78,00%** | 78% ca nhẹ/vừa AI tự xử lý hoàn tất; 22% ca cờ đỏ chuyển tiếp điều dưỡng trực. |
| **Breakeven Containment** | **70,29%** | Ngưỡng containment tối thiểu để duy trì Gross Margin ≥ 60%. |
| **Gross Margin (GM%)** | **71,25%** | Biên lợi nhuận gộp đạt chuẩn Vertical AI (65–75% Bessemer Cloud Index 2024). |
| **ARPU cơ sở** | **$790 / tháng** | $300 phí hạ tầng + $490 phí 1.000 ca triage hoàn thành tại 1 cơ sở. |
| **Ngân sách CAC tối đa** | **$10.132 / cơ sở** | ARPU ($790) × GM (71,25%) × Payback 18 tháng (phân khúc Mid-market). |
| **Kênh GTM chính** | Partner-Led (Vinmec) | Nhúng vào hệ sinh thái có sẵn của Vinmec, không dùng Sales-Led riêng lẻ (CAC sales-led $32.000 > ngân sách cho phép). |

---

## 3. Cấu trúc bộ tài liệu nộp bài

```text
K4-L3-DAY23-NinhQuangMinh-2A202602432-AIProductGrowth/
├── README.md        # Họ tên, MSSV, tên sản phẩm, 1 câu chốt loại mô hình & số liệu cơ sở
├── worksheet.md     # Trạm 1–4: Bảng đèn ✅/🔧/❌, thẻ đèn chi tiết, ngưỡng [BM]/[MH]/[TB], 3 phép tính [MH], 5 luật quyết định
├── dashboard.md     # Trạm 5: Operating Dashboard vừa đúng 1 trang A4
└── dashboard.pdf    # Bản in chuẩn 2 trang (Trang 1: Dashboard; Trang 2: Phụ lục phép tính [MH])
```
