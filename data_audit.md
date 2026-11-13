# Training Data Audit

Original training rows: 1445
Excluded malformed JSON labels: 25
Priority field corrections: 70
Date format corrections: 30
Cleaned training rows: 1420

Cleaning policy:
- Excluded truncated or malformed JSON labels.
- Converted prio low/medium/high to priority P3/P2/P1.
- Converted DD/MM/YYYY dates to YYYY-MM-DD.
- Preserved original ticket text and other label fields.

Note: This regenerated dataset has not been used to
retrain the already evaluated models.
