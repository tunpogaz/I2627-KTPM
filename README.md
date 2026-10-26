# THỰC HÀNH: WAF TRÊN NGINX

---

## MỤC TIÊU BÀI THỰC HÀNH

Dựng một "người gác cổng" Gatekeeper Pattern:
- Nginx + ModSecurity + bộ luật OWASP CRS đứng trước ứng dụng OWASP Juice Shop: web bán hàng cố ý có lỗ hổng.
- Thử nghiệm tấn công SQL Injection khi không có WAF và khi có WAF để kiểm chứng khả năng bảo vệ.

---

## CẤU HÌNH HỆ THỐNG `docker-compose.yml`

```yaml
services:
  web:
    image: bkimminich/juice-shop
    ports: ["3000:3000"]
  waf:
    image: owasp/modsecurity-crs:nginx
    ports: ["8080:8080"]
    environment:
      BACKEND: http://web:3000
      MODSEC_RULE_ENGINE: "On"
      BLOCKING_PARANOIA: "1"
```

---

## CÁCH KHỞI ĐỘNG HỆ THỐNG

Mở terminal tại thư mục này và chạy:
```bash
docker compose up -d
```
*Chờ khoảng 20 - 30 giây để cả hai container khởi động hoàn tất.*

---

## HOẶC KIỂM THỬ BẰNG LỆNH CURL TRONG TERMINAL

Nếu không muốn mở trình duyệt, bạn có thể chạy 3 lệnh này trực tiếp trong terminal:

1. Tấn công cổng 3000 không qua WAF:
```bash
curl -i -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": "\x27 OR 1=1--", "password": "any"}'
```
Kết quả: `200 OK`, Hack thành công vào tài khoản Admin.

2. Tấn công cổng 8080 qua WAF:
```bash
curl -i -X POST http://localhost:8080/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": "\x27 OR 1=1--", "password": "any"}'
```
Kết quả: `403 Forbidden`, Bị WAF chặn.

3. Đăng nhập hợp lệ cổng 8080:
```bash
curl -i -X POST http://localhost:8080/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@juice-sh.op", "password": "admin123"}'
```
Kết quả: `200 OK`, Đăng nhập bình thường.

4. Xem log WAF phát hiện luật 942100:
```bash
docker compose logs waf | grep 942100
```

---

## SAU KHI TEST XONG, DỌN DẸP:

```bash
docker compose down
```

