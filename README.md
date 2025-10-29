# EProject-Phase-1

Một dự án minh họa microservices nhỏ gồm các dịch vụ: api-gateway, auth, product, order cùng hạ tầng MongoDB và RabbitMQ (docker-compose).

Mục tiêu

- Demo cách tổ chức microservice: tách biệt auth/product/order, giao tiếp qua HTTP và message queue (RabbitMQ).
- Cung cấp API để quản lý sản phẩm, đăng ký/đăng nhập, và xử lý đơn hàng bất đồng bộ.

Thành phần chính

- api-gateway: reverse-proxy đơn giản gom các route của các service.
- auth: đăng ký / đăng nhập (phát JWT), lưu users vào MongoDB.
- product: quản lý sản phẩm, tạo order (publish message tới queue `orders`) và long-poll chờ kết quả từ queue `products`.
- order: tiêu thụ queue `orders`, lưu order vào MongoDB và publish kết quả lên queue `products`.
- mongo: MongoDB
- rabbitmq: RabbitMQ (management UI trên port 15672)

Chạy project (local, yêu cầu Docker & docker-compose)

1. Khởi stack:

```bash
docker-compose up --build -d
```

2. Các port chính (mặc định trong `docker-compose.yml`):

- API Gateway: http://localhost:3003
- Auth service: http://localhost:3000
- Product service: http://localhost:3001
- Order service: http://localhost:3002
- MongoDB (host port): 27017 (container 27017)
- RabbitMQ UI: http://localhost:15672 (guest/guest)

3. Luồng demo (Postman / curl):

- Đăng ký: POST http://localhost:3003/auth/register
- Login: POST http://localhost:3003/auth/login → lấy token
- Tạo product: POST http://localhost:3003/api/products (Header: Authorization: Bearer `<token>`)
- Mua product: POST http://localhost:3003/api/products/buy (gửi `ids` array) → long-poll trả khi order hoàn tất

Ghi chú kỹ thuật

- Schema `product` yêu cầu `name` và `price` (Mongoose required). Nếu nhận lỗi validation, kiểm tra body JSON và header `Content-Type: application/json`.
- API Gateway forward `/products` → product service; gateway cũng hỗ trợ `/api/products`.
- `ordersMap` trong `product` service là in-memory (ephemeral) — không dùng cho production.
