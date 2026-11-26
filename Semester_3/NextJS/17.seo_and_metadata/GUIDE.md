# Step-by-Step Guide: Assignment 17 — SEO & Metadata Optimization

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router ke **SEO & Metadata API** ko use karke E-commerce application ko search engines aur social platforms ke liye fully optimize karna hai:
- Global title templates aur metadata base URL configure karna
- Product detail pages ke liye **Dynamic Metadata** (`generateMetadata`) generate karna
- Social previews ke liye **Open Graph** aur **Twitter Cards** setup karna
- Canonical URLs set karna
- Dynamic `app/sitemap.ts` aur `app/robots.ts` build karna
- Product pages par **JSON-LD Structured Data** inject karna.

---

## 2. Zaroori Concepts (Recap)
- **Static Metadata (`export const metadata: Metadata`)**: Fixed pages (Home, Contact) ke liye jo build time par HTML `<head>` me inject hota hai.
- **Dynamic Metadata (`generateMetadata`)**: Dynamic routes (`/products/[id]`) ke liye jaha product data fetch karke page title, description aur preview images dynamic set hoti hain.
- **Title Template (`%s | NextStore`)**: Root layout me define kiya jata hai taaki sub-pages sirf page name dein aur brand name automatically append ho jaye.
- **Open Graph (OG) & Twitter Cards**: WhatsApp, LinkedIn, Twitter par link share karne par beautiful preview card (image, title, summary) display karne ke liye.
- **`sitemap.ts` & `robots.ts`**: Search engine crawlers (Googlebot) ko site structure aur crawl permissions batane ke liye TypeScript-based route handlers.
- **JSON-LD (Structured Data)**: Google search results me rich snippets (ratings, price, in-stock status) display karne ke liye schema markup format.

> 📖 Course Notes Reference: SEO aur metadata architecture ke liye is folder ki note file [17.seo_and_metadata.md](./17.seo_and_metadata.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Global Metadata Template configure karo (`src/app/layout.tsx`)
- **Kya karna hai:** Root layout me `metadataBase`, `title` with template, aur default description define karo.
- **Kyu karna hai:** Brand consistency maintain karne aur social sharing images ke absolute URLs automatically resolve karne ke liye.
- **Code (`src/app/layout.tsx`):**
```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_APP_URL || "https://nextstore.com"),
  title: {
    default: "NextStore — Quality Products for Modern Lifestyle",
    template: "%s | NextStore",
  },
  description: "Browse thousands of premium electronics, fashion, and lifestyle items with fast delivery.",
  openGraph: {
    type: "website",
    locale: "en_US",
    siteName: "NextStore",
  },
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```
- **Expected result:** Sub-pages ka title automatically `Product Name | NextStore` format me generate hoga.

---

### Step 2: Dynamic Metadata implement karo (`src/app/products/[id]/page.tsx`)
- **Kya karna hai:** `generateMetadata()` function export karo jo dynamic params read karke product details fetch kare aur Open Graph metadata return kare.
- **Kyu karna hai:** Har individual product ka unique title, description aur social share preview card banane ke liye.
- **Code (`src/app/products/[id]/page.tsx`):**
```tsx
import type { Metadata } from "next";

interface Props {
  params: Promise<{ id: string }>;
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { id } = await params;

  // Fetch product data
  const res = await fetch(`http://localhost:4000/products/${id}`).catch(() => null);
  const product = res && res.ok ? await res.json() : null;

  if (!product) {
    return {
      title: "Product Not Found",
      description: "The requested product does not exist.",
    };
  }

  const productUrl = `/products/${id}`;

  return {
    title: product.name,
    description: product.description || `Buy ${product.name} at the best price on NextStore.`,
    alternates: {
      canonical: productUrl, // Canonical URL for SEO
    },
    openGraph: {
      title: product.name,
      description: product.description,
      url: productUrl,
      images: [
        {
          url: product.imageUrl || "/images/default-product.jpg",
          width: 800,
          height: 600,
          alt: product.name,
        },
      ],
    },
    twitter: {
      card: "summary_large_image",
      title: product.name,
      description: product.description,
      images: [product.imageUrl || "/images/default-product.jpg"],
    },
  };
}
```
- **Expected result:** Page source me accurate `<meta property="og:title">`, `<meta name="twitter:card">`, aur `<link rel="canonical">` render honge.

---

### Step 3: Product JSON-LD Structured Data inject karo
- **Kya karna hai:** Product detail page ke JSX me Schema.org Product JSON-LD `<script>` tag render karo.
- **Kyu karna hai:** Google search results me price, currency aur stock status ke rich snippets dikhane ke liye.
- **Code snippet (inside `src/app/products/[id]/page.tsx`):**
```tsx
export default async function ProductDetailPage({ params }: Props) {
  const { id } = await params;
  const res = await fetch(`http://localhost:4000/products/${id}`);
  const product = await res.json();

  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "Product",
    name: product.name,
    image: product.imageUrl,
    description: product.description,
    sku: `SKU-${product.id}`,
    offers: {
      "@type": "Offer",
      priceCurrency: "USD",
      price: product.price,
      availability: product.stock > 0 ? "https://schema.org/InStock" : "https://schema.org/OutOfStock",
      url: `https://nextstore.com/products/${product.id}`,
    },
  };

  return (
    <article className="p-8">
      {/* JSON-LD Script tag */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <h1 className="text-3xl font-bold">{product.name}</h1>
      <p className="text-2xl font-bold text-blue-600 mt-2">${product.price}</p>
      <p className="mt-4 text-slate-600">{product.description}</p>
    </article>
  );
}
```
- **Expected result:** Google Rich Results Test tool me valid Schema.org Product detect hoga.

---

### Step 4: Dynamic Sitemap create karo (`src/app/sitemap.ts`)
- **Kya karna hai:** `src/app/sitemap.ts` create karo jo static pages aur dynamic product URLs ka array return kare.
- **Kyu karna hai:** Search engines ko batane ke liye ki website par kaunse pages index hone chahiye.
- **Code (`src/app/sitemap.ts`):**
```typescript
import { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = "https://nextstore.com";

  // Static routes
  const staticRoutes: MetadataRoute.Sitemap = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: "daily",
      priority: 1.0,
    },
    {
      url: `${baseUrl}/products`,
      lastModified: new Date(),
      changeFrequency: "daily",
      priority: 0.8,
    },
    {
      url: `${baseUrl}/contact`,
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 0.5,
    },
    {
      url: `${baseUrl}/terms`,
      lastModified: new Date(),
      changeFrequency: "yearly",
      priority: 0.3,
    },
  ];

  // Fetch dynamic product IDs
  let productRoutes: MetadataRoute.Sitemap = [];
  try {
    const res = await fetch("http://localhost:4000/products");
    if (res.ok) {
      const data = await res.json();
      productRoutes = data.data.map((p: any) => ({
        url: `${baseUrl}/products/${p.id}`,
        lastModified: new Date(p.updatedAt || Date.now()),
        changeFrequency: "weekly",
        priority: 0.7,
      }));
    }
  } catch (error) {
    console.error("Sitemap product fetch failed:", error);
  }

  return [...staticRoutes, ...productRoutes];
}
```
- **Expected result:** Browser me `http://localhost:3000/sitemap.xml` visit karne par valid XML format me saare links render honge.

---

### Step 5: Robots configuration create karo (`src/app/robots.ts`)
- **Kya karna hai:** `src/app/robots.ts` create karo jo search crawler permissions return kare.
- **Kyu karna hai:** Public pages ko index hone dena aur private admin/dashboard routes ko web bots se disallow karne ke liye.
- **Code (`src/app/robots.ts`):**
```typescript
import { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: "*",
      allow: "/",
      disallow: ["/admin/", "/dashboard/", "/api/"],
    },
    sitemap: "https://nextstore.com/sitemap.xml",
  };
}
```
- **Expected result:** Browser me `http://localhost:3000/robots.txt` open karne par standard crawler rules render honge.

---

### Step 6: Semantic HTML aur Headings verify karo
- **Kya karna hai:** Ensure karo ki page me single `<h1>` tag ho, sections ke liye `<section>` ya `<article>` tags hon, aur logical heading hierarchy (`h1` -> `h2` -> `h3`) follow ho rahi ho.
- **Kyu karna hai:** Screen readers aur search bot algorithms semantic structure ko prefer karte hain.
- **Expected result:** Clean HTML document outline verify hoga.

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── products/
│   │   ├── [id]/
│   │   │   └── page.tsx        # generateMetadata() + JSON-LD script
│   │   └── page.tsx            # Static catalog metadata
│   ├── layout.tsx              # Root metadata template (%s | NextStore)
│   ├── robots.ts               # Generates /robots.txt
│   └── sitemap.ts              # Generates /sitemap.xml
```

---

## 5. Common Mistakes / Errors

1. **`metadataBase` configure na karna:**
   - *Galti:* Relative image paths (`/images/og.jpg`) use karna bina `metadataBase` ke, jisse Next.js console warning deta hai ki full URL resolve nahi ho pa raha.
   - *Fix:* Root `app/layout.tsx` me `metadataBase: new URL(...)` zaroor configure karein.

2. **Client component me `metadata` ya `generateMetadata` export karna:**
   - *Galti:* `"use client"` waali file me `export const metadata = ...` likhna. Next.js me metadata sirf Server Components me support hota hai.
   - *Fix:* Metadata hamesha server `page.tsx` ya `layout.tsx` me hi export karein.

3. **Robots rules ko access security samajh lena:**
   - *Galti:* Sochna ki `disallow: ['/admin/']` likh diya to ab admin page secure ho gaya. Bad bots robots.txt ignore kar dete hain.
   - *Fix:* Authentication aur Authorization server-side session checks se hi enforce karein.

4. **JSON-LD script me invalid JSON hona:**
   - *Galti:* Missing quotes ya trailing commas hona jisse Google Search Console structured data error throw karta hai.
   - *Fix:* Always `JSON.stringify(jsonLdObject)` use karein.

---

## 6. Verification Checklist
- [ ] Root layout me title template `%s | NextStore` configured hai.
- [ ] Product detail page par title automatically `${product.name} | NextStore` display hota hai.
- [ ] Page source inspect karne par `<meta property="og:image">` aur `<meta name="twitter:card">` render ho rahe hain.
- [ ] `http://localhost:3000/sitemap.xml` par valid XML sitemap khul raha hai jisme product URLs included hain.
- [ ] `http://localhost:3000/robots.txt` `/admin` aur `/dashboard` ko disallow show kar raha hai.
- [ ] Product page me valid JSON-LD `<script type="application/ld+json">` present hai.

---

## 7. Bonus (Extra Practice)
1. **Dynamic Open Graph Image (`opengraph-image.tsx`):** Next.js ke `@vercel/og` ImageResponse API se dynamic social cards render karo jisme product name aur price graphically print ho.
2. **BreadcrumbList Schema:** Google search me breadcrumbs display karne ke liye JSON-LD me `BreadcrumbList` schema markup inject karo.
