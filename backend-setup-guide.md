# VisMatrix.space - Node.js Backend Setup Guide

## Quick Start

```bash
# Create project directory
mkdir vismatrix-backend
cd vismatrix-backend

# Initialize Node.js project
npm init -y

# Install core dependencies
npm install express mongoose bcryptjs jsonwebtoken cors dotenv axios
npm install --save-dev typescript @types/express @types/node ts-node nodemon

# Initialize TypeScript
npx tsc --init

# Create folder structure
mkdir -p src/{routes,controllers,middleware,models,services,utils,types}
```

## Project Structure

```
vismatrix-backend/
├── src/
│   ├── index.ts                      # Entry point
│   ├── server.ts                     # Express setup
│   │
│   ├── config/
│   │   ├── database.ts               # MongoDB connection
│   │   └── env.ts                    # Environment variables
│   │
│   ├── models/
│   │   ├── User.ts                   # User schema
│   │   ├── Project.ts                # Project schema
│   │   ├── Task.ts                   # Task/Subtask schema
│   │   ├── Message.ts                # Message schema
│   │   └── ProgressLog.ts            # Progress history
│   │
│   ├── routes/
│   │   ├── auth.routes.ts            # Auth endpoints
│   │   ├── projects.routes.ts        # Project CRUD
│   │   ├── tasks.routes.ts           # Task CRUD
│   │   ├── users.routes.ts           # User discovery
│   │   ├── messages.routes.ts        # Messaging
│   │   └── analytics.routes.ts       # Progress analytics
│   │
│   ├── controllers/
│   │   ├── authController.ts
│   │   ├── projectController.ts
│   │   ├── taskController.ts
│   │   ├── userController.ts
│   │   ├── messageController.ts
│   │   └── analyticsController.ts
│   │
│   ├── middleware/
│   │   ├── auth.ts                   # JWT verification
│   │   ├── errorHandler.ts           # Error handling
│   │   └── validation.ts             # Input validation
│   │
│   ├── services/
│   │   ├── authService.ts            # Auth logic
│   │   ├── projectService.ts         # Business logic
│   │   ├── messageService.ts         # Messaging logic
│   │   └── analyticsService.ts       # Analytics logic
│   │
│   ├── utils/
│   │   ├── jwt.ts                    # JWT utilities
│   │   ├── logger.ts                 # Logging
│   │   └── validators.ts             # Validation helpers
│   │
│   ├── types/
│   │   └── index.ts                  # TypeScript interfaces
│   │
│   └── websocket/
│       └── socket-handler.ts         # Socket.io setup
│
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

## Key Technologies

| Technology | Purpose |
|-----------|---------|
| `Express.js` | Web framework |
| `MongoDB` (Atlas) | Database |
| `Mongoose` | ODM/Schema validation |
| `JWT` | Stateless authentication |
| `bcryptjs` | Password hashing |
| `Socket.io` | Real-time messaging |
| `CORS` | Cross-origin requests |
| `TypeScript` | Type safety |

## Environment Variables (`.env.example`)

```env
# Server
PORT=5000
NODE_ENV=development

# Database
MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/vismatrix

# JWT
JWT_SECRET=your_super_secret_jwt_key_change_in_production
JWT_REFRESH_SECRET=your_super_secret_refresh_key
JWT_EXPIRY=24h
JWT_REFRESH_EXPIRY=7d

# CORS
FRONTEND_URL=http://localhost:3000

# Socket.io
SOCKET_PORT=5001
```

## Core API Endpoints

### Authentication
```
POST   /api/auth/register       # User registration
POST   /api/auth/login          # User login
POST   /api/auth/refresh        # Refresh token
POST   /api/auth/logout         # Logout
GET    /api/auth/verify         # Verify token
```

### Projects
```
GET    /api/projects            # List user's projects
POST   /api/projects            # Create project
GET    /api/projects/:id        # Get project details
PUT    /api/projects/:id        # Update project
DELETE /api/projects/:id        # Delete project
GET    /api/projects/:id/progress # Get progress timeline
```

### Tasks
```
GET    /api/projects/:id/tasks  # List tasks
POST   /api/projects/:id/tasks  # Create task
PUT    /api/tasks/:id           # Update task
DELETE /api/tasks/:id           # Delete task
PUT    /api/tasks/:id/status    # Update task status
POST   /api/tasks/:id/subtasks  # Create subtask
```

### Social/Users
```
GET    /api/users/discover      # Discover users
GET    /api/users/:username     # User profile
GET    /api/users/:username/projects  # Public projects
POST   /api/users/:username/follow    # Follow user
DELETE /api/users/:username/follow    # Unfollow
```

### Messaging
```
GET    /api/messages/:userId    # Get conversation
POST   /api/messages/:userId    # Send message
GET    /api/messages            # List conversations
DELETE /api/messages/:id        # Delete message
```

### Analytics
```
GET    /api/analytics/progress  # Progress over time
GET    /api/analytics/summary   # Projects summary
GET    /api/analytics/completion-rate # Completion metrics
```

## Database Schema Overview

### User
```typescript
{
  _id: ObjectId
  email: string (unique)
  username: string (unique)
  password: string (hashed)
  displayName: string
  avatar: string (URL)
  bio: string
  followers: ObjectId[]
  following: ObjectId[]
  isPublic: boolean
  createdAt: Date
  updatedAt: Date
}
```

### Project
```typescript
{
  _id: ObjectId
  userId: ObjectId
  name: string
  description: string
  status: 'active' | 'completed' | 'archived'
  startDate: Date
  endDate: Date
  visibility: 'private' | 'public'
  createdAt: Date
  updatedAt: Date
}
```

### Task
```typescript
{
  _id: ObjectId
  projectId: ObjectId
  title: string
  description: string
  status: 'todo' | 'in-progress' | 'completed'
  priority: 'low' | 'medium' | 'high'
  dueDate: Date
  subtasks: [{
    _id: ObjectId
    title: string
    status: boolean
    completedAt: Date
  }]
  completedAt: Date
  createdAt: Date
  updatedAt: Date
}
```

### Message
```typescript
{
  _id: ObjectId
  senderId: ObjectId
  recipientId: ObjectId
  content: string
  isRead: boolean
  readAt: Date
  createdAt: Date
  updatedAt: Date
}
```

### ProgressLog
```typescript
{
  _id: ObjectId
  projectId: ObjectId
  completedTasks: number
  totalTasks: number
  completionPercentage: number
  recordedAt: Date
}
```

## Authentication Flow

### JWT Strategy (Access + Refresh Tokens)

1. User registers/logs in
2. Server generates:
   - **Access Token** (short-lived, 24h)
   - **Refresh Token** (long-lived, 7d)
3. Frontend stores in HTTP-only cookie (refresh) and memory (access)
4. All API calls include: `Authorization: Bearer <access_token>`
5. On token expiry, use refresh token to get new access token
6. Refresh token stored in secure HTTP-only cookie

## Real-Time Messaging (WebSocket)

Using Socket.io for real-time features:

- Message delivery
- Online status
- Typing indicators
- Message read receipts
- Notification delivery

Connection namespace: `/messages`

Events:
```
- user:online / user:offline
- message:send / message:receive
- message:read
- user:typing
- user:notification
```

## Security Best Practices Implemented

✅ Password hashing with bcryptjs
✅ JWT token-based authentication
✅ CORS configuration
✅ Input validation
✅ Error handling middleware
✅ HTTP-only secure cookies
✅ Rate limiting (to be added)
✅ SQL injection prevention (MongoDB + Mongoose)
✅ XSS protection
✅ HTTPS required in production

## Deployment

### MongoDB Atlas Setup
1. Create cluster on mongodb.com
2. Generate connection string
3. Add IP whitelist
4. Set environment variable

### Deploy Backend
- Heroku / Railway / Render / AWS
- Use production MongoDB URI
- Set NODE_ENV=production
- Enable HTTPS

### Deploy Frontend
- Vercel / Netlify
- Point to production API URL
- Enable auto-deployment from GitHub

## Development Commands

```bash
# Start development server
npm run dev

# Build TypeScript
npm run build

# Start production
npm start

# Run tests (if configured)
npm run test
```

## Next Steps

1. Set up MongoDB Atlas cluster
2. Initialize Express server
3. Create authentication system
4. Build CRUD endpoints
5. Implement WebSocket messaging
6. Add progress tracking
7. Deploy both frontend and backend
