From: Arjun Mehta <arjun@fiftyfive-tech.io>
To: Project Lead
Date: April 14, 2026
Subject: StyleMart — tech concerns before we commit

Read the brief. Some issues we need to resolve before architecture.

**1. Shopify is wrong for this project.**
Client mentioned loyalty points and a possible marketplace direction.
Shopify loyalty requires paid third-party apps ($50–200/month ongoing) with limited customisation.
Multi-vendor is simply not possible on standard Shopify — requires a full platform migration later.
Recommendation: custom React + Node.js + PostgreSQL. Higher initial build cost, full control, no lock-in.

**2. Mobile-first, not Phase 2.**
Notes say the client's customers are on Instagram and WhatsApp — that is a mobile audience.
Launching web-only will hurt conversion even with a responsive site.
Recommendation: React Native from day 1 — web and mobile share most of the codebase.
This adds roughly 6–8 weeks. Realistic go-live with this approach: Q2 2027.

**3. Multi-vendor — need a hard answer.**
If there is even a 20% chance of marketplace in the future, the data model needs multi-tenant
architecture from day 1. Retrofitting single-tenant → multi-tenant is expensive and painful.
This is a blocking question for architecture. We need YES or NO from the client.

**4. COD is not optional in Indian e-commerce.**
"To be confirmed" in the brief is not acceptable for an architectural decision.
COD typically drives 40–60% higher return rates. Returns, refund flow, and logistics integration
all need to be designed for COD from the start.
Get this confirmed before we touch the checkout flow.

— Arjun
