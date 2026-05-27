# KoloShop - Mobile UX & Frontend Optimization

## Recommandations UX Mobile Afrique Centrale

### 1. Optimisation pour Contexte Africain

#### Connectivité Intermittente
```typescript
// Offline-first architecture
import WatermelonDB from "@nozbe/watermelondb";
import sqlite from "@nozbe/watermelondb/adapters/sqlite";

// Local-first data management
const database = new Database({
  adapter: new SQLiteAdapter({
    dbName: "koloshop",
    schema: appSchema
  })
});

// Automatic sync when online
useEffect(() => {
  const handleOnline = async () => {
    await syncWithServer();
  };
  window.addEventListener("online", handleOnline);
  return () => window.removeEventListener("online", handleOnline);
}, []);
```

#### Low Bandwidth
```typescript
// Image optimization for slow networks
import Image from "next/image";

<Image
  src="/product.jpg"
  width={300}
  height={300}
  quality={60}  // Reduced quality for slow connections
  placeholder="blur"  // LQIP (Low Quality Image Placeholder)
  blurDataURL={generateBlur()} // Ultra-light placeholder
  priority={false}  // Lazy load
  sizes="(max-width: 640px) 100vw, 50vw"
  responsive
/>;
```

#### Phone Model Diversity
- Support "small" phones (4.5"-5.5" screens)
- Optimize for 2GB+ RAM devices
- Support Android 9+ (majority of users)
- Test on real devices (not just emulators)

### 2. Payment Flow Mobile-First

```typescript
// Mobile Money UX Pattern
const MobileMoneyCheckout = () => {
  const [phone, setPhone] = useState("");
  const [otp, setOTP] = useState("");
  const [step, setStep] = useState<"phone" | "otp" | "confirming">("phone");

  const handlePhoneSubmit = async () => {
    // 1. Validate phone
    if (!isValidPhoneNumber(phone)) {
      return toast("Invalid phone number");
    }

    // 2. Initiate payment
    setStep("confirming");
    const paymentId = await initiatePayment({
      orderId,
      phone,
      provider: "moov"  // Auto-detect from phone
    });

    // 3. Show instruction: "Check your phone for USSD prompt"
    toast.info("Check your phone for USSD prompt", { duration: 0 });

    // 4. Start polling for completion (not blocking)
    pollPaymentStatus(paymentId);

    setStep("otp");
  };

  return (
    <div className="space-y-4">
      {step === "phone" && (
        <>
          <InputPhone
            value={phone}
            onChange={setPhone}
            placeholder="+237 6XX XXX XXX"
            autoFocus
          />
          <Button onClick={handlePhoneSubmit} fullWidth>
            Proceed
          </Button>
        </>
      )}

      {step === "otp" && (
        <>
          <Alert>
            <Info /> Check your phone for payment prompt
          </Alert>
          <Text size="sm" color="muted">
            Waiting for payment confirmation...
          </Text>
          <LoadingSpinner />
          <Button onClick={() => setStep("phone")} variant="ghost" fullWidth>
            Try different number
          </Button>
        </>
      )}
    </div>
  );
};
```

### 3. Navigation Optimized for Fat Fingers

```typescript
// Touch-friendly spacing & sizing
const TouchSizeGuide = {
  MINIMUM_TAP_TARGET: 44, // px (44x44 minimum)
  COMFORTABLE_TAP: 48,    // px (48x48)
  BUTTON_PADDING: 16,     // px
  SPACING_VERTICAL: 8,    // px minimum
  ICON_SIZE: 24           // px
};

// Example button
<button
  className="w-12 h-12 p-4 rounded-lg bg-blue-600 text-white"
  // Minimum 48x48 tap target
>
  <Icon size={24} />
  <span className="ml-2">Add to Cart</span>
</button>
```

### 4. Language/Currency Localization

```typescript
// Francophone-first + multi-language
const i18nConfig = {
  defaultLanguage: "fr",
  supportedLanguages: ["fr", "en", "pt"],
  
  fallbackLanguage: "fr",
  
  countryDefaults: {
    CM: { currency: "XAF", language: "fr", timezone: "Africa/Douala" },
    CG: { currency: "XAF", language: "fr", timezone: "Africa/Brazzaville" },
    TD: { currency: "XAF", language: "fr", timezone: "Africa/Ndjamena" },
    GQ: { currency: "XAF", language: "es/pt", timezone: "Africa/Malabo" },
    GA: { currency: "XAF", language: "fr", timezone: "Africa/Libreville" }
  }
};

// Currency formatting
const formatPrice = (price: number, locale: string) => {
  return new Intl.NumberFormat(locale, {
    style: "currency",
    currency: "XAF",
    minimumFractionDigits: 0
  }).format(price);
};

// Usage
<Price 
  amount={100000} 
  locale="fr-CM" 
  // Displays: "100 000 XAF FCFA"
/>
```

### 5. Light Mode (Critical for Low Bandwidth)

```typescript
// Dark/Light mode toggle
const useTheme = () => {
  const [theme, setTheme] = useState("light"); // Default light
  
  const css = theme === "light" 
    ? globalLightStyles  // Minimal CSS
    : globalDarkStyles;  // More CSS
  
  return { theme, setTheme };
};

/* Global light styles (minimal) */
body {
  background: #ffffff;  /* White uses less battery on older phones */
  color: #000000;
}

/* Light colors for faster rendering */
```

### 6. Accessibility for All Users

```typescript
// WCAG 2.1 AA compliance
const AccessibleProductCard = ({ product }) => {
  return (
    <article
      className="product-card"
      role="article"
      aria-label={`Product: ${product.name}`}
    >
      <h2 className="text-lg font-semibold">{product.name}</h2>
      
      <img
        src={product.image}
        alt={`${product.name} - ${product.seller}`}
        // Descriptive alt text, not "image" or "photo"
      />
      
      <Price
        amount={product.price}
        ariaLabel={`Price: ${formattedPrice(product.price, "fr-CM")}`}
      />

      <button
        aria-label={`Add ${product.name} to cart`}
        className="py-3 px-4 bg-blue-600"  // 48x44 min tap
      >
        Add to Cart
      </button>
    </article>
  );
};

// Keyboard navigation
<ProductList
  products={products}
  onKeyDown={(e) => {
    if (e.key === "Enter") handleSelect();
    if (e.key === "ArrowDown") moveToNext();
  }}
/>
```

### 7. Mobile Performance Audit

```bash
# Lighthouse performance targets
Metrics:
  - First Contentful Paint (FCP): < 1.5s
  - Largest Contentful Paint (LCP): < 2.5s
  - Cumulative Layout Shift (CLS): < 0.1
  - First Input Delay (FID): < 100ms
  
Performance score: >= 90

# Tools
- Google Lighthouse CI
- WebPageTest
- Mobile Chrome DevTools
```

#### Optimization Checklist
```yaml
Images:
  ✅ WebP format with fallback
  ✅ Responsive srcset
  ✅ Max 50KB for thumbnails
  ✅ LQIP for placeholders
  ✅ CDN delivery with compression

JavaScript:
  ✅ Code splitting by route
  ✅ Tree shake unused code
  ✅ Minified + gzipped
  ✅ Dynamic imports for heavy libs
  ✅ Remove console.log() in prod

CSS:
  ✅ Purge unused CSS (PurgeCSS)
  ✅ Minify
  ✅ Critical CSS inlined
  ✅ Defer non-critical
  ✅ Preload fonts

Delivery:
  ✅ CloudFlare CDN
  ✅ gzip + brotli enabled
  ✅ Cache headers optimized
  ✅ Prefers-reduced-motion respected
```

### 8. Features by Phase

#### Phase 1 (MVP on Mobile)
- ✅ Product browsing (offline capable)
- ✅ Search (client-side initially)
- ✅ Cart (local storage)
- ✅ Checkout (simple mobile money)
- ✅ Order tracking (polling)
- ✅ Ratings + reviews
- ✅ Basic profile

#### Phase 2 (Mobile Polish)
- ✅ Wishlist with sync
- ✅ Push notifications
- ✅ One-click checkout
- ✅ Saved addresses
- ✅ Order history (offline)
- ✅ Chat with seller

#### Phase 3 (Mobile +)
- ✅ AR product preview
- ✅ Voice search
- ✅ Biometric auth
- ✅ App shortcuts
- ✅ Widget support

### 9. Native App Strategy (React Native)

```typescript
// Shared component library
// mobile/src/components/ProductCard.tsx

import React from "react";
import { View, Text, Image, TouchableOpacity } from "react-native";
import { Ionicons } from "@expo/vector-icons";

export const ProductCard: React.FC<ProductCardProps> = ({
  product,
  onPress,
  onFavorite
}) => (
  <TouchableOpacity
    onPress={onPress}
    className="bg-white rounded-lg overflow-hidden shadow-sm"
  >
    <Image
      source={{ uri: product.thumbnailUrl }}
      className="w-full h-48 bg-gray-100"
      resizeMode="cover"
    />

    <View className="p-3">
      <Text
        numberOfLines={2}
        className="font-semibold text-sm"
      >
        {product.name}
      </Text>

      <Text className="text-gray-600 text-xs mt-1">
        {product.seller.businessName}
      </Text>

      <View className="flex-row items-center justify-between mt-2">
        <Text className="font-bold text-lg">
          {formatPrice(product.basePrice, "fr-CM")}
        </Text>

        <TouchableOpacity
          onPress={() => onFavorite(product.id)}
          className="p-2"
        >
          <Ionicons
            name={product.isFavorited ? "heart" : "heart-outline"}
            size={20}
            color="red"
          />
        </TouchableOpacity>
      </View>

      {product.discountPercentage > 0 && (
        <View className="bg-red-100 px-2 py-1 rounded mt-2">
          <Text className="text-red-600 text-xs font-semibold">
            -{product.discountPercentage}%
          </Text>
        </View>
      )}
    </View>
  </TouchableOpacity>
);
```

### 10. Testing on Real African Networks

```bash
# Simulate slow networks
- Chrome DevTools throttling
- 2G: ~50 Kbps
- 3G: ~400 Kbps (Cameroon avg)
- 4G: ~1.5 Mbps

# Test devices
- Samsung Galaxy A12 (4GB RAM, Android 11)
- Infinix Smart series
- Tecno Spark series
- Real SIM cards (MTN, Orange, Moov)

# Metrics to monitor
- Time to interactive
- Bounce rate
- Error logs
- Battery consumption
```

---

## Frontend Architecture Détaillée

### Structure Next.js 14

#### App Router (moderne)
```
app/
├── (public)/
│   ├── page.tsx              # Homepage
│   ├── layout.tsx
│   ├── products/
│   │   ├── page.tsx          # Catalog
│   │   └── [id]/
│   │       └── page.tsx      # PDP
│   └── seller/[id]/
│       └── page.tsx          # Seller store
│
├── (auth)/
│   ├── login/page.tsx
│   ├── register/page.tsx
│   ├── forgot-password/page.tsx
│   └── verify-email/page.tsx
│
├── (dashboard)/             # Protected routes
│   ├── layout.tsx           # Auth guard
│   ├── page.tsx             # Redirect to role
│   ├── profile/page.tsx
│   ├── orders/page.tsx
│   ├── orders/[id]/page.tsx
│   ├── addresses/page.tsx
│   └── settings/page.tsx
│
├── (seller)/                # Seller routes
│   ├── layout.tsx
│   ├── page.tsx
│   ├── products/
│   │   ├── page.tsx
│   │   ├── new/page.tsx
│   │   └── [id]/edit/page.tsx
│   ├── orders/page.tsx
│   ├── analytics/page.tsx
│   └── payouts/page.tsx
│
├── api/                     # API routes
│   ├── auth/[...auth]/route.ts
│   ├── webhooks/moov/route.ts
│   └── health/route.ts
│
├── layout.tsx               # Root layout
└── not-found.tsx            # 404 page
```

#### State Management (Zustand)
```typescript
// store/authStore.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AuthStore {
  user: User | null;
  accessToken: string | null;
  login: (credentials) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,

      login: async (credentials) => {
        const response = await apiClient.post("/auth/login", credentials);
        set({
          user: response.data.user,
          accessToken: response.data.accessToken
        });
      },

      logout: () => {
        set({ user: null, accessToken: null });
	// Clear API client auth
      }
    }),
    {
      name: "auth-storage",
      partialize: (state) => ({ user: state.user })
      // Don't persist accessToken (security)
    }
  )
);
```

#### API Client (TanStack Query)
```typescript
// lib/queryClient.ts
import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5 minutes
      gcTime: 10 * 60 * 1000,        // 10 minutes (was cacheTime)
      retry: (failureCount, error) => {
        if (error.status === 401) return false;
        return failureCount < 3;
      }
    }
  }
});

// hooks/useProducts.ts
import { useInfiniteQuery } from "@tanstack/react-query";

export const useProducts = (filters) => {
  return useInfiniteQuery({
    queryKey: ["products", filters],
    queryFn: ({ pageParam = 1 }) =>
      apiClient.get("/products", { params: { page: pageParam, ...filters } }),
    getNextPageParam: (lastPage) =>
      lastPage.data.hasMore ? lastPage.data.nextPage : undefined
  });
};
```

---

## Deployment Guide

### Docker Image
```dockerfile
# Dockerfile.web
FROM node:22-alpine AS builder

WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

COPY . .
RUN pnpm run build

# Production image
FROM node:22-alpine

WORKDIR /app
RUN npm install -g pnpm
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

EXPOSE 3000
ENV NODE_ENV=production
CMD ["pnpm", "start"]
```

### Nginx Configuration
```nginx
# nginx.conf
upstream web {
  least_conn;
  server web:3000 max_fails=3 fail_timeout=30s;
}

server {
  listen 80;
  server_name koloshop.cm www.koloshop.cm;
  
  # Redirect HTTP to HTTPS
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl http2;
  server_name koloshop.cm www.koloshop.cm;

  # SSL certificates
  ssl_certificate /etc/nginx/certs/cert.pem;
  ssl_certificate_key /etc/nginx/certs/key.pem;

  # Security headers
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
  add_header X-Frame-Options "SAMEORIGIN" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header X-XSS-Protection "1; mode=block" always;

  # Caching
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }

  # Next.js
  location / {
    proxy_pass http://web;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_buffering off;
  }
}
```

---

**Frontend & Mobile UX Document**  
Last Updated: May 27, 2026
