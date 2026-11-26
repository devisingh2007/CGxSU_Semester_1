# Step-by-Step Guide: Assignment 15 — Advanced Routing (Parallel & Intercepting Routes)

---

## 1. Assignment ka Goal
Is assignment ka main objective Next.js App Router ke do sabse powerful routing patterns — **Parallel Routes** aur **Intercepting Routes** — ko master karna hai. Hume ek **Product Quick-View Modal** experience banana hai:
- Jab user catalog page (`/products`) par kisi product card par click kare, to URL `/products/123` par switch ho aur product details ek **Modal Overlay** me open ho bina catalog context kho-e.
- Browser ka **Back button** dabane par modal close ho jaye aur catalog par wapas aa jaye.
- Agar koi user `/products/123` ko direct new tab me open kare ya page reload (hard refresh) kare, to modal ki jagah **Full Standalone Product Details Page** load ho.

---

## 2. Zaroori Concepts (Recap)
- **Parallel Routes (`@slot`)**: Ek hi layout ke andar do alag pages ko concurrently render karne ke liye named slots define karte hain (e.g., `@modal`). Layout in slots ko as props receive karta hai (`props.modal`).
- **`default.tsx`**: Parallel route slots me fallback file hoti hai. Jab current URL par slot ke liye koi matching route na ho, to `default.tsx` render hota hai (usually returning `null`).
- **Intercepting Routes (`(.)`, `(..)`, `(...)`)**: Client-side navigation ke waqt kisi doosre route ko intercept karke current context ke andar modal me render karne ki facility:
  - `(.)` : Same-level segment match karta hai (e.g. `(.)[id]`).
  - `(..)` : One-level above segment match karta hai.
  - `(...)` : Root `app` directory se match karta hai.
- **Direct Navigation vs Client Navigation**: Next.js client-side link click par route intercept karta hai, jabki browser refresh ya direct URL par standard destination page load karta hai.

> 📖 Course Notes Reference: Parallel aur intercepting routes ke conventions ke liye is folder ki note file [15.advanced_routing.md](./15.advanced_routing.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Parallel Route Slot `@modal` create karo
- **Kya karna hai:** `src/app/products/` ke andar ek named slot folder banao: `@modal`.
- **Kyu karna hai:** Products section ke layout ko batane ke liye ki yaha ek secondary view (modal) render ho sakta hai.
- **Code:**
```bash
mkdir src/app/products/@modal
```
- **Expected result:** Folder structure tayyar ho jayega.

---

### Step 2: Fallback `default.tsx` banao (`src/app/products/@modal/default.tsx`)
- **Kya karna hai:** `@modal` folder ke andar `default.tsx` file create karo jo `null` return kare.
- **Kyu karna hai:** Jab user normal `/products` page par ho aur koi modal active na ho, to Next.js blank render kare bina 404 error diye.
- **Code (`src/app/products/@modal/default.tsx`):**
```tsx
export default function DefaultModal() {
  return null; // Jab koi modal open na ho to kuch mat dikhao
}
```
- **Expected result:** Active route na hone par empty slot render hoga.

---

### Step 3: Products Layout me Modal Slot integrate karo (`src/app/products/layout.tsx`)
- **Kya karna hai:** `layout.tsx` ke props me `children` ke sath `modal` prop add karo aur JSX me render karo.
- **Kyu karna hai:** Dono regular page (`children`) aur quick-view popup (`modal`) ek hi layout me sath render ho sakein.
- **Code (`src/app/products/layout.tsx`):**
```tsx
export default function ProductsLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <div className="relative">
      {/* Main catalog view */}
      {children}

      {/* Parallel modal slot */}
      {modal}
    </div>
  );
}
```
- **Expected result:** Layout ab modal overlay render karne ke liye ready hai.

---

### Step 4: Reusable Modal Wrapper component banao (`src/components/Modal.tsx`)
- **Kya karna hai:** `src/components/Modal.tsx` create karo jo background backdrop, dialog box aur close button (`useRouter().back()`) render kare.
- **Kyu karna hai:** Consistent backdrop blur aur ESC/backdrop click par URL history revert karne ke liye.
- **Code (`src/components/Modal.tsx`):**
```tsx
"use client";

import { useRouter } from "next/navigation";
import { useEffect } from "react";

export default function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter();

  const handleClose = () => {
    router.back(); // Reverts URL back to /products
  };

  // Close on Escape key press
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === "Escape") handleClose();
    };
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, []);

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm p-4">
      <div className="relative w-full max-w-lg rounded-2xl bg-white p-6 shadow-2xl animate-in fade-in zoom-in-95 duration-200">
        <button
          onClick={handleClose}
          className="absolute top-4 right-4 text-slate-400 hover:text-slate-700 font-bold text-xl"
          aria-label="Close modal"
        >
          ✕
        </button>
        {children}
      </div>
    </div>
  );
}
```
- **Expected result:** Clean backdrop aur keyboard-friendly close behavior.

---

### Step 5: Intercepting Route implement karo (`src/app/products/@modal/(.)[id]/page.tsx`)
- **Kya karna hai:** Folder banao `@modal/(.)[id]/page.tsx`. `(.)` ka matlab same folder level ke dynamic `[id]` route ko intercept karna hai.
- **Kyu karna hai:** Jab user `/products` se kisi product link par click kare, to full page navigation ke badle modal view load ho.
- **Code (`src/app/products/@modal/(.)[id]/page.tsx`):**
```tsx
import Modal from "@/components/Modal";
import Link from "next/link";

export default async function ProductQuickViewModal({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  // Fetch product or mock details
  return (
    <Modal>
      <div className="space-y-4">
        <span className="text-xs font-bold uppercase tracking-wider text-blue-600">Quick View</span>
        <h2 className="text-2xl font-bold text-slate-900">Product #{id}</h2>
        <p className="text-slate-600 text-sm">
          This is a quick preview of product details loaded dynamically inside an intercepting route modal.
        </p>
        <div className="pt-4 flex justify-between items-center border-t">
          <Link
            href={`/products/${id}`}
            className="text-sm font-semibold text-blue-600 hover:underline"
          >
            Open Full Product Page &rarr;
          </Link>
          <button className="px-4 py-2 bg-blue-600 text-white rounded-lg font-medium text-sm">
            Add to Cart
          </button>
        </div>
      </div>
    </Modal>
  );
}
```
- **Expected result:** Click karne par background catalog rehta hai aur upar popup modal khul jata hai.

---

### Step 6: Full Standalone Product Page ensure karo (`src/app/products/[id]/page.tsx`)
- **Kya karna hai:** Standalone details page create karo jo direct visits par khulega.
- **Kyu karna hai:** Jab user link share kare ya page refresh kare to proper standalone view mil sake.
- **Code (`src/app/products/[id]/page.tsx`):**
```tsx
import Link from "next/link";

export default async function FullProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return (
    <div className="max-w-3xl mx-auto p-8 bg-white border rounded-2xl shadow-sm mt-8">
      <Link href="/products" className="text-sm text-blue-600 hover:underline mb-4 inline-block">
        &larr; Back to Catalog
      </Link>
      <h1 className="text-4xl font-extrabold text-slate-900">Full Details: Product #{id}</h1>
      <p className="mt-4 text-slate-600 text-base leading-relaxed">
        You are viewing the dedicated standalone product page. This page renders directly when accessed via direct URL, bookmark, or hard refresh.
      </p>
    </div>
  );
}
```
- **Expected result:** Standalone full page view bina kisi modal backdrop ke render hoga.

---

### Step 7: Dual Navigation Behavior test karo
- **Kya karna hai:**
  1. `/products` par jao aur kisi product link par click karo → Modal khulna chahiye aur address bar `/products/123` ban jana chahiye.
  2. Browser ka Back button dabao → Modal band ho jana chahiye aur URL wapas `/products` hona chahiye.
  3. Modal open hone ke baad page par `F5` (Refresh) dabao → Modal ke bajaye Full standalone page open hona chahiye!
- **Expected result:** Complete advanced routing functionality verified.

---

## 4. Final File Structure

```text
src/
├── app/
│   └── products/
│       ├── @modal/
│       │   ├── (.)[id]/
│       │   │   └── page.tsx     # Intercepted modal route
│       │   └── default.tsx      # Fallback when modal is closed (returns null)
│       ├── [id]/
│       │   └── page.tsx         # Full standalone product detail page
│       ├── layout.tsx           # Renders { children, modal }
│       └── page.tsx             # Products listing catalog
└── components/
    └── Modal.tsx                # Client modal wrapper with router.back()
```

---

## 5. Common Mistakes / Errors

1. **`default.tsx` create karna bhool jana:**
   - *Galti:* `@modal` ke andar `default.tsx` na rakhna. Isse jab user normal `/products` page par refresh karega to Next.js 404 dega.
   - *Fix:* Always `@modal/default.tsx` file add karein jo `return null;` karti ho.

2. **Intercepting prefix galat lagana:**
   - *Galti:* `(..)[id]` likh dena jabki dynamic route usi same directory level par hai.
   - *Fix:* Same level intercept karne ke liye single dot `(.)` use karein: `(.)[id]`.

3. **Modal close par state reset na karke normal navigate karna:**
   - *Galti:* `router.push('/products')` karna jisse browser history stack kharab ho jati hai.
   - *Fix:* Always `router.back()` use karein taaki browser history clean rahe.

4. **Layout me `modal` prop type define na karna:**
   - *Galti:* Layout me sirf `{ children }` accept karna aur `{ modal }` ko ignore karna.
   - *Fix:* Layout component me `{ children, modal }: { children: React.ReactNode; modal: React.ReactNode }` zaroor include karein.

---

## 6. Verification Checklist
- [ ] Catalog page (`/products`) se product link click karne par popup modal open hota hai.
- [ ] Modal open hone ke sath address bar `/products/[id]` me change hota hai.
- [ ] Close button ya ESC key dabane par modal disappear hota hai aur address bar `/products` par wapas aa jata hai.
- [ ] Browser ka native Back button dabane par modal gracefully close ho jata hai.
- [ ] Modal open rehne ke dauran page refresh karne par full standalone product page display hota hai.
- [ ] New incognito tab me directly `/products/123` paste karne par directly full standalone page load hota hai.

---

## 7. Bonus (Extra Practice)
1. **Animated Backdrop Transitions:** Tailwind CSS `transition-opacity` aur CSS keyframe animations ke zariye smooth scale-up effect add karo.
2. **Scroll Lock Hook:** Modal open hone par background page ki scrolling ko disable (`document.body.style.overflow = 'hidden'`) karo.
