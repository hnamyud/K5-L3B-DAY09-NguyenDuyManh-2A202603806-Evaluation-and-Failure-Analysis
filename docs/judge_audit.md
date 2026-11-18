# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (`tp=24, fp=4, fn=6, tn=66`, `p_obs=0.28 -> p_corr=0.30`) | **dùng** (sau khi cập nhật rubric & sửa nhãn `H-081`) |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (`tp=9, fp=0, fn=21, tn=70`, `p_obs=0.09 -> p_corr=0.30`) | **loại** (không được dùng làm gate) |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric / nhãn |
|---|---|---|---|---|---|
| 1 | `H-008`: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* | PASS | FAIL | Người đúng (`FN`) | Xưng *"em"* + *"nha"* quá suồng sã; judge lầm tưởng là thân thiện. **Sửa rubric:** bắt buộc xưng *"LogiX"* và gọi khách là *"anh/chị"*, cấm từ khẩu ngữ (*"nha"*, *"nhé"*). |
| 2 | `H-017`: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* | PASS | FAIL | Người đúng (`FN`) | Xưng *"bạn ơi"* + *"nha"* không chuẩn mực với khách hàng. **Sửa rubric:** chỉ cho phép đại từ *"anh/chị"*, đánh `FAIL` nếu gọi khách là *"bạn"*. |
| 3 | `H-029`: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* | PASS | FAIL | Người đúng (`FN`) | Câu cuối *"không cần nhắn thêm"* mang giọng điệu ra lệnh/đuổi khách, nhưng judge bị đánh lừa bởi cụm *"Dạ, vui lòng"* ở đầu câu. **Sửa rubric:** kiểm tra toàn bộ câu, cấm các mệnh lệnh ngăn khách phản hồi. |
| 4 | `H-042`: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* | FAIL | PASS | Người đúng (`FP`) | Câu ngắn gọn, lịch sự, đủ thông tin nhưng judge phạt vì *"thiếu câu xin lỗi"*. **Sửa rubric:** với thông báo hoàn tiền thành công (`resolved`), chỉ cần *"Dạ ... ạ"* và thông tin số tiền/thời gian, không bắt buộc câu xin lỗi. |
| 5 | `H-051`: *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* | FAIL | PASS | Người đúng (`FP`) | Quy chuẩn CSKH LogiX cho phép dùng thuật ngữ *"refund"*, nhưng judge phạt vì lẫn tiếng Anh. **Sửa rubric:** bổ sung danh sách từ mượn được phép (`refund`, `order`, `claim`, `voucher`). |
| 6 | `H-063`: *"Dạ, LogiX đã hoàn 20.000đ và sẽ hoàn thêm cho anh/chị ạ."* | PASS | FAIL | Người đúng (`FN`) | Tự ý hứa hẹn *"sẽ hoàn thêm"* ngoài quyết định của hệ thống. **Sửa rubric:** đánh `FAIL` mọi cam kết/hứa hẹn bồi thường thêm không nằm trong kết quả giao dịch. |
| 7 | `H-070`: *"Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂"* | PASS | FAIL | Người đúng (`FN`) | Dùng emoji cười (`😂😂`) phản cảm trong ngữ cảnh xử lý khiếu nại. **Sửa rubric:** cấm hoàn toàn emoji/icon cảm xúc trong câu trả lời CSKH. |
| 8 | `H-081`: *"Dạ, đơn O-107 chưa đủ điều kiện hoàn theo chính sách hiện hành ạ."* | FAIL | PASS | **Judge đúng** (`FP` giả) | Người gán nhãn đọc nhầm sang ticket khác (câu từ chối cộc lốc thiếu câu hỗ trợ thêm theo mẫu chuẩn). **Hành động:** sửa lại nhãn người (`human = FAIL`) cho `H-081` trong bộ golden. |
| 9 | `H-088`: *"Dạ, dạ, LogiX đã hoàn 40.000đ cho đơn O-233 ạ."* | FAIL | PASS | Người đúng (`FP`) | Lỗi lặp từ *"Dạ, dạ"* nhẹ nhưng thái độ vẫn lịch sự; judge phạt quá nặng lỗi chính tả nhỏ. **Sửa rubric:** không đánh `FAIL` lỗi lặp từ lịch sự nếu câu vẫn rõ nghĩa và có dấu tiếng Việt đầy đủ. |
| 10 | `H-094`: *"Da, LogiX da hoan 40.000đ cho don O-341, xin loi vi su bat tien."* | PASS | FAIL | Người đúng (`FN`) | Câu trả lời mất toàn bộ dấu tiếng Việt. **Sửa rubric:** bắt buộc văn bản tiếng Việt phải có dấu chuẩn; không dấu ⇒ `FAIL`. |

## Kết luận
- **Tiêu chí bị loại và lý do (dẫn ma trận nhầm lẫn):**
  - Tiêu chí **`helpful`** bị **loại bỏ hoàn toàn khỏi CI gate** vì chỉ đạt `Recall = 0.300` (< sàn `0.70`) và `κ = 0.375` (< sàn `0.60`), với ma trận nhầm lẫn `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`.
  - Phân tích 21 ca bỏ lọt lỗi (`fn = 21`, dẫn đến tỉ lệ lỗi quan sát `0.09` thấp hơn 3.3 lần so với tỉ lệ thực `0.30` sau hiệu chỉnh Rogan-Gladen): phần lớn là các ca agent báo **sai số tiền hoàn so với ledger** (`HL-015`, `HL-016`, `HL-020`, `HL-028` — tức `FM-08 misreport`) hoặc trả lời rập khuôn xin lỗi rất dài dòng (*verbosity bias*, như `HL-016`, `HL-035`, `HL-043`, `HL-053`). Do LLM Judge chỉ đọc văn bản `reply` mà không đối chiếu (hoặc bị lời xin lỗi trôi chảy che mắt trước sai lệch số học trên sổ cái), nó chấm `PASS` cho cả các ca hoàn sai tiền. Việc kiểm tra khớp số tiền giữa `reply` và `ledger` bắt buộc phải tách sang **Code Grader** (M3), không được giao cho LLM Judge.
- **Rủi ro còn lại:**
  - Judge chỉ mới được kiểm định trên dữ liệu replay đã công khai (`n = 100`); nếu tinh chỉnh rubric theo chính 100 mẫu này rồi đo lại trên cùng tập sẽ bị overfitting. Cần thu thập và gán nhãn người độc lập trên một tập **held-out mới** trước khi đưa `tone_ok` vào production.
