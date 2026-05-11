# Ticket Intake — TiffinConnect
## How to raise bugs and requirements using Claude

---

### 1. Getting started (one time only)

1. Open **Claude Enterprise**
2. Find the Project named **Ticket Intake — TiffinConnect**
3. That is your entry point for all ticket creation going forward

---

### 2. How to report a bug or requirement

Pick whichever option fits:

**Option A — Just describe it freely**
Open the Project, type or paste what you found.
Claude will ask at most one question, then draft the ticket for your review.

**Option B — Use the template**
Copy the relevant section from **BUG_REPORT_TEMPLATE.md** (pinned in #bugs).
Fill it in and paste it into the Project.

**Option C — Paste from a document**
Copy a section from a PRD, spec, or notes document and paste it directly.
Claude will extract the requirements automatically.

---

### 3. QA — end-of-session batch

After a test session, paste all your findings at once.
Any format works — numbered list, test case failures, free text.
Claude processes everything in one batch.
Review the table it shows you, then type **CONFIRM ALL** to create all tickets.

---

### 4. Commands cheat sheet

| Command | What it does |
|---|---|
| `CONFIRM ALL` | Create all tickets in the current batch |
| `DETAIL [n]` | See full description of ticket n |
| `EDIT [n] [field]=[value]` | Change one field on ticket n |
| `DROP [n]` | Remove ticket n from this batch |
| `FIX [n]` | Add missing detail to ticket n |
| `YES` | Approve a pending action |
| `NO` | Cancel a pending action |
| `BREAKDOWN` | Split a large task across sessions |
