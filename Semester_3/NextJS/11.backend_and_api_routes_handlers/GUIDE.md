# Step-by-Step Guide: Assignment 11 — Backend & API Route Handlers

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router ke **Route Handlers** (`route.ts`) ke through ek complete RESTful API architecture build karna hai. Hume:
- Collection endpoint `app/api/products/route.ts` (`GET` with category filter, `POST` with validation and status 201)
- Single item endpoint `app/api/products/[id]/route.ts` (`GET`, `PUT`, `PATCH`, `DELETE` with 204 No Content)
- Shared product service `lib/services/product.service.ts` aur Zod schema `lib/validations/product.schema.ts`
- Async dynamic params (`await context.params`), proper HTTP status codes, error handling, aur frontend UI connection implement karna hai.

---

## 2. Zaroori Concepts (Recap)
- **Route Handlers (`route.ts`)**: App Router ka replacement for Pages Router `pages/api`. Standard Web `Request` aur `Response` objects use karta hai.
- **Supported HTTP Methods**: Exported functions `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`.
- **Parsing Request Body**: `const body = await request.json()`.
- **Dynamic Route Params**: Next.js 15+ me `context.params` ek Promise hota hai: `const { id } = await context.params`.
- **HTTP Status Codes**:
  - `200 OK` (Standard success)
  - `201 Created` (Resource created)
  - `204 No Content` (Successful deletion without body)
  - `400 Bad Request` (Validation or malformed JSON)
  - `404 Not Found` (Missing resource)
  - `500 Internal Server Error` (Unexpected server issue)

> 📖 Course Notes Reference: Route handler APIs and best practices ke liye is folder ki note file [11.backend_and_api_routes_handlers.md](./11.backend_and_api_routes_handlers.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Shared Product Validation Schema define karo (`src/lib/validations/product.schema.ts`)
- **Kya karna hai:** Zod schemas create karo full product validation (`POST`/`PUT`) aur partial validation (`PATCH`) ke liye.
- **Kyu karna hai:** API endpoints par request payload ko strictly validate karne ke liye.
- **Code (`src/lib/validations/product.schema.ts`):**
```typescript
import { z } from "zod";

export const createProductSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  description: z.string().optional(),
  price: z.number().positive("Price must be greater than 0"),
  category: z.string().min(2, "Category is required"),
  stock: z.number().int().min(0, "Stock cannot be negative").default(0),
});

export const updateProductSchema = createProductSchema; // For PUT (full replacement)
export const patchProductSchema = createProductSchema.partial(); // For PATCH (optional fields)

export type CreateProductDTO = z.infer<typeof createProductSchema>;
export type PatchProductDTO = z.infer<typeof patchProductSchema>;
```
- **Expected result:** Type-safe DTOs aur validation helpers ready ho jayenge.

---

### Step 2: Shared Product Service create karo (`src/lib/services/product.service.ts`)
- **Kya karna hai:** Data storage aur business logic ko Route Handlers se alag rakhne ke liye service layer banao.
- **Kyu karna hai:** Code duplication prevent karne ke liye taaki saari API files ek hi data layer share karein.
- **Code (`src/lib/services/product.service.ts`):**
```typescript
import { CreateProductDTO, PatchProductDTO } from "../validations/product.schema";

export interface ProductModel {
  id: string;
  name: string;
  description?: string;
  price: number;
  category: string;
  stock: number;
  createdAt: string;
}

// Global in-memory dataset to preserve across hot-reloads
const globalStore = global as unknown as { productsDB: ProductModel[] };
if (!globalStore.productsDB) {
  globalStore.productsDB = [
    {
      id: "1",
      name: "Mechanical Keyboard",
      description: "RGB backlit mechanical keyboard with blue switches.",
      price: 89.99,
      category: "electronics",
      stock: 25,
      createdAt: new Date().toISOString(),
    },
    {
      id: "2",
      name: "Ergonomic Mouse",
      description: "Wireless vertical mouse designed for wrist comfort.",
      price: 49.99,
      category: "electronics",
      stock: 40,
      createdAt: new Date().toISOString(),
    },
  ];
}

const products = globalStore.productsDB;

export const ProductService = {
  getAll(category?: string): ProductModel[] {
    if (category) {
      return products.filter((p) => p.category.toLowerCase() === category.toLowerCase());
    }
    return products;
  },

  getById(id: string): ProductModel | null {
    return products.find((p) => p.id === id) || null;
  },

  create(data: CreateProductDTO): ProductModel {
    const newProduct: ProductModel = {
      id: String(Date.now()),
      ...data,
      createdAt: new Date().toISOString(),
    };
    products.push(newProduct);
    return newProduct;
  },

  replace(id: string, data: CreateProductDTO): ProductModel | null {
    const index = products.findIndex((p) => p.id === id);
    if (index === -1) return null;

    const replacedProduct: ProductModel = {
      id,
      ...data,
      createdAt: products[index].createdAt,
    };
    products[index] = replacedProduct;
    return replacedProduct;
  },

  patch(id: string, data: PatchProductDTO): ProductModel | null {
    const index = products.findIndex((p) => p.id === id);
    if (index === -1) return null;

    products[index] = { ...products[index], ...data };
    return products[index];
  },

  delete(id: string): boolean {
    const index = products.findIndex((p) => p.id === id);
    if (index === -1) return false;
    products.splice(index, 1);
    return true;
  },
};
```
- **Expected result:** Clean service methods export ho jayenge.

---

### Step 3: Product Collection Route Handler (`src/app/api/products/route.ts`) implement karo
- **Kya karna hai:** `GET` (with category search param) aur `POST` (with JSON validation) export karo.
- **Kyu karna hai:** `/api/products` endpoint par products list aur create karne ke liye.
- **Code (`src/app/api/products/route.ts`):**
```typescript
import { NextRequest, NextResponse } from "next/server";
import { ProductService } from "@/lib/services/product.service";
import { createProductSchema } from "@/lib/validations/product.schema";

// GET /api/products?category=electronics
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const category = searchParams.get("category") || undefined;

    const products = ProductService.getAll(category);
    return NextResponse.json(products, { status: 200 });
  } catch (error) {
    return NextResponse.json({ error: "Failed to fetch products" }, { status: 500 });
  }
}

// POST /api/products
export async function POST(request: NextRequest) {
  try {
    let body;
    try {
      body = await request.json();
    } catch {
      return NextResponse.json({ error: "Invalid JSON body" }, { status: 400 });
    }

    const validation = createProductSchema.safeParse(body);
    if (!validation.success) {
      return NextResponse.json(
        { error: "Validation failed", details: validation.error.flatten().fieldErrors },
        { status: 400 }
      );
    }

    const newProduct = ProductService.create(validation.data);
    return NextResponse.json(newProduct, { status: 201 });
  } catch (error) {
    return NextResponse.json({ error: "Internal server error" }, { status: 500 });
  }
}
```
- **Expected result:** `GET` 200 array return karega aur `POST` valid data par 201 Created return karega.

---

### Step 4: Individual Product Route Handler (`src/app/api/products/[id]/route.ts`) implement karo
- **Kya karna hai:** `GET`, `PUT`, `PATCH`, aur `DELETE` methods implement karo.
- **Kyu karna hai:** `/api/products/:id` par single item operations perform karne ke liye.
- **Code (`src/app/api/products/[id]/route.ts`):**
```typescript
import { NextRequest, NextResponse } from "next/server";
import { ProductService } from "@/lib/services/product.service";
import { updateProductSchema, patchProductSchema } from "@/lib/validations/product.schema";

interface RouteContext {
  params: Promise<{ id: string }>;
}

// 1. GET /api/products/:id
export async function GET(request: NextRequest, context: RouteContext) {
  const { id } = await context.params;
  const product = ProductService.getById(id);

  if (!product) {
    return NextResponse.json({ error: "Product not found" }, { status: 404 });
  }
  return NextResponse.json(product, { status: 200 });
}

// 2. PUT /api/products/:id (Full replace)
export async function PUT(request: NextRequest, context: RouteContext) {
  const { id } = await context.params;
  const body = await request.json().catch(() => null);

  if (!body) return NextResponse.json({ error: "Invalid JSON" }, { status: 400 });

  const validation = updateProductSchema.safeParse(body);
  if (!validation.success) {
    return NextResponse.json({ error: "Validation error", details: validation.error.flatten().fieldErrors }, { status: 400 });
  }

  const updated = ProductService.replace(id, validation.data);
  if (!updated) return NextResponse.json({ error: "Product not found" }, { status: 404 });

  return NextResponse.json(updated, { status: 200 });
}

// 3. PATCH /api/products/:id (Partial update)
export async function PATCH(request: NextRequest, context: RouteContext) {
  const { id } = await context.params;
  const body = await request.json().catch(() => null);

  if (!body) return NextResponse.json({ error: "Invalid JSON" }, { status: 400 });

  const validation = patchProductSchema.safeParse(body);
  if (!validation.success) {
    return NextResponse.json({ error: "Validation error", details: validation.error.flatten().fieldErrors }, { status: 400 });
  }

  const patched = ProductService.patch(id, validation.data);
  if (!patched) return NextResponse.json({ error: "Product not found" }, { status: 404 });

  return NextResponse.json(patched, { status: 200 });
}

// 4. DELETE /api/products/:id
export async function DELETE(request: NextRequest, context: RouteContext) {
  const { id } = await context.params;
  const deleted = ProductService.delete(id);

  if (!deleted) {
    return NextResponse.json({ error: "Product not found" }, { status: 404 });
  }

  // 204 No Content must have empty response body
  return new NextResponse(null, { status: 204 });
}
```
- **Expected result:** Single product par CRUD operations perfectly respond karenge aur DELETE par 204 status aayega bina body ke.

---

### Step 5: Frontend UI ko Route Handlers se connect karo
- **Kya karna hai:** Existing product listing ya creation form se `/api/products` endpoint ko call karo.
- **Kyu karna hai:** Verify karna ki frontend client Route Handlers ke sath smoothly communicate kar raha hai.
- **Code snippet (Client fetch call):**
```tsx
const res = await fetch("/api/products", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Gaming Headset", price: 79.99, category: "electronics" }),
});
const data = await res.json();
```
- **Expected result:** Response code 201 receive hoga aur list me naya item display ho jayega.

---

## 4. Final File Structure

```text
src/
├── app/
│   └── api/
│       └── products/
│           ├── [id]/
│           │   └── route.ts     # GET, PUT, PATCH, DELETE (:id)
│           └── route.ts         # GET (collection), POST (create)
└── lib/
    ├── services/
    │   └── product.service.ts   # Shared business logic & in-memory store
    └── validations/
        └── product.schema.ts    # Zod request validation schemas
```

---

## 5. Common Mistakes / Errors

1. **`204 No Content` response me body return karna:**
   - *Galti:* `return NextResponse.json({ message: "Deleted" }, { status: 204 });` likhna. HTTP standard ke mutabiq 204 me koi content allowed nahi hota aur browser warning deta hai.
   - *Fix:* `return new NextResponse(null, { status: 204 });` use karein.

2. **Next.js 15 me `context.params` ko synchronous read karna:**
   - *Galti:* `const id = context.params.id;` likhna.
   - *Fix:* Always `const { id } = await context.params;` likhein.

3. **Malformed JSON handling na karna:**
   - *Galti:* Direct `await request.json()` call karna bina `try/catch` ke. Agar client empty ya invalid JSON bhej de to server 500 error throw karega.
   - *Fix:* Catch block lagakar clear `400 Bad Request` return karein.

4. **Product Collection ko alag-alag file me duplicate kar dena:**
   - *Galti:* `route.ts` aur `[id]/route.ts` dono me alag-alag `let products = []` rakhna, jisse ek route par create kiya product doosre me nahi milta.
   - *Fix:* Shared `product.service.ts` create karein jo single store access kare.

---

## 6. Verification Checklist
- [ ] `GET /api/products` status `200` aur product list return kar raha hai.
- [ ] `GET /api/products?category=electronics` category filtering support kar raha hai.
- [ ] `POST /api/products` valid data par status `201` aur created item return kar raha hai.
- [ ] Invalid body bhejne par status `400` aur validation errors mil rahe hain.
- [ ] `GET /api/products/1` status `200` return kar raha hai, jabki `GET /api/products/999` status `404` deta hai.
- [ ] `PUT /api/products/:id` pura record replace karta hai.
- [ ] `PATCH /api/products/:id` sirf provided fields update karta hai.
- [ ] `DELETE /api/products/:id` status `204` return karta hai bina kisi body ke.

---

## 7. Bonus (Extra Practice)
1. **API Rate Limiter Middleware Simulation:** In-memory map bana kar check karo ki ek IP se 1 minute me 50 se zyada requests na aayein (return 429 Too Many Requests).
2. **Standard API Envelope:** Saare success responses ko `{ success: true, data: ... }` aur errors ko `{ success: false, error: ... }` format me wrap karo.
