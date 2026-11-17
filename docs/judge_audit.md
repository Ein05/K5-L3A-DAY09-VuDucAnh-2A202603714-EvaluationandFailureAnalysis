# Judge audit

Chạy: `python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5` và `python -m lab9.judge --csv data/eval/judge_helpful_public.csv --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Đạt (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | Dùng làm gate |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không đạt (κ=0.375 < 0.6, R=0.300 < 0.7 quá thấp) | Loại |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Reply: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* Xưng hô "anh - em" và từ "nha" mang tính cá nhân, suồng sã, thiếu tính chuyên nghiệp CSKH doanh nghiệp LogiX. **Hành động rubric:** Bổ sung ràng buộc nghiêm ngặt: xưng hô chuẩn phải là đại diện thương hiệu ("LogiX") và khách hàng ("anh/chị/quý khách"), cấm xưng hô thân mật tự phát. |
| 2 | H-017 | PASS | FAIL | Người đúng | Reply: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* Từ "bạn ơi" và "nha" là khẩu ngữ chat cá nhân. Judge bỏ sót vì thấy có chữ "Dạ". **Hành động rubric:** Phạt các khẩu ngữ chat mạng xã hội không trang trọng ("bạn ơi", "nha", "nhé bạn"). |
| 3 | H-029 | PASS | FAIL | Người đúng | Reply: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* Câu *"không cần nhắn thêm"* mang sắc thái cộc lốc, gắt gỏng, ngăn cản khách hàng tương tác. **Hành động rubric:** Cấm các phát ngôn từ chối tiếp nhận giao tiếp hoặc ra lệnh tiêu cực đối với khách hàng. |
| 4 | H-042 | FAIL | PASS | Người đúng | Reply: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* Câu trả lời ngắn gọn, lịch sự, có đầy đủ mốc thời gian nhận tiền. Judge LLM bắt lỗi nhầm (False Positive). **Hành động rubric:** Làm rõ trong rubric rằng câu trả lời ngắn gọn súc tích có mốc thời gian giải ngân là hợp lệ, không đánh rớt vì thiếu độ dài. |
| 5 | H-070 | PASS | FAIL | Người đúng | Reply: *"Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂"*. Chèn emoji cười nghiêng ngả "😂😂" trong văn cảnh bồi thường tài chính là thái độ cợt nhả, không tôn trọng sự cố của khách. Judge không nắm được ngữ cảnh châm biếm/cợt nhả của emoji. **Hành động rubric:** Cấm tuyệt đối emoji không phù hợp, chỉ cho phép text thuần hoặc emoji lịch thiệp nếu được định nghĩa trước. |
| 6 | H-094 | PASS | FAIL | Người đúng | Reply: *"Da, LogiX da hoan 40.000đ cho don O-341, xin loi vi su bat tien."* Tiếng Việt không dấu. Judge vẫn hiểu nghĩa nên cho PASS, nhưng người đánh FAIL vì chuẩn văn bản hành chính. **Hành động rubric:** Yêu cầu bắt buộc 100% tiếng Việt có dấu chuẩn chính tả. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` bị loại vì ma trận nhầm lẫn cho thấy:
  - `tp=9, fp=0, fn=21, tn=70`.
  - Recall cực thấp ($R = 9 / (9 + 21) = 30\%$), bỏ sót đến 70% số ca không hữu ích (ví dụ: câu reply lặp lại 2 lần liên tiếp lời xin lỗi mà không giải thích điều khoản, hoặc trả lời quá cộc lốc).
  - Độ đồng thuận Cohen's $\kappa = 0.375$ (thấp hơn nhiều so với ngưỡng tối thiểu $0.6$). Nếu dùng `helpful` làm gate CI, các phiên bản agent trả lời vô nghĩa hoặc lặp câu sẽ dễ dàng lọt qua gate.
- **Rủi ro còn lại:** Cả hai judge đều mới chỉ được kiểm định trên tập dữ liệu replay công khai ($n=100$). Cần thu thập thêm dữ liệu gán nhãn thực tế từ người dùng mới (held-out data) để chống overfitting vào bộ dữ liệu test hiện tại.
