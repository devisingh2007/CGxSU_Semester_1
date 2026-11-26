# Step-by-Step Guide: Assignment 16 — Images, Fonts & Performance Optimization

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js ke built-in performance optimization tools ko implement karna hai:
- Standard `<img>` tags ko Next.js ke optimized `<Image>` component se replace karna (`alt`, `fill`, `sizes`, `priority`)
- External image domains ke liye `next.config.ts` me `remotePatterns` configure karna
- Google Fonts ko zero-layout-shift ke sath configure karna using `next/font/google` in root layout
- Heavy components ko dynamically import karna using `next/dynamic` with loading skeleton fallback.

---

## 2. Zaroori Concepts (Recap)
- **`next/image` Optimization**: Automatic modern WebP/AVIF format conversion, responsive image generation (srcset), lazy loading by default, aur layout shift (CLS) prevention.
- **`priority` Attribute**: Page ke sabse important hero / above-the-fold image par `priority` flag lagane se LCP (Largest Contentful Paint) score optimize hota hai.
- **`fill` & `sizes`**: Dynamic responsive containers ke liye jaha image ki fixed width/height pehle se maloom na ho. `sizes` browser ko batata hai ki alag-alag screen sizes par kitna viewport width expect karna hai.
- **`next/font`**: Fonts ko build time par download karke static assets me host karta hai; browser se external Google servers ko network request nahi jaati, jisse zero layout shift aur better privacy milti hai.
- **`next/dynamic`**: Client components ko code-split karke on-demand load karta hai taaki initial JavaScript bundle chhota rahe.

> 📖 Course Notes Reference: Web performance optimization ke liye is folder ki note file [16.images_fonts_performance.md](./16.images_fonts_performance.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: External CDN domains configure karo (`next.config.ts`)
- **Kya karna hai:** `next.config.ts` me `images.remotePatterns` array add karo (e.g. Unsplash, Cloudinary, etc.).
- **Kyu karna hai:** Security reason se Next.js unknown external image domains ko optimize karne se block karta hai jab tak explicitly allow na kiya jaye.
- **Code (`next.config.ts`):**
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.unsplash.com",
      },
      {
        protocol: "https",
        hostname: "via.placeholder.com",
      },
    ],
  },
};

export default nextConfig;
```
- **Expected result:** External CDN images bina error ke optimize hokar load hongi.

---

### Step 2: Next.js Google Font configure karo (`src/app/layout.tsx`)
- **Kya karna hai:** `next/font/google` se Inter ya Poppins font import karke `<body>` par className apply karo.
- **Kyu karna hai:** Font loading delay aur Flash of Unstyled Text (FOUT) khatam karne ke liye.
- **Code (`src/app/layout.tsx`):**
```tsx
import { Inter, Plus_Jakarta_Sans } from "next/font/google";
import "./globals.css";

// Font configuration
const inter = Inter({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-inter",
});

const fontJakarta = Plus_Jakarta_Sans({
  subsets: ["latin"],
  weight: ["400", "600", "700"],
  display: "swap",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.variable}>
      <body className={`${fontJakarta.className} antialiased bg-slate-50 text-slate-900`}>
        {children}
      </body>
    </html>
  );
}
```
- **Expected result:** High-performance local-hosted font apply ho jayega without any layout shift.

---

### Step 3: Hero Banner me `priority` Image optimize karo (`src/components/HeroBanner.tsx`)
- **Kya karna hai:** Above-the-fold banner image ko `<Image priority ... />` ke sath render karo.
- **Kyu karna hai:** Browser ise pre-load karega taaki LCP score green (> 90) rahe.
- **Code (`src/components/HeroBanner.tsx`):**
```tsx
import Image from "next/image";

export default function HeroBanner() {
  return (
    <div className="relative w-full h-[400px] rounded-2xl overflow-hidden shadow-lg mb-10">
      <Image
        src="https://images.unsplash.com/photo-1441986300917-64674bd600d8"
        alt="NextStore Mega Summer Sale Collection"
        fill
        priority // Preloads above-the-fold hero image
        sizes="(max-width: 768px) 100vw, 1200px"
        className="object-cover"
      />
      <div className="absolute inset-0 bg-gradient-to-r from-black/70 to-transparent flex items-center p-10">
        <div className="text-white max-w-md">
          <h1 className="text-4xl font-extrabold">Exclusive New Arrivals</h1>
          <p className="mt-2 text-slate-200">Upgrade your lifestyle with modern essentials.</p>
        </div>
      </div>
    </div>
  );
}
```
- **Expected result:** Hero banner instant render hoga bina kisi image pop-in delay ke.

---

### Step 4: Responsive Product Cards me `fill` aur `sizes` use karo
- **Kya karna hai:** `src/components/ProductCard.tsx` me fixed dimension ke badle responsive container aur `fill` use karo.
- **Kyu karna hai:** Mobile par chhoti image aur desktop par high-resolution image serve karne ke liye data bandwidth save ho sake.
- **Code (`src/components/ProductCard.tsx`):**
```tsx
import Image from "next/image";

interface ProductCardProps {
  name: string;
  price: number;
  imageUrl: string;
}

export default function ProductCard({ name, price, imageUrl }: ProductCardProps) {
  return (
    <div className="border border-slate-200 rounded-xl overflow-hidden bg-white shadow-sm flex flex-col">
      {/* Responsive Image Aspect Ratio Box */}
      <div className="relative w-full h-48 bg-slate-100">
        <Image
          src={imageUrl}
          alt={name}
          fill
          sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
          className="object-cover hover:scale-105 transition-transform duration-300"
          loading="lazy" // Default behavior for below-the-fold
        />
      </div>
      <div className="p-4 flex-1 flex flex-col justify-between">
        <h3 className="font-semibold text-slate-800">{name}</h3>
        <p className="text-blue-600 font-bold text-lg mt-2">${price}</p>
      </div>
    </div>
  );
}
```
- **Expected result:** Responsive image cards render honge jo layout shift create nahi karenge.

---

### Step 5: Heavy Component ko `next/dynamic` se lazy load karo
- **Kya karna hai:** Heavy chart ya interactive reviews panel ko `dynamic()` ke sath import karo with skeleton fallback.
- **Kyu karna hai:** Initial JavaScript bundle size kam karne ke liye taaki page super-fast open ho.
- **Code (`src/app/products/[id]/page.tsx`):**
```tsx
import dynamic from "next/dynamic";

// Dynamically import heavy sales chart with loading skeleton
const HeavySalesChart = dynamic(() => import("@/components/HeavySalesChart"), {
  loading: () => (
    <div className="h-64 bg-slate-100 animate-pulse rounded-xl flex items-center justify-center text-slate-400">
      Loading Performance Analytics...
    </div>
  ),
  ssr: false, // Browser-only component if using canvas/window APIs
});

export default function ProductAnalyticsSection() {
  return (
    <div className="mt-12 border-t pt-8">
      <h2 className="text-2xl font-bold mb-4">Product Demand & Analytics</h2>
      <HeavySalesChart />
    </div>
  );
}
```
- **Expected result:** Heavy chart ka JavaScript alag chunk me code-split ho jayega aur sirf zaroorat padne par browser download karega.

---

### Step 6: Performance Audit verify karo (Lighthouse / Network tab)
- **Kya karna hai:** Browser DevTools me Network tab kholo aur `Img` filter select karo.
- **Kyu karna hai:** Confirm karna ki image format `webp` ya `avif` me convert ho raha hai.
- **Expected result:** Images compressed format me aayengi aur viewport me aane par hi load hongi.

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── layout.tsx              # next/font/google configured on <body>
│   └── products/
│       └── [id]/
│           └── page.tsx        # Uses next/dynamic for HeavySalesChart
├── components/
│   ├── HeavySalesChart.tsx     # Large client component (lazy loaded)
│   ├── HeroBanner.tsx          # High-priority Image banner
│   └── ProductCard.tsx         # Responsive <Image fill sizes="..." />
└── next.config.ts              # images.remotePatterns configured
```

---

## 5. Common Mistakes / Errors

1. **`fill` use karte waqt parent container me `relative` position na dena:**
   - *Galti:* `<div className="w-full h-48"><Image fill ... /></div>` likhna. Isse image poori screen par stretch ho jayegi.
   - *Fix:* Parent container me hamesha `relative` class zaroor add karein: `<div className="relative w-full h-48">`.

2. **Har image par `priority` flag laga dena:**
   - *Galti:* Page ke saare 50 product cards par `priority` likhna jisse bandwidth choke ho jati hai.
   - *Fix:* `priority` sirf 1 ya 2 above-the-fold hero images par use karein; baaki images ko default lazy loading rehne dein.

3. **External domain error `Invalid src prop`:**
   - *Galti:* Next.js error: `hostname "images.unsplash.com" is not configured under images in your next.config.js`.
   - *Fix:* `next.config.ts` ke `images.remotePatterns` me domain allow karein aur dev server restart karein.

4. **`sizes` attribute omit kar dena responsive images me:**
   - *Galti:* `<Image fill />` likhna bina `sizes` ke. Next.js warning dega aur browser desktop resolution download kar lega mobile par bhi.
   - *Fix:* Always sensible `sizes="(max-width: 768px) 100vw, 33vw"` provide karein.

---

## 6. Verification Checklist
- [ ] `next.config.ts` me external CDN hostnames configured hain.
- [ ] Saare product photos `<Image />` component use kar rahe hain with meaningful `alt` text.
- [ ] Hero image par `priority` attribute laga hua hai.
- [ ] Responsive card images `fill` aur `sizes` use kar rahi hain.
- [ ] Font optimization `next/font/google` ke through configure hai (layout shift check passed).
- [ ] Heavy component `next/dynamic` se lazy load ho raha hai aur skeleton fallback display karta hai.
- [ ] Network tab me images modern WebP format me serve ho rahi hain.

---

## 7. Bonus (Extra Practice)
1. **Blur Placeholder:** `<Image placeholder="blur" blurDataURL="..." />` add karo taaki image load hone se pehle smooth blurred shimmer effect dikhe.
2. **Dynamic Import on User Interaction:** Ek button banao "View Full Product Analytics" jo click hone par hi `import()` trigger kare using React state.
