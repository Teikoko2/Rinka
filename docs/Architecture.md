# Rinka Architecture

## 1. Overview

Rinka is an AI-based daily conversation and safety check service for elderly people and single-person households.

The first implementation should prioritize:

1. Web application for family/admin users
2. LINE-based conversation interface for the monitored person
3. iPhone app
4. Android app
5. Robotics integration in the future

The MVP should keep initial operating costs low while preserving a scalable architecture for rapid growth.

---

## 2. Core Concept

Rinka should not feel like a surveillance or monitoring system.

The core product principle is:

> Natural conversation becomes safety confirmation.

The monitored person talks with Rinka in LINE.  
The family can check the latest response status from a web dashboard.  
If no conversation occurs for a certain period, Rinka notifies registered contacts.

---

## 3. Initial System Architecture

```text
[Monitored Person]
  LINE
    ↓
[LINE Messaging API]
    ↓ Webhook
[API Layer]
  Next.js API Route / Cloudflare Workers
    ↓
[Conversation Controller]
    ↓
[LLM Provider]
  Gemini Flash-Lite / other low-cost LLM
    ↓
[Database]
  Supabase PostgreSQL
    ↓
[Family Web Dashboard]
  Next.js + Supabase Auth
```

---

## 4. Recommended Technology Stack

### Frontend

```text
Next.js
TypeScript
Tailwind CSS
```

### Backend / API

```text
Next.js API Routes
or
Cloudflare Workers
```

### Database / Auth

```text
Supabase Auth
Supabase PostgreSQL
```

### Messaging

```text
LINE Messaging API
LINE Login
LINE Mini App in future phase
```

### LLM

```text
Primary: Gemini Flash-Lite or equivalent low-cost model
Secondary: OpenAI mini-class model for important analysis
Future: Llama-based self-hosted model if cost and scale require it
```

### Notification

```text
Email notification
LINE notification
Future: SMS / phone call integration
```

---

## 5. Platform Strategy

### Phase 1: Web Application

Purpose:

- Family account management
- Monitored person registration
- LINE linking
- Status dashboard
- Alert confirmation
- Notification settings

Target user:

- Family members
- Caregivers
- Future business users

---

### Phase 2: LINE Interface

Purpose:

- Daily conversation with the monitored person
- Simple safety confirmation
- Low-friction adoption
- No app installation required

Initial implementation should use LINE Bot rather than a full LINE Mini App.

Reason:

- Faster MVP development
- Lower complexity
- Easier testing
- Works naturally in daily life

---

### Phase 3: iPhone App

Purpose:

- Richer user experience
- Voice interaction
- Push notifications
- More personal AI companion experience

Recommended framework:

```text
React Native / Expo
```

Reason:

- Works well with VS Code
- TypeScript-based development
- Can later expand to Android
- Faster vibe-coding workflow

---

### Phase 4: Android App

Purpose:

- Expand user base
- Support wider elderly and family users

Recommended framework:

```text
React Native / Expo
```

---

### Phase 5: Robotics

Purpose:

- Physical companion experience
- Voice-based interaction
- Emotional attachment
- Differentiation from ordinary apps

Initial robot should be a simple terminal device.

```text
Small speaker
Microphone
LED eyes
Wi-Fi connection
Server API integration
```

The robot should not run a full LLM locally at first.  
It should act as an input/output device connected to the Rinka backend.

---

## 6. Cost Control Architecture

The service must avoid high LLM costs, especially for free users.

### Basic Principle

```text
Do not send every message to an LLM.
```

Instead, use a layered response model.

```text
User message
  ↓
Rule-based classification
  ├ Greeting / simple reply → template response
  ├ Safety check → template response + database update
  ├ General conversation → low-cost LLM
  ├ Health risk / emergency word → high-priority handling
  └ Abuse / spam → rate limit
```

---

## 7. Free User Cost Policy

For free users:

```text
Daily LLM conversation limit: 3 to 5 turns
Response length: short
Conversation history retention: 7 days
Family notification: limited or unavailable
Weekly report: unavailable
Advanced analysis: unavailable
```

The free plan should demonstrate value without creating uncontrolled cost.

---

## 8. Paid User Cost Policy

For paid users:

```text
Daily LLM conversation limit: higher limit
Family notification: enabled
Weekly summary: enabled
Conversation history: extended retention
Risk detection: enhanced
Multiple contacts: enabled
```

---

## 9. Scalability Strategy

The architecture should allow gradual migration from simple MVP to scalable service.

### Initial Stage

```text
Next.js
Supabase
LINE Messaging API
Gemini Flash-Lite
Email notification
```

### Growth Stage

```text
Cloudflare Workers for webhook/API
Queue-based alert processing
Background jobs
Separate notification service
Log storage optimization
```

### Large Scale Stage

```text
Dedicated backend services
Message queue
LLM routing layer
Usage-based billing control
Analytics warehouse
B2B tenant management
```

---

## 10. Suggested Repository Structure

```text
/apps
  /web
    Family web dashboard

  /line-bot
    LINE webhook and conversation API

/packages
  /core
    Safety check logic
    Conversation rules
    Alert calculation

  /db
    Database types
    Supabase client
    Migration files

  /prompts
    LLM prompts
    System prompts
    Conversation templates

/docs
  Architecture.md
  MvpSpec.md
  CostPolicy.md
  Roadmap.md
```

---

## 11. Main Data Flow

### Daily Conversation

```text
1. User sends message to Rinka on LINE
2. LINE sends webhook to backend
3. Backend identifies LINE userId
4. Backend updates latest response timestamp
5. Backend classifies the message
6. Backend returns template or LLM-generated reply
7. Conversation is stored in database
```

---

### Safety Alert

```text
1. Scheduled job checks latest response timestamp
2. If no response for 48 hours, status becomes warning
3. If no response for 72 hours, status becomes critical
4. Registered family contact receives notification
5. Alert is recorded in database
```

---

## 12. Key Design Principle

Rinka is not a monitoring tool.

Rinka is a relationship-based safety confirmation service.

The product should always prioritize:

```text
Friendly
Lightweight
Low pressure
Low cost
Scalable
Privacy-conscious
```
