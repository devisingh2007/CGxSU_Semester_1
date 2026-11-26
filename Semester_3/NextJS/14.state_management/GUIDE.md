# Step-by-Step Guide: Assignment 14 — State Management with Zustand

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router application me shared client-side state implement karna hai using **Zustand**. Hume:
- **Shopping Cart store** (`lib/stores/cart.store.ts`) create karna hai with actions (add, remove, quantity update, clear, duplicate detection)
- **Wishlist store** (`lib/stores/wishlist.store.ts`) create karna hai with toggle actions
- Dedicated `/cart` page (item subtotal, cart total, quantity controls) aur `/wishlist` page build karna hai
- Navbar me real-time total cart count badge show karna hai bina hydration error ke.

---

## 2. Zaroori Concepts (Recap)
- **Zustand**: Ek ultra-lightweight, boilerplate-free state management library. Redux ki tarah heavy providers ya context wrappers ki zaroorat nahi hoti.
- **Client Component Boundary**: Zustand stores client-side memory me run hote hain, isliye stores ko consume karne waale components `"use client"` hone chahiye.
- **Local State vs Global Store vs Server State**:
  - *Local Component State (`useState`)*: Modals, dropdowns, UI toggles ke liye.
  - *Global Client Store (Zustand)*: Cross-component shared UI data jaise Cart badge aur Cart page.
  - *Server State*: Product catalogue aur orders jo database me reside karte hain.
- **Next.js Hydration & SSR**: Server par render hote waqt client storage (localStorage) empty hota hai. Agar proper hydration guard na lagaya jaye to `Text content did not match` error aata hai.

> 📖 Course Notes Reference: Zustand patterns aur best practices ke liye is folder ki note file [14.state_management.md](./14.state_management.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Zustand install karo
- **Kya karna hai:** Terminal me `zustand` package install karo.
- **Kyu karna hai:** State store library add karne ke liye.
- **Code (Terminal):**
```bash
npm install zustand
```
- **Expected result:** `package.json` me zustand successfully add ho jayegi.

---

### Step 2: Shopping Cart Store create karo (`src/lib/stores/cart.store.ts`)
- **Kya karna hai:** Cart item structure define karo aur `create()` ke sath store banao jo item add, update, remove aur clear support kare.
- **Kyu karna hai:** Cart state ko globally manage karne ke liye aur duplicate products add hone par quantity increase karne ke liye.
- **Code (`src/lib/stores/cart.store.ts`):**
```typescript
import { create } from "zustand";

export interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: Omit<CartItem, "quantity">) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  getTotalItems: () => number;
  getTotalPrice: () => number;
}

export const useCartStore = create<CartStore>((set, get) => ({
  items: [],

  addItem: (product) => {
    set((state) => {
      const existing = state.items.find((i) => i.productId === product.productId);
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.productId === product.productId ? { ...i, quantity: i.quantity + 1 } : i
          ),
        };
      }
      return { items: [...state.items, { ...product, quantity: 1 }] };
    });
  },

  removeItem: (productId) => {
    set((state) => ({
      items: state.items.filter((i) => i.productId !== productId),
    }));
  },

  updateQuantity: (productId, quantity) => {
    if (quantity <= 0) {
      get().removeItem(productId);
      return;
    }
    set((state) => ({
      items: state.items.map((i) =>
        i.productId === productId ? { ...i, quantity } : i
      ),
    }));
  },

  clearCart: () => set({ items: [] }),

  getTotalItems: () => get().items.reduce((sum, item) => sum + item.quantity, 0),

  getTotalPrice: () => get().items.reduce((sum, item) => sum + item.price * item.quantity, 0),
}));
```
- **Expected result:** Type-safe cart store ready ho jayega with automated duplicate detection.

---

### Step 3: Wishlist Store create karo (`src/lib/stores/wishlist.store.ts`)
- **Kya karna hai:** Wishlisted product IDs ka set/array maintain karne wala store banao with toggle action.
- **Kyu karna hai:** User ke favourite items track karne ke liye.
- **Code (`src/lib/stores/wishlist.store.ts`):**
```typescript
import { create } from "zustand";

interface WishlistStore {
  wishlistIds: string[];
  toggleWishlist: (productId: string) => void;
  isWishlisted: (productId: string) => boolean;
}

export const useWishlistStore = create<WishlistStore>((set, get) => ({
  wishlistIds: [],

  toggleWishlist: (productId) => {
    set((state) => {
      const exists = state.wishlistIds.includes(productId);
      return {
        wishlistIds: exists
          ? state.wishlistIds.filter((id) => id !== productId)
          : [...state.wishlistIds, productId],
      };
    });
  },

  isWishlisted: (productId) => get().wishlistIds.includes(productId),
}));
```
- **Expected result:** Reusable wishlist toggle store tayyar ho jayega.

---

### Step 4: Interactive Add to Cart & Wishlist Buttons banao
- **Kya karna hai:** `src/components/ProductActions.tsx` create karo jo Zustand store ke actions trigger kare.
- **Kyu karna hai:** User ko product list ya product detail se cart aur wishlist interact karne ki suvidha dena.
- **Code (`src/components/ProductActions.tsx`):**
```tsx
"use client";

import { useCartStore } from "@/lib/stores/cart.store";
import { useWishlistStore } from "@/lib/stores/wishlist.store";

interface ProductActionsProps {
  product: { id: string; name: string; price: number };
}

export default function ProductActions({ product }: ProductActionsProps) {
  const addItem = useCartStore((state) => state.addItem);
  const { toggleWishlist, isWishlisted } = useWishlistStore();
  const wishlisted = isWishlisted(product.id);

  return (
    <div className="flex gap-2 mt-4">
      <button
        onClick={() => addItem({ productId: product.id, name: product.name, price: product.price })}
        className="flex-1 py-2 px-4 bg-blue-600 text-white rounded-lg text-sm font-semibold hover:bg-blue-700 transition"
      >
        Add to Cart
      </button>
      <button
        onClick={() => toggleWishlist(product.id)}
        className={`px-3 py-2 rounded-lg border text-lg transition ${
          wishlisted ? "bg-red-50 border-red-300 text-red-500" : "bg-white border-slate-300 text-slate-400"
        }`}
        title="Toggle Wishlist"
      >
        ♥
      </button>
    </div>
  );
}
```
- **Expected result:** Buttons click karne par store me items update ho jayenge.

---

### Step 5: Navbar Cart Badge banao with Hydration Safety
- **Kya karna hai:** `src/components/CartBadge.tsx` create karo with mounted state check.
- **Kyu karna hai:** SSR hydration mismatch prevent karne ke liye aur live item count show karne ke liye.
- **Code (`src/components/CartBadge.tsx`):**
```tsx
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
import { useCartStore } from "@/lib/stores/cart.store";

export default function CartBadge() {
  const [mounted, setMounted] = useState(false);
  const totalItems = useCartStore((state) => state.getTotalItems());

  useEffect(() => {
    setMounted(true);
  }, []);

  return (
    <Link href="/cart" className="relative p-2 text-slate-700 hover:text-blue-600">
      <span className="text-xl">🛒</span>
      {mounted && totalItems > 0 && (
        <span className="absolute -top-1 -right-1 bg-blue-600 text-white text-xs font-bold rounded-full h-5 w-5 flex items-center justify-center animate-scaleIn">
          {totalItems}
        </span>
      )}
    </Link>
  );
}
```
- **Expected result:** Navbar me shopping cart icon ke upar real-time count badge dikhega bina console hydration warning ke.

---

### Step 6: Full Cart Page implement karo (`src/app/cart/page.tsx`)
- **Kya karna hai:** `/cart` page create karo jo items table, item subtotal, quantity buttons, remove button, clear cart button aur cart total render kare.
- **Kyu karna hai:** User ko poora shopping cart review aur modify karne ka experience dene ke liye.
- **Code (`src/app/cart/page.tsx`):**
```tsx
"use client";

import { useCartStore } from "@/lib/stores/cart.store";
import Link from "next/link";

export default function CartPage() {
  const { items, updateQuantity, removeItem, clearCart, getTotalPrice } = useCartStore();

  if (items.length === 0) {
    return (
      <div className="p-12 text-center max-w-md mx-auto">
        <h1 className="text-2xl font-bold mb-3">Your Cart is Empty</h1>
        <p className="text-slate-500 mb-6">Looks like you haven't added anything to your cart yet.</p>
        <Link href="/products" className="px-5 py-2.5 bg-blue-600 text-white rounded-lg font-medium">
          Start Shopping
        </Link>
      </div>
    );
  }

  return (
    <div className="max-w-4xl mx-auto p-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Shopping Cart</h1>
        <button
          onClick={clearCart}
          className="text-sm text-red-600 hover:underline font-medium"
        >
          Clear Entire Cart
        </button>
      </div>

      <div className="space-y-4">
        {items.map((item) => (
          <div
            key={item.productId}
            className="flex items-center justify-between p-4 bg-white rounded-xl border border-slate-200 shadow-sm"
          >
            <div>
              <h3 className="font-semibold text-slate-800">{item.name}</h3>
              <p className="text-sm text-slate-500">${item.price.toFixed(2)} each</p>
            </div>

            <div className="flex items-center gap-6">
              <div className="flex items-center border rounded-lg">
                <button
                  onClick={() => updateQuantity(item.productId, item.quantity - 1)}
                  className="px-2.5 py-1 text-slate-600 hover:bg-slate-100"
                >
                  -
                </button>
                <span className="px-3 font-semibold text-sm">{item.quantity}</span>
                <button
                  onClick={() => updateQuantity(item.productId, item.quantity + 1)}
                  className="px-2.5 py-1 text-slate-600 hover:bg-slate-100"
                >
                  +
                </button>
              </div>

              <span className="font-bold text-slate-900 min-w-[5rem] text-right">
                ${(item.price * item.quantity).toFixed(2)}
              </span>

              <button
                onClick={() => removeItem(item.productId)}
                className="text-red-500 hover:text-red-700 text-sm font-medium"
              >
                Remove
              </button>
            </div>
          </div>
        ))}
      </div>

      <div className="mt-8 p-6 bg-slate-100 rounded-xl flex justify-between items-center">
        <span className="text-lg font-bold text-slate-800">Total Amount:</span>
        <span className="text-2xl font-extrabold text-blue-600">
          ${getTotalPrice().toFixed(2)}
        </span>
      </div>
    </div>
  );
}
```
- **Expected result:** Real-time dynamic cart calculation screen smoothly kaam karegi.

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── cart/
│   │   └── page.tsx             # Full Cart view ("use client")
│   └── wishlist/
│       └── page.tsx             # Saved Wishlist view ("use client")
├── components/
│   ├── CartBadge.tsx            # Navbar count badge (hydration-safe)
│   └── ProductActions.tsx       # AddToCart & WishlistToggle
└── lib/
    └── stores/
        ├── cart.store.ts        # Zustand cart store
        └── wishlist.store.ts    # Zustand wishlist store
```

---

## 5. Common Mistakes / Errors

1. **Hydration Mismatch Error on Cart Badge:**
   - *Galti:* Server render par count 0 render hona aur client par store me 3 items hona, jisse Next.js hydration error throw karta hai.
   - *Fix:* `CartBadge` me `const [mounted, setMounted] = useState(false)` lagao aur sirf `mounted === true` hone par badge render karo.

2. **Zustand store ko Server Component me import karke hook call karna:**
   - *Galti:* Server component file me `useCartStore()` call karna.
   - *Fix:* Hooks sirf Client Components me kaam karte hain; component ke top par `"use client"` hona zaroori hai.

3. **Duplicate items ko alag-alag entries bana dena:**
   - *Galti:* Same product do baar add karne par cart list me do alag rows ban jaana.
   - *Fix:* `addItem` method me check karo ki agar `productId` already array me hai to sirf `quantity: quantity + 1` update karein.

4. **Product catalog ko Zustand store me source-of-truth bana dena:**
   - *Galti:* Database product data ko Zustand me rakh kar sochna ki database sync ho gaya.
   - *Fix:* Zustand sirf client UI interactions (cart, wishlist) ke liye hai; catalog ka source-of-truth database/API hi hona chahiye.

---

## 6. Verification Checklist
- [ ] Product card par "Add to Cart" click karne par Navbar me count badge increment hota hai.
- [ ] Same product dobara add karne par nayi row banne ke bajaye quantity update hoti hai.
- [ ] `/cart` page par har item ka subtotal (`price * quantity`) sahi calculate hota hai.
- [ ] Cart total saare items ka sum sahi reflect karta hai.
- [ ] `+` aur `-` buttons quantity update karte hain; `0` hone par item remove ho jata hai.
- [ ] "Clear Entire Cart" dabane par saare items clean ho jaate hain aur empty state dikhti hai.
- [ ] Wishlist toggle button favourite items ko `/wishlist` me add/remove karta hai.

---

## 7. Bonus (Extra Practice)
1. **LocalStorage Persistence:** Zustand ke built-in `persist` middleware (`import { persist } from "zustand/middleware"`) use karke cart items ko browser refresh ke baad bhi retain karo.
2. **Fly-out Cart Drawer:** Navbar ke cart button par click karne par full page jaane ke bajaye right-side slide-over drawer open karo.
