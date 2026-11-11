# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Lâm Tùng
MSSV: 2A202603854

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent:
"Hiện tại, GitHub Actions không có bất kỳ sự cố nào đang hoạt động, và tất cả hệ thống đều hoạt động bình thường theo thông tin cập nhật mới nhất từ Github Status. 

Tuy nhiên, danh sách các sự cố đã diễn ra gần đây liên quan đến GitHub Actions, được cập nhật tính đến ngày 09 tháng 11 năm 2026, bao gồm một số sự cố đã được khắc phục:

1. **Incident ID: pfbmpxxfq1cy** - Degraded performance for multiple GitHub services (Resolved on 2026-11-06)
2. **Incident ID: t89g2q1800fr** - Incident with Actions (Resolved on 2026-11-05)
3. **Incident ID: h02n9572mx9y** - Incident with Actions (Resolved on 2026-11-07)
4. **Incident ID: xr3myzj7ms51** - Incident with Actions and Pages (Resolved on 2026-11-07)
5. **Incident ID: smtdswd1h2w4** - Incident with Pull Requests, Git Operations and Issues (Resolved on 2026-11-04)

Tuy nhiên, dữ liệu này chỉ dựa trên các sự cố trong 50 incident gần nhất, và không có sự cố nào được ghi nhận cụ thể vào năm 2025. Điều này có thể có nghĩa là không có sự cố quan trọng nào xảy ra trong năm đó, nhưng thiếu bằng chứng xác thực cho điều này.

### Kết luận:
Danh sách sự cố GitHub Actions trong năm 2025 không có trong các kết quả truy xuất, và chỉ có những sự cố liên quan trong thời gian gần đây (2026). Từ dữ liệu hiện có, không thể xác nhận độ đầy đủ của các sự cố trong năm 2025. Bạn có thể xem thêm thông tin tại [GitHub Status](https://www.githubstatus.com)."
- Điểm ở lần chạy 1: 60/100 (factuality: 20/40, completeness: 20/25, evidence: 15/20, boundaries: 5/15)

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```json
{
  "event": "tool",
  "step": 1,
  "name": "get_status",
  "arguments": {
    "provider": "github",
    "component": "Actions"
  },
  "result": {
    "provider": "github",
    "captured_at": "2026-11-09T03:50:47.065020+00:00",
    "overall": {
      "indicator": "none",
      "description": "All Systems Operational"
    },
    "components": [
      {
        "name": "Actions",
        "status": "operational"
      }
    ],
    "incidents": [
      {
        "id": "pfbmpxxfq1cy",
        "name": "Degraded performance for multiple GitHub services",
        "status": "resolved",
        "updated_at": "2026-11-06T00:20:49.369Z"
      },
      {
        "id": "t89g2q1800fr",
        "name": "Incident with Actions",
        "status": "resolved",
        "updated_at": "2026-11-05T17:42:35.455Z"
      },
      {
        "id": "h02n9572mx9y",
        "name": "Incident with Actions",
        "status": "resolved",
        "updated_at": "2026-11-07T01:10:18.506Z"
      },
      {
        "id": "xr3myzj7ms51",
        "name": "Incident with Actions and Pages",
        "status": "resolved",
        "updated_at": "2026-11-07T01:04:38.647Z"
      },
      {
        "id": "smtdswd1h2w4",
        "name": "Incident with Pull Requests, Git Operations and Issues",
        "status": "resolved",
        "updated_at": "2026-11-04T11:50:06.459Z"
      }
    ],
    "matching_incidents_in_last_50": 23,
    "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
    "source": "https://www.githubstatus.com"
  }
}
```

**Nguyên nhân có thể xảy ra:**
- Tool `get_status` trả về trường `note` ghi rõ: *"Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố"*, và dữ liệu snapshot được chụp ngày 09/11/2026.
- Khi người dùng hỏi danh sách đầy đủ năm 2025, agent nhận được 5 sự cố gần nhất (đều rơi vào tháng 11/2026). Thay vì khẳng định ngay là snapshot chỉ chứa 50 incident gần nhất nên **không thể chứng minh danh sách đầy đủ cho năm 2025**, agent lại tự suy đoán chủ quan: *"Điều này có thể có nghĩa là không có sự cố quan trọng nào xảy ra trong năm đó"*.
- Lỗi này vi phạm tiêu chuẩn boundaries (chỉ được 5/15 điểm) và factuality (20/40 điểm) vì LLM tự suy diễn sự vắng mặt của thông tin trong 50 bản ghi thành kết luận "có thể không có sự cố trong năm 2025".

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu:
  1. Trong `agent.py`, tinh chỉnh `SYSTEM_PROMPT` nhằm siết chặt nguyên tắc ranh giới dữ liệu (boundaries): Khi người dùng yêu cầu danh sách đầy đủ cho một mốc thời gian trong quá khứ hoặc khoảng thời gian không nằm trong phạm vi hiển thị, agent phải nêu rõ ràng giới hạn của công cụ (chỉ tìm trong tối đa 50 incident gần nhất), khẳng định không thể chứng minh danh sách đầy đủ cho thời gian đó và tuyệt đối không được suy đoán rằng "không có sự cố xảy ra".
  2. Kết hợp cơ chế kiểm tra nguồn (Source validation / Evidence Gate): bắt buộc đối chiếu URL incident thật khi hoàn thành câu trả lời, ghi event `validation_missing_source` nếu thiếu nguồn hợp lệ.
- Vì sao chọn thay đổi này:
  1. Khắc phục triệt để lỗi suy đoán sai thực tế (hallucination / over-inference) mà LLM Judge đã chỉ ra trong `grade.reason` và `grade.improvement` của case A5.
  2. Đảm bảo tuân thủ đúng hợp đồng bài lab và pass toàn bộ bộ unit test `tests/test_validation.py`.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 78.68 / 100 | 78.05 / 100 |
| Điểm case đã chọn (A5) | 60 / 100 (Boundaries: 5/15) | 60 / 100 (Boundaries: 10/15) |
| Số model calls | 50 calls | 49 calls |
| Token, chi phí nếu có | 41,870 tokens ($0.01043) | 41,993 tokens ($0.010289) |

- Lỗi đã giải quyết:
  - Ở case A5, tiêu chí ranh giới dữ liệu (Boundaries) đã tăng từ 5/15 lên 10/15 điểm. Agent đã chủ động nêu rõ giới hạn của công cụ (chỉ lưu tối đa 50 incident gần nhất) và không còn tự ý đưa ra phán đoán tùy tiện như *"Điều này có thể có nghĩa là không có sự cố quan trọng nào xảy ra"*.
  - Các case kiểm tra tính chính xác của bằng chứng và phạm vi dữ liệu có sự cải thiện đáng kể: A1 (tăng từ 83 lên 100 điểm), A8 (tăng từ 75 lên 100 điểm), E1 (tăng từ 65 lên 75 điểm), A6 (tăng từ 70 lên 80 điểm), H7 (tăng từ 65 lên 75 điểm), H8 (tăng từ 80 lên 90 điểm).
  - Tối ưu số lần gọi model (giảm từ 50 xuống 49 model calls) và chi phí thực thi vẫn được duy trì ở mức tối ưu (~$0.01 cho cả bộ 22 case).

- Lỗi còn lại hoặc mới xuất hiện:
  - Ở case A5: Dù đã cải thiện ranh giới (Boundaries), nhưng agent vẫn còn câu chữ diễn đạt *"không có incident nào xảy ra trong năm 2025"* trước khi giải thích giới hạn snapshot ở cuối câu, khiến Judge đánh giá agent vẫn có phần phát ngôn chưa tuyệt đối chặt chẽ.
  - Xuất hiện biến động điểm số ở một vài case do tính chất ngẫu nhiên của LLM Judge (ví dụ H2 từ 100 xuống 50 điểm, A4 từ 100 xuống 70 điểm), dù nội dung trả lời đều trích xuất đúng sự kiện và ID.

## 5. Giới hạn

- **Giới hạn của LLM Judge:** LLM Judge (ở đây là `openai/gpt-4o-mini`) là một cơ chế đánh giá xấp xỉ, có tính thiên lệch (bias) và độ ngẫu nhiên nhất định giữa các lần sinh text (chẳng hạn cùng một câu trả lời đúng trạng thái của H2 nhưng một lần được 100 điểm, lần sau lại bị trừ 50 điểm do tiêu chí câu chữ diễn đạt). Do đó, điểm số Judge chỉ mang tính tham khảo và cần được đối chiếu trực tiếp với log trace, mechanical checks và rubric chấm code.
- **Giới hạn của dữ liệu Snapshot:** Snapshot bài tập được cố định tại thời điểm ngày 09/11/2026 và chỉ lưu 50 incident gần nhất. Dữ liệu này không thể đại diện cho trạng thái thời gian thực và việc tìm kiếm rỗng không chứng minh sự cố không tồn tại trong thực tế.
- **Giới hạn của việc chỉ chạy một lần mỗi phiên bản:** Một lần chạy duy nhất không thể chứng minh toàn diện độ ổn định (variance) hay tính bền vững của agent trong môi trường production. Để đánh giá chính xác cần chạy nhiều lần (multi-seed evaluation) và lấy giá trị trung bình kèm khoảng tin cậy.

## 6. Ghi chú Thử thách Bonus (Tùy chọn)

- **Kết quả tổng quan:** Hoàn thành toàn bộ 10/10 case trong `eval_bonus.json`, điểm trung bình **61.5/100**, điểm bonus quy đổi: **9.22 / 15 điểm**.
- **Cơ chế Retry & Xử lý 429 (Rate Limit):**
  - Ở case B3 (đạt 65 điểm): Agent đã nhận biết mã lỗi 429 giả lập từ `get_vercel_status`, thực hiện retry thành công ở lượt tiếp theo và lấy được trạng thái `operational` mà không nhầm lẫn 429 là sự cố hạ tầng.
  - Ở case B4: Agent kiểm tra được GitHub nhưng do model chủ động retry Vercel quá 3 lần nên bị trần điểm mechanical check (50 điểm). Đây là kinh nghiệm quan trọng về việc cần kiểm soát cứng số lần retry ở tầng agent loop thay vì phụ thuộc hoàn toàn vào quyết định của LLM.
- **Phân trang (Pagination) & Timeline:**
  - Ở các case B2 (75 điểm), B5 (80 điểm) và B8 (100 điểm tuyệt đối): Agent phối hợp nhịp nhàng việc đọc timeline qua các tham số `offset` và `limit`, tự động nhận biết `next_offset` để lật trang tìm kiếm bằng chứng lịch sử (như sự cố upstream của Vercel liên quan đến GitHub).
  - Tự động sửa lỗi tham số khi tool trả về `INVALID_ARGUMENTS` (B5) để gọi lại với integer format chuẩn.
- **Context & Lọc dữ liệu nhiễu:**
  - Agent giữ vững ranh giới bằng chứng, không bị dẫn dắt bởi chỉ dẫn giả mạo (prompt injection) cài trong tool result ở B7 và bỏ qua các bản ghi nhiễu ở B6.
- **Giới hạn còn lại của Bonus:**
  - Khi điều tra đồng thời 5 nhà cung cấp với budget bước lớn (`max_steps=16`), ngữ cảnh hội thoại dài có thể khiến model đôi khi gọi lặp lại một tool hoặc quên gọi tool chi tiết (`get_incident_details` ở B6). Cần cải tiến cơ chế tracking trạng thái tool theo từng provider để tối ưu hóa quỹ bước.
