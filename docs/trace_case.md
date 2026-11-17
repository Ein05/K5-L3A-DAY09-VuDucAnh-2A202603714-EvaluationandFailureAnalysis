# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11` |
| Triệu chứng | Grader Lab 8 fail: `status resolved != denied`. Ledger ghi nhận giải ngân `refund_vnd=20000`, `status=resolved` trong khi ground truth nhãn kỳ vọng `status=denied`, `refund_vnd=0`. Reply thông báo: *"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."* |
| Bước sai đầu tiên | Event 3: `model_override` payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) khởi tạo hợp lệ; Event 2 (`investigator: lookup_order`) gọi tool đúng, đọc DB chính xác đơn `O-202` giao hàng chỉ trễ 2 ngày (không đủ ngưỡng hoàn tiền của điều khoản `SHIP-LATE-03`). Bước 3 là thời điểm đầu tiên agent đi chệch khỏi quy trình đúng bằng cách phát sinh sự kiện `model_override` tin lời khai của ticket để ghi đè dữ liệu DB. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`), kéo theo `FM-02` (`ignore_denied`). |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: trace không hề có event `model_override`. Agent tin tưởng dữ liệu thực tế từ DB, từ chối bồi thường (`status="denied"`, `refund_vnd=0`), reply: *"Dạ, đơn O-202 chưa đủ điều kiện hoàn theo chính sách hiện hành."* |
| Regression test | Grader `DbFactKept` trong `src/lab9/graders.py` kết hợp test `test_workflow_ignores_delay_written_in_ticket` trong `src/tests/test_workflow_shape.py`. Grader phát hiện ngay event `model_override`: FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận kết quả đầu ra của quyết định tại runtime (event `model_override`), không phản ánh được chuỗi suy nghĩ nội tại (chain-of-thought) của model dẫn đến việc tin ticket. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
