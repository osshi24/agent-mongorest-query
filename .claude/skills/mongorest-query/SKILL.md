---
name: mongorest-query
description: Hỗ trợ viết query params cho MGS Core V2 API (PostgREST Style). Sử dụng khi người dùng cần tạo endpoint với filtering, sorting, pagination, auto-join relationships. Skill này CHẶT CHẼ tuân thủ syntax chuẩn, KHÔNG tự tạo syntax mới.
---

# MGS Query Params Writer (PostgREST Style)

Skill chuyên hỗ trợ viết query parameters theo chuẩn PostgREST Style cho hệ thống MGS Core V2 API.

## ⚠️ CRITICAL RULES - BẮT BUỘC TUÂN THỦ

**NGHIÊM NGẶT KHÔNG ĐƯỢC VI PHẠM:**

1. **CHỈ SỬ DỤNG SYNTAX TRONG DOCS** - KHÔNG TỰ TẠO SYNTAX MỚI
2. **CHỈ SỬ DỤNG OPERATORS Đà LIỆT KÊ** - KHÔNG PHÁT MINH OPERATORS MỚI
3. **EMBEDDED RELATIONSHIPS** - Bắt buộc dùng syntax: `relation(),relation.field` KHÔNG DÙNG `relation(field)`
4. **STRING VALUES** - Luôn dùng dấu ngoặc kép cho string có khoảng trắng: `name=eq."John Doe"`
5. **VALIDATE MỌI QUERY** - Kiểm tra syntax trước khi trả về
6. **KHI KHÔNG CHẮC** - Hỏi lại người dùng thay vì đoán

---

## Base URL

```
Production: https://docss-api.mangoads.com.vn/api/v1/
```

---

## Khi Nào Sử dụng Skill Này

- Người dùng cần tạo API endpoint với query params
- Người dùng cần filter, sort, paginate dữ liệu
- Người dùng cần auto-join/embed relationships
- Người dùng cần validate syntax query
- Người dùng cần tối ưu query hiện có

---

## Quy Trình Làm Việc

### Bước 1: Thu Thập Thông Tin

Trước khi viết query, **BẮT BUỘC** hỏi người dùng các thông tin sau nếu chưa rõ:

1. **Collection/Entity**: Tên collection cần query (user, posts, products, orders...)
2. **Mục đích**: Lấy danh sách, chi tiết theo ID, hay thống kê?
3. **Fields cần lấy**: Những fields nào cần trả về? (hoặc tất cả?)
4. **Điều kiện lọc**: Filter theo tiêu chí gì? (status, price, date...)
5. **Relationships**: Có cần auto-join với collection khác không? Nếu có:
   - Tên relationship (posts, comments, orders, customer...)
   - Nested relationships? (posts → comments → author)
6. **Sắp xếp**: Sort theo field nào? Tăng dần hay giảm dần?
7. **Phân trang**: Cần bao nhiêu records? Skip bao nhiêu?

### Bước 2: Viết Query

Sau khi có đủ thông tin, tạo endpoint hoàn chỉnh theo format:

```
GET {base_url}{collection}?{query_params}
GET {base_url}{collection}/{id}?{query_params}
```

**Thứ tự tham số khuyến nghị**: `select → filters → order → limit/skip/count`

### Bước 3: Giải Thích

Giải thích từng phần của query:
- Select clause (fields nào được chọn)
- Filter conditions (điều kiện lọc)
- Auto-join relationships (join với bảng nào, lấy fields gì)
- Sort clause (sắp xếp theo field nào)
- Pagination (limit, skip, count)

### Bước 4: Validate & Suggest

- Validate syntax đúng chuẩn PostgREST
- Kiểm tra operators hợp lệ
- Đề xuất tối ưu nếu có thể

---

## Query Syntax Reference (PostgREST Style)

### 1. Select (Projection)

**Cú pháp**: `?select=field1,field2,field3`

```bash
# Chọn fields cụ thể
?select=id,name,email

# Tất cả fields
?select=*

# Loại trừ fields (prefix -)
?select=*,-password,-secretKey

# Chỉ cần ID và name
?select=_id,username
```

### 2. Filtering

**Format cơ bản**: `field=operator.value`

**Nested fields**: `field.nested_field=operator.value`

**String có khoảng trắng**: `field=operator."value with spaces"`

#### Danh sách Operators (CHỈ SỬ DỤNG TRONG DANH SÁCH NÀY)

**So sánh cơ bản:**

| Operator | Ý Nghĩa | Ví Dụ |
|----------|---------|-------|
| `eq` | Bằng (equal) | `status=eq.active`, `name=eq."John Doe"` |
| `neq` | Không bằng (not equal) | `status=neq.deleted` |
| `gt` | Lớn hơn (greater than) | `age=gt.18`, `price=gt.1000000` |
| `gte` | Lớn hơn hoặc bằng | `age=gte.18` |
| `lt` | Nhỏ hơn (less than) | `price=lt.1000000` |
| `lte` | Nhỏ hơn hoặc bằng | `price=lte.1000000` |

**Operators cho Array:**

| Operator | Ý Nghĩa | Ví Dụ |
|----------|---------|-------|
| `in` | Nằm trong danh sách | `category=in.[phone,laptop,tablet]` |
| `nin` | Không nằm trong danh sách | `status=nin.[deleted,archived]` |

**Operators cho String:**

| Operator | Ý Nghĩa | Ví Dụ |
|----------|---------|-------|
| `like` | Tìm kiếm pattern (case-sensitive) | `name=like.*John*` |
| `ilike` | Tìm kiếm pattern (case-insensitive) | `title=ilike.*mobile*` |
| `contains` | Chứa chuỗi | `description=contains.smartphone` |
| `startswith` | Bắt đầu bằng | `email=startswith.admin` |
| `endswith` | Kết thúc bằng | `email=endswith.@gmail.com` |
| `not_contains` | Không chứa | `description=not_contains.old` |
| `not_startswith` | Không bắt đầu bằng | `code=not_startswith.TEMP` |
| `not_endswith` | Không kết thúc bằng | `filename=not_endswith..tmp` |
| `regex` | Regular expression | `phone=regex."^0[0-9]{9}$"` |

**Operators đặc biệt:**

| Operator | Ý Nghĩa | Ví Dụ |
|----------|---------|-------|
| `is` | Kiểm tra null | `deletedAt=is.null` |
| `not.is` | Không phải null | `email=not.is.null` |
| `exists` | Trường tồn tại | `metadata=exists.true` |
| `between` | Trong khoảng | `price=between.(1000000,5000000)` |
| `not_between` | Ngoài khoảng | `age=not_between.(18,60)` |

**Lưu ý:**
- `between` dùng dấu ngoặc tròn `()` KHÔNG phải `[]`
- String có khoảng trắng PHẢI dùng dấu ngoặc kép: `name=eq."John Doe"`

#### Shorthand (không cần operator)

```bash
?status=active    # Tương đương: status=eq.active
?age=18          # Tương đương: age=eq.18
```

#### Logical Operators

```bash
# AND (mặc định khi có nhiều params)
?status=eq.active&age=gt.18

# OR
?or=(status=eq.active,status=eq.pending)

# NOT
?not=status=eq.deleted

# AND với nhiều điều kiện
?and=(name=eq."thuan",profile.role=eq."admin")

# Nested logical operators
?and=(age=gt.18,or=(status=eq.active,profile.type=eq."vip"))
```

### 3. Embedded Relationships (Auto-join)

**⚠️ CÚ PHÁP QUAN TRỌNG - BẮT BUỘC TUÂN THỦ:**

```bash
# ĐÚNG ✅ - Cú pháp chuẩn PostgREST
?select=id,name,posts(),posts.title,posts.content,posts.status

# SAI ❌ - KHÔNG DÙNG
?select=id,name,posts(title,content,status)
```

**Cú pháp Auto-join:**

```bash
# 1. Khai báo relationship với ()
relation()

# 2. Truy cập fields của relationship bằng dấu chấm
relation.field1,relation.field2

# 3. Kết hợp trong select
?select=main_fields,relation(),relation.field1,relation.field2
```

**Ví dụ cụ thể:**

```bash
# User với posts
?select=id,name,email,posts(),posts.title,posts.content,posts.status

# User với posts và filter posts
?select=id,name,posts(),posts.title,posts.content&posts.status=eq.published

# Nested auto-join (3 cấp): user → posts → comments
?select=id,name,posts(comments()),posts.title,posts.comments.text,posts.comments.author

# Multiple auto-join cùng cấp: user có cả posts và orders
?select=id,name,posts(),posts.title,orders(),orders.order_code,orders.total

# Auto-join với filter phức tạp
?select=id,order_code,customer(),customer.name,customer.email,items(),items.name,items.price&customer.address.city=eq."Ho Chi Minh"&items.price=gte.100000
```

**Lưu ý quan trọng:**
- Luôn dùng `relation()` để khai báo relationship trước
- Sau đó dùng `relation.field` để lấy fields cụ thể
- Nested join: `posts(comments())` rồi `posts.title,posts.comments.text`
- KHÔNG BAO GIỜ dùng `posts(title,content)` - syntax này SAI

### 4. Sorting (Order)

**Cú pháp**: `?order=field1,field2,-field3`

```bash
# Ascending (tăng dần) - không prefix
?order=name
?order=created_at

# Descending (giảm dần) - prefix dấu trừ (-)
?order=-created_at
?order=-price

# Multi-field sort
?order=-score,name              # Điểm giảm dần, rồi tên tăng dần
?order=-priority,status,-updated_at
```

### 5. Pagination

**Cú pháp Pagination:**

```bash
# Limit - Giới hạn số lượng kết quả
?limit=20

# Skip/Offset - Bỏ qua số lượng bản ghi
?skip=40
?offset=40      # Tương tự skip

# Count - Đếm tổng số bản ghi
?count=exact

# Kết hợp
?limit=20&skip=40&count=exact   # Trang 3 (skip 40, lấy 20)
```

**Ví dụ phân trang:**

```bash
# Trang 1: 20 bản ghi đầu
?limit=20

# Trang 2: Bỏ qua 20, lấy 20 tiếp
?limit=20&skip=20

# Trang 3: Bỏ qua 40, lấy 20 tiếp
?limit=20&skip=40

# Với count để biết tổng số
?limit=10&count=exact
```

### 6. Value Types

| Type | Cú Pháp | Ví Dụ |
|------|---------|-------|
| String | `field=value` hoặc `field="value with spaces"` | `name=John`, `name=eq."John Doe"` |
| Number | `field=number` | `age=25`, `price=99.99` |
| Boolean | `field=true/false` | `active=true`, `verified=false` |
| Null | `field=is.null` | `deletedAt=is.null` |
| Array | `field=in.[value1,value2]` | `status=in.[active,pending]` |
| Date (ISO) | `field=gte."2024-01-01T00:00:00Z"` | `createdAt=gte."2024-01-01T00:00:00Z"` |
| ObjectId | `field=507f1f77bcf86cd799439011` | `_id=eq.507f1f77bcf86cd799439011` |

---

## Output Format

Khi trả lời, **BẮT BUỘC** sử dụng format sau:

### 📍 Endpoint

```
GET https://docss-api.mangoads.com.vn/api/v1/{collection}?{params}
```

### 📊 Giải Thích Chi Tiết

| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Collection | `{collection}` | {mô tả collection} |
| Select | `{select_clause}` | {giải thích fields được chọn} |
| Filter | `{filter_clause}` | {giải thích điều kiện lọc} |
| Auto-join | `{join_clause}` | {giải thích relationships} |
| Sort | `{sort_clause}` | {giải thích sắp xếp} |
| Pagination | `{pagination}` | {giải thích phân trang} |

### 💡 Lưu Ý (nếu có)

- Performance considerations
- Security notes
- Best practices

### ✨ Đề Xuất Tối Ưu (nếu có)

- Cách tối ưu query hiện tại
- Alternatives

---

## Validation Rules

**BẮT BUỘC kiểm tra trước khi trả về query:**

1. ✅ **Operator hợp lệ**: Chỉ dùng operators trong danh sách: `eq, neq, gt, gte, lt, lte, in, nin, like, ilike, contains, startswith, endswith, not_contains, not_startswith, not_endswith, regex, is, not.is, exists, between, not_between`

2. ✅ **Array format**: Dùng `[value1,value2]` cho `in`, `nin`

3. ✅ **Between format**: Dùng `(value1,value2)` KHÔNG phải `[value1,value2]`

4. ✅ **Logical operators**: `and`, `or` phải có dấu ngoặc `()`

5. ✅ **Sort prefix**: Dấu `-` cho descending, không prefix cho ascending

6. ✅ **Pagination**: `limit > 0`, `skip >= 0`, `offset >= 0`

7. ✅ **Embedded relationships**: Phải dùng `relation(),relation.field` KHÔNG phải `relation(field)`

8. ✅ **String có khoảng trắng**: Bắt buộc dấu ngoặc kép `"value with spaces"`

9. ✅ **Base URL**: Luôn dùng `https://docss-api.mangoads.com.vn/api/v1/`

---

## Ví Dụ Tham Khảo

### Ví dụ 1: Filter đơn giản với select

**Yêu cầu**: Lấy danh sách users active, chỉ lấy username và email, giới hạn 20 records.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=username,email&status=eq.active&limit=20
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Collection | `user` | Bảng người dùng |
| Select | `username,email` | Chỉ lấy 2 fields: username và email |
| Filter | `status=eq.active` | Chỉ lấy users có status = "active" |
| Pagination | `limit=20` | Giới hạn 20 records |

**Response mẫu**:
```json
{
  "message": "Request successful",
  "statusCode": 200,
  "data": [
    {
      "_id": "68aed6105fc92059e5326fc8",
      "username": "john_doe",
      "email": "john@example.com"
    }
  ],
  "meta": {
    "total": 15,
    "last_page": 1,
    "current_page": 1
  }
}
```

---

### Ví dụ 2: Auto-join cơ bản (User với Posts)

**Yêu cầu**: Lấy user với posts của họ, chỉ lấy title và status của posts.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,email,posts(),posts.title,posts.status&limit=2
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Collection | `user` | Bảng user |
| Select - Main | `id,name,email` | Fields của user chính |
| Auto-join | `posts()` | Khai báo join với bảng posts |
| Select - Posts | `posts.title,posts.status` | Lấy title và status từ posts |
| Pagination | `limit=2` | Giới hạn 2 users |

**Response mẫu**:
```json
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "thuan",
      "email": "thuan@docs.com.vn",
      "posts": [
        {
          "_id": "507f1f77bcf86cd799439019",
          "title": "Hướng dẫn sử dụng API",
          "status": "published"
        }
      ]
    }
  ]
}
```

---

### Ví dụ 3: Auto-join với filter trên relationship

**Yêu cầu**: Lấy user với posts đã published.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(),posts.title,posts.content,posts.created_at&posts.status=eq.published&limit=1
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Auto-join | `posts()` | Join với posts |
| Select - Posts | `posts.title,posts.content,posts.created_at` | Fields từ posts |
| Filter - Posts | `posts.status=eq.published` | Chỉ lấy posts đã published |

---

### Ví dụ 4: Nested auto-join 3 cấp (User → Posts → Comments)

**Yêu cầu**: Lấy user với posts và comments của từng post.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(comments()),posts.title,posts.comments.text,posts.comments.author,posts.comments.created_at&limit=1
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Cấp 1 | `user` | Bảng chính |
| Cấp 2 | `posts(comments())` | Join posts và khai báo sub-join comments |
| Cấp 3 | `posts.comments.text,posts.comments.author` | Truy cập fields của comments qua posts |

**Response mẫu**:
```json
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "thuan",
      "posts": [
        {
          "_id": "507f1f77bcf86cd799439019",
          "title": "Hướng dẫn sử dụng API",
          "comments": [
            {
              "_id": "507f1f77bcf86cd799439023",
              "text": "Bài viết rất hữu ích!",
              "author": "reader_1",
              "created_at": "2024-02-16T08:15:00Z"
            }
          ]
        }
      ]
    }
  ]
}
```

---

### Ví dụ 5: Multiple auto-join cùng cấp

**Yêu cầu**: Lấy user với cả posts và orders.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(),posts.title,posts.status,orders(),orders.order_code,orders.total,orders.status&limit=1
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Join 1 | `posts(),posts.title,posts.status` | Join với posts |
| Join 2 | `orders(),orders.order_code,orders.total,orders.status` | Join với orders |

---

### Ví dụ 6: Filter phức tạp với nested fields

**Yêu cầu**: Tìm user theo role trong profile object.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?profile.role=eq."admin"&limit=10
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Filter - Nested | `profile.role=eq."admin"` | Truy cập field `role` trong object `profile` |

---

### Ví dụ 7: Logical operators phức tạp

**Yêu cầu**: Lấy users tuổi > 18 VÀ (status=active HOẶC profile.type=vip).

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?and=(age=gt.18,or=(status=eq.active,profile.type=eq."vip"))&limit=10
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Logical AND | `and=(...)` | Kết hợp điều kiện bằng AND |
| Condition 1 | `age=gt.18` | Tuổi lớn hơn 18 |
| Logical OR | `or=(status=eq.active,profile.type=eq."vip")` | Status active HOẶC type vip |

---

### Ví dụ 8: Search với contains và multiple filters

**Yêu cầu**: Tìm products tên chứa "iPhone", giá ≤ 20 triệu, status active.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/products?name=contains."iPhone"&price=lte.20000000&status=eq.active&order=-created_at&limit=20
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Filter 1 | `name=contains."iPhone"` | Tên chứa "iPhone" |
| Filter 2 | `price=lte.20000000` | Giá ≤ 20 triệu |
| Filter 3 | `status=eq.active` | Status active |
| Sort | `-created_at` | Sắp xếp theo ngày tạo giảm dần |

---

### Ví dụ 9: Between operator

**Yêu cầu**: Tìm products giá từ 1 triệu đến 5 triệu.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/products?price=between.(1000000,5000000)&limit=20
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Filter | `price=between.(1000000,5000000)` | Giá từ 1 triệu đến 5 triệu (dùng dấu ngoặc tròn) |

---

### Ví dụ 10: Auto-join với filter phức tạp

**Yêu cầu**: Lấy orders của khách hàng ở HCM có items giá ≥ 100k.

**Endpoint**:
```
GET https://docss-api.mangoads.com.vn/api/v1/orders?select=id,order_code,customer(),customer.name,customer.email,customer.address,items(),items.name,items.price,items.quantity&customer.address.city=eq."Ho Chi Minh"&items.price=gte.100000&limit=10
```

**Giải thích**:
| Phần | Giá Trị | Ý Nghĩa |
|------|---------|---------|
| Join 1 | `customer(),customer.name,customer.email,customer.address` | Join customer |
| Join 2 | `items(),items.name,items.price,items.quantity` | Join items |
| Filter 1 | `customer.address.city=eq."Ho Chi Minh"` | Thành phố HCM (nested 2 cấp) |
| Filter 2 | `items.price=gte.100000` | Giá items ≥ 100k |

---

## Best Practices

### Performance

1. **Chỉ select fields cần thiết** - Tránh `select=*` khi có thể
2. **Luôn dùng pagination** - Tránh lấy quá nhiều records (`limit` hợp lý)
3. **Filter trước khi join** - Giảm data cần xử lý
4. **Giới hạn join depth** - Tối đa 2-3 levels nested joins
5. **Sử dụng count=exact cẩn thận** - Chỉ khi thực sự cần tổng số

### Security

1. **Không expose sensitive fields** - Loại trừ `password`, `tokens`, `secrets`
2. **Validate user input** - Tránh injection attacks
3. **Giới hạn limit** - Không cho phép limit quá lớn

### Readability

1. **Sắp xếp params theo thứ tự**: `select → filters → order → pagination`
2. **Dùng shorthand khi đơn giản**: `status=active` thay vì `status=eq.active`
3. **Dùng dấu ngoặc kép cho string**: `name=eq."John Doe"` rõ ràng hơn

### Data Types và URL Encoding

1. **ObjectId**: Hệ thống tự động convert `_id` sang ObjectId
2. **Date**: Dùng ISO 8601: `created_at=gte."2024-01-01T00:00:00Z"`
3. **String có khoảng trắng**: Dùng dấu ngoặc kép và URL encode: `name=eq."John%20Doe"`
4. **String có ký tự đặc biệt**: URL encode đúng cách: `desc=contains."100%25"`

---

## Xử Lý Yêu Cầu Không Rõ Ràng

Khi người dùng yêu cầu không đủ thông tin, **HỎI LẠI** trước khi viết query:

### Template câu hỏi:

```
Để viết query chính xác, tôi cần thêm thông tin:

1. **Collection**: Bạn muốn query từ collection nào? (user, posts, products, orders...)
2. **Fields**: Cần lấy những fields nào? (hoặc tất cả fields?)
3. **Relationships**: Có cần auto-join với collection khác không?
   - Nếu có: Tên relationship? Lấy fields nào từ relationship?
   - Nested joins? (ví dụ: posts → comments)
4. **Filters**: Điều kiện lọc cụ thể? (status, price, date, nested fields...)
5. **Sort**: Sắp xếp theo field nào? Tăng dần hay giảm dần?
6. **Pagination**: Cần bao nhiêu records? Skip bao nhiêu? Có cần count không?
```

---

## Đề Xuất Tối Ưu

Khi query có thể tối ưu, đề xuất alternatives:

```
### 🚀 Đề Xuất Tối Ưu

**Query hiện tại**:
{current_query}

**Vấn đề**:
{issue}

**Query tối ưu**:
{optimized_query}

**Lý do**:
{reason}
```

---

## Error Handling

API trả về error khi có lỗi:

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Unsupported operator: invalid_op"
}
```

**Các lỗi thường gặp:**

1. **Operator không hợp lệ**: Chỉ dùng operators trong danh sách
2. **Syntax sai**: Kiểm tra dấu ngoặc, dấu chấm, format
3. **Missing tenant**: Một số entity cần header `X-Tenant-Id`

---

## Response Format Chuẩn

Tất cả response từ API đều có cấu trúc:

```json
{
  "message": "Request successful",
  "statusCode": 200,
  "data": [...],      // Array hoặc Object
  "meta": {           // Phân trang (nếu có)
    "total": 100,
    "last_page": 10,
    "current_page": 1
  }
}
```

---

## Ví Dụ cURL

```bash
# Lấy users với select
curl -s "https://docss-api.mangoads.com.vn/api/v1/user?limit=2&select=username,email" \
  -H "Accept: application/json"

# Filter đơn giản
curl -s "https://docss-api.mangoads.com.vn/api/v1/user?name=eq.\"thuan\"&limit=3" \
  -H "Accept: application/json"

# Filter nested fields
curl -s "https://docss-api.mangoads.com.vn/api/v1/user?profile.role=eq.\"admin\"&limit=3" \
  -H "Accept: application/json"

# Logical operators
curl -s "https://docss-api.mangoads.com.vn/api/v1/user?and=(name=eq.\"thuan\",profile.role=eq.\"admin\")&limit=5" \
  -H "Accept: application/json"

# Entity cần tenant
curl -s "https://docss-api.mangoads.com.vn/api/v1/menu?limit=5" \
  -H "Accept: application/json" \
  -H "X-Tenant-Id: default"
```

---

## Tóm Tắt - Checklist Trước Khi Trả Về Query

- [ ] Base URL đúng: `https://docss-api.mangoads.com.vn/api/v1/`
- [ ] Select dùng `?select=` KHÔNG phải `?fields=`
- [ ] Embedded relationships dùng `relation(),relation.field` KHÔNG phải `relation(field)`
- [ ] Operators nằm trong danh sách cho phép
- [ ] String có khoảng trắng dùng dấu ngoặc kép
- [ ] Between dùng `(value1,value2)` KHÔNG phải `[value1,value2]`
- [ ] Logical operators có dấu ngoặc `()`
- [ ] Thứ tự params: select → filters → order → pagination
- [ ] Giải thích đầy đủ từng phần của query

**NHỚ: KHI KHÔNG CHẮC CHẮN, HỎI LẠI NGƯỜI DÙNG - ĐỪNG TỰ ĐOÁN!**
