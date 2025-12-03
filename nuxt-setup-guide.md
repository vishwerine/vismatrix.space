# VisMatrix.space - Nuxt Frontend Setup Guide

## Quick Start

```bash
# Create new Nuxt project
npm create nuxt@latest vismatrix-frontend

# Navigate to project
cd vismatrix-frontend

# Install dependencies
npm install

# Install Nuxt UI v4 (now free and unified)
npm install @nuxt/ui

# Install additional dependencies
npm install axios pinia @vueuse/core chart.js vue-chartjs dayjs

# Start dev server
npm run dev
```

## Project Structure

```
vismatrix-frontend/
├── app.vue                     # Root component
├── nuxt.config.ts              # Nuxt configuration
├── package.json
├── tailwind.config.ts
├── tsconfig.json
├── .env.example
│
├── app/
│   ├── app.vue                 # App layout wrapper
│   ├── layouts/
│   │   ├── default.vue         # Default layout
│   │   └── auth.vue            # Auth pages layout
│   │
│   ├── pages/
│   │   ├── index.vue           # Landing/redirect
│   │   ├── dashboard.vue       # Main dashboard
│   │   ├── projects/
│   │   │   ├── [id].vue        # Project detail view
│   │   │   └── index.vue       # Projects list
│   │   ├── social/
│   │   │   ├── discover.vue    # Find users
│   │   │   ├── [username].vue  # User profile
│   │   │   └── messages.vue    # Messaging interface
│   │   ├── auth/
│   │   │   ├── login.vue       # Login page
│   │   │   ├── register.vue    # Register page
│   │   │   └── logout.vue      # Logout confirmation
│   │   └── profile.vue         # User settings
│   │
│   ├── components/
│   │   ├── Dashboard/
│   │   │   ├── ProjectCard.vue
│   │   │   ├── TaskList.vue
│   │   │   ├── ProgressChart.vue
│   │   │   └── StatsSummary.vue
│   │   ├── Social/
│   │   │   ├── UserCard.vue
│   │   │   ├── MessageThread.vue
│   │   │   └── UserDiscovery.vue
│   │   └── Common/
│   │       ├── Header.vue
│   │       └── Sidebar.vue
│   │
│   ├── composables/
│   │   ├── useAuth.ts          # Authentication logic
│   │   ├── useProjects.ts      # Project CRUD
│   │   ├── useMessages.ts      # Real-time messaging
│   │   └── useAnalytics.ts     # Progress analytics
│   │
│   ├── stores/
│   │   ├── auth.ts             # Pinia auth store
│   │   ├── projects.ts         # Projects state
│   │   ├── messages.ts         # Messages state
│   │   └── ui.ts               # UI state
│   │
│   ├── middleware/
│   │   ├── auth.ts             # Route protection
│   │   └── redirect.ts         # Auto redirects
│   │
│   ├── utils/
│   │   ├── api.ts              # API client (axios)
│   │   ├── constants.ts        # App constants
│   │   └── helpers.ts          # Utility functions
│   │
│   └── types/
│       └── index.ts            # TypeScript interfaces

├── public/                      # Static assets
│   └── logo.png

└── .env.example
```

## Key Configuration Files

### `nuxt.config.ts`
```typescript
export default defineNuxtConfig({
  modules: ['@nuxt/ui'],
  
  colorMode: {
    preference: 'system',
    fallback: 'light',
    classSuffix: '',
  },
  
  runtimeConfig: {
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_URL || 'http://localhost:5000/api',
    }
  },
  
  devServer: {
    port: 3000,
  },
  
  ssr: false, // Can be enabled for production
});
```

### `tailwind.config.ts`
```typescript
import type { Config } from 'tailwindcss'
import defaultTheme from 'tailwindcss/defaultTheme'

export default {
  content: ['./components/**/*.{js,vue,ts}', './pages/**/*.{js,vue,ts}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0fdf4',
          500: '#22c55e',
          600: '#16a34a',
          700: '#15803d',
        }
      }
    },
    fontFamily: {
      sans: ['Inter', ...defaultTheme.fontFamily.sans],
    }
  },
  plugins: [],
} satisfies Config
```

## Environment Setup

Create `.env.development`:
```
NUXT_PUBLIC_API_URL=http://localhost:5000/api
NUXT_PUBLIC_WS_URL=ws://localhost:5000
```

Create `.env.production`:
```
NUXT_PUBLIC_API_URL=https://api.vismatrix.space/api
NUXT_PUBLIC_WS_URL=wss://api.vismatrix.space
```

## Key Dependencies Overview

| Dependency | Purpose |
|-----------|---------|
| `@nuxt/ui` | Modern UI components (v4 free) |
| `pinia` | State management |
| `axios` | HTTP client |
| `chart.js` + `vue-chartjs` | Progress visualization |
| `dayjs` | Date manipulation |
| `@vueuse/core` | Vue composition utilities |

## Next Steps

1. Set up backend API (Node.js/Express)
2. Create Pinia stores for state management
3. Build authentication flow
4. Implement real-time messaging with WebSockets
5. Create progress visualization charts
6. Deploy to Vercel/Netlify
