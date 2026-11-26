# Step-by-Step Guide: Assignment 05 — Server & Client Components

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router me **Server Components** aur **Client Components** ke darmiyan clear boundary establish karna hai. Hume product page ko Server Component rakh kar fast SSR (Server-Side Rendering) retain karni hai, aur sirf interactive parts — jaise **Variant Selector**, **Quantity Selector**, aur **Add to Cart** — ko small Client Components me isolate karke props aur `children` pattern ke through integrate karna hai.

---

## 2. Zaroori Concepts (Recap)
- **Server Components (Default)**: Next.js App Router me sabhi components by default Server Components hote hain. Ye server par execute hote hain, inka JavaScript browser me nahi jata, aur ye fast data loading dete hain. Inme `useState`, `useEffect`, ya DOM event listeners (`onClick`) use nahi ho sakte.
- **Client Components (`"use client"`)**: File ke top par `"use client"` likhne se component client boundary me switch hota hai. Ye browser me hydrate hote hain aur React state (`useState`), browser events, aur hooks support karte hain.
- **Passing Props across the Boundary**: Server Components se Client Components ko data props ke zariye bheja jata hai. Data serializable (strings, numbers, objects, arrays) hona chahiye.
- **Composition with `children` Pattern**: Ek Client Component server-rendered content ko as `children` accept kar sakta hai. Isse wrapped server component Client Component banaye bina render hota rehta hai.

> 📖 Course Notes Reference: Server vs Client decision rules ke liye is folder ki note file [05.server_and_client_components.md](./05.server_and_client_components.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Types define karo (`src/types/product.ts`)
- **Kya karna hai:** Product aur ProductVariant ke TypeScript interfaces create karo.
- **Kyu karna hai:** Data passing aur component props ko type-safe banane ke liye.
- **Code (`src/types/product.ts`):**
```typescript
export interface ProductVariant {
  id: string;
  name: string; // e.g. "Midnight Black", "Space Gray"
  inStock: boolean;
}

export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  brand: string;
  category: string;
  rating: number;
  images: string[];
  stock: number;
  variants: ProductVariant[];
}
```
- **Expected result:** Type definitions ready ho jayengi jo dono Server aur Client components me import ho sakti hain.

---

### Step 2: Client Component — Variant Selector banao
- **Kya karna hai:** `src/components/VariantSelector.tsx` banao `'use client'` directive aur local state ke sath.
- **Kyu karna hai:** User click karke variant choose kar sake aur selected variant highlighted rahe.
- **Code (`src/components/VariantSelector.tsx`):**
```tsx
"use client";

import { useState } from "react";
import { ProductVariant } from "@/types/product";

interface VariantSelectorProps {
  variants: ProductVariant[];
  onSelect?: (variantId: string) => void;
}

export default function VariantSelector({ variants, onSelect }: VariantSelectorProps) {
  const [selectedId, setSelectedId] = useState<string>(variants[0]?.id || "");

  const handleSelect = (id: string) => {
    setSelectedId(id);
    if (onSelect) onSelect(id);
  };

  return (
    <div className="my-4">
      <label className="block text-sm font-semibold text-slate-700 mb-2">
        Select Variant:
      </label>
      <div className="flex flex-wrap gap-2">
        {variants.map((v) => {
          const isSelected = v.id === selectedId;
          return (
            <button
              key={v.id}
              onClick={() => handleSelect(v.id)}
              disabled={!v.inStock}
              className={`px-4 py-2 text-sm rounded-lg border font-medium transition ${
                !v.inStock
                  ? "border-slate-200 bg-slate-100 text-slate-400 cursor-not-allowed"
                  : isSelected
                  ? "border-blue-600 bg-blue-50 text-blue-700 shadow-sm"
                  : "border-slate-300 bg-white text-slate-700 hover:border-slate-400"
              }`}
            >
              {v.name} {!v.inStock && "(Out of Stock)"}
            </button>
          );
        })}
      </div>
    </div>
  );
}
```
- **Expected result:** Interactive variant picker button group render hoga.

---

### Step 3: Client Component — Quantity Selector banao
- **Kya karna hai:** `src/components/QuantitySelector.tsx` create karo jo count maintain kare aur 1 se kam na hone de.
- **Kyu karna hai:** Client-side counter state manage karne ke liye.
- **Code (`src/components/QuantitySelector.tsx`):**
```tsx
"use client";

import { useState } from "react";

interface QuantitySelectorProps {
  maxStock: number;
  onChange?: (quantity: number) => void;
}

export default function QuantitySelector({ maxStock, onChange }: QuantitySelectorProps) {
  const [quantity, setQuantity] = useState<number>(1);

  const increment = () => {
    if (quantity < maxStock) {
      const next = quantity + 1;
      setQuantity(next);
      onChange?.(next);
    }
  };

  const decrement = () => {
    if (quantity > 1) {
      const next = quantity - 1;
      setQuantity(next);
      onChange?.(next);
    }
  };

  return (
    <div className="my-4 flex items-center gap-3">
      <span className="text-sm font-semibold text-slate-700">Quantity:</span>
      <div className="flex items-center border border-slate-300 rounded-lg overflow-hidden bg-white">
        <button
          onClick={decrement}
          disabled={quantity <= 1}
          className="px-3 py-1 bg-slate-100 hover:bg-slate-200 disabled:opacity-50 text-slate-700 font-bold transition"
        >
          -
        </button>
        <span className="px-4 py-1 text-sm font-semibold text-slate-800 min-w-[2.5rem] text-center">
          {quantity}
        </span>
        <button
          onClick={increment}
          disabled={quantity >= maxStock}
          className="px-3 py-1 bg-slate-100 hover:bg-slate-200 disabled:opacity-50 text-slate-700 font-bold transition"
        >
          +
        </button>
      </div>
      <span className="text-xs text-slate-500">({maxStock} available)</span>
    </div>
  );
}
```
- **Expected result:** Click karne par quantity smoothly increment/decrement hogi aur minimum 1 par freeze rahegi.

---

### Step 4: Client Component — Add to Cart Button banao
- **Kya karna hai:** `src/components/AddToCartButton.tsx` create karo jo click hone par visual feedback (e.g. "Added!") provide kare.
- **Kyu karna hai:** Cart interaction demonstration ke liye bina heavy cart architecture add kiye.
- **Code (`src/components/AddToCartButton.tsx`):**
```tsx
"use client";

import { useState } from "react";

interface AddToCartButtonProps {
  productId: string;
  productName: string;
}

export default function AddToCartButton({ productId, productName }: AddToCartButtonProps) {
  const [added, setAdded] = useState(false);

  const handleAddToCart = () => {
    setAdded(true);
    console.log(`Product added to cart: ${productName} (ID: ${productId})`);
    setTimeout(() => setAdded(false), 2000);
  };

  return (
    <button
      onClick={handleAddToCart}
      className={`w-full py-3 px-6 rounded-lg font-bold text-white transition shadow-md ${
        added ? "bg-emerald-600" : "bg-blue-600 hover:bg-blue-700"
      }`}
    >
      {added ? "✓ Added to Cart!" : "Add to Cart"}
    </button>
  );
}
```
- **Expected result:** Button click karne par button green ho jayega aur console me event log hoga.

---

### Step 5: Client Wrapper with `children` Pattern banao
- **Kya karna hai:** `src/components/ExpandableSection.tsx` banao jo server content ko as `children` accept karke expand/collapse kare.
- **Kyu karna hai:** Yeh demonstrate karna ki Server Component ko Client Component ke andar bina server rendering loose kiye kaise render kiya jata hai.
- **Code (`src/components/ExpandableSection.tsx`):**
```tsx
"use client";

import { useState } from "react";

interface ExpandableSectionProps {
  title: string;
  children: React.ReactNode;
}

export default function ExpandableSection({ title, children }: ExpandableSectionProps) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div className="border border-slate-200 rounded-lg p-4 mt-6 bg-white">
      <button
        onClick={() => setIsExpanded(!isExpanded)}
        className="w-full flex justify-between items-center text-left font-bold text-slate-800"
      >
        <span>{title}</span>
        <span className="text-blue-600 text-lg">{isExpanded ? "▲" : "▼"}</span>
      </button>
      {isExpanded && <div className="mt-4 pt-4 border-t border-slate-100">{children}</div>}
    </div>
  );
}
```
- **Expected result:** Server-rendered product specs ya description ko toggleable accordion me display kiya ja sakega.

---

### Step 6: Server Product Page (`src/app/products/[id]/page.tsx`) assemble karo
- **Kya karna hai:** Product detail page ko Server Component rehne do aur Client Components ko import karke data props pass karo.
- **Kyu karna hai:** Server-first architecture ensure karne ke liye.
- **Code (`src/app/products/[id]/page.tsx`):**
```tsx
import VariantSelector from "@/components/VariantSelector";
import QuantitySelector from "@/components/QuantitySelector";
import AddToCartButton from "@/components/AddToCartButton";
import ExpandableSection from "@/components/ExpandableSection";
import { Product } from "@/types/product";

// Simulated product data
const dummyProduct: Product = {
  id: "prod-101",
  name: "Sony WH-1000XM5 Wireless Headphones",
  description: "Industry-leading noise canceling with two processors and 8 microphones.",
  price: 399,
  brand: "Sony",
  category: "Electronics",
  rating: 4.8,
  images: ["/images/headphones.jpg"],
  stock: 12,
  variants: [
    { id: "v1", name: "Black", inStock: true },
    { id: "v2", name: "Silver", inStock: true },
    { id: "v3", name: "Midnight Blue", inStock: false },
  ],
};

export default async function ProductDetailPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = { ...dummyProduct, id };

  return (
    <div className="max-w-4xl mx-auto p-6">
      {/* Server-Rendered Content */}
      <div className="border-b border-slate-200 pb-4">
        <span className="text-xs uppercase font-semibold tracking-wider text-blue-600">
          {product.category} &bull; {product.brand}
        </span>
        <h1 className="text-3xl font-bold text-slate-900 mt-1">{product.name}</h1>
        <p className="text-2xl font-extrabold text-slate-900 mt-2">${product.price}</p>
        <p className="text-slate-600 mt-3">{product.description}</p>
      </div>

      {/* Client Components for Interactivity */}
      <VariantSelector variants={product.variants} />
      <QuantitySelector maxStock={product.stock} />
      <AddToCartButton productId={product.id} productName={product.name} />

      {/* Children pattern: Server content inside Client Component */}
      <ExpandableSection title="Product Specifications & Details (Server Rendered)">
        <ul className="list-disc pl-5 text-sm text-slate-600 space-y-1">
          <li>Battery Life: Up to 30 hours</li>
          <li>Weight: 250 grams</li>
          <li>Driver Unit: 30mm Precision engineered</li>
          <li>Product SKU: <span className="font-mono">{product.id}</span></li>
        </ul>
      </ExpandableSection>
    </div>
  );
}
```
- **Expected result:** Clean separation: page server-rendered rehta hai jabki controls fully interactive hote hain.

---

## 4. Final File Structure

```text
src/
├── app/
│   └── products/
│       └── [id]/
│           └── page.tsx         # Server Component (default)
├── components/
│   ├── AddToCartButton.tsx      # Client Component ("use client")
│   ├── ExpandableSection.tsx    # Client Component using children
│   ├── QuantitySelector.tsx     # Client Component ("use client")
│   └── VariantSelector.tsx      # Client Component ("use client")
└── types/
    └── product.ts               # Shared TypeScript interfaces
```

---

## 5. Common Mistakes / Errors

1. **Poore page par `"use client"` laga dena:**
   - *Galti:* Interactive button chahiye tha isliye `page.tsx` ke top par `"use client"` likh diya. Isse page ka saara code client bundle me chala jata hai aur SEO/initial load speed kharab hoti hai.
   - *Fix:* Page ko Server Component rehne do; sirf button ya selector me `"use client"` lagao.

2. **Non-serializable props pass karna:**
   - *Galti:* Server Component se Client Component ko function (e.g. `onClick={() => console.log()}`) pass karna jo Server Actions ke bina pass nahi ho sakti.
   - *Fix:* Client component ke andar hi handler function define karein ya state update karein.

3. **Client Component me direct Server Secrets access karna:**
   - *Galti:* Client component me `process.env.DATABASE_URL` padhna (ye browser me security risk ya undefined hoga).
   - *Fix:* Sensitive data sirf Server Components me rahe; Client Component me sirf UI ke liye zaroori props bhejo.

4. **Children pattern na samajhna:**
   - *Galti:* Client Component ke andar Server Component direct import kar lena. (Direct import karne par imported component bhi client bundle ka hissa ban jata hai).
   - *Fix:* Parent Server Component me Server Component ko render karke use `<ClientWrapper>{serverContent}</ClientWrapper>` as `children` pass karo.

---

## 6. Verification Checklist
- [ ] `src/app/products/[id]/page.tsx` me `"use client"` directive nahi hai (ye Server Component hai).
- [ ] `VariantSelector`, `QuantitySelector`, aur `AddToCartButton` me `"use client"` mojood hai.
- [ ] Variants click karne par highlight change ho rahi hai aur out-of-stock disable hai.
- [ ] Quantity selector 1 se kam nahi jaata aur max stock se zyada nahi badhta.
- [ ] Add to cart click hone par interactive visual change dikhata hai.
- [ ] `ExpandableSection` me server-rendered specs safely `children` ke zariye toggle ho rahi hain.
- [ ] Browser console me koi serialization error ya React hydration mismatch nahi hai.

---

## 7. Bonus (Extra Practice)
1. **Live Price Calculation:** Quantity change hone par total price dynamically calculate karke "Total: $XYZ" display karo using client state.
2. **Stock Badge Indicator:** Agar stock 3 se kam ho to red pulsing badge "Only X left!" display karo.
