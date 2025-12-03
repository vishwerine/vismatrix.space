# VisMatrix.space - Quick Start Guide

## 🚀 5-Minute Quick Start

### Prerequisites
- Node.js 18+
- MongoDB Atlas account (free)
- Git

### Step 1: Create Frontend (Nuxt 4)

```bash
npm create nuxt@latest vismatrix-frontend -- --typescript
cd vismatrix-frontend

# Install dependencies
npm install @nuxt/ui pinia axios @vueuse/core chart.js vue-chartjs dayjs socket.io-client

# Start dev server
npm run dev
# Frontend runs on http://localhost:3000
```

### Step 2: Create Backend (Express)

```bash
# In another directory/terminal
mkdir vismatrix-backend && cd vismatrix-backend

# Initialize Node project
npm init -y

# Install dependencies
npm install express mongoose bcryptjs jsonwebtoken cors dotenv socket.io
npm install --save-dev typescript @types/express @types/node ts-node nodemon

# Initialize TypeScript
npx tsc --init

# Create source directory
mkdir src

# Start dev server
npm run dev
# Backend runs on http://localhost:5000
```

### Step 3: Set Up MongoDB

1. Go to https://www.mongodb.com/cloud/atlas
2. Sign up (free)
3. Create a free M0 cluster
4. Get connection string
5. Create `.env` file in backend:

```env
PORT=5000
NODE_ENV=development
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/vismatrix
JWT_SECRET=your_super_secret_key_here
JWT_REFRESH_SECRET=your_refresh_secret_here
FRONTEND_URL=http://localhost:3000
```

### Step 4: Create Basic Backend Server

**backend/src/index.ts:**
```typescript
import express from 'express';
import mongoose from 'mongoose';
import cors from 'cors';
import dotenv from 'dotenv';

dotenv.config();

const app = express();

// Middleware
app.use(express.json());
app.use(cors({ origin: process.env.FRONTEND_URL }));

// Connect MongoDB
mongoose.connect(process.env.MONGODB_URI!)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB error:', err));

// Test route
app.get('/api/health', (req, res) => {
  res.json({ status: 'API is running' });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Step 5: Create Basic Frontend Dashboard

**frontend/app.vue:**
```vue
<template>
  <div class="min-h-screen bg-gray-50 dark:bg-gray-900">
    <NuxtPage />
  </div>
</template>
```

**frontend/pages/dashboard.vue:**
```vue
<template>
  <div class="p-8 max-w-7xl mx-auto">
    <h1 class="text-4xl font-bold mb-8">VisMatrix Dashboard</h1>
    
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <!-- Project Card Example -->
      <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-6">
        <h3 class="text-lg font-semibold mb-2">Total Projects</h3>
        <p class="text-3xl font-bold text-primary">0</p>
        <p class="text-sm text-gray-500 mt-2">Ready to create your first project</p>
      </div>

      <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-6">
        <h3 class="text-lg font-semibold mb-2">Active Tasks</h3>
        <p class="text-3xl font-bold text-orange-500">0</p>
        <p class="text-sm text-gray-500 mt-2">Keep track of progress</p>
      </div>

      <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-6">
        <h3 class="text-lg font-semibold mb-2">Completed</h3>
        <p class="text-3xl font-bold text-green-500">0%</p>
        <p class="text-sm text-gray-500 mt-2">Overall completion rate</p>
      </div>
    </div>

    <div class="mt-8 flex gap-4">
      <UButton color="primary" size="lg">Create Project</UButton>
      <UButton color="gray" variant="outline" size="lg" to="/social">Discover Users</UButton>
    </div>
  </div>
</template>

<script setup lang="ts">
// Dashboard logic here
</script>
```

---

## 📁 Project Structure Template

### Frontend (`/frontend`)
```
app/
├── pages/
│   ├── index.vue                    # Redirect to dashboard
│   ├── dashboard.vue                # Main dashboard
│   ├── auth/
│   │   ├── login.vue
│   │   └── register.vue
│   ├── projects/
│   │   ├── index.vue                # Projects list
│   │   ├── [id].vue                 # Project detail
│   │   └── new.vue                  # Create project
│   └── social/
│       ├── discover.vue             # Find users
│       ├── messages.vue             # Chat
│       └── [username].vue           # User profile
│
├── components/
│   ├── Dashboard/
│   │   ├── ProjectCard.vue
│   │   ├── TaskList.vue
│   │   ├── ProgressChart.vue
│   │   └── StatCard.vue
│   ├── Social/
│   │   ├── UserCard.vue
│   │   └── MessageThread.vue
│   └── Common/
│       ├── Header.vue
│       └── Sidebar.vue
│
├── stores/
│   ├── auth.ts
│   ├── projects.ts
│   └── messages.ts
│
└── composables/
    ├── useAuth.ts
    ├── useProjects.ts
    └── useMessages.ts
```

### Backend (`/backend`)
```
src/
├── index.ts                        # Entry point
├── server.ts                       # Express setup
│
├── models/
│   ├── User.ts
│   ├── Project.ts
│   ├── Task.ts
│   └── Message.ts
│
├── controllers/
│   ├── authController.ts
│   ├── projectController.ts
│   ├── taskController.ts
│   └── userController.ts
│
├── routes/
│   ├── auth.routes.ts
│   ├── projects.routes.ts
│   ├── tasks.routes.ts
│   └── users.routes.ts
│
├── middleware/
│   ├── auth.ts
│   └── errorHandler.ts
│
└── utils/
    ├── jwt.ts
    └── validators.ts
```

---

## 🔐 Core Features Checklist

### Authentication
- [ ] User registration
- [ ] User login
- [ ] JWT token generation
- [ ] Protected routes
- [ ] Logout functionality

### Projects
- [ ] Create projects
- [ ] List projects
- [ ] View project details
- [ ] Update project
- [ ] Delete project
- [ ] Project visibility (public/private)

### Tasks
- [ ] Create tasks
- [ ] Add subtasks
- [ ] Update task status
- [ ] Delete tasks
- [ ] Task priority levels
- [ ] Due dates

### Progress Visualization
- [ ] Task completion chart
- [ ] Progress percentage
- [ ] Timeline view
- [ ] Completion metrics

### Social Features
- [ ] User discovery
- [ ] User profiles
- [ ] Follow/unfollow
- [ ] Public projects view

### Messaging
- [ ] Send messages
- [ ] Message history
- [ ] Real-time delivery
- [ ] Online status
- [ ] Conversation list

---

## 🚀 Implementation Order

### Week 1: Foundation
1. Set up both projects
2. Connect MongoDB
3. Create User model
4. Implement auth system
5. Create login/register pages

### Week 2: Core Features
1. Project CRUD API
2. Task model and API
3. Dashboard UI
4. Create/edit project forms

### Week 3: Task Management
1. Kanban board UI
2. Drag-and-drop
3. Status updates
4. Subtask management

### Week 4: Social & Messaging
1. User discovery
2. Follow system
3. WebSocket setup
4. Real-time messaging

### Week 5: Analytics
1. Progress charts
2. Analytics endpoints
3. Visualization components

### Week 6: Polish & Deploy
1. Performance optimization
2. Error handling
3. Security review
4. Deploy to production

---

## 🔗 Key API Endpoints

```
Authentication
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout

Projects
GET    /api/projects
POST   /api/projects
GET    /api/projects/:id
PUT    /api/projects/:id
DELETE /api/projects/:id

Tasks
GET    /api/projects/:projectId/tasks
POST   /api/projects/:projectId/tasks
PUT    /api/tasks/:id
DELETE /api/tasks/:id

Users
GET    /api/users/discover
GET    /api/users/:username
POST   /api/users/:id/follow

Messages
GET    /api/messages
POST   /api/messages/:userId
```

---

## 🎨 Frontend Layout Components

### Main Layout Structure
```
┌─────────────────────────────────────────┐
│            Header / Navigation          │
├──────────────┬──────────────────────────┤
│              │                          │
│   Sidebar    │      Main Content        │
│              │      (Pages)             │
│              │                          │
└──────────────┴──────────────────────────┘
```

### Page Layouts
- **Dashboard**: Grid of cards + recent activity
- **Projects**: List/grid view with filters
- **Project Detail**: Kanban board + sidebar
- **Social**: User cards in grid
- **Messages**: Conversation list + chat window

---

## 📦 Essential npm Packages

### Frontend
```json
{
  "@nuxt/ui": "^4.0.0",           // UI components
  "pinia": "^2.1.0",              // State management
  "axios": "^1.6.0",              // HTTP client
  "chart.js": "^4.4.0",           // Charts
  "socket.io-client": "^4.7.0",   // Real-time
  "@vueuse/core": "^10.0.0"       // Utilities
}
```

### Backend
```json
{
  "express": "^4.18.0",           // Web framework
  "mongoose": "^7.0.0",           // MongoDB ODM
  "bcryptjs": "^2.4.3",           // Password hashing
  "jsonwebtoken": "^9.0.0",       // JWT
  "socket.io": "^4.7.0",          // WebSocket
  "cors": "^2.8.5"                // CORS
}
```

---

## 🐛 Debugging Tips

### Frontend
- Use Vue DevTools extension
- Check browser console for errors
- Network tab to inspect API calls
- `localStorage.getItem('accessToken')` to check token

### Backend
- Use `console.log()` for debugging
- Check MongoDB Compass for data
- Postman/Insomnia for API testing
- Enable detailed error messages in dev

### Common Issues

**CORS Error**
- Check `FRONTEND_URL` in backend `.env`
- Ensure CORS middleware is enabled

**MongoDB Connection**
- Verify connection string
- Check IP whitelist in Atlas
- Ensure database name is correct

**Auth Issues**
- Check token expiration
- Verify JWT secret matches
- Ensure Authorization header format: `Bearer <token>`

---

## 📚 Learning Resources

### Nuxt 4
- https://nuxt.com/docs
- Nuxt UI v4: https://ui4.nuxt.com

### Express.js
- https://expressjs.com
- TypeScript: https://www.typescriptlang.org/docs

### MongoDB
- https://docs.mongodb.com
- Mongoose: https://mongoosejs.com

### Vue 3
- https://vuejs.org/guide
- Composition API: https://vuejs.org/api/composition-api-setup.html

### Socket.io
- https://socket.io/docs/v4
- Real-time: https://socket.io/blog

---

## ✅ Pre-Launch Checklist

Before deploying to production:

**Backend**
- [ ] All environment variables set
- [ ] Database backups configured
- [ ] Error logging enabled
- [ ] Rate limiting configured
- [ ] HTTPS enforced
- [ ] CORS properly restricted

**Frontend**
- [ ] Build tested: `npm run build`
- [ ] No console errors in production build
- [ ] Performance optimized
- [ ] Mobile responsive verified
- [ ] All routes working

**Security**
- [ ] Passwords hashed
- [ ] JWT secrets strong
- [ ] Input validation enabled
- [ ] SQL injection prevented
- [ ] XSS protection enabled

**Testing**
- [ ] User registration works
- [ ] Login/logout works
- [ ] Create project works
- [ ] Create task works
- [ ] Messages send in real-time

---

## 🎯 Success Criteria

Your VisMatrix.space MVP is ready when:

✅ Users can register and login
✅ Users can create projects with tasks and subtasks
✅ Users can track progress on dashboard
✅ Users can see tasks in kanban columns
✅ Users can discover other users
✅ Users can send real-time messages
✅ Progress is visualized in charts
✅ No crashes or major bugs

---

**Start building! The tech stack is modern, the architecture is scalable, and the guides are comprehensive. You've got this! 🚀**
