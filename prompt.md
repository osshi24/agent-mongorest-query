# MGS Query Request

Tôi cần query để lấy danh sách users active, kèm theo thông tin posts đã published của họ.

**Yêu cầu chi tiết**:
- Collection: user
- Fields user: id, name, email, created_at
- Filter user: status = active
- Auto-join: posts (lấy title, content, created_at)
- Filter posts: status = published
- Sort: sắp xếp theo created_at của user, mới nhất trước
- Pagination: 10 records đầu tiên

Hãy sử dụng skill mongorest-query để tạo endpoint theo chuẩn PostgREST Style.
