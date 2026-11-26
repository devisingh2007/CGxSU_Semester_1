# Step-by-Step Guide: Assignment 03 — Routing & Navigation

---

## 1. Assignment ka Goal
Is assignment ka goal Next.js App Router ke routing capabilities ko master karna hai. Isme hume static routes (`/products`, `/categories`), nested routes, dynamic product pages (`/products/[id]`), search query parameters, catch-all routes (`/shop/[...slug]`), optional catch-all routes (`/store/[[...slug]]`), route groups (`(store)`, `(support)`), active links ke sath navigation, aur programmatic redirects implement karne hain.

---

## 2. Zaroori Concepts (Recap)
- **Folder-based Routing**: Har folder ek URL segment banta hai aur uske andar `page.tsx` file hone par hi wo route publicly accessible hota hai.
- **Dynamic Routes `[id]`**: Square brackets dynamic parameters ke liye use hote hain (e.g., `app/products/[id]/page.tsx` match karega `/products/123`).
- **Catch-All `[...slug]` vs Optional `[[...slug]]`**:
  - `[...slug]`: 1 ya 1 se zyada segments match karta hai (e.g. `/shop/a/b`). Base path `/shop` match nahi karta.
  - `[[...slug]]`: 0 ya zyada segments match karta hai (base `/store` aur nested `/store/a/b` dono match karega).
- **Route Groups `(folder)`**: Parantheses `(group)` lagane se code organize ho jata hai lekin group ka naam URL me include nahi hota.
- **Client Navigation (`<Link>`, `usePathname`, `useRouter`)**: Page reload kiye bina fast navigation deta hai.
- **Server Redirect (`redirect`)**: Server Component me kisi condition (jaise login check) par user ko doosre route par bhejta hai.

> 📖 Course Notes Reference: Detailed routing explanation ke liye is folder ki note file [03.routing_and_navigation.md](./03.routing_and_navigation.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Static Routes create karo
- **Kya karna hai:** `src/app/` ke andar folders banao: `contact`, `terms`, aur unke andar `page.tsx` add karo.
- **Kyu karna hai:** Har static page ka ek unique URL path hota hai jo direct information render karta hai.
- **Code (`src/app/contact/page.tsx`):**
```tsx
export default function ContactPage() {
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold">Contact Us</h1>
      <p className="mt-2 text-slate-600">Email: support@nextstore.com</p>
    </div>
  );
}
```
*(Similarly `src/app/terms/page.tsx` me Terms of Service ka simple content likho).*
- **Expected result:** `/contact` aur `/terms` browse karne par respective page dikhega.

---

### Step 2: Nested Category Routes banao
- **Kya karna hai:** Folder hierarchy create karo: `src/app/categories/electronics/mobile/page.tsx`.
- **Kyu karna hai:** Multi-level URL structure ko Next.js automatically nested folders ke through represent karta hai.
- **Code (`src/app/categories/electronics/mobile/page.tsx`):**
```tsx
export default function MobileCategoryPage() {
  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold">Category: Electronics &gt; Mobile Phones</h1>
      <p className="mt-2 text-slate-600">Browse all smartphones and mobile accessories.</p>
    </div>
  );
}
```
- **Expected result:** Browser me `http://localhost:3000/categories/electronics/mobile` visit karne par page render hoga.

---

### Step 3: Dynamic Product Route (`/products/[id]`) create karo
- **Kya karna hai:** `src/app/products/[id]/page.tsx` file banao aur route parameter ko read karke display karo.
- **Kyu karna hai:** Dynamic segments URL se variable values (jaise product ID) nikaalne ke kaam aate hain. Next.js 15+ me `params` ek Promise hota hai isliye `await params` use karte hain.
- **Code (`src/app/products/[id]/page.tsx`):**
```tsx
interface ProductDetailProps {
  params: Promise<{ id: string }>;
}

export default async function ProductDetailPage({ params }: ProductDetailProps) {
  const { id } = await params;

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold">Product Details</h1>
      <p className="mt-4 text-lg">
        Viewing details for Product ID: <span className="font-mono bg-blue-100 text-blue-800 px-2 py-1 rounded">{id}</span>
      </p>
    </div>
  );
}
```
- **Expected result:** `/products/p-101` ya `/products/macbook-pro` visit karne par wahi ID screen par display hogi.

---

### Step 4: Search Parameters handle karo (`/products`)
- **Kya karna hai:** `src/app/products/page.tsx` me `searchParams` prop read karo (`search`, `page`, `sort`).
- **Kyu karna hai:** URL query string (e.g. `?search=shoes&page=2&sort=asc`) se filters aur pagination read karne ke liye.
- **Code (`src/app/products/page.tsx`):**
```tsx
interface ProductsPageProps {
  searchParams: Promise<{
    search?: string;
    page?: string;
    sort?: string;
  }>;
}

export default async function ProductsPage({ searchParams }: ProductsPageProps) {
  const params = await searchParams;
  const search = params.search || "All Products";
  const page = params.page || "1";
  const sort = params.sort || "default";

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold">Product Catalog</h1>
      <div className="mt-4 bg-slate-100 p-4 rounded-lg space-y-2 text-sm">
        <p><strong>Query:</strong> {search}</p>
        <p><strong>Page:</strong> {page}</p>
        <p><strong>Sort By:</strong> {sort}</p>
      </div>
    </div>
  );
}
```
- **Expected result:** `/products?search=laptop&page=2&sort=price_asc` visit karne par values screen par reflect hongi.

---

### Step 5: Catch-All Route banao (`/shop/[...slug]`)
- **Kya karna hai:** Folder banao `src/app/shop/[...slug]/page.tsx`.
- **Kyu karna hai:** Kisi bhi depth ke nested URL path (e.g. `/shop/electronics/audio/wireless`) ko ek hi page me handle karne ke liye.
- **Code (`src/app/shop/[...slug]/page.tsx`):**
```tsx
interface ShopCatchAllProps {
  params: Promise<{ slug: string[] }>;
}

export default async function ShopCatchAllPage({ params }: ShopCatchAllProps) {
  const { slug } = await params;

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold">Shop Hierarchy</h1>
      <p className="mt-2 text-slate-600">Active Path Segments:</p>
      <ul className="list-disc ml-6 mt-2 space-y-1">
        {slug.map((segment, index) => (
          <li key={index} className="capitalize font-medium">{segment}</li>
        ))}
      </ul>
    </div>
  );
}
```
- **Expected result:** `/shop/electronics/smartphones/android` par teeno segments list format me display honge.

---

### Step 6: Optional Catch-All Route banao (`/store/[[...slug]]`)
- **Kya karna hai:** Folder banao `src/app/store/[[...slug]]/page.tsx`.
- **Kyu karna hai:** Double brackets `[[...]]` lagane se root `/store` bhi match karta hai bina extra `page.tsx` banaye.
- **Code (`src/app/store/[[...slug]]/page.tsx`):**
```tsx
interface StoreCatchAllProps {
  params: Promise<{ slug?: string[] }>;
}

export default async function StorePage({ params }: StoreCatchAllProps) {
  const { slug } = await params;

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold">Store Main Hub</h1>
      {slug && slug.length > 0 ? (
        <p className="mt-2 text-slate-700">Sub-sections: {slug.join(" &gt; ")}</p>
      ) : (
        <p className="mt-2 text-slate-500">Welcome to Root Store! No sub-path specified.</p>
      )}
    </div>
  );
}
```
- **Expected result:** `/store` aur `/store/men/shoes` dono routes bina 404 error ke render honge.

---

### Step 7: Route Groups apply karo (`(store)` aur `(support)`)
- **Kya karna hai:** Logical grouping ke liye `src/app/(store)` aur `src/app/(support)` folders banao.
- **Kyu karna hai:** Routes ko structure aur clean rakhne ke liye bina URL me group name dikhaye.
- **Structure:**
  - `src/app/(store)/categories/page.tsx` → URL banega `/categories`
  - `src/app/(support)/contact/page.tsx` → URL banega `/contact`
- **Expected result:** URL me `(store)` ya `(support)` nahi aayega, direct `/categories` aur `/contact` chalega.

---

### Step 8: Active Link Navigation component banao
- **Kya karna hai:** `src/components/Navigation.tsx` banao `'use client'` ke sath jo current pathname match karke active link ko highlight kare.
- **Kyu karna hai:** Fast client-side navigation provide karne aur user ko batane ke liye ki wo kis page par hai.
- **Code (`src/components/Navigation.tsx`):**
```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";

export default function Navigation() {
  const pathname = usePathname();

  const navLinks = [
    { name: "Home", href: "/" },
    { name: "Products", href: "/products" },
    { name: "Categories", href: "/categories" },
    { name: "Contact", href: "/contact" },
  ];

  return (
    <nav className="flex gap-4 p-4 bg-slate-900 text-white">
      {navLinks.map((link) => {
        const isActive = pathname === link.href;
        return (
          <Link
            key={link.href}
            href={link.href}
            className={`px-3 py-1 rounded transition ${
              isActive ? "bg-blue-600 font-bold" : "hover:bg-slate-800"
            }`}
          >
            {link.name}
          </Link>
        );
      })}
    </nav>
  );
}
```
- **Expected result:** Jo link active hoga wo blue background me highlighted dikhega.

---

### Step 9: Programmatic Navigation implement karo
- **Kya karna hai:** Client buttons banao jo `useRouter().back()` aur `useRouter().push()` use karein.
- **Kyu karna hai:** Buttons ya event handlers (jaise form submit hone ke baad) se user ko navigate karwane ke liye.
- **Code snippet (Product Details page par Back Button):**
```tsx
"use client";

import { useRouter } from "next/navigation";

export function BackButton() {
  const router = useRouter();
  return (
    <button
      onClick={() => router.back()}
      className="px-4 py-2 bg-slate-200 text-slate-800 rounded hover:bg-slate-300 transition"
    >
      &larr; Back to previous page
    </button>
  );
}
```
- **Expected result:** Click karne par browser instantly previous page par wapas chala jayega.

---

### Step 10: Server-Side Redirect implement karo (`/dashboard`)
- **Kya karna hai:** `src/app/dashboard/page.tsx` banao jisme check karo ki user logged in hai ya nahi, aur na hone par `redirect('/login')` trigger karo.
- **Kyu karna hai:** Unauthenticated users ko server-level par protect karke login page bhejna.
- **Code (`src/app/dashboard/page.tsx`):**
```tsx
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const isLoggedIn = false; // Simulated condition

  if (!isLoggedIn) {
    redirect("/login");
  }

  return <div className="p-8">Secret Dashboard Content</div>;
}
```
*(Aur ek simple `src/app/login/page.tsx` bana lo taaki redirect successful ho).*
- **Expected result:** `/dashboard` visit karne par URL turant `/login` me change ho jayega.

---

## 4. Final File Structure

```text
src/app/
├── (store)/
│   ├── categories/
│   │   ├── page.tsx
│   │   └── electronics/
│   │       └── mobile/
│   │           └── page.tsx
│   ├── products/
│   │   ├── [id]/
│   │   │   └── page.tsx
│   │   └── page.tsx
│   ├── shop/
│   │   └── [...slug]/
│   │       └── page.tsx
│   └── store/
│       └── [[...slug]]/
│           └── page.tsx
├── (support)/
│   ├── contact/
│   │   └── page.tsx
│   └── terms/
│       └── page.tsx
├── dashboard/
│   └── page.tsx             # Server redirect demo
├── login/
│   └── page.tsx
├── layout.tsx
└── page.tsx
src/components/
└── Navigation.tsx           # Active link nav bar
```

---

## 5. Common Mistakes / Errors

1. **Next.js 15 me `params` ya `searchParams` ko await na karna:**
   - *Galti:* `const { id } = props.params;` likhne se compilation warning ya runtime error aayega.
   - *Fix:* `const { id } = await params;` use karein kyunki modern App Router me params asynchronous Promise hote hain.

2. **Standard HTML `<a>` tag use karna `<Link>` ke badle:**
   - *Galti:* `<a href="/products">Products</a>` likhna jisse poora page refresh ho jata hai.
   - *Fix:* Always `import Link from "next/link"` use karo taaki fast client-side transition ho.

3. **`useRouter` ya `usePathname` ko Server Component me call karna:**
   - *Galti:* Server component file ke top par `'use client'` likhe bina hooks call karna.
   - *Fix:* Navigation hooks sirf Client Components me kaam karte hain. File ke top par `"use client";` add karein.

4. **`redirect()` ko broad `try/catch` block ke andar wrap karna:**
   - *Galti:* `try { redirect('/login') } catch(err) { ... }` likhna. Next.js redirect ek internal NEXT_REDIRECT error throw karke kaam karta hai.
   - *Fix:* `redirect()` ko `try/catch` ke bahar call karein ya error type check karein.

---

## 6. Verification Checklist
- [ ] Static routes `/contact`, `/terms`, `/products`, `/categories` kaam kar rahe hain.
- [ ] Nested route `/categories/electronics/mobile` load ho raha hai.
- [ ] Dynamic route `/products/42` product ID screen par render kar raha hai.
- [ ] `/products?search=shoes&page=3` parameters ko parse karke display kar raha hai.
- [ ] Catch-all route `/shop/a/b/c` multiple segments print kar raha hai.
- [ ] Optional catch-all route `/store` aur `/store/deals` dono chal rahe hain.
- [ ] Route groups `(store)` aur `(support)` URL me expose nahi ho rahe hain.
- [ ] Nav links me active page highlight ho raha hai.
- [ ] Back button programmatic navigation operate kar raha hai.
- [ ] `/dashboard` par jaane par automatically `/login` par redirect ho raha hai.

---

## 7. Bonus (Extra Practice)
1. **Dynamic Active Class Helpers:** Ek helper function likho jo parent routes (e.g. `/products` match even when on `/products/123`) ko bhi active mark kare using `pathname.startsWith()`.
2. **Breadcrumb Component:** Catch-all route `[...slug]` ke segments ko use karke clickable breadcrumb navigation (`Home > Shop > Electronics > Mobile`) render karo.
