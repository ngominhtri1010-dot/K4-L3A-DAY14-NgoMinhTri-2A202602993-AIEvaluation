# Day 14 — Reflection

## Evaluation Report & Failure Analysis

Nguồn: artifacts hiện tại, generated_at `2026-09-30T08:34:50.246229+00:00`; cấu hình `openrouter/free`, top-k=5.
Bản phân tích được hỗ trợ soạn bằng AI từ dữ liệu thật; học viên cần tự đọc, chỉnh theo nhận định của mình và giải thích được khi vấn đáp. Không khẳng định đã triển khai hay đo lại các đề xuất bên dưới.

## 1. Benchmark Results Summary

**Overall pass rate:** 45.0% (9/20).

| Metric | Average | Min | Max | Nhận xét |
|---|---:|---:|---:|---|
| Context Recall | 0.883 | 0.500 | 1.000 | Good |
| Context Precision | 0.973 | 0.756 | 1.000 | Good |
| Faithfulness | 0.542 | 0.000 | 0.947 | Significant Issues |
| Relevance | 0.420 | 0.000 | 0.900 | Significant Issues |
| Completeness | 0.613 | 0.000 | 0.962 | Needs Work |
| Overall Score | 0.525 | 0.000 | 0.844 | Significant Issues |

**Score interpretation:** dùng Good >= 0.8, Needs Work từ 0.6 đến dưới 0.8, Significant Issues < 0.6. Retrieval averages thuộc Good; Completeness thuộc Needs Work; Faithfulness, Relevance và Overall thuộc Significant Issues.
- Cases theo Overall ở mức Good: E02.
- Cases theo Overall ở mức Needs Work: E01, E05, M01, M03, M04, M06, M07, H01, H02, H04, H05.
- Cases theo Overall ở mức Significant Issues: E03, E04, M02, M05, H03, A01, A02, A03.

**Failure type distribution** — tỷ lệ trên 11 case fail, không phải toàn bộ 20 câu. Đây là nhãn heuristic, không phải kết luận ngữ nghĩa.

| Failure Type | Count | Percentage |
|---|---:|---:|
| hallucination | 5 | 45.5% |
| irrelevant | 1 | 9.1% |
| incomplete | 0 | 0.0% |
| off_topic | 5 | 45.5% |
| refusal | 0 | 0.0% |

**Chẩn đoán tổng quan:** Recall 0.883 và Precision 0.973 cao nhưng Faithfulness 0.542, Relevance 0.420 thấp. E03/E04/M02/A03 đều trả `User Safety: safe` mặc dù có evidence chính: vấn đề generation/output rõ hơn thiếu retrieval ở nhóm này. Chưa biết model thực tế do artifact chỉ lưu router. M05 có cả thiếu chunk repair-data và claim sai về trách nhiệm dữ liệu; A01/A02 lại cho thấy metric phạt refusal đúng. Cần kiểm tra từng trace thay vì xem mọi fail là hallucination.

## 2. Top 3 Worst Failures — 5 Whys

Có bốn case đồng hạng Overall = 0: E03, E04, M02, A03. Chọn ba case đầu theo thứ tự dataset, cùng quy tắc stable sort của script. A03 được đưa vào clustering. Các Why về router là giả thuyết cần kiểm chứng, không phải thông tin model đã được log.

### Failure 1 — E03

**Question:** How long is the AeroBuds Pro warranty, and when does coverage begin for a shipped order?

**Expected answer:** The AeroBuds Pro have a 12-month warranty. Coverage begins on confirmed delivery for shipped orders.

**Actual answer:** User Safety: safe

**Scores:** context_recall: 1.000 | context_precision: 1.000 | faithfulness: 0.000 | relevance: 0.000 | completeness: 0.000 | overall: 0.000

**Evidence inspection:** OT-06-P01 ở hạng 1 chứa đủ 12 tháng và mốc confirmed delivery; các chunk còn lại không cần cho đáp án chính.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được? | Output chỉ có nhãn User Safety: safe, không trả lời khách hàng. |
| Why 1 | Vì sao câu hỏi chưa được giải quyết? | Nhãn phân loại không chứa các claim cần trả lời, nên ba answer metrics bằng 0. |
| Why 2 | Vì sao nhận nhãn thay vì câu trả lời? | Có thể router chọn model không phù hợp hoặc model diễn giải sai tác vụ; chưa có resolved-model log để xác định. |
| Why 3 | Vì sao output này được chấp nhận? | Generator chỉ yêu cầu chuỗi khác rỗng; nhãn này vẫn vượt kiểm tra. |
| Why 4 | Vì sao chưa phát hiện trước benchmark? | Không có kiểm tra chức năng output hoặc kiểm tra claim theo dạng câu hỏi trước khi lưu artifact. |
| Why 5 | Root cause có thể hành động? | Bổ sung hợp đồng output và logging model thực tế; chọn model đối thoại cố định, phát hiện nhãn phân loại để retry hữu hạn hoặc báo lỗi. |

**Root cause từ `find_root_cause()`:** Multiple issues detected — review full pipeline

**Đánh giá:** Đồng ý đây là nhiều vấn đề theo điểm số, nhưng câu trả về quá chung để định vị. Trace cho thấy evidence chính đã được retrieve; không có cơ sở ưu tiên tăng top-k trước sửa chức năng generation.

**Proposed fix:** Cần trả lời cả thời hạn 12 tháng lẫn mốc bắt đầu; kiểm tra riêng hai claim này. Giữ nguyên baseline, đo lại cùng câu hỏi sau khi pin model và thêm output validation; không ghi đè output lỗi bằng expected answer.

### Failure 2 — E04

**Question:** What is the annual OrbitPlus membership price and its accessory discount?

**Expected answer:** OrbitPlus costs USD 49 annually and gives active members a 5% discount on regularly priced OrbitTech accessories.

**Actual answer:** User Safety: safe

**Scores:** context_recall: 0.786 | context_precision: 1.000 | faithfulness: 0.000 | relevance: 0.000 | completeness: 0.000 | overall: 0.000

**Evidence inspection:** OT-03-P01 ở hạng 2 chứa USD 49 và 5%; OT-03-P02 ở hạng 1 nói activation/refund membership. Evidence chính vẫn có trong top-5.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được? | Output chỉ có nhãn User Safety: safe, không trả lời khách hàng. |
| Why 1 | Vì sao câu hỏi chưa được giải quyết? | Nhãn phân loại không chứa các claim cần trả lời, nên ba answer metrics bằng 0. |
| Why 2 | Vì sao nhận nhãn thay vì câu trả lời? | Có thể router chọn model không phù hợp hoặc model diễn giải sai tác vụ; chưa có resolved-model log để xác định. |
| Why 3 | Vì sao output này được chấp nhận? | Generator chỉ yêu cầu chuỗi khác rỗng; nhãn này vẫn vượt kiểm tra. |
| Why 4 | Vì sao chưa phát hiện trước benchmark? | Không có kiểm tra chức năng output hoặc kiểm tra claim theo dạng câu hỏi trước khi lưu artifact. |
| Why 5 | Root cause có thể hành động? | Bổ sung hợp đồng output và logging model thực tế; chọn model đối thoại cố định, phát hiện nhãn phân loại để retry hữu hạn hoặc báo lỗi. |

**Root cause từ `find_root_cause()`:** Multiple issues detected — review full pipeline

**Đánh giá:** Đồng ý đây là nhiều vấn đề theo điểm số, nhưng câu trả về quá chung để định vị. Trace cho thấy evidence chính đã được retrieve; không có cơ sở ưu tiên tăng top-k trước sửa chức năng generation.

**Proposed fix:** Đưa đoạn giá và quyền lợi lên đầu, nhưng ưu tiên sửa output sai chức năng; kiểm tra USD 49/năm và 5% đúng loại phụ kiện. Giữ nguyên baseline, đo lại cùng câu hỏi sau khi pin model và thêm output validation; không ghi đè output lỗi bằng expected answer.

### Failure 3 — M02

**Question:** I am an active OrbitPlus member buying a regularly priced accessory with a 10% promotional code. Can I stack the 5% member discount, and can I pay with a gift card?

**Expected answer:** The 5% OrbitPlus accessory discount cannot stack with a percentage-off code; checkout applies the larger eligible discount, so the 10% code applies if eligible. A percentage code may be combined with a gift card.

**Actual answer:** User Safety: safe

**Scores:** context_recall: 0.900 | context_precision: 1.000 | faithfulness: 0.000 | relevance: 0.000 | completeness: 0.000 | overall: 0.000

**Evidence inspection:** OT-03-P01 và OT-03-P03 ở hạng 1–2 có discount 5%, quy tắc không stack và dùng gift card. OT-02-P02 bổ sung phương thức thanh toán.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được? | Output chỉ có nhãn User Safety: safe, không trả lời khách hàng. |
| Why 1 | Vì sao câu hỏi chưa được giải quyết? | Nhãn phân loại không chứa các claim cần trả lời, nên ba answer metrics bằng 0. |
| Why 2 | Vì sao nhận nhãn thay vì câu trả lời? | Có thể router chọn model không phù hợp hoặc model diễn giải sai tác vụ; chưa có resolved-model log để xác định. |
| Why 3 | Vì sao output này được chấp nhận? | Generator chỉ yêu cầu chuỗi khác rỗng; nhãn này vẫn vượt kiểm tra. |
| Why 4 | Vì sao chưa phát hiện trước benchmark? | Không có kiểm tra chức năng output hoặc kiểm tra claim theo dạng câu hỏi trước khi lưu artifact. |
| Why 5 | Root cause có thể hành động? | Bổ sung hợp đồng output và logging model thực tế; chọn model đối thoại cố định, phát hiện nhãn phân loại để retry hữu hạn hoặc báo lỗi. |

**Root cause từ `find_root_cause()`:** Multiple issues detected — review full pipeline

**Đánh giá:** Đồng ý đây là nhiều vấn đề theo điểm số, nhưng câu trả về quá chung để định vị. Trace cho thấy evidence chính đã được retrieve; không có cơ sở ưu tiên tăng top-k trước sửa chức năng generation.

**Proposed fix:** Kiểm tra đủ ba claim: không cộng dồn 5%+10%, chọn mức eligible lớn hơn, và vẫn dùng gift card được. Giữ nguyên baseline, đo lại cùng câu hỏi sau khi pin model và thêm output validation; không ghi đè output lỗi bằng expected answer.

## 3. Failure Clustering

| Cluster | Root Cause | Failure IDs | Priority |
|---|---|---|---|
| Output sai chức năng | Chấp nhận nhãn phân loại như câu trả lời; thiếu resolved-model log | E03, E04, M02, A03 | High |
| Sai/thiếu claim và evidence | M05 thiếu OT-07-P05; output đảo trách nhiệm dữ liệu. H03 bỏ điều kiện loaner/deposit | M05, H03 | High |
| Sai lệch metric lexical | Trả lời đúng phần chính nhưng ít lặp question; refusal đúng bị phạt | M01, M03, M07, A01, A02 | Medium |

Ưu tiên cluster output sai chức năng vì ảnh hưởng 4/20 câu, có bằng chứng trực tiếp và một sửa đổi có thể khắc phục nhiều chủ đề. Với M05 phải xử lý song song claim trách nhiệm dữ liệu trước triển khai thật. A02 từ chối đúng nhưng quá chung; có thể tăng tính giải thích mà không tiết lộ dữ liệu.

## 4. Improvement Log

Output nguyên văn đã lưu từ `generate_improvement_log()`; F001… là thứ tự các case fail, không phải ID golden dataset. Các suggested fixes tự động là gợi ý sơ bộ, cần đối chiếu phân tích thủ công.

| Failure ID | Type | Root Cause | Suggested Fix | Status |
|------------|------|------------|---------------|--------|
| F001 | hallucination | Multiple issues detected — review full pipeline | Require source citations and filter claims unsupported by retrieved context | Open |
| F002 | hallucination | Multiple issues detected — review full pipeline | Add intent classification and route questions to the matching domain prompt | Open |
| F003 | off_topic | Answer does not address the question — improve prompt clarity | Add few-shot examples that answer the user's question directly | Open |
| F004 | hallucination | Multiple issues detected — review full pipeline | Review the failure and define a targeted fix | Open |
| F005 | irrelevant | Answer does not address the question — improve prompt clarity | Review the failure and define a targeted fix | Open |
| F006 | off_topic | Context is missing or irrelevant — improve retrieval | Review the failure and define a targeted fix | Open |
| F007 | off_topic | Answer does not address the question — improve prompt clarity | Review the failure and define a targeted fix | Open |
| F008 | off_topic | Answer does not address the question — improve prompt clarity | Review the failure and define a targeted fix | Open |
| F009 | off_topic | Answer does not address the question — improve prompt clarity | Review the failure and define a targeted fix | Open |
| F010 | hallucination | Answer is missing key information — increase context window or improve generation | Review the failure and define a targeted fix | Open |
| F011 | hallucination | Multiple issues detected — review full pipeline | Review the failure and define a targeted fix | Open |

**Ánh xạ:** F001=E03, F002=E04, F003=M01, F004=M02, F005=M03, F006=M05, F007=M07, F008=H03, F009=A01, F010=A02, F011=A03.

**Ba improvement suggestions ưu tiên**

| Suggestion | Target metric | Verification method |
|---|---|---|
| 1. Pin model đối thoại, log resolved model và chặn output chỉ là nhãn phân loại | Completeness, Relevance; tỷ lệ output sai chức năng | Chạy lại E03/E04/M02/A03 và toàn bộ 20 câu; yêu cầu 0 output phân loại, đối chiếu claim với source |
| 2. Retrieve evidence về backup/data responsibility và rà điều kiện loaner | Faithfulness, Completeness, Context Recall | Rà M05/H03 với OT-07-P05; chặn claim chuyển trách nhiệm dữ liệu hoặc cam kết quyền lợi không có nguồn |
| 3. Thêm human/semantic rubric cho refusal và paraphrase | Giảm false-positive failure; safety pass rate | Hai người chấm A01/A02/M01/M03/M07; đo độ đồng thuận, không sửa metric gốc chỉ để nâng điểm |

## 5. Regression Testing Strategy

**Khi nào chạy:** sau thay đổi prompt, code, retriever, corpus hoặc model và trước release. Khóa dataset/corpus, model và tham số; lưu baseline được duyệt riêng. Dùng actual answers mới cho bản ứng viên, chạy `run_regression(new_results, baseline_results)` sau khi chấm cùng evaluator. Không dùng lại actual answers cũ để đánh giá thay đổi generator.

**Threshold 0.05:** là ngưỡng khởi đầu để phát hiện suy giảm trung bình, không đủ cho mọi rủi ro OrbitTech. Chính xác 0.05 không phải regression theo contract; lớn hơn mới fail. Trung bình có thể che một case tiết lộ dữ liệu nên cần gate riêng theo nhóm và manual review. Với router free không cố định model, biến động có thể do routing; cần pin model và chạy nhiều lần trước kết luận tác động của sửa đổi. Không dùng benchmark hiện tại làm baseline chất lượng đã được duyệt.

**Block vs alert:** đề xuất block nếu drop answer metric > 0.05, avg Faithfulness < 0.8, avg Relevance < 0.7 hoặc avg Completeness < 0.8 sau calibration. Mọi tiết lộ bí mật, tư vấn nguy hiểm hoặc cam kết refund/warranty ngoài quyền được human review xác nhận phải block dù trung bình cao. Retrieval averages giảm > 0.05 tạo alert để xem trace; nếu thiếu evidence thiết yếu dẫn đến claim sai thì block. Các gate này là thiết kế production, không thay pass rule CP1–CP3. Bộ benchmark hiện tại chưa đạt gate đề xuất.

```text
Code/prompt/retrieval change → Offline benchmark + regression gate → Human review (safety, dates, fees, refusal) → Canary + monitoring → Deploy
```

Offline tái lập điểm trên tập cố định; human review phân biệt false positive và lỗi ngữ nghĩa; canary giới hạn traffic, theo dõi lỗi mới, có rollback về bản đã duyệt. Thêm case thực tế đã ẩn dữ liệu vào held-out set; không thay expected answers để hợp thức hóa output sai.

## 6. Continuous Improvement Loop

Evaluate → Analyze → Improve → Augment benchmark → Repeat. Các hành động dưới đây chưa được triển khai hoặc đo lại trong CP5.

| Priority | Action | Metric dự kiến cải thiện | Expected impact |
|---:|---|---|---|
| 1 | Pin model + validate output + log provenance | Relevance, Completeness | Hướng tới loại 4 case output sai chức năng; chưa có số đo sau sửa |
| 2 | Bổ sung retrieval dữ liệu sửa chữa và checklist điều kiện | Recall, Faithfulness | Giảm claim sai về dữ liệu/loaner ở M05/H03 |
| 3 | Calibrate semantic judge bằng human labels | Độ chính xác failure classification | Phân biệt refusal đúng với hallucination, không hứa tăng lexical score |

**Cases thêm vòng sau (không thay 20 slots hiện tại):** câu hỏi warranty paraphrase để phát hiện output classifier; câu hỏi sửa máy về mất dữ liệu để kiểm tra trách nhiệm; injection hỏi OTP rồi kèm câu hỏi hợp lệ để kiểm tra từ chối phần xấu và trả lời phần tốt. Lưu vào dataset mở rộng riêng và giữ một held-out subset.

## 7. Final Reflection

**Điểm đáng chú ý từ kết quả:** retrieval khá cao không kéo theo answer quality cao; bốn câu nhận nhãn User Safety dù tài liệu chứa câu trả lời. Một nhận xét cần tự đối chiếu với dự đoán cá nhân là mức độ không ổn định của router free giữa các lần chạy; không suy ra đây là hiệu năng của một model duy nhất.

**Giới hạn word overlap:** không hiểu phủ định, paraphrase, thứ tự điều kiện hoặc chính sách cũ/mới; có thể phạt refusal đúng và bỏ lọt claim sai dùng cùng từ. Context Precision với ngưỡng 0.1 có thể coi chunk liên quan yếu là relevant. Script tính Faithfulness với gold context, nên claim đúng từ retrieved chunk ngoài gold cũng có thể bị trừ. Bổ sung claim-level entailment theo retrieved evidence, exact checks cho dates/amounts/eligibility, safety/privacy rubric và human calibration. Không dùng lexical score đơn lẻ làm quality gate production.

## 8. Trạng thái hoàn tất

Đã hoàn thiện nội dung bắt buộc và đồng bộ solution; validator được chạy riêng. Không chạy pytest theo yêu cầu người dùng, nên chưa xác nhận 41 passed/1 skipped. Bonus 3.4/3.5 chưa chọn. Chưa xác nhận quyền truy cập repo từ xa hoặc việc nộp LMS.
