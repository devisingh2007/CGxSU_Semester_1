# Step-by-Step Guide: Assignment 12 — Database Integration with MongoDB & Mongoose

---

## 1. Assignment ka Goal
Is assignment ka main objective temporary in-memory data store ko replace karke **MongoDB** ko as a persistent database integrate karna hai using **Mongoose**. Hume:
- Next.js development hot-reloading ke liye cached connection helper (`lib/mongodb.ts`) banana hai
- `Product` Mongoose model (`models/Product.ts`) create karna hai with validation aur timestamps
- `app/api/products/route.ts` Route Handler me MongoDB se `GET` aur `POST` implement karna hai
- Server Action me product save karke `revalidatePath('/products')` trigger karna hai
- Development server restart ke baad bhi products persist rehne chahiye.

---

## 2. Zaroori Concepts (Recap)
- **Mongoose**: MongoDB ke liye object data modeling (ODM) library jo schema validation, casting aur business logic provide karti hai.
- **Connection Caching in Next.js**: Development me file change hone par Next.js module re-execute karta hai. Agar connection global cache me save na ho to hazaron open database connections create ho jaate hain.
- **Model Recompilation Prevention**: Next.js me model define karte waqt `mongoose.models.Product || mongoose.model('Product', ProductSchema)` likhna padta hai taaki `OverwriteModelError` na aaye.
- **Server Action vs Route Handler with DB**: Dono server environment me run hote hain aur safe tareeqe se directly Mongoose models ko query/mutate kar sakte hain.

> 📖 Course Notes Reference: Detailed MongoDB connection architecture ke liye is folder ki note file [12.database_mongodb_and_mongoose.md](./12.database_mongodb_and_mongoose.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Mongoose install karo aur `.env.local` configure karo
- **Kya karna hai:** Project me `mongoose` package install karo aur MongoDB connection string set karo.
- **Kyu karna hai:** Database communication library add karne aur credentials ko safe environment variable me store karne ke liye.
- **Code (Terminal):**
```bash
npm install mongoose
```
- **Code (`.env.local`):**
```env
# MongoDB Connection String (Local Compass ya Atlas URL)
MONGODB_URI="mongodb://localhost:27017/nextjs_ecommerce"
```
- **Expected result:** `package.json` me mongoose dependency add ho jayegi aur `.env.local` me connection string configure ho jayegi.

---

### Step 2: Cached MongoDB Connection Helper banao (`src/lib/mongodb.ts`)
- **Kya karna hai:** Reusable `connectToDatabase()` function create karo jo global cache check kare.
- **Kyu karna hai:** Hot-reloads ke dauran duplicate database connections ko prevent karne ke liye.
- **Code (`src/lib/mongodb.ts`):**
```typescript
import mongoose from "mongoose";

const MONGODB_URI = process.env.MONGODB_URI;

if (!MONGODB_URI) {
  throw new Error("Please define the MONGODB_URI environment variable inside .env.local");
}

interface MongooseCache {
  conn: typeof mongoose | null;
  promise: Promise<typeof mongoose> | null;
}

// Global cache variable define karo (TypeScript support ke sath)
declare global {
  var mongooseCache: MongooseCache | undefined;
}

let cached = global.mongooseCache;

if (!cached) {
  cached = global.mongooseCache = { conn: null, promise: null };
}

export async function connectToDatabase(): Promise<typeof mongoose> {
  if (cached!.conn) {
    return cached!.conn;
  }

  if (!cached!.promise) {
    const opts = {
      bufferCommands: false,
    };

    cached!.promise = mongoose.connect(MONGODB_URI!, opts).then((mongooseInstance) => {
      console.log("=> Connected to MongoDB successfully");
      return mongooseInstance;
    });
  }

  try {
    cached!.conn = await cached!.promise;
  } catch (e) {
    cached!.promise = null;
    throw e;
  }

  return cached!.conn;
}
```
- **Expected result:** Single shared database connection maintain hoga.

---

### Step 3: Product Schema & Model create karo (`src/models/Product.ts`)
- **Kya karna hai:** Product Mongoose schema define karo with required fields (`name`, `price`, `stock`, timestamps) aur model compile karo.
- **Kyu karna hai:** MongoDB documents ke structure aur validation rules ko enforce karne ke liye.
- **Code (`src/models/Product.ts`):**
```typescript
import mongoose, { Schema, Document, Model } from "mongoose";

export interface IProduct extends Document {
  name: string;
  description?: string;
  price: number;
  stock: number;
  createdAt: Date;
  updatedAt: Date;
}

const ProductSchema = new Schema<IProduct>(
  {
    name: {
      type: String,
      required: [true, "Product name is required"],
      trim: true,
    },
    description: {
      type: String,
      trim: true,
      default: "",
    },
    price: {
      type: Number,
      required: [true, "Price is required"],
      min: [0, "Price cannot be negative"],
    },
    stock: {
      type: Number,
      default: 0,
      min: [0, "Stock cannot be negative"],
    },
  },
  {
    timestamps: true, // Automatically manages createdAt and updatedAt
  }
);

// Prevent recompilation in Next.js development
const Product: Model<IProduct> =
  mongoose.models.Product || mongoose.model<IProduct>("Product", ProductSchema);

export default Product;
```
- **Expected result:** `Product` model export ho jayega jo Next.js hot-reload par duplicate error nahi dega.

---

### Step 4: MongoDB-backed Route Handler (`src/app/api/products/route.ts`) implement karo
- **Kya karna hai:** `GET` (fetch all products from MongoDB) aur `POST` (create new product in DB) implement karo.
- **Kyu karna hai:** REST API clients aur client-side frontend ko database data provide karne ke liye.
- **Code (`src/app/api/products/route.ts`):**
```typescript
import { NextRequest, NextResponse } from "next/server";
import { connectToDatabase } from "@/lib/mongodb";
import Product from "@/models/Product";

// GET /api/products
export async function GET() {
  try {
    await connectToDatabase();
    const products = await Product.find({}).sort({ createdAt: -1 }).lean();
    return NextResponse.json(products, { status: 200 });
  } catch (error) {
    console.error("Database fetch error:", error);
    return NextResponse.json({ error: "Failed to fetch products" }, { status: 500 });
  }
}

// POST /api/products
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    await connectToDatabase();

    const product = await Product.create({
      name: body.name,
      description: body.description,
      price: Number(body.price),
      stock: Number(body.stock || 0),
    });

    return NextResponse.json(product, { status: 201 });
  } catch (error: any) {
    console.error("Database create error:", error);
    return NextResponse.json(
      { error: error.message || "Failed to create product" },
      { status: 400 }
    );
  }
}
```
- **Expected result:** Route handler real MongoDB database me data read aur write karega.

---

### Step 5: Server Action me Mongoose integration aur Revalidation implement karo
- **Kya karna hai:** `src/actions/product.ts` me `createProductDBAction` define karo jo Mongoose model se document save kare aur `revalidatePath('/products')` call kare.
- **Kyu karna hai:** Form submissions ko directly database me save karke instant UI update dekhne ke liye.
- **Code (`src/actions/product.ts`):**
```typescript
"use server";

import { revalidatePath } from "next/cache";
import { connectToDatabase } from "@/lib/mongodb";
import Product from "@/models/Product";

export async function createProductDBAction(formData: FormData) {
  try {
    await connectToDatabase();

    const name = formData.get("name") as string;
    const description = (formData.get("description") as string) || "";
    const price = parseFloat(formData.get("price") as string);
    const stock = parseInt((formData.get("stock") as string) || "0", 10);

    if (!name || isNaN(price)) {
      return { success: false, message: "Name and valid Price are required" };
    }

    await Product.create({
      name,
      description,
      price,
      stock,
    });

    revalidatePath("/products");
    return { success: true, message: "Product saved to database successfully!" };
  } catch (error: any) {
    return { success: false, message: error.message || "Database operation failed" };
  }
}
```
- **Expected result:** Server Action se product MongoDB me insert hoga aur products page revalidate ho jayega.

---

### Step 6: Database Persistence verify karo
- **Kya karna hai:** Ek naya product form ya API ke through create karo, Next.js terminal restart karo (`Ctrl+C` then `npm run dev`), aur page refresh karo.
- **Kyu karna hai:** Confirm karna ki data memory me nahi balki actual persistent database me save ho raha hai.
- **Expected result:** Server restart ke baad bhi naya create kiya hua product screen par display hoga aur MongoDB Compass/Atlas me document visible hoga.

---

## 4. Final File Structure

```text
src/
├── actions/
│   └── product.ts            # Server Action calling connectToDatabase() & Product.create()
├── app/
│   ├── api/
│   │   └── products/
│   │       └── route.ts       # Route Handler querying MongoDB
│   └── products/
│       └── page.tsx           # Product catalog displaying MongoDB products
├── lib/
    └── mongodb.ts             # Cached Mongoose connection helper
models/
└── Product.ts                 # Mongoose Product Schema & Model
.env.local                     # MONGODB_URI connection string
```

---

## 5. Common Mistakes / Errors

1. **`MONGODB_URI` me `NEXT_PUBLIC_` laga dena:**
   - *Galti:* `NEXT_PUBLIC_MONGODB_URI` likhna. Isse database username, password aur cluster URL browser ke network tab aur JS bundle me leak ho jayenge.
   - *Fix:* Sirf `MONGODB_URI` rakhein taaki ye strictly server-side par rahe.

2. **`OverwriteModelError: Cannot overwrite 'Product' model once compiled`:**
   - *Galti:* `export default mongoose.model('Product', ProductSchema);` likhna bina existing models check kiye.
   - *Fix:* Hamesha `mongoose.models.Product || mongoose.model('Product', ProductSchema)` use karein.

3. **`connectToDatabase()` ko call karna bhool jaana:**
   - *Galti:* Route handler ya server action me `Product.find()` seedha call karna bina connection open kiye.
   - *Fix:* Database query karne se pehle `await connectToDatabase()` zaroor call karein.

4. **Connection caching na karna:**
   - *Galti:* Har function call par naya `mongoose.connect()` banana jisse development me "Too many connections" ka error aata hai.
   - *Fix:* `src/lib/mongodb.ts` me global cache pattern zaroor follow karein.

---

## 6. Verification Checklist
- [ ] `mongoose` installed hai aur `MONGODB_URI` configured hai.
- [ ] `src/lib/mongodb.ts` connection caching implement karta hai.
- [ ] `models/Product.ts` me required validation aur timestamps configured hain.
- [ ] `GET /api/products` MongoDB se documents return kar raha hai.
- [ ] `POST /api/products` MongoDB me naya document insert kar raha hai.
- [ ] Server Action se form submit karne par database me product store hota hai.
- [ ] Dev server restart karne ke baad bhi create kiye gaye products persist rehte hain.
- [ ] MongoDB Compass ya Atlas me collection aur documents inspect ho rahe hain.

---

## 7. Bonus (Extra Practice)
1. **Search Query in MongoDB:** `GET /api/products?search=phone` handle karne ke liye Mongoose me regex search implement karo: `Product.find({ name: { $regex: search, $options: "i" } })`.
2. **MongoDB Connection Status Indicator:** Ek chhota health utility function banao jo `mongoose.connection.readyState` return kare (0 = disconnected, 1 = connected).
