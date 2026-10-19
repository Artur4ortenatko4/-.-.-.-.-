# Звіт до лабораторної роботи № 2
## Створення складних SQL-запитів

**Виконав:** Юрдамян Артур Торосович  
**Група:** ІПЗ-33  
**СУБД:** PostgreSQL (Supabase)

## 1. Мета роботи

Набути практичних навичок створення складних SQL-запитів, з'єднання таблиць, використання агрегатних і віконних функцій, підзапитів, CTE та засобів оптимізації аналітичних запитів.

## 2. Хід роботи

Запити розраховані на навчальну базу TechnoMart, яка містить таблиці `customers`, `orders`, `order_items`, `products`, `categories`, `suppliers`, `regions` та `employees`. Запити потрібно виконати у SQL Editor Supabase. Фактичні результати залежать від даних у базі.

## 3. Рівень 1 — з'єднання таблиць

### 3.1. INNER JOIN: товари, категорії та постачальники

```sql
-- Вивести товар разом із його категорією та постачальником
SELECT p.product_name,
       c.category_name,
       s.company_name AS supplier,
       p.unit_price
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN suppliers s ON p.supplier_id = s.supplier_id
ORDER BY c.category_name, p.product_name;
```

**Пояснення:** `INNER JOIN` повертає лише ті товари, для яких знайдено відповідні категорію та постачальника.

### 3.2. LEFT JOIN: усі клієнти та кількість їхніх замовлень

```sql
SELECT c.customer_id,
       c.contact_name,
       c.customer_type,
       r.region_name,
       COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.contact_name, c.customer_type, r.region_name
ORDER BY order_count DESC, c.contact_name;
```

**Пояснення:** `LEFT JOIN` зберігає в результаті всіх клієнтів, навіть якщо замовлень немає. `COUNT(o.order_id)` для клієнта без замовлень дасть 0.

### 3.3. Множинне з'єднання: деталі замовлень

```sql
-- Поєднати замовлення, клієнтів, позиції замовлення, товари й категорії
SELECT o.order_id,
       o.order_date,
       o.order_status,
       cu.contact_name,
       p.product_name,
       c.category_name,
       oi.quantity,
       oi.unit_price,
       oi.discount,
       oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0)) AS line_total
FROM orders o
JOIN customers cu ON cu.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p ON p.product_id = oi.product_id
JOIN categories c ON c.category_id = p.category_id
ORDER BY o.order_date DESC, o.order_id, p.product_name;
```

**Пояснення:** запит збирає інформацію з п'яти пов'язаних таблиць і розраховує суму позиції з урахуванням знижки.

## 4. Агрегатні функції та групування

### 4.1. Кількість і ціни товарів за категоріями

```sql
SELECT c.category_name,
       COUNT(p.product_id) AS product_count,
       ROUND(AVG(p.unit_price), 2) AS avg_price,
       MIN(p.unit_price) AS min_price,
       MAX(p.unit_price) AS max_price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY product_count DESC, c.category_name;
```

### 4.2. Загальні продажі за регіонами

```sql
SELECT r.region_name,
       SUM(oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0))) AS total_sales,
       COUNT(DISTINCT o.order_id) AS orders_count
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
JOIN customers cu ON cu.customer_id = o.customer_id
LEFT JOIN regions r ON r.region_id = cu.region_id
WHERE o.order_status = 'delivered'
GROUP BY r.region_id, r.region_name
HAVING SUM(oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0))) > 0
ORDER BY total_sales DESC;
```

**Пояснення:** `SUM` підсумовує продажі, а `HAVING` залишає лише групи з позитивною сумою продажів.

### 4.3. Постачальники, які мають більше двох товарів

```sql
SELECT s.supplier_id,
       s.company_name,
       COUNT(p.product_id) AS product_count
FROM suppliers s
JOIN products p ON p.supplier_id = s.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY product_count DESC, s.company_name;
```

## 5. Підзапити

### 5.1. Товари дорожчі за середню ціну у своїй категорії

```sql
SELECT p.product_name,
       p.unit_price,
       c.category_name
FROM products p
JOIN categories c ON c.category_id = p.category_id
WHERE p.unit_price > (
    SELECT AVG(p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.category_name, p.unit_price DESC;
```

### 5.2. Клієнти, які мали замовлення у 2024 році

```sql
SELECT c.customer_id, c.contact_name
FROM customers c
WHERE c.customer_id IN (
    SELECT o.customer_id
    FROM orders o
    WHERE o.order_date >= DATE '2024-01-01'
      AND o.order_date < DATE '2025-01-01'
)
ORDER BY c.contact_name;
```

### 5.3. Кількість проданих одиниць для кожного товару

```sql
SELECT p.product_id,
       p.product_name,
       COALESCE((
           SELECT SUM(oi.quantity)
           FROM order_items oi
           WHERE oi.product_id = p.product_id
       ), 0) AS units_sold
FROM products p
ORDER BY units_sold DESC, p.product_name;
```

### 5.4. EXISTS: клієнти, що мають хоча б одне замовлення

```sql
SELECT c.customer_id, c.contact_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
ORDER BY c.contact_name;
```

## 6. Рівень 2 — складні з'єднання та аналітика

### 6.1. RIGHT JOIN: усі категорії, навіть порожні

```sql
SELECT c.category_name,
       COUNT(p.product_id) AS products_count,
       COALESCE(AVG(p.unit_price), 0) AS avg_price
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY products_count DESC, c.category_name;
```

### 6.2. Self-join: співробітники та їхні керівники

```sql
SELECT e.first_name || ' ' || e.last_name AS employee,
       e.title AS employee_title,
       m.first_name || ' ' || m.last_name AS manager,
       m.title AS manager_title
FROM employees e
LEFT JOIN employees m ON e.reports_to = m.employee_id
ORDER BY manager, employee;
```

### 6.3. Умовне з'єднання

```sql
-- Усі клієнти та лише їхні доставлені замовлення, якщо такі є
SELECT c.contact_name,
       o.order_id,
       o.order_date,
       o.order_status
FROM customers c
LEFT JOIN orders o
  ON o.customer_id = c.customer_id
 AND o.order_status = 'delivered'
ORDER BY c.contact_name, o.order_date DESC;
```

**Пояснення:** умова статусу розміщена в `ON`, тому клієнти без доставлених замовлень не зникають з результату.

### 6.4. FULL OUTER JOIN

```sql
-- Показати всі категорії та товари, включно з рядками без відповідності
SELECT c.category_name, p.product_name
FROM categories c
FULL OUTER JOIN products p
  ON p.category_id = c.category_id
ORDER BY c.category_name NULLS LAST, p.product_name NULLS LAST;
```

### 6.5. ROW_NUMBER, RANK та DENSE_RANK

```sql
SELECT p.product_name,
       c.category_name,
       p.unit_price,
       ROW_NUMBER() OVER (
           PARTITION BY c.category_name
           ORDER BY p.unit_price DESC, p.product_name
       ) AS row_num,
       RANK() OVER (
           PARTITION BY c.category_name
           ORDER BY p.unit_price DESC
       ) AS price_rank,
       DENSE_RANK() OVER (
           PARTITION BY c.category_name
           ORDER BY p.unit_price DESC
       ) AS price_dense_rank
FROM products p
JOIN categories c ON c.category_id = p.category_id
ORDER BY c.category_name, p.unit_price DESC;
```

### 6.6. LAG та LEAD: попереднє і наступне замовлення клієнта

```sql
SELECT customer_id,
       order_id,
       order_date,
       LAG(order_date) OVER (
           PARTITION BY customer_id ORDER BY order_date, order_id
       ) AS previous_order_date,
       LEAD(order_date) OVER (
           PARTITION BY customer_id ORDER BY order_date, order_id
       ) AS next_order_date,
       order_date - LAG(order_date) OVER (
           PARTITION BY customer_id ORDER BY order_date, order_id
       ) AS days_since_previous
FROM orders
ORDER BY customer_id, order_date, order_id;
```

## 7. Рівень 3 — оптимізація та складна аналітика

### 7.1. Матеріалізоване представлення місячних продажів

```sql
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT
    EXTRACT(YEAR FROM o.order_date)::int AS sales_year,
    EXTRACT(MONTH FROM o.order_date)::int AS sales_month,
    c.category_name,
    r.region_name,
    SUM(oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0))) AS total_revenue,
    COUNT(DISTINCT o.order_id) AS orders_count,
    AVG(oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0))) AS avg_line_value
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p ON p.product_id = oi.product_id
JOIN categories c ON c.category_id = p.category_id
JOIN customers cu ON cu.customer_id = o.customer_id
LEFT JOIN regions r ON r.region_id = cu.region_id
WHERE o.order_status = 'delivered'
GROUP BY
    EXTRACT(YEAR FROM o.order_date)::int,
    EXTRACT(MONTH FROM o.order_date)::int,
    c.category_name,
    r.region_name;

-- Індекс для пошуку за роком і місяцем
CREATE INDEX idx_mv_monthly_sales_date
ON mv_monthly_sales (sales_year, sales_month);

-- Перегляд підготовленої аналітики
SELECT *
FROM mv_monthly_sales
ORDER BY sales_year, sales_month, total_revenue DESC;

-- Оновлення даних у матеріалізованому представленні після змін у джерелі
REFRESH MATERIALIZED VIEW mv_monthly_sales;
```

**Пояснення:** матеріалізоване представлення зберігає результат агрегатного запиту, тому повторне читання аналітики може бути швидшим. Воно не оновлюється автоматично після зміни базових таблиць — для оновлення застосовують `REFRESH MATERIALIZED VIEW`.

### 7.2. Рекурсивний CTE для ієрархії співробітників

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Базовий випадок: співробітники без керівника
    SELECT employee_id,
           first_name,
           last_name,
           title,
           reports_to,
           0 AS level,
           CAST(last_name || ' ' || first_name AS VARCHAR(1000)) AS hierarchy_path
    FROM employees
    WHERE reports_to IS NULL

    UNION ALL

    -- Рекурсивний випадок: підлеглі знайдених співробітників
    SELECT e.employee_id,
           e.first_name,
           e.last_name,
           e.title,
           e.reports_to,
           eh.level + 1,
           CAST(eh.hierarchy_path || ' -> ' ||
                e.last_name || ' ' || e.first_name AS VARCHAR(1000))
    FROM employees e
    JOIN employee_hierarchy eh ON e.reports_to = eh.employee_id
    WHERE eh.level < 10
)
SELECT employee_id, first_name, last_name, title, reports_to, level, hierarchy_path
FROM employee_hierarchy
ORDER BY hierarchy_path;
```

### 7.3. Параметризована аналітика через функцію PostgreSQL

```sql
-- Функція повертає продажі за вказаний період
CREATE OR REPLACE FUNCTION sales_by_period(
    p_start_date DATE,
    p_end_date DATE
)
RETURNS TABLE (
    category_name TEXT,
    total_revenue NUMERIC,
    orders_count BIGINT
)
LANGUAGE sql
AS $$
    SELECT c.category_name::TEXT,
           SUM(oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0)))::NUMERIC AS total_revenue,
           COUNT(DISTINCT o.order_id) AS orders_count
    FROM orders o
    JOIN order_items oi ON oi.order_id = o.order_id
    JOIN products p ON p.product_id = oi.product_id
    JOIN categories c ON c.category_id = p.category_id
    WHERE o.order_status = 'delivered'
      AND o.order_date >= p_start_date
      AND o.order_date < p_end_date
    GROUP BY c.category_name
    ORDER BY total_revenue DESC;
$$;

-- Приклад виклику для календарного 2024 року
SELECT *
FROM sales_by_period(DATE '2024-01-01', DATE '2025-01-01');
```

## 8. Аналіз продуктивності та індекси

### 8.1. Перегляд плану виконання

```sql
EXPLAIN ANALYZE
SELECT o.order_id,
       o.order_date,
       cu.contact_name,
       SUM(oi.quantity * oi.unit_price * (1 - COALESCE(oi.discount, 0))) AS order_total
FROM orders o
JOIN customers cu ON cu.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
WHERE o.order_status = 'delivered'
GROUP BY o.order_id, o.order_date, cu.contact_name
ORDER BY o.order_date DESC;
```

`EXPLAIN ANALYZE` показує фактичний план виконання та час роботи запиту. Визначити найповільніший запит можна лише після запуску на реальній базі та порівняння показників `actual time`, кількості рядків і типів сканування.

### 8.2. Індекси для типових з'єднань і фільтрів

```sql
-- Індекси для ключів з'єднання та пошуку замовлень
CREATE INDEX IF NOT EXISTS idx_orders_customer_id
    ON orders(customer_id);

CREATE INDEX IF NOT EXISTS idx_orders_status_date
    ON orders(order_status, order_date);

CREATE INDEX IF NOT EXISTS idx_order_items_order_id
    ON order_items(order_id);

CREATE INDEX IF NOT EXISTS idx_order_items_product_id
    ON order_items(product_id);

CREATE INDEX IF NOT EXISTS idx_products_category_id
    ON products(category_id);

CREATE INDEX IF NOT EXISTS idx_products_supplier_id
    ON products(supplier_id);
```

**Способи оптимізації:**

1. Створювати індекси на стовпцях, які часто використовуються у `JOIN`, `WHERE` та `ORDER BY`.
2. Вибирати лише потрібні стовпці замість `SELECT *`.
3. Фільтрувати дані до агрегації, коли це не змінює логіку результату.
4. Для повторюваних важких звітів розглядати матеріалізовані представлення.
5. Перевіряти користь індексів через `EXPLAIN ANALYZE`; зайві індекси збільшують витрати на запис і зберігання.
6. Після створення матеріалізованого представлення індексувати його за полями, за якими часто виконують пошук.

## 9. Оцінка складності запитів

| Запит | Орієнтовна складність | Пояснення |
|---|---|---|
| Простий `SELECT` з `WHERE` | Низька | Одна таблиця, фільтрація рядків |
| `INNER JOIN` трьох таблиць | Середня | Потрібно зіставити пов'язані записи |
| `LEFT JOIN` з `GROUP BY` | Середня | З'єднання та агрегування |
| Корельований підзапит | Середня/висока | Підзапит логічно залежить від зовнішнього рядка |
| Віконні функції | Середня/висока | Потрібне впорядкування рядків у групах |
| Матеріалізоване представлення | Висока | Агрегація кількох таблиць і окреме оновлення результату |
| Рекурсивний CTE | Висока | Повторне проходження ієрархії до умови зупинки |
| Функція з параметрами | Висока | Повторно використовувана параметризована аналітика |

Оцінка є якісною. Реальна продуктивність залежить від обсягу даних, індексів, статистики та плану PostgreSQL.

## 10. Контрольні запитання

1. **Різниця між INNER JOIN і LEFT JOIN.** `INNER JOIN` повертає лише рядки, для яких знайдено відповідність в обох таблицях. `LEFT JOIN` повертає всі рядки лівої таблиці, а для відсутніх відповідностей заповнює поля правої таблиці значеннями `NULL`. Для клієнтів без замовлень потрібен `LEFT JOIN`.
2. **Чому WHERE не фільтрує агрегати?** `WHERE` відбирає окремі рядки до групування, тоді як `HAVING` фільтрує вже сформовані групи й може містити умови на кшталт `HAVING SUM(...) > 0`.
3. **Корельований підзапит.** Він використовує значення рядка зовнішнього запиту. Наприклад, порівнює ціну товару із середньою ціною товарів тієї самої категорії, де підзапит має умову `p2.category_id = p.category_id`.
4. **Коли використовувати віконні функції замість GROUP BY?** Коли потрібно додати до кожного замовлення аналітичне значення, наприклад попередню дату через `LAG`, ранг або середнє по клієнту, не згортаючи всі рядки до одного на групу.
5. **Матеріалізоване представлення.** Це збережений результат запиту. Воно пришвидшує повторне читання складних агрегатів, але потребує оновлення після зміни вихідних даних.
6. **Оптимізація множинних JOIN.** Індексувати ключі з'єднання та поля фільтрації, фільтрувати нерелевантні рядки, не вибирати непотрібні стовпці, аналізувати план через `EXPLAIN ANALYZE` і перевіряти статистику.
7. **Різниця RANK і DENSE_RANK.** Якщо ціни за спаданням — 1000, 800, 800, 600, то `RANK()` дасть 1, 2, 2, 4, а `DENSE_RANK()` — 1, 2, 2, 3. `RANK()` пропускає ранг після однакових значень, а `DENSE_RANK()` — ні.

## 11. Висновок

Під час роботи підготовлено SQL-запити з `JOIN`, агрегатними функціями, `GROUP BY` і `HAVING`, підзапитами, віконними функціями та рекурсивним CTE. Розглянуто матеріалізовані представлення, параметризовану функцію, індекси й аналіз плану виконання запитів.

**Примітка щодо результатів:** SQL-код і пояснення підготовлені для виконання на навчальній базі TechnoMart. Час виконання, фактичні числа та скриншоти потрібно додати після запуску запитів у Supabase; вони не підмінялися вигаданими результатами.
