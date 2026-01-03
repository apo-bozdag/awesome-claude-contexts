# React Native Expo Project Guidelines

Bu dosya, React Native/Expo projelerinde spagetti koddan kaçınmak için feature-based mimari kurallarını ve best practice'leri içerir.

---

## Proje Yapısı (Feature-Based Architecture)

```
src/
├── app/                      # Expo Router - Sadece routing, layout, navigation
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── index.tsx
│   │   └── profile.tsx
│   ├── (auth)/
│   │   ├── _layout.tsx
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── _layout.tsx           # Root layout (providers wrap here)
│   └── +not-found.tsx
│
├── features/                 # Feature modules - Her feature kendi içinde izole
│   ├── auth/
│   │   ├── api/              # Feature-specific API calls
│   │   │   ├── auth.api.ts
│   │   │   └── auth.queries.ts
│   │   ├── components/       # Feature-specific components
│   │   │   ├── LoginForm.tsx
│   │   │   └── RegisterForm.tsx
│   │   ├── hooks/            # Feature-specific hooks
│   │   │   └── useAuth.ts
│   │   ├── store/            # Feature-specific state (Zustand slice)
│   │   │   └── auth.store.ts
│   │   ├── types/            # Feature-specific types
│   │   │   └── auth.types.ts
│   │   ├── utils/            # Feature-specific utilities
│   │   │   └── validation.ts
│   │   └── index.ts          # Public API - sadece dışarı expose edilecekler
│   │
│   ├── posts/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── store/
│   │   ├── types/
│   │   └── index.ts
│   │
│   └── [other-features]/
│
├── components/               # Shared/Reusable UI components
│   ├── ui/                   # Atomic UI components (Button, Input, Text, etc.)
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Text.tsx
│   │   ├── Modal.tsx
│   │   └── index.ts
│   ├── layout/               # Layout components
│   │   ├── Container.tsx
│   │   ├── SafeArea.tsx
│   │   └── index.ts
│   └── feedback/             # Toast, Alert, Loading components
│       ├── Toast.tsx
│       └── index.ts
│
├── lib/                      # Core utilities & configurations
│   ├── api/                  # API client setup
│   │   ├── client.ts         # Axios instance + interceptors
│   │   ├── endpoints.ts      # API endpoint constants
│   │   └── types.ts          # Global API types
│   ├── storage/              # Async storage utilities
│   │   └── storage.ts
│   ├── i18n/                 # Internationalization
│   │   ├── index.ts
│   │   └── locales/
│   ├── providers/            # Context providers
│   │   ├── QueryProvider.tsx
│   │   └── index.ts
│   └── utils/                # Global utility functions
│       ├── date.ts
│       ├── format.ts
│       └── index.ts
│
├── hooks/                    # Shared custom hooks
│   ├── useDebounce.ts
│   ├── useKeyboard.ts
│   └── index.ts
│
├── store/                    # Global state (Zustand)
│   ├── app.store.ts          # App-wide state (theme, language, etc.)
│   └── index.ts
│
├── constants/                # App constants
│   ├── colors.ts
│   ├── spacing.ts
│   ├── typography.ts
│   └── index.ts
│
├── types/                    # Global TypeScript types
│   ├── navigation.ts
│   ├── common.ts
│   └── index.ts
│
└── assets/                   # Static assets
    ├── images/
    ├── fonts/
    └── icons/
```

---

## Kurallar

### 1. Feature Module Kuralları

**Her feature kendi klasöründe izole olmalı:**

```typescript
// ✅ DOĞRU: features/auth/index.ts - Public API
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/useAuth';
export { useAuthStore } from './store/auth.store';
export type { User, AuthState } from './types/auth.types';

// ❌ YANLIŞ: Dışarıdan doğrudan internal dosyalara erişim
import { validateEmail } from '@/features/auth/utils/validation'; // HAYIR
```

**Feature'lar arası bağımlılık kuralı:**
- Feature'lar birbirine doğrudan bağımlı OLMAMALI
- Ortak ihtiyaçlar `lib/` veya `components/` altına taşınmalı
- Feature'lar sadece kendi `index.ts` üzerinden export etmeli

### 2. Component Kuralları

**Component dosya yapısı:**

```typescript
// components/ui/Button.tsx
import { TouchableOpacity, Text, ActivityIndicator } from 'react-native';
import { ComponentProps } from 'react';

// Types önce
interface ButtonProps extends ComponentProps<typeof TouchableOpacity> {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  children: React.ReactNode;
}

// Component
export function Button({ 
  variant = 'primary', 
  size = 'md', 
  loading = false,
  children,
  disabled,
  ...props 
}: ButtonProps) {
  return (
    <TouchableOpacity 
      disabled={disabled || loading}
      {...props}
    >
      {loading ? <ActivityIndicator /> : <Text>{children}</Text>}
    </TouchableOpacity>
  );
}
```

**Component isimlendirme:**
- PascalCase kullan: `UserProfile.tsx`
- Compound components için: `Card.tsx`, `Card.Header.tsx`
- Test dosyaları: `Button.test.tsx`

### 3. API Layer Kuralları

**Axios client setup:**

```typescript
// lib/api/client.ts
import axios from 'axios';
import { getToken, removeToken } from '@/lib/storage/storage';

export const apiClient = axios.create({
  baseURL: process.env.EXPO_PUBLIC_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  async (config) => {
    const token = await getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      await removeToken();
      // Navigate to login
    }
    return Promise.reject(error);
  }
);
```

**Feature API dosyası:**

```typescript
// features/posts/api/posts.api.ts
import { apiClient } from '@/lib/api/client';
import type { Post, CreatePostDto } from '../types/posts.types';

export const postsApi = {
  getAll: () => 
    apiClient.get<Post[]>('/posts').then(res => res.data),
  
  getById: (id: string) => 
    apiClient.get<Post>(`/posts/${id}`).then(res => res.data),
  
  create: (data: CreatePostDto) => 
    apiClient.post<Post>('/posts', data).then(res => res.data),
  
  update: (id: string, data: Partial<CreatePostDto>) => 
    apiClient.patch<Post>(`/posts/${id}`, data).then(res => res.data),
  
  delete: (id: string) => 
    apiClient.delete(`/posts/${id}`),
};
```

**TanStack Query hooks:**

```typescript
// features/posts/api/posts.queries.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { postsApi } from './posts.api';

// Query keys - merkezi yönetim
export const postKeys = {
  all: ['posts'] as const,
  lists: () => [...postKeys.all, 'list'] as const,
  list: (filters: object) => [...postKeys.lists(), filters] as const,
  details: () => [...postKeys.all, 'detail'] as const,
  detail: (id: string) => [...postKeys.details(), id] as const,
};

// Queries
export function usePosts() {
  return useQuery({
    queryKey: postKeys.lists(),
    queryFn: postsApi.getAll,
  });
}

export function usePost(id: string) {
  return useQuery({
    queryKey: postKeys.detail(id),
    queryFn: () => postsApi.getById(id),
    enabled: !!id,
  });
}

// Mutations
export function useCreatePost() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: postsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
  });
}
```

### 4. State Management Kuralları

**Zustand store yapısı:**

```typescript
// features/auth/store/auth.store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import type { User } from '../types/auth.types';

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
}

interface AuthActions {
  setUser: (user: User | null) => void;
  logout: () => void;
  setLoading: (loading: boolean) => void;
}

export const useAuthStore = create<AuthState & AuthActions>()(
  persist(
    (set) => ({
      // State
      user: null,
      isAuthenticated: false,
      isLoading: true,

      // Actions
      setUser: (user) => set({ 
        user, 
        isAuthenticated: !!user,
        isLoading: false 
      }),
      
      logout: () => set({ 
        user: null, 
        isAuthenticated: false 
      }),
      
      setLoading: (isLoading) => set({ isLoading }),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({ user: state.user }), // Sadece user persist et
    }
  )
);
```

**State kullanım kuralları:**
- Server state için TanStack Query kullan
- Client state için Zustand kullan
- Form state için React Hook Form kullan
- İkisini karıştırma

### 5. Type Tanımlama Kuralları

```typescript
// features/posts/types/posts.types.ts

// Entity types
export interface Post {
  id: string;
  title: string;
  content: string;
  authorId: string;
  createdAt: string;
  updatedAt: string;
}

// DTO types
export interface CreatePostDto {
  title: string;
  content: string;
}

export interface UpdatePostDto {
  title?: string;
  content?: string;
}

// Response types
export interface PostsResponse {
  data: Post[];
  meta: {
    total: number;
    page: number;
    limit: number;
  };
}

// Component prop types - component dosyasında tanımla
```

### 6. Import/Export Kuralları

**Absolute imports kullan:**

```typescript
// ✅ DOĞRU
import { Button } from '@/components/ui';
import { useAuth } from '@/features/auth';
import { apiClient } from '@/lib/api/client';

// ❌ YANLIŞ
import { Button } from '../../../components/ui/Button';
```

**tsconfig.json paths:**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

**babel.config.js:**

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      ['module-resolver', {
        root: ['./'],
        alias: {
          '@': './src',
        },
      }],
    ],
  };
};
```

### 7. Barrel Export Kuralları

**Her klasörde index.ts:**

```typescript
// components/ui/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Text } from './Text';
export { Modal } from './Modal';

// Type exports
export type { ButtonProps } from './Button';
export type { InputProps } from './Input';
```

### 8. Screen/Route Kuralları

**Expo Router screen yapısı:**

```typescript
// app/(tabs)/index.tsx
import { View } from 'react-native';
import { PostList } from '@/features/posts';
import { SafeArea } from '@/components/layout';

export default function HomeScreen() {
  return (
    <SafeArea>
      <PostList />
    </SafeArea>
  );
}
```

**Screen dosyaları sadece:**
- Layout composition yapar
- Feature component'larını birleştirir
- Business logic İÇERMEZ

### 9. Provider Setup

**Root layout provider chain:**

```typescript
// app/_layout.tsx
import { Stack } from 'expo-router';
import { QueryProvider } from '@/lib/providers/QueryProvider';
import { ThemeProvider } from '@/lib/providers/ThemeProvider';

export default function RootLayout() {
  return (
    <QueryProvider>
      <ThemeProvider>
        <Stack screenOptions={{ headerShown: false }} />
      </ThemeProvider>
    </QueryProvider>
  );
}
```

**QueryClient config:**

```typescript
// lib/providers/QueryProvider.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 30,   // 30 minutes
      retry: 2,
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 1,
    },
  },
});

export function QueryProvider({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Component | PascalCase | `UserProfile.tsx` |
| Hook | camelCase, use prefix | `useAuth.ts` |
| Store | camelCase, .store suffix | `auth.store.ts` |
| API | camelCase, .api suffix | `posts.api.ts` |
| Query | camelCase, .queries suffix | `posts.queries.ts` |
| Types | camelCase, .types suffix | `auth.types.ts` |
| Utils | camelCase | `format.ts` |
| Constants | SCREAMING_SNAKE_CASE | `API_URL` |

---

## Checklist - Yeni Feature Eklerken

1. [ ] `features/[feature-name]/` klasörü oluştur
2. [ ] Types tanımla: `types/[feature].types.ts`
3. [ ] API layer: `api/[feature].api.ts`
4. [ ] Query hooks: `api/[feature].queries.ts`
5. [ ] Components: `components/`
6. [ ] Store (gerekirse): `store/[feature].store.ts`
7. [ ] Public API: `index.ts` ile export et
8. [ ] Route ekle: `app/` altında

---

## Paket Tercihleri

| Kategori | Paket | Neden |
|----------|-------|-------|
| Navigation | expo-router | File-based routing, Expo native |
| State (Server) | @tanstack/react-query | Caching, background sync |
| State (Client) | zustand | Minimal, hooks-based |
| HTTP Client | axios | Interceptors, error handling |
| Forms | react-hook-form + zod | Performance, validation |
| Storage | @react-native-async-storage/async-storage | Standard, reliable |
| Styling | nativewind | Tailwind for RN |

---

## Anti-Patterns (YAPMA)

```typescript
// ❌ Component içinde API call
function UserList() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/users').then(res => res.json()).then(setUsers);
  }, []);
}

// ✅ Query hook kullan
function UserList() {
  const { data: users } = useUsers();
}
```

```typescript
// ❌ Props drilling
<App>
  <Parent user={user}>
    <Child user={user}>
      <GrandChild user={user} />
    </Child>
  </Parent>
</App>

// ✅ Store kullan
function GrandChild() {
  const user = useAuthStore(state => state.user);
}
```

```typescript
// ❌ God component
function Dashboard() {
  // 500 satır kod, her şeyi yapıyor
}

// ✅ Feature composition
function Dashboard() {
  return (
    <>
      <StatsOverview />
      <RecentActivity />
      <QuickActions />
    </>
  );
}
```

---

## Notes

- Her PR'da bu kurallara uygunluk kontrol edilmeli
- Yeni pattern eklendiğinde bu dosya güncellenmel
- TypeScript strict mode aktif olmalı
