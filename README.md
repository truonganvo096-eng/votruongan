# Hệ thống gọi món ăn cho nhà hàng tích hợp gợi ý món ăn thông minh

Đồ án môn học — CNTT. Stack: React + Node.js/Express + Prisma (SQLite mặc định, dễ đổi sang PostgreSQL).

## Cấu trúc project

```
project/
├── backend/     # API: Node.js + Express + Prisma
└── frontend/    # Giao diện: React + Vite + Tailwind CSS
```

## 1. Chạy Backend

```bash
cd backend
npm install
npx prisma generate
npx prisma migrate dev --name init   # tạo database SQLite (file dev.db)
npm run seed                          # tạo dữ liệu mẫu (món ăn + lịch sử đơn hàng giả lập)
npm run dev                           # chạy server tại http://localhost:4000
```

Tài khoản mẫu sau khi seed:
- Admin: `admin@shop.com` / `admin123`
- Khách: `customer@shop.com` / `customer123`

## 2. Chạy Frontend

Mở terminal khác:

```bash
cd frontend
npm install
npm run dev                           # chạy tại http://localhost:5173
```

Frontend tự động proxy các request `/api/...` sang backend (xem `vite.config.js`), nên không cần cấu hình CORS phức tạp lúc dev.

## 3. Đổi từ SQLite sang PostgreSQL (khi triển khai thật)

1. Cài PostgreSQL, tạo database mới
2. Trong `backend/prisma/schema.prisma`, đổi:
   ```prisma
   datasource db {
     provider = "postgresql"   // đổi từ "sqlite"
     url      = env("DATABASE_URL")
   }
   ```
3. Trong `backend/.env`, đổi `DATABASE_URL`:
   ```
   DATABASE_URL="postgresql://user:password@localhost:5432/food_ordering"
   ```
4. Trong `backend/src/services/recommendationService.js`, đổi `CAST(x AS REAL)` thành `x::numeric` (cú pháp Postgres) — đã có ghi chú ngay trong file.
5. Chạy lại `npx prisma migrate dev` và `npm run seed`

## 4. Giải thích module gợi ý AI (trọng tâm đồ án)

File chính: `backend/src/services/recommendationService.js`

| Hàm | Kỹ thuật | Mô tả |
|---|---|---|
| `getFrequentlyBoughtTogether` | Association Rule Mining | Đếm các món hay xuất hiện cùng nhau trong lịch sử đơn hàng, tính % Confidence |
| `getSimilarItems` | Content-based filtering | Lọc món cùng danh mục, giá tương đương |
| `getForYouRecommendations` | Cá nhân hóa | Dựa trên danh mục người dùng hay tương tác nhất, fallback về best-seller |

Dữ liệu hành vi được ghi lại tự động khi người dùng: xem món (`view`), thêm giỏ hàng (`add_to_cart`), đặt món (`purchase`) — xem bảng `BehaviorLog` trong `schema.prisma`.

Script `backend/prisma/seed.js` tạo sẵn ~175 đơn hàng giả lập có quy luật rõ ràng (VD: Phở bò đi kèm Trà đá ~75% số lần) để bạn demo module gợi ý có kết quả ngay mà không cần đợi dữ liệu thật.

## 5. Danh sách chức năng đã có

**Khách hàng:** đăng ký/đăng nhập, xem thực đơn theo danh mục, tìm kiếm, xem chi tiết món, giỏ hàng, đặt món, xem lịch sử đơn hàng, nhận gợi ý AI ở 3 vị trí (trang chủ, trang chi tiết món x2 loại).

**Admin:** quản lý món ăn (CRUD), quản lý đơn hàng và cập nhật trạng thái.

## 6. Việc bạn nên tự làm thêm để hoàn thiện báo cáo

- Chèn ảnh món ăn thật (hiện đang để placeholder icon 🍽️)
- Viết phần "Cơ sở lý thuyết" trong báo cáo giải thích công thức Support/Confidence
- Chụp màn hình demo các trang để đưa vào slide bảo vệ
- Có thể mở rộng thêm: đánh giá/review món, mã giảm giá, thông báo real-time (Socket.io)
