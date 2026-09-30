# Hướng dẫn nộp bài (SUBMISSION)

## 1. Hình thức nộp bài
- Bài tập được thực hiện theo hình thức **cá nhân**.
- **Mỗi cá nhân phải tự nộp link repo của mình lên hệ thống LMS / Codelab** theo thông báo của giảng viên hoặc coach (mỗi học viên một repository riêng, không nộp hộ, không dùng chung repo).
- Repository phải được để ở chế độ Public (hoặc cấp quyền truy cập cho giảng viên / coach nếu được yêu cầu).

## 2. Quy chuẩn đặt tên Repository

Cấu trúc tên repository nộp bài:

```text
K4-L3A-DAY14-<HoVaTen>-<MSSV>-AIEvaluation
```

- `<HoVaTen>`: Họ và tên viết liền không dấu (PascalCase).
- `<MSSV>`: Mã số sinh viên chính xác.

**Ví dụ:**
```text
K4-L3A-DAY14-NguyenVanAn-L3A202600280-AIEvaluation
```

> ⚠️ **Lưu ý:** Đặt sai tên repository sẽ bị trừ **5 điểm** theo quy định trong [RUBRIC.md](RUBRIC.md).

## 3. Thành phần bài nộp (Deliverables)

| File | Yêu cầu |
|---|---|
| `solution/solution.py` | Hoàn thiện tất cả TODO bắt buộc |
| `golden_dataset.json` | Đủ 20 QA, đúng schema |
| `exercises.md` | worksheet, benchmark 3.2, rubric 3.3 |
| `reflection.md` | report, 3 failures, 5 Whys, regression |

Các file sinh ra trong quá trình chạy (artifacts) là tùy chọn (optional):
- `artifacts/actual_answers.json`
- `artifacts/benchmark_results.json`

> ⚠️ **CẢNH BÁO BẢO MẬT:** Tuyệt đối **KHÔNG commit** file `.env`, OpenAI API key hoặc bất kỳ thông tin bí mật nào lên GitHub repository. Vi phạm sẽ bị trừ **10 điểm**.

## 4. Nơi nộp và Hạn nộp (Deadline)
- **Nơi nộp:** Nộp link GitHub repository cá nhân lên LMS / Codelab.
- **Hạn chót mặc định:** **23h59 ngày lab (GMT+7)**.
- Coach có thể gia hạn tối đa không quá **48 giờ (≤48h)** đối với các trường hợp đặc biệt có lý do chính đáng được phê duyệt trước.

## 5. Checklist kiểm tra trước khi nộp

Hãy chạy các kiểm tra sau và tích chọn đầy đủ trước khi nộp bài:

- [ ] Repository đã được đặt đúng tên chuẩn: `K4-L3A-DAY14-<HoVaTen>-<MSSV>-AIEvaluation`.
- [x] Chạy `python validate_golden_dataset.py` báo `PASS`.
- [ ] Toàn bộ required tests pass khi chạy `pytest tests/ -v` (41 passed, 1 skipped nếu không làm bonus).
- [x] `golden_dataset.json` đủ 20 QA (5 Easy + 7 Medium + 5 Hard + 3 Adversarial).
- [x] Đã kiểm tra `artifacts/actual_answers.json` sau khi chạy RAG (`python domain_assistant.py`).
- [x] `exercises.md` đã hoàn thành đầy đủ: Exercise 3.2 có đủ năm metrics và ba cases thấp nhất; Exercise 3.3 có rubric 1–5 và edge cases (Exercise 3.4 & 3.5 nếu chọn làm bonus).
- [x] `reflection.md` có ba 5 Whys analyses, bảng failure taxonomy và improvement log / regression strategy.
- [x] `solution/solution.py` là bản hoàn thiện của `template.py` (học viên đã copy sau khi hoàn thành code).
- [ ] Không commit `.env`, API key hoặc dữ liệu nhạy cảm lên GitHub.

---

## Tài liệu liên quan
- [README.md](README.md) — Tổng quan bài lab và hướng dẫn khởi động
- [RUBRIC.md](RUBRIC.md) — Tiêu chí chấm điểm chi tiết và các trường hợp trừ điểm
- [CHECKPOINTS.md](CHECKPOINTS.md) — Hướng dẫn từng checkpoint và tiêu chuẩn nghiệm thu
- [RULES.md](RULES.md) — Quy định làm bài, sử dụng AI và bảo mật


## Kết quả rà soát CP5

- Benchmark đang dùng: artifact ngày 2026-09-30T08:34:50.246229+00:00, pass rate **45% (9/20)**; worksheet và reflection đã đồng bộ. Điểm benchmark thấp không đồng nghĩa code/tests fail hoặc bài lab thiếu theo RUBRIC.
- Dataset validator: **PASS**, 20 QA đúng tỷ lệ 5/7/5/3 và coverage 10/10 tài liệu.
- Đã đọc cú pháp Python bằng AST và đối chiếu `solution/solution.py` trùng `template.py`. Chỉ bonus reranking còn `NotImplementedError`; các comment TODO được giữ theo yêu cầu.
- **Chưa chạy pytest theo yêu cầu người dùng**, không tích mục required tests pass. Người học cần tự chạy `.venv/bin/python -m pytest tests/ -v` và xác nhận kỳ vọng 41 passed, 1 skipped.
- Exercise 1.1–1.3, 3.1–3.3 và reflection đã có nội dung. Exercise 3.4–3.5 là bonus không chọn.
- Tên thư mục local đúng cấu trúc `K4-L3A-DAY14-NgoMinhTri-2A202602993-AIEvaluation`; người học cần xác nhận MSSV và tên repo GitHub thực tế, quyền Public/quyền coach và tự nộp link lên LMS. Chưa thực hiện commit/push/nộp bài.
- `.env` không được Git track; không tìm thấy giá trị secret đang cấu hình trong các file tracked ở working tree. Đây không phải kiểm chứng toàn bộ lịch sử Git hoặc repo remote, nên vẫn cần rà trước commit/push.
- Nội dung phân tích có hỗ trợ AI: người học cần tự đọc, sửa theo nhận định cá nhân và hiểu rõ trước nộp, theo RULES.md. Các đề xuất cải tiến và production gates chưa được triển khai/đo lại.
