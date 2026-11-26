# Step-by-Step Guide: Assignment 19 — Logging, Monitoring & Health Checks

---

## 1. Assignment ka Goal
Is assignment ka objective Next.js App Router application me production-grade **Observability & Reliability** tools add karna hai:
- Important server-side operations (product retrieval, order creation) me **Structured Application Logging** implement karna
- Route-level error handling (`error.tsx`) verify karke secure error responses ensure karna (bina stack trace leak kiye)
- **Sentry Error Monitoring** integrate karna aur test error capture verify karna
- Database connectivity report karne wala **/api/health** endpoint create karna
- Performance bottle-neck identify karke review document karna.

---

## 2. Zaroori Concepts (Recap)
- **Structured Logging**: Plain `console.log("error")` ke bajaye JSON format me standardized event names (`product.fetch.success`, `order.created`), timestamps, aur user context log karna taaki CloudWatch ya Datadog me easily search aur alert setup ho sake.
- **Sanitized Error Responses**: Production me internal server errors (jaise database connection string ya stack trace) client ko expose nahi hone chahiye; client ko sirf safe user-facing messages milne chahiye.
- **Error Monitoring (Sentry)**: Uncaught exceptions aur client-side crashes ko automatically capture karke development team ko dashboard me alerting provide karta hai.
- **Health Check Endpoint (`/api/health`)**: Load balancers (AWS ALB, Kubernetes) aur uptime monitors (Pingdom, UptimeRobot) ke liye endpoint jo server status aur database connectivity report karta hai.

> 📖 Course Notes Reference: Observability aur health monitoring patterns ke liye is folder ki note file [19.logging_and_monitoring.md](./19.logging_and_monitoring.md) padhein.

---

## 3. Step-by-Step Breakdown

### Step 1: Structured Logger Utility create karo (`src/lib/logger.ts`)
- **Kya karna hai:** Reusable logger function banao jo event name, level (`INFO`, `WARN`, `ERROR`), timestamp aur safe metadata log kare.
- **Kyu karna hai:** Logs ko readable, parseable aur secure banane ke liye (passwords/secrets ko exclude karke).
- **Code (`src/lib/logger.ts`):**
```typescript
type LogLevel = "INFO" | "WARN" | "ERROR";

interface LogPayload {
  event: string;
  level: LogLevel;
  timestamp: string;
  metadata?: Record<string, any>;
  error?: string;
}

export const logger = {
  info(event: string, metadata?: Record<string, any>) {
    this.log("INFO", event, metadata);
  },
  warn(event: string, metadata?: Record<string, any>) {
    this.log("WARN", event, metadata);
  },
  error(event: string, error: any, metadata?: Record<string, any>) {
    this.log("ERROR", event, metadata, error?.message || String(error));
  },

  log(level: LogLevel, event: string, metadata?: Record<string, any>, errorMessage?: string) {
    const payload: LogPayload = {
      level,
      event,
      timestamp: new Date().toISOString(),
      ...(metadata && { metadata }),
      ...(errorMessage && { error: errorMessage }),
    };

    if (level === "ERROR") {
      console.error(JSON.stringify(payload));
    } else {
      console.log(JSON.stringify(payload));
    }
  },
};
```
- **Expected result:** Clean JSON formatted logs terminal aur cloud logs me print honge.

---

### Step 2: Server-side Operations me Logging integrate karo
- **Kya karna hai:** Product fetch API aur Order creation action me `logger.info` aur `logger.error` call karo.
- **Kyu karna hai:** Production me traffic flow aur failure points ko easily audit karne ke liye.
- **Code snippet (inside `src/app/api/products/[id]/route.ts`):**
```typescript
import { logger } from "@/lib/logger";

export async function GET(request: NextRequest, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  logger.info("product.fetch.start", { productId: id });

  try {
    const product = await getProductById(id);
    if (!product) {
      logger.warn("product.fetch.not_found", { productId: id });
      return NextResponse.json({ error: "Product not found" }, { status: 404 });
    }

    logger.info("product.fetch.success", { productId: id, name: product.name });
    return NextResponse.json(product);
  } catch (error) {
    logger.error("product.fetch.failure", error, { productId: id });
    return NextResponse.json({ error: "Failed to retrieve product details" }, { status: 500 });
  }
}
```
- **Expected result:** Har product access par structured event print hoga.

---

### Step 3: Health Check Endpoint banao (`src/app/api/health/route.ts`)
- **Kya karna hai:** `/api/health` endpoint create karo jo uptime, environment status, aur MongoDB connection status check kare.
- **Kyu karna hai:** Infrastructure monitoring tools ko server aur database ka health status batane ke liye.
- **Code (`src/app/api/health/route.ts`):**
```typescript
import { NextResponse } from "next/server";
import mongoose from "mongoose";

export async function GET() {
  const startTime = Date.now();

  // Check MongoDB connection status
  // 0: disconnected, 1: connected, 2: connecting, 3: disconnecting
  const dbState = mongoose.connection.readyState;
  const isDbHealthy = dbState === 1;

  const status = isDbHealthy ? "healthy" : "degraded";
  const statusCode = isDbHealthy ? 200 : 503;

  const healthData = {
    status,
    timestamp: new Date().toISOString(),
    uptimeSeconds: Math.floor(process.uptime()),
    services: {
      web: "healthy",
      database: isDbHealthy ? "connected" : "disconnected",
    },
    responseTimeMs: Date.now() - startTime,
  };

  return NextResponse.json(healthData, { status: statusCode });
}
```
- **Expected result:** `GET /api/health` par JSON response aayega:
```json
{
  "status": "healthy",
  "timestamp": "2026-11-26T04:15:00.000Z",
  "uptimeSeconds": 1420,
  "services": {
    "web": "healthy",
    "database": "connected"
  },
  "responseTimeMs": 4
}
```

---

### Step 4: Sentry Error Monitoring integrate karo
- **Kya karna hai:** `@sentry/nextjs` install karo aur configuration initialize karo.
- **Kyu karna hai:** Real-time production crash alerts pane ke liye.
- **Code (Terminal):**
```bash
npx @sentry/wizard@latest -i nextjs
# ya manually install karo:
npm install @sentry/nextjs
```
- **Code (`sentry.client.config.ts`):**
```typescript
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN || "https://examplePublicKey@o0.ingest.sentry.io/0",
  tracesSampleRate: 1.0,
});
```
- **Code (`sentry.server.config.ts`):**
```typescript
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.SENTRY_DSN || process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```
- **Expected result:** Sentry Next.js configuration files generate ho jayengi.

---

### Step 5: Test Error Route create karo (`src/app/test-sentry/page.tsx`)
- **Kya karna hai:** Ek test page banao jisme button click par intentionally error throw ho.
- **Kyu karna hai:** Sentry dashboard par event capture verify karne ke liye.
- **Code (`src/app/test-sentry/page.tsx`):**
```tsx
"use client";

export default function TestSentryPage() {
  const triggerClientError = () => {
    throw new Error("Sentry Test Error: Client-side exception captured successfully!");
  };

  return (
    <div className="p-8 max-w-md mx-auto text-center">
      <h1 className="text-2xl font-bold mb-4">Sentry Monitoring Verification</h1>
      <p className="text-slate-600 mb-6 text-sm">
        Click the button below to trigger an intentional test exception.
      </p>
      <button
        onClick={triggerClientError}
        className="px-6 py-3 bg-red-600 text-white font-bold rounded-lg hover:bg-red-700 transition"
      >
        Trigger Test Error
      </button>
    </div>
  );
}
```
- **Expected result:** Button click karne par error throw hoga aur Sentry dashboard me capture ho jayega.

---

### Step 6: Safe Error Handling verify karo
- **Kya karna hai:** `src/app/products/error.tsx` aur API handlers me confirm karo ki koi bhi sensitive error message ya database stack trace UI me na dikhe.
- **Kyu karna hai:** Security audits aur OWASP standards pass karne ke liye.
- **Expected result:** Users ko "An unexpected error occurred" jaisa generic message dikhega jabki internal details sirf server logs me record hongi.

---

## 4. Final File Structure

```text
src/
├── app/
│   ├── api/
│   │   └── health/
│   │       └── route.ts          # GET /api/health status endpoint
│   ├── test-sentry/
│   │   └── page.tsx              # Trigger test error for verification
│   └── products/
│       └── error.tsx             # Sanitized user-facing error UI
├── lib/
│   └── logger.ts                 # JSON structured logger utility
sentry.client.config.ts           # Sentry browser config
sentry.server.config.ts           # Sentry server config
```

---

## 5. Common Mistakes / Errors

1. **Stack traces aur database errors client JSON me return kar dena:**
   - *Galti:* `catch (error) { return NextResponse.json({ error: error.stack }, { status: 500 }) }`. Isse attackers ko server directories aur database details mil jaati hain.
   - *Fix:* Always `logger.error(event, error)` server console me karein aur client ko safe `{ error: "Internal server error" }` bhejein.

2. **Health check me sensitive credentials expose kar dena:**
   - *Galti:* Response me MongoDB URI ya password return karna.
   - *Fix:* Health check sirf service status (`connected`/`disconnected`) aur latency report kare.

3. **`console.log()` unorganized use karna:**
   - *Galti:* `console.log("here 1")` likhna jo cloud logging me koi searchable value nahi deta.
   - *Fix:* Hamesha structured event names (`order.payment.failed`) aur metadata use karein.

4. **Sentry DSN `.env` me configure na karna:**
   - *Galti:* Sentry DSN missing hona jisse errors remote dashboard par send nahi hote.
   - *Fix:* Sentry project create karke `.env.local` me DSN string zaroor add karein.

---

## 6. Verification Checklist
- [ ] `GET /api/health` 200 OK aur database connectivity status return kar raha hai.
- [ ] Server operations console me JSON-formatted structured logs print kar rahe hain.
- [ ] `/test-sentry` button click karne par test error trigger hota hai aur Sentry dashboard par reflect hota hai.
- [ ] Simulated server errors client ko generic, safe messages return karte hain.
- [ ] README me health check response aur monitoring verification screenshots document kiye gaye hain.

---

## 7. Bonus (Extra Practice)
1. **Request ID Tracing:** Proxy/Middleware me unique `x-request-id` header generate karo aur use logger ke saare events me include karo taaki ek user request ke saare logs trace ho sakein.
2. **Discord / Slack Webhook Alert:** Logger error function me ek optional webhook call attach karo jo critical severity errors ko direct team ke Slack/Discord channel par bhej de.
