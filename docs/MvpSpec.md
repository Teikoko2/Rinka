# Rinka MVP Specification

## 1. MVP Goal

The MVP goal is to validate the core value of Rinka:

> A monitored person talks with Rinka through LINE, and family members can confirm safety status from a web dashboard.

The first MVP should not aim to build a full AI companion.  
It should prove that daily conversation can function as natural safety confirmation.

---

## 2. MVP Scope

### Included

```text
Family account registration
Family login
Monitored person registration
LINE user linking
LINE chat with Rinka
Latest response timestamp tracking
Basic conversation reply
Warning status after 48 hours of no response
Critical status after 72 hours of no response
Family notification
Status dashboard
```

### Not Included in Initial MVP

```text
Native iPhone app
Native Android app
Robotics
Payment integration
Advanced health analysis
Medical diagnosis
Legal will execution
Voice conversation
Complex AI memory
```

---

## 3. User Types

### Family User

The family user manages the monitored person and receives alerts.

Main actions:

```text
Register account
Register monitored person
Set notification contact
Check latest response status
Check alert status
Receive notification
```

---

### Monitored Person

The monitored person uses LINE to talk with Rinka.

Main actions:

```text
Add Rinka on LINE
Link account
Send daily messages
Reply to Rinka
Receive simple friendly messages
```

---

### System Admin

The system admin monitors service health and usage.

Main actions:

```text
Check user count
Check message count
Check LLM usage
Check failed notifications
Check system errors
```

---

## 4. MVP Functional Requirements

## 4.1 Family Account

### Requirement

Family users can create an account and log in.

### Implementation

```text
Supabase Auth
Email login
Password login
```

### Required Fields

```text
family_user_id
email
display_name
created_at
updated_at
```

---

## 4.2 Monitored Person Registration

### Requirement

Family users can register a monitored person.

### Required Fields

```text
monitored_person_id
family_user_id
display_name
nickname
timezone
alert_after_hours_warning
alert_after_hours_critical
created_at
updated_at
```

### Default Alert Settings

```text
warning: 48 hours
critical: 72 hours
```

---

## 4.3 LINE Linking

### Requirement

The monitored person can link their LINE account to the registered profile.

### MVP Method

Generate a linking code from the web dashboard.

Example:

```text
RINKA-123456
```

The monitored person sends the code to Rinka in LINE.

### Required Data

```text
line_user_id
monitored_person_id
linked_at
```

---

## 4.4 LINE Webhook

### Requirement

The backend receives messages from LINE.

### Process

```text
1. Receive LINE webhook
2. Validate request signature
3. Extract line_user_id and message text
4. Identify monitored person
5. Save message
6. Update latest_response_at
7. Generate reply
8. Send reply through LINE Messaging API
```

---

## 4.5 Conversation Reply

### Requirement

Rinka replies to the monitored person in a friendly and short tone.

### Priority Order

```text
1. Emergency keyword detection
2. Template reply for simple greetings
3. Template reply for safety check
4. Low-cost LLM reply for general conversation
5. Fallback reply
```

### Example Template Replies

```text
おはようございます。今日もお話しできてうれしいです。
今日はどんな一日になりそうですか？

こんにちは。体調はいかがですか？
一言でも返してくれて安心しました。

話してくれてありがとうございます。
無理せず、ゆっくりいきましょう。
```

---

## 4.6 Emergency Keyword Detection

### Requirement

The system should detect urgent expressions.

### Example Keywords

```text
助けて
倒れた
苦しい
動けない
胸が痛い
救急車
事故
転んだ
意識
```

### MVP Behavior

```text
1. Mark message as emergency_candidate
2. Save alert
3. Notify registered family contact
4. Reply with a short safety-oriented message
```

### Example Reply

```text
心配です。近くに人がいれば声をかけてください。
必要ならすぐに119へ連絡してください。
```

Important:

Rinka should not provide medical diagnosis.

---

## 4.7 Latest Response Tracking

### Requirement

Every valid message from the monitored person updates the latest response timestamp.

### Data

```text
latest_response_at
latest_message_preview
status
```

---

## 4.8 Safety Status Calculation

### Requirement

The system calculates safety status based on latest response time.

### Status Rules

```text
normal:
  latest response within 48 hours

warning:
  no response for 48 to 72 hours

critical:
  no response for more than 72 hours

emergency:
  emergency keyword detected
```

---

## 4.9 Scheduled Alert Check

### Requirement

A scheduled job checks inactive users.

### Frequency

```text
Every 1 hour
```

### Process

```text
1. Get monitored persons
2. Compare current time with latest_response_at
3. Update status
4. Create alert if status changed
5. Notify family if needed
```

---

## 4.10 Family Notification

### Requirement

Family users receive notification when warning or critical status occurs.

### Initial Notification Method

```text
Email
```

### Future Notification Method

```text
LINE notification
SMS
Phone call
Push notification
```

### Notification Example

```text
件名: Rinka 安否確認のお知らせ

リンカからのお知らせです。

対象者: 山田 花子さん
状態: 72時間以上会話がありません
最後の会話: 2026-04-29 08:30

念のため、ご連絡をおすすめします。
```

---

## 4.11 Family Dashboard

### Requirement

Family users can view monitored person status.

### Display Items

```text
Name
Current status
Latest response time
Latest message preview
Alert history
Notification setting
LINE link status
```

### Status Display

```text
normal: 通常
warning: 注意
critical: 警戒
emergency: 緊急候補
```

---

## 5. Database Draft

## 5.1 family_users

```sql
id uuid primary key
email text not null
display_name text
created_at timestamp
updated_at timestamp
```

---

## 5.2 monitored_persons

```sql
id uuid primary key
family_user_id uuid references family_users(id)
display_name text not null
nickname text
timezone text default 'Asia/Tokyo'
line_user_id text
latest_response_at timestamp
latest_message_preview text
status text default 'normal'
warning_after_hours integer default 48
critical_after_hours integer default 72
created_at timestamp
updated_at timestamp
```

---

## 5.3 conversations

```sql
id uuid primary key
monitored_person_id uuid references monitored_persons(id)
line_user_id text
role text
message text
message_type text
llm_used boolean default false
created_at timestamp
```

---

## 5.4 alerts

```sql
id uuid primary key
monitored_person_id uuid references monitored_persons(id)
alert_type text
status text
message text
notified_at timestamp
resolved_at timestamp
created_at timestamp
```

---

## 5.5 usage_limits

```sql
id uuid primary key
monitored_person_id uuid references monitored_persons(id)
date date
message_count integer default 0
llm_count integer default 0
created_at timestamp
updated_at timestamp
```

---

## 6. LLM Usage Policy

### Free User Policy

```text
Maximum LLM replies: 3 to 5 per day
Template replies should be used whenever possible
Response should be short
Do not store long-term memory in MVP
Do not summarize all conversations automatically
```

### LLM Routing

```text
Greeting:
  template

Simple safety check:
  template

General conversation:
  low-cost LLM

Emergency candidate:
  rule detection + family notification

Family summary:
  paid feature or manual trigger
```

---

## 7. Non-Functional Requirements

## 7.1 Cost

The service must avoid cost explosion.

Required controls:

```text
Daily LLM usage limit
Message length limit
Template-first reply strategy
Rate limiting
Usage logging
Plan-based feature restriction
```

---

## 7.2 Scalability

The MVP should support future growth.

Required design:

```text
Stateless API
Database-backed user state
Webhook validation
Background alert job
LLM provider abstraction
Notification provider abstraction
```

---

## 7.3 Privacy

The service handles sensitive conversation data.

Required design:

```text
Minimum data retention
Family visibility control
No unnecessary medical judgment
Secure authentication
Database row-level security
Audit logs in future phase
```

---

## 7.4 Reliability

Safety alerts must be reliable.

Required design:

```text
Alert job retry
Notification failure log
Manual status check
Admin error monitoring
```

---

## 8. MVP Development Order

### Step 1: Project Setup

```text
Create Next.js project
Set up TypeScript
Set up Tailwind CSS
Set up Supabase project
Create environment variable structure
```

---

### Step 2: Database

```text
Create tables
Set up Supabase client
Set up Row Level Security
Create basic seed data
```

---

### Step 3: Family Web App

```text
Create login page
Create dashboard page
Create monitored person registration
Create LINE linking code display
Create status card
```

---

### Step 4: LINE Bot

```text
Create LINE channel
Set webhook URL
Implement signature validation
Receive message
Identify user
Reply with template message
Save conversation
Update latest_response_at
```

---

### Step 5: LLM Integration

```text
Create LLM provider interface
Implement low-cost LLM call
Add daily usage limit
Add template-first routing
```

---

### Step 6: Alert Job

```text
Create scheduled function
Check inactive users
Update status
Create alert record
Send email notification
```

---

### Step 7: Testing

```text
Test LINE linking
Test daily conversation
Test warning status
Test critical status
Test emergency keyword detection
Test email notification
```

---

## 9. Vibe-Coding Prompt

```md
You are building Rinka, an AI-based safety confirmation chat service.

Use:
- Next.js App Router
- TypeScript
- Tailwind CSS
- Supabase Auth
- Supabase PostgreSQL
- LINE Messaging API
- Low-cost LLM provider abstraction

Build the MVP with the following features:
1. Family user registration/login
2. Monitored person registration
3. LINE linking code
4. LINE webhook receiver
5. Conversation storage
6. latest_response_at update
7. Template-first conversation response
8. Low-cost LLM fallback
9. 48-hour warning status
10. 72-hour critical status
11. Family email notification
12. Dashboard status display

Important:
- Keep LLM cost low.
- Do not send every message to the LLM.
- Use template replies first.
- Add daily LLM usage limit.
- Design the code so iPhone, Android, and robotics clients can be added later.
```

---

## 10. MVP Success Criteria

The MVP is successful when:

```text
A monitored person can talk with Rinka on LINE.
A family user can see the latest response status on the web dashboard.
The system can detect no-response status after 48/72 hours.
The family can receive a notification.
The LLM cost can be controlled by daily usage limits.
```
