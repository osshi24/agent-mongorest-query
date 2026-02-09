# MGS Query Request

Tôi cần query để lấy danh sách users active, kèm theo thông tin posts của họ.

**Yêu cầu chi tiết**:
- Collection: users
- Fields: id, name, email, createdAt
- Filter: status = active
- Join: posts (lấy title, content, status)
- Sort: sắp xếp theo createdAt mới nhất
- Pagination: 20 records đầu tiên

Hãy sử dụng skill mongorest-query để tạo endpoint.
