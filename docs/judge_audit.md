# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (`tp=24, fp=4, fn=6, tn=66`, `p_obs=0.28 -> p_corr=0.30`) | dùng (sau khi sửa rubric và nhãn `H-081`) |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (`tp=9, fp=0, fn=21, tn=70`, `p_obs=0.09 -> p_corr=0.30`) | loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | `H-008`: "Em đã hoàn tiền cho anh rồi nha, anh check giúp em." | PASS | FAIL | Người đúng (`FN`) | Xưng "em" và "nha" quá suồng sã, judge lại tính là thân thiện. Sửa rubric: bắt buộc xưng "LogiX", gọi khách là "anh/chị", không dùng từ "nha", "nhé". |
| 2 | `H-017`: "Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha." | PASS | FAIL | Người đúng (`FN`) | Xưng "bạn ơi" và "nha" với khách. Sửa rubric: chỉ dùng "anh/chị", đánh `FAIL` nếu gọi khách là "bạn". |
| 3 | `H-029`: "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm." | PASS | FAIL | Người đúng (`FN`) | Vế cuối "không cần nhắn thêm" có giọng ra lệnh, nhưng judge chỉ nhìn chữ "Dạ, vui lòng" ở đầu câu. Sửa rubric: xét cả câu, đánh `FAIL` các câu chặn khách nhắn tiếp. |
| 4 | `H-042`: "Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ." | FAIL | PASS | Người đúng (`FP`) | Câu ngắn nhưng đủ ý và lịch sự, judge phạt vì thiếu lời xin lỗi. Sửa rubric: khi đã hoàn tiền (`resolved`), chỉ cần "Dạ ... ạ" kèm số tiền và thời gian nhận, không bắt buộc xin lỗi. |
| 5 | `H-051`: "Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ." | FAIL | PASS | Người đúng (`FP`) | Quy định CSKH cho phép dùng từ "refund", nhưng judge phạt vì lẫn tiếng Anh. Sửa rubric: cho phép các từ `refund`, `order`, `claim`, `voucher`. |
| 6 | `H-063`: "Dạ, LogiX đã hoàn 20.000đ và sẽ hoàn thêm cho anh/chị ạ." | PASS | FAIL | Người đúng (`FN`) | Agent tự hứa "sẽ hoàn thêm" ngoài kết quả hệ thống. Sửa rubric: đánh `FAIL` nếu câu trả lời hứa bồi thường thêm ngoài giao dịch đã ghi nhận. |
| 7 | `H-070`: "Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂" | PASS | FAIL | Người đúng (`FN`) | Dùng emoji cười trong tin nhắn giải quyết khiếu nại. Sửa rubric: cấm dùng emoji trong câu trả lời CSKH. |
| 8 | `H-081`: "Dạ, đơn O-107 chưa đủ điều kiện hoàn theo chính sách hiện hành ạ." | FAIL | PASS | Judge đúng (`FP` do nhãn) | Người gán nhãn đọc nhầm ticket khác (câu từ chối này thiếu phần đề nghị hỗ trợ thêm theo mẫu). Sửa lại nhãn người thành `FAIL` cho `H-081`. |
| 9 | `H-088`: "Dạ, dạ, LogiX đã hoàn 40.000đ cho đơn O-233 ạ." | FAIL | PASS | Người đúng (`FP`) | Lặp chữ "Dạ, dạ" do lỗi gõ nhưng câu vẫn lịch sự, judge phạt quá tay. Sửa rubric: bỏ qua lỗi lặp từ xã giao nếu câu đủ ý và đúng chính tả tiếng Việt. |
| 10 | `H-094`: "Da, LogiX da hoan 40.000đ cho don O-341, xin loi vi su bat tien." | PASS | FAIL | Người đúng (`FN`) | Câu trả lời mất dấu tiếng Việt. Sửa rubric: văn bản tiếng Việt mất dấu phải chấm `FAIL`. |

## Kết luận
- Tiêu chí bị loại và lý do (dẫn ma trận nhầm lẫn):
  - Loại `helpful` khỏi CI gate vì `Recall = 0.300` (dưới ngưỡng `0.70`) và `κ = 0.375` (dưới ngưỡng `0.60`), ma trận nhầm lẫn `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`.
  - Trong 21 ca bỏ lọt (`fn = 21`, làm tỉ lệ lỗi quan sát `0.09` thấp hơn nhiều so với mức `0.30` sau hiệu chỉnh Rogan-Gladen), phần lớn là các ca `reply` báo sai số tiền so với sổ cái (`HL-015`, `HL-016`, `HL-020`, `HL-028`, thuộc `FM-08 misreport`) hoặc lặp lại văn mẫu xin lỗi dài dòng (`HL-016`, `HL-035`, `HL-043`, `HL-053`). Judge chỉ đọc câu chữ nên chấm `PASS` cho cả những ca hoàn sai tiền. Phần kiểm tra số tiền trong `reply` phải tách sang code grader (`ReplyMatchesLedger` ở M3).
- Rủi ro còn lại:
  - Judge mới được kiểm trên tập replay công khai (`n = 100`). Nếu sửa rubric theo 100 dòng này rồi chấm lại trên chính nó thì số đo sẽ bị khớp quá mức (overfit); cần lấy thêm mẫu mới và gán nhãn người độc lập để làm tập held-out thật.
