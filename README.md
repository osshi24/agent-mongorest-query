# MGS Query Agent SDK

Agent SDK sử dụng Claude để tự động tạo MGS Core V2 API query parameters thông qua skill `mongorest-query`.

## Cài đặt

```bash
npm install
```

## Sử dụng

1. **Tạo file prompt.md** với yêu cầu query của bạn:

```markdown
# MGS Query Request

Tôi cần query để lấy danh sách users active...

Hãy sử dụng skill mongorest-query để tạo endpoint.
```

2. **Chạy agent**:

```bash
node index.js
```

3. **Kết quả** sẽ được lưu vào `output/query-result.md`

## Cấu trúc Project

```
.
├── index.js              # Main agent SDK
├── prompt.md             # Input prompt (yêu cầu query)
├── output/
│   └── query-result.md  # Output result
└── .claude/
    └── skills/
        └── mongorest-query/  # Skill definition
```

## Ví dụ Prompt

### Ví dụ 1: Lấy users với posts

```markdown
Tôi cần query để lấy danh sách users active, kèm theo posts.
- Collection: users
- Filter: status = active
- Join: posts (title, content)
- Sort: theo createdAt mới nhất
- Limit: 20

Hãy sử dụng skill mongorest-query.
```

### Ví dụ 2: Search products

```markdown
Tìm products có tên chứa "laptop", category là electronics, giá từ 500-2000.
- Fields: name, price, category, stock
- Sort: theo giá tăng dần

Hãy sử dụng skill mongorest-query.
```

### Ví dụ 3: Complex filters

```markdown
Lấy users là admin HOẶC moderator, đang active, không bị banned.
- Fields: id, name, email, role
- Sort: theo tên

Hãy sử dụng skill mongorest-query.
```

## Output Format

File `output/query-result.md` sẽ chứa:
- Timestamp
- Prompt gốc
- Endpoint đầy đủ
- Giải thích chi tiết
- Lưu ý và tối ưu (nếu có)

## Features

- ✅ Đọc prompt từ file markdown
- ✅ Tự động gọi skill mongorest-query
- ✅ Validate và tối ưu query
- ✅ Giải thích chi tiết từng phần
- ✅ Ghi kết quả ra file .md
- ✅ Console log real-time
- ✅ Error handling

## Requirements

- Node.js >= 18
- @anthropic-ai/claude-agent-sdk
- Skill mongorest-query được cấu hình trong `.claude/skills/`

## Troubleshooting

### File prompt.md không tồn tại

```
❌ File prompt.md không tồn tại!
💡 Vui lòng tạo file prompt.md với yêu cầu query của bạn.
```

→ Tạo file `prompt.md` trong thư mục gốc

### Skill không được load

→ Kiểm tra đường dẫn `.claude/skills/mongorest-query/skill.md` tồn tại

## License

MIT
