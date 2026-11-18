# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

## Case audit thực tế: C-002@11

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `E-seeded_regression-20261117-ea6d` |
| Triệu chứng | Grader `pass_rate` FAIL (status `resolved` != `denied`), `db_fact_kept` FAIL, `refund_amount_correct` FAIL (ledger ghi nhận 20.000đ trong khi nhãn kỳ vọng là 0đ). Reply: *"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."* |
| Bước sai đầu tiên | Event 3: `model_override {"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) đúng thẩm quyền và scope; Event 2 (`investigator`) tra cứu dữ liệu gốc từ DB hoàn toàn chính xác (`visible: True, order_id: "O-202", delay_days: 2`). Đến Event 3, agent bất ngờ ghi đè dữ liệu tin cậy của DB bằng số ngày 5 do khách tự khai trong ticket. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Cùng claim C-002@11 trên shape `fixed`: Không có event `model_override`. Event 3 là `quote` với `amount_vnd: 0, reason: "no_entitlement"`. Event 4 trả lời từ chối bồi thường hợp lệ. Ledger ghi 0đ, status `denied` (PASS). |
| Regression test | `test_override_event_breaks_db_fact` (kiểm tra grader `DbFactKept`) và `RefundAmountCorrect` trong `tests/test_lab9_graders.py`. Đã xác nhận: FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi override sau bước parse của MockLLM; chưa chứng minh được trong thực tế nếu dùng LLM sinh văn bản tự do thì agent có bị prompt injection dẫn dắt thêm hành vi nào khác hay không. |

## Quy tắc áp dụng
- Đã đối chiếu **ledger trước reply**: phát hiện ledger bị ghi nhận bồi hoàn trái phép 20.000đ dù đơn chỉ trễ 2 ngày (không đủ điều kiện bồi hoàn).
- Bước sai đầu tiên được dẫn chứng chính xác bằng Event số 3 trong execution trace.
