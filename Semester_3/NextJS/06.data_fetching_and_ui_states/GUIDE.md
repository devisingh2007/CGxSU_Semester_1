# Step-by-Step Guide: Assignment 06 — Data Fetching & UI States

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router ke **Complete UI States** (Loading, Error, Not-Found, Suspense) ko implement karna hai. Iske sath hum ek lightweight **Express API server** banayenge jo `products.json` dataset serve karega, aur Next.js server se us API ko call karke gracefully loading states, skeleton fallbacks, retry buttons, aur not-found screens handle karenge.

---

## 2. Zaroori Concepts (Recap)
- **Server Data Fetching**: Next.js App Router me Server Components directly `async/await fetch()` kar sakte hain without `useEffect`.
- **Route-level `loading.tsx`**: Route transition ke dauran Next.js automatically React Suspense boundary trigger karta hai aur `loading.tsx` render karta hai.
- **Component-level `<Suspense>`**: Page ke kisi slow section (jaise related products ya reviews) ko wrap karke independent fallback dikhata hai taaki baki page turant load ho sake.
- **Error Boundary `error.tsx`**: Route me error aane par crash hone ke bajaye friendly error screen dikhata hai. Isme `'use client'` aur `reset()` function zaroori hota hai.
- **`notFound()` aur `not-found.tsx`**: Jab product ID exist na kare to `notFound()` function call karke custom 404 screen render ki jati hai.

> 📖 Course Notes Reference: Data fetching and streaming architecture ke liye is folder ki note file [06.data_fetching_and_ui_states.md](./06.data_fetching_and_ui_states.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Lightweight Express Server banao (`server/index.js`)
- **Kya karna hai:** Ek simple Express server create karo jo `/products` (with filters & pagination) aur `/products/:id` endpoints expose kare.
- **Kyu karna hai:** Real-world API simulation ke liye jisse Next.js data fetch karega.
- **Code (`server/index.js`):**
```javascript
const express = require("express");
const cors = require("cors");
const products = require("../products.json"); // Repository ke products.json ko load karo

const app = express();
app.use(cors());
app.use(express.json());

// 1. GET /products with query filters & pagination
app.get("/products", (req, res) => {
  const { category, subCategory, page = 1, limit = 10 } = req.query;
  let filtered = products;

  if (category) {
    filtered = filtered.filter((p) => p.category?.toLowerCase() === category.toLowerCase());
  }
  if (subCategory) {
    filtered = filtered.filter((p) => p.subCategory?.toLowerCase() === subCategory.toLowerCase());
  }

  const pageNum = parseInt(page);
  const limitNum = parseInt(limit);
  const startIndex = (pageNum - 1) * limitNum;
  const paginated = filtered.slice(startIndex, startIndex + limitNum);

  res.json({
    data: paginated,
    total: filtered.length,
    page: pageNum,
    totalPages: Math.ceil(filtered.length / limitNum),
  });
});

// 2. GET /products/:id
app.get("/products/:id", (req, res) => {
  const product = products.find((p) => String(p.id) === String(req.params.id));
  if (!product) {
    return res.status(404).json({ message: "Product not found" });
  }
  res.json(product);
});

const PORT = 4000;
app.listen(PORT, () => console.log(`Express API running on http://localhost:${PORT}`));
```
- **Expected result:** Terminal me `node server/index.js` chalane par port 4000 par API ready ho jayegi.

---

### Step 2: Route-level Loading UI (`src/app/products/loading.tsx`) banao
- **Kya karna hai:** `src/app/products/loading.tsx` file add karo jisme skeleton animation cards hon.
- **Kyu karna hai:** Jab server data fetch kar raha ho to user ko blank white screen ki jagah loading skeletons dikhein.
- **Code (`src/app/products/loading.tsx`):**
```tsx
export default function ProductsLoading() {
  return (
    <div className="p-6">
      <div className="h-8 w-48 bg-slate-200 animate-pulse rounded mb-6"></div>
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        {[1, 2, 3, 4, 5, 6].map((n) => (
          <div key={n} className="border border-slate-200 rounded-xl p-4 bg-white shadow-sm space-y-3">
            <div className="h-44 bg-slate-200 animate-pulse rounded-lg"></div>
            <div className="h-5 w-3/4 bg-slate-200 animate-pulse rounded"></div>
            <div className="h-4 w-1/2 bg-slate-200 animate-pulse rounded"></div>
            <div className="h-6 w-1/3 bg-slate-200 animate-pulse rounded"></div>
          </div>
        ))}
      </div>
    </div>
  );
}
```
- **Expected result:** `/products` route load hote waqt animated skeleton cards display honge.

---

### Step 3: Server-side Data Fetching implement karo (`src/app/products/page.tsx`)
- **Kya karna hai:** Express API se products fetch karo query parameters (`category`, `page`) ke sath.
- **Kyu karna hai:** Server Component me initial data load karke SEO friendly markup generate karne ke liye.
- **Code (`src/app/products/page.tsx`):**
```tsx
import Link from "next/link";

interface PageProps {
  searchParams: Promise<{ category?: string; page?: string }>;
}

async function getProducts(category?: string, page: string = "1") {
  const query = new URLSearchParams({ page });
  if (category) query.append("category", category);

  const res = await fetch(`http://localhost:4000/products?${query.toString()}`, {
    cache: "no-store", // Ensure fresh data
  });

  if (!res.ok) {
    throw new Error("Failed to fetch products from server");
  }
  return res.json();
}

export default async function ProductsPage({ searchParams }: PageProps) {
  const { category, page } = await searchParams;
  const result = await getProducts(category, page);

  return (
    <div className="p-6">
      <h1 className="text-3xl font-bold mb-6 text-slate-900">
        Products {category && <span className="text-blue-600">({category})</span>}
      </h1>
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        {result.data.map((product: any) => (
          <Link
            key={product.id}
            href={`/products/${product.id}`}
            className="border rounded-xl p-4 bg-white hover:shadow-md transition block"
          >
            <h2 className="font-semibold text-lg text-slate-800">{product.name}</h2>
            <p className="text-sm text-slate-500 mt-1 capitalize">{product.category}</p>
            <p className="text-xl font-bold text-blue-600 mt-3">${product.price}</p>
          </Link>
        ))}
      </div>
    </div>
  );
}
```
- **Expected result:** Express API ka real data cards ke andar display hoga.

---

### Step 4: Route-level Error Handling (`src/app/products/error.tsx`) implement karo
- **Kya karna hai:** `src/app/products/error.tsx` file create karo `'use client'` ke sath jo error message aur retry button render kare.
- **Kyu karna hai:** API down hone ya network fail hone par app crash na ho aur user ko retry ka option mile.
- **Code (`src/app/products/error.tsx`):**
```tsx
"use client";

import { useEffect } from "react";

export default function ProductsError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error("Products Route Error:", error);
  }, [error]);

  return (
    <div className="p-12 text-center max-w-lg mx-auto">
      <div className="text-red-500 text-5xl mb-4">⚠️</div>
      <h2 className="text-2xl font-bold text-slate-900">Unable to load products</h2>
      <p className="mt-2 text-slate-600 text-sm">
        We encountered a problem while fetching the product catalog. Please ensure the API is running.
      </p>
      <button
        onClick={() => reset()}
        className="mt-6 px-6 py-2.5 bg-blue-600 text-white rounded-lg font-semibold hover:bg-blue-700 transition"
      >
        Try Again
      </button>
    </div>
  );
}
```
- **Expected result:** Agar Express API band ho to user-friendly error box aur retry button dikhega.

---

### Step 5: Product Not Found UI (`not-found.tsx` aur `notFound()`) banao
- **Kya karna hai:** `src/app/products/[id]/page.tsx` me 404 response check karke `notFound()` call karo, aur `src/app/products/[id]/not-found.tsx` create karo.
- **Kyu karna hai:** Invalid product ID enter karne par clear feedback dene ke liye.
- **Code (`src/app/products/[id]/not-found.tsx`):**
```tsx
import Link from "next/link";

export default function ProductNotFound() {
  return (
    <div className="p-16 text-center max-w-md mx-auto">
      <h2 className="text-4xl font-extrabold text-slate-900">Product Not Found</h2>
      <p className="mt-3 text-slate-600">
        Sorry, the product you are searching for does not exist or has been removed.
      </p>
      <Link
        href="/products"
        className="inline-block mt-6 px-5 py-2.5 bg-blue-600 text-white font-medium rounded-lg hover:bg-blue-700 transition"
      >
        Return to Catalog
      </Link>
    </div>
  );
}
```
- **Code trigger in `src/app/products/[id]/page.tsx`:**
```tsx
import { notFound } from "next/navigation";

async function getProductById(id: string) {
  const res = await fetch(`http://localhost:4000/products/${id}`, { cache: "no-store" });
  if (res.status === 404) return null;
  if (!res.ok) throw new Error("Failed to fetch product details");
  return res.json();
}

export default async function ProductDetailPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const product = await getProductById(id);

  if (!product) {
    notFound(); // Triggers not-found.tsx
  }

  return <div className="p-8"><h1>{product.name}</h1></div>;
}
```
- **Expected result:** `/products/invalid-999` par custom product not found screen display hogi.

---

### Step 6: Suspense Boundary se Slow Section stream karo
- **Kya karna hai:** Product detail page me ek artificially slow component (jaise Customer Reviews) ko `<Suspense fallback={<ReviewsSkeleton />}>` me wrap karo.
- **Kyu karna hai:** Slow API call poore page ko block na kare aur instant streaming response mile.
- **Code (`src/components/SlowReviews.tsx`):**
```tsx
async function getReviews(productId: string) {
  // Artificial 3-second delay simulation
  await new Promise((resolve) => setTimeout(resolve, 3000));
  return [
    { id: 1, user: "Aarav", comment: "Outstanding build quality!" },
    { id: 2, user: "Priya", comment: "Totally worth the price." },
  ];
}

export default async function SlowReviews({ productId }: { productId: string }) {
  const reviews = await getReviews(productId);

  return (
    <div className="mt-8 border-t pt-6">
      <h3 className="text-xl font-bold mb-4">Customer Reviews</h3>
      <div className="space-y-3">
        {reviews.map((r) => (
          <div key={r.id} className="p-3 bg-slate-50 rounded border text-sm">
            <p className="font-semibold text-slate-800">{r.user}</p>
            <p className="text-slate-600 mt-1">{r.comment}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```
- **Code integration inside `page.tsx`:**
```tsx
import { Suspense } from "react";
import SlowReviews from "@/components/SlowReviews";

// Inside ProductDetailPage:
<Suspense fallback={<div className="h-24 bg-slate-100 animate-pulse rounded mt-6 p-4">Loading reviews...</div>}>
  <SlowReviews productId={id} />
</Suspense>
```
- **Expected result:** Main product details turant open ho jayengi aur reviews section 3 seconds baad smoothly pop in hoga.

---

## 4. Final File Structure

```text
server/
└── index.js                    # Express API server (port 4000)
src/
├── app/
│   └── products/
│       ├── [id]/
│       │   ├── not-found.tsx   # Custom 404 for missing product
│       │   └── page.tsx        # Product detail with Suspense & notFound()
│       ├── error.tsx           # Route-level error boundary ("use client")
│       ├── loading.tsx         # Route-level skeleton loader
│       └── page.tsx            # Main catalog fetching from Express API
└── components/
    └── SlowReviews.tsx         # Independently streamed async component
```

---

## 5. Common Mistakes / Errors

1. **`error.tsx` me `"use client"` na lagana:**
   - *Galti:* `error.tsx` ko Server Component chhod dena jisse build fail ho jati hai.
   - *Fix:* Error boundaries hamesha Client Components hone chahiye; top par `"use client";` zaroor lagayein.

2. **`reset()` call karne ke baad data update na hona:**
   - *Galti:* `error.tsx` ka `reset()` sirf error state ko re-render karta hai.
   - *Fix:* Ensure karo ki server fetch me stale cache na ho (use `{ cache: "no-store" }` ya dynamic fetching).

3. **Missing Product par generic 500 error throw karna:**
   - *Galti:* Express se 404 aane par seedha `throw new Error()` likhna jisse user ko error screen dikhe na ki not-found screen.
   - *Fix:* Status 404 aane par `notFound()` function call karo taaki Next.js ka `not-found.tsx` trigger ho.

4. **Express CORS configure na karna:**
   - *Galti:* Express server me `cors` middleware add na karna jisse browser client fetch fail ho jaye.
   - *Fix:* `app.use(cors())` Express server me zaroor include karein.

---

## 6. Verification Checklist
- [ ] Express API `http://localhost:4000/products` aur `http://localhost:4000/products/:id` par successfully respond kar raha hai.
- [ ] `/products` load hone par route skeleton `loading.tsx` display hota hai.
- [ ] Valid product ID par page render hota hai aur `SlowReviews` Suspense fallback ke baad appear hota hai.
- [ ] Invalid product ID par `not-found.tsx` trigger hota hai.
- [ ] Express server band karne par `error.tsx` fallback aur "Try Again" button render hota hai.
- [ ] Express server wapas chalu karke "Try Again" click karne par page recover ho jata hai.

---

## 7. Bonus (Extra Practice)
1. **Pagination Controls:** Products page ke bottom par "Previous" aur "Next" buttons lagao jo URL query param `?page=2` update karein.
2. **Error Logging Service Hook:** `error.tsx` ke `useEffect` me client error details ko console ke sath kisi analytics endpoint par POST karne ka simulation karo.
