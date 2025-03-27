# 🍩 The Donut Daily Summary Engine – Full Spec (v3)
_Last updated: March 27, 2025_

## 🎯 Purpose

Track daily Campaign Monitor sends (DD Campaigns), analyze recipient behavior, extract subscriber origin (`source`) and segmentation (`subid`), and visualize summary metrics in a retro MS-DOS dashboard.

---

## 📅 Daily Objective

The `DD` campaign is The Donut’s **daily newsletter**.

- Extract historical data starting **March 1, 2025**
- Daily update from the most recent campaign
- Track and store recipient behavior
- Attribute each user to a **source** and **subid**
- Generate daily summary metrics
- Visualize 5-day & 30-day campaign summaries
- Allow CSV export of summaries and recipient records

---

## ✅ Dashboard Behavior

- Show **last 5 days** by default
- Toggle to **last 30 days**
- Export:
  - `.csv` summary (open/click rate)
  - `.csv` recipient activity

---

## 📊 Expected Daily Summary (Checksum for March 10–14)

| Date   | Subs | Open | Clicks | Open % | Click % |
|--------|------|------|--------|--------|----------|
| 3/10   | 0    | 0    | 0      | 0.00%  | 0.00%    |
| 3/11   | 585  | 170  | 8      | 29.06% | 1.37%    |
| 3/12   | 1140 | 283  | 9      | 24.82% | 0.79%    |
| 3/13   | 1703 | 465  | 10     | 27.30% | 0.59%    |
| 3/14   | 2253 | 504  | 20     | 22.37% | 0.89%    |

Campaign names:
```
DD March 10, 2025_AllRecipients
...
DD March 14, 2025_AllRecipients
```

---

## 📁 Data Model

### 1. `campaigns`

```sql
CREATE TABLE campaigns (
  id SERIAL PRIMARY KEY,
  campaign_id VARCHAR(50) UNIQUE,
  name TEXT,
  date DATE,
  is_processed BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

### 2. `campaign_recipient_events`

```sql
CREATE TABLE campaign_recipient_events (
  id SERIAL PRIMARY KEY,
  email TEXT,
  name TEXT,
  opens INTEGER DEFAULT 0,
  clicks INTEGER DEFAULT 0,
  source TEXT,
  subid TEXT,
  campaign_id VARCHAR(50),
  campaign_date DATE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

### 3. `daily_summary`

```sql
CREATE TABLE daily_summary (
  id SERIAL PRIMARY KEY,
  date DATE NOT NULL,
  total_subs INTEGER,
  total_opens INTEGER,
  total_clicks INTEGER,
  open_rate NUMERIC(5,2),
  click_rate NUMERIC(5,2),
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🔁 Daily Cron Job Flow

Runs at **9:00 AM MST** (`0 16 * * *` UTC)

1. Fetch all campaigns via:
```http
GET /clients/{clientID}/campaigns.json
```

2. Filter for campaigns starting with `"DD "`  
3. Parse name for date (e.g. `"DD March 14, 2025"` → `2025-03-14`)  
4. Skip if `is_processed = TRUE`  
5. Fetch recipient emails:
```http
GET /campaigns/{campaignID}/recipients.json
```

6. Fetch opens and clicks:
```http
GET /campaigns/{campaignID}/opens.json  
GET /campaigns/{campaignID}/clicks.json
```

7. For each recipient, fetch details:
```http
GET /subscribers/{listID}.json?email={email}
```

8. Extract:
- `source` (via list name or custom field)
- `subid` (if any)
- open/click counts per recipient

9. Insert into:
- `campaign_recipient_events`
- `campaigns`
- `daily_summary` (with calculated metrics)

---

## 🧠 Optional Features

- Label "New subscribers" based on non-existence in previous campaign
- Slack/email notification when daily sync completes
- Re-sync historical campaign manually via admin page
- Flag abnormal bounce or open rates
- Schedule summary CSV exports weekly

---

## 🔐 Required `.env` Variables

```env
CM_API_KEY=...
CM_CLIENT_ID=...
LIST_ID=...
DB_URL=postgres://...
```

---

## 🟢 Summary

You now have:
- A system that tracks all Campaign Monitor DD sends
- Stores every recipient's engagement
- Breaks it down by `source` and `subid`
- Powers a retro DOS-style dashboard
- Scales for deeper segmentation, reporting, and exports

Built for humans.
Styled for terminals.
Powered by The Donut.
