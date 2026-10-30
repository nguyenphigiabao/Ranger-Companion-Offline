# Hướng dẫn chạy migration cho bảng quiz_attempts

## Bước 1: Kết nối với Supabase Database

Có 2 cách để chạy migration:

### Cách 1: Sử dụng Supabase Dashboard (Khuyên dùng)

1. Truy cập [Supabase Dashboard](https://app.supabase.com)
2. Chọn project của bạn
3. Vào **SQL Editor** (ở menu bên trái)
4. Tạo một query mới
5. Copy toàn bộ nội dung file `supabase/migrations/20260322_create_quiz_attempts.sql`
6. Paste vào SQL Editor
7. Click **Run** để thực thi

### Cách 2: Sử dụng Supabase CLI

```bash
# Nếu chưa cài Supabase CLI, cài đặt trước:
npm install -g supabase

# Login vào Supabase
supabase login

# Link project (chỉ cần làm 1 lần)
supabase link --project-ref YOUR_PROJECT_REF

# Chạy migration
supabase db push
```

## Bước 2: Kiểm tra migration đã chạy thành công

Chạy query sau trong SQL Editor để kiểm tra:

```sql
-- Kiểm tra bảng đã được tạo
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public' 
AND table_name = 'quiz_attempts';

-- Kiểm tra RLS policies
SELECT * FROM pg_policies WHERE tablename = 'quiz_attempts';

-- Kiểm tra indexes
SELECT * FROM pg_indexes WHERE tablename = 'quiz_attempts';
```

## Giải thích về bảng quiz_attempts

Bảng này lưu **tất cả các lần chơi quiz** của user (không giới hạn), khác với bảng `user_progress` chỉ lưu kết quả cao nhất.

### Cấu trúc:
- `id`: UUID primary key
- `user_id`: ID của user (foreign key)
- `lesson_id`: ID của bài học (foreign key)
- `points_earned`: Điểm đạt được trong lần chơi này
- `correct_answers`: Số câu trả lời đúng
- `total_questions`: Tổng số câu hỏi
- `completed_at`: Thời gian hoàn thành
- `created_at`: Thời gian tạo record

### RLS Policies:
- User chỉ có thể xem lịch sử của chính họ
- User chỉ có thể insert lịch sử của chính họ

## Lưu ý

- Migration này an toàn và không ảnh hưởng đến dữ liệu hiện tại
- Bảng mới sẽ tự động bắt đầu lưu lịch sử từ lần chơi quiz tiếp theo
- Giới hạn hiển thị 50 lần chơi gần nhất để tối ưu performance

---

## Migration: Nội dung bài học dạng Markdown (`lessons.content`)

**File:** `supabase/migrations/20260322_lessons_content_markdown.sql`

- Chỉ **UPDATE** cột `content` của bảng **`public.lessons`** (10 bài học seed).
- **Không** thay đổi `categories`, `quizzes`, `user_progress`, v.v.
- Dùng dollar-quoting (`$md1101$` …) để tránh lỗi escape ký tự.

Chạy giống các migration khác (SQL Editor hoặc `supabase db push`).

Schema/seed cyber safety cũ đã gỡ khỏi repo; schema app hiện tại nằm trong `supabase/ranger_app_schema.sql`.
