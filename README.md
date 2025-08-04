# 💊 Kimia Farma Big Data Analytics

This project aims to analyze sales transaction data of Kimia Farma using Google BigQuery, and visualize the insights through Looker Studio. The purpose is to uncover trends in sales performance, customer behavior, and branch activity to support strategic decision-making.

---

## 📁 Project Overview

- **Tools**: Google BigQuery, Looker Studio, CSV
- **Data Size**: >100MB (handled via Google Cloud)
- **Data Format**: CSV, BigQuery Tables
- **Objective**:
  - Identify top-performing products and outlets

  - Analyze monthly, daily, and yearly sales trends
  - Understand customer purchase behavior

---

## 🛠️ Tech Stack

- **Google BigQuery** – for SQL-based analytics
- **Looker Studio** – for visualization/dashboard
- **SQL** – for data querying and cleaning

---

## 📚 Background

Driven by a quest to gain deeper insights into Kimia Farma’s business performance, this project was initiated as part of the **Big Data Analytics program by Rakamin Academy in collaboration with Kimia Farma**. The goal is to analyze key business metrics from 2020 to 2023, providing valuable insights for decision-making.

The dataset is obtained directly from Kimia Farma’s internal records. Due to the **sensitive and confidential** nature of the data, the dataset will not be publicly shared. This analysis aims to generate a comprehensive dashboard in Google Looker Studio, allowing us to visualize Kimia Farma’s business trends effectively.

### Through SQL queries and data visualization, the project seeks to answer:

- 📈 How has Kimia Farma’s revenue changed from 2020 to 2023?  
- 🗺️ What is Indonesia’s geo map distribution of total profit by province?  
- 🏆 Which are the top 10 provinces with the highest total transactions?  
- 💰 Which are the top 10 provinces with the highest net sales?  
- ⭐ Which are the top 10 provinces with the highest ratings but the lowest transaction counts?

By developing this project, I aim to enhance my **data analytics skills**, particularly in **SQL, BigQuery, and Google Looker Studio**, while delivering valuable insights for **Kimia Farma’s business strategy**. 🧠

---

## 📁 Data Understanding

This project involves four primary datasets:

- `kf_final_transaction` 🧾: Contains transactional records including sales and revenue details.
- `kf_inventory` 📦: Details of product stock and inventory movement.
- `kf_kantor_cabang` 🏢: Contains information about branch locations.
- `kf_product` 🏷️: Includes product-related data such as category and pricing.

All datasets were uploaded to **Google BigQuery**, and a new table named `analyze` was created as a result of merging all four datasets.

---

## 🧪 SQL Query to Create `analyze` Table

```sql
CREATE OR REPLACE TABLE `kimia_farma.big_data` AS
SELECT 
  ft.transaction_id,                                -- ID transaksi
  ft.date,                                           -- Tanggal transaksi
  kc.branch_id,                                      -- ID cabang
  kc.branch_name,                                    -- Nama cabang
  kc.kota,                                           -- Kota cabang
  kc.provinsi,                                       -- Provinsi cabang
  kc.rating AS rating_cabang,                        -- Rating cabang
  
  ft.customer_name,                                  -- Nama customer
  ft.product_id,                                     -- ID produk
  p.product_name,                                    -- Nama produk
  ft.price AS actual_price,                          -- Harga produk
  ft.discount_percentage,                            -- Persentase diskon

  -- Persentase Gross Laba Berdasarkan Ketentuan Harga
  CASE 
    WHEN ft.price <= 50000 THEN 0.10
    WHEN ft.price <= 100000 THEN 0.15
    WHEN ft.price <= 300000 THEN 0.20
    WHEN ft.price <= 500000 THEN 0.25
    ELSE 0.30
  END AS persentase_gross_laba,

  -- Harga setelah diskon
  ft.price * (1 - ft.discount_percentage) AS nett_sales,

  -- Keuntungan yang diperoleh
  ft.price * (1 - ft.discount_percentage) *
    CASE 
      WHEN ft.price <= 50000 THEN 0.10
      WHEN ft.price <= 100000 THEN 0.15
      WHEN ft.price <= 300000 THEN 0.20
      WHEN ft.price <= 500000 THEN 0.25
      ELSE 0.30
    END AS nett_profit,

  ft.rating AS rating_transaksi                      -- Rating transaksi
FROM 
  `proven-octane-467802-m9.Rakamin_Final_Project.kf_final_transaction` ft
JOIN 
  `proven-octane-467802-m9.Rakamin_Final_Project.kf_kantor_cabang` kc 
  ON ft.branch_id = kc.branch_id
JOIN 
  `proven-octane-467802-m9.Rakamin_Final_Project.kf_product` p 
  ON ft.product_id = p.product_id;
```
---
## 📈 Analisis & Query SQL

### 1. Provinsi dengan Jumlah Transaksi Tertinggi
```sql
SELECT 
  kc.provinsi,
  COUNT(ft.transaction_id) AS total_transaksi
FROM 
  `Rakamin_Final_Project.kf_final_transaction` ft
JOIN 
  `Rakamin_Final_Project.kf_kantor_cabang` kc 
ON 
  ft.branch_id = kc.branch_id
GROUP BY 
  kc.provinsi
ORDER BY 
  total_transaksi DESC;
```
### 2. Korelasi Rating Cabang dan Jumlah Transaksi
```sql

SELECT 
  kc.branch_name,
  kc.rating AS rating_cabang,
  COUNT(ft.transaction_id) AS total_transaksi
FROM 
  `Rakamin_Final_Project.kf_final_transaction` ft
JOIN 
  `Rakamin_Final_Project.kf_kantor_cabang` kc 
ON 
  ft.branch_id = kc.branch_id
GROUP BY 
  kc.branch_name, kc.rating
ORDER BY 
  kc.rating DESC;
```
### 3. Kategori Produk Paling Menguntungkan
```sql

SELECT 
  p.product_category,
  SUM(ft.price * (1 - ft.discount_percentage)) AS total_penjualan,
  AVG(ft.rating) AS avg_rating
FROM 
  `Rakamin_Final_Project.kf_final_transaction` ft
JOIN 
  `Rakamin_Final_Project.kf_product` p 
ON 
  ft.product_id = p.product_id
GROUP BY 
  p.product_category
ORDER BY 
  total_penjualan DESC;
```
### 4. Cabang dengan Over-Stok (Stok Besar, Transaksi Rendah)
```sql
WITH stok AS (
  SELECT 
    i.branch_id,
    SUM(i.opname_stock) AS total_stok
  FROM 
    `Rakamin_Final_Project.kf_inventory` i
  GROUP BY 
    i.branch_id
),
transaksi AS (
  SELECT 
    ft.branch_id,
    COUNT(ft.transaction_id) AS total_transaksi
  FROM 
    `Rakamin_Final_Project.kf_final_transaction` ft
  GROUP BY 
    ft.branch_id
)
SELECT 
  kc.branch_name,
  s.total_stok,
  t.total_transaksi
FROM 
  stok s
JOIN 
  transaksi t ON s.branch_id = t.branch_id
JOIN 
  `Rakamin_Final_Project.kf_kantor_cabang` kc ON kc.branch_id = s.branch_id
ORDER BY 
  s.total_stok DESC, t.total_transaksi ASC;
```
### 5. Pelanggan Loyal Berdasarkan Frekuensi Transaksi
```sql

SELECT 
  customer_name,
  COUNT(transaction_id) AS jumlah_transaksi
FROM 
  `Rakamin_Final_Project.kf_final_transaction`
GROUP BY 
  customer_name
ORDER BY 
  jumlah_transaksi DESC
LIMIT 10;
```
### 6. Efektivitas Diskon terhadap Rating Transaksi
```sql
SELECT
  ROUND(discount_percentage, 1) AS discount_group,
  COUNT(transaction_id) AS jumlah_transaksi,
  AVG(rating) AS rata_rating,
  STDDEV(rating) AS deviasi_rating
FROM 
  `Rakamin_Final_Project.kf_final_transaction`
WHERE 
  rating IS NOT NULL
GROUP BY 
  discount_group
HAVING 
  COUNT(transaction_id) > 30
ORDER BY 
  discount_group;
```
### 7. Margin Keuntungan Bersih per Cabang
```sql
SELECT 
  kc.branch_name,
  kc.kota,
  kc.provinsi,
  COUNT(ft.transaction_id) AS total_transaksi,
  SUM(ft.price * (1 - ft.discount_percentage)) AS total_penjualan_setelah_diskon,
  SUM(ft.price * (1 - ft.discount_percentage) * 0.2) AS estimasi_nett_profit,
  SAFE_DIVIDE(
    SUM(ft.price * (1 - ft.discount_percentage) * 0.2),
    SUM(ft.price * (1 - ft.discount_percentage))
  ) AS margin_keuntungan_bersih
FROM 
  `Rakamin_Final_Project.kf_final_transaction` ft
JOIN 
  `Rakamin_Final_Project.kf_kantor_cabang` kc
ON 
  ft.branch_id = kc.branch_id
GROUP BY 
  kc.branch_name, kc.kota, kc.provinsi
ORDER BY 
  margin_keuntungan_bersih DESC
LIMIT 10;
```
### 8. Mismatch Rating vs Penjualan Cabang
```sql

SELECT
  kc.branch_id,
  kc.branch_name,
  kc.provinsi,
  kc.rating AS rating_cabang,
  COUNT(ft.transaction_id) AS jumlah_transaksi,
  SUM(ft.price * (1 - ft.discount_percentage)) AS total_penjualan,
  RANK() OVER (ORDER BY kc.rating DESC) AS rank_rating,
  RANK() OVER (ORDER BY SUM(ft.price * (1 - ft.discount_percentage)) DESC) AS rank_penjualan,
  ABS(
    RANK() OVER (ORDER BY kc.rating DESC) - 
    RANK() OVER (ORDER BY SUM(ft.price * (1 - ft.discount_percentage)) DESC)
  ) AS gap_rank
FROM 
  `Rakamin_Final_Project.kf_kantor_cabang` kc
LEFT JOIN 
  `Rakamin_Final_Project.kf_final_transaction` ft
ON 
  kc.branch_id = ft.branch_id
GROUP BY 
  kc.branch_id, kc.branch_name, kc.provinsi, kc.rating
ORDER BY 
  gap_rank DESC
LIMIT 10;
```
---
## 📊 Dashboard Interaktif
🔗 Link ke Dashboard Looker Studio – Kimia Farma Final Project 
https://lookerstudio.google.com/reporting/16292f61-2110-4352-a7bb-3b4142b23d95 

<img width="728" height="481" alt="image" src="https://github.com/user-attachments/assets/b244aa53-c9f0-4a7e-8d40-9172e044149b" />
<img width="651" height="488" alt="image" src="https://github.com/user-attachments/assets/6f1e8613-438c-41c8-875e-96574282fd7e" />





---
## ✨ Author
Muhammad Rayhan Aidy Abshar
Rakamin x Kimia Farma Big Data Analytics Program
📅 August 2025


