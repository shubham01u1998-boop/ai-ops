# Discovery Call Notes — SupplySync
# Call date: April 14, 2026
# Attendees: Rohan Kapoor (Prakash Distributors, Head of Procurement), fiftyfive-tech consultant

---

## Key points from Rohan

**Current pain:**
- Procurement team is 4 people, all using separate Excel files — no single source of truth
- Had a situation last month where two managers raised POs to the same supplier for the same item — ended up with double inventory worth 2L
- Contract renewals are tracked in a personal Google Sheet by Rohan himself — "if I leave, no one knows what's expiring"

**Users:**
- Primary: 4 procurement managers (internal, Prakash Distributors)
- Rohan himself (Procurement Head) — needs a dashboard view
- Suppliers: should be able to receive notifications but NOT have login access initially
  - "We don't want suppliers logging in — too complicated for them. Just send them links."

**Tech:**
- They have a small in-house IT team but not developers
- Rohan mentioned: "Our IT guy uses Vue.js for our internal tools — if your team can match that it's easier for us to maintain later"
- (Note: spec doc says React — need to clarify which is actually preferred)

**Timeline:**
- Rohan said the CEO wants this live before their annual supplier summit
- "The summit is in December. CEO has already told suppliers a new system is coming."
- (Note: spec says Q3 2026 — December is Q4. Need to clarify which is the real deadline)

**Scale:**
- ~80 suppliers currently, expected to grow to 120–150 in 2 years
- Order volume: roughly 200–300 POs per month
- No specific storage requirement mentioned

**Important requirements Rohan mentioned that are NOT in the spec:**
- WhatsApp notifications in addition to email — "all our suppliers are on WhatsApp, email goes unread"
- The system must be able to send a WhatsApp message when a PO is confirmed
- Language: English for the platform, but WhatsApp messages might need Hindi or Marathi option

**Deferred:**
- ERP integration (SAP B1) — Rohan knows it's deferred but wants it in Phase 2
- Mobile app — not mentioned as a priority, website being mobile-friendly is sufficient

**Budget:**
- Not discussed on this call. Rohan said "our CFO handles that, I'll connect you."
- (Budget is unknown)
