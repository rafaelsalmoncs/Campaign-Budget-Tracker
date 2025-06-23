
# 📄 Pseudo-code: Django + Celery Budget Management System

## 📌 Objective
Design a backend system to manage ad budgets across multiple brands and campaigns, with automated enforcement of spend limits and dayparting using Django and Celery.

---

## 🧱 Data Models (Language-Agnostic)

### 🏢 Brand
- **Fields**
  - `name`: string
  - `daily_budget`: float
  - `monthly_budget`: float
  - `current_daily_spend`: float
  - `current_monthly_spend`: float
  - `is_active`: boolean

### 📣 Campaign
- **Fields**
  - `brand`: foreign key → Brand
  - `name`: string
  - `is_active`: boolean
  - `daily_spend`: float
  - `schedule`: foreign key → Schedule
  - `created_at`: datetime

### 🕒 Schedule (Dayparting)
- **Fields**
  - `campaign`: foreign key → Campaign (or one-to-one)
  - `start_hour`: integer (0–23)
  - `end_hour`: integer (0–23)

---

## 🔁 Core System Logic

### 🔹 Spend Tracking
```pseudocode
on_spend_event(campaign_id, amount):
    campaign = get_campaign(campaign_id)
    brand = campaign.brand

    campaign.daily_spend += amount
    brand.current_daily_spend += amount
    brand.current_monthly_spend += amount

    if brand.current_daily_spend > brand.daily_budget or
       brand.current_monthly_spend > brand.monthly_budget:
        campaign.is_active = False
        save(campaign)
```

---

### 🔹 Budget Enforcement
```pseudocode
periodic_task(enforce_budget_limits):
    for each campaign in all_campaigns:
        if campaign.brand.current_daily_spend > campaign.brand.daily_budget or
           campaign.brand.current_monthly_spend > campaign.brand.monthly_budget:
            campaign.is_active = False
        else:
            if is_within_schedule(campaign.schedule):
                campaign.is_active = True
        save(campaign)
```

---

### 🔹 Dayparting Checks
```pseudocode
is_within_schedule(schedule):
    current_hour = get_current_hour()
    return schedule.start_hour <= current_hour < schedule.end_hour
```

```pseudocode
periodic_task(enforce_dayparting):
    for each campaign in all_campaigns:
        if not is_within_schedule(campaign.schedule):
            campaign.is_active = False
        else if budgets_are_ok(campaign.brand):
            campaign.is_active = True
        save(campaign)
```

---

### 🔹 Daily Reset
```pseudocode
daily_midnight_task(reset_daily_budgets):
    for each brand in all_brands:
        brand.current_daily_spend = 0
        save(brand)

    for each campaign in all_campaigns:
        campaign.daily_spend = 0
        if budgets_are_ok(campaign.brand) and is_within_schedule(campaign.schedule):
            campaign.is_active = True
        save(campaign)
```

---

### 🔹 Monthly Reset
```pseudocode
first_of_month_task(reset_monthly_budgets):
    for each brand in all_brands:
        brand.current_monthly_spend = 0
        save(brand)
```

---

## ✅ Additional Notes
- Celery periodic tasks handle:
  - Budget enforcement (every few minutes)
  - Dayparting (hourly)
  - Daily resets (midnight)
  - Monthly resets (first of the month)
- Use static typing in all implementations (Python type hints).
- Ensure idempotency of scheduled tasks to prevent double updates.
