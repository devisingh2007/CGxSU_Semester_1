# Step-by-Step Guide: Assignment 18 — Proxy & Request Interception

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router ke request-level interception capabilities ko master karna hai using **Proxy / Middleware**. Hume:
- Protected `/dashboard` route par session cookie check karke unauthenticated visitors ko `/login` par redirect karna
- Purane route `/old-products` ko new `/products` route par **Redirect** karna (address bar update hoga)
- Catalog route `/catalog` ko internally `/products` par **Rewrite** karna (address bar me `/catalog` hi rahega lekin content `/products` ka display hoga)
- Matcher config ke zariye unnecessary routes par execution avoid karna.

> [!NOTE]
> Course note me bataya gaya hai: Modern Next.js me is file convention ko `proxy.ts` kaha jata hai, jabki standard Next.js versions me ise `middleware.ts` ke naam se jaana jata hai. Dono same logic follow karte hain. Hum is guide me `proxy.ts` (ya `middleware.ts`) dono support cover karenge.

---

## 2. Zaroori Concepts (Recap)
- **Proxy / Middleware**: Incoming HTTP request jab browser se server par aati hai to kisi bhi page ya API handler tak pahunchne se pehle proxy execute hota hai.
- **Redirect (`NextResponse.redirect`)**: Browser ko naye URL par bhejta hai. Status code 307 ya 308 hota hai aur browser ka address bar update ho jata hai.
- **Rewrite (`NextResponse.rewrite`)**: Internal URL mapping karta hai. User ka address bar wahi rehta hai jo usne type kiya tha, lekin server internally kisi doosre page ka HTML serve karta hai (Masked URL).
- **Lightweight Execution**: Proxy Edge runtime par chalna chahiye. Isme heavy database queries ya complex business logic nahi likhna chahiye; sirf cookie check ya routing headers inspect karni chahiye.
- **Config Matcher**: Regex ya glob pattern jo Next.js ko batata hai ki proxy sirf specific routes par run kare, static assets (`_next/static`, images) ko bypass kare.

> 📖 Course Notes Reference: Proxy architecture aur routing rules ke liye is folder ki note file [18.proxy_request_interception.md](./18.proxy_request_interception.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Proxy / Middleware file create karo (`src/proxy.ts` ya `src/middleware.ts`)
- **Kya karna hai:** `src/` folder ke root par `proxy.ts` (ya `middleware.ts`) file banao.
- **Kyu karna hai:** Next.js request interception pipeline ko register karne ke liye.
- **Code Structure:**
```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  // Logic goes here
  return NextResponse.next();
}

// Older Next.js version fallback
export const middleware = proxy;
```
- **Expected result:** Request interception interceptor initialize ho jayega.

---

### Step 2: Matcher configure karo
- **Kya karna hai:** `config.matcher` define karo jisme sirf wahi paths hon jinko intercept karna hai (`/dashboard/:path*`, `/old-products`, `/catalog`).
- **Kyu karna hai:** Har static image, font aur irrelevant request par proxy run karke unnecessary latency na ho.
- **Code snippet:**
```typescript
export const config = {
  matcher: [
    "/dashboard/:path*",
    "/old-products",
    "/catalog",
  ],
};
```
- **Expected result:** Proxy sirf in 3 targeted paths par hi trigger hoga.

---

### Step 3: Task 1 — Dashboard Route Protection (Cookie Check & Redirect)
- **Kya karna hai:** Check karo ki agar request path `/dashboard` se shuru ho raha hai aur `session_token` cookie missing hai, to user ko `/login` par redirect karo.
- **Kyu karna hai:** Unauthenticated visitors ko dashboard tak pahunchne se pehle hi rokhne ke liye.
- **Code snippet:**
```typescript
const { pathname } = request.nextUrl;

if (pathname.startsWith("/dashboard")) {
  const sessionToken = request.cookies.get("session_token")?.value;

  if (!sessionToken) {
    const loginUrl = new URL("/login", request.url);
    // Optional: redirect back parameter taaki login ke baad wapas dashboard aa sake
    loginUrl.searchParams.set("from", pathname);
    return NextResponse.redirect(loginUrl);
  }
}
```
- **Expected result:** Cookie na hone par user instant `/login?from=/dashboard` par redirect ho jayega.

---

### Step 4: Task 2 — Old Route Redirect (`/old-products` &rarr; `/products`)
- **Kya karna hai:** Agar user `/old-products` visit kare to use permanently `/products` par redirect karo.
- **Kyu karna hai:** Deprecated URLs ko valid new location par route karne ke liye taaki broken links na banein.
- **Code snippet:**
```typescript
if (pathname === "/old-products") {
  return NextResponse.redirect(new URL("/products", request.url), 308); // 308 Permanent Redirect
}
```
- **Expected result:** Browser address bar me `/old-products` gayab hokar `/products` ban jayega.

---

### Step 5: Task 3 — Catalog Route Rewrite (`/catalog` &rarr; `/products`)
- **Kya karna hai:** Agar user `/catalog` visit kare to internal rewrite rule ke zariye `/products` ka page serve karo.
- **Kyu karna hai:** Clean marketing URL maintain karne ke liye jaha address bar `/catalog` hi dikhaye lekin backend par code `/products` ka execute ho.
- **Code snippet:**
```typescript
if (pathname === "/catalog") {
  return NextResponse.rewrite(new URL("/products", request.url));
}
```
- **Expected result:** Address bar me `/catalog` rahega lekin screen par `/products` ka content dikhega!

---

### Step 6: Complete Combined Code assemble karo (`src/proxy.ts` / `src/middleware.ts`)
- **Kya karna hai:** Saari conditions ko ek clean pipeline me combine karo aur end me `NextResponse.next()` return karo.
- **Code:**
```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // 1. Task 1: Protect Dashboard with cookie check
  if (pathname.startsWith("/dashboard")) {
    const sessionToken = request.cookies.get("session_token")?.value;
    if (!sessionToken) {
      const loginUrl = new URL("/login", request.url);
      loginUrl.searchParams.set("from", pathname);
      return NextResponse.redirect(loginUrl);
    }
  }

  // 2. Task 2: Redirect legacy route
  if (pathname === "/old-products") {
    return NextResponse.redirect(new URL("/products", request.url), 308);
  }

  // 3. Task 3: Rewrite catalog URL to products page
  if (pathname === "/catalog") {
    return NextResponse.rewrite(new URL("/products", request.url));
  }

  // Allow all other matching requests to continue normally
  return NextResponse.next();
}

// Export middleware as alias for standard compatibility
export const middleware = proxy;

export const config = {
  matcher: [
    "/dashboard/:path*",
    "/old-products",
    "/catalog",
  ],
};
```
- **Expected result:** Single file se teeno rules safely govern honge.

---

### Step 7: Testing & Verification
- **Kya karna hai:**
  1. Browser me `http://localhost:3000/dashboard` open karo -> Address bar `/login?from=/dashboard` ban jana chahiye.
  2. DevTools Application tab me jakar Cookie add karo: Name: `session_token`, Value: `valid-token-123`.
  3. Ab dobara `http://localhost:3000/dashboard` visit karo -> Dashboard page cleanly open hona chahiye!
  4. Address bar me `http://localhost:3000/old-products` enter karo -> URL automatically `/products` ban jana chahiye.
  5. Address bar me `http://localhost:3000/catalog` enter karo -> URL `/catalog` hi rahega lekin `/products` ka page render hoga.
- **Expected result:** All interception rules successfully tested!

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── dashboard/
│   │   └── page.tsx        # Protected page
│   ├── login/
│   │   └── page.tsx        # Login page
│   └── products/
│       └── page.tsx        # Catalog page
├── proxy.ts                # (Or middleware.ts) Request interceptor
```

---

## 5. Common Mistakes / Errors

1. **Proxy cookie check ko actual security samajh lena:**
   - *Galti:* Proxy me cookie presence check karke sochna ki API aur Server Actions secure ho gaye. Koi bhi user DevTools me fake cookie set kar sakta hai.
   - *Fix:* Proxy sirf lightweight user-routing check hai. Actual database aur Server Actions me secure JWT verification aur role checks hamesha hone chahiye.

2. **Redirect me absolute URL na banana:**
   - *Galti:* `NextResponse.redirect('/login')` likhna. Next.js error throw karega: `URL must be absolute`.
   - *Fix:* Always `new URL('/login', request.url)` use karein.

3. **Rewrite aur Redirect ke fark ko confuse karna:**
   - *Galti:* Rewrite me address bar change hone ki umeed karna.
   - *Fix:* Redirect address bar change karta hai (external navigation), rewrite address bar same rakh kar internal routing change karta hai.

4. **Matcher configure na karna:**
   - *Galti:* Matcher omit kar dena jisse static images, chunks aur favicons par bhi proxy run hota hai aur page slow ho jata hai.
   - *Fix:* Always selective `config.matcher` use karein.

---

## 6. Verification Checklist
- [ ] Cookie missing hone par `/dashboard` visit karne par user `/login` par redirect hota hai.
- [ ] Cookie present hone par `/dashboard` request allow hoti hai aur page dikhta hai.
- [ ] `/old-products` visit karne par address bar change hokar `/products` display karta hai.
- [ ] `/catalog` visit karne par address bar `/catalog` rehta hai lekin `/products` ka content display hota hai.
- [ ] Static images aur assets par proxy unnecessarily execute nahi hota.

---

## 7. Bonus (Extra Practice)
1. **Custom Request Headers Injection:** Rewrite karte waqt request headers me custom tracking header add karo: `requestHeaders.set('x-rewritten-from', 'catalog')`.
2. **Geo-Location Based Redirect:** Request header `x-vercel-ip-country` inspect karke specific country ke user ko currency/locale route par redirect karne ka simulation karo.
