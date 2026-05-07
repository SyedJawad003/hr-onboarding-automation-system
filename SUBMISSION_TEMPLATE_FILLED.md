# Candidate Submission Template

## Candidate Information
- Full Name: [Your Full Name]
- Email: [Your Email]
- LinkedIn or Portfolio: [Your LinkedIn URL]
- Submission Date: May 8, 2026

---

## Task 1: Intermediate Airtable Skills

### 1. Full Name Formula

**Field Name:** `Full Name`  
**Formula:**
```
TRIM(First Name) & " " & TRIM(Last Name)
```
Uses `TRIM()` on both fields to strip accidental whitespace, then concatenates with a single space separator using `&`. Result example: `"John Doe"`.

---

### 2. Cleaned Email Formula

**Field Name:** `Cleaned Email`  
**Formula:**
```
LOWER(TRIM(SUBSTITUTE(SUBSTITUTE({Email Input}, " ", ""), CHAR(9), "")))
```
- `SUBSTITUTE` removes all space and tab characters from the raw input.
- `TRIM` clears any remaining leading/trailing whitespace.
- `LOWER` normalizes the result to lowercase.
- Handles common data-entry issues: extra spaces, tabs, and mixed-case emails.

---

### 3. Status Categorization Formula

**Field Name:** `Status`  
**Formula:**
```
IF(
  {Amount Spent on Equipment} > 1000,
  "High Value",
  IF(
    AND({Amount Spent on Equipment} >= 500, {Amount Spent on Equipment} <= 999),
    "Medium Value",
    "Low Value"
  )
)
```
- `Amount > 1000` → **High Value**
- `500 ≤ Amount ≤ 999` → **Medium Value**
- `Amount < 500` → **Low Value**

Note: An amount of exactly $1,000 resolves to Medium Value since the threshold is strictly `> 1000`. See `task2_solution.md` Assumptions section for details.

---

### 4. Days Since Created Formula

**Field Name:** `Days Since Created`  
**Formula:**
```
DATETIME_DIFF(TODAY(), {Created Date}, 'days')
```
`DATETIME_DIFF` calculates the number of full days between the `Created Date` and today's date. Updates dynamically each time the base is viewed.

---

## Task 2: Advanced Airtable Automation

### 1. Automation Steps

**Automation Name:** `Notify HR on High Value Hire`

**Step 1 — Trigger:**  
When a record in the New Hires table matches the condition `Status = "High Value"`.

**Step 2 — Conditional Branch (Email Validity Check):**  
Checks: `AND({Cleaned Email} != "", FIND("@", {Cleaned Email}) > 0, LEN({Cleaned Email}) > 5)`
- **Branch A (valid email):** Proceed to Steps 3 and 4.
- **Branch B (missing/invalid email):** Proceed to Step 5 (fallback).

**Step 3 — Send Email (Branch A):**  
Sends a dynamic email to `hr-team@yourcompany.com` including the hire's full name, cleaned email, equipment spend, status, and days since creation.

**Step 4 — Create Record in Tracking Table (Branch A):**  
Logs the high-value hire into a separate **High Value Hire Tracking** table with fields: Hire Name, Email, Equipment Spend, Onboarding Status (default: "Pending Priority Review"), and Logged At timestamp.

**Step 5 — Fallback Email (Branch B):**  
Sends an alert email to HR with the hire's name, original (unvalidated) email input, and a prompt to collect correct contact info.

Full step-by-step detail, formula logic, and flow diagram are in `task2_solution.md`.

---

### 2. Interface Design

**Interface Name:** `HR Onboarding Dashboard`

The interface contains three sections:

**Section 1 — Summary Tiles (top of dashboard):**  
Three numeric count tiles showing the total number of High Value, Medium Value, and Low Value hires. Each tile is connected to the New Hires table with the corresponding Status filter applied.

**Section 2 — Filtered Grid View:**  
A grid element filtered to `Status = "High Value"`, showing columns for Full Name, Cleaned Email, Amount Spent on Equipment, Status, and Days Since Created. Sorted by Amount descending.

**Section 3 — Detail View:**  
Clicking any row in the grid opens a record detail panel displaying all fields for that hire: Full Name, First Name, Last Name, Email Address, Original Email Input, Equipment Budget, Onboarding Priority, and creation date details.

Full layout diagram and configuration notes are in `task2_solution.md`.

> **Screenshots note:** As this is a written submission, screenshots of the interface and automation configuration would be attached as image files in a live Airtable environment (e.g., `interface_overview.png`, `automation_config.png`, `tracking_table.png`).

---

### 3. Formula Logic

**Used in automation conditional branch (email validation):**
```
AND(
  {Cleaned Email} != "",
  FIND("@", {Cleaned Email}) > 0,
  LEN({Cleaned Email}) > 5
)
```

**Automation trigger condition:**
```
{Status} = "High Value"
```

Which resolves via the Task 1 Status formula:
```
IF(
  {Amount Spent on Equipment} > 1000, "High Value",
  IF(AND({Amount Spent on Equipment} >= 500, {Amount Spent on Equipment} <= 999), "Medium Value", "Low Value")
)
```

---

### 4. Assumptions

1. **Airtable plan:** Pro or Business plan assumed (required for conditional branching in automations).
2. **Notification channel:** Email used as primary channel. Slack can be added via a "Send Slack message" action using a webhook in the same automation branch.
3. **Amount = $1,000 exactly:** Treated as Medium Value (threshold is `> 1000`). Adjustable to `>= 1000` if needed.
4. **Formula field trigger:** Automation fires on any record update where Status resolves to "High Value" — either on creation or when Amount is edited.
5. **Tracking Table is separate:** A dedicated High Value Hire Tracking table is used as an audit log, distinct from the New Hires table.
6. **HR email:** `hr-team@yourcompany.com` is a placeholder for the actual HR distribution address.

---

### 5. Optional Notes

- The interface can be extended with a date range filter to scope the dashboard to active onboarding cycles (last 30/60/90 days).
- Slack and email notifications can run in parallel within the same automation branch — both actions can coexist.
- Linking the Tracking Table to New Hires via a Linked Record field would enable bidirectional navigation.
- A second automation could trigger on `Onboarding Status = "Completed"` in the Tracking Table to auto-send a completion confirmation.

Full solution detail for both tasks is available in:
- `submissions/formulas.md` — Task 1 complete formula solutions
- `submissions/task2_solution.md` — Task 2 automation and interface documentation
