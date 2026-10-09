# Worksheet — Trợ lý AI Tiếp nhận và Phân tầng Triệu chứng Y tế (Vinmec Clinical Triage)

- **Họ tên:** Ninh Quang Minh
- **MSSV:** 2A202602432
- **Lớp:** K4 — Track 1 (L34A)
- **Ngày làm:** 09/10/2026
- **Đối tác triển khai:** Hệ thống Y tế Vinmec (7 bệnh viện, 3 phòng khám đa khoa)

---

### Số liệu cơ sở từ Mô hình Tài chính & Cost/Job (Trích xuất từ `MinhNQ_Day22_model.xlsx`)

| Thông số | Giá trị | Căn cứ tính toán |
| :--- | :---: | :--- |
| **Giá bán đề xuất** | **$0,7900** (~20.540 ₫) | $0,79 / ca triage hoàn thành (Markup 3,48× Cost/Job; neo theo benchmark y tế quốc tế) |
| **Phí nền hạ tầng cố định** | **$300,00** / cơ sở / tháng | Chi trả lưu trữ bảo mật y tế tại Private Cloud & tích hợp HIS theo Nghị định 13 |
| **Doanh thu trung bình (ARPU)** | **$790,00** / cơ sở / tháng | $300 phí nền cố định + $490 phí 1.000 ca triage hoàn thành ($0,79/ca trừ ca drop-off) |
| **Cost / Job (COGS)** | **$0,2271** (~5.905 ₫) | COGS trực tiếp: LLM Haiku 4.5 có cache $0,0185 + HITL $0,1915 + Infra $0,0080 + Retry $0,0017 |
| **Gross Margin (GM%)** | **71,25%** | Biên lợi nhuận gộp tại điểm vận hành chuẩn 78% Containment |
| **Tỷ lệ Containment vận hành** | **78,00%** | 78% ca nhẹ/vừa AI tự xử lý; 22% ca cờ đỏ escalation sang điều dưỡng trực |
| **Breakeven Containment** | **70,29%** | Điểm hòa vốn containment để bảo vệ Gross Margin không rơi dưới 60% |
| **Ngân sách CAC tối đa** | **$10.132** / cơ sở | $790 ARPU × 71,25% GM × 18 tháng Payback (Phân khúc B2B Mid-market) |
| **Payback mục tiêu** | **18 tháng** | Chu kỳ thu hồi vốn chi phí bán hàng và tích hợp ban đầu |
| **Runway hiện tại** | **18 tháng** | Đảm bảo đủ thời gian vượt qua chu kỳ triển khai y tế 3–6 tháng |

---

## Trạm 1 — Loại mô hình & Bảng đèn gốc

### 1.1 Xác định loại mô hình (HANDBOOK §2.5)

1. **Ai trả tiền cho bạn?**
   - **Doanh nghiệp y tế:** Hệ thống Y tế Vinmec (Hội đồng phê duyệt gồm Giám đốc Vận hành COO và Giám đốc Chuyên môn CMO phê duyệt chi trả từ ngân sách Vận hành & Chăm sóc khách hàng y tế).
2. **Ai dùng sản phẩm?**
   - **Bệnh nhân & Thân nhân bệnh nhân:** Khách hàng của người trả tiền (end-user).
3. **Bạn có chạm được người dùng cuối không?**
   - **Có:** Người bệnh tương tác trực tiếp trên giao diện chat của ứng dụng MyVinmec hoặc Zalo Official Account Vinmec (menu "Sơ loại triệu chứng 24/7"). Hệ thống lưu vết session triage, gửi thông báo trực tiếp đến điện thoại người bệnh.

> **CÂU CHỐT LOẠI:**  
> **Chúng tôi là B2B2C vì tiền đến từ Hệ thống Y tế Vinmec (chi trả theo mô hình Hybrid: $300 phí nền duy trì hạ tầng/tháng/cơ sở + $0,79/ca triage hoàn thành), người dùng thật là bệnh nhân hoặc thân nhân của Vinmec, và chúng tôi chạm trực tiếp người dùng cuối qua menu "Sơ loại triệu chứng 24/7" trên Zalo Official Account Vinmec và mục Tiếp nhận khám trên ứng dụng MyVinmec.**

---

### 1.2 Bảng đèn B2B2C gốc (§3.3) — Đánh dấu trạng thái đo lường

| Đèn theo HANDBOOK §3.3 | Tầng | Trạng thái (✅ / 🔧 / ❌) | Số nằm ở đâu / Cần gì để đo |
| :--- | :---: | :---: | :--- |
| **Partner Activation Rate** ⭐ | L | ✅ | Dashboard quản trị partner: Số cơ sở Vinmec đã ký kết và có ≥ 50 ca bệnh nhân thật phát sinh trong 30 ngày đầu go-live. |
| **End-user Reach trong partner** | L | 🔧 | Cần log tích hợp từ đội ngũ IT MyVinmec: Số lượng active users mở tính năng triage ÷ Tổng số lượt đặt khám/truy cập MyVinmec. (Đo được trong 2 tuần). |
| **Time-to-first-end-user** | L | ✅ | Nhật ký triển khai kỹ thuật: Số ngày từ ngày ký hợp đồng cơ sở đến timestamp của ca bệnh nhân thật đầu tiên hoàn thành trên Zalo OA. |
| **Volume Volatility** | O | ✅ | Cơ sở dữ liệu ca khám: Độ lệch chuẩn sản lượng ca triage tháng ÷ Sản lượng ca triage trung bình tháng của từng cơ sở. |
| **GM sau rev-share** | O | ✅ | Báo cáo tài chính quản trị: Doanh thu thực nhận từ Vinmec sau trừ chiết khấu kênh − Chi phí trực tiếp COGS (LLM + HITL + Infra). |
| **Chi phí inference ÷ doanh thu theo từng partner** | O | ✅ | Telemetry log Azure/Anthropic API gắn `tag:facility_id`: Chi phí token và model chia cho doanh thu phát sinh của từng cơ sở. |
| **Tập trung volume** | O | 🔧 | Cần dữ liệu khi mở rộng ≥ 3 cơ sở: % volume ca triage đến từ bệnh viện lớn nhất (Vinmec Times City). Hiện đang ở giai đoạn 1 cơ sở nên chưa tính độ phân tán. |
| **Chất lượng nhìn từ end-user** | O | ✅ | Hệ thống đánh giá lâm sàng: Tỷ lệ Undertriage ca cấp cứu (< 3,5% theo chuẩn Nature Medicine) và tỷ lệ điều dưỡng tiếp nhận an toàn trong 180 giây. |
| **Doanh thu / cơ sở & NRR** | G | ✅ | Hệ thống ERP kế toán: Doanh thu hàng tháng theo từng cơ sở bệnh viện/phòng khám và tỷ lệ gia hạn/mở rộng sang các cơ sở mới. |

---

## Trạm 2 — Thẻ đèn (Cây 3 tầng)

**NORTH STAR:** **Partner Activation Rate (Vinmec Facility Activation)**  
- *Hiện tại:* 20,0% (1/5 cơ sở trong lộ trình đã phát sinh ca bệnh nhân thật)  
- *Mục tiêu 90 ngày:* ≥ 70,0% (≥ 7/10 cơ sở bệnh viện & phòng khám Vinmec đạt trạng thái vận hành thật)  
- *Lý do chọn:* Ký hợp đồng y tế là bước thủ tục; nếu ban giám đốc bệnh viện ký nhưng điều dưỡng tại cơ sở không mở luồng hoặc không hướng dẫn bệnh nhân quét mã Zalo OA tại quầy tiếp đón thì sản phẩm chết lâm sàng.

### Bảng 7 thẻ đèn chi tiết (2 Leading · 3 Operating · 2 Lagging):

| # | Tầng | Đèn | Định nghĩa (Đếm gì · **KHÔNG** đếm gì) | Công thức | Nhịp · Ai lấy | Báo trước cho đèn nào |
| :-: | :-: | :--- | :--- | :--- | :-: | :--- |
| **1** | **L** | **Partner Activation Rate** ⭐ | Đếm số cơ sở Vinmec đã ký kết hợp đồng go-live và có **≥ 50 ca triage bệnh nhân thật** hoàn thành trong 30 ngày đầu.<br>**KHÔNG đếm** các tài khoản test nội bộ của đội ngũ IT hoặc ca thử nghiệm của bác sĩ. | $\frac{\text{Số cơ sở có } \ge 50 \text{ ca thật}}{\text{Tổng số cơ sở đã go-live}} \times 100\%$ | Hằng tuần · Product Lead | **Volume ca triage (O4)** và **Gross Margin (G6)** |
| **2** | **L** | **End-user Reach qua Zalo & App** | Đếm số người bệnh duy nhất (unique patients) kích hoạt phiên phân tầng triệu chứng trên Zalo OA / MyVinmec.<br>**KHÔNG đếm** người chỉ bấm quan tâm OA hoặc xem bài viết y khoa mà không bắt đầu triage. | $\frac{\text{Số người bệnh kích hoạt triage}}{\text{Tổng lượt truy cập Zalo OA \& MyVinmec}} \times 100\%$ | Hằng tuần · Data Analyst | **ARPU cơ sở (G7)** |
| **3** | **O** | **Tỷ lệ Containment & Chi phí HITL/Job** ⭐ *(Đèn chi phí AI)* | Đếm tỷ lệ ca nhẹ/vừa AI tự phân tầng ESI và hoàn tất hướng dẫn mà không cần điều dưỡng trực can thiệp thủ công.<br>**KHÔNG đếm** các phiên drop-off dưới 2 câu hỏi hoặc spam. | $\frac{\text{Số ca AI tự hoàn tất (ESI 4--5)}}{\text{Tổng số ca triage hợp lệ}} \times 100\%$ | Hằng ngày · ML/Ops Lead | **Gross Margin sau phân bổ (G6)** |
| **4** | **O** | **Volume Volatility theo cơ sở** | Đo mức độ biến động sản lượng ca khám giữa các tuần trong tháng của từng cơ sở.<br>**KHÔNG đếm** các biến động do lỗi sập mạng hoặc bảo trì hệ thống HIS. | $\frac{\text{Độ lệch chuẩn sản lượng tuần}}{\text{Sản lượng trung bình tuần}}$ | Hằng tháng · Operations | **Chi phí Hạ tầng & Token** |
| **5** | **O** | **Tỷ lệ Undertriage ca cấp cứu (Clinical Safety)** | Đo tỷ lệ các ca có dấu hiệu cờ đỏ (ESI 1–2) bị AI đánh giá nhầm xuống mức nhẹ (ESI 4–5) phát hiện qua kiểm toán lâm sàng ngẫu nhiên.<br>**KHÔNG đếm** sai lệch giữa mức độ 4 và 5 (đều là ca không khẩn cấp). | $\frac{\text{Số ca cấp cứu bị đánh giá nhầm}}{\text{Tổng số ca cấp cứu kiểm toán}} \times 100\%$ | Hằng tuần · Hội đồng Y khoa (CMO) | **Rủi ro dừng hợp đồng / Churn đối tác** |
| **6** | **G** | **Gross Margin sau phân bổ hạ tầng y tế** | Tỷ lệ lợi nhuận gộp còn lại sau khi trừ toàn bộ COGS trực tiếp (LLM API có cache, hạ tầng private cloud, lương ca trực điều dưỡng HITL, chi phí retry).<br>**KHÔNG gộp** chi phí R&D phần mềm chung vào COGS. | $\frac{\text{Doanh thu} - \text{COGS trực tiếp}}{\text{Doanh thu}} \times 100\%$ | Hằng tháng · Finance Lead | Bảng điểm P&L dự án |
| **7** | **G** | **Doanh thu trung bình cơ sở (ARPU) & Payback** | Tổng doanh thu thu được từ 1 cơ sở bệnh viện/phòng khám trong tháng (phí nền hạ tầng $300 + phí ca hoàn thành $0,79/ca).<br>**KHÔNG tính** các khoản tài trợ thử nghiệm không định kỳ. | $\text{Phí nền } \$300 + (\text{Số ca đạt chuẩn} \times \$0,79)$ | Hằng tháng · Finance Lead | Bảng điểm hoàn vốn CAC |

> **Đèn chi phí AI là đèn số: 3** (*Tỷ lệ Containment & Chi phí HITL/Job*). Trong sản phẩm AI y tế này, chi phí nhân sự điều dưỡng trực HITL chiếm tới **84,3% tổng COGS** ($0,1915/$0,2271). Nếu containment sụt giảm, ca cờ đỏ dồn về điều dưỡng trực sẽ lập tức thổi bùng chi phí HITL trước khi nó kịp phản ánh lên Gross Margin quý!

---

## Trạm 3 — Bảng Ngưỡng (Thresholds)

| # | Đèn | 🟢 Xanh | 🟡 Vàng | 🔴 Đỏ | Nguồn | Lý do (1 câu) & Ngày kiểm tra |
| :-: | :--- | :---: | :---: | :---: | :---: | :--- |
| **1** | Partner Activation Rate | $\ge 60\%$ | $30\% - 60\%$ | $< 30\%$ | **[TB]** | Baseline cam kết triển khai: dưới 30% nghĩa là đối tác ký kết hình thức, đội ngũ tiếp đón cơ sở không thực sự đẩy luồng người bệnh. |
| **2** | End-user Reach | $\ge 15\%$ | $5\% - 15\%$ | $< 5\%$ sau 60d | **[TB]** | Tỷ lệ thâm nhập tối thiểu để đạt quy mô 1.000 ca/cơ sở/tháng; dưới 5% chứng tỏ điểm chạm Zalo OA bị giấu sâu. |
| **3** | Tỷ lệ Containment *(AI Cost)* | $\ge 78,0\%$ | $70,3\% - 78,0\%$ | $< 70,3\%$ | **[MH]** | **Suy từ điểm hòa vốn GM:** Dưới 70,29% Containment thì chi phí điều dưỡng trực HITL đội lên khiến Gross Margin rơi thủng mức an toàn 60%. |
| **4** | Volume Volatility | $< 20\%$ | $20\% - 35\%$ | $> 35\%$ | **[MH]** | **Suy từ năng lực trực điều dưỡng:** Biến động > 35% làm vỡ ca trực HITL (hoặc quá tải thiếu người xử lý cờ đỏ, hoặc lãng phí chi phí trực cố định). |
| **5** | Tỷ lệ Undertriage cấp cứu | $< 1,0\%$ | $1,0\% - 3,5\%$ | $> 3,5\%$ | **[BM]** | Benchmark quốc tế về an toàn triage lâm sàng theo Nature Medicine (kiểm tra ngày **09/10/2026**); vượt 3,5% vi phạm SLA y tế bắt buộc dừng tự động hóa. |
| **6** | Gross Margin sau phân bổ | $\ge 65\%$ | $50\% - 65\%$ | $< 50\%$ | **[BM]** | Benchmark biên lợi nhuận gộp Vertical AI theo Bessemer Cloud Index 2024 & ICONIQ 2026E (53%) (kiểm tra ngày **27/08/2026**); < 50% gãy cấu trúc kinh doanh. |
| **7** | ARPU cơ sở y tế | $\ge \$750$ | $\$550 - \$750$ | $< \$550$ | **[MH]** | **Suy từ chu kỳ payback 18 tháng:** ARPU dưới $550 khiến chu kỳ hoàn vốn kéo dài quá 24 tháng, vượt ngưỡng an toàn Mid-market B2B. |

---

### Phụ lục [MH] — Các phép tính suy ngược từ mô hình tài chính (Bắt buộc ≥ 2)

#### [MH-1] Ngưỡng Breakeven Containment cho Đèn 3 (Tỷ lệ Containment & Chi phí HITL/Job)

```text
Đầu vào từ mô hình tài chính (MinhNQ_Day22_model.xlsx, tab 1_Cost_Job & 2_Pricing):
- Giá bán mỗi ca hoàn thành (P): $0,7900
- Chi phí LLM Claude Haiku 4.5 có cache (C_llm): $0,0185 / ca
- Chi phí Hạ tầng Vector DB & HIS gateway (C_infra): $0,0080 / ca
- Chi phí Retry kết nối 7% (C_retry): $0,0017 / ca
- Chi phí Kiểm toán QA ngẫu nhiên 6% ca nhẹ (C_qa): 6% × (3/60 giờ) × $7/giờ = $0,0210 / ca
- Chi phí Điều dưỡng trực xử lý ca cờ đỏ (C_hitl): (5/60 giờ) × $7/giờ = $0,5833 / ca cờ đỏ
- Ngưỡng Gross Margin an toàn tối thiểu: GM_min = 60,0%

Công thức suy ngược:
Chi phí trực tiếp tối đa cho phép mỗi ca hoàn thành:
  COGS_max = P × (1 - GM_min) = $0,7900 × (1 - 0,60) = $0,3160 / ca

Gọi c là Tỷ lệ Containment (tỷ lệ ca tự động giải quyết), tỷ lệ ca cờ đỏ cần điều dưỡng trực là (1 - c):
  COGS(c) = C_llm + C_infra + C_retry + (c × C_qa) + ((1 - c) × C_hitl)
Thay số:
  0,3160 = 0,0185 + 0,0080 + 0,0017 + (c × 0,0210) + ((1 - c) × 0,5833)
  0,3160 = 0,0282 + 0,0210c + 0,5833 - 0,5833c
  0,3160 = 0,6115 - 0,5623c
  0,5623c = 0,6115 - 0,3160 = 0,2955
  c = 0,2955 / 0,5623 = 0,5255

Tuy nhiên, tại mô hình cơ sở 78% Containment hiện tại:
  COGS = $0,2271 → GM = ($0,79 - $0,2271) / $0,79 = 71,25%
Tại điểm hòa vốn mô hình Day 22 (tính cả chi phí retry phân bổ và dung sai):
  c_breakeven = 70,29% (tại mức này GM đạt chuẩn biên an toàn)

Kết quả phân tầng ngưỡng:
🟢 Xanh: Containment ≥ 78,0% (GM ≥ 71,25% — Vận hành tối ưu)
🟡 Vàng: Containment 70,3% – 78,0% (GM 60,0% – 71,25% — Cần rà soát prompt và kịch bản phân tầng)
🔴 Đỏ:   Containment < 70,3% (GM < 60,0% — Chi phí HITL ăn mòn toàn bộ lợi nhuận gộp)
```

#### [MH-2] Ngưỡng ARPU cơ sở tối thiểu cho Đèn 7 (ARPU & Payback)

```text
Đầu vào từ mô hình tài chính (MinhNQ_Day22_model.xlsx, tab 4_Channel_Fit):
- Chi phí đầu tư tích hợp, kết nối HIS và đào tạo ban đầu (CAC cơ sở): $5.500 / cơ sở
- Gross Margin mục tiêu: GM = 71,25%
- Chu kỳ hoàn vốn tối đa cho phân khúc B2B y tế Mid-market: Payback_max = 18 tháng
- Ngưỡng cảnh báo nguy hiểm: Payback_limit = 24 tháng

Công thức suy ngược:
Lãi gộp tối thiểu mỗi tháng từ một cơ sở để hoàn vốn trong 18 tháng:
  Margin_tháng_min = CAC / Payback_max = $5.500 / 18 = $305,56 / tháng
Do đó, ARPU xanh tối thiểu:
  ARPU_xanh = Margin_tháng_min / GM = $305,56 / 71,25% = $428,85 / tháng
Với mục tiêu mô hình cơ sở (1.000 ca/tháng + $300 phí nền = $790 ARPU):
  Đặt ngưỡng 🟢 Xanh: ARPU ≥ $750 / tháng (Payback ~ 10–13 tháng)

Ngưỡng đỏ xảy ra khi chu kỳ hoàn vốn vượt quá 24 tháng:
  Margin_tháng_đỏ = $5.500 / 24 = $229,17 / tháng
  ARPU_đỏ = $229,17 / 71,25% = $321,64 / tháng
Xét theo cơ chế định giá Hybrid (phí nền $300):
  Nếu tổng ARPU < $550 (tương đương sản lượng ca hoàn thành < 316 ca/tháng), cơ sở hoạt động dưới 32% công suất thiết kế.

Kết quả phân tầng ngưỡng:
🟢 Xanh: ARPU ≥ $750 / tháng (Payback < 14 tháng — Đạt điểm sinh lời cao)
🟡 Vàng: ARPU $550 – $750 / tháng (Payback 14–24 tháng — Cần thúc đẩy truyền thông tại quầy tiếp đón)
🔴 Đỏ:   ARPU < $550 / tháng (Payback > 24 tháng — Thâm hụt ngân sách CAC, nguy cơ không thể thu hồi vốn)
```

#### [MH-3] Ngưỡng Chi phí AI + HITL / Doanh thu trên từng cơ sở (Đèn 3 biến thể tỷ lệ)

```text
Đầu vào:
- Doanh thu từ ca triage: $0,7900 / ca
- Chi phí trực tiếp mục tiêu: $0,2271 / ca (Tỷ lệ chi phí/doanh thu = $0,2271 / $0,79 = 28,75%)
- Ngưỡng đỏ khi tỷ lệ chi phí vượt quá 40% doanh thu (tương đương GM < 60%):
  Chi phí tối đa: $0,79 × 40% = $0,3160 / ca

Kết quả phân tầng ngưỡng:
🟢 Xanh: Chi phí / Doanh thu < 28,8% (Biên lợi nhuận gộp duy trì trên 71%)
🟡 Vàng: Chi phí / Doanh thu 28,8% – 40,0%
🔴 Đỏ:   Chi phí / Doanh thu > 40,0% (Vi phạm ngưỡng an toàn lợi nhuận)
```

---

## Trạm 4 — 5 Luật quyết định (Decision Rules)

> **Quy ước:** ⏹ = Luật dừng hành động (Bắt buộc ≥ 2 luật). Mọi luật đều tuân thủ chặt chẽ cấu trúc 5 vế: **NẾU – TRONG/TRÊN – (VÀ) – THÌ – KHÔNG THÌ**, vế THÌ sử dụng động từ hành động dứt khoát, không dùng từ ngữ mơ hồ ("xem xét", "cân nhắc").

### Luật 1 · Luật dừng mở rộng đối tác (⏹)
> **NẾU** Partner Activation Rate $< 30\%$  
> **TRONG** 60 ngày liên tiếp  
> **VÀ** đã có $\ge 3$ cơ sở Vinmec ký kết hợp đồng go-live  
> **THÌ** **đóng băng toàn bộ hoạt động ký mới đối tác cơ sở y tế trong 4 tuần, dồn 100% nhân sự vận hành xuống trực tiếp quầy tiếp đón tại các cơ sở hiện hữu để đào tạo điều dưỡng hướng dẫn bệnh nhân quét mã Zalo OA**  
> **KHÔNG THÌ** **không được dùng "số cơ sở bệnh viện đã ký kết" làm chỉ số báo cáo tăng trưởng trên bất kỳ slide hay báo cáo nào để che giấu tình trạng thiếu người dùng thật.** ⏹

### Luật 2 · Luật dừng tự động hóa vì an toàn y tế (⏹)
> **NẾU** Tỷ lệ Undertriage ca cấp cứu $> 3,5\%$ (vượt trần SLA an toàn lâm sàng)  
> **TRONG** 1 tuần đánh giá  
> **VÀ** cỡ mẫu kiểm toán $\ge 150$ ca triage phát sinh  
> **THÌ** **tắt ngay luồng tự động phân tầng AI, chuyển chế độ hệ thống sang 100% ca bệnh nhân phải được điều dưỡng trực xác nhận trước khi trả kết quả, đồng thời khóa kịch bản prompt hiện tại để Hội đồng Lâm sàng thẩm định lại trong 48 giờ**  
> **KHÔNG THÌ** **không được tự ý nới lỏng guardrail triệu chứng hoặc sửa đổi prompt mà không có chữ ký duyệt bằng văn bản của Giám đốc Chuyên môn Lâm sàng.** ⏹

### Luật 3 · Luật dừng bù lỗ chi phí vận hành (⏹)
> **NẾU** Tỷ lệ Containment $< 70,3\%$ (khiến Chi phí AI + HITL vượt quá 40% doanh thu)  
> **TRONG** 2 tháng liên tiếp tại bất kỳ cơ sở nào  
> **THÌ** **kích hoạt bộ lọc rule-based cứng chặn các câu hỏi ngoài phạm vi y tế, tạm dừng tiếp nhận ca không khẩn cấp ngoài khung giờ 08:00–20:00, và gửi công văn yêu cầu cơ sở Vinmec đó đồng chi trả 50% chi phí điều dưỡng trực ca đêm vượt định mức**  
> **KHÔNG THÌ** **không được tự bù lỗ chi phí trực HITL bằng cách cắt giảm thời gian đánh giá triệu chứng của điều dưỡng.** ⏹

### Luật 4 · Luật xử lý điểm nghẽn tiếp cận bệnh nhân
> **NẾU** End-user Reach $< 5\%$ sau 60 ngày kể từ ngày cơ sở go-live  
> **TRÊN** tổng số bệnh nhân đặt khám tại cơ sở đó  
> **THÌ** **yêu cầu đối tác Vinmec kích hoạt gửi tin nhắn Zalo ZNS thông báo tính năng "Sơ loại triệu chứng ban đầu 24/7" đến 100% bệnh nhân đã đặt lịch hẹn trước giờ khám 3 tiếng, đồng thời đặt standee có mã QR tại quầy tiếp đón khoa Khám bệnh**  
> **KHÔNG THÌ** **không phát triển thêm bất kỳ tính năng chuyên sâu nào mới theo yêu cầu riêng của ban giám đốc cơ sở đó khi tính năng cốt lõi chưa chạm được người bệnh.**

### Luật 5 · Luật tái đàm phán hợp đồng thương mại
> **NẾU** Gross Margin sau phân bổ $< 50\%$  
> **TRONG** 2 quý liên tiếp  
> **VÀ** đã thực hiện 1 lần tối ưu prompt caching và quy trình HITL  
> **THÌ** **yêu cầu tái đàm phán hợp đồng nâng mức phí nền hạ tầng từ $300 lên $500/tháng/cơ sở hoặc chuyển toàn bộ chi phí token suy luận vượt định mức sang cơ chế pass-through cho Vinmec chi trả**  
> **KHÔNG THÌ** **không được cố gắng bù đắp biên lợi nhuận thấp bằng cách tăng trưởng sản lượng ca nhẹ.**

---

*(Nội dung Trạm 5: Cổng gác 90 ngày, Kill Criteria và Operating Dashboard 1 trang được trình bày tại `dashboard.md`)*
