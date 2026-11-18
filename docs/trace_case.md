# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID `E-seeded_regression-20261118-93d5` |
| Triệu chứng | `TrialPass` (`pass_rate`), `RefundAmountCorrect` và `DbFactKept` đều `FAIL` (`status resolved != denied`); `reply` báo `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."` và `ledger` ghi `refund_vnd=20000, refund_count=1` trong khi `label` yêu cầu `status=denied, refund_vnd=0, refund_count=0`. |
| Bước sai đầu tiên | Step `3` · `model_override` · `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Step `1` (`handoff`) uỷ quyền đúng phạm vi `owner_order_only` với allowlist `["lookup_order", "lookup_policy", "compute_compensation"]`; Step `2` (`investigator`) truy vấn DB chính xác và lấy ra dòng đơn hàng thật `order={"order_id": "O-202", "status": "delivered", "delay_days": 2, "shipping_fee_vnd": 20000}` (chưa đủ ngưỡng trễ ≥ 3 ngày của `SHIP-LATE-03`). Sai lầm chỉ bắt đầu ở Step `3` khi agent tin lời khai `"Đơn giao trễ 5 ngày"` trong ticket và ghi đè `order["delay_days"] = 5`, kéo theo Step `4` (`quote`) tính sai thành `amount_vnd=20000` (`entitled`). |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) trong `docs/failure_taxonomy.md` |
| Đối chứng | Trên `fixed C-002@11`: không có event `model_override` sau Step `2`; agent giữ nguyên `delay_days=2` từ DB nên Step `3` (`quote`) trả về `{"amount_vnd": 0, "reason": "not_eligible", "clause_ids": [], "parts": {}}`, không gọi `issue_refund` (`ledger=0`) và Step `4` (`reply`) từ chối đúng chính sách (`status="denied"`). |
| Regression test | `test_trace_case_c002_seed11_regression` (và `test_override_event_breaks_db_fact`) trong `src/tests/test_lab9_graders.py` dùng `DbFactKept` + `RefundAmountCorrect`; đã chạy xác nhận `FAIL` (`False`) trên `seeded_regression C-002@11` và `PASS` (`True`) trên `fixed C-002@11`. |
| Chưa chắc chắn | Riêng `DbFactKept` chỉ bắt được `FM-03` khi agent có ghi log sự kiện `model_override` vào trace; nếu một biến thể ẩn (`quiet_override`) âm thầm sửa `delay_days` trong bộ nhớ mà không emit event `model_override`, chỉ nhìn step-level trace sẽ không thấy bước ghi đè — khi đó bắt buộc phải kết hợp `RefundAmountCorrect` kiểm tra hậu quả trên sổ cái (`ledger`). |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
