# Step-by-Step Guide: Assignment 04 — Layouts

---

## 1. Assignment ka Goal
Is assignment ka main goal Next.js me **Nested Layouts** ka architecture samajhna aur implement karna hai. Hume poori app ke liye ek **Root Layout** (`Navbar` aur `Footer` ke sath) banana hai aur specific `/products` section ke liye ek **Nested Products Layout** banana hai jisme ek shared **Sidebar** hoga, bina unnecessary page re-renders ke aur layout ko Server Component rakhte hue.

---

## 2. Zaroori Concepts (Recap)
- **Root Layout (`app/layout.tsx`)**: Har Next.js project me mandatory hota hai. Isme `<html>` aur `<body>` tags hote hain aur ye sabhi pages par persist karta hai.
- **Nested Layout (`app/products/layout.tsx`)**: Kisi specific sub-tree ke pages (jaise product listing aur product details) ke beech shared UI (jaise sidebar) maintain karta hai. Jab user sub-routes ke beech switch karta hai, to layout re-render nahi hota, sirf `children` update hote hain.
- **Server Components in Layouts**: By default layouts Server Components hote hain jo fast initial render dete hain. Agar interactive UI (e.g. mobile toggle menu) chahiye, to sirf us chhote component ko Client Component (`"use client"`) banaya jata hai.

> 📖 Course Notes Reference: Layout architecture aur best practices ke liye is folder ki note file [04.layouts.md](./04.layouts.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Reusable Navbar component banao
- **Kya karna hai:** `src/components/Navbar.tsx` create karo jisme store brand aur primary navigation links hon.
- **Kyu karna hai:** Header UI ko modular rakhne ke liye taaki root layout clean rahe.
- **Code (`src/components/Navbar.tsx`):**
```tsx
import Link from "next/link";

export default function Navbar() {
  return (
    <header className="border-b bg-white shadow-sm sticky top-0 z-50">
      <div className="container mx-auto flex h-16 items-center justify-between px-4">
        <Link href="/" className="text-xl font-bold text-blue-600">
          NextStore
        </Link>
        <nav className="flex items-center gap-6 text-sm font-medium text-slate-700">
          <Link href="/" className="hover:text-blue-600 transition">Home</Link>
          <Link href="/products" className="hover:text-blue-600 transition">Products</Link>
          <Link href="/categories" className="hover:text-blue-600 transition">Categories</Link>
          <Link href="/contact" className="hover:text-blue-600 transition">Contact</Link>
        </nav>
      </div>
    </header>
  );
}
```
- **Expected result:** Ek clean navigation bar component export ho jayega.

---

### Step 2: Reusable Footer component banao
- **Kya karna hai:** `src/components/Footer.tsx` create karo jisme copyright aur store links hon.
- **Kyu karna hai:** Application ke bottom me consistent footer UI show karne ke liye.
- **Code (`src/components/Footer.tsx`):**
```tsx
export default function Footer() {
  return (
    <footer className="border-t bg-slate-900 text-slate-400 py-8 text-sm">
      <div className="container mx-auto px-4 flex flex-col md:flex-row items-center justify-between gap-4">
        <p>&copy; {new Date().getFullYear()} NextStore Inc. All rights reserved.</p>
        <div className="flex gap-6">
          <a href="/terms" className="hover:text-white transition">Terms of Service</a>
          <a href="/privacy" className="hover:text-white transition">Privacy Policy</a>
          <a href="/contact" className="hover:text-white transition">Support</a>
        </div>
      </div>
    </footer>
  );
}
```
- **Expected result:** Standalone footer ready ho jayega.

---

### Step 3: Root Layout (`src/app/layout.tsx`) update karo
- **Kya karna hai:** `src/app/layout.tsx` me `Navbar` aur `Footer` ko import karke children ke upar aur neeche place karo.
- **Kyu karna hai:** Root layout poori application ki foundation hai; isse har page par automatic Navbar aur Footer mil jayega.
- **Code (`src/app/layout.tsx`):**
```tsx
import type { Metadata } from "next";
import Navbar from "@/components/Navbar";
import Footer from "@/components/Footer";
import "./globals.css";

export const metadata: Metadata = {
  title: "NextStore — Modern E-commerce",
  description: "Browse thousands of high-quality products.",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className="flex min-h-screen flex-col bg-slate-50 text-slate-900 antialiased">
        <Navbar />
        <main className="flex-1">{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```
- **Expected result:** App ke kisi bhi route par Navbar top par aur Footer bottom par render hoga.

---

### Step 4: Products Section Sidebar component banao
- **Kya karna hai:** `src/components/ProductSidebar.tsx` banao jisme categories aur filters hon.
- **Kyu karna hai:** Product-specific navigation ko alag component me organize karne ke liye.
- **Code (`src/components/ProductSidebar.tsx`):**
```tsx
import Link from "next/link";

export default function ProductSidebar() {
  const categories = [
    { name: "All Products", href: "/products" },
    { name: "Electronics", href: "/products?category=electronics" },
    { name: "Footwear", href: "/products?category=footwear" },
    { name: "Apparel", href: "/products?category=apparel" },
    { name: "Accessories", href: "/products?category=accessories" },
  ];

  return (
    <aside className="w-64 shrink-0 rounded-xl border border-slate-200 bg-white p-5 shadow-sm h-fit">
      <h3 className="font-bold text-slate-900 text-lg mb-4">Filter Products</h3>
      <div className="space-y-4">
        <div>
          <h4 className="text-xs font-semibold text-slate-500 uppercase tracking-wider mb-2">Categories</h4>
          <ul className="space-y-2 text-sm">
            {categories.map((cat) => (
              <li key={cat.name}>
                <Link href={cat.href} className="text-slate-700 hover:text-blue-600 block transition">
                  {cat.name}
                </Link>
              </li>
            ))}
          </ul>
        </div>
      </div>
    </aside>
  );
}
```
- **Expected result:** Product category links wala reusable sidebar ready ho jayega.

---

### Step 5: Products Section Layout (`src/app/products/layout.tsx`) create karo
- **Kya karna hai:** `src/app/products/` directory ke andar `layout.tsx` file add karo jo `ProductSidebar` aur `children` ko 2-column grid me render kare.
- **Kyu karna hai:** Ye shared layout products listing (`/products`) aur single product detail (`/products/[id]`) dono me sidebar ko persist karega.
- **Code (`src/app/products/layout.tsx`):**
```tsx
import ProductSidebar from "@/components/ProductSidebar";

export default function ProductsLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="container mx-auto px-4 py-8">
      <div className="flex flex-col md:flex-row gap-8">
        <ProductSidebar />
        <section className="flex-1">{children}</section>
      </div>
    </div>
  );
}
```
- **Expected result:** `/products` aur `/products/1` dono routes par sidebar automatically left side me render hoga.

---

### Step 6: Interactive Client State ko isolate karo (Server vs Client Boundary)
- **Kya karna hai:** Agar sidebar me accordion ya toggle button chahiye, to poore layout ko `'use client'` banane ke bajaye sirf us toggle button ko alag client component banao.
- **Kyu karna hai:** Layout ko Server Component rakhne se initial load performance boost milti hai aur unnecessary JavaScript browser me download nahi hoti.
- **Code snippet (`src/components/SidebarToggle.tsx`):**
```tsx
"use client";

import { useState } from "react";

export default function SidebarToggle() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <button
      onClick={() => setIsOpen(!isOpen)}
      className="md:hidden mb-4 px-3 py-1.5 bg-slate-200 rounded text-sm"
    >
      {isOpen ? "Hide Filters" : "Show Filters"}
    </button>
  );
}
```
- **Expected result:** Client interaction safely encapsulate ho gayi bina parent layout ko client component banaye.

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── products/
│   │   ├── [id]/
│   │   │   └── page.tsx        # Rendered inside ProductsLayout
│   │   ├── layout.tsx          # Nested layout for products section
│   │   └── page.tsx            # Rendered inside ProductsLayout
│   ├── layout.tsx              # Root Layout (Navbar + Footer)
│   └── page.tsx                # Homepage
└── components/
    ├── Footer.tsx              # Reusable footer
    ├── Navbar.tsx              # Reusable header navbar
    ├── ProductSidebar.tsx      # Sidebar for products section
    └── SidebarToggle.tsx       # Isolated client component
```

---

## 5. Common Mistakes / Errors

1. **Product Sidebar ko Root Layout me daal dena:**
   - *Galti:* `src/app/layout.tsx` me sidebar add kar dena jisse wo `/contact`, `/terms` aur home page par bhi dikhne lagta hai.
   - *Fix:* Section-specific UI ko hamesha uske route folder ke layout (`src/app/products/layout.tsx`) me rakho.

2. **Poore Layout ko `"use client"` mark kar dena:**
   - *Galti:* Layout me chhota sa interactive element tha, isliye top par `'use client'` likh diya.
   - *Fix:* Layout ko Server Component rehne do; sirf interactive button/input ko alag component bana kar import karo.

3. **Nested Layout me `<html>` aur `<body>` tags add karna:**
   - *Galti:* `src/app/products/layout.tsx` ke andar dobara `<html>` ya `<body>` return karna.
   - *Fix:* `<html>` aur `<body>` sirf aur sirf root `src/app/layout.tsx` me honge. Nested layouts simple `<div>` ya `<section>` return karte hain.

4. **Layout me `children` prop bhool jana:**
   - *Galti:* Layout component me `{children}` render na karna jisse child pages screen par aate hi nahi hain.
   - *Fix:* Always `{children}` ko layout ke JSX me include karo.

---

## 6. Verification Checklist
- [ ] Navbar har page (Home, Products, Contact, Terms) par visible hai.
- [ ] Footer application ke bottom me reliably render ho raha hai.
- [ ] Products section (`/products` aur `/products/[id]`) me Sidebar visible hai.
- [ ] Home page aur Contact page par Product Sidebar bilkul nahi dikh raha hai.
- [ ] Products listing se product detail page navigate karne par sidebar flicker ya re-render nahi hota.
- [ ] `src/app/layout.tsx` aur `src/app/products/layout.tsx` Server Components hain (top par `'use client'` nahi hai).

---

## 7. Bonus (Extra Practice)
1. **Collapsible Filter Groups:** Sidebar me categories aur brands ke filter groups ko collapse/expand karne ke liye ek interactive Client Component accordion banao.
2. **Dynamic Breadcrumbs in Section Layout:** `src/app/products/layout.tsx` ke top par dynamic breadcrumb navigation add karo jo user ko bataye ki wo Store ke kis section me hai.
