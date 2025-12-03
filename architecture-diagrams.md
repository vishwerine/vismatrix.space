# VisMatrix.space - Architecture & Technical Diagrams

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                          VisMatrix.space                            │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ CLIENT LAYER (Browser - http://localhost:3000)                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─ Nuxt 4 + Vue 3 ──────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │  Pages:                    Components:                        │  │
│  │  • Dashboard               • ProjectCard                     │  │
│  │  • Projects                • TaskList                        │  │
│  │  • Tasks (Kanban)          • ProgressChart                  │  │
│  │  • Social (Discover)       • UserCard                       │  │
│  │  • Messages                • MessageThread                  │  │
│  │  • Auth (Login/Register)   • StatsSummary                   │  │
│  │                                                               │  │
│  │  State Management: Pinia Stores                              │  │
│  │  • auth.ts (User, Token)                                    │  │
│  │  • projects.ts (Projects, Tasks)                            │  │
│  │  • messages.ts (Conversations, Messages)                    │  │
│  │                                                               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                          ↓                                            │
│                   HTTP & WebSocket                                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ API GATEWAY LAYER (http://localhost:5000/api)                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Express.js + TypeScript                                             │
│  • CORS Middleware                                                   │
│  • Authentication Middleware (JWT)                                   │
│  • Error Handling Middleware                                         │
│  • Rate Limiting (To be added)                                      │
│                                                                      │
│  ┌─ Routes ────────────────────────────────────────────────────┐   │
│  │                                                             │   │
│  │  /auth                 /projects              /tasks        │   │
│  │  ├─ POST /register     ├─ GET /               ├─ GET /    │   │
│  │  ├─ POST /login        ├─ POST /              ├─ POST /   │   │
│  │  ├─ POST /logout       ├─ GET /:id            ├─ PUT /:id │   │
│  │  ├─ POST /refresh      ├─ PUT /:id            └─ DELETE/:id│   │
│  │  └─ GET /verify        └─ DELETE /:id                      │   │
│  │                                                             │   │
│  │  /users                /messages              /analytics    │   │
│  │  ├─ GET /discover      ├─ GET /               ├─ GET /progress  │
│  │  ├─ GET /:username     ├─ POST /:userId       ├─ GET /summary   │
│  │  ├─ GET /:id/projects  ├─ PUT /:id/read       └─ GET /metrics   │
│  │  ├─ POST /:id/follow   └─ DELETE /:id                      │   │
│  │  └─ DELETE /:id/follow                                     │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌─ Controllers ──────────────────────────────────────────────┐   │
│  │  authController.ts    projectController.ts                │   │
│  │  userController.ts    taskController.ts                   │   │
│  │  messageController.ts analyticsController.ts             │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                          ↓                                            │
│          Middleware & Services Layer                                 │
│                                                                      │
│  ┌─ Services ─────────────────────────────────────────────────┐   │
│  │  • authService.ts (Login, Registration, Token Mgmt)      │   │
│  │  • projectService.ts (CRUD, Progress Calculation)        │   │
│  │  • messageService.ts (Send, Store, Retrieve)             │   │
│  │  • analyticsService.ts (Data Aggregation)                │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ REAL-TIME LAYER (WebSocket - ws://localhost:5000)                  │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Socket.io Server                                                    │
│                                                                      │
│  Events:                                                             │
│  • user:online        → Broadcast user is online                   │
│  • user:offline       → Broadcast user is offline                  │
│  • message:send       → Send message to recipient                  │
│  • message:receive    → Receive message                            │
│  • message:read       → Mark message as read                       │
│  • user:typing        → Show typing indicator                      │
│  • notification:*     → Send notifications                        │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ DATA LAYER                                                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MongoDB Atlas (Cloud Database)                                      │
│  mongodb+srv://user:password@cluster.mongodb.net/vismatrix          │
│                                                                      │
│  Collections:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ users                    projects                           │   │
│  │ ├─ _id (ObjectId)        ├─ _id (ObjectId)                │   │
│  │ ├─ email (unique)        ├─ userId (ref User)            │   │
│  │ ├─ username (unique)     ├─ name                         │   │
│  │ ├─ password (hashed)     ├─ description                  │   │
│  │ ├─ displayName           ├─ status                       │   │
│  │ ├─ avatar                ├─ visibility (private/public)  │   │
│  │ ├─ bio                   ├─ startDate                    │   │
│  │ ├─ followers[]           ├─ endDate                      │   │
│  │ ├─ following[]           └─ timestamps                   │   │
│  │ └─ isPublic                                               │   │
│  │                                                            │   │
│  │ tasks                    messages                         │   │
│  │ ├─ _id (ObjectId)        ├─ _id (ObjectId)               │   │
│  │ ├─ projectId (ref)       ├─ senderId (ref User)          │   │
│  │ ├─ title                 ├─ recipientId (ref User)       │   │
│  │ ├─ description           ├─ content                      │   │
│  │ ├─ status                ├─ isRead                       │   │
│  │ ├─ priority              ├─ readAt                       │   │
│  │ ├─ dueDate               └─ timestamps                   │   │
│  │ ├─ subtasks[]                                             │   │
│  │ └─ timestamps            progressLogs                     │   │
│  │                          ├─ _id (ObjectId)                │   │
│  │                          ├─ projectId (ref)              │   │
│  │                          ├─ completedTasks               │   │
│  │                          ├─ totalTasks                   │   │
│  │                          ├─ percentage                   │   │
│  │                          └─ recordedAt                   │   │
│  │                                                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  Indexes:                                                            │
│  • users.email (unique)      • tasks.projectId                      │
│  • users.username (unique)   • messages.senderId + recipientId     │
│  • projects.userId           • messages.createdAt (-1)             │
│  • projects.createdAt (-1)   • progressLogs.projectId              │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### Authentication Flow

```
User Registration:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│ 1. Frontend /register.vue                                      │
│    User fills: email, username, password, displayName          │
│    → POST /api/auth/register                                   │
│                                                                 │
│ 2. Backend authController.register()                           │
│    ├─ Validate inputs                                          │
│    ├─ Check if user exists                                     │
│    ├─ Hash password with bcryptjs                             │
│    ├─ Create User document in MongoDB                          │
│    └─ Generate JWT tokens                                      │
│                                                                 │
│ 3. Return response:                                            │
│    {                                                           │
│      accessToken: "jwt.token",     (Bearer token)             │
│      refreshToken: "jwt.token",    (HttpOnly cookie)          │
│      user: { id, email, username, displayName }               │
│    }                                                           │
│                                                                 │
│ 4. Frontend stores:                                            │
│    ├─ localStorage.setItem('accessToken', token)              │
│    └─ Auto-set in cookies by browser (HttpOnly)               │
│                                                                 │
│ 5. Redirect to /dashboard                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Protected Route Request:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│ 1. Frontend sends API request                                  │
│    GET /api/projects                                           │
│    Headers: {                                                  │
│      Authorization: "Bearer <accessToken>"                    │
│    }                                                           │
│                                                                 │
│ 2. Backend authMiddleware.ts                                   │
│    ├─ Extract token from Authorization header                 │
│    ├─ Verify JWT signature                                    │
│    ├─ Check token expiration                                  │
│    └─ Attach user to req.user                                 │
│                                                                 │
│ 3. If valid: Continue to route handler                        │
│    If expired: Return 401 → Trigger refresh                   │
│                                                                 │
│ 4. API Interceptor (frontend) handles 401:                    │
│    ├─ POST /api/auth/refresh                                  │
│    ├─ Server returns new accessToken                          │
│    ├─ Retry original request with new token                   │
│    └─ Update localStorage                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Project Creation Flow

```
Create Project:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│ 1. Frontend pages/projects/new.vue                             │
│    User fills: name, description, visibility                  │
│    → POST /api/projects                                        │
│    {                                                           │
│      name: "My Awesome Project",                              │
│      description: "Building something great",                 │
│      visibility: "public"                                      │
│    }                                                           │
│                                                                 │
│ 2. Backend projectController.createProject()                  │
│    ├─ Validate inputs                                         │
│    ├─ Create Project document                                 │
│    │  {                                                        │
│    │    userId: req.user.userId,                              │
│    │    name, description, visibility,                        │
│    │    status: "active",                                     │
│    │    startDate: new Date()                                │
│    │  }                                                        │
│    └─ Save to MongoDB                                         │
│                                                                 │
│ 3. Pinia projectStore (frontend)                              │
│    ├─ Add to projects array                                   │
│    └─ Update UI                                               │
│                                                                 │
│ 4. Response: { project: { _id, name, ... } }                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Add Task to Project:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│ 1. Frontend projects/[id].vue (Kanban board)                   │
│    User creates task: title, priority, dueDate               │
│    → POST /api/projects/:projectId/tasks                      │
│                                                                 │
│ 2. Backend taskController.createTask()                        │
│    ├─ Create Task document                                    │
│    ├─ status: "todo" (default)                               │
│    └─ Save to MongoDB                                         │
│                                                                 │
│ 3. Calculate Project Progress                                 │
│    ├─ Count completed tasks                                   │
│    ├─ Total tasks                                             │
│    └─ Percentage = (completed/total) * 100                   │
│                                                                 │
│ 4. Create ProgressLog entry                                   │
│    {                                                           │
│      projectId, completedTasks, totalTasks,                   │
│      completionPercentage, recordedAt: now                    │
│    }                                                           │
│                                                                 │
│ 5. Return task to frontend                                    │
│                                                                 │
│ 6. Update Pinia store & UI                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Real-Time Messaging Flow

```
Send Message:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│ 1. Frontend pages/social/messages.vue                          │
│    User types message, hits Enter                             │
│    → REST: POST /api/messages/:userId                         │
│    → WEBSOCKET: socket.emit('message:send', data)             │
│                                                                 │
│ 2. Backend REST Handler (Message persists)                    │
│    ├─ POST /api/messages/:recipientId                        │
│    ├─ Create Message document                                 │
│    │  {                                                        │
│    │    senderId: req.user.userId,                            │
│    │    recipientId,                                          │
│    │    content,                                              │
│    │    isRead: false,                                        │
│    │    createdAt: now                                       │
│    │  }                                                        │
│    └─ Save to MongoDB                                         │
│                                                                 │
│ 3. Backend WebSocket Handler (Real-time delivery)             │
│    ├─ Listen: socket.on('message:send')                       │
│    ├─ Emit to recipient: io.to('user:${recipientId}')       │
│    └─ Event: 'message:receive'                               │
│                                                                 │
│ 4. Recipient Frontend (Real-time update)                      │
│    ├─ Listen: socket.on('message:receive')                    │
│    ├─ Add to message list                                     │
│    ├─ Update Pinia store                                      │
│    └─ Scroll to newest message                                │
│                                                                 │
│ 5. Message Read Receipt                                        │
│    ├─ Recipient opens conversation                            │
│    ├─ Emit: socket.emit('message:read', messageId)            │
│    ├─ Backend updates isRead = true, readAt = now            │
│    ├─ Emit back: 'message:read' event                         │
│    └─ Sender sees ✓✓ (read status)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### User Discovery Flow

```
Discover Users:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│ 1. Frontend pages/social/discover.vue                          │
│    User navigates to Discover                                 │
│    → GET /api/users/discover                                  │
│                                                                 │
│ 2. Backend userController.discoverUsers()                     │
│    ├─ Query users.find()                                      │
│    │  where: {                                                 │
│    │    isPublic: true,                                        │
│    │    _id: { $ne: currentUser }  // Exclude self            │
│    │  }                                                        │
│    ├─ Sort by followers count (trending)                      │
│    ├─ Limit 20 per page                                       │
│    └─ Return with public projects count                       │
│                                                                 │
│ 3. Frontend renders UserCard for each user                    │
│    ├─ Display: avatar, displayName, bio, followers           │
│    ├─ Show "Follow" button                                    │
│    └─ Click to view full profile                              │
│                                                                 │
│ 4. Follow User                                                 │
│    ├─ POST /api/users/:userId/follow                         │
│    ├─ Backend adds userId to follower's following[]          │
│    ├─ Backend adds follower to user's followers[]            │
│    └─ Return updated following status                         │
│                                                                 │
│ 5. View User Profile                                           │
│    ├─ GET /api/users/:username                                │
│    ├─ GET /api/users/:username/projects                       │
│    ├─ Frontend displays public projects                        │
│    └─ Can view progress on public projects                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Hierarchy

```
App (Root)
├── Layouts/default.vue
│   ├── Header
│   │   ├── Logo
│   │   ├── NavLinks
│   │   ├── UserMenu (Dropdown)
│   │   └── ThemeToggle
│   ├── Sidebar
│   │   ├── NavItems
│   │   └── Collapse Toggle
│   └── <NuxtPage /> (Router outlet)
│
└── Pages
    ├── /index
    │   └── Redirect to /dashboard
    │
    ├── /dashboard
    │   ├── StatsSummary (3 cards)
    │   ├── ProjectGrid
    │   │   └── ProjectCard (multiple)
    │   ├── ActivityFeed (Social)
    │   └── MessagePreview (Recent conversations)
    │
    ├── /projects/index
    │   ├── ProjectFilter/Search
    │   └── ProjectList/Grid
    │       └── ProjectCard
    │
    ├── /projects/[id]
    │   ├── ProjectHeader
    │   ├── ProgressChart
    │   ├── KanbanBoard
    │   │   ├── TaskColumn (todo)
    │   │   ├── TaskColumn (in-progress)
    │   │   └── TaskColumn (completed)
    │   │       └── TaskCard
    │   │           └── SubtaskList
    │   └── TaskModal (Create/Edit)
    │
    ├── /social/discover
    │   ├── SearchBar
    │   └── UserGrid
    │       └── UserCard (multiple)
    │           ├── Avatar
    │           ├── Stats
    │           └── FollowButton
    │
    ├── /social/[username]
    │   ├── UserProfile
    │   │   ├── CoverImage
    │   │   ├── Avatar
    │   │   ├── Info (name, bio, followers)
    │   │   └── FollowButton
    │   ├── TabNavigation
    │   │   ├── Projects Tab
    │   │   └── Activity Tab
    │   └── ProjectGrid (Public projects only)
    │
    ├── /social/messages
    │   ├── ConversationList (Left sidebar)
    │   │   └── ConversationItem (multiple)
    │   ├── ChatWindow (Main area)
    │   │   ├── ChatHeader
    │   │   ├── MessageList
    │   │   │   └── MessageBubble (multiple)
    │   │   └── MessageInput
    │   └── TypingIndicator
    │
    └── /auth/*
        ├── /login
        │   └── LoginForm
        ├── /register
        │   └── RegisterForm
        └── /logout
            └── LogoutConfirmation
```

---

## Database Schema Relationships

```
┌─────────────┐
│    User     │
├─────────────┤
│ _id         │◄────────────────┐
│ email       │                 │
│ username    │                 │
│ password    │                 │
│ displayName │                 │
│ avatar      │                 │
│ bio         │                 │
│ followers[] │◄──────┐         │
│ following[] │◄──┐   │         │
│ isPublic    │   │   │         │
│ createdAt   │   │   │         │
│ updatedAt   │   │   │         │
└─────────────┘   │   │         │
       ▲          │   │         │
       │(One)     │   │(Many)   │
       │          │   │         │
       └──────────┼───┴─────┐   │
                  │         │   │
       ┌──────────┘         │   │
       │                    │   │
┌──────┴────────┐      ┌────┴───────┐
│   Project     │      │   Message  │
├───────────────┤      ├────────────┤
│ _id           │      │ _id        │
│ userId────────┼─────►│ senderId   │
│ name          │      │ recipientId│
│ description   │      │ content    │
│ status        │      │ isRead     │
│ visibility    │      │ readAt     │
│ startDate     │      │ createdAt  │
│ endDate       │      │ updatedAt  │
│ createdAt     │      └────────────┘
│ updatedAt     │
└──────┬────────┘
       │(One)
       │
       │(Many)
       │
┌──────▼────────┐
│    Task       │
├───────────────┤
│ _id           │
│ projectId─────┼──┐(ref)
│ title         │  │
│ description   │  │
│ status        │  │
│ priority      │  │
│ dueDate       │  │
│ subtasks[]    │  │
│ completedAt   │  │
│ createdAt     │  │
│ updatedAt     │  │
└───────────────┘  │
                   │
         ┌─────────┘
         │
┌────────▼──────┐
│ ProgressLog   │
├───────────────┤
│ _id           │
│ projectId─────┼──┐(ref)
│ completedTasks│  │
│ totalTasks    │  │
│ percentage    │  │
│ recordedAt    │  │
└───────────────┘  │
                   │
         ┌─────────┘
```

---

## Deployment Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION DEPLOYMENT                       │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────┐
│      Domain (vismatrix.space)   │
│                                 │
│  www.vismatrix.space            │
│  api.vismatrix.space            │
└────────────┬────────────────────┘
             │
             │ HTTPS
             │
    ┌────────┴─────────┐
    │                  │
    ▼                  ▼
┌──────────────┐   ┌──────────────┐
│   Vercel     │   │ Railway.app  │
│  (Frontend)  │   │   (Backend)  │
│              │   │              │
│ Nuxt Build   │   │ Express.js   │
│ Auto-deploy  │   │ Socket.io    │
│ CDN          │   │              │
└──────────────┘   └──────┬───────┘
                          │
                          │ Connection
                          │
                   ┌──────▼───────┐
                   │  MongoDB     │
                   │   Atlas      │
                   │              │
                   │ Cluster M2   │
                   │ Auto-backup  │
                   │ Monitoring   │
                   └──────────────┘
```

---

## Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────┐
│                   TECHNOLOGY STACK                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  FRONTEND                     BACKEND                      │
│  ├─ Nuxt 4                    ├─ Node.js 18+              │
│  ├─ Vue 3                     ├─ Express.js               │
│  ├─ TypeScript                ├─ TypeScript               │
│  ├─ Tailwind CSS              ├─ Mongoose                 │
│  ├─ Nuxt UI v4                ├─ JWT (jsonwebtoken)       │
│  ├─ Pinia (State)             ├─ bcryptjs                 │
│  ├─ Chart.js                  ├─ Socket.io                │
│  ├─ Axios (HTTP)              ├─ CORS                     │
│  ├─ Socket.io Client          └─ dotenv                   │
│  └─ DayJS                                                   │
│                                                             │
│  DATABASE                     INFRASTRUCTURE              │
│  ├─ MongoDB Atlas              ├─ Vercel (Frontend)       │
│  ├─ Collections 5              ├─ Railway (Backend)       │
│  ├─ Indexes 5+                 ├─ MongoDB Atlas           │
│  ├─ Aggregation Pipeline       ├─ GitHub (Version Control)│
│  └─ Real-time Updates          └─ SSL/TLS (HTTPS)         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

These diagrams provide a complete visual reference for the architecture, data flows, component hierarchy, and deployment strategy of VisMatrix.space!
