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

# Abdullah's Design System — "Vinyl Editorial"

Bir plak kapağının iç notları gibi. Vintage, sıcak, tipografi-odaklı.

---

## Design Philosophy

```
"Dijital ama elle tutulur hissettiren,
 modern ama nostaljik,
 minimal ama karakterli."
```

**Temel Prensipler:**
- Typography-first: Yazı tasarımın omurgası
- Warm over cool: Soğuk mavi/gri yerine sıcak tonlar
- Subtle texture: Flat değil, hafif doku hissi
- Intentional whitespace: Nefes alan tasarım
- Details matter: İtalikler, ince çizgiler, küçük dokunuşlar

---

## Color Tokens

### NativeWind / Tailwind Config

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        // === BASE PALETTE ===
        cream: {
          50:  '#FDFCFA',   // Lightest - page background
          100: '#FAF8F5',   // Default cream
          200: '#F5F0E8',   // Slightly darker
          300: '#EBE4D8',   // Cards, elevated surfaces
          400: '#DDD4C4',   // Borders, dividers
        },
        
        // === TEXT COLORS ===
        ink: {
          900: '#2C2416',   // Darkest - headlines
          800: '#3D3929',   // Primary text
          700: '#5C5344',   // Secondary text
          600: '#7A7162',   // Tertiary/muted
          500: '#968C7D',   // Placeholder
          400: '#B5ADA0',   // Disabled
        },
        
        // === ACCENT COLORS ===
        terracotta: {
          DEFAULT: '#C45A3B',
          light:   '#D4735A',
          dark:    '#A84A2F',
          muted:   '#C45A3B20', // 12% opacity for backgrounds
        },
        
        burgundy: {
          DEFAULT: '#722F37',
          light:   '#8B3D47',
          dark:    '#5A252C',
          muted:   '#722F3720',
        },
        
        // === SEMANTIC COLORS ===
        success: '#4A6741',     // Olive green
        warning: '#C4873B',     // Warm amber
        error:   '#A84A2F',     // Same as terracotta dark
        info:    '#5C5344',     // Ink 700
        
        // === SURFACE COLORS ===
        surface: {
          primary:   '#FAF8F5',  // Main background
          secondary: '#F5F0E8',  // Cards
          elevated:  '#FDFCFA',  // Modals, sheets
          inverse:   '#2C2416',  // Dark surfaces
        },
      },
      
      fontFamily: {
        // Serif - Headlines, emphasis
        'serif': ['Lora', 'Georgia', 'serif'],
        'display': ['Playfair Display', 'Georgia', 'serif'],
        
        // Sans - Body text
        'sans': ['Inter', 'system-ui', 'sans-serif'],
        'body': ['Source Sans 3', 'system-ui', 'sans-serif'],
        
        // Mono - Code, data
        'mono': ['JetBrains Mono', 'Menlo', 'monospace'],
      },
      
      fontSize: {
        // Mobile-first scale
        'xs':   ['12px', { lineHeight: '16px', letterSpacing: '0.02em' }],
        'sm':   ['14px', { lineHeight: '20px', letterSpacing: '0.01em' }],
        'base': ['16px', { lineHeight: '24px', letterSpacing: '0' }],
        'lg':   ['18px', { lineHeight: '28px', letterSpacing: '-0.01em' }],
        'xl':   ['20px', { lineHeight: '28px', letterSpacing: '-0.02em' }],
        '2xl':  ['24px', { lineHeight: '32px', letterSpacing: '-0.02em' }],
        '3xl':  ['30px', { lineHeight: '36px', letterSpacing: '-0.02em' }],
        '4xl':  ['36px', { lineHeight: '40px', letterSpacing: '-0.03em' }],
      },
      
      borderRadius: {
        'none': '0',
        'sm':   '4px',
        'DEFAULT': '8px',
        'lg':   '12px',
        'xl':   '16px',
        'full': '9999px',
      },
      
      boxShadow: {
        'subtle': '0 1px 2px rgba(44, 36, 22, 0.04)',
        'soft':   '0 2px 8px rgba(44, 36, 22, 0.08)',
        'medium': '0 4px 16px rgba(44, 36, 22, 0.12)',
        'strong': '0 8px 32px rgba(44, 36, 22, 0.16)',
      },
      
      borderWidth: {
        'thin': '0.5px',
        'DEFAULT': '1px',
        'medium': '1.5px',
        'thick': '2px',
      },
      
      spacing: {
        '0.5': '2px',
        '1':   '4px',
        '1.5': '6px',
        '2':   '8px',
        '3':   '12px',
        '4':   '16px',
        '5':   '20px',
        '6':   '24px',
        '8':   '32px',
        '10':  '40px',
        '12':  '48px',
        '16':  '64px',
        '20':  '80px',
      },
    },
  },
};
```

---

## Typography System

### Hierarchy Rules

```
HEADLINE (Serif, Bold)
├── H1: 30-36px, Lora Bold, ink-900, letter-spacing: -2%
├── H2: 24px, Lora Bold, ink-900, letter-spacing: -2%
├── H3: 20px, Lora SemiBold, ink-800, letter-spacing: -1%
└── H4: 18px, Lora Medium, ink-800, letter-spacing: -1%

BODY (Sans)
├── Large: 18px, Inter Regular, ink-700
├── Base: 16px, Inter Regular, ink-700
├── Small: 14px, Inter Regular, ink-600
└── Caption: 12px, Inter Medium, ink-500, letter-spacing: +2%

SPECIAL
├── Quote: 18px, Lora Italic, ink-700
├── Label: 12px, Inter SemiBold, ink-600, uppercase, letter-spacing: +8%
├── Code: 14px, JetBrains Mono, ink-800
└── Link: Underline, terracotta on hover
```

### React Native Implementation

```tsx
// src/components/ui/typography/Text.tsx
import { Text as RNText, TextProps, StyleSheet } from 'react-native';
import { styled } from 'nativewind';

type Variant = 
  | 'h1' | 'h2' | 'h3' | 'h4'
  | 'body' | 'body-lg' | 'body-sm'
  | 'caption' | 'label' | 'quote';

interface Props extends TextProps {
  variant?: Variant;
  color?: 'primary' | 'secondary' | 'muted' | 'accent';
  italic?: boolean;
}

const variantStyles = {
  h1: 'font-serif text-3xl font-bold text-ink-900 tracking-tight',
  h2: 'font-serif text-2xl font-bold text-ink-900 tracking-tight',
  h3: 'font-serif text-xl font-semibold text-ink-800',
  h4: 'font-serif text-lg font-medium text-ink-800',
  
  'body': 'font-sans text-base text-ink-700',
  'body-lg': 'font-sans text-lg text-ink-700',
  'body-sm': 'font-sans text-sm text-ink-600',
  
  'caption': 'font-sans text-xs text-ink-500 tracking-wide',
  'label': 'font-sans text-xs font-semibold text-ink-600 uppercase tracking-widest',
  'quote': 'font-serif text-lg text-ink-700 italic',
};

const colorStyles = {
  primary: 'text-ink-800',
  secondary: 'text-ink-600',
  muted: 'text-ink-500',
  accent: 'text-terracotta',
};

export function Text({ 
  variant = 'body', 
  color,
  italic,
  className,
  ...props 
}: Props) {
  return (
    <RNText 
      className={`
        ${variantStyles[variant]}
        ${color ? colorStyles[color] : ''}
        ${italic ? 'italic' : ''}
        ${className}
      `}
      {...props}
    />
  );
}

// Usage:
// <Text variant="h1">Headline</Text>
// <Text variant="quote">Bu bir alıntı metnidir.</Text>
// <Text variant="label">KATEGORI</Text>
// <Text variant="body" italic>Vurgulu metin</Text>
```

---

## Component Patterns

### Card Component

```tsx
// src/components/ui/Card.tsx
import { View, ViewProps, Pressable } from 'react-native';

interface CardProps extends ViewProps {
  variant?: 'default' | 'elevated' | 'outlined';
  padding?: 'none' | 'sm' | 'md' | 'lg';
  onPress?: () => void;
}

const variants = {
  default: 'bg-cream-200',
  elevated: 'bg-cream-50 shadow-soft',
  outlined: 'bg-transparent border border-cream-400',
};

const paddings = {
  none: 'p-0',
  sm: 'p-3',
  md: 'p-4',
  lg: 'p-6',
};

export function Card({ 
  variant = 'default',
  padding = 'md',
  onPress,
  className,
  children,
  ...props 
}: CardProps) {
  const Wrapper = onPress ? Pressable : View;
  
  return (
    <Wrapper
      onPress={onPress}
      className={`
        rounded-lg
        ${variants[variant]}
        ${paddings[padding]}
        ${onPress ? 'active:opacity-90' : ''}
        ${className}
      `}
      {...props}
    >
      {children}
    </Wrapper>
  );
}

// Subcomponents
Card.Header = function CardHeader({ children, className }: ViewProps) {
  return (
    <View className={`border-b border-cream-400 pb-3 mb-3 ${className}`}>
      {children}
    </View>
  );
};

Card.Footer = function CardFooter({ children, className }: ViewProps) {
  return (
    <View className={`border-t border-cream-400 pt-3 mt-3 ${className}`}>
      {children}
    </View>
  );
};
```

### Button Component

```tsx
// src/components/ui/Button.tsx
import { Pressable, PressableProps, ActivityIndicator } from 'react-native';
import { Text } from './typography/Text';

interface ButtonProps extends PressableProps {
  variant?: 'primary' | 'secondary' | 'ghost' | 'link';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
}

const variants = {
  primary: {
    container: 'bg-terracotta active:bg-terracotta-dark',
    text: 'text-cream-50 font-semibold',
  },
  secondary: {
    container: 'bg-cream-300 active:bg-cream-400',
    text: 'text-ink-800 font-semibold',
  },
  ghost: {
    container: 'bg-transparent active:bg-cream-200',
    text: 'text-ink-700 font-medium',
  },
  link: {
    container: 'bg-transparent',
    text: 'text-terracotta underline font-medium',
  },
};

const sizes = {
  sm: { container: 'px-3 py-1.5 rounded', text: 'text-sm' },
  md: { container: 'px-4 py-2.5 rounded-lg', text: 'text-base' },
  lg: { container: 'px-6 py-3.5 rounded-lg', text: 'text-lg' },
};

export function Button({
  variant = 'primary',
  size = 'md',
  loading,
  disabled,
  icon,
  iconPosition = 'left',
  children,
  className,
  ...props
}: ButtonProps) {
  const v = variants[variant];
  const s = sizes[size];
  
  return (
    <Pressable
      disabled={disabled || loading}
      className={`
        flex-row items-center justify-center
        ${v.container}
        ${s.container}
        ${disabled ? 'opacity-50' : ''}
        ${className}
      `}
      {...props}
    >
      {loading ? (
        <ActivityIndicator color={variant === 'primary' ? '#FAF8F5' : '#3D3929'} />
      ) : (
        <>
          {icon && iconPosition === 'left' && <View className="mr-2">{icon}</View>}
          <Text className={`${v.text} ${s.text}`}>{children}</Text>
          {icon && iconPosition === 'right' && <View className="ml-2">{icon}</View>}
        </>
      )}
    </Pressable>
  );
}
```

### Input Component

```tsx
// src/components/ui/Input.tsx
import { TextInput, TextInputProps, View } from 'react-native';
import { Text } from './typography/Text';

interface InputProps extends TextInputProps {
  label?: string;
  hint?: string;
  error?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export function Input({
  label,
  hint,
  error,
  leftIcon,
  rightIcon,
  className,
  ...props
}: InputProps) {
  const hasError = !!error;
  
  return (
    <View className="w-full">
      {label && (
        <Text variant="label" className="mb-1.5 ml-0.5">
          {label}
        </Text>
      )}
      
      <View className={`
        flex-row items-center
        bg-cream-100
        border rounded-lg
        ${hasError ? 'border-error' : 'border-cream-400'}
        focus-within:border-terracotta
      `}>
        {leftIcon && (
          <View className="pl-3 pr-2">{leftIcon}</View>
        )}
        
        <TextInput
          className={`
            flex-1 py-3 px-3
            font-sans text-base text-ink-800
            ${leftIcon ? 'pl-0' : ''}
            ${rightIcon ? 'pr-0' : ''}
            ${className}
          `}
          placeholderTextColor="#968C7D"
          {...props}
        />
        
        {rightIcon && (
          <View className="pr-3 pl-2">{rightIcon}</View>
        )}
      </View>
      
      {(hint || error) && (
        <Text 
          variant="caption" 
          className={`mt-1.5 ml-0.5 ${hasError ? 'text-error' : ''}`}
        >
          {error || hint}
        </Text>
      )}
    </View>
  );
}
```

### Divider / Rule

```tsx
// src/components/ui/Divider.tsx
import { View } from 'react-native';
import { Text } from './typography/Text';

interface DividerProps {
  label?: string;
  spacing?: 'sm' | 'md' | 'lg';
}

const spacings = {
  sm: 'my-3',
  md: 'my-4',
  lg: 'my-6',
};

export function Divider({ label, spacing = 'md' }: DividerProps) {
  if (label) {
    return (
      <View className={`flex-row items-center ${spacings[spacing]}`}>
        <View className="flex-1 h-[0.5px] bg-cream-400" />
        <Text variant="label" className="mx-4">{label}</Text>
        <View className="flex-1 h-[0.5px] bg-cream-400" />
      </View>
    );
  }
  
  return (
    <View className={`h-[0.5px] bg-cream-400 ${spacings[spacing]}`} />
  );
}
```

### Quote Block

```tsx
// src/components/ui/Quote.tsx
import { View } from 'react-native';
import { Text } from './typography/Text';

interface QuoteProps {
  children: string;
  author?: string;
  source?: string;
}

export function Quote({ children, author, source }: QuoteProps) {
  return (
    <View className="border-l-2 border-terracotta pl-4 py-2 my-4">
      <Text variant="quote" className="mb-2">
        "{children}"
      </Text>
      {(author || source) && (
        <Text variant="caption" className="text-ink-600">
          — {author}{source && `, ${source}`}
        </Text>
      )}
    </View>
  );
}
```

---

## Layout Patterns

### Screen Container

```tsx
// src/components/layout/ScreenContainer.tsx
import { View, ScrollView, SafeAreaView } from 'react-native';

interface ScreenContainerProps {
  children: React.ReactNode;
  scrollable?: boolean;
  padded?: boolean;
  safeArea?: boolean;
}

export function ScreenContainer({
  children,
  scrollable = true,
  padded = true,
  safeArea = true,
}: ScreenContainerProps) {
  const Container = safeArea ? SafeAreaView : View;
  const Inner = scrollable ? ScrollView : View;
  
  return (
    <Container className="flex-1 bg-cream-100">
      <Inner 
        className={`flex-1 ${padded ? 'px-4' : ''}`}
        contentContainerStyle={scrollable ? { paddingBottom: 32 } : undefined}
        showsVerticalScrollIndicator={false}
      >
        {children}
      </Inner>
    </Container>
  );
}
```

### Section

```tsx
// src/components/layout/Section.tsx
import { View } from 'react-native';
import { Text } from '../ui/typography/Text';

interface SectionProps {
  title?: string;
  subtitle?: string;
  children: React.ReactNode;
  spacing?: 'sm' | 'md' | 'lg';
}

const spacings = {
  sm: 'mb-4',
  md: 'mb-6',
  lg: 'mb-8',
};

export function Section({ 
  title, 
  subtitle, 
  children, 
  spacing = 'md' 
}: SectionProps) {
  return (
    <View className={spacings[spacing]}>
      {(title || subtitle) && (
        <View className="mb-3">
          {title && <Text variant="h3">{title}</Text>}
          {subtitle && <Text variant="body-sm" color="secondary">{subtitle}</Text>}
        </View>
      )}
      {children}
    </View>
  );
}
```

---

## Icon Style

**Kullanılacak icon seti:** Lucide veya Phosphor (outline style)

```tsx
// Icon wrapper for consistent sizing and color
import { LucideIcon } from 'lucide-react-native';

interface IconProps {
  icon: LucideIcon;
  size?: 'sm' | 'md' | 'lg';
  color?: 'primary' | 'secondary' | 'accent' | 'muted';
}

const sizes = { sm: 16, md: 20, lg: 24 };
const colors = {
  primary: '#3D3929',
  secondary: '#5C5344',
  accent: '#C45A3B',
  muted: '#968C7D',
};

export function Icon({ icon: IconComponent, size = 'md', color = 'primary' }: IconProps) {
  return <IconComponent size={sizes[size]} color={colors[color]} strokeWidth={1.5} />;
}
```

---

## Micro-interactions

### Press States
```
Default → Press: opacity 0.9, scale 0.98
Duration: 100ms
Easing: ease-out
```

### Page Transitions
```
Slide from right: translateX(100%) → translateX(0)
Duration: 250ms
Easing: cubic-bezier(0.25, 0.1, 0.25, 1)
```

### Loading States
```
Skeleton: cream-200 → cream-300 pulse
Duration: 1.5s
```

---

## Dark Mode (Optional)

```javascript
// Dark mode overrides
colors: {
  cream: {
    50:  '#1C1A17',  // Inverted
    100: '#242220',
    200: '#2E2B28',
    300: '#3A3632',
    400: '#4A4540',
  },
  ink: {
    900: '#FAF8F5',  // Inverted
    800: '#EBE4D8',
    700: '#DDD4C4',
    600: '#B5ADA0',
    500: '#968C7D',
    400: '#7A7162',
  },
}
```

---

## Usage Examples

### Typical Screen

```tsx
export default function ProfileScreen() {
  return (
    <ScreenContainer>
      {/* Header */}
      <View className="pt-6 pb-4">
        <Text variant="h1">Profil</Text>
        <Text variant="body-sm" color="secondary">
          Hesap ayarlarını buradan yönetebilirsin
        </Text>
      </View>
      
      <Divider />
      
      {/* User Info Card */}
      <Card variant="elevated" className="mb-4">
        <View className="flex-row items-center">
          <View className="w-16 h-16 rounded-full bg-cream-300 mr-4" />
          <View className="flex-1">
            <Text variant="h4">Abdullah</Text>
            <Text variant="body-sm" color="muted">@abdullah</Text>
          </View>
        </View>
      </Card>
      
      {/* Quote */}
      <Quote author="Unknown">
        Yol, yürümekle oluşur.
      </Quote>
      
      {/* Settings Section */}
      <Section title="Ayarlar" subtitle="Tercihlerini düzenle">
        <Card variant="outlined" padding="none">
          <SettingsRow icon={Bell} label="Bildirimler" />
          <SettingsRow icon={Moon} label="Karanlık Mod" />
          <SettingsRow icon={Globe} label="Dil" value="Türkçe" />
        </Card>
      </Section>
      
      {/* Actions */}
      <View className="mt-6 gap-3">
        <Button variant="primary">Kaydet</Button>
        <Button variant="ghost">Çıkış Yap</Button>
      </View>
    </ScreenContainer>
  );
}
```

### List Item Pattern

```tsx
function ListItem({ title, subtitle, rightText, onPress }) {
  return (
    <Pressable 
      onPress={onPress}
      className="flex-row items-center py-3 px-4 active:bg-cream-200"
    >
      <View className="flex-1">
        <Text variant="body">{title}</Text>
        {subtitle && (
          <Text variant="body-sm" color="muted">{subtitle}</Text>
        )}
      </View>
      {rightText && (
        <Text variant="body-sm" color="secondary">{rightText}</Text>
      )}
      <Icon icon={ChevronRight} size="sm" color="muted" />
    </Pressable>
  );
}
```

---

## Do's and Don'ts

### ✅ Do
- Serif fontları sadece başlıklar ve alıntılar için kullan
- İnce border'lar (0.5-1px) tercih et
- Bol whitespace bırak
- İtalik'i vurgu için kullan, abartma
- Terracotta'yı accent olarak seyrek kullan
- Cream tonları arasında kontrast oluştur

### ❌ Don't
- Parlak, doygun renkler kullanma
- Drop shadow'ları abartma
- Her yere icon koyma
- Gradient kullanma (flat tut)
- Rounded-full her yerde kullanma (seçici ol)
- Uppercase'i body text'te kullanma

---

## Font Loading (Expo)

```tsx
// App.tsx veya _layout.tsx
import { useFonts } from 'expo-font';
import {
  Lora_400Regular,
  Lora_400Regular_Italic,
  Lora_600SemiBold,
  Lora_700Bold,
} from '@expo-google-fonts/lora';
import {
  Inter_400Regular,
  Inter_500Medium,
  Inter_600SemiBold,
} from '@expo-google-fonts/inter';

export default function App() {
  const [fontsLoaded] = useFonts({
    Lora_400Regular,
    Lora_400Regular_Italic,
    Lora_600SemiBold,
    Lora_700Bold,
    Inter_400Regular,
    Inter_500Medium,
    Inter_600SemiBold,
  });

  if (!fontsLoaded) {
    return <SplashScreen />;
  }

  return <RootNavigator />;
}
```

---

**Design System Version**: 1.0  
**Last Updated**: 2026-01-03  
**Author**: Abdullah's Vinyl Editorial

