# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Bùi Xuân Hòa
- **Student ID**: 2A202601202
- **Date**: 2026-07-28

---

## I. Technical Contribution (15 Points)

*Mô tả đóng góp cụ thể cho codebase.*

- **Modules Implementated**: 
  - [`docs/trace_eval.md`](file:///d:/Aithucchien/K4-Day-3-2A202601400/docs/trace_eval.md) (Báo cáo đánh giá Scoring Matrix & phân tích trace logs).
  - [`docs/hybrid_flowchart.mermaid`](file:///d:/Aithucchien/K4-Day-3-2A202601400/docs/hybrid_flowchart.mermaid) (Sơ đồ phân luồng Hybrid Flowchart).
- **Code Highlights**:
  - Thiết lập bảng Scoring Matrix phân tích 4 tiêu chí Agentic Fit cho chủ đề Trợ lý Tuyển dụng (đạt 19/20 điểm).
  - Lập tài liệu so sánh phản hồi chi tiết giữa Chatbot Baseline và ReAct Agent ở các câu hỏi điển hình (`TC-04`) và câu hỏi bẫy (`TC-05`).
- **Documentation**: Soạn sơ đồ Hybrid Flowchart bằng mã Mermaid thể hiện rõ ràng luồng định tuyến rẽ nhánh giữa Chatbot Baseline (câu hỏi lý thuyết tĩnh) và ReAct Agent Loop (câu hỏi yêu cầu gọi tool dữ liệu động).

---

## II. Debugging Case Study (10 Points)

*Phân tích một lỗi cụ thể gặp phải trong quá trình chạy thử hệ thống.*

- **Problem Description**: Ở phiên bản đầu tiên của ReAct Agent, khi gặp câu bẫy đặt lịch phỏng vấn lúc 3 giờ sáng (`TC-05`), Agent suy luận (`Thought`) rằng thời gian này không hợp lý nhưng bị ép bởi prompt phải gọi Action, dẫn đến tự bịa ra dòng: `Action: Không có hành động cụ thể tại thời điểm này...` (sai định dạng regex của Action) gây ra lỗi format hệ thống `LỖI: Phản hồi sai định dạng...`.
- **Log Source**: Nhật ký chạy thực tế được ghi lại tại [`docs/trace_eval.md#L315-L330`](file:///d:/Aithucchien/K4-Day-3-2A202601400/docs/trace_eval.md#L315-L330).
- **Diagnosis**: Do System Prompt ban đầu chưa định nghĩa rõ quy tắc xử lý khi dữ liệu đầu vào phi lý. LLM cố gắng phản hồi bằng ngôn ngữ tự nhiên trong phần Action thay vì gọi tool hợp lệ hoặc kết thúc bằng Final Answer.
- **Solution**: Cải tiến `REACT_SYSTEM_PROMPT` trong [`src/prompts.py`](file:///d:/Aithucchien/K4-Day-3-2A202601400/src/prompts.py): Yêu cầu Agent nếu phát hiện tham số phi lý thì lập tức chuyển sang Thought và kết thúc bằng `Final Answer` để báo lỗi cho người dùng, không được tự bịa Action.

---

## III. Personal Insights: Chatbot vs ReAct (10 Points)

*Suy nghĩ cá nhân về sự khác biệt khả năng suy luận.*

1.  **Reasoning**: Khối `Thought` đóng vai trò là "không gian suy nghĩ" của Agent. Nó giúp LLM lập kế hoạch rõ ràng trước khi hành động, tự kiểm chứng tính đúng đắn của tham số (như phát hiện ngày 30/02 không tồn tại) tốt hơn hẳn so với việc Chatbot sinh phản hồi trực tiếp không qua suy luận.
2.  **Reliability**: Agent có thể chạy kém hơn Chatbot trong trường hợp kết nối API gặp lỗi (ví dụ: lỗi giới hạn hạn mức rate limit `429` hoặc server lỗi `503`). Ngoài ra, chi phí gọi nhiều vòng lặp ReAct lớn hơn nhiều so với việc gọi trực tiếp 1 lần của Chatbot Baseline.
3.  **Observation**: Phản hồi từ môi trường (`Observation`) là bằng chứng thực tế giúp Agent "grounding" kiến thức của mình. Kết quả của bước trước quyết định rẽ nhánh logic tiếp theo (ví dụ: Nếu đánh giá CV đạt -> gọi tiếp tool hẹn lịch; Nếu CV trượt -> dừng lại và gửi thư từ chối).

---

## IV. Future Improvements (5 Points)

*Cải tiến hệ thống lên môi trường production chạy thực tế.*

- **Scalability**: Thiết kế cơ chế xử lý bất đồng bộ (Asynchronous Task Queue) cho các tác vụ phân tích file CV nặng để tránh chặn luồng chính.
- **Safety**: Xây dựng bộ lọc dữ liệu nhạy cảm (PII Guardrails) để ẩn các thông tin cá nhân của ứng viên trước khi chuyển văn bản CV lên LLM API của nhà cung cấp thứ ba.
- **Performance**: Xây dựng một classifier nhỏ (nhanh và rẻ) để phân loại yêu cầu ngay từ đầu trước khi khởi tạo ReAct Agent, giảm tải chi phí token cho các câu hỏi lý thuyết thông thường.

---

> [!NOTE]
> Submit this report by renaming it to `REPORT_BUI_XUAN_HOA.md` and placing it in this folder.

