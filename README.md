# 🚗 CarBazaar — Car Buy & Sell Platform

A full-stack car marketplace where users can list, browse, and purchase cars with ease.

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14 (App Router) |
| Styling | Tailwind CSS |
| Authentication | Clerk |
| Database | PostgreSQL |
| ORM | Prisma |
| File Uploads | Uploadthing / Cloudinary |
| Payments | Stripe |
| Maps | Google Maps API / Mapbox |
| Deployment | Vercel + Railway / Supabase |

---

## 📁 Project Structure

```
carbazaar/
├── app/
│   ├── (auth)/
│   │   ├── sign-in/
│   │   └── sign-up/
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   ├── my-listings/
│   │   └── saved-cars/
│   ├── cars/
│   │   ├── [id]/
│   │   └── new/
│   ├── admin/
│   └── page.tsx
├── components/
│   ├── ui/
│   ├── cars/
│   └── shared/
├── lib/
│   ├── prisma.ts
│   └── utils.ts
├── prisma/
│   └── schema.prisma
├── public/
└── middleware.ts
```

---

## 🗄️ Database Schema (Prisma)

```prisma
model User {
  id          String   @id @default(cuid())
  clerkId     String   @unique
  name        String
  email       String   @unique
  phone       String?
  avatar      String?
  isVerified  Boolean  @default(false)
  listings    Car[]
  savedCars   SavedCar[]
  reviews     Review[]
  createdAt   DateTime @default(now())
}

model Car {
  id           String   @id @default(cuid())
  title        String
  make         String
  model        String
  year         Int
  price        Float
  mileage      Int
  fuelType     String
  transmission String
  color        String
  description  String
  images       String[]
  status       String   @default("available")
  city         String
  state        String
  sellerId     String
  seller       User     @relation(fields: [sellerId], references: [id])
  savedBy      SavedCar[]
  reviews      Review[]
  createdAt    DateTime @default(now())
}

model SavedCar {
  id        String   @id @default(cuid())
  userId    String
  carId     String
  user      User     @relation(fields: [userId], references: [id])
  car       Car      @relation(fields: [carId], references: [id])
}

model Review {
  id        String   @id @default(cuid())
  rating    Int
  comment   String
  userId    String
  carId     String
  user      User     @relation(fields: [userId], references: [id])
  car       Car      @relation(fields: [carId], references: [id])
  createdAt DateTime @default(now())
}
```

---

## 🔐 Authentication — Clerk

Clerk handles all auth including sign up, sign in, OAuth, and session management.

**Setup:**

```bash
npm install @clerk/nextjs
```

**middleware.ts:**

```ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isProtectedRoute = createRouteMatcher([
  "/dashboard(.*)",
  "/cars/new(.*)",
  "/my-listings(.*)",
]);

export default clerkMiddleware((auth, req) => {
  if (isProtectedRoute(req)) auth().protect();
});

export const config = {
  matcher: ["/((?!.*\\..*|_next).*)", "/", "/(api|trpc)(.*)"],
};
```

**.env.local:**

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard
```

---

## 🐘 Database — PostgreSQL + Prisma

**Setup:**

```bash
npm install prisma @prisma/client
npx prisma init
```

**lib/prisma.ts:**

```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ?? new PrismaClient({ log: ["query"] });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

**.env:**

```env
DATABASE_URL="postgresql://user:password@localhost:5432/carbazaar"
```

**Run migrations:**

```bash
npx prisma migrate dev --name init
npx prisma generate
npx prisma studio   # GUI to view data
```

---

## ✨ Key Features

### 🔍 Car Listings & Search
- Browse all cars with filters (make, model, year, price, fuel type, city)
- Sort by price, date, mileage
- Paginated results

### 📸 Post a Car for Sale
- Multi-image upload (Uploadthing / Cloudinary)
- Full car details form
- Draft & publish support

### 👤 User Dashboard
- My listings (edit / delete / mark as sold)
- Saved / wishlisted cars
- Enquiries received

### 💬 Buyer–Seller Chat
- In-app messaging between buyer and seller
- WhatsApp link fallback

### 💰 EMI Calculator
- Calculate monthly installments
- Adjust down payment, tenure, interest rate

### ⭐ Reviews & Ratings
- Rate sellers after a deal
- Verified purchase badge

### 📍 Location Features
- City/state based search
- Map view of listings (Google Maps / Mapbox)

### 🛡️ Admin Panel
- Approve / reject listings
- Manage users
- View analytics

---

## 🚀 Getting Started

```bash
# Clone the repo
git clone https://github.com/yourusername/carbazaar.git
cd carbazaar

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Fill in all keys in .env.local

# Run database migrations
npx prisma migrate dev

# Start dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---

## 🌐 Environment Variables

```env
# Clerk Auth
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=

# Database
DATABASE_URL=

# Uploadthing
UPLOADTHING_SECRET=
UPLOADTHING_APP_ID=

# Stripe (optional)
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# Google Maps (optional)
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
```

---

## 📦 Key Dependencies

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "@clerk/nextjs": "^5.0.0",
    "@prisma/client": "^5.0.0",
    "prisma": "^5.0.0",
    "tailwindcss": "^3.4.0",
    "uploadthing": "^6.0.0",
    "stripe": "^14.0.0",
    "zod": "^3.22.0",
    "react-hook-form": "^7.49.0",
    "lucide-react": "^0.300.0",
    "shadcn-ui": "^0.5.0"
  }
}
```

---

## 📄 License

MIT © 2025 CarBazaar
