# Thông Tin Deploy — Checkpoint 5

> Service đã được triển khai công khai. Tài liệu này chỉ ghi tên biến môi
> trường và nguồn cấu hình, không chứa giá trị API key.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Lương Trung Chiến |
| Mã học viên | 2A202601391 |
| Repo | https://github.com/luongtrungchien-249/DAY12-2A202601391-LuongTrungChien |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-1244.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Railway tự gán |
| `AGENT_API_KEY` | ✅ | Đặt trong Railway Variables, không nằm trong repo |
| `REDIS_URL` | ✅ | Tham chiếu Redis managed service `day12-redis` của Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Kết Quả Chạy Thật

```text
GET /health         → 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready          → 200 {"status":"ready","redis":true}
POST /ask không key → 401 {"detail":"invalid or missing API key"}
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — Railway dashboard hiển thị deployment thành công.
- `screenshots/health.png` — public endpoint `/health` trả HTTP 200.
