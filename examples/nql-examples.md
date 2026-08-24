# Example questions — Arabic and English

All examples follow the pattern: natural-language question → generated (read-only) SQL.

## Retail / FMCG

**EN:** "Show me sales by branch for the last quarter, ranked."
```sql
SELECT b.name, SUM(o.total_amount) AS sales
FROM orders o JOIN branches b ON b.id = o.branch_id
WHERE o.order_date >= DATE_TRUNC('quarter', CURRENT_DATE - INTERVAL '3 months')
GROUP BY b.name
ORDER BY sales DESC;
```

**AR:** ما هو متوسط قيمة الطلب في كل فرع هذا الشهر؟
```sql
SELECT b.name, AVG(o.total_amount) AS avg_order_value
FROM orders o JOIN branches b ON b.id = o.branch_id
WHERE DATE_TRUNC('month', o.order_date) = DATE_TRUNC('month', CURRENT_DATE)
GROUP BY b.name;
```

## Healthcare

**EN:** "How many appointments per department in the last 30 days?"
```sql
SELECT department, COUNT(*) AS appointments
FROM appointments
WHERE appointment_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY department
ORDER BY appointments DESC;
```

**AR:** ما هو متوسط فترة الانتظار في قسم الطوارئ الأسبوع الماضي؟
```sql
SELECT AVG(wait_minutes) AS avg_wait
FROM emergency_visits
WHERE DATE_TRUNC('week', visit_date) = DATE_TRUNC('week', CURRENT_DATE - INTERVAL '1 week');
```

## Logistics / Distribution

**EN:** "Which routes had delivery delays last month, and how many?"
```sql
SELECT route_name, COUNT(*) AS delayed_deliveries
FROM deliveries
WHERE status = 'delayed'
  AND DATE_TRUNC('month', delivery_date) = DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
GROUP BY route_name
ORDER BY delayed_deliveries DESC;
```

**AR:** كم نسبة الطلبات المتأخرة في كل مدينة هذا الشهر؟
```sql
SELECT city,
       ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'delayed') / COUNT(*), 2) AS delay_pct
FROM deliveries
WHERE DATE_TRUNC('month', delivery_date) = DATE_TRUNC('month', CURRENT_DATE)
GROUP BY city
ORDER BY delay_pct DESC;
```

## Manufacturing

**EN:** "What was our OEE for production lines last week?"
```sql
SELECT line_name, ROUND(AVG(oee_pct), 1) AS avg_oee
FROM production_runs
WHERE DATE_TRUNC('week', run_date) = DATE_TRUNC('week', CURRENT_DATE - INTERVAL '1 week')
GROUP BY line_name;
```

**AR:** كم عدد أوامر الصيانة المفتوحة لكل خط إنتاج؟
```sql
SELECT line_name, COUNT(*) AS open_work_orders
FROM maintenance_orders
WHERE status = 'open'
GROUP BY line_name;
```

---

*Every generated query is validated read-only, role-scoped, and logged to the audit trail.*
