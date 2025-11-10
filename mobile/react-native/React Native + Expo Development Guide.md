# React Native + Expo Development Guide
> Practical guide for real-world mobile app development with Expo SDK 53+, React Native 0.80+, TypeScript

## Project Structure

### Core Principles
- **Start simple**: Context API first, add Zustand only if Context causes performance issues
- **Platform files**: Create `.ios.tsx`/`.android.tsx` only when platforms actually differ
- **Type everything**: Enable TypeScript strict mode, no `any` types
- **Co-locate code**: Keep components, hooks, and types close together
- **New Architecture**: Now default in SDK 53+, mandatory in SDK 55+

### Common Commands

```bash
# Daily Development
npx expo start
npx expo start --clear  # Clear cache when things break
npx expo start --ios
npx expo start --android

# Type check
npx tsc --noEmit

# Build & Deploy
eas build --profile development --platform ios
eas build --profile production --platform all
eas submit --platform ios
eas update --branch production

# Testing Deep Links
# iOS Simulator
xcrun simctl openurl booted rota://venue/123

# Android Emulator
adb shell am start -W -a android.intent.action.VIEW -d "rota://venue/123"

# Troubleshooting
npx expo start --clear
npx expo prebuild --clean
cd android && ./gradlew clean && cd ..
```

## Directory Organization
```
app/
├── _layout.tsx              # Root layout
├── index.tsx                # Landing screen
├── (tabs)/                  # Tab-based routes
│   ├── _layout.tsx
│   ├── home.tsx
│   └── profile.tsx
└── venue/[id].tsx           # Dynamic routes

components/
├── VenueCard.tsx
├── MapView.ios.tsx          # Only when iOS differs
└── MapView.android.tsx

hooks/
├── useApi.ts
├── useAuth.ts
└── useLocation.ts

services/
├── api.ts
├── storage.ts
└── location.ts

types/
├── api.ts
├── models.ts
└── navigation.ts

constants/
└── config.ts

assets/
├── fonts/
└── images/
```

## New Architecture (2025)

The New Architecture is now **default in SDK 53+** and will be **mandatory in SDK 55**.

**What you need to know:**
- All expo-* packages support it
- ~75% of SDK 53+ projects use it
- Legacy Architecture will be removed in late 2025
- Most popular libraries now support it (Stripe, react-native-maps)

**Check if enabled:**
```json
// app.json
{
  "expo": {
    "newArchEnabled": true  // Default in SDK 53+
  }
}
```

If you're on SDK 53+, you're already using it. No action needed.

## TypeScript Rules

### Always Do
- Use explicit return types for functions
- Define interfaces for component props
- Enable strict mode in tsconfig.json
- Use `unknown` instead of `any` when type is unclear

### Examples

```typescript
// ✅ Good: Explicit types
async function fetchVenues(city: string): Promise<Venue[]> {
  const response = await api.get<Venue[]>(`/venues?city=${city}`);
  return response.data;
}

// ✅ Good: Interface for props
interface VenueCardProps {
  venue: Venue;
  onPress: (id: string) => void;
}

export function VenueCard({ venue, onPress }: VenueCardProps) {
  return <Pressable onPress={() => onPress(venue.id)} />;
}

// ❌ Bad: Using 'any'
function processData(data: any) { }

// ✅ Good: Use 'unknown'
function processData(data: unknown) {
  if (typeof data === 'string') {
    // TypeScript knows data is string here
  }
}
```

## Platform-specific Development

Create `.ios.tsx`/`.android.tsx` only when actually needed:
- Completely different UI
- Platform-specific APIs (camera, maps, biometrics)
- Performance optimization for one platform

```typescript
// components/MapView.ios.tsx
import MapView from 'react-native-maps';

export function VenueMap({ venues }: Props) {
  return <MapView provider="apple" />;
}

// For small differences, use Platform.select()
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    padding: Platform.select({ ios: 16, android: 12 }),
  },
});
```

## State Management

### Decision Tree
1. **Is state used in only one component?** → Use `useState`
2. **Is state shared between 2-3 screens?** → Use Context API
3. **Is Context causing performance issues?** → Use Zustand
4. **Large enterprise app with existing Redux?** → Keep Redux, but prefer simpler solutions for new features

### Key Rule
Context re-renders all consumers when any value changes. If this becomes a problem, split into multiple contexts or switch to Zustand.

### 1. Local State (90% of cases)
```typescript
function VenueList() {
  const [search, setSearch] = useState('');
  const [filter, setFilter] = useState<'all' | 'open'>('all');
  
  return <View>{/* ... */}</View>;
}
```

### 2. Context API (shared state across screens)
```typescript
// contexts/AuthContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const response = await api.post('/auth/login', { email, password });
    await SecureStore.setItemAsync('auth_token', response.data.token);
    setUser(response.data.user);
  };

  const logout = async () => {
    await SecureStore.deleteItemAsync('auth_token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

### 3. Zustand (when Context causes performance issues)
```typescript
// stores/useAppStore.ts
import { create } from 'zustand';

interface AppStore {
  venues: Venue[];
  setVenues: (venues: Venue[]) => void;
}

export const useAppStore = create<AppStore>((set) => ({
  venues: [],
  setVenues: (venues) => set({ venues }),
}));

// Usage
function VenueList() {
  const venues = useAppStore((state) => state.venues);
  return <FlatList data={venues} />;
}
```

**When to use what:**
- **useState**: Component-only data
- **Context**: Auth, theme, language (2-3 contexts max)
- **Zustand**: When Context causes performance issues
- **Redux**: Only for large enterprise apps or existing codebases

## API Integration (Laravel Backend)

### Simple API Client
```typescript
// services/api.ts
import Constants from 'expo-constants';
import * as SecureStore from 'expo-secure-store';

const API_URL = Constants.expoConfig?.extra?.apiUrl ?? 'http://localhost:8000/api';

async function getAuthToken(): Promise<string | null> {
  return await SecureStore.getItemAsync('auth_token');
}

class ApiError extends Error {
  constructor(
    public status: number,
    message: string,
    public errors?: Record<string, string[]>
  ) {
    super(message);
  }
}

async function fetchApi<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
  const token = await getAuthToken();
  
  const response = await fetch(`${API_URL}${endpoint}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      ...(token ? { 'Authorization': `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });

  const data = await response.json();

  if (!response.ok) {
    throw new ApiError(response.status, data.message || 'Request failed', data.errors);
  }

  return data.data;
}

export const api = {
  get: <T>(endpoint: string) => fetchApi<T>(endpoint),
  post: <T>(endpoint: string, body: unknown) =>
    fetchApi<T>(endpoint, { method: 'POST', body: JSON.stringify(body) }),
  put: <T>(endpoint: string, body: unknown) =>
    fetchApi<T>(endpoint, { method: 'PUT', body: JSON.stringify(body) }),
  delete: <T>(endpoint: string) =>
    fetchApi<T>(endpoint, { method: 'DELETE' }),
};
```

### Data Fetching Hook
```typescript
// hooks/useApi.ts
import { useState, useEffect } from 'react';

export function useApi<T>(endpoint: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      try {
        setLoading(true);
        setError(null);
        const result = await api.get<T>(endpoint);
        if (!cancelled) setData(result);
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err.message : 'Unknown error');
        }
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetchData();
    return () => { cancelled = true; };
  }, [endpoint]);

  const refetch = async () => {
    setLoading(true);
    try {
      const result = await api.get<T>(endpoint);
      setData(result);
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  };

  return { data, loading, error, refetch };
}
```

## Data Storage

### Rules
- **SecureStore**: Auth tokens, API keys, sensitive data only
- **AsyncStorage**: Theme, language, settings, cache
- **Never**: Store passwords in AsyncStorage
- **Never**: Store large data in SecureStore (2KB limit)

### SecureStore
```typescript
// services/storage.ts
import * as SecureStore from 'expo-secure-store';

export const secureStorage = {
  async setToken(token: string) {
    await SecureStore.setItemAsync('auth_token', token);
  },
  
  async getToken() {
    return await SecureStore.getItemAsync('auth_token');
  },
  
  async deleteToken() {
    await SecureStore.deleteItemAsync('auth_token');
  },
};
```

### AsyncStorage
```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

export const storage = {
  async setItem(key: string, value: string) {
    await AsyncStorage.setItem(key, value);
  },
  
  async getItem(key: string) {
    return await AsyncStorage.getItem(key);
  },
  
  async setObject<T>(key: string, value: T) {
    await AsyncStorage.setItem(key, JSON.stringify(value));
  },
  
  async getObject<T>(key: string): Promise<T | null> {
    const item = await AsyncStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  },
};
```

## Keyboard Handling

Use `react-native-keyboard-controller` instead of the built-in `KeyboardAvoidingView`.

```bash
npx expo install react-native-keyboard-controller
```

```typescript
// app/_layout.tsx
import { KeyboardProvider } from 'react-native-keyboard-controller';

export default function RootLayout() {
  return (
    <KeyboardProvider>
      <Stack />
    </KeyboardProvider>
  );
}

// In screens
import { KeyboardAwareScrollView } from 'react-native-keyboard-controller';

function LoginScreen() {
  return (
    <KeyboardAwareScrollView>
      <TextInput placeholder="Email" />
      <TextInput placeholder="Password" secureTextEntry />
      <Button title="Login" />
    </KeyboardAwareScrollView>
  );
}
```

## Navigation & Deep Linking

Expo Router is now the standard (comes with `create-expo-app`). File-based routing with automatic deep linking.

```typescript
// app.json
{
  "expo": {
    "scheme": "rota"
  }
}

// app/venue/[id].tsx
import { useLocalSearchParams } from 'expo-router';

export default function VenueScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const { data: venue } = useApi<Venue>(`/venues/${id}`);
  
  return <Text>{venue?.name}</Text>;
}

// Navigate programmatically
import { router } from 'expo-router';

router.push('/venue/123');
router.replace('/home');
router.back();
```

### Universal Links (iOS/Android)
```json
// app.json
{
  "expo": {
    "ios": {
      "associatedDomains": ["applinks:rota.tips"]
    },
    "android": {
      "intentFilters": [
        {
          "action": "VIEW",
          "autoVerify": true,
          "data": [
            {
              "scheme": "https",
              "host": "rota.tips"
            }
          ],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ]
    }
  }
}
```

Host `/.well-known/apple-app-site-association` and `/.well-known/assetlinks.json` on your domain.

## Styling

```typescript
import { StyleSheet, Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#FFFFFF',
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    marginBottom: 8,
  },
  button: {
    paddingVertical: Platform.OS === 'ios' ? 12 : 10,
  },
});

// Use constants
// constants/theme.ts
export const Colors = {
  primary: '#FF6B6B',
  background: '#FFFFFF',
  text: '#000000',
};

export const Spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
};
```

## Performance

```typescript
// Memoize components
const VenueCard = React.memo(({ venue }: Props) => {
  return <View>{/* ... */}</View>;
});

// Memoize callbacks
const handlePress = useCallback((id: string) => {
  router.push(`/venue/${id}`);
}, []);

// Memoize computed values
const sortedVenues = useMemo(() => {
  return venues.sort((a, b) => a.name.localeCompare(b.name));
}, [venues]);

// FlatList optimization
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  removeClippedSubviews
  maxToRenderPerBatch={10}
  windowSize={5}
  initialNumToRender={10}
/>

// Use expo-image instead of Image
import { Image } from 'expo-image';

<Image
  source={{ uri: venue.imageUrl }}
  style={{ width: '100%', height: 200 }}
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
/>
```

## Error Handling

```typescript
// API errors
try {
  await api.post('/venues', venueData);
} catch (error) {
  if (error instanceof ApiError) {
    if (error.status === 422) {
      console.log(error.errors); // { email: ['Email is required'] }
    } else if (error.status === 401) {
      router.replace('/login');
    }
  }
}

// Global error boundary
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error }: { error: Error }) {
  return (
    <View>
      <Text>Something went wrong</Text>
      <Text>{error.message}</Text>
    </View>
  );
}

export default function RootLayout() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <Stack />
    </ErrorBoundary>
  );
}
```

## Environment Variables

```typescript
// app.config.ts
export default {
  expo: {
    extra: {
      apiUrl: process.env.API_URL,
      googleMapsApiKey: process.env.GOOGLE_MAPS_API_KEY,
    },
  },
};

// .env
API_URL=http://localhost:8000/api
GOOGLE_MAPS_API_KEY=your_key_here

// Access in code
import Constants from 'expo-constants';

const API_URL = Constants.expoConfig?.extra?.apiUrl;
```

## Package Management

```bash
# npm works normally
npm install

# pip: ALWAYS use --break-system-packages
pip install pandas --break-system-packages
```

## Resources
- [Expo Documentation](https://docs.expo.dev/)
- [React Native Documentation](https://reactnative.dev/)
- [Expo Router Documentation](https://docs.expo.dev/router/introduction/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
