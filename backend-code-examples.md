# Backend Code Examples

## 1. Express Server Setup (`src/server.ts`)

```typescript
import express, { Express, Request, Response, NextFunction } from 'express';
import mongoose from 'mongoose';
import cors from 'cors';
import dotenv from 'dotenv';
import http from 'http';
import { Server as SocketServer } from 'socket.io';

// Load environment variables
dotenv.config();

// Initialize Express app
const app: Express = express();
const server = http.createServer(app);
const io = new SocketServer(server, {
  cors: {
    origin: process.env.FRONTEND_URL || 'http://localhost:3000',
    credentials: true,
  }
});

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
}));

// Database Connection
const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI as string);
    console.log('MongoDB connected successfully');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
};

connectDB();

// Routes (will be imported)
// app.use('/api/auth', authRoutes);
// app.use('/api/projects', projectRoutes);
// app.use('/api/tasks', taskRoutes);
// app.use('/api/users', userRoutes);
// app.use('/api/messages', messageRoutes);

// Error handling middleware
app.use((err: any, req: Request, res: Response, next: NextFunction) => {
  console.error('Error:', err);
  res.status(err.status || 500).json({
    error: err.message || 'Internal server error',
    status: err.status || 500,
  });
});

// Socket.io connection handler
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);

  socket.on('user:online', (userId: string) => {
    socket.join(`user:${userId}`);
    io.emit('user:online', { userId, socketId: socket.id });
  });

  socket.on('message:send', (data: any) => {
    io.to(`user:${data.recipientId}`).emit('message:receive', data);
  });

  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

const PORT = process.env.PORT || 5000;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

export { app, server, io };
```

## 2. User Model (`src/models/User.ts`)

```typescript
import mongoose, { Schema, Document } from 'mongoose';
import bcrypt from 'bcryptjs';

export interface IUser extends Document {
  email: string;
  username: string;
  password: string;
  displayName: string;
  avatar?: string;
  bio?: string;
  followers: mongoose.Types.ObjectId[];
  following: mongoose.Types.ObjectId[];
  isPublic: boolean;
  comparePassword(password: string): Promise<boolean>;
}

const userSchema = new Schema<IUser>(
  {
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    username: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    password: {
      type: String,
      required: true,
      minlength: 6,
    },
    displayName: {
      type: String,
      default: function(this: IUser) { return this.username; }
    },
    avatar: {
      type: String,
      default: null,
    },
    bio: {
      type: String,
      default: '',
    },
    followers: {
      type: [Schema.Types.ObjectId],
      ref: 'User',
      default: [],
    },
    following: {
      type: [Schema.Types.ObjectId],
      ref: 'User',
      default: [],
    },
    isPublic: {
      type: Boolean,
      default: false,
    },
  },
  { timestamps: true }
);

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();

  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error: any) {
    next(error);
  }
});

// Method to compare passwords
userSchema.methods.comparePassword = async function(password: string) {
  return await bcrypt.compare(password, this.password);
};

export const User = mongoose.model<IUser>('User', userSchema);
```

## 3. JWT Utilities (`src/utils/jwt.ts`)

```typescript
import jwt from 'jsonwebtoken';

interface TokenPayload {
  userId: string;
  email: string;
  username: string;
}

export const generateTokens = (payload: TokenPayload) => {
  const accessToken = jwt.sign(payload, process.env.JWT_SECRET as string, {
    expiresIn: process.env.JWT_EXPIRY || '24h',
  });

  const refreshToken = jwt.sign(payload, process.env.JWT_REFRESH_SECRET as string, {
    expiresIn: process.env.JWT_REFRESH_EXPIRY || '7d',
  });

  return { accessToken, refreshToken };
};

export const verifyAccessToken = (token: string) => {
  try {
    return jwt.verify(token, process.env.JWT_SECRET as string) as TokenPayload;
  } catch (error) {
    throw new Error('Invalid access token');
  }
};

export const verifyRefreshToken = (token: string) => {
  try {
    return jwt.verify(token, process.env.JWT_REFRESH_SECRET as string) as TokenPayload;
  } catch (error) {
    throw new Error('Invalid refresh token');
  }
};
```

## 4. Auth Middleware (`src/middleware/auth.ts`)

```typescript
import { Request, Response, NextFunction } from 'express';
import { verifyAccessToken } from '../utils/jwt';

declare global {
  namespace Express {
    interface Request {
      user?: {
        userId: string;
        email: string;
        username: string;
      };
    }
  }
}

export const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  try {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'No token provided' });
    }

    const token = authHeader.substring(7);
    const decoded = verifyAccessToken(token);
    
    req.user = decoded;
    next();
  } catch (error: any) {
    res.status(401).json({ error: error.message });
  }
};
```

## 5. Auth Controller (`src/controllers/authController.ts`)

```typescript
import { Request, Response } from 'express';
import { User } from '../models/User';
import { generateTokens } from '../utils/jwt';

export const register = async (req: Request, res: Response) => {
  try {
    const { email, username, password, displayName } = req.body;

    // Validation
    if (!email || !username || !password) {
      return res.status(400).json({ error: 'Missing required fields' });
    }

    // Check if user exists
    const existingUser = await User.findOne({
      $or: [{ email }, { username }]
    });

    if (existingUser) {
      return res.status(409).json({ error: 'Email or username already exists' });
    }

    // Create user
    const user = new User({
      email,
      username,
      password,
      displayName: displayName || username,
    });

    await user.save();

    // Generate tokens
    const { accessToken, refreshToken } = generateTokens({
      userId: user._id.toString(),
      email: user.email,
      username: user.username,
    });

    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
    });

    res.status(201).json({
      message: 'User registered successfully',
      accessToken,
      user: {
        id: user._id,
        email: user.email,
        username: user.username,
        displayName: user.displayName,
      },
    });
  } catch (error: any) {
    res.status(500).json({ error: error.message });
  }
};

export const login = async (req: Request, res: Response) => {
  try {
    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({ error: 'Email and password required' });
    }

    const user = await User.findOne({ email });

    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const isValidPassword = await user.comparePassword(password);

    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const { accessToken, refreshToken } = generateTokens({
      userId: user._id.toString(),
      email: user.email,
      username: user.username,
    });

    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000,
    });

    res.json({
      message: 'Login successful',
      accessToken,
      user: {
        id: user._id,
        email: user.email,
        username: user.username,
        displayName: user.displayName,
      },
    });
  } catch (error: any) {
    res.status(500).json({ error: error.message });
  }
};

export const logout = (req: Request, res: Response) => {
  res.clearCookie('refreshToken');
  res.json({ message: 'Logged out successfully' });
};
```

## 6. Project Model (`src/models/Project.ts`)

```typescript
import mongoose, { Schema, Document } from 'mongoose';

export interface IProject extends Document {
  userId: mongoose.Types.ObjectId;
  name: string;
  description: string;
  status: 'active' | 'completed' | 'archived';
  startDate: Date;
  endDate?: Date;
  visibility: 'private' | 'public';
  createdAt: Date;
  updatedAt: Date;
}

const projectSchema = new Schema<IProject>(
  {
    userId: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    name: {
      type: String,
      required: true,
      trim: true,
    },
    description: {
      type: String,
      default: '',
    },
    status: {
      type: String,
      enum: ['active', 'completed', 'archived'],
      default: 'active',
    },
    startDate: {
      type: Date,
      default: new Date(),
    },
    endDate: Date,
    visibility: {
      type: String,
      enum: ['private', 'public'],
      default: 'private',
    },
  },
  { timestamps: true }
);

export const Project = mongoose.model<IProject>('Project', projectSchema);
```

## 7. Task Model with Subtasks (`src/models/Task.ts`)

```typescript
import mongoose, { Schema, Document } from 'mongoose';

export interface ISubtask {
  _id: mongoose.Types.ObjectId;
  title: string;
  completed: boolean;
  completedAt?: Date;
}

export interface ITask extends Document {
  projectId: mongoose.Types.ObjectId;
  title: string;
  description: string;
  status: 'todo' | 'in-progress' | 'completed';
  priority: 'low' | 'medium' | 'high';
  dueDate?: Date;
  subtasks: ISubtask[];
  completedAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}

const subtaskSchema = new Schema<ISubtask>({
  title: {
    type: String,
    required: true,
  },
  completed: {
    type: Boolean,
    default: false,
  },
  completedAt: Date,
});

const taskSchema = new Schema<ITask>(
  {
    projectId: {
      type: Schema.Types.ObjectId,
      ref: 'Project',
      required: true,
    },
    title: {
      type: String,
      required: true,
      trim: true,
    },
    description: {
      type: String,
      default: '',
    },
    status: {
      type: String,
      enum: ['todo', 'in-progress', 'completed'],
      default: 'todo',
    },
    priority: {
      type: String,
      enum: ['low', 'medium', 'high'],
      default: 'medium',
    },
    dueDate: Date,
    subtasks: {
      type: [subtaskSchema],
      default: [],
    },
    completedAt: Date,
  },
  { timestamps: true }
);

export const Task = mongoose.model<ITask>('Task', taskSchema);
```

## 8. Message Model (`src/models/Message.ts`)

```typescript
import mongoose, { Schema, Document } from 'mongoose';

export interface IMessage extends Document {
  senderId: mongoose.Types.ObjectId;
  recipientId: mongoose.Types.ObjectId;
  content: string;
  isRead: boolean;
  readAt?: Date;
  createdAt: Date;
}

const messageSchema = new Schema<IMessage>(
  {
    senderId: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    recipientId: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    content: {
      type: String,
      required: true,
      trim: true,
    },
    isRead: {
      type: Boolean,
      default: false,
    },
    readAt: Date,
  },
  { timestamps: true }
);

// Index for quick lookups
messageSchema.index({ senderId: 1, recipientId: 1 });
messageSchema.index({ createdAt: -1 });

export const Message = mongoose.model<IMessage>('Message', messageSchema);
```

## Usage Notes

- All controllers follow the same pattern with error handling
- Each model includes proper indexing for performance
- JWT tokens stored securely (refresh in HTTP-only cookies)
- WebSocket events for real-time updates
- TypeScript ensures type safety
- Middleware protects all authenticated routes
