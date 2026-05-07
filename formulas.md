# Task 1 — Intermediate Airtable Skills: Formula Solutions

## Table: New Hires

Fields used across all formulas:
- `First Name` (Single line text)
- `Last Name` (Single line text)
- `Email Input` (Single line text)
- `Amount Spent on Equipment` (Number)
- `Created Date` (Date)

---

## Formula 1 — Full Name

**Field Name:** `Full Name`  
**Field Type:** Formula → Result type: Text

### Formula:
```
TRIM(First Name) & " " & TRIM(Last Name)
```

### Explanation:
- `TRIM()` removes any accidental leading/trailing spaces from both name fields before joining.
- The `&` operator concatenates the cleaned first name, a space, and the cleaned last name.
- Result example: `"John"` + `" "` + `"Doe"` → `"John Doe"`

---

## Formula 2 — Cleaned Email

**Field Name:** `Cleaned Email`  
**Field Type:** Formula → Result type: Text

### Formula:
```
LOWER(TRIM(SUBSTITUTE(SUBSTITUTE({Email Input}, " ", ""), CHAR(9), "")))
```

### Explanation:
- `SUBSTITUTE({Email Input}, " ", "")` — removes all space characters from the email.
- The second `SUBSTITUTE(...)` removes tab characters (`CHAR(9)`) which can appear in copy-pasted data.
- `TRIM()` removes any residual leading/trailing whitespace.
- `LOWER()` normalizes the entire email to lowercase (e.g., `"John.DOE@Gmail.COM"` → `"john.doe@gmail.com"`).
- This handles the most common data-entry issues: extra spaces, tabs, and mixed case.

### Alternative (simpler) formula if only spaces need removing:
```
LOWER(TRIM({Email Input}))
```

---

## Formula 3 — Status Categorization

**Field Name:** `Status`  
**Field Type:** Formula → Result type: Text

### Formula:
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

### Explanation:
- The outer `IF` checks whether the amount is **greater than 1000** → assigns `"High Value"`.
- The nested `IF` checks whether the amount is **between 500 and 999 (inclusive)** using `AND()` → assigns `"Medium Value"`.
- If neither condition is met (amount < 500), the field defaults to `"Low Value"`.
- Logic summary:
  - `Amount > 1000` → **High Value**
  - `500 ≤ Amount ≤ 999` → **Medium Value**
  - `Amount < 500` → **Low Value**

### Edge Cases Handled:
- An amount of exactly `1000` falls into `Medium Value` since the condition is `> 1000` (strictly greater than). If the requirement intended `≥ 1000` for High Value, the formula can be adjusted to `{Amount Spent on Equipment} >= 1000`.
- A blank/empty amount field will return `"Low Value"` (blank is treated as 0 by Airtable).

---

## Formula 4 — Days Since Created

**Field Name:** `Days Since Created`  
**Field Type:** Formula → Result type: Number

### Formula:
```
DATETIME_DIFF(TODAY(), {Created Date}, 'days')
```

### Explanation:
- `TODAY()` returns the current date at the time the record is viewed.
- `{Created Date}` is the date the new hire record was created.
- `DATETIME_DIFF(end, start, unit)` calculates the difference between two dates in the specified unit — here `'days'`.
- Result is a positive integer representing how many days have passed since the record was created.
- Example: if `Created Date` = `2025-01-01` and today is `2025-03-28`, the result = `86`.

### Alternative (equivalent) formula:
```
INT(( TODAY() - {Created Date} ) / 86400)
```
> This divides the timestamp difference (in seconds) by 86400 seconds/day. The `DATETIME_DIFF` approach is preferred as it is more readable.

---

## Summary Table

| # | Field Name | Formula | Purpose |
|---|-----------|---------|---------|
| 1 | Full Name | `TRIM(First Name) & " " & TRIM(Last Name)` | Combines first and last name |
| 2 | Cleaned Email | `LOWER(TRIM(SUBSTITUTE(SUBSTITUTE({Email Input}, " ", ""), CHAR(9), "")))` | Removes spaces/tabs, lowercases email |
| 3 | Status | Nested `IF` on Amount | Categorizes hire as High/Medium/Low Value |
| 4 | Days Since Created | `DATETIME_DIFF(TODAY(), {Created Date}, 'days')` | Days elapsed since record creation |

---

## Schema Setup Notes

To implement these formulas in Airtable:

1. Open your **New Hires** table.
2. Click the **+** button to add a new field.
3. Choose **Formula** as the field type.
4. Paste the relevant formula from above.
5. Rename the field to match the **Field Name** column above.
6. Click **Save**.

> **Note:** Screenshots of the applied formulas in the Airtable base are included in the submission folder as `task1_screenshots/` (see accompanying image files).
