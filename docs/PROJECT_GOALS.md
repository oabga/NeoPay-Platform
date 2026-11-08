# NeoWallet Core — Mục tiêu dự án (nguồn gốc, bất biến)

> File này lưu lại **nguyên văn ý định** của người dùng khi khởi tạo project,
> để tham chiếu lại bất cứ lúc nào. Mọi hướng dẫn/trợ giúp của Claude trong
> các buổi làm việc sau đều phải **bám theo file này** — nếu có đề xuất đi
> chệch hướng (bỏ bớt, đổi tech, đổi scope), phải nêu rõ lý do và được người
> dùng đồng ý, không tự ý thay đổi mục tiêu gốc.

## Bối cảnh & cách làm việc

- Đây là project **tự học + tự làm**, phần lớn kiến thức người dùng **chưa
  biết**.
- Claude **không tự viết/sửa code thay** người dùng. Thay vào đó: đi từng
  bước nhỏ, hướng dẫn, tìm kiếm & giải thích khái niệm, brainstorm thiết kế
  để người dùng tự hiểu và tự code.
- **Mục tiêu cuối cùng: làm một project "đến nơi đến chốn"** — đủ chiều sâu,
  đủ chất lượng để dùng làm sản phẩm trong CV, apply đi xin việc (không phải
  demo hời hợt). Điều này có nghĩa: các phần "nâng cao"/"stretch" trong
  roadmap không phải tùy chọn có thể bỏ qua vĩnh viễn — chúng là mục tiêu
  cuối cùng cần đạt tới, chỉ là làm **sau**, theo đúng trình tự học tăng dần.

## Tên & mục tiêu dự án

**NeoWallet Core – Digital Wallet & Ledger Platform**

Xây dựng một hệ thống ví điện tử rút gọn nhưng có các thành phần
production-grade: xử lý tiền an toàn, ledger double-entry, idempotency,
event-driven, caching, queue/streaming, observability, CI/CD, deploy cloud.

## Nghiệp vụ

### MVP bắt buộc
- User đăng ký / đăng nhập.
- Tạo ví cho user.
- Nạp tiền giả lập.
- Rút tiền giả lập.
- Chuyển tiền giữa 2 user.
- Xem số dư.
- Xem lịch sử giao dịch.
- Idempotency cho các API tạo giao dịch.
- Rate limit.
- Audit log.
- Notification khi giao dịch thành công/thất bại.
- Admin xem giao dịch, khóa tài khoản.

### Nghiệp vụ nâng cao (làm sau khi xong MVP)
- Hạn mức chuyển tiền theo ngày.
- Freeze/block account.
- Fraud rule đơn giản:
  - Giao dịch quá lớn.
  - Giao dịch liên tục trong thời gian ngắn.
  - Tài khoản mới tạo đã chuyển tiền nhiều.
- KYC level: chưa KYC → hạn mức thấp; đã KYC → hạn mức cao hơn.
- Reconciliation: import file CSV giao dịch giả lập từ "bank partner", so
  khớp với giao dịch trong hệ thống, phát hiện missing/mismatch.
- Multi-currency.
- QR payment giả lập.
- Webhook notification.
- Fee calculation.
- Dashboard báo cáo.

## Tech stack đề xuất (full, mục tiêu cuối cùng)

Không cần dùng tất cả ngay từ đầu, nhưng đây là đích đến cho CV Java/Fintech.

### Backend

| Layer | Technology | Lý do |
|---|---|---|
| Language | Java 21 LTS | Hiện đại, ổn định, doanh nghiệp dùng nhiều |
| Framework | Spring Boot 3.x | Chuẩn thị trường Java |
| Build tool | Maven | (đã chọn — giữ nguyên, không đổi Gradle) |
| API | REST + OpenAPI/Swagger | Dễ test, dễ tài liệu hóa |
| Security | Spring Security + OAuth2/OIDC/JWT | Học authentication/authorization |
| Identity provider | Keycloak hoặc Spring Authorization Server | (bắt đầu tự viết JWT, Keycloak là mục tiêu sau) |
| ORM/Data access | Spring Data JPA/Hibernate | Phổ biến |
| SQL nâng cao | jOOQ hoặc QueryDSL | Optional, hữu ích cho report |
| Database | PostgreSQL | Rất phù hợp cho transactional |
| Migration | Flyway | Quản lý schema version |
| Validation | Jakarta Validation | Bắt buộc |
| Caching | Redis + Caffeine | Local cache + distributed cache |
| Message queue | RabbitMQ | Command/task/retry |
| Event streaming | Kafka | Event log, audit, analytics, fan-out |
| Resilience | Resilience4j | Retry, circuit breaker, timeout |
| Observability | Micrometer + Prometheus + Grafana | Metrics |
| Logging | Logback + JSON logging + Loki/ELK | Structured logging |
| Tracing | OpenTelemetry hoặc Micrometer Tracing | Trace request giữa các service |
| Testing | JUnit 5, Mockito, Testcontainers, Awaitility | Quan trọng để ra "production" |
| API doc | springdoc-openapi | Swagger UI |

### Frontend (nếu làm fullstack, ở giai đoạn sau)

| Layer | Technology |
|---|---|
| Framework | React + TypeScript |
| Meta-framework | Next.js |
| UI | TailwindCSS + shadcn/ui |
| Data fetching | TanStack Query |
| State | Zustand hoặc Redux Toolkit |
| Form | React Hook Form + Zod |
| Auth | OIDC/JWT với Keycloak |

## Kiến trúc đề xuất

- Bắt đầu: **modular monolith** (chưa đủ kiến thức sâu để làm microservices
  phức tạp ngay). Tách service sau nếu cần, để học event-driven/CV đẹp.
- Các module/service chính (dù monolith hay tách service, ranh giới nghiệp
  vụ vẫn giữ nguyên):

  1. **Identity & KYC**: đăng ký, đăng nhập, profile, KYC level, RBAC
     (user/admin/support/risk). Có thể dùng Keycloak nếu muốn nhanh.
  2. **Account/Wallet**: tạo ví, quản lý tài khoản, freeze/unfreeze, hạn
     mức, truy vấn số dư.
  3. **Ledger** (quan trọng nhất): ghi sổ kế toán double-entry — mỗi giao
     dịch tạo các bút toán đối ứng, tổng debit/credit luôn cân bằng. Không
     cập nhật số dư trực tiếp tùy tiện — số dư suy ra/cập nhật transactional
     từ ledger.
     - Ví dụ chuyển tiền A → B:
       ```
       Transaction: TXN-123
       Entry 1: source wallet A, amount = -100000
       Entry 2: target wallet B, amount = +100000
       ```
  4. **Payment Orchestration**: nhận request chuyển/nạp/rút, validate,
     kiểm tra idempotency, kiểm tra hạn mức, gọi risk/fraud, gọi ledger,
     publish event, trả kết quả.
  5. **Fraud/Risk**: kiểm tra rule đơn giản, ghi risk score, block/review.
     Ví dụ: số tiền > 50.000.000 → cần review; 10 giao dịch/phút → block;
     account mới < 24h mà chuyển > 10.000.000 → flag.
  6. **Notification**: gửi email/SMS/in-app giả lập, retry nếu lỗi,
     dead-letter queue.
  7. **Reconciliation**: import CSV giả lập từ đối tác, so khớp với DB,
     đánh dấu matched / missing in core / missing in partner / mismatch.
  8. **Admin Portal**: tìm user, xem ví, xem giao dịch, khóa tài khoản,
     duyệt giao dịch rủi ro, xem reconciliation.

## Hạn chế của bản đề xuất gốc & cách cải thiện

Những điểm này là phân tích của Claude khi review bản đề xuất gốc — **không
làm thay đổi mục tiêu cuối cùng ở trên**, chỉ thay đổi **trình tự và cách
tiếp cận** để người tự học không bị ngộp và bỏ giữa đường.

| Hạn chế | Rủi ro nếu không sửa | Cách cải thiện |
|---|---|---|
| Liệt kê quá nhiều công nghệ cùng lúc (Keycloak, Kafka, RabbitMQ, Redis, Prometheus, Grafana, OTel, React...) không theo thứ tự ưu tiên | Dễ ngộp, mất động lực trước khi chạm phần lõi (ledger) | Chia theo **Phase tăng dần độ khó** (xem `docs/ROADMAP.md`) — core nghiệp vụ trước, hạ tầng sau |
| Không có tiêu chí "xong" rõ ràng cho từng bước | Dễ làm lan man, không biết khi nào chuyển sang bước tiếp | Mỗi Phase trong roadmap có nghiệp vụ/tech cụ thể, checklist tiến độ |
| Auth đề xuất Keycloak ngay — bỏ qua cơ hội học cơ chế authentication từ gốc | Hiểu "có xác thực" nhưng không hiểu JWT/filter chain hoạt động thế nào — thiệt khi phỏng vấn bị hỏi sâu | Tự viết Spring Security + JWT trước, Keycloak là lựa chọn nâng cấp sau (đã chốt) |
| Không nói rõ balance được tính thế nào — dễ bị hiểu nhầm "update balance trực tiếp" là đủ | Đây là lỗi thiết kế tài chính kinh điển (race condition, mất tính nhất quán) | Nhấn mạnh ngay từ Phase 2: balance suy ra từ ledger, không update tùy tiện |
| Thiếu bước kiểm thử sớm cho logic tiền (chỉ nhắc Testcontainers ở cuối) | Bug ở ledger phát hiện muộn, khó truy ngược | Viết test cho ledger (debit=credit, concurrency) ngay khi code Phase 2, không chờ đến Phase riêng về testing |
| Không đề cập rõ "concurrency" (nhiều giao dịch cùng lúc trên 1 wallet) | Đây là vấn đề production thật, dễ bị hỏi phỏng vấn, dễ bug nếu bỏ qua | Thêm hẳn mục học về DB transaction isolation, optimistic/pessimistic locking ở Phase 2 |
| Chưa có kế hoạch deploy cụ thể (chỉ nói "deploy cloud") | Dễ bị bỏ qua vì là phần khó/tốn thời gian nhất cuối project | Chốt rõ: Docker Compose local trước, AWS sau khi có CI/CD (đã chốt) |

## Liên kết

- Kế hoạch thực thi theo từng Phase (thứ tự, mốc tiến độ): `docs/ROADMAP.md`
- File này là **"tại sao & cái gì"** (mục tiêu, scope đầy đủ, định nghĩa
  "xong đến nơi đến chốn" nghĩa là gì) — không chỉnh sửa trừ khi mục tiêu
  dự án thực sự thay đổi.
