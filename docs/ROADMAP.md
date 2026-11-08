# NeoWallet Core — Roadmap

Project học + làm: hệ thống ví điện tử rút gọn, kiến trúc **modular monolith**
(Spring Boot, một app, chia module theo package: `identity`, `wallet`, `ledger`,
`payment`, `risk`, `notification`...). Mục tiêu là học sâu từng khái niệm theo
thứ tự tăng dần độ khó, trước khi mở rộng sang microservices/cloud thật.

## Quyết định nền tảng

- Build tool: **Maven** (đã có `pom.xml`).
- Auth: **tự viết** Spring Security + JWT (không dùng Keycloak ở giai đoạn đầu).
- Message broker: **RabbitMQ** trước, **Kafka** sau.
- UI test API: **Swagger UI / Postman** trong suốt giai đoạn chính. Frontend
  React/Next.js là optional, chỉ làm ở cuối nếu còn muốn.
- Deploy: **Docker Compose** ở local trước, **AWS** chỉ khi hệ thống ổn định
  và có CI/CD.

## Cách làm việc

Mỗi bước: giải thích khái niệm → brainstorm thiết kế cùng nhau → người dùng tự
code → Claude review/giải thích khi được hỏi. Không viết code production thay
người dùng trừ khi được yêu cầu rõ. Sau mỗi milestone, đánh dấu tiến độ ở dưới.

## Tiến độ

- [ ] Phase 0 — Khởi tạo nền tảng
- [ ] Phase 1 — Identity (JWT tự viết)
- [ ] Phase 2 — Wallet & Ledger (double-entry) — **phần lõi quan trọng nhất**
- [ ] Phase 3 — Idempotency & validation
- [ ] Phase 4 — Observability cơ bản (logging, Actuator/Micrometer, audit log)
- [ ] Phase 5 — Caching (Caffeine → Redis)
- [ ] Phase 6 — Async & Notification (RabbitMQ)
- [ ] Phase 7 — Fraud/Risk rules đơn giản, hạn mức, KYC level
- [ ] Phase 8 — Admin & Rate limiting
- [ ] Phase 9 — Reconciliation (import CSV, so khớp giao dịch)
- [ ] Phase 10 — Testing nghiêm túc (JUnit, Mockito, Testcontainers)
- [ ] Phase 11 — Containerize, CI/CD, Deploy (Docker Compose → AWS)
- [ ] Phase 12 — Stretch/Optional (Kafka, Keycloak, multi-currency, fee,
      webhook, QR payment, OpenTelemetry, Prometheus/Grafana, tách service,
      frontend React/Next.js)

## Chi tiết từng Phase

### Phase 0 — Khởi tạo nền tảng
- Thêm Spring Boot starter dependencies vào `pom.xml` (web, data-jpa, validation).
- Kết nối PostgreSQL, giới thiệu Flyway để quản lý schema version.
- Tạo 1 entity đơn giản + 1 endpoint health-check để xác nhận toolchain chạy được.
- Giới thiệu springdoc-openapi (Swagger UI) để test API ngay từ đầu.

### Phase 1 — Identity
- User entity, đăng ký/đăng nhập, hash password (BCrypt).
- Spring Security + JWT: access token, refresh token, filter chain.
- Role đơn giản: `USER`, `ADMIN`.

### Phase 2 — Wallet & Ledger
- Account/Wallet entity.
- **Ledger double-entry**: Transaction → nhiều Entry (debit/credit), tổng luôn = 0.
- Balance không lưu trực tiếp — suy ra từ ledger hoặc cập nhật transactional
  cùng DB transaction với ledger.
- Nghiệp vụ: nạp tiền giả lập, rút tiền giả lập, chuyển tiền giữa 2 user, xem
  số dư, xem lịch sử giao dịch.
- Trọng tâm học: ACID transaction, vì sao ledger pattern an toàn hơn update
  balance trực tiếp, concurrency (race condition khi 2 giao dịch cùng lúc
  trên 1 wallet).

### Phase 3 — Idempotency & validation
- Idempotency key cho API tạo giao dịch (bảng lưu key + kết quả, trả lại
  response cũ nếu trùng key).
- Jakarta Validation cho input.
- Vì sao idempotency quan trọng trong hệ thống tiền (client retry không được
  tạo giao dịch trùng).

### Phase 4 — Observability cơ bản
- Structured logging (Logback + JSON).
- Spring Boot Actuator + Micrometer (metrics cơ bản) — chưa cần
  Prometheus/Grafana dashboard ngay.
- Audit log riêng, không trộn với ledger.

### Phase 5 — Caching
- Caffeine (local cache) cho query đọc nhiều (lookup user/account).
- Vì sao cache balance nguy hiểm, nên cache gì mới an toàn.
- Redis giới thiệu sau, khi cần cache distributed (nhiều instance).

### Phase 6 — Async & Notification (RabbitMQ)
- Publish event khi giao dịch thành công/thất bại.
- Notification module tiêu thụ event, gửi email/SMS giả lập.
- Retry + dead-letter queue.

### Phase 7 — Fraud/Risk rules
- Rule engine tối giản: số tiền lớn, giao dịch liên tục, tài khoản mới tạo
  chuyển nhiều.
- Hạn mức chuyển tiền theo ngày, KYC level ảnh hưởng hạn mức.
- Freeze/block account.

### Phase 8 — Admin & Rate limiting
- Admin endpoints: xem giao dịch, khóa tài khoản, duyệt giao dịch rủi ro.
- Rate limit (Resilience4j hoặc bucket đơn giản) cho API công khai.

### Phase 9 — Reconciliation
- Import CSV giao dịch giả lập từ "bank partner".
- So khớp với transaction trong hệ thống: matched / missing in core /
  missing in partner / amount mismatch.

### Phase 10 — Testing nghiêm túc
- JUnit 5 + Mockito cho unit test.
- Testcontainers (PostgreSQL thật trong test) cho integration test — đặc biệt
  quan trọng cho ledger (test tính đúng đắn debit/credit, concurrency).

### Phase 11 — Containerize, CI/CD, Deploy
- Dockerfile cho app, Docker Compose cho app + Postgres + RabbitMQ (+ Redis
  nếu đã dùng).
- CI: build + test tự động (GitHub Actions).
- Deploy lên AWS khi đã ổn định (ví dụ ECS/Fargate hoặc Elastic Beanstalk,
  RDS cho Postgres).

### Phase 12 — Stretch/Optional
- Kafka cho event streaming/audit log quy mô lớn.
- Keycloak thay cho tự-viết JWT.
- Multi-currency, fee calculation, webhook, QR payment giả lập.
- OpenTelemetry tracing, Prometheus + Grafana dashboard đầy đủ.
- Tách 1-2 module thành service riêng (học microservices boundary).
- Frontend React/Next.js đơn giản.

## Verification mỗi Phase

- Chạy app: `mvn spring-boot:run`, test qua Swagger UI.
- Từ Phase 10: chạy `mvn test` với Testcontainers.
- Đánh dấu checklist "Tiến độ" phía trên khi hoàn thành mỗi Phase.
