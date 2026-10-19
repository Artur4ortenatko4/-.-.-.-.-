# Звіт до лабораторної роботи № 1
## Робота з СУБД PostgreSQL та основи SQL

**Виконав:** Юрдамян Артур Торосович  
**Група:** ІПЗ-33  
**СУБД:** PostgreSQL (Supabase)

## 1. Мета роботи

Ознайомитися з реляційними базами даних, підключенням до хмарної СУБД PostgreSQL та виконанням SQL-запитів для вибірки, фільтрації, сортування й аналізу даних.

## 2. Хід роботи

Для виконання запитів використовується навчальна база даних TechnoMart з таблицями `customers`, `products`, `orders`, `employees` та іншими. Запити нижче написані для PostgreSQL. Їх потрібно виконати в SQL Editor підключеного проєкту Supabase. Фактичні результати залежать від вмісту бази даних.

### Рівень 1. Основні SELECT-запити

```sql
-- 1. Усі записи клієнтів
SELECT *
FROM customers;

-- 2. Назви та ціни товарів
SELECT product_name, unit_price
FROM products;

-- 3. Контактні дані співробітників
SELECT first_name, last_name, phone, email
FROM employees;
```

### Фільтрація за допомогою WHERE

```sql
-- Клієнти з Києва
SELECT *
FROM customers
WHERE city = 'Київ';

-- Товари дорожчі за 25 000 грн
SELECT product_name, unit_price
FROM products
WHERE unit_price > 25000;

-- Доставлені замовлення
SELECT *
FROM orders
WHERE order_status = 'delivered';

-- Співробітники, у назві посади яких є слово «продаж»
SELECT first_name, last_name, title
FROM employees
WHERE title ILIKE '%продаж%';
```

### Сортування та обмеження результатів

```sql
-- Товари від дешевших до дорожчих
SELECT product_name, unit_price
FROM products
ORDER BY unit_price ASC;

-- Клієнти за іменем контактної особи
SELECT contact_name, city
FROM customers
ORDER BY contact_name ASC;

-- Замовлення від найновіших до найстаріших
SELECT order_id, order_date, order_status
FROM orders
ORDER BY order_date DESC;

-- 10 найдорожчих товарів
SELECT product_name, unit_price
FROM products
ORDER BY unit_price DESC
LIMIT 10;

-- 5 останніх замовлень
SELECT order_id, order_date, order_status
FROM orders
ORDER BY order_date DESC
LIMIT 5;

-- Перші 8 клієнтів за алфавітом
SELECT contact_name, city
FROM customers
ORDER BY contact_name ASC
LIMIT 8;
```

### Рівень 2. Пошук за зразком LIKE

```sql
-- Імена, які починаються на «Іван»
SELECT *
FROM customers
WHERE contact_name ILIKE 'Іван%';

-- Назви товарів, що містять phone або телефон
SELECT product_name
FROM products
WHERE product_name ILIKE '%phone%'
   OR product_name ILIKE '%телефон%';

-- Власний запит: назви товарів, що починаються на Samsung
SELECT product_name
FROM products
WHERE product_name ILIKE 'Samsung%';

-- Власний запит: email-адреси Gmail
SELECT contact_name, email
FROM customers
WHERE email ILIKE '%@gmail.com';

-- Власний запит: імена, що закінчуються на «енко»
SELECT contact_name
FROM customers
WHERE contact_name ILIKE '%енко';
```

### Логічні оператори AND, OR, NOT

```sql
-- Товари дорожчі за 15 000 та дешевші за 50 000 грн
SELECT product_name, unit_price
FROM products
WHERE unit_price > 15000
  AND unit_price < 50000;

-- Юридичні особи з Києва або Львова
SELECT contact_name, city, customer_type
FROM customers
WHERE city IN ('Київ', 'Львів')
  AND customer_type = 'company';

-- Власний запит 1: активні товари в наявності
SELECT product_name, unit_price, units_in_stock
FROM products
WHERE NOT discontinued
  AND units_in_stock > 0;

-- Власний запит 2: товари дешевші за 5 000 або дорожчі за 50 000 грн
SELECT product_name, unit_price
FROM products
WHERE unit_price < 5000 OR unit_price > 50000;

-- Власний запит 3: клієнти з Києва, які мають email
SELECT contact_name, city, email
FROM customers
WHERE city = 'Київ'
  AND email IS NOT NULL;

-- Власний запит 4: замовлення, які ще не доставлено
SELECT order_id, order_date, order_status
FROM orders
WHERE NOT (order_status = 'delivered');
```

### Оператори IN, BETWEEN, IS NULL та IS NOT NULL

```sql
-- Клієнти з визначених міст
SELECT contact_name, city
FROM customers
WHERE city IN ('Київ', 'Харків', 'Одеса', 'Дніпро');

-- Товари в діапазоні 10 000–30 000 грн
SELECT product_name, unit_price
FROM products
WHERE unit_price BETWEEN 10000 AND 30000;

-- Власний IN 1: товари з категорій 1, 2 або 3
SELECT product_name, category_id
FROM products
WHERE category_id IN (1, 2, 3);

-- Власний IN 2: замовлення зі статусами delivered або processing
SELECT order_id, order_status
FROM orders
WHERE order_status IN ('delivered', 'processing');

-- Власний BETWEEN 1: замовлення за 2024 рік
SELECT order_id, order_date
FROM orders
WHERE order_date >= DATE '2024-01-01'
  AND order_date < DATE '2025-01-01';

-- Власний BETWEEN 2: товари за ціною від 5 000 до 15 000 грн
SELECT product_name, unit_price
FROM products
WHERE unit_price BETWEEN 5000 AND 15000;

-- Клієнти без назви компанії
SELECT contact_name, company_name
FROM customers
WHERE company_name IS NULL;

-- Клієнти, для яких вказано компанію
SELECT contact_name, company_name
FROM customers
WHERE company_name IS NOT NULL;

-- Товари без опису
SELECT product_name
FROM products
WHERE description IS NULL;

-- Замовлення без дати відправлення
SELECT order_id, order_date
FROM orders
WHERE shipped_date IS NULL;
```

### Комбіновані умови

```sql
-- 1. Активні товари в ціні 10 000–50 000 грн, крім чохлів
SELECT product_name, unit_price
FROM products
WHERE unit_price BETWEEN 10000 AND 50000
  AND NOT discontinued
  AND product_name NOT ILIKE '%чохол%';

-- 2. Товари Apple або Samsung дорожчі за 15 000 грн
SELECT product_name, unit_price
FROM products
WHERE (product_name ILIKE '%Apple%' OR product_name ILIKE '%Samsung%')
  AND unit_price > 15000;

-- 3. Клієнти з великих міст, які мають email
SELECT contact_name, city, email
FROM customers
WHERE city IN ('Київ', 'Харків', 'Одеса', 'Дніпро')
  AND email IS NOT NULL;

-- 4. Товари категорій 1 або 2 у наявності
SELECT product_name, category_id, units_in_stock
FROM products
WHERE category_id IN (1, 2)
  AND units_in_stock > 0;

-- 5. Замовлення, створені у 2024 році, які не скасовані
SELECT order_id, order_date, order_status
FROM orders
WHERE order_date >= DATE '2024-01-01'
  AND order_date < DATE '2025-01-01'
  AND order_status <> 'cancelled';
```

### Складне сортування та пагінація

```sql
-- Сортування 1: категорія за зростанням, ціна за спаданням
SELECT product_name, category_id, unit_price
FROM products
ORDER BY category_id ASC, unit_price DESC;

-- Сортування 2: спочатку тип клієнта, потім місто та ім'я
SELECT contact_name, customer_type, city
FROM customers
ORDER BY customer_type, city, contact_name;

-- Сортування 3: статус замовлення та дата
SELECT order_id, order_status, order_date
FROM orders
ORDER BY order_status ASC, order_date DESC;

-- Пагінація: перша сторінка по 10 товарів
SELECT product_name, unit_price
FROM products
ORDER BY product_name
LIMIT 10 OFFSET 0;

-- Пагінація: друга сторінка по 10 товарів
SELECT product_name, unit_price
FROM products
ORDER BY product_name
LIMIT 10 OFFSET 10;
```

### Рівень 3. Складні умови та аналітика

```sql
-- Samsung або Apple, але не чохли
SELECT product_name, unit_price
FROM products
WHERE (product_name ILIKE '%Samsung%' OR product_name ILIKE '%Apple%')
  AND product_name NOT ILIKE '%чохол%';

-- Товари дорожчі за 20 000 грн у категоріях 1 або 2,
-- або товари дешевші за 5 000 грн у будь-якій категорії
SELECT product_name, category_id, unit_price
FROM products
WHERE (unit_price > 20000 AND category_id IN (1, 2))
   OR unit_price < 5000;

-- Складний звіт товарів із кількома критеріями
SELECT product_name, category_id, unit_price, units_in_stock
FROM products
WHERE unit_price BETWEEN 5000 AND 50000
  AND category_id IN (1, 2, 3)
  AND units_in_stock > 0
  AND NOT discontinued
  AND product_name NOT ILIKE '%чохол%'
  AND product_name NOT ILIKE '%кабель%'
ORDER BY category_id, unit_price DESC;

-- Аналіз клієнтської бази
SELECT contact_name, city, customer_type, email
FROM customers
WHERE city IN ('Київ', 'Львів', 'Харків', 'Одеса')
  AND email IS NOT NULL
  AND contact_name NOT ILIKE '%тест%'
  AND (customer_type = 'company' OR city IN ('Київ', 'Львів'))
ORDER BY customer_type DESC, city, contact_name;

-- Аналіз цінових сегментів
SELECT
    CASE
        WHEN unit_price < 5000 THEN 'Бюджетний'
        WHEN unit_price < 20000 THEN 'Середній'
        WHEN unit_price < 50000 THEN 'Преміум'
        ELSE 'Люкс'
    END AS price_segment,
    COUNT(*) AS products_count,
    ROUND(AVG(unit_price), 2) AS average_price,
    MIN(unit_price) AS min_price,
    MAX(unit_price) AS max_price
FROM products
GROUP BY price_segment
ORDER BY MIN(unit_price);

-- Географічний розподіл клієнтів
SELECT city, COUNT(*) AS customers_count
FROM customers
GROUP BY city
ORDER BY customers_count DESC, city;

SELECT region_id, COUNT(*) AS customers_count
FROM customers
GROUP BY region_id
ORDER BY customers_count DESC;

SELECT customer_type, city, COUNT(*) AS customers_count
FROM customers
GROUP BY customer_type, city
ORDER BY customer_type, customers_count DESC;

SELECT city, COUNT(*) AS customers_with_email
FROM customers
WHERE email IS NOT NULL
GROUP BY city
ORDER BY customers_with_email DESC;

-- Часові закономірності замовлень
SELECT EXTRACT(YEAR FROM order_date) AS order_year,
       COUNT(*) AS orders_count
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date)
ORDER BY order_year;

SELECT EXTRACT(MONTH FROM order_date) AS order_month,
       COUNT(*) AS orders_count
FROM orders
WHERE order_date >= DATE '2024-01-01'
  AND order_date < DATE '2025-01-01'
GROUP BY EXTRACT(MONTH FROM order_date)
ORDER BY order_month;

SELECT order_status, COUNT(*) AS orders_count,
       MIN(order_date) AS first_order,
       MAX(order_date) AS last_order
FROM orders
GROUP BY order_status
ORDER BY orders_count DESC;

-- Креативні запити
-- 1. Найдорожчі товари в кожній категорії
SELECT product_name, category_id, unit_price
FROM (
    SELECT p.*,
           ROW_NUMBER() OVER (
               PARTITION BY category_id
               ORDER BY unit_price DESC, product_name
           ) AS rn
    FROM products p
) ranked
WHERE rn = 1;

-- 2. Клієнти, які ще не мають замовлень
SELECT c.customer_id, c.contact_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- 3. Кількість товарів у кожній категорії, включно з порожніми категоріями
SELECT c.category_name, COUNT(p.product_id) AS products_count
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY products_count DESC;

-- 4. Середня ціна товарів порівняно із середньою ціною всієї бази
SELECT product_name, unit_price,
       (SELECT AVG(unit_price) FROM products) AS overall_avg
FROM products
WHERE unit_price > (SELECT AVG(unit_price) FROM products)
ORDER BY unit_price DESC;

-- 5. Частка товару в загальній кількості товарних позицій
SELECT product_name, units_in_stock,
       ROUND(
           100.0 * units_in_stock / NULLIF(SUM(units_in_stock) OVER (), 0),
           2
       ) AS stock_percent
FROM products
ORDER BY stock_percent DESC NULLS LAST;
```

## 3. Пояснення бізнес-логіки

- `WHERE` відбирає рядки за заданими умовами до сортування та обмеження результатів.
- `LIKE` і `ILIKE` знаходять текст за шаблоном; `ILIKE` у PostgreSQL не враховує регістр.
- `AND`, `OR` та дужки дозволяють точно визначити поєднання критеріїв.
- `IN` перевіряє належність до списку, `BETWEEN` — включний діапазон, `IS NULL` — відсутнє значення.
- `ORDER BY` забезпечує визначений порядок, а `LIMIT` та `OFFSET` використовуються для сторінкового виведення.
- `GROUP BY` та агрегатні функції допомагають аналізувати асортимент, клієнтів і замовлення.

## 4. Контрольні запитання

1. **Що таке SQL?** SQL — декларативна мова запитів до реляційних баз даних. У SQL описують, які дані потрібно отримати, а не покроковий алгоритм їх пошуку. Імперативна мова натомість задає послідовність дій.
2. **Порядок виконання SELECT.** Концептуально спочатку обробляються `FROM`/`JOIN`, потім `WHERE`, `GROUP BY`, `HAVING`, `SELECT`, `ORDER BY` і `LIMIT`/`OFFSET`. Розуміння порядку допомагає правильно розміщувати фільтри та агрегати.
3. **Різниця між `=` та `LIKE`.** `=` порівнює значення повністю, наприклад `city = 'Київ'`. `LIKE` шукає за шаблоном, наприклад `contact_name LIKE 'Іван%'`.
4. **Шаблони `%` та `_`.** `%` відповідає будь-якій кількості символів, а `_` — рівно одному символу. Їх можна комбінувати, наприклад `LIKE 'Ів_н%'`.
5. **Чому `= NULL` не працює?** `NULL` позначає невідоме або відсутнє значення, тому звичайне порівняння не повертає true. Використовують `IS NULL` та `IS NOT NULL`.
6. **AND та OR.** `AND` вимагає істинності всіх умов, `OR` — хоча б однієї. `AND` має вищий пріоритет: `A OR B AND C` читається як `A OR (B AND C)`. Дужки роблять логіку явною.
7. **BETWEEN.** Оператор включає обидві межі діапазону. Для виключення меж потрібно писати `value > lower_bound AND value < upper_bound`.
8. **LIMIT та OFFSET.** `LIMIT` задає кількість рядків, а `OFFSET` — скільки рядків пропустити. Формула: `OFFSET = (номер_сторінки - 1) * розмір_сторінки`.
9. **ORDER BY разом із LIMIT.** Без `ORDER BY` порядок рядків не гарантований, тому набір записів у результаті `LIMIT` може змінюватися.
10. **LIKE та ILIKE.** `LIKE` виконує шаблонний пошук з урахуванням регістру залежно від правил порівняння; `ILIKE` у PostgreSQL виконує пошук без урахування регістру.
11. **NULL при сортуванні.** У PostgreSQL за замовчуванням `NULL` розташовуються наприкінці при `ASC` і на початку при `DESC`. Це можна змінити через `NULLS FIRST` або `NULLS LAST`.
12. **Приклад складної умови.** Для вибору товарів категорій 1 або 2, які дорожчі за 20 000 грн, потрібно написати `(category_id IN (1, 2)) AND unit_price > 20000`. Без дужок у складніших виразах `AND` може мати вищий пріоритет за `OR` і включити зайві записи.

## 5. Висновок

Під час лабораторної роботи розглянуто основні засоби SQL у PostgreSQL: вибірку стовпців, фільтрацію, пошук за шаблоном, логічні оператори, роботу з `NULL`, сортування, обмеження та пагінацію. Також підготовлено запити для сегментації товарів і аналізу клієнтів та замовлень.

**Примітка щодо результатів:** запити підготовлені для виконання в навчальній базі TechnoMart. Фактичні таблиці результатів і скриншоти потрібно додати після запуску запитів у Supabase; результати виконання тут не вигадувалися.
