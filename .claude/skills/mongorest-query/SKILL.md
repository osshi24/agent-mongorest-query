---
name: mongorest-query
description: Tạo query params cho MGS Core V2 API (PostgREST Style). Trả về ngay endpoint KHÔNG hỏi lại, KHÔNG giải thích. Tuân thủ CHẶT CHẼ syntax chuẩn, KHÔNG tự tạo syntax mới.
---

# MGS Query Params Writer (PostgREST Style)

Tạo query parameters theo chuẩn PostgREST Style cho MGS Core V2 API.

## ⚠️ CRITICAL RULES - BẮT BUỘC TUÂN THỦ

**NGHIÊM NGẶT:**

1. **CHỈ SỬ DỤNG SYNTAX TRONG DOCS** - KHÔNG TỰ TẠO SYNTAX MỚI
2. **CHỈ SỬ DỤNG OPERATORS ĐÃ LIỆT KÊ** - KHÔNG PHÁT MINH OPERATORS MỚI
3. **EMBEDDED RELATIONSHIPS** - Bắt buộc: `relation(),relation.field` KHÔNG DÙNG `relation(field)`
4. **STRING CÓ KHOẢNG TRẮNG** - Luôn dùng dấu ngoặc kép: `name=eq."John Doe"`
5. **VALIDATE SYNTAX** - Kiểm tra syntax trước khi trả về
6. **KHÔNG HỎI LẠI** - Suy luận từ context, trả về ngay query
7. **KHÔNG GIẢI THÍCH** - Chỉ trả về endpoint

---

## Base URL

```
https://docss-api.mangoads.com.vn/api/v1/
```

---

## Output Format

**CHỈ TRẢ VỀ ENDPOINT - KHÔNG GIẢI THÍCH:**

```
GET https://docss-api.mangoads.com.vn/api/v1/{collection}?{query_params}
```

**Thứ tự params**: `select → filters → order → limit/skip/count`

---

## Query Syntax Reference

### 1. Select

**Cú pháp**: `?select=field1,field2,field3`

```bash
?select=id,name,email          # Chọn fields cụ thể
?select=*                       # Tất cả fields
?select=*,-password,-secretKey  # Loại trừ fields
```

### 2. Filtering

**Format**: `field=operator.value` hoặc `field.nested=operator.value`

**String có khoảng trắng**: `field=operator."value with spaces"`

#### Operators (CHỈ SỬ DỤNG TRONG DANH SÁCH)

**So sánh:**
- `eq` - Bằng: `status=eq.active`
- `neq` - Không bằng: `status=neq.deleted`
- `gt` - Lớn hơn: `age=gt.18`
- `gte` - Lớn hơn hoặc bằng: `age=gte.18`
- `lt` - Nhỏ hơn: `price=lt.1000000`
- `lte` - Nhỏ hơn hoặc bằng: `price=lte.1000000`

**Array:**
- `in` - Trong danh sách: `category=in.[phone,laptop]`
- `nin` - Không trong danh sách: `status=nin.[deleted,archived]`

**String:**
- `like` - Pattern (case-sensitive): `name=like.*John*`
- `ilike` - Pattern (case-insensitive): `title=ilike.*mobile*`
- `contains` - Chứa: `description=contains.smartphone`
- `startswith` - Bắt đầu: `email=startswith.admin`
- `endswith` - Kết thúc: `email=endswith.@gmail.com`
- `not_contains` - Không chứa: `description=not_contains.old`
- `not_startswith` - Không bắt đầu: `code=not_startswith.TEMP`
- `not_endswith` - Không kết thúc: `filename=not_endswith..tmp`
- `regex` - Regex: `phone=regex."^0[0-9]{9}$"`

**Đặc biệt:**
- `is` - Null: `deletedAt=is.null`
- `not.is` - Không null: `email=not.is.null`
- `exists` - Tồn tại: `metadata=exists.true`
- `between` - Trong khoảng: `price=between.(1000000,5000000)`
- `not_between` - Ngoài khoảng: `age=not_between.(18,60)`

**Lưu ý:**
- `between` dùng `()` KHÔNG phải `[]`
- String có khoảng trắng PHẢI dùng `"value"`

#### Shorthand

```bash
?status=active    # = status=eq.active
?age=18          # = age=eq.18
```

#### Logical Operators

```bash
# AND (mặc định)
?status=eq.active&age=gt.18

# OR
?or=(status=eq.active,status=eq.pending)

# NOT
?not=status=eq.deleted

# AND nhiều điều kiện
?and=(name=eq."thuan",profile.role=eq."admin")

# Nested
?and=(age=gt.18,or=(status=eq.active,profile.type=eq."vip"))
```

### 3. Embedded Relationships (Auto-join)

**⚠️ CÚ PHÁP QUAN TRỌNG:**

```bash
# ✅ ĐÚNG
?select=id,name,posts(),posts.title,posts.content

# ❌ SAI - KHÔNG DÙNG
?select=id,name,posts(title,content)
```

**Syntax:**

```bash
# Khai báo relationship
relation()

# Truy cập fields
relation.field1,relation.field2

# Kết hợp
?select=main_fields,relation(),relation.field1,relation.field2
```

**Ví dụ:**

```bash
# User với posts
?select=id,name,posts(),posts.title,posts.content

# User với posts và filter
?select=id,name,posts(),posts.title&posts.status=eq.published

# Nested (3 cấp)
?select=id,name,posts(comments()),posts.title,posts.comments.text

# Multiple joins
?select=id,name,posts(),posts.title,orders(),orders.total

# Join với filter phức tạp
?select=id,customer(),customer.name,items(),items.price&customer.address.city=eq."Ho Chi Minh"&items.price=gte.100000
```

### 4. Sorting

**Cú pháp**: `?order=field1,field2,-field3`

```bash
?order=name                          # Tăng dần
?order=-created_at                   # Giảm dần
?order=-score,name                   # Multi-field
?order=-priority,status,-updated_at  # Nhiều fields
```

### 5. Pagination

```bash
?limit=20              # Giới hạn
?skip=40               # Bỏ qua
?offset=40             # = skip
?count=exact           # Đếm tổng
?limit=20&skip=40      # Kết hợp
```

### 6. Value Types

| Type | Cú Pháp | Ví Dụ |
|------|---------|-------|
| String | `field=value` hoặc `field="value"` | `name=John`, `name=eq."John Doe"` |
| Number | `field=number` | `age=25`, `price=99.99` |
| Boolean | `field=true/false` | `active=true` |
| Null | `field=is.null` | `deletedAt=is.null` |
| Array | `field=in.[val1,val2]` | `status=in.[active,pending]` |
| Date | `field=gte."ISO8601"` | `createdAt=gte."2024-01-01T00:00:00Z"` |
| ObjectId | `field=id` | `_id=eq.507f1f77bcf86cd799439011` |

---

## Validation Checklist

**Kiểm tra trước khi trả về:**

- [ ] Base URL: `https://docss-api.mangoads.com.vn/api/v1/`
- [ ] Select: `?select=` KHÔNG phải `?fields=`
- [ ] Embedded: `relation(),relation.field` KHÔNG phải `relation(field)`
- [ ] Operators trong danh sách cho phép
- [ ] String có khoảng trắng dùng `"value"`
- [ ] Between dùng `(val1,val2)` KHÔNG phải `[val1,val2]`
- [ ] Logical operators có `()`
- [ ] Thứ tự: select → filters → order → pagination
- [ ] KHÔNG giải thích - CHỈ trả về endpoint

---

## Ví Dụ Tham Khảo

### Ví dụ 1: Filter + Select

**Input**: Lấy users active, chỉ username và email, limit 20

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=username,email&status=eq.active&limit=20
```

---

### Ví dụ 2: Auto-join cơ bản

**Input**: User với posts, lấy title và status

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(),posts.title,posts.status&limit=10
```

---

### Ví dụ 3: Auto-join với filter

**Input**: User với posts published

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(),posts.title,posts.content&posts.status=eq.published&limit=10
```

---

### Ví dụ 4: Nested join 3 cấp

**Input**: User → posts → comments

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(comments()),posts.title,posts.comments.text,posts.comments.author&limit=5
```

---

### Ví dụ 5: Multiple joins

**Input**: User với posts và orders

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?select=id,name,posts(),posts.title,orders(),orders.total&limit=10
```

---

### Ví dụ 6: Filter nested fields

**Input**: User có profile.role = admin

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?profile.role=eq."admin"&limit=10
```

---

### Ví dụ 7: Logical operators

**Input**: Users tuổi > 18 VÀ (status=active HOẶC type=vip)

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/user?and=(age=gt.18,or=(status=eq.active,profile.type=eq."vip"))&limit=10
```

---

### Ví dụ 8: Search với contains

**Input**: Products tên chứa "iPhone", giá ≤ 20tr, active

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/products?name=contains."iPhone"&price=lte.20000000&status=eq.active&order=-created_at&limit=20
```

---

### Ví dụ 9: Between operator

**Input**: Products giá từ 1tr đến 5tr

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/products?price=between.(1000000,5000000)&limit=20
```

---

### Ví dụ 10: Join với filter phức tạp

**Input**: Orders khách HCM, items giá ≥ 100k

**Output**:
```
GET https://docss-api.mangoads.com.vn/api/v1/orders?select=id,order_code,customer(),customer.name,items(),items.name,items.price&customer.address.city=eq."Ho Chi Minh"&items.price=gte.100000&limit=10
```

---

## Best Practices

1. **Chỉ select fields cần thiết** - Tránh `select=*`
2. **Luôn dùng limit** - Tránh lấy quá nhiều
3. **Filter trước join** - Giảm data
4. **Join depth ≤ 3** - Tối đa 3 levels
5. **Loại trừ sensitive fields** - Không expose password, tokens

---

## Response Format (Tham khảo)

```json
{
  "message": "Request successful",
  "statusCode": 200,
  "data": [...],
  "meta": {
    "total": 100,
    "last_page": 10,
    "current_page": 1
  }
}
```

---

## Tóm Tắt

**NHỚ:**
1. Base URL: `https://docss-api.mangoads.com.vn/api/v1/`
2. Embedded: `relation(),relation.field`
3. Between: `(val1,val2)` KHÔNG `[val1,val2]`
4. String: Dùng `"value"` nếu có khoảng trắng
5. CHỈ trả về endpoint - KHÔNG giải thích
6. KHÔNG hỏi lại - Suy luận và trả về ngay
