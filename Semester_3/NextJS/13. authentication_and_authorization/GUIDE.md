# Step-by-Step Guide: Assignment 13 — Authentication & Authorization

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router application me complete **Authentication & Authorization** architecture implement karna hai using **Auth.js (NextAuth)** aur **MongoDB**. Hume:
- Credentials login (email/password with bcrypt hashing) aur Google OAuth configure karna hai
- Protected `/dashboard` route banana hai (unauthenticated users redirect to `/login`)
- Role-based authorization (`USER` vs `ADMIN`) implement karke `/admin` page restrict karna hai
- Sensitive Server Actions aur Route Handlers ko server-side session checks se secure karna hai.

---

## 2. Zaroori Concepts (Recap)
- **Auth.js (NextAuth v5)**: Next.js App Router ke liye standard authentication solution jo Edge-compatible session management, OAuth providers aur custom credentials support karta hai.
- **Password Hashing (bcrypt)**: Plain text passwords ko database me kabhi store nahi karte; unhe secure salt ke sath hash kiya jata hai.
- **Session vs Token**: App Router me JWT-based session strategy lightweight hoti hai aur server components me `auth()` helper ke zariye instant session data provide karti hai.
- **Server-Side Authorization**: UI me buttons chupana security nahi hota. Har Server Action aur Route Handler ke top par session aur user role verify karna mandatory hai.

> 📖 Course Notes Reference: Auth architecture aur role verification ke liye is folder ki note file [13.authentication_and_authorization.md](./13.authentication_and_authorization.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Dependencies install karo aur Environment Variables configure karo
- **Kya karna hai:** `next-auth` aur `bcryptjs` install karo aur `.env.local` me auth secrets add karo.
- **Kyu karna hai:** Authentication library aur password encryption tools setup karne ke liye.
- **Code (Terminal):**
```bash
npm install next-auth@beta bcryptjs
npm install -D @types/bcryptjs
```
- **Code (`.env.local`):**
```env
AUTH_SECRET="super-secret-random-jwt-key-minimum-32-chars-long"
# Optional Google OAuth Credentials
AUTH_GOOGLE_ID="your-google-client-id"
AUTH_GOOGLE_SECRET="your-google-client-secret"
```
- **Expected result:** Packages install ho jayenge aur auth secrets configure ho jayenge.

---

### Step 2: User Schema & Model with Roles define karo (`src/models/User.ts`)
- **Kya karna hai:** Mongoose schema create karo jisme `name`, `email`, `password` (hashed), aur `role` (`USER` | `ADMIN`) ho.
- **Kyu karna hai:** User accounts aur unke permissions ko MongoDB me store karne ke liye.
- **Code (`src/models/User.ts`):**
```typescript
import mongoose, { Schema, Document, Model } from "mongoose";

export interface IUser extends Document {
  name: string;
  email: string;
  password?: string;
  role: "USER" | "ADMIN";
  createdAt: Date;
}

const UserSchema = new Schema<IUser>(
  {
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true, lowercase: true },
    password: { type: String }, // Hashed password
    role: { type: String, enum: ["USER", "ADMIN"], default: "USER" },
  },
  { timestamps: true }
);

const User: Model<IUser> = mongoose.models.User || mongoose.model<IUser>("User", UserSchema);
export default User;
```
- **Expected result:** Type-safe User model with role support export ho jayega.

---

### Step 3: Auth.js Configuration create karo (`src/auth.ts` aur Route Handler)
- **Kya karna hai:** `src/auth.ts` me Credentials provider define karo aur session callbacks me role attach karo.
- **Kyu karna hai:** User credentials verify karne aur session token me role expose karne ke liye.
- **Code (`src/auth.ts`):**
```typescript
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import Google from "next-auth/providers/google";
import bcrypt from "bcryptjs";
import { connectToDatabase } from "@/lib/mongodb";
import User from "@/models/User";

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Google({
      clientId: process.env.AUTH_GOOGLE_ID,
      clientSecret: process.env.AUTH_GOOGLE_SECRET,
    }),
    Credentials({
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      authorize: async (credentials) => {
        if (!credentials?.email || !credentials?.password) return null;

        await connectToDatabase();
        const user = await User.findOne({ email: credentials.email });
        if (!user || !user.password) return null;

        const isMatch = await bcrypt.compare(credentials.password as string, user.password);
        if (!isMatch) return null;

        return {
          id: user._id.toString(),
          name: user.name,
          email: user.email,
          role: user.role,
        };
      },
    }),
  ],
  callbacks: {
    jwt({ token, user }) {
      if (user) {
        token.role = (user as any).role || "USER";
        token.id = user.id;
      }
      return token;
    },
    session({ session, token }) {
      if (session.user) {
        session.user.role = token.role as "USER" | "ADMIN";
        session.user.id = token.id as string;
      }
      return session;
    },
  },
  pages: {
    signIn: "/login",
  },
});
```
- **Code Route Handler (`src/app/api/auth/[...nextauth]/route.ts`):**
```typescript
import { handlers } from "@/auth";
export const { GET, POST } = handlers;
```
- **Expected result:** NextAuth API endpoints `/api/auth/*` handle hone lagenge.

---

### Step 4: Login UI banao (`src/app/login/page.tsx`)
- **Kya karna hai:** Clean login form create karo jo email aur password accept kare.
- **Kyu karna hai:** Users ko authenticate karne ke liye.
- **Code (`src/app/login/page.tsx`):**
```tsx
import { signIn } from "@/auth";

export default function LoginPage() {
  return (
    <div className="max-w-md mx-auto mt-16 p-6 bg-white rounded-xl shadow-sm border">
      <h1 className="text-2xl font-bold text-center mb-6">Sign In to NextStore</h1>
      <form
        action={async (formData) => {
          "use server";
          await signIn("credentials", formData, { redirectTo: "/dashboard" });
        }}
        className="space-y-4"
      >
        <div>
          <label className="block text-sm font-medium text-slate-700">Email Address</label>
          <input
            name="email"
            type="email"
            required
            className="w-full mt-1 p-2.5 border rounded-lg text-sm"
          />
        </div>
        <div>
          <label className="block text-sm font-medium text-slate-700">Password</label>
          <input
            name="password"
            type="password"
            required
            className="w-full mt-1 p-2.5 border rounded-lg text-sm"
          />
        </div>
        <button
          type="submit"
          className="w-full py-2.5 bg-blue-600 text-white rounded-lg font-semibold hover:bg-blue-700 transition"
        >
          Sign In
        </button>
      </form>
    </div>
  );
}
```
- **Expected result:** Form submit hone par session create hoga aur user dashboard par redirect hoga.

---

### Step 5: Protected Dashboard Route (`src/app/dashboard/page.tsx`) banao
- **Kya karna hai:** `auth()` call karke session check karo; session na hone par redirect karo.
- **Kyu karna hai:** Unauthenticated access prevent karne ke liye.
- **Code (`src/app/dashboard/page.tsx`):**
```tsx
import { auth, signOut } from "@/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();

  if (!session?.user) {
    redirect("/login");
  }

  return (
    <div className="p-8 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold">Welcome, {session.user.name}!</h1>
      <p className="text-slate-600 mt-2">Email: {session.user.email}</p>
      <p className="text-sm font-semibold text-blue-600 mt-1">Role: {session.user.role}</p>

      <form
        action={async () => {
          "use server";
          await signOut({ redirectTo: "/" });
        }}
        className="mt-6"
      >
        <button className="px-4 py-2 bg-slate-200 text-slate-800 rounded font-medium hover:bg-slate-300">
          Sign Out
        </button>
      </form>
    </div>
  );
}
```
- **Expected result:** Unauthenticated user login page par redirect hoga; logged in user ko apna profile dikhega.

---

### Step 6: Admin Route Protection (`src/app/admin/page.tsx`) aur Server Operations Secure karo
- **Kya karna hai:** Check karo ki `session.user.role === 'ADMIN'`. Agar nahi hai to access deny karo.
- **Kyu karna hai:** Regular users ko administrative tools se block karne ke liye.
- **Code (`src/app/admin/page.tsx`):**
```tsx
import { auth } from "@/auth";
import { redirect } from "next/navigation";

export default async function AdminPortalPage() {
  const session = await auth();

  if (!session?.user) redirect("/login");
  if (session.user.role !== "ADMIN") {
    return (
      <div className="p-8 text-center text-red-600 font-bold">
        Access Denied: You need Administrator privileges to view this portal.
      </div>
    );
  }

  return <div className="p-8"><h1 className="text-3xl font-bold">Admin Portal</h1></div>;
}
```
- **Code snippet (Protecting Server Action):**
```typescript
"use server";
import { auth } from "@/auth";

export async function adminDeleteOrder(orderId: string) {
  const session = await auth();
  if (!session || session.user?.role !== "ADMIN") {
    throw new Error("Forbidden: Admin access required");
  }
  // Proceed with deletion...
}
```
- **Expected result:** Client side tampering ke bavajood unauthorized user operation perform nahi kar payega.

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── admin/
│   │   └── page.tsx                 # Restricted to ADMIN role
│   ├── api/
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.ts         # Auth.js request handlers
│   ├── dashboard/
│   │   └── page.tsx                 # Protected route (session check)
│   └── login/
│       └── page.tsx                 # Sign-in form
├── auth.ts                          # NextAuth configuration & callbacks
└── models/
    └── User.ts                      # User schema with role enum
```

---

## 5. Common Mistakes / Errors

1. **Client component se user role trust kar lena:**
   - *Galti:* Client se `{ userId, role }` bhejna aur server action me check karna. Client ise tamper kar sakta hai.
   - *Fix:* Always server par `const session = await auth()` call karke verified session se role padhein.

2. **JWT callback me role pass na karna:**
   - *Galti:* `User` model me role add kiya lekin NextAuth callbacks me token aur session ko map nahi kiya, jisse `session.user.role` undefined aata hai.
   - *Fix:* `src/auth.ts` me `jwt` aur `session` callbacks me explicit mapping karein.

3. **Plain text password compare karna:**
   - *Galti:* `user.password === credentials.password` likhna.
   - *Fix:* Always `bcrypt.compare(credentials.password, user.password)` use karein.

4. **Secret key `.env` me define na karna:**
   - *Galti:* `AUTH_SECRET` missing hona jisse production me JWT verification fail ho jata hai.
   - *Fix:* `.env.local` me strong `AUTH_SECRET` zaroor provide karein.

---

## 6. Verification Checklist
- [ ] Credentials login existing MongoDB user ke hashed password ke sath authenticate hota hai.
- [ ] Invalid password enter karne par login fail hota hai.
- [ ] Logged out state me `/dashboard` par jaane par automatically `/login` par redirect ho raha hai.
- [ ] Regular `USER` account se `/admin` open karne par Access Denied message show hota hai.
- [ ] `ADMIN` account se `/admin` page access hota hai.
- [ ] Sensitive Server Actions bina valid admin session ke reject ho rahi hain.
- [ ] Sign out button click karne par session properly destroy hota hai.

---

## 7. Bonus (Extra Practice)
1. **User Registration Form:** Ek signup page banao jo new users ko `bcrypt.hash(password, 10)` karke MongoDB me default `USER` role ke sath create kare.
2. **Dynamic Navbar User Avatar:** Navbar me user ka name/avatar aur role display karo, aur role ke hisab se `/admin` link toggle karo.
