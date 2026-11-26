# Step-by-Step Guide: Assignment 07 & 08 — Rendering Strategies

---

## 1. Assignment ka Goal
Is assignment ka main objective Next.js App Router ke alag-alag rendering patterns ko sahi use-case ke hisab se implement karna hai:
- Product listing par **60-second time-based ISR**
- Product detail pages par **SSG + ISR (`generateStaticParams`)**
- Informational pages (`/contact`, `/terms`, `/about`) par **Static Rendering**
- User account page (`/account`) par **Dynamic Server Rendering**
- Search & filtering UI par **Client-Side Rendering (CSR)**

---

## 2. Zaroori Concepts (Recap)
- **Static Rendering (SSG - Static Site Generation)**: Build time par HTML generate hota hai. Best for marketing, terms, contact pages. Fast aur CDN-cacheable.
- **ISR (Incremental Static Regeneration)**: Build ke baad background me specific interval (e.g. 60s) par page revalidate hota hai bina poora project rebuild kiye.
- **Dynamic Rendering**: Har request par server page render karta hai. Ye tab trigger hota hai jab `cookies()`, `headers()`, ya uncached data fetch use ho.
- **Client-Side Rendering (CSR)**: React component browser me execute hota hai (`"use client"`), interactive filters aur search ke liye ideal.
- **`generateStaticParams`**: Dynamic routes (jaise `/products/[id]`) ke liye build time par specific IDs pre-render karne ke kaam aata hai.

> 📖 Course Notes Reference: Rendering deep dive ke liye is folder ki note file [07.rendering.md](./07.rendering.md) aur caching concepts ke liye [../08.caching_and_revalidation/08.caching_and_revalidation.md](../08.caching_and_revalidation/08.caching_and_revalidation.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Product Listing me 60s Time-based ISR configure karo
- **Kya karna hai:** `src/app/products/page.tsx` me route segment config `revalidate = 60` export karo ya fetch me `next: { revalidate: 60 }` pass karo.
- **Kyu karna hai:** Catalog page statically cached rahega, lekin har 60 seconds baad background me fresh data se regenerate ho jayega.
- **Code (`src/app/products/page.tsx`):**
```tsx
export const revalidate = 60; // Route-level ISR interval (seconds)

export default async function ProductsCatalogPage() {
  const res = await fetch("http://localhost:4000/products", {
    next: { revalidate: 60 },
  });
  const data = await res.json();
  const renderedAt = new Date().toLocaleTimeString();

  return (
    <div className="p-8">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Catalog (ISR 60s)</h1>
        <span className="text-xs font-mono bg-amber-100 text-amber-800 px-3 py-1 rounded-full">
          Generated at: {renderedAt}
        </span>
      </div>
      {/* Product list cards */}
    </div>
  );
}
```
- **Expected result:** 60 seconds ke andar refresh karne par "Generated at" timestamp same rahega, 60s ke baad next visit par background me update ho jayega.

---

### Step 2: Product Detail route par SSG + ISR (`generateStaticParams`) implement karo
- **Kya karna hai:** `src/app/products/[id]/page.tsx` me `generateStaticParams()` function export karo aur selected top product IDs return karo.
- **Kyu karna hai:** Important product pages build time par hi pre-render ho jayein taaki first user ko zero-latency response mile.
- **Code (`src/app/products/[id]/page.tsx`):**
```tsx
export const revalidate = 60; // Periodic regeneration

// Build time par pre-generate hone waali product IDs
export async function generateStaticParams() {
  // Top 5 popular products pre-render karo
  return [
    { id: "1" },
    { id: "2" },
    { id: "3" },
    { id: "4" },
    { id: "5" },
  ];
}

export default async function ProductDetailPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const res = await fetch(`http://localhost:4000/products/${id}`, {
    next: { revalidate: 60 },
  });
  const product = await res.json();

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold">{product.name}</h1>
      <p className="text-blue-600 font-bold text-xl mt-2">${product.price}</p>
    </div>
  );
}
```
- **Expected result:** Build time par ID 1 se 5 ke HTML files generate honge, aur baki IDs on-demand generate hokar cache ho jayengi.

---

### Step 3: Pure Static Pages verify karo (`/contact`, `/terms`)
- **Kya karna hai:** `src/app/contact/page.tsx` aur `src/app/terms/page.tsx` me check karo ki koi request-specific API (`cookies()`, `headers()`) call na ho.
- **Kyu karna hai:** In pages ka data sabhi users ke liye static rehta hai, isliye ye build time par completely static HTML (`○ Static`) banenge.
- **Code (`src/app/contact/page.tsx`):**
```tsx
export default function ContactPage() {
  return (
    <div className="p-8 max-w-xl mx-auto">
      <h1 className="text-3xl font-bold">Contact Customer Support</h1>
      <p className="mt-4 text-slate-600">Email: help@nextstore.com | Phone: +1-800-555-0199</p>
    </div>
  );
}
```
- **Expected result:** Next.js build output me ye routes `○ (Static)` mark honge.

---

### Step 4: Dynamic Rendering Page (`/account`) create karo
- **Kya karna hai:** `src/app/account/page.tsx` create karo jo `cookies()` ya `headers()` call kare.
- **Kyu karna hai:** User account page personalized hota hai, isliye iska build time par static banna galat hai; ye har incoming request par dynamically execute hona chahiye.
- **Code (`src/app/account/page.tsx`):**
```tsx
import { cookies } from "next/headers";

export default async function AccountPage() {
  const cookieStore = await cookies();
  const sessionToken = cookieStore.get("session_token")?.value;
  const requestTime = new Date().toLocaleTimeString();

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold">User Account (Dynamic Rendering)</h1>
      <div className="mt-4 p-4 bg-slate-100 rounded-lg text-sm space-y-2">
        <p><strong>Status:</strong> {sessionToken ? "Logged In" : "Guest Viewer"}</p>
        <p><strong>Rendered at (Per Request):</strong> {requestTime}</p>
      </div>
    </div>
  );
}
```
- **Expected result:** Har bar browser refresh karne par "Rendered at" time immediately change hoga.

---

### Step 5: Client-Side Interactive Filter Component banao
- **Kya karna hai:** `src/components/ProductClientFilter.tsx` banao `'use client'` ke sath jo initial server products ko client state me search/filter kare.
- **Kyu karna hai:** Instant typing response (zero network latency) dene ke liye bina poore page ko CSR banaye.
- **Code (`src/components/ProductClientFilter.tsx`):**
```tsx
"use client";

import { useState } from "react";
import Link from "next/link";

interface ProductItem {
  id: string | number;
  name: string;
  price: number;
}

export default function ProductClientFilter({ initialProducts }: { initialProducts: ProductItem[] }) {
  const [searchTerm, setSearchTerm] = useState("");

  const filtered = initialProducts.filter((p) =>
    p.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div>
      <input
        type="text"
        placeholder="Search products live..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        className="w-full max-w-md px-4 py-2 border border-slate-300 rounded-lg mb-6 focus:ring-2 focus:ring-blue-500"
      />
      <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
        {filtered.map((item) => (
          <Link
            key={item.id}
            href={`/products/${item.id}`}
            className="p-4 border rounded-lg bg-white shadow-sm hover:shadow"
          >
            <h3 className="font-semibold">{item.name}</h3>
            <p className="text-blue-600 font-bold mt-1">${item.price}</p>
          </Link>
        ))}
      </div>
    </div>
  );
}
```
- **Expected result:** Search box me type karte hi products live filter honge.

---

### Step 6: Production Build karke Rendering Strategy verify karo
- **Kya karna hai:** Terminal me build command chalao aur routes ka status table inspect karo.
- **Kyu karna hai:** Next.js build output me clearly dikhata hai ki kaunsa page Static hai, kaunsa SSG/ISR hai, aur kaunsa Dynamic hai.
- **Code:**
```bash
npm run build
```
- **Expected result (Terminal Build Output):**
```text
Route (app)                              Size     First Load JS
┌ ○ /                                    ... kB   ... kB
├ ○ /contact                             ... kB   ... kB
├ ○ /terms                               ... kB   ... kB
├ ƒ /account                             ... kB   ... kB  (Dynamic)
├ ● /products                            ... kB   ... kB  (ISR: 60s)
└ ● /products/[id]                       ... kB   ... kB  (SSG / ISR)
  ├ /products/1
  ├ /products/2
  └ ...
○  (Static)   prerendered as static content
●  (SSG)      prerendered as static HTML (uses generateStaticParams)
ƒ  (Dynamic)  server-rendered on demand
```

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── account/
│   │   └── page.tsx            # Dynamic Rendering (cookies())
│   ├── contact/
│   │   └── page.tsx            # Static Rendering
│   ├── products/
│   │   ├── [id]/
│   │   │   └── page.tsx        # SSG + ISR (generateStaticParams, revalidate: 60)
│   │   └── page.tsx            # ISR (revalidate = 60)
│   ├── terms/
│   │   └── page.tsx            # Static Rendering
│   └── page.tsx                # Static Homepage
└── components/
    └── ProductClientFilter.tsx # Client-Side Rendering ("use client")
```

---

## 5. Common Mistakes / Errors

1. **`revalidate` set karne ke baad dev server me check karna:**
   - *Galti:* `npm run dev` me check karna aur sochna ki 60 seconds tak cache kyu nahi ho raha.
   - *Fix:* Development mode me Next.js hamesha har request par render karta hai. Rendering aur caching behavior test karne ke liye hamesha `npm run build && npm run start` run karein.

2. **Static page me `cookies()` ya `headers()` call karna:**
   - *Galti:* `/contact` page me `headers()` call kar lena, jisse wo unintentionally Dynamic route ban jata hai.
   - *Fix:* Static pages ko request-specific APIs se isolated rakhein.

3. **`generateStaticParams` me hazaron items return karna:**
   - *Galti:* 10,000 products ko build time par render karne ki koshish karna jisse build hours tak chalti rahe.
   - *Fix:* Sirf top / featured products (e.g. 5-20) return karo; baaki on-demand generate ho kar automatically cache ho jayenge.

4. **On-demand APIs (`revalidatePath`) use karna jabki assignment me prohibited hai:**
   - *Galti:* Is assignment me time-based ISR manga gaya hai; `revalidatePath` ya `revalidateTag` use karna restrictions ke khilaf hai.
   - *Fix:* Sirf `revalidate = 60` time-based strategy use karein.

---

## 6. Verification Checklist
- [ ] `npm run build` output me `/contact` aur `/terms` `○ (Static)` dikh rahe hain.
- [ ] `npm run build` output me `/account` `ƒ (Dynamic)` dikh raha hai.
- [ ] `/products` aur `/products/[id]` `● (SSG / ISR)` mark hue hain.
- [ ] `generateStaticParams` top IDs return kar raha hai.
- [ ] `npm run start` chalane par 60s ke andar `/products` page reload karne par timestamp freeze rehta hai aur 60s baad background me revalidate hota hai.
- [ ] Search input me type karne par client side filter smoothly operate karta hai.

---

## 7. Bonus (Extra Practice)
1. **Dynamic Segment Fallback Mode:** `export const dynamicParams = true` (default) vs `false` configure karke dekho ki agar user non-generated ID enter kare to on-demand ISR generate hota hai ya 404.
2. **Revalidation Countdown UI:** Client component me ek chhota countdown timer show karo jo bataye "Data refreshes in X seconds".
