# Step-by-Step Guide: Assignment 10 — Server Actions & Mutations

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js **Server Actions** (`"use server"`) aur modern form handling implement karna hai. Hume `/admin` section ke andar product management feature banana hai jisme:
- **Add Product** form (with Zod validation, pending state via `useActionState`, aur React Hook Form)
- **Edit Product** form (passing ID via `.bind(null, id)`)
- **Delete Product** action (with confirmation)
- Server-side authorization check aur mutation ke baad `revalidatePath('/products')` & `redirect()`.

---

## 2. Zaroori Concepts (Recap)
- **Server Actions (`"use server"`)**: Server par chalne waale asynchronous functions jinhe client forms ya buttons se direct invoke kiya ja sakta hai without manual API route handlers.
- **`useActionState`**: Form state (validation errors, server feedback) aur `isPending` state ko cleanly handle karne waala modern React hook.
- **`useFormStatus`**: Child component (jaise submit button) me parent form ka `pending` status read karne ke liye.
- **Cache Revalidation (`revalidatePath`)**: Database ya store me mutation ke baad Next.js data cache ko purge karta hai taaki updated list dikhe.
- **Server Authorization**: Client ke buttons chupane ke alawa Server Action function ke andar bhi permissions check karna mandatory hota hai.

> 📖 Course Notes Reference: Server actions ke core patterns ke liye [10.server_actions.md](./10.server_actions.md) aur validation ke liye [../09.forms_and_validation/09.forms_and_validation.md](../09.forms_and_validation/09.forms_and_validation.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Shared Zod Schema banao (`src/lib/validations/product.ts`)
- **Kya karna hai:** Product creation aur update ke liye shared Zod schema define karo.
- **Kyu karna hai:** Client aur server dono jagah same validation rules re-use karne ke liye.
- **Code (`src/lib/validations/product.ts`):**
```typescript
import { z } from "zod";

export const productSchema = z.object({
  name: z.string().min(3, "Name must be at least 3 characters").max(100),
  description: z.string().min(10, "Description must be at least 10 characters"),
  price: z.coerce.number().positive("Price must be a positive number"),
  category: z.string().min(2, "Category is required"),
  stock: z.coerce.number().int().nonnegative("Stock cannot be negative"),
  imageUrl: z.string().url("Must be a valid image URL"),
});

export type ProductInput = z.infer<typeof productSchema>;
```
- **Expected result:** Type-safe schema ready ho jayega jise forms aur server actions dono import kar sakte hain.

---

### Step 2: Shared In-Memory Product Store banao (`src/lib/store.ts`)
- **Kya karna hai:** Temporary products array aur helper methods create karo.
- **Kyu karna hai:** Database integration aane se pehle mutations (create, update, delete) ko persist rakhne ke liye.
- **Code (`src/lib/store.ts`):**
```typescript
export interface StoreProduct {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  stock: number;
  imageUrl: string;
}

// Global store to persist across development reloads
const globalStore = global as unknown as { products: StoreProduct[] };
if (!globalStore.products) {
  globalStore.products = [
    {
      id: "prod-1",
      name: "Wireless ANC Headphones",
      description: "Premium sound quality with active noise cancellation.",
      price: 199,
      category: "Electronics",
      stock: 15,
      imageUrl: "https://images.unsplash.com/photo-1505740420928-5e560c06d30e",
    },
  ];
}

export const productsDB = globalStore.products;
```
- **Expected result:** Fast shared store ready ho jayega.

---

### Step 3: Server Actions implement karo (`src/actions/product.ts`)
- **Kya karna hai:** File ke top par `"use server"` likho aur `createProduct`, `updateProduct`, aur `deleteProduct` functions export karo.
- **Kyu karna hai:** Server-side mutations, Zod validation, authorization check aur cache revalidation handle karne ke liye.
- **Code (`src/actions/product.ts`):**
```typescript
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { productSchema } from "@/lib/validations/product";
import { productsDB } from "@/lib/store";

export type ActionState = {
  success?: boolean;
  message?: string;
  errors?: Record<string, string[]>;
  fields?: Record<string, any>;
};

// Simulated admin check
function checkAdminAuth() {
  const isAdmin = true; // Placeholder auth check
  if (!isAdmin) {
    throw new Error("Unauthorized: Only admins can manage products");
  }
}

// 1. Create Product Action
export async function createProductAction(
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> {
  checkAdminAuth();

  const rawData = Object.fromEntries(formData.entries());
  const validated = productSchema.safeParse(rawData);

  if (!validated.success) {
    return {
      success: false,
      message: "Validation failed. Please correct the fields.",
      errors: validated.error.flatten().fieldErrors,
      fields: rawData,
    };
  }

  const newProduct = {
    id: `prod-${Date.now()}`,
    ...validated.data,
  };

  productsDB.unshift(newProduct);

  revalidatePath("/products");
  revalidatePath("/admin/products");

  return {
    success: true,
    message: "Product created successfully!",
  };
}

// 2. Update Product Action
export async function updateProductAction(
  productId: string,
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> {
  checkAdminAuth();

  const rawData = Object.fromEntries(formData.entries());
  const validated = productSchema.safeParse(rawData);

  if (!validated.success) {
    return {
      success: false,
      errors: validated.error.flatten().fieldErrors,
      fields: rawData,
    };
  }

  const index = productsDB.findIndex((p) => p.id === productId);
  if (index === -1) {
    return { success: false, message: "Product not found" };
  }

  productsDB[index] = { ...productsDB[index], ...validated.data };

  revalidatePath("/products");
  revalidatePath("/admin/products");

  redirect("/admin/products");
}

// 3. Delete Product Action
export async function deleteProductAction(productId: string) {
  checkAdminAuth();

  const index = productsDB.findIndex((p) => p.id === productId);
  if (index !== -1) {
    productsDB.splice(index, 1);
  }

  revalidatePath("/products");
  revalidatePath("/admin/products");
}
```
- **Expected result:** Secure server actions ready ho jayengi jo validation errors structured format me return karti hain.

---

### Step 4: Add Product Form with `useActionState` banao
- **Kya karna hai:** `src/app/admin/products/new/page.tsx` me form banao jo `useActionState` use kare field errors display karne ke liye.
- **Kyu karna hai:** Progressive enhancement aur pending state handle karne ke liye.
- **Code (`src/app/admin/products/new/page.tsx`):**
```tsx
"use client";

import { useActionState } from "react";
import { createProductAction, ActionState } from "@/actions/product";

const initialState: ActionState = {};

export default function AddProductPage() {
  const [state, formAction, isPending] = useActionState(createProductAction, initialState);

  return (
    <div className="max-w-2xl mx-auto p-6 bg-white rounded-xl shadow-sm border mt-6">
      <h1 className="text-2xl font-bold text-slate-900 mb-6">Add New Product</h1>

      {state.message && (
        <div
          role="alert"
          className={`p-3 rounded mb-4 text-sm ${
            state.success ? "bg-emerald-100 text-emerald-800" : "bg-red-100 text-red-800"
          }`}
        >
          {state.message}
        </div>
      )}

      <form action={formAction} className="space-y-4">
        <div>
          <label htmlFor="name" className="block text-sm font-medium text-slate-700">Product Name</label>
          <input
            id="name"
            name="name"
            defaultValue={state.fields?.name}
            className="mt-1 w-full border rounded-lg p-2.5 text-sm"
          />
          {state.errors?.name && <p className="text-red-500 text-xs mt-1">{state.errors.name[0]}</p>}
        </div>

        <div>
          <label htmlFor="price" className="block text-sm font-medium text-slate-700">Price ($)</label>
          <input
            id="price"
            name="price"
            type="number"
            step="0.01"
            defaultValue={state.fields?.price}
            className="mt-1 w-full border rounded-lg p-2.5 text-sm"
          />
          {state.errors?.price && <p className="text-red-500 text-xs mt-1">{state.errors.price[0]}</p>}
        </div>

        <div>
          <label htmlFor="category" className="block text-sm font-medium text-slate-700">Category</label>
          <input
            id="category"
            name="category"
            defaultValue={state.fields?.category}
            className="mt-1 w-full border rounded-lg p-2.5 text-sm"
          />
          {state.errors?.category && <p className="text-red-500 text-xs mt-1">{state.errors.category[0]}</p>}
        </div>

        <div>
          <label htmlFor="stock" className="block text-sm font-medium text-slate-700">Stock Quantity</label>
          <input
            id="stock"
            name="stock"
            type="number"
            defaultValue={state.fields?.stock}
            className="mt-1 w-full border rounded-lg p-2.5 text-sm"
          />
          {state.errors?.stock && <p className="text-red-500 text-xs mt-1">{state.errors.stock[0]}</p>}
        </div>

        <div>
          <label htmlFor="imageUrl" className="block text-sm font-medium text-slate-700">Image URL</label>
          <input
            id="imageUrl"
            name="imageUrl"
            defaultValue={state.fields?.imageUrl}
            className="mt-1 w-full border rounded-lg p-2.5 text-sm"
          />
          {state.errors?.imageUrl && <p className="text-red-500 text-xs mt-1">{state.errors.imageUrl[0]}</p>}
        </div>

        <div>
          <label htmlFor="description" className="block text-sm font-medium text-slate-700">Description</label>
          <textarea
            id="description"
            name="description"
            rows={3}
            defaultValue={state.fields?.description}
            className="mt-1 w-full border rounded-lg p-2.5 text-sm"
          />
          {state.errors?.description && <p className="text-red-500 text-xs mt-1">{state.errors.description[0]}</p>}
        </div>

        <button
          type="submit"
          disabled={isPending}
          className="w-full py-2.5 bg-blue-600 text-white font-semibold rounded-lg hover:bg-blue-700 disabled:opacity-50 transition"
        >
          {isPending ? "Creating Product..." : "Create Product"}
        </button>
      </form>
    </div>
  );
}
```
- **Expected result:** Form invalid data par specific field ke neeche red error dikhayega aur previous input preserve karega.

---

### Step 5: Edit Product form with `.bind()` banao
- **Kya karna hai:** `src/app/admin/products/[id]/edit/page.tsx` me existing values load karo aur action ko product ID bind karo.
- **Kyu karna hai:** Update Server Action ko pata chal sake ki kaunsa record edit ho raha hai.
- **Code snippet:**
```tsx
const updateActionWithId = updateProductAction.bind(null, product.id);
const [state, formAction, isPending] = useActionState(updateActionWithId, initialState);
// form render karo with defaultValue={product.name}, etc.
```
- **Expected result:** Form submit hone par wahi specific product update hoga aur redirect execute hoga.

---

### Step 6: Delete Button with Server Action banao
- **Kya karna hai:** `src/components/DeleteProductButton.tsx` create karo jisme confirmation dialog ho.
- **Kyu karna hai:** Accidental deletion prevent karne ke liye.
- **Code (`src/components/DeleteProductButton.tsx`):**
```tsx
"use client";

import { useTransition } from "react";
import { deleteProductAction } from "@/actions/product";

export default function DeleteProductButton({ productId }: { productId: string }) {
  const [isPending, startTransition] = useTransition();

  const handleDelete = () => {
    if (confirm("Are you sure you want to delete this product?")) {
      startTransition(async () => {
        await deleteProductAction(productId);
      });
    }
  };

  return (
    <button
      onClick={handleDelete}
      disabled={isPending}
      className="px-3 py-1 bg-red-600 text-white rounded text-xs font-semibold hover:bg-red-700 disabled:opacity-50 transition"
    >
      {isPending ? "Deleting..." : "Delete"}
    </button>
  );
}
```
- **Expected result:** Confirm karne par item remove hoga aur list bina reload ke update ho jayegi.

---

## 4. Final File Structure

```text
src/
├── actions/
│   └── product.ts                  # "use server" Server Actions
├── app/
│   └── admin/
│       └── products/
│           ├── [id]/
│           │   └── edit/
│           │       └── page.tsx    # Edit form (.bind(null, id))
│           ├── new/
│           │   └── page.tsx        # Add form (useActionState)
│           └── page.tsx            # Admin product listing table
├── components/
│   └── DeleteProductButton.tsx     # Client deletion trigger
└── lib/
    ├── store.ts                    # Shared in-memory data store
    └── validations/
        └── product.ts              # Zod validation schema
```

---

## 5. Common Mistakes / Errors

1. **`redirect()` ko broad `try/catch` block ke andar rakhna:**
   - *Galti:* `try { ... redirect('/admin/products'); } catch (err) { return { error: err.message }; }`
   - *Fix:* Next.js me `redirect()` internally special error throw karta hai. Isko `try/catch` block ke bilkul bahar call karein ya error check karein (`if (isRedirectError(err)) throw err;`).

2. **Form inputs me `name` attribute bhool jana:**
   - *Galti:* `<input id="price" />` likhna bina `name="price"` ke.
   - *Fix:* Server Actions me data `FormData` se read hota hai, isliye har input field me `name` attribute mandatory hai.

3. **Sirf client-side validation par depend karna:**
   - *Galti:* Browser inspect karke validation bypass ki ja sakti hai agar server par Zod parse na ho.
   - *Fix:* Server Action ke andar `productSchema.safeParse()` zaroor run karein.

4. **`revalidatePath` bhool jana:**
   - *Galti:* Product create hone ke baad `/products` page par naya product na dikhna kyunki Next.js page cache retain karta hai.
   - *Fix:* Mutation ke turant baad `revalidatePath('/products')` call karein.

---

## 6. Verification Checklist
- [ ] Add Product form empty submit karne par field-level validation errors dikhata hai.
- [ ] Valid data enter karne par new product create hota hai aur `revalidatePath` se public catalog me appear hota hai.
- [ ] Submission ke waqt button "Creating..." pending state me disable hota hai.
- [ ] Edit Product form me current values pre-filled aati hain aur save karne par redirect kaam karta hai.
- [ ] Delete button browser confirmation maangta hai aur confirm hone par product remove hota hai.
- [ ] Accessibility: Har input ke sath proper `<label htmlFor="...">` associated hai.

---

## 7. Bonus (Extra Practice)
1. **Toast Notification:** Action result aane par modern toast notification (e.g. Sonner ya react-hot-toast) trigger karo.
2. **Optimistic UI with `useOptimistic`:** Delete click hote hi item ko turant list se hide karo bina server confirmation ka wait kiye.
