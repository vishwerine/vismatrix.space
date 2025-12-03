# VisMatrix.space - Complete Implementation & Deployment Guide

## Table of Contents
1. Project Setup
2. Development Workflow
3. Database Setup
4. API Implementation Roadmap
5. Frontend Implementation Roadmap
6. Real-Time Features
7. Deployment Strategy
8. Performance Optimization
9. Security Checklist

---

## 1. Project Setup

### Create Both Projects

```bash
# Frontend
npm create nuxt@latest vismatrix-frontend -- --typescript
cd vismatrix-frontend
npm install @nuxt/ui pinia axios @vueuse/core chart.js vue-chartjs dayjs socket.io-client

# Backend (in separate directory)
mkdir vismatrix-backend
cd vismatrix-backend
npm init -y
npm install express mongoose bcryptjs jsonwebtoken cors dotenv socket.io
npm install --save-dev typescript @types/express @types/node ts-node nodemon
npx tsc --init
```

### Directory Structure

```
vismatrix/
├── vismatrix-frontend/        # Nuxt 4 app
│   ├── package.json
│   ├── nuxt.config.ts
│   ├── tsconfig.json
│   └── app/
│       ├── pages/
│       ├── components/
│       ├── stores/
│       ├── utils/
│       └── types/
│
├── vismatrix-backend/         # Express + MongoDB
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── index.ts
│   │   ├── server.ts
│   │   ├── models/
│   │   ├── controllers/
│   │   ├── routes/
│   │   ├── middleware/
│   │   ├── services/
│   │   └── websocket/
│   └── .env
│
└── README.md
```

---

## 2. Development Workflow

### Start Both Servers

**Terminal 1 - Backend**
```bash
cd vismatrix-backend
npm run dev
# Runs on http://localhost:5000
```

**Terminal 2 - Frontend**
```bash
cd vismatrix-frontend
npm run dev
# Runs on http://localhost:3000
```

### Development with Live Reload

Both Nuxt and Express support hot reload:

**Nuxt**: Changes to `.vue` files auto-reload
**Express**: Use `nodemon` to watch TypeScript changes

Create `nodemon.json` in backend:
```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["node_modules"],
  "exec": "ts-node src/index.ts",
  "env": {
    "NODE_ENV": "development"
  }
}
```

Add to backend `package.json`:
```json
{
  "scripts": {
    "dev": "nodemon",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

---

## 3. Database Setup

### MongoDB Atlas (Cloud)

1. **Create Account**: https://www.mongodb.com/cloud/atlas
2. **Create Cluster**:
   - Choose Free tier (M0)
   - Select nearest region
   - Configure security
3. **Get Connection String**:
   - Cluster > Connect > Connection String
   - Format: `mongodb+srv://username:password@cluster.mongodb.net/vismatrix`
4. **Update `.env`**:
   ```
   MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/vismatrix
   ```

### Alternative: Local MongoDB

```bash
# Install MongoDB locally
# macOS: brew install mongodb-community
# Linux: Follow MongoDB docs

# Start local server
mongod

# Connection string for local
MONGODB_URI=mongodb://localhost:27017/vismatrix
```

### Create Collections with Indexes

```typescript
// In backend initialization
import { User, Project, Task, Message, ProgressLog } from './models';

const createIndexes = async () => {
  // User indexes
  await User.collection.createIndex({ email: 1 }, { unique: true });
  await User.collection.createIndex({ username: 1 }, { unique: true });

  // Project indexes
  await Project.collection.createIndex({ userId: 1, createdAt: -1 });

  // Task indexes
  await Task.collection.createIndex({ projectId: 1 });

  // Message indexes (critical for performance)
  await Message.collection.createIndex({ senderId: 1, recipientId: 1 });
  await Message.collection.createIndex({ createdAt: -1 });
  await Message.collection.createIndex({ recipientId: 1, isRead: 1 });

  console.log('Database indexes created');
};
```

---

## 4. API Implementation Roadmap

### Phase 1: Authentication (Priority: CRITICAL)

**Endpoints:**
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/refresh-token
GET  /api/auth/verify
```

**Implementation:**
1. Create User model with password hashing
2. Create JWT utilities (sign/verify)
3. Create auth middleware
4. Implement auth controller
5. Test with Postman/Insomnia

### Phase 2: Projects CRUD (Priority: HIGH)

**Endpoints:**
```
GET    /api/projects                  # List user's projects
POST   /api/projects                  # Create project
GET    /api/projects/:id              # Get project with tasks
PUT    /api/projects/:id              # Update project
DELETE /api/projects/:id              # Delete project
GET    /api/projects/:id/progress     # Progress timeline
```

**Data Structure:**
```typescript
{
  _id: ObjectId,
  userId: ObjectId,
  name: string,
  description: string,
  status: 'active' | 'completed' | 'archived',
  visibility: 'private' | 'public',
  startDate: Date,
  endDate: Date,
  createdAt: Date,
  updatedAt: Date
}
```

### Phase 3: Tasks & Subtasks (Priority: HIGH)

**Endpoints:**
```
GET    /api/projects/:projectId/tasks
POST   /api/projects/:projectId/tasks
PUT    /api/tasks/:taskId
DELETE /api/tasks/:taskId
PUT    /api/tasks/:taskId/status
POST   /api/tasks/:taskId/subtasks
PUT    /api/tasks/:taskId/subtasks/:subtaskId
```

**Features:**
- Drag-and-drop status updates (frontend only)
- Subtask management
- Progress calculation
- Timestamps for completion tracking

### Phase 4: Users & Social (Priority: MEDIUM)

**Endpoints:**
```
GET    /api/users/discover                    # Discover users
GET    /api/users/:username                   # User profile
GET    /api/users/:username/projects          # Public projects
POST   /api/users/:userId/follow              # Follow user
DELETE /api/users/:userId/follow              # Unfollow
GET    /api/users/followers                   # Get followers
GET    /api/users/following                   # Get following
```

**Features:**
- User profiles with public/private
- Follow/unfollow relationships
- Public project visibility
- User discovery algorithm (trending users, similar interests)

### Phase 5: Messaging (Priority: MEDIUM)

**REST Endpoints:**
```
GET    /api/messages                          # List conversations
GET    /api/messages/:userId                  # Get conversation
POST   /api/messages/:userId                  # Send message
GET    /api/messages/:conversationId/history  # Message history
PUT    /api/messages/:messageId/read          # Mark as read
```

**WebSocket Events:**
```
user:online
user:offline
message:send
message:receive
message:read
user:typing
user:notification
```

### Phase 6: Analytics (Priority: MEDIUM)

**Endpoints:**
```
GET /api/analytics/progress              # Progress over time
GET /api/analytics/summary               # Current summary
GET /api/analytics/completion-rate       # Metrics
GET /api/analytics/project/:id/timeline  # Project timeline
```

**Data Tracked:**
- Tasks completed per day
- Project progress percentage
- Burndown chart data
- User activity metrics

---

## 5. Frontend Implementation Roadmap

### Phase 1: Auth Pages (Priority: CRITICAL)

**Pages:**
- `/auth/login.vue` - Login form
- `/auth/register.vue` - Registration form
- `/auth/logout.vue` - Logout confirmation
- Middleware to protect routes

**Features:**
- Form validation
- Error messages
- Loading states
- Auto-redirect on login
- Token storage

### Phase 2: Dashboard (Priority: HIGH)

**Pages:**
- `/dashboard.vue` - Main dashboard
- `/projects/index.vue` - Projects list
- `/projects/[id].vue` - Project detail

**Components:**
- ProjectCard
- StatsSummary
- ProgressChart
- TaskColumn (for kanban)

**Features:**
- Project list with filters
- Create/edit/delete projects
- Task management
- Progress visualization

### Phase 3: Task Management (Priority: HIGH)

**Features:**
- Kanban board (todo, in-progress, completed)
- Drag-and-drop (Vue Beautiful DnD)
- Subtask management
- Task priority/due date
- Quick edit inline

### Phase 4: Social Features (Priority: MEDIUM)

**Pages:**
- `/social/discover.vue` - Find users
- `/social/[username].vue` - User profiles
- `/social/messages.vue` - Messaging interface

**Features:**
- User search
- Follow/unfollow
- Public profile viewing
- Activity feed

### Phase 5: Messaging (Priority: MEDIUM)

**Features:**
- Real-time messaging
- Conversation list
- Message history
- Online/offline status
- Typing indicators
- Unread count

### Phase 6: Analytics & Visualization (Priority: LOW)

**Charts:**
- Progress timeline (line chart)
- Task distribution (pie chart)
- Burndown chart
- Productivity metrics

---

## 6. Real-Time Features (WebSocket)

### Socket.io Setup

**Backend (`src/websocket/socket-handler.ts`):**
```typescript
import { Server } from 'socket.io';
import { Message } from '../models/Message';

export const setupSocket = (io: Server) => {
  io.on('connection', (socket) => {
    console.log('User connected:', socket.id);

    // User goes online
    socket.on('user:online', (userId: string) => {
      socket.join(`user:${userId}`);
      socket.join('online_users');
      io.to('online_users').emit('user:online', { userId });
    });

    // Message sending
    socket.on('message:send', async (data: {
      senderId: string;
      recipientId: string;
      content: string;
    }) => {
      const message = new Message(data);
      await message.save();
      
      io.to(`user:${data.recipientId}`).emit('message:receive', {
        _id: message._id,
        ...data,
        timestamp: new Date(),
      });
    });

    // Read receipt
    socket.on('message:read', async (messageId: string) => {
      await Message.findByIdAndUpdate(messageId, {
        isRead: true,
        readAt: new Date(),
      });
    });

    // Typing indicator
    socket.on('user:typing', (data: { recipientId: string; username: string }) => {
      io.to(`user:${data.recipientId}`).emit('user:typing', data);
    });

    socket.on('disconnect', () => {
      socket.leave('online_users');
      console.log('User disconnected:', socket.id);
    });
  });
};
```

**Frontend (`composables/useMessages.ts`):**
```typescript
import { ref, onMounted } from 'vue';
import { io } from 'socket.io-client';

export const useMessages = () => {
  const socket = ref(null);
  const onlineUsers = ref(new Set());
  const typingUsers = ref(new Set());

  const initSocket = () => {
    socket.value = io(useRuntimeConfig().public.wsUrl);

    socket.value.on('connect', () => {
      const auth = useAuthStore();
      socket.value.emit('user:online', auth.user.id);
    });

    socket.value.on('message:receive', (message: any) => {
      // Handle incoming message
      const messageStore = useMessageStore();
      messageStore.addMessage(message);
    });

    socket.value.on('user:online', (data: any) => {
      onlineUsers.value.add(data.userId);
    });

    socket.value.on('user:typing', (data: any) => {
      typingUsers.value.add(data.userId);
      setTimeout(() => typingUsers.value.delete(data.userId), 3000);
    });
  };

  const sendMessage = (recipientId: string, content: string) => {
    const auth = useAuthStore();
    socket.value.emit('message:send', {
      senderId: auth.user.id,
      recipientId,
      content,
    });
  };

  const markAsRead = (messageId: string) => {
    socket.value.emit('message:read', messageId);
  };

  const emitTyping = (recipientId: string) => {
    const auth = useAuthStore();
    socket.value.emit('user:typing', {
      recipientId,
      username: auth.user.username,
    });
  };

  onMounted(initSocket);

  return {
    onlineUsers,
    typingUsers,
    sendMessage,
    markAsRead,
    emitTyping,
  };
};
```

---

## 7. Deployment Strategy

### Backend Deployment (Express + MongoDB)

**Option 1: Railway.app (Recommended for MVP)**
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Deploy
railway up
```

**Option 2: Render.com**
```
1. Connect GitHub repo
2. Create Web Service
3. Set environment variables
4. Deploy automatically
```

**Option 3: Heroku (Free tier deprecated, use paid)**
```bash
npm install -g heroku
heroku login
heroku create vismatrix-api
git push heroku main
```

### Frontend Deployment (Nuxt)

**Vercel (Recommended)**
```bash
npm install -g vercel
vercel

# Auto-deploys from GitHub pushes
```

**Netlify**
```
1. Connect GitHub repo
2. Build command: npm run build
3. Publish directory: .output/public
```

### MongoDB Atlas Production Setup

1. Create production cluster (M2 or higher)
2. Configure IP whitelist
3. Create backup snapshots
4. Enable authentication
5. Use strong passwords
6. Enable encryption at rest

### Environment Variables - Production

**Backend `.env.production`:**
```
NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/vismatrix
JWT_SECRET=<very-long-random-string>
JWT_REFRESH_SECRET=<another-long-random-string>
FRONTEND_URL=https://vismatrix.space
```

**Frontend `.env.production`:**
```
NUXT_PUBLIC_API_URL=https://api.vismatrix.space/api
NUXT_PUBLIC_WS_URL=wss://api.vismatrix.space
```

### Domain & SSL

1. Register domain (namecheap.com, godaddy.com, etc.)
2. Point to hosting service's nameservers
3. SSL certificate (automatic with Vercel/Railway)
4. Set up CNAME records

---

## 8. Performance Optimization

### Backend Optimization

```typescript
// 1. Database query optimization
const project = await Project.findById(id)
  .populate('tasks')
  .lean(); // Returns plain JS objects

// 2. Pagination
const limit = 20;
const skip = (page - 1) * limit;
const projects = await Project.find({ userId })
  .limit(limit)
  .skip(skip)
  .sort({ createdAt: -1 });

// 3. Caching
import redis from 'redis';
const client = redis.createClient();

app.get('/projects', async (req, res) => {
  const cached = await client.get('projects:' + userId);
  if (cached) return res.json(JSON.parse(cached));
  
  const projects = await Project.find({ userId });
  await client.setEx('projects:' + userId, 300, JSON.stringify(projects));
  res.json(projects);
});
```

### Frontend Optimization

```typescript
// 1. Code splitting - automatic with Nuxt
// 2. Image optimization - use Nuxt Image
<NuxtImg src="/avatar.jpg" width="200" height="200" />

// 3. Virtual scrolling for large lists
<RecycleScroller :items="tasks" key-field="_id">
  <TaskItem :task="task" />
</RecycleScroller>

// 4. Lazy loading components
const AnalyticsChart = defineAsyncComponent(() =>
  import('~/components/Dashboard/AnalyticsChart.vue')
);
```

### Monitoring

```bash
# Backend health check
GET /api/health

# Frontend performance monitoring
npm install @sentry/vue
npm install web-vitals
```

---

## 9. Security Checklist

### Authentication & Authorization
- [ ] Passwords hashed with bcryptjs
- [ ] JWT tokens with expiration
- [ ] Refresh token rotation
- [ ] CORS properly configured
- [ ] Rate limiting on auth endpoints
- [ ] HTTPS only in production

### Data Protection
- [ ] MongoDB encryption at rest
- [ ] Secure HTTP-only cookies
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (Mongoose)
- [ ] XSS protection (Vue auto-escapes)
- [ ] CSRF tokens for state-changing operations

### API Security
- [ ] API rate limiting
- [ ] Request size limits
- [ ] Timeout configuration
- [ ] Error messages don't leak info
- [ ] NoSQL injection prevention
- [ ] Field access control

### Infrastructure
- [ ] Environment variables never committed
- [ ] .env files in .gitignore
- [ ] Secrets in production deployment
- [ ] Backups configured
- [ ] Monitoring & logging enabled
- [ ] DDoS protection (Cloudflare)

### Frontend Security
- [ ] Content Security Policy headers
- [ ] Token stored securely (not localStorage)
- [ ] HTTPS enforcement
- [ ] Dependency vulnerability scanning
- [ ] Build artifact verification

---

## Implementation Timeline

```
Week 1: Project Setup & Auth
- [ ] Set up both projects
- [ ] Create User model
- [ ] Implement JWT auth
- [ ] Create login/register pages

Week 2: Projects & Tasks
- [ ] Project CRUD endpoints
- [ ] Task model with subtasks
- [ ] Task management UI
- [ ] Kanban board

Week 3: Social Features
- [ ] User discovery
- [ ] Follow/unfollow
- [ ] User profiles
- [ ] Public projects

Week 4: Messaging & Analytics
- [ ] Real-time messaging with Socket.io
- [ ] Message history
- [ ] Progress charts
- [ ] Analytics endpoints

Week 5: Polish & Optimization
- [ ] Performance optimization
- [ ] Security hardening
- [ ] Testing
- [ ] Documentation

Week 6: Deployment
- [ ] Deploy backend
- [ ] Deploy frontend
- [ ] Domain setup
- [ ] Monitoring setup
```

---

## Getting Help

### Resources
- [Nuxt 4 Documentation](https://nuxt.com)
- [Express.js Guide](https://expressjs.com)
- [MongoDB Documentation](https://docs.mongodb.com)
- [Socket.io Guide](https://socket.io/docs)
- [Vue 3 Guide](https://vuejs.org)

### Community
- Stack Overflow (tag: nuxt, express, mongodb)
- Discord communities
- GitHub discussions
- Reddit r/vuejs, r/node

---

This guide provides everything needed to build, deploy, and scale VisMatrix.space! Start with the authentication system and progressively add features.
