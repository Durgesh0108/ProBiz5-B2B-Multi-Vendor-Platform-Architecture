# Project Title: ProBiz5 – B2B Multi-Vendor Platform Architecture

> **Note:** This repository serves as a technical case study and architectural overview. The source code is proprietary intellectual property of **R5 Advertising** and cannot be shared publicly. This document highlights the strategies used for SEO optimization, search algorithms, and secure payment processing.

## 1. Abstract
ProBiz5 is a multi-vendor B2B platform designed to help Small and Medium Enterprises (SMEs) capture business leads. The engineering focus was on overcoming the limitations of Client-Side Rendering (CSR) to improve **Search Engine Optimization (SEO)** and reduce **Time-to-Interactive (TTI)**. By migrating to a **Next.js Server-Side Rendering (SSR)** architecture and implementing optimized database indexing, we achieved a **2x increase in vendor lead generation**.

## 2. Technical Stack
* **Framework:** Next.js (App Router) - Utilized for Server-Side Rendering (SSR) and API routes.
* **Database:** MongoDB - NoSQL storage for flexible vendor profiles.
* **ORM:** Prisma - Type-safe schema management and query optimization.
* **Authentication:** Clerk - Vendor and User identity management.
* **Payments:** Razorpay Integration (Webhooks).
* **Media:** Cloudinary (Image Optimization) & AWS S3.

## 3. The Engineering Challenge (Problem Statement)
The initial prototype faced critical bottlenecks that hindered business growth:
1.  **Poor SEO Performance:** Traditional React CSR rendered blank HTML initially, meaning search engine crawlers could not index vendor profiles effectively.
2.  **High Search Latency:** Unoptimized database queries for vendor services resulted in slow search results (>1.2s), leading to user drop-off.
3.  **Trust & Security:** Need for a secure transaction layer to handle subscription payments without exposing sensitive data.

## 4. Architectural Solution & Methodology

### A. Server-Side Rendering (SSR) for SEO
To ensure vendor pages were indexable by Google/Bing, I leveraged Next.js Server Components. Unlike standard React, this fetches data on the server *before* sending HTML to the client.

*Pseudocode Logic (Next.js App Router):*
```tsx
// app/vendor/[id]/page.tsx
import prisma from '@/lib/prisma';

export default async function VendorProfile({ params }) {
  // 1. Fetch data directly on the server (No API round-trip latency)
  const vendor = await prisma.vendor.findUnique({
    where: { id: params.id },
    include: { services: true } // Eager loading
  });

  // 2. Return pre-rendered HTML (SEO Friendly)
  return (
    <main>
      <h1>{vendor.businessName}</h1>
      <meta name="description" content={vendor.description} />
      <ServiceList services={vendor.services} />
    </main>
  );
}
```


### B. Optimized Search Algorithm (Indexing)
To fix the slow search, I implemented **Compound Indexing** in the database and utilized Prisma's filtering capabilities to create a "Smart Search" feature.

*Pseudocode Logic (Search Query):*
```typescript
// Optimized Query Strategy
const results = await prisma.service.findMany({
  where: {
    OR: [
      { name: { contains: query, mode: 'insensitive' } }, // Case-insensitive match
      { category: { equals: selectedCategory } }
    ],
    isActive: true
  },
  take: 20, // Pagination limit to reduce load
  orderBy: {
    rating: 'desc' // Relevance sorting
  }
});
```


### C. Secure Payment Webhooks ###
Integrated Razorpay for vendor subscriptions. To prevent payment spoofing, I implemented **Cryptographic Signature Verification** on the backend webhook listener.

Pseudocode Logic:

```js
import crypto from 'crypto';

export async function POST(req) {
  const body = await req.text();
  const signature = req.headers.get('x-razorpay-signature');

  // Verify that the request actually came from Razorpay
  const expectedSignature = crypto
    .createHmac('sha256', process.env.RAZORPAY_SECRET)
    .update(body)
    .digest('hex');

  if (signature === expectedSignature) {
    // Safe to activate subscription
    await activateVendorSubscription(body.payload.payment.entity.notes.vendorId);
    return new Response('Success', { status: 200 });
  } else {
    return new Response('Invalid Signature', { status: 400 });
  }
}
```

### 5. Quantitative Results ###
* **SEO & Traffic:** Switching to SSR reduced Initial Page Load time by 40%, leading to a measurable increase in organic search traffic.

* **Lead Conversion:** Optimized search logic improved service discoverability, resulting in a 2x increase in inquiries sent to vendors.

* **Transaction Reliability:** Webhook implementation ensured 100% accuracy in subscription status updates (zero failed activations).

