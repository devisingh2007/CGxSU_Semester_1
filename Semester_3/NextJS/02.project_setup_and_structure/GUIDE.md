# Step-by-Step Guide: Assignment 02 — Project Setup & Structure

---

## 1. Assignment ka Goal
Is assignment ka main goal ek naya Next.js E-commerce project create karna hai jo **App Router**, **TypeScript**, aur **Tailwind CSS** par based ho. Isme hume ek clean folder structure (`src/components`, `src/lib`, etc.) tayyar karna hai, `.env` files configure karni hain, aur ek simple E-commerce hero section waala homepage logo ke sath display karna hai bina kisi advanced feature (database ya routing) ke.

---

## 2. Zaroori Concepts (Recap)
Is assignment ko solve karne se pehle ye core concepts samajh lo:
- **App Router**: Next.js ka modern file-system based router jo `src/app/` folder ke andar `page.tsx` aur `layout.tsx` files use karta hai.
- **Import Alias (`@/*`)**: Absolute paths ke liye shortcut jo `src/` ko point karta hai, taaki `../../components` jaise complex paths na likhne padein.
- **Environment Variables**: Sensitive ya configuration data jo code me hardcode nahi hota. Server-only variables private hote hain jabki `NEXT_PUBLIC_` prefix waale variables browser (client) me expose hote hain.

> 📖 Course Notes Reference: Detailed explanation ke liye is folder ki note file [02.project_setup_and_structure.md](./02.project_setup_and_structure.md) zaroor padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Naya Next.js project generate karo
- **Kya karna hai:** Terminal me `create-next-app` command run karke required configuration select karo.
- **Kyu karna hai:** Ye command saari essential dependencies, TypeScript config aur Tailwind setup automatically configure kar deti hai.
- **Code:**
```bash
npx create-next-app@latest ecommerce
```
Prompt aane par ye options choose karo:
```text
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like your code inside a `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to use Turbopack for `next dev`? … Yes / No (default choose karo)
✔ Would you like to customize the import alias (`@/*` by default)? … No
```
- **Expected result:** Ek naya folder `ecommerce` ban jayega aur saare base packages install ho jayenge.

---

### Step 2: Project directory me navigate karke folder structure banao
- **Kya karna hai:** `ecommerce` folder ke andar jaakar `src/` me required directories create karo.
- **Kyu karna hai:** Ek scalable Next.js project me code separation (components, utilities, custom hooks, types) maintain karna zaroori hota hai.
- **Code:**
```bash
cd ecommerce
mkdir src\components src\lib src\hooks src\types public\images
```
*(Windows PowerShell me: `New-Item -ItemType Directory -Path src/components, src/lib, src/hooks, src/types, public/images -Force`)*
- **Expected result:** `src/` ke andar `components`, `lib`, `hooks`, `types` folders aur `public/` ke andar `images` folder create ho jayenge.

---

### Step 3: Logo image add karo
- **Kya karna hai:** `public/images/` folder ke andar ek sample logo image save karo (e.g. `logo.svg` ya `logo.png`).
- **Kyu karna hai:** Next.js `public/` folder ke static assets ko root URL `/images/logo.png` se directly serve karta hai.
- **Code (Simple SVG Logo - `public/images/logo.svg`):**
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 50" width="200" height="50">
  <rect width="200" height="50" rx="10" fill="#2563eb"/>
  <text x="20" y="32" font-family="Arial, sans-serif" font-size="20" font-weight="bold" fill="#ffffff">NextStore</text>
</svg>
```
- **Expected result:** File `public/images/logo.svg` create ho jayegi jise browser se `/images/logo.svg` par access kiya ja sakega.

---

### Step 4: Environment Variables (`.env.local` aur `.env.example`) configure karo
- **Kya karna hai:** Project root par `.env.local` aur `.env.example` files create karo.
- **Kyu karna hai:** Configuration values ko safe rakhne ke liye aur team members ke liye variable list document karne ke liye.
- **Code (`.env.local`):**
```env
# Server-only variables (Private - browser me expose nahi hote)
STORE_NAME="NextStore Flagship"
DATABASE_URL="mongodb://localhost:27017/ecommerce_dev"

# Browser-exposed variables (NEXT_PUBLIC_ prefix zaroori hai)
NEXT_PUBLIC_STORE_NAME="NextStore"
```
- **Code (`.env.example`):**
```env
# Store configuration
STORE_NAME=
DATABASE_URL=

# Public variables
NEXT_PUBLIC_STORE_NAME=
```
- **Expected result:** Project root par dono files create ho jayengi. `.env.local` git me commit nahi hoti (default `.gitignore` me added rehti hai).

---

### Step 5: Homepage (`src/app/page.tsx`) ko E-commerce look do
- **Kya karna hai:** `src/app/page.tsx` ka default code replace karke store name, hero section, description aur logo image render karo.
- **Kyu karna hai:** Homepage par E-commerce store ki pehchan dikhani hai server environment variable aur static asset ko use karke.
- **Code (`src/app/page.tsx`):**
```tsx
import Image from "next/image";

export default function HomePage() {
  const storeName = process.env.STORE_NAME || "NextStore";

  return (
    <main className="min-h-screen bg-slate-50 text-slate-800">
      {/* Header with Logo */}
      <header className="border-b bg-white py-4 shadow-sm">
        <div className="container mx-auto flex items-center justify-between px-4">
          <div className="flex items-center gap-3">
            <Image
              src="/images/logo.svg"
              alt="NextStore Logo"
              width={140}
              height={35}
              priority
            />
          </div>
          <span className="text-sm font-medium text-slate-500">
            Welcome to {storeName}
          </span>
        </div>
      </header>

      {/* Hero Section */}
      <section className="container mx-auto px-4 py-16 text-center">
        <h1 className="text-4xl font-extrabold tracking-tight text-slate-900 sm:text-5xl">
          Discover Quality Products Every Day
        </h1>
        <p className="mx-auto mt-4 max-w-2xl text-lg text-slate-600">
          Your one-stop destination for premium electronics, fashion, and lifestyle essentials.
        </p>
        <div className="mt-8 flex justify-center gap-4">
          <button className="rounded-lg bg-blue-600 px-6 py-3 font-semibold text-white shadow-md hover:bg-blue-700 transition">
            Shop Catalog
          </button>
          <button className="rounded-lg border border-slate-300 bg-white px-6 py-3 font-semibold text-slate-700 hover:bg-slate-100 transition">
            Learn More
          </button>
        </div>
      </section>

      {/* Featured Products Placeholder */}
      <section className="container mx-auto px-4 pb-16">
        <h2 className="text-2xl font-bold text-slate-900 mb-6">Featured Collections</h2>
        <div className="grid grid-cols-1 sm:grid-cols-3 gap-6">
          {["Electronics", "Footwear", "Accessories"].map((item) => (
            <div key={item} className="rounded-xl border border-slate-200 bg-white p-6 shadow-sm text-center">
              <h3 className="font-semibold text-slate-800 text-lg">{item}</h3>
              <p className="text-sm text-slate-500 mt-2">Explore the latest trending items.</p>
            </div>
          ))}
        </div>
      </section>
    </main>
  );
}
```
- **Expected result:** Clean aur responsive homepage design render hoga.

---

### Step 6: Development server start karke check karo
- **Kya karna hai:** Dev server run karo aur browser me `http://localhost:3000` open karo.
- **Kyu karna hai:** Confirm karna ki koi compilation error ya missing asset to nahi hai.
- **Code:**
```bash
npm run dev
```
- **Expected result:** Terminal me output aayega: `Ready in ...ms (http://localhost:3000)` aur browser me logo + hero section perfectly load hoga.

---

## 4. Final File Structure

```text
ecommerce/
├── public/
│   └── images/
│       └── logo.svg
├── src/
│   ├── app/
│   │   ├── favicon.ico
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx           # Updated with store content
│   ├── components/            # Ready for reusable UI
│   ├── hooks/                 # Ready for custom hooks
│   ├── lib/                   # Ready for helper utilities
│   └── types/                 # Ready for TS type definitions
├── .env.example               # Template for environment variables
├── .env.local                 # Local secret values (git ignored)
├── next.config.ts
├── package.json
├── tailwind.config.ts
└── tsconfig.json
```

---

## 5. Common Mistakes / Errors

1. **`NEXT_PUBLIC_` prefix bhool jaana ya galat jagah use karna:**
   - *Galti:* Browser me `process.env.STORE_NAME` access karna chahna (ye `undefined` aayega), ya database secret me `NEXT_PUBLIC_DATABASE_URL` laga dena jo client bundle me expose ho jayega.
   - *Fix:* Server-only variable bina prefix ke rakhein (`DATABASE_URL`), aur agar browser me koi constant chahiye tabhi `NEXT_PUBLIC_` use karein.

2. **Public images ka relative path galat likhna:**
   - *Galti:* `<Image src="../public/images/logo.svg" />` likhna.
   - *Fix:* `public` folder root se serve hota hai, isliye seedha `/images/logo.svg` likho.

3. **Pages Router choose kar lena:**
   - *Galti:* `create-next-app` ke waqt Pages Router select karna jisse `pages/` folder ban jata hai.
   - *Fix:* App Router choose karo taaki `src/app/` structure bane.

4. **Folders ko `src/` ke bahar bana dena:**
   - *Galti:* Root directory par `components/` ya `lib/` banana jabki assignment me `src/` directory specify ki gayi hai.
   - *Fix:* Ensure karo ki `src/components`, `src/lib`, `src/hooks`, `src/types` saare `src/` ke andar hon.

---

## 6. Verification Checklist
- [ ] Project TypeScript, Tailwind CSS aur App Router ke sath initialize hua hai.
- [ ] `src/components`, `src/lib`, `src/hooks`, `src/types` folders mojood hain.
- [ ] `public/images/` me logo image mojood hai aur homepage par load ho rahi hai.
- [ ] `.env.local` aur `.env.example` dono create ho chuke hain.
- [ ] Koi private secret (jaise Database URL) `NEXT_PUBLIC_` ke zariye expose nahi hai.
- [ ] `npm run dev` chalane par browser me `http://localhost:3000` bina kisi console warning ya TS error ke render ho raha hai.

---

## 7. Bonus (Extra Practice)
1. **Metadata update karo:** `src/app/layout.tsx` me jaakar default title `"Create Next App"` ko change karke `"NextStore — Best Online Shopping"` set karo.
2. **Environment variable validation:** `src/lib/env.ts` file banao jisme check karo ki agar `STORE_NAME` missing ho to default value return kare ya clear console warning de.
