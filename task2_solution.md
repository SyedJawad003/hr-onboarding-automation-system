# Task 2 — Advanced Airtable Automation: Solution

---

## Overview

This document describes the complete automation workflow and interface design built for Task 2 of the HR Onboarding Automation System. The solution builds on the formula fields created in Task 1, specifically the `Status` field that categorizes new hires as High Value, Medium Value, or Low Value.

---

## Part 1: Automation — High Value Hire Workflow

### Automation Name: `Notify HR on High Value Hire`

---

### Step 1 — Trigger: Record Matches Condition

**Trigger Type:** When a record matches conditions  
**Table:** New Hires  
**Condition:**  
```
Status = "High Value"
```

**How it works:**  
Airtable continuously monitors the New Hires table. Whenever a record is created or updated such that the computed `Status` formula field evaluates to `"High Value"` (i.e., `Amount Spent on Equipment > 1000`), this automation fires.

> **Note:** Since `Status` is a formula field (not a manually set field), the trigger fires whenever the underlying data — `Amount Spent on Equipment` — changes in a way that results in the Status becoming "High Value."

---

### Step 2 — Conditional Branch: Check Email Validity

**Action Type:** Conditional (If/Else)  
**Condition:**  
```
AND(
  {Cleaned Email} != "",
  FIND("@", {Cleaned Email}) > 0,
  LEN({Cleaned Email}) > 5
)
```

**Branch A (Email is valid):** Proceed to Steps 3 and 4.  
**Branch B (Email is missing or invalid):** Proceed to Step 5 (fallback notification).

**Logic explanation:**
- Checks that `Cleaned Email` is not blank.
- Checks that the email contains an `@` symbol (basic format validation).
- Checks that the email has a reasonable minimum length (> 5 characters).
- If all conditions are true, the email is treated as valid.

---

### Step 3 (Branch A) — Action: Send Email Notification to HR

**Action Type:** Send an email  
**Configured fields:**

| Field | Value |
|-------|-------|
| **To** | `hr-team@yourcompany.com` (static HR email) |
| **Subject** | `🚨 High Value New Hire Alert: {Full Name}` |
| **Body** | Dynamic (see below) |

**Dynamic Email Body:**
```
Hi HR Team,

A new hire has been categorized as HIGH VALUE and requires priority onboarding attention.

--- New Hire Details ---
Full Name:         {Full Name}
Email:             {Cleaned Email}
Equipment Spend:   ${Amount Spent on Equipment}
Status:            {Status}
Record Created:    {Created Date}
Days Since Created:{Days Since Created} day(s) ago

Please initiate priority onboarding for this hire immediately.

This is an automated notification from the HR Onboarding System.
```

---

### Step 4 (Branch A) — Action: Create Record in Tracking Table

**Action Type:** Create record  
**Table:** High Value Hire Tracking *(see schema below)*

**Field Mappings:**

| Tracking Table Field | Value |
|---------------------|-------|
| Hire Name | `{Full Name}` from trigger record |
| Email | `{Cleaned Email}` from trigger record |
| Equipment Spend | `{Amount Spent on Equipment}` |
| Onboarding Status | `"Pending Priority Review"` (static default) |
| Logged At | Use Airtable's `NOW()` or set via Created Time field |
| Source Record ID | `{Record ID}` of the trigger record |

**Tracking Table Schema:**

| Field Name | Field Type | Notes |
|-----------|-----------|-------|
| Hire Name | Single line text | Linked from New Hires |
| Email | Email | Validated email |
| Equipment Spend | Currency | Amount from trigger |
| Onboarding Status | Single select | Pending / In Progress / Completed |
| Logged At | Created time | Auto-set when record is created |
| Source Record ID | Single line text | For cross-referencing |

---

### Step 5 (Branch B) — Action: Fallback — Notify HR of Missing Email

**Action Type:** Send an email  
**Configured fields:**

| Field | Value |
|-------|-------|
| **To** | `hr-team@yourcompany.com` |
| **Subject** | `⚠️ High Value Hire Alert — Missing/Invalid Email: {Full Name}` |
| **Body** | See below |

**Dynamic Email Body:**
```
Hi HR Team,

A new hire has been categorized as HIGH VALUE, but their email address is missing or invalid.

--- New Hire Details ---
Full Name:       {Full Name}
Email on File:   {Email Input}  ← Unvalidated original input
Equipment Spend: ${Amount Spent on Equipment}
Status:          {Status}
Created:         {Created Date}

ACTION REQUIRED:
Please contact this hire through alternative means and update their email address in the system.

This is an automated notification from the HR Onboarding System.
```

---

### Automation Flow Diagram

```
[TRIGGER]
Record in "New Hires" matches condition: Status = "High Value"
        |
        v
[CONDITION CHECK]
Is Cleaned Email valid?
(not blank + contains "@" + length > 5)
        |
   _____|_____
  |           |
 YES          NO
  |           |
  v           v
[STEP 3]    [STEP 5]
Send email  Send fallback
to HR with  alert to HR:
hire details "Missing Email"
  |
  v
[STEP 4]
Log record into
High Value Hire
Tracking Table
```

---

## Part 2: Interface Design

### Interface Name: `HR Onboarding Dashboard`

The interface is built using Airtable Interface Designer and contains three main sections.

---

### Section 1 — Summary Tiles

**Layout:** Three side-by-side numeric summary tiles at the top of the dashboard.

| Tile | Label | Data Source | Filter |
|------|-------|-------------|--------|
| Tile 1 | 🔴 High Value Hires | New Hires table | Status = "High Value" |
| Tile 2 | 🟡 Medium Value Hires | New Hires table | Status = "Medium Value" |
| Tile 3 | 🟢 Low Value Hires | New Hires table | Status = "Low Value" |

**Configuration steps in Interface Designer:**
1. Add a **Number element** from the element panel.
2. Connect it to the **New Hires** table.
3. Set the aggregation to **Count of records**.
4. Apply the appropriate filter per tile (e.g., `Status = "High Value"`).
5. Label each tile with the corresponding category name and an emoji indicator.

---

### Section 2 — Filtered Grid View: High Value Hires

**Element Type:** Grid  
**Table:** New Hires  
**Filter:** `Status = "High Value"`  
**Visible Fields:**

| Field | Notes |
|-------|-------|
| Full Name | Computed formula field |
| Cleaned Email | Validated email |
| Amount Spent on Equipment | Currency |
| Status | Should display "High Value" for all rows |
| Days Since Created | Computed formula field |
| Created Date | Original date |

**Sorting:** `Amount Spent on Equipment` — descending (highest spenders first).

**Purpose:** Gives HR teams an at-a-glance list of all high-value hires that need priority attention, sorted by equipment spend.

---

### Section 3 — Detail View per New Hire

**Element Type:** Record detail / expand record  
**Trigger:** Click on any row in the High Value grid (Section 2).

**Fields displayed in detail view:**

| Field | Display Label |
|-------|--------------|
| Full Name | Full Name |
| First Name | First Name |
| Last Name | Last Name |
| Cleaned Email | Email Address |
| Email Input | Original Email Input |
| Amount Spent on Equipment | Equipment Budget |
| Status | Onboarding Priority |
| Days Since Created | Days Since Record Created |
| Created Date | Record Creation Date |

**Purpose:** Allows HR staff to drill into each individual hire's profile to review all onboarding details, verify contact information, and take action.

---

### Interface Layout Summary

```
┌─────────────────────────────────────────────────────────────┐
│           HR ONBOARDING DASHBOARD                           │
├────────────────┬────────────────────┬───────────────────────┤
│  🔴 HIGH VALUE │  🟡 MEDIUM VALUE   │  🟢 LOW VALUE         │
│     Count: 12  │     Count: 34      │     Count: 51         │
├────────────────┴────────────────────┴───────────────────────┤
│  HIGH VALUE HIRES — GRID VIEW                               │
│  ┌──────────────┬──────────────────┬──────────┬───────────┐ │
│  │ Full Name    │ Email            │ Amount   │ Days Ago  │ │
│  ├──────────────┼──────────────────┼──────────┼───────────┤ │
│  │ Jane Smith   │ jane@example.com │ $2,400   │ 3         │ │
│  │ John Doe     │ john@example.com │ $1,800   │ 7         │ │
│  │ ...          │ ...              │ ...      │ ...       │ │
│  └──────────────┴──────────────────┴──────────┴───────────┘ │
│                                                             │
│  [Click any row to open detail view →]                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 3: Formula Logic Used in Automation

The automation relies on the following formula conditions for its conditional branching step:

### Email Validation Check (used in Step 2):
```
AND(
  {Cleaned Email} != "",
  FIND("@", {Cleaned Email}) > 0,
  LEN({Cleaned Email}) > 5
)
```

### High Value Condition (used as automation trigger):
```
{Status} = "High Value"
```

Which internally resolves via the Status formula from Task 1:
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

---

## Part 4: Assumptions

1. **Airtable plan supports automations:** The automation features described (conditional branching, email actions, create record actions) require an Airtable Pro or Business plan. A free plan allows only basic automations.

2. **HR email address:** The recipient address `hr-team@yourcompany.com` is a placeholder. In production, this would be replaced with the actual HR team distribution list or a specific HR manager's email.

3. **Slack vs Email:** The instructions mention Slack or Email. This solution uses **email** as the primary notification channel since it requires no additional third-party integration setup. To enable Slack, the same automation step can be replaced with a **Send a Slack message** action, pointing to an `#hr-alerts` channel using a Slack webhook integration configured in Airtable.

4. **Amount = 1000 exactly:** An amount of exactly `$1,000` is treated as **Medium Value** (the threshold is `> 1000`, not `≥ 1000`). This can be adjusted to `>= 1000` if the business intent includes $1,000 as High Value.

5. **Formula field trigger behavior:** Since `Status` is a computed formula field (not a user-editable field), Airtable's automation trigger monitors underlying data changes. The trigger fires when a record is updated in a way that the Status formula resolves to "High Value" — either on initial record creation or upon editing of `Amount Spent on Equipment`.

6. **Tracking Table is separate:** The High Value Hire Tracking table is a new, standalone table within the same Airtable base. It is not the same as the New Hires table. This separation allows HR to maintain a clean audit log of high-value onboarding events.

7. **No personal data stored externally:** All data stays within Airtable. Email notifications contain record data but no automation sends data to external storage beyond Airtable's own tables.

---

## Optional Notes

- The interface can be further improved by adding a **date range filter** to show only hires from the past 30/60/90 days, making the dashboard more operationally relevant for active onboarding cycles.
- A **Slack integration** can be added in parallel with the email step (both actions can coexist in the same automation branch) to ensure HR receives alerts through multiple channels.
- The Tracking Table could be linked to the New Hires table using a **Linked Record** field for tighter relational integrity, allowing bidirectional navigation between the two tables in the interface.
- Future iterations could trigger a second automation when a tracked hire's `Onboarding Status` is updated to `"Completed"` to send a completion confirmation to the hire.
