# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | **dùng** |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375 < 0.6, R=0.300 < 0.7, bỏ sót 21 ca lỗi) | **loại** |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa Rubric |
|---|---|---|---|---|---|
| 1 | H-008 (`judge_tone`) | PASS | FAIL | Người đúng | Reply: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* – Xưng hô thân mật suồng sã ("anh - em", "nha", "check") không đạt chuẩn mực giao tiếp doanh nghiệp. **Sửa rubric:** Bổ sung tiêu chí cấm văn phong mạng xã hội, tiếng lóng và đại từ xưng hô phi chuẩn mực trong hỗ trợ khách hàng. |
| 2 | H-017 (`judge_tone`) | PASS | FAIL | Người đúng | Reply: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* – Cụm từ "bạn ơi", "rồi nha" làm giảm tính nghiêm túc của dịch vụ tài chính. **Sửa rubric:** Quy định cụ thể định dạng xưng hô chuẩn ("quý khách", "anh/chị") và loại bỏ thán từ xuề xòa ở đuôi câu. |
| 3 | H-029 (`judge_tone`) | PASS | FAIL | Người đúng | Reply: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* – Cụm *"không cần nhắn thêm"* mang sắc thái cộc cằn, xua đuổi khách hàng. **Sửa rubric:** Thêm quy tắc cấm ngôn từ phủ định gây ức chế hoặc tạo cảm giác đóng kênh giao tiếp thô bạo. |
| 4 | H-042 (`judge_tone`) | FAIL | PASS | Người đúng | Reply: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* – Câu trả lời ngắn gọn, lịch sự, nêu rõ mốc thời gian hoàn tiền; Judge khắt khe phạt nhầm (False Positive). **Sửa rubric:** Nêu rõ câu trả lời đi thẳng vào giải pháp và có timeline không bị tính là lỗi tone cộc lốc. |
| 5 | H-051 (`judge_tone`) | FAIL | PASS | Người đúng | Reply: *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* – Chèn từ tiếng Anh "refund", nhưng vẫn giữ thái độ nhã nhặn ("Dạ... ạ"). Judge phạt nhầm tone. **Sửa rubric:** Phân tách rõ ràng giữa tiêu chí ngôn ngữ thuần Việt với tiêu chí thái độ phục vụ (tone). |
| 6 | HL-016 (`judge_helpful`) | PASS | FAIL | Người đúng | Reply: *"Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này..."* – Xin lỗi chung chung nhưng không đưa ra bất kỳ hành động giải quyết hay hướng dẫn cụ thể nào. Judge bị đánh lừa bởi độ lịch sự nên cho PASS, dẫn đến Recall cực thấp (0.3). **Sửa rubric:** Cấm đánh giá PASS cho các phản hồi mang tính "xin lỗi suông" thiếu next steps. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` bị loại hoàn toàn khỏi CI gate do chỉ đạt $\kappa = 0.375$ (dưới sàn 0.6) và Recall chỉ đạt $0.300$ (dưới sàn 0.7). Ma trận nhầm lẫn của `helpful` là `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}` cho thấy Judge bỏ sót tới 21 trên tổng số 30 trường hợp câu trả lời không hữu ích (False Negative rate lên tới 70%). Nếu dùng judge này làm gate, các phản hồi vô nghĩa/xin lỗi suông vẫn sẽ dễ dàng lọt qua kiểm duyệt.
- **Tiêu chí được chấp thuận:** `tone_ok` được chấp thuận vì đạt $\kappa = 0.757 \ge 0.6$, Precision $= 0.857 \ge 0.8$, và Recall $= 0.800 \ge 0.7$. Tỉ lệ lỗi quan sát $28\%$ sau khi hiệu chỉnh bằng Rogan-Gladen là $30\%$.
- **Rủi ro còn lại:** Judge hiện tại mới chỉ được hiệu chuẩn trên tập dữ liệu replay công khai (100 mẫu). Cần liên tục thu thập nhãn từ chuyên viên vận hành thực tế (human held-out data) để re-calibrate định kỳ, tránh hiện tượng drift khi prompt của Agent thay đổi.
