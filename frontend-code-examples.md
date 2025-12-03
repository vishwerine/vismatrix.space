# Frontend Vue 3 + Nuxt Code Examples

## 1. Pinia Auth Store (`stores/auth.ts`)

```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import api from '~/utils/api';

export const useAuthStore = defineStore('auth', () => {
  const user = ref<any>(null);
  const accessToken = ref<string>('');
  const loading = ref(false);
  const error = ref<string>('');

  const isAuthenticated = computed(() => !!accessToken.value && !!user.value);

  const register = async (email: string, username: string, password: string, displayName: string) => {
    loading.value = true;
    error.value = '';
    try {
      const { data } = await api.post('/auth/register', {
        email,
        username,
        password,
        displayName,
      });
      accessToken.value = data.accessToken;
      user.value = data.user;
      localStorage.setItem('accessToken', data.accessToken);
      return true;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Registration failed';
      return false;
    } finally {
      loading.value = false;
    }
  };

  const login = async (email: string, password: string) => {
    loading.value = true;
    error.value = '';
    try {
      const { data } = await api.post('/auth/login', { email, password });
      accessToken.value = data.accessToken;
      user.value = data.user;
      localStorage.setItem('accessToken', data.accessToken);
      return true;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Login failed';
      return false;
    } finally {
      loading.value = false;
    }
  };

  const logout = async () => {
    try {
      await api.post('/auth/logout');
    } finally {
      accessToken.value = '';
      user.value = null;
      localStorage.removeItem('accessToken');
    }
  };

  const loadStoredToken = () => {
    const token = localStorage.getItem('accessToken');
    if (token) {
      accessToken.value = token;
    }
  };

  return {
    user,
    accessToken,
    isAuthenticated,
    loading,
    error,
    register,
    login,
    logout,
    loadStoredToken,
  };
});
```

## 2. Pinia Projects Store (`stores/projects.ts`)

```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import api from '~/utils/api';

interface Task {
  _id: string;
  title: string;
  status: 'todo' | 'in-progress' | 'completed';
  priority: 'low' | 'medium' | 'high';
  subtasks: any[];
  dueDate?: string;
}

interface Project {
  _id: string;
  name: string;
  description: string;
  status: 'active' | 'completed' | 'archived';
  visibility: 'private' | 'public';
  startDate: string;
  tasks?: Task[];
  userId: string;
}

export const useProjectStore = defineStore('projects', () => {
  const projects = ref<Project[]>([]);
  const currentProject = ref<Project | null>(null);
  const loading = ref(false);
  const error = ref<string>('');

  const projectStats = computed(() => {
    if (!currentProject.value?.tasks) return { total: 0, completed: 0, inProgress: 0 };
    const tasks = currentProject.value.tasks;
    return {
      total: tasks.length,
      completed: tasks.filter(t => t.status === 'completed').length,
      inProgress: tasks.filter(t => t.status === 'in-progress').length,
    };
  });

  const completionPercentage = computed(() => {
    if (projectStats.value.total === 0) return 0;
    return Math.round((projectStats.value.completed / projectStats.value.total) * 100);
  });

  const fetchProjects = async () => {
    loading.value = true;
    error.value = '';
    try {
      const { data } = await api.get('/projects');
      projects.value = data.projects;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Failed to fetch projects';
    } finally {
      loading.value = false;
    }
  };

  const fetchProject = async (projectId: string) => {
    loading.value = true;
    error.value = '';
    try {
      const { data } = await api.get(`/projects/${projectId}`);
      currentProject.value = data.project;
      return currentProject.value;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Failed to fetch project';
    } finally {
      loading.value = false;
    }
  };

  const createProject = async (name: string, description: string, visibility: 'private' | 'public' = 'private') => {
    loading.value = true;
    error.value = '';
    try {
      const { data } = await api.post('/projects', {
        name,
        description,
        visibility,
      });
      projects.value.push(data.project);
      return data.project;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Failed to create project';
    } finally {
      loading.value = false;
    }
  };

  const updateProject = async (projectId: string, updates: Partial<Project>) => {
    loading.value = true;
    error.value = '';
    try {
      const { data } = await api.put(`/projects/${projectId}`, updates);
      const index = projects.value.findIndex(p => p._id === projectId);
      if (index > -1) projects.value[index] = data.project;
      if (currentProject.value?._id === projectId) currentProject.value = data.project;
      return data.project;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Failed to update project';
    } finally {
      loading.value = false;
    }
  };

  const deleteProject = async (projectId: string) => {
    loading.value = true;
    error.value = '';
    try {
      await api.delete(`/projects/${projectId}`);
      projects.value = projects.value.filter(p => p._id !== projectId);
      if (currentProject.value?._id === projectId) currentProject.value = null;
      return true;
    } catch (err: any) {
      error.value = err.response?.data?.error || 'Failed to delete project';
      return false;
    } finally {
      loading.value = false;
    }
  };

  return {
    projects,
    currentProject,
    loading,
    error,
    projectStats,
    completionPercentage,
    fetchProjects,
    fetchProject,
    createProject,
    updateProject,
    deleteProject,
  };
});
```

## 3. API Client (`utils/api.ts`)

```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: useRuntimeConfig().public.apiBase,
  withCredentials: true,
});

// Request interceptor to add auth token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor for token refresh
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Token expired, try refresh
      try {
        const { data } = await axios.post(`${useRuntimeConfig().public.apiBase}/auth/refresh`, {}, {
          withCredentials: true,
        });
        localStorage.setItem('accessToken', data.accessToken);
        // Retry original request
        return api(error.config);
      } catch {
        // Refresh failed, redirect to login
        localStorage.removeItem('accessToken');
        await navigateTo('/auth/login');
      }
    }
    return Promise.reject(error);
  }
);

export default api;
```

## 4. Dashboard Page (`pages/dashboard.vue`)

```vue
<template>
  <div class="space-y-6">
    <!-- Header -->
    <div class="flex justify-between items-center">
      <h1 class="text-3xl font-bold text-gray-900 dark:text-white">Dashboard</h1>
      <UButton to="/projects/new" color="primary" size="lg">
        <UIcon name="i-heroicons-plus" /> New Project
      </UButton>
    </div>

    <!-- Stats Summary -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <StatCard 
        title="Total Projects"
        :value="projectStore.projects.length"
        icon="i-heroicons-folder"
        color="blue"
      />
      <StatCard 
        title="Active Projects"
        :value="activeProjectsCount"
        icon="i-heroicons-fire"
        color="orange"
      />
      <StatCard 
        title="Completed"
        :value="completedProjectsCount"
        icon="i-heroicons-check-circle"
        color="green"
      />
    </div>

    <!-- Projects Grid -->
    <div v-if="projectStore.projects.length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <ProjectCard 
        v-for="project in projectStore.projects"
        :key="project._id"
        :project="project"
        @click="navigateTo(`/projects/${project._id}`)"
      />
    </div>

    <!-- Empty State -->
    <div v-else class="text-center py-12">
      <UIcon name="i-heroicons-folder-open" class="mx-auto h-12 w-12 text-gray-400" />
      <h3 class="mt-2 text-sm font-medium text-gray-900 dark:text-white">No projects yet</h3>
      <p class="mt-1 text-sm text-gray-500">Get started by creating your first project.</p>
      <UButton to="/projects/new" class="mt-4">Create Project</UButton>
    </div>

    <!-- Social Element - Recent Activity -->
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 mt-8">
      <div class="lg:col-span-2">
        <h2 class="text-xl font-bold mb-4">Following Activity</h2>
        <div class="space-y-4">
          <ActivityFeed v-if="followingActivity.length > 0" :activities="followingActivity" />
          <div v-else class="text-center py-8 text-gray-500">
            <p>No activity from users you're following</p>
            <UButton to="/social/discover" variant="ghost">Discover Users</UButton>
          </div>
        </div>
      </div>

      <!-- Messages Sidebar -->
      <div class="space-y-4">
        <h2 class="text-xl font-bold">Messages</h2>
        <MessagePreview v-for="conversation in recentConversations" :key="conversation._id" :conversation="conversation" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useProjectStore } from '~/stores/projects';

const projectStore = useProjectStore();
const followingActivity = ref([]);
const recentConversations = ref([]);

const activeProjectsCount = computed(() => {
  return projectStore.projects.filter(p => p.status === 'active').length;
});

const completedProjectsCount = computed(() => {
  return projectStore.projects.filter(p => p.status === 'completed').length;
});

onMounted(async () => {
  await projectStore.fetchProjects();
});
</script>
```

## 5. Project Detail Page (`pages/projects/[id].vue`)

```vue
<template>
  <div class="space-y-6">
    <div class="flex justify-between items-center">
      <h1 class="text-3xl font-bold">{{ projectStore.currentProject?.name }}</h1>
      <UDropdown :items="projectActions">
        <UButton color="gray" variant="ghost">
          <UIcon name="i-heroicons-ellipsis-vertical" />
        </UButton>
      </UDropdown>
    </div>

    <!-- Progress Visualization -->
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div class="lg:col-span-2">
        <ProgressChart 
          :project="projectStore.currentProject"
          title="Project Progress"
        />
      </div>
      <StatCard 
        title="Completion"
        :value="`${projectStore.completionPercentage}%`"
        :subtitle="`${projectStore.projectStats.completed}/${projectStore.projectStats.total} tasks`"
        color="primary"
      />
    </div>

    <!-- Tasks Section -->
    <div class="space-y-4">
      <div class="flex justify-between items-center">
        <h2 class="text-2xl font-bold">Tasks</h2>
        <UButton @click="showNewTaskModal = true" color="primary">
          <UIcon name="i-heroicons-plus" /> Add Task
        </UButton>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <!-- Todo Column -->
        <TaskColumn status="todo" title="To Do" :tasks="todoTasks" @add-task="addTask" />
        
        <!-- In Progress Column -->
        <TaskColumn status="in-progress" title="In Progress" :tasks="inProgressTasks" @add-task="addTask" />
        
        <!-- Completed Column -->
        <TaskColumn status="completed" title="Completed" :tasks="completedTasks" @add-task="addTask" />
      </div>
    </div>

    <!-- New Task Modal -->
    <UModal v-model="showNewTaskModal">
      <UCard>
        <template #header>
          <h3 class="text-lg font-bold">Create New Task</h3>
        </template>

        <form @submit.prevent="submitNewTask" class="space-y-4">
          <UFormGroup label="Task Title">
            <UInput v-model="newTask.title" placeholder="Enter task title" />
          </UFormGroup>

          <UFormGroup label="Description">
            <UTextarea v-model="newTask.description" placeholder="Task description" />
          </UFormGroup>

          <UFormGroup label="Priority">
            <USelectMenu 
              v-model="newTask.priority"
              :options="['low', 'medium', 'high']"
            />
          </UFormGroup>

          <UFormGroup label="Due Date">
            <UInput v-model="newTask.dueDate" type="date" />
          </UFormGroup>

          <div class="flex gap-2">
            <UButton type="submit" color="primary">Create Task</UButton>
            <UButton @click="showNewTaskModal = false" color="gray">Cancel</UButton>
          </div>
        </form>
      </UCard>
    </UModal>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useProjectStore } from '~/stores/projects';
import { useRoute } from 'vue-router';

const route = useRoute();
const projectStore = useProjectStore();
const showNewTaskModal = ref(false);
const newTask = ref({
  title: '',
  description: '',
  priority: 'medium',
  dueDate: '',
});

const todoTasks = computed(() => 
  projectStore.currentProject?.tasks?.filter(t => t.status === 'todo') || []
);

const inProgressTasks = computed(() =>
  projectStore.currentProject?.tasks?.filter(t => t.status === 'in-progress') || []
);

const completedTasks = computed(() =>
  projectStore.currentProject?.tasks?.filter(t => t.status === 'completed') || []
);

const projectActions = [
  [{
    label: 'Edit Project',
    icon: 'i-heroicons-pencil',
    click: () => console.log('Edit')
  }],
  [{
    label: 'Archive Project',
    icon: 'i-heroicons-archive-box',
    click: () => console.log('Archive')
  }],
  [{
    label: 'Delete Project',
    icon: 'i-heroicons-trash',
    click: () => confirmDelete()
  }]
];

const addTask = async () => {
  showNewTaskModal.value = true;
};

const submitNewTask = async () => {
  if (!newTask.value.title.trim()) {
    alert('Task title is required');
    return;
  }

  // API call here
  showNewTaskModal.value = false;
  newTask.value = { title: '', description: '', priority: 'medium', dueDate: '' };
};

const confirmDelete = () => {
  if (confirm('Are you sure you want to delete this project?')) {
    projectStore.deleteProject(route.params.id as string)
      .then(() => navigateTo('/dashboard'));
  }
};

onMounted(async () => {
  await projectStore.fetchProject(route.params.id as string);
});
</script>
```

## 6. Social Discovery Page (`pages/social/discover.vue`)

```vue
<template>
  <div class="space-y-6">
    <div class="flex justify-between items-center">
      <h1 class="text-3xl font-bold">Discover Users</h1>
      <UInput 
        v-model="searchQuery"
        placeholder="Search users..."
        icon="i-heroicons-magnifying-glass"
        @update:model-value="searchUsers"
      />
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <UserCard 
        v-for="user in users"
        :key="user._id"
        :user="user"
        :is-following="isFollowing(user._id)"
        @follow="followUser"
        @unfollow="unfollowUser"
        @message="openChat"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import api from '~/utils/api';

const users = ref([]);
const searchQuery = ref('');
const following = ref(new Set());

const isFollowing = (userId: string) => following.value.has(userId);

const searchUsers = async (query: string) => {
  if (!query.trim()) {
    await loadUsers();
    return;
  }

  try {
    const { data } = await api.get('/users/discover', { params: { search: query } });
    users.value = data.users;
  } catch (error) {
    console.error('Search failed:', error);
  }
};

const loadUsers = async () => {
  try {
    const { data } = await api.get('/users/discover');
    users.value = data.users;
  } catch (error) {
    console.error('Failed to load users:', error);
  }
};

const followUser = async (userId: string) => {
  try {
    await api.post(`/users/${userId}/follow`);
    following.value.add(userId);
  } catch (error) {
    console.error('Follow failed:', error);
  }
};

const unfollowUser = async (userId: string) => {
  try {
    await api.delete(`/users/${userId}/follow`);
    following.value.delete(userId);
  } catch (error) {
    console.error('Unfollow failed:', error);
  }
};

const openChat = (userId: string) => {
  navigateTo(`/social/messages?with=${userId}`);
};

onMounted(loadUsers);
</script>
```

## 7. Messaging Page (`pages/social/messages.vue`)

```vue
<template>
  <div class="flex h-screen gap-4">
    <!-- Conversations List -->
    <div class="w-80 border-r border-gray-200 dark:border-gray-700 overflow-y-auto">
      <div class="p-4 space-y-2">
        <ConversationItem 
          v-for="conv in conversations"
          :key="conv._id"
          :conversation="conv"
          :is-active="activeConversation?._id === conv._id"
          @select="selectConversation"
        />
      </div>
    </div>

    <!-- Chat Window -->
    <div v-if="activeConversation" class="flex-1 flex flex-col">
      <!-- Header -->
      <div class="border-b border-gray-200 dark:border-gray-700 p-4 flex justify-between items-center">
        <h2 class="text-lg font-bold">{{ activeConversation.otherUser.displayName }}</h2>
        <span v-if="isOnline(activeConversation.otherUser._id)" class="text-sm text-green-500">Online</span>
      </div>

      <!-- Messages -->
      <div class="flex-1 overflow-y-auto p-4 space-y-4">
        <MessageBubble 
          v-for="msg in activeConversation.messages"
          :key="msg._id"
          :message="msg"
          :is-own="msg.senderId === currentUser._id"
        />
      </div>

      <!-- Input -->
      <div class="border-t border-gray-200 dark:border-gray-700 p-4 flex gap-2">
        <UInput 
          v-model="messageInput"
          placeholder="Type a message..."
          @keydown.enter="sendMessage"
        />
        <UButton @click="sendMessage" color="primary">
          <UIcon name="i-heroicons-paper-airplane" />
        </UButton>
      </div>
    </div>

    <div v-else class="flex-1 flex items-center justify-center text-gray-500">
      Select a conversation to start messaging
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import api from '~/utils/api';
import { io } from 'socket.io-client';

const route = useRoute();
const conversations = ref([]);
const activeConversation = ref(null);
const messageInput = ref('');
const currentUser = ref(null);
const onlineUsers = ref(new Set());
let socket: any;

const isOnline = (userId: string) => onlineUsers.value.has(userId);

const loadConversations = async () => {
  try {
    const { data } = await api.get('/messages');
    conversations.value = data.conversations;
  } catch (error) {
    console.error('Failed to load conversations:', error);
  }
};

const selectConversation = (conversation: any) => {
  activeConversation.value = conversation;
};

const sendMessage = async () => {
  if (!messageInput.value.trim() || !activeConversation.value) return;

  const message = {
    content: messageInput.value,
    recipientId: activeConversation.value.otherUser._id,
    timestamp: new Date(),
  };

  try {
    const { data } = await api.post(`/messages/${activeConversation.value.otherUser._id}`, {
      content: messageInput.value,
    });

    activeConversation.value.messages.push(data.message);
    messageInput.value = '';

    // Emit via socket for real-time delivery
    socket.emit('message:send', {
      ...data.message,
      recipientId: activeConversation.value.otherUser._id,
    });
  } catch (error) {
    console.error('Failed to send message:', error);
  }
};

const initSocket = () => {
  socket = io(useRuntimeConfig().public.wsUrl);

  socket.on('connect', () => {
    socket.emit('user:online', currentUser.value._id);
  });

  socket.on('user:online', (data: any) => {
    onlineUsers.value.add(data.userId);
  });

  socket.on('user:offline', (data: any) => {
    onlineUsers.value.delete(data.userId);
  });

  socket.on('message:receive', (message: any) => {
    // Handle incoming message
    if (activeConversation.value?.otherUser._id === message.senderId) {
      activeConversation.value.messages.push(message);
    }
  });
};

onMounted(async () => {
  await loadConversations();
  initSocket();

  // Auto-select conversation if userId provided in query
  if (route.query.with) {
    const conv = conversations.value.find(c => c.otherUser._id === route.query.with);
    if (conv) selectConversation(conv);
  }
});
</script>
```

These components showcase:
- ✅ Modern Vue 3 Composition API
- ✅ TypeScript integration
- ✅ Nuxt UI components
- ✅ Real-time features
- ✅ State management with Pinia
- ✅ API integration with interceptors
- ✅ WebSocket messaging
- ✅ Task management with statuses
- ✅ User discovery and social features
- ✅ Responsive design
