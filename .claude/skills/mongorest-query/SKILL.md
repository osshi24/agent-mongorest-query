---
name: mgs-query-writer
description: Hỗ trợ viết query params cho MGS Core V2 API. Sử dụng khi người dùng cần tạo endpoint với filtering, sorting, pagination, joins/relationships, hoặc aggregations. Skill này validate syntax và giải thích chi tiết từng phần của query.
---

# MGS Query Params Writer

Skill chuyên hỗ trợ viết query parameters cho hệ thống MGS Core V2.

## Khi Nào Sử Dụng Skill Này

- Người dùng cần tạo API endpoint với query params
- Người dùng cần filter, sort, paginate dữ liệu
- Người dùng cần join/embed relationships
- Người dùng cần validate syntax query
- Người dùng cần tối ưu query hiện có

## Base URL

```
https://crm-mge.mangoads.com.vn/api/v1/
```

---

## Quy Trình Làm Việc

### Bước 1: Thu Thập Thông Tin

Trước khi viết query, **BẮT BUỘC** hỏi người dùng các thông tin sau nếu chưa rõ:

1. **Collection/Entity**: Tên collection cần query (users, posts, products...)
2. **Mục đích**: Lấy danh sách, chi tiết, hay thống kê?
3. **Fields cần lấy**: Những fields nào cần trả về?
4. **Điều kiện lọc**: Filter theo tiêu chí gì?
5. **Relationships**: Có cần join với collection khác không? Nếu có:
   - Tên relationship
   - Local field và foreign field
   - Kiểu quan hệ (1-1, 1-n, n-n)
6. **Sắp xếp**: Sort theo field nào, tăng hay giảm?
7. **Phân trang**: Cần bao nhiêu records? Có phân trang không?

### Bước 2: Viết Query

Sau khi có đủ thông tin, tạo endpoint hoàn chỉnh theo format:

```
[METHOD] {base_url}{collection}?{query_params}
```

### Bước 3: Giải Thích

Giải thích từng phần của query:
- Select clause
- Filter conditions
- Join/Embed expressions
- Sort clause
- Pagination

### Bước 4: Validate & Suggest

- Validate syntax đúng format
- Đề xuất tối ưu nếu có thể

---

## Query Syntax Reference

### 1. Select (Projection)

```bash
# Chọn fields cụ thể
?select=field1,field2,field3

# Loại trừ fields (prefix -)
?select=*,-password,-secretKey

# Nested fields
?select=user.name,user.email
```

### 2. Filtering

#### Format: `field=operator.value`

| Operator | Ý Nghĩa | Ví Dụ |
|----------|---------|-------|
| `eq` | Bằng | `status=eq.active` |
| `neq` | Không bằng | `status=neq.deleted` |
| `gt` | Lớn hơn | `age=gt.18` |
| `gte` | Lớn hơn hoặc bằng | `age=gte.18` |
| `lt` | Nhỏ hơn | `price=lt.100` |
| `lte` | Nhỏ hơn hoặc bằng | `price=lte.100` |
| `in` | Trong danh sách | `status=in.[active,pending]` |
| `nin` | Không trong danh sách | `status=nin.[deleted,banned]` |
| `like` | Chứa (case-sensitive) | `name=like.John` |
| `ilike` | Chứa (case-insensitive) | `name=ilike.john` |
| `regex` | Regular expression | `email=regex..*@gmail.com` |
| `exists` | Field tồn tại | `phone=exists.true` |
| `null` | Field là null | `deleted_at=null.true` |
| `notnull` | Field không null | `email=notnull.true` |
| `contains` | Chứa (string/array) | `tags=contains.javascript` |
| `startswith` | Bắt đầu với | `name=startswith.John` |
| `endswith` | Kết thúc với | `email=endswith.@gmail.com` |
| `between` | Trong khoảng | `age=between.[18,65]` |

#### Shorthand (không cần operator)
```bash
?status=active    # Tương đương: status=eq.active
```

#### Logical Operators

```bash
# AND (mặc định)
?status=eq.active&age=gt.18

# OR
?or=(status=eq.active,status=eq.pending)

# NOT
?not=(status=eq.banned)

# Nested
?and=(status=eq.active,or=(role=eq.admin,role=eq.moderator))
```

### 3. Embed Expressions (Joins/Relationships)

```bash
# LEFT JOIN - lấy cả records không có relationship
?select=name,posts()
?select=name,posts(title,content)
?select=name,posts(title,status=eq.published)

# INNER JOIN (prefix !) - chỉ lấy records CÓ relationship
?select=name,posts(!status=eq.published)

# Nested joins
?select=name,posts(title,comments(text,author(name)))
```

**Lưu ý quan trọng về Relationships:**
- Cần biết `local field` và `foreign field` để join đúng
- LEFT JOIN: Records không có relationship vẫn được trả về (relationship = [])
- INNER JOIN: Chỉ records CÓ relationship được trả về

### 4. Sorting

```bash
# Ascending (tăng dần)
?order=name

# Descending (giảm dần) - prefix -
?order=-createdAt

# Multi-field sort
?order=status,-createdAt,name
```

### 5. Pagination

```bash
# Offset-based
?limit=20&offset=40

# Page-based
?page=2&limit=20

# Defaults: limit=20, max=100
```

### 6. Count

```bash
?count=true
```

### 7. Value Types

| Type | Ví Dụ |
|------|-------|
| String | `name=John` hoặc `name="John Doe"` |
| Number | `age=25`, `price=99.99` |
| Boolean | `active=true`, `verified=false` |
| Null | `deleted_at=null` |
| Array | `status=in.[active,pending]` |
| Date (ISO) | `createdAt=gte.2024-01-01T00:00:00Z` |
| ObjectId | `_id=507f1f77bcf86cd799439011` |

---

## Output Format

Khi trả lời, sử dụng format sau:

### Endpoint

```
GET https://crm-mge.mangoads.com.vn/api/v1/{collection}?{params}
```

### Giải Thích

| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Collection | `{collection}` | {mô tả} |
| Select | `{select_clause}` | {mô tả} |
| Filter | `{filter_clause}` | {mô tả} |
| Join | `{join_clause}` | {mô tả} |
| Sort | `{sort_clause}` | {mô tả} |
| Pagination | `{pagination}` | {mô tả} |

### Lưu Ý (nếu có)

- Các lưu ý về performance, security, best practices

### Đề Xuất Tối Ưu (nếu có)

- Các cách tối ưu query

---

## Validation Rules

Khi validate query, kiểm tra:

1. **Operator hợp lệ**: eq, neq, gt, gte, lt, lte, in, nin, like, ilike, regex, exists, null, notnull, contains, startswith, endswith, between, not_between
2. **Array format**: Phải dùng `[value1,value2]`
3. **Logical operators**: and, or, not phải có dấu ngoặc `()`
4. **Sort prefix**: `-` cho descending, không prefix cho ascending
5. **Pagination**: limit > 0, offset >= 0, limit <= 100

---

## Ví Dụ Tham Khảo

### Ví dụ 1: Lấy danh sách users active

**Yêu cầu**: Lấy danh sách users đang active, chỉ lấy name và email, sắp xếp theo ngày tạo mới nhất, 20 records đầu tiên.

**Endpoint**:
```
GET https://crm-mge.mangoads.com.vn/api/v1/users?select=name,email&status=eq.active&order=-createdAt&limit=20
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Collection | `users` | Bảng người dùng |
| Select | `name,email` | Chỉ lấy 2 fields |
| Filter | `status=eq.active` | Chỉ lấy users có status = "active" |
| Sort | `-createdAt` | Sắp xếp giảm dần theo ngày tạo (mới nhất trước) |
| Pagination | `limit=20` | Lấy tối đa 20 records |

---

### Ví dụ 2: Lấy posts với author và comments

**Yêu cầu**: Lấy posts đã published, kèm thông tin author (name, avatar) và comments (text, likes).

**Thông tin relationships**:
- `posts.author_id` → `users._id` (many-to-one)
- `posts._id` → `comments.post_id` (one-to-many)

**Endpoint**:
```
GET https://crm-mge.mangoads.com.vn/api/v1/posts?select=title,content,author(name,avatar),comments(text,likes)&status=eq.published&order=-createdAt&limit=10
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Collection | `posts` | Bảng bài viết |
| Select | `title,content` | Fields của post |
| Join 1 | `author(name,avatar)` | LEFT JOIN với users, lấy name và avatar |
| Join 2 | `comments(text,likes)` | LEFT JOIN với comments, lấy text và likes |
| Filter | `status=eq.published` | Chỉ lấy posts đã publish |
| Sort | `-createdAt` | Mới nhất trước |
| Pagination | `limit=10` | 10 posts |

---

### Ví dụ 3: Filter phức tạp với OR

**Yêu cầu**: Lấy users là admin HOẶC moderator, đang active, không bị banned.

**Endpoint**:
```
GET https://crm-mge.mangoads.com.vn/api/v1/users?select=id,name,email,role&and=(status=eq.active,or=(role=eq.admin,role=eq.moderator),not=(banned=eq.true))&order=name
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Filter | `and=(...)` | Kết hợp nhiều điều kiện bằng AND |
| Condition 1 | `status=eq.active` | User phải active |
| Condition 2 | `or=(role=eq.admin,role=eq.moderator)` | Role là admin HOẶC moderator |
| Condition 3 | `not=(banned=eq.true)` | KHÔNG bị banned |

---

### Ví dụ 4: INNER JOIN - Chỉ lấy users CÓ posts

**Yêu cầu**: Lấy users có ít nhất 1 post đã published.

**Endpoint**:
```
GET https://crm-mge.mangoads.com.vn/api/v1/users?select=name,email,posts(!status=eq.published)&limit=20
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Join | `posts(!status=eq.published)` | INNER JOIN (prefix `!`) - chỉ trả về users CÓ posts với status=published |

**So sánh với LEFT JOIN**:
- `posts(status=eq.published)` → Trả về TẤT CẢ users, posts=[] nếu không có
- `posts(!status=eq.published)` → CHỈ users có posts published

---

### Ví dụ 5: Search với ilike

**Yêu cầu**: Tìm products có tên chứa "laptop" (không phân biệt hoa thường), category là electronics, giá từ 500-2000.

**Endpoint**:
```
GET https://crm-mge.mangoads.com.vn/api/v1/products?select=name,price,category,stock&name=ilike.laptop&category=eq.electronics&price=between.[500,2000]&order=price&limit=20
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Filter 1 | `name=ilike.laptop` | Tên chứa "laptop" (case-insensitive) |
| Filter 2 | `category=eq.electronics` | Category = electronics |
| Filter 3 | `price=between.[500,2000]` | Giá từ 500 đến 2000 |
| Sort | `price` | Sắp xếp tăng dần theo giá |

---

## Best Practices

### Performance

1. **Chỉ select fields cần thiết** - Tránh `select=*`
2. **Luôn dùng pagination** - Tránh lấy quá nhiều records
3. **Filter trước khi join** - Giảm data cần xử lý
4. **Giới hạn join depth** - Tối đa 2-3 levels
5. **Dùng INNER JOIN khi phù hợp** - Giảm data trả về

### Security

1. **Không expose sensitive fields** - Loại trừ password, tokens
2. **Validate user input** - Tránh injection

### Readability

1. **Sắp xếp params theo thứ tự**: select → filter → order → pagination
2. **Dùng shorthand khi có thể**: `status=active` thay vì `status=eq.active`

---

## Xử Lý Yêu Cầu Không Rõ Ràng

Khi người dùng yêu cầu không đủ thông tin, **HỎI LẠI** trước khi viết query:

### Template câu hỏi:

```
Để viết query chính xác, tôi cần thêm thông tin:

1. **Collection**: Bạn muốn query từ bảng/collection nào?
2. **Fields**: Cần lấy những fields nào? (hoặc tất cả?)
3. **Relationships**: Có cần join với bảng khác không?
   - Nếu có: tên bảng, local field, foreign field?
4. **Filters**: Điều kiện lọc cụ thể?
5. **Sort**: Sắp xếp theo field nào?
6. **Pagination**: Cần bao nhiêu records?
```

---

## Suggest Alternatives

Khi query có thể tối ưu, đề xuất alternatives:

```
### Đề Xuất Tối Ưu

**Query hiện tại**: {current_query}

**Vấn đề**: {issue}

**Query tối ưu**: {optimized_query}

**Lý do**: {reason}
```
