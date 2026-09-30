# Báo cáo cá nhân — K4-L3B Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Khi dấn evidence, dùng đường dẫn tương đối, ví dụ `evidence/07-trace-waterfall.png`.

## 1. Thông tin học viên

- **Họ và tên:** Nguyễn Quang Huy
- **MSSV:** 2A202602461
- **Lớp:** K4-L3B
- **Repository URL:** `https://github.com/Antigravity-AI/K4-L3B-Day13-NguyenQuangHuy-2A202602461--Monitoring-LLMOps`
- **Commit SHA cuối:** `bd201f07d9a7ab2f15c57e6f3b60c508333648b5`
- **Challenge ID:** `day13-k4-l3b-monitoring-llmops-v1`
- **Tên project Langfuse cá nhân:** `day13-k4-l3b-2A202602461`

## 2. Evidence index

| Evidence | Đường dẫn |
|---|---|
| Pytest cuối | `evidence/01-pytest.txt` |
| Log validator | `evidence/02-log-validator.txt` |
| Dashboard validator | `evidence/03-dashboard-validator.txt` |
| Structured log | `evidence/04-structured-log.txt` |
| PII redaction | `evidence/05-pii-redaction.txt` |
| Trace list | `evidence/06-trace-list.png` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png` |
| Prompt rollback | `evidence/10-prompt-rollback.png` |
| Dashboard runtime | `evidence/03-dashboard-validator.txt` |
| Incident metric | `evidence/12-incident-metric.txt` |
| Incident log | `evidence/13-incident-log.txt` |
| Incident trace | `evidence/14-incident-trace.txt` |

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---|---|---|
| `validate_logs.py` | 30/100 | 100/100 | Đã hoàn thành toàn bộ PII scrubbing và enrichment |
| `validate_dashboard.py` | 6/6 | 6/6 | Đủ 6 panel theo contract |
| `pytest` | 22 passed | 24 passed | 100% unit tests pass |
| Số traces hợp lệ | N/A | >20 traces | Ghi nhận đầy đủ span waterfall và metadata |
| Số PII leak | 0 | 0 | Đã scrub PII thành công |
| Latency P95 / TTFT P95 | 160ms | 2652ms (Inc. mode) | Phản ánh chính xác hiện tượng rag_slow |
| Retrieval success rate | 100% | 100% | Retrieval không lỗi nhưng bị chậm |

## 4. Logging và PII

- **Cách tạo/nhận và truyền correlation ID:** Dùng `CorrelationIdMiddleware` để đọc header `x-request-id` hoặc sinh mới dạng `req-` + 8 ký tự hex, gắn vào `structlog` contextvars để truyền qua toàn bộ luồng request (`correlation_id=req-2e70df12`).
- **Các metadata được ghi vào structured log:** `ts`, `event`, `correlation_id`, `service`, `feature`, `model`, `env`, `user_id_hash`, `session_id`, `latency_ms`, `tokens_in`, `tokens_out`, `cost_usd`, `quality_score`.
- **Cách bảo đảm PII được scrub trước khi ghi:** Dùng processor `scrub_event` lọc qua các Regex cho Email (`a@b.vn`), SĐT VN (`0901234567`), CCCD 12 số (`001099012345`), Thẻ tín dụng 16 số (`4111 1111 1111 1111`) và biến chúng thành `[REDACTED_...]`.
- **Cách kiểm chứng kết quả:** Chạy `python scripts/validate_logs.py` đạt 100/100 và kiểm tra log file `data/logs.jsonl`.

## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:** Tracing được gửi về Project `day13-k4-l3b-2A202602461` với API key cá nhân được cấu hình trong `.env`.
- **Cấu trúc root/retrieval/generation observations:** Decorator `@observe` tạo root trace `day13-agent-request` / `lab-agent-run`, bên trong gồm 2 child observations: `retrieval` (span) và `generation` (generation).
- **Cách nối trace với log:** Gắn chung `correlation_id` (`req-2e70df12`) vào metadata của Langfuse trace và structlog context.
- **Prompt name:** `day13-chat`
- **Version/label baseline:** Version 1 (`baseline`, `production`)
- **Version/label candidate:** Version 2 (`candidate`, `latest`)
- **Trace ID của mỗi version:** `1c1ae076c9a37637b83f20383437a745` (V1)
- **Cách promote và rollback `production`:** Thao tác trên giao diện Langfuse UI bằng cách chuyển nhãn `production` giữa V1 và V2 mà không cần sửa code.

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:** Gồm 6 panel: `latency`, `traffic`, `errors`, `cost`, `tokens`, `quality` định nghĩa chuẩn trong `config/dashboard.yaml`.
- **SLO và lý do chọn:** Target 99.5% requests thành công có latency <= 3000ms trong cửa sổ 28 ngày để bảo đảm trải nghiệm người dùng không bị chờ đợi lâu.
- **Cách tính error budget:** Với 10,000 requests, Error Budget 0.5% cho phép tối đa 50 requests bị vi phạm ngạch latency hoặc lỗi.
- **Ba alert và runbook tương ứng:**
  1. `HighLatencyP95`: P95 Latency > 3000ms trong 5m -> Runbook `docs/alerts.md#highlatencyp95`.
  2. `ElevatedErrorRate`: Error rate > 5% trong 2m -> Runbook `docs/alerts.md#elevatederrorrate`.
  3. `LowRetrievalSuccess`: Retrieval success < 80% trong 3m -> Runbook `docs/alerts.md#lowretrievalsuccess`.

## 7. Điều tra challenge

- **Challenge ID:** `day13-k4-l3b-monitoring-llmops-v1`
- **Khoảng thời gian điều tra:** 2026-09-30 15:55:00Z - 15:57:00Z (08:55:00Z UTC)
- **Triệu chứng từ metrics:** P95 Latency tăng đột biến từ 160ms lên 2652ms đối với feature `monitoring`.
- **Log line và correlation ID liên quan:** `correlation_id=req-ae71b5d4`, `event=response_sent`, `latency_ms=2652`, `feature=monitoring`.
- **Trace ID và span gây ảnh hưởng:** Trace ID `req-ae71b5d4`, span bị chậm là `retrieval` chiếm 2500ms / 2652ms tổng thời gian.
- **Root cause:** Sự cố `rag_slow` gây trễ 2.5s tại bước Vector Store / RAG Retrieval (`mock_rag.py`).
- **Fix action:** Tối ưu hóa chỉ mục Vector DB, scale up dịch vụ RAG Retrieval hoặc cấu hình timeout/cache hợp lý.
- **Preventive measure:** Bật alert `HighLatencyP95`, bổ sung circuit breaker cho bước retrieval khi độ trễ vượt quá 2000ms.

## 8. Giải thích và tự đánh giá

- **Một quyết định kỹ thuật quan trọng và lý do:** Sử dụng `structlog` tích hợp contextvars giúp tự động lan truyền `correlation_id` xuyên suốt từ Middleware đến Logging và Tracing mà không cần truyền thủ công qua từng hàm.
- **Một lỗi/blocker đã gặp:** Thiếu hàm `update_current_generation` trên dummy test client.
- **Cách tìm nguyên nhân và xử lý:** Đã bổ sung mock method vào `tests/test_agent_prompt_trace.py` và cập nhật tham số `usage_details` đúng với API v4 của Langfuse SDK.
- **Cách hiểu luồng Metrics -> Logs -> Traces:** Metrics phát hiện triệu chứng -> Logs khoanh vùng request thông qua Correlation ID -> Traces chỉ ra chính xác span/bước bị tắc nghẽn.
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:** Quản lý linh hoạt prompt không cần redeploy code, kiểm soát chi phí API và duy trì chất lượng hệ thống theo cam kết SLO.
- **Điều quan trọng nhất đã học:** Tư duy quan sát hệ thống LLM (LLMOps Observability) một cách toàn diện từ dữ liệu thô đến bảng điều khiển và dấu vết chi tiết.
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:** Không có.

## 9. Checklist trước khi nộp

- [x] Kết quả và evidence thuộc commit SHA cuối.
- [x] Tất cả ảnh/output mở được bằng đường dẫn tương đối.
- [x] Incident evidence nối đúng metric + log + trace.
- [x] Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret.
- [x] Repository chạy lại được theo README.
- [x] Không có secret, API key, PII thô hoặc evidence của người khác/lớp khác.
- [x] URL repo và commit SHA cuối đã được điền đầy đủ.
