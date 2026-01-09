# 🚀 Team Task Management System - Hệ thống quản lý task tự động

Hệ thống quản lý task toàn diện cho team dev, sử dụng Google Sheets + n8n + Telegram để tự động hóa việc nhắc nhở và báo cáo.

## 📋 Tổng quan

Hệ thống bao gồm:
- ✅ **5 workflows n8n** tự động nhắc task và báo cáo
- 📊 **Google Sheets** làm database đơn giản
- 📱 **Telegram Bot** gửi thông báo tự động
- 🤖 **AI Analysis** (OpenAI) phân tích hiệu suất team

## 🎯 Tính năng chính

### 1️⃣ Workflows tự động

| Workflow | Mô tả | Thời gian chạy |
|----------|-------|----------------|
| **Flow A** | Nhắc task sắp đến deadline (2 ngày trước) | 08:00 hàng ngày |
| **Flow B** | Nhắc task overdue + escalate cho leader nếu quá 5 ngày | 09:00 hàng ngày |
| **Flow C** | Nhắc task không được update quá 3 ngày (stale) | 10:00 hàng ngày |
| **Flow D** | Báo cáo daily cho leader (thống kê tổng quan) | 08:30 hàng ngày |
| **Flow E** | Báo cáo weekly với AI analysis | 09:00 Thứ Hai |
| **Flow F** | Alert khi có task mới được tạo | Mỗi 15 phút |

### 2️⃣ Google Sheets Structure

#### Sheet: **TASKS**
Đây là sheet chính, dev chỉ cần quan tâm sheet này.

| Cột | Tên | Ai sửa | Mô tả |
|-----|-----|--------|-------|
| A | TaskID | Manager | Tự sinh (TASK-001) |
| B | Title | Manager | Mô tả ngắn task |
| C | Description | Manager | Chi tiết (có thể paste spec) |
| D | Type | Manager | Feature / Bug / Tech / Ops |
| E | Priority | Manager | P0 / P1 / P2 |
| F | **Status** | **Dev** | Todo / Doing / Blocked / Review / Done |
| G | Owner | Manager | Tên dev được assign |
| H | Application | Manager | App-Core, App-Auth, App-API |
| I | Estimate | Manager | 1d / 2d / 5h |
| J | StartDate | Manager | Ngày bắt đầu |
| K | Deadline | Manager | Deadline |
| L | **LastUpdate** | **Dev** | Ngày cập nhật gần nhất |
| M | **BlockReason** | **Dev** | Nếu Status = Blocked |
| N | ManagerNote | Manager | Ghi chú của leader |

> ⚠️ **Rule quan trọng**: Dev chỉ được sửa 3 cột: `Status`, `LastUpdate`, `BlockReason`

#### Sheet: **USERS**

| Cột | Tên | Mô tả |
|-----|-----|-------|
| A | Name | Tên thành viên |
| B | Email | Email |
| C | TelegramID | Telegram Chat ID để gửi thông báo |
| D | Role | Manager / Member |

#### Sheet: **SETTINGS**

| Key | Value | Mô tả |
|-----|-------|-------|
| reminder_before_days | 2 | Nhắc trước deadline bao nhiêu ngày |
| stale_task_days | 3 | Task không update bao nhiêu ngày = stale |
| overdue_escalate_days | 5 | Overdue bao nhiêu ngày thì báo leader |
| report_time | 08:30 | Giờ gửi báo cáo daily |

## 🛠️ Hướng dẫn Setup

### Bước 1: Tạo Google Sheet

1. Tạo Google Sheet mới
2. Tạo 3 sheets: `TASKS`, `USERS`, `SETTINGS`
3. Copy dữ liệu từ các file template trong folder `google-sheets-templates/`:
   - `TASKS-template.csv` → Sheet TASKS
   - `USERS-template.csv` → Sheet USERS
   - `SETTINGS-template.csv` → Sheet SETTINGS
4. Lưu lại Sheet ID từ URL (phần giữa `/d/` và `/edit`)
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
   ```

### Bước 2: Tạo Telegram Bot

1. Mở Telegram, chat với [@BotFather](https://t.me/botfather)
2. Gửi lệnh `/newbot` và làm theo hướng dẫn
3. Lưu lại **Bot Token** (dạng: `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`)
4. Lấy Telegram Chat ID của mỗi user:
   - Chat với bot [@userinfobot](https://t.me/userinfobot)
   - Lưu lại ID (dạng: `123456789`)
   - Cập nhật vào sheet USERS

### Bước 3: Setup n8n

1. Cài đặt n8n (nếu chưa có):
   ```bash
   npm install -g n8n
   # hoặc
   docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
   ```

2. Truy cập n8n: `http://localhost:5678`

3. Thêm Credentials:

   **a) Google Sheets OAuth2:**
   - Settings → Credentials → New Credential
   - Chọn "Google Sheets OAuth2 API"
   - Làm theo hướng dẫn để connect Google Account

   **b) Telegram Bot API:**
   - Settings → Credentials → New Credential
   - Chọn "Telegram API"
   - Paste Bot Token từ Bước 2

   **c) OpenAI API (cho Flow E):**
   - Settings → Credentials → New Credential
   - Chọn "OpenAI API"
   - Paste OpenAI API Key

### Bước 4: Import Workflows

1. Trong n8n, click **"+"** → **"Import from File"**
2. Import lần lượt 6 workflows từ folder `workflows/`:
   - `flow-a-deadline-reminder.json`
   - `flow-b-overdue-reminder.json`
   - `flow-c-stale-task-reminder.json`
   - `flow-d-daily-report.json`
   - `flow-e-weekly-report-ai.json`
   - `flow-f-new-task-alert.json`

3. Mỗi workflow cần sửa:
   - **Google Sheets nodes**: Chọn credential và nhập Sheet ID
   - **Telegram nodes**: Chọn credential
   - **OpenAI node** (Flow E only): Chọn credential

4. **Activate** từng workflow (toggle ở góc trên bên phải)

### Bước 5: Test thử

1. Mở workflow Flow A
2. Click **"Execute Workflow"** ở góc trên
3. Kiểm tra:
   - Workflow chạy thành công
   - Có gửi message vào Telegram
   - Message format đúng

## 📱 Cách sử dụng hàng ngày

### Cho Dev:

1. **Xem task được assign**: Mở Google Sheet, tìm tên mình trong cột `Owner`
2. **Bắt đầu làm task**: 
   - Sửa cột `Status` → `Doing`
   - Sửa cột `LastUpdate` → ngày hôm nay
3. **Task bị block**:
   - Sửa cột `Status` → `Blocked`
   - Sửa cột `BlockReason` → ghi rõ lý do (vd: "Chờ API backend")
   - Sửa cột `LastUpdate` → ngày hôm nay
4. **Hoàn thành task**:
   - Sửa cột `Status` → `Done`
   - Sửa cột `LastUpdate` → ngày hôm nay

> 💡 **Lưu ý**: Cập nhật `LastUpdate` mỗi khi có thay đổi để tránh nhận "stale task warning"

### Cho Leader:

- Nhận **Daily Report** lúc 08:30 mỗi ngày
- Nhận **Weekly Report + AI Analysis** lúc 09:00 Thứ Hai
- Nhận **Escalation** khi task overdue quá 5 ngày
- Quản lý toàn bộ Google Sheet (assign task, update priority, etc.)

## 📊 Các message mẫu

### Nhắc task sắp deadline:
```
⏰ Task sắp đến hạn

📋 Task: Implement user authentication
🆔 TaskID: TASK-002
📅 Deadline: 2026-01-05
📊 Status: Doing
👤 Owner: Bình

✅ Vui lòng cập nhật trước hạn.
```

### Daily Report (Leader):
```
📊 Daily Dev Report
━━━━━━━━━━━━━━━━━━

📈 Tổng quan:
📦 Tổng task: 25
✅ Done hôm qua: 5
🔄 Đang làm: 7
📝 Todo: 8
⛔ Blocked: 2
🚨 Overdue: 3

🚨 Top Overdue Tasks:
- TASK-014 (An) – 4 ngày
- TASK-019 (Bình) – 2 ngày

━━━━━━━━━━━━━━━━━━
⏰ 09/01/2026, 08:30:00
```

## 🔧 Troubleshooting

### Không nhận được message Telegram:

1. Kiểm tra Bot Token có đúng không
2. Kiểm tra Telegram ID trong sheet USERS
3. User phải start chat với bot trước (gửi `/start`)
4. Kiểm tra workflow có được activate không

### Google Sheets không đọc được:

1. Kiểm tra OAuth2 credential đã authorize chưa
2. Kiểm tra Sheet ID có đúng không
3. Kiểm tra range (A:N) có khớp với sheet không
4. Sheet phải được share với Google Account đã authorize

### Workflow không chạy đúng giờ:

1. Kiểm tra Cron expression trong Trigger node
2. Kiểm tra timezone của n8n server
3. Có thể set lại schedule hoặc trigger manually để test

## 🚀 Nâng cấp (Roadmap)

- [ ] Sync với GitHub Issues / PRs
- [ ] Auto tạo task từ commit message
- [ ] Slack integration (thay Telegram)
- [ ] Voice note → task (Speech to Text)
- [ ] Dashboard web với charts
- [ ] Mobile app notification

## 📞 Support

Nếu có vấn đề hoặc cần customize thêm, liên hệ:
- Email: tech@company.com
- Telegram: @your_username

---

Made with ❤️ for Vibe Coding Team
