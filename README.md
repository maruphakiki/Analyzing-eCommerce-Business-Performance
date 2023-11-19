## 📂 **STAGE 1: Problem Statement**

### **Background Story**
Perusahaan ini merupakan salah satu marketplace terbesar di Amerika Selatan yang menghubungkan pelaku bisnis mikro dengan para pelanggannya. Sebagai anggota dari tim Data Analytics, saya mendapatkan tanggung jawab untuk menganalisis tiga aspek yang berkaitan dengan performa bisnis perusahaan tersebut. Ketiga aspek tersebut di antaranya adalah **pertumbuhan pelanggan**, **kualitas produk** dan **tipe pembayaran**. saya akan mengolah data yang sudah disediakan untuk membuat laporan performa bisnis pada ketiga aspek tersebut.

### **Objective**
Mengumpulkan insight dari analisis dan dengan visualisasi berupa :
1. **Annual Customer Activity Growth**
2. **Annual Product Category Quality**
3. **Annual Payment Type Usage**

---


## 📂 **STAGE 2: Data Preparation**

Dataset yang digunakan adalah dataset sebuah perusahaan eCommerce Brasil yang memiliki informasi pesanan dengan jumlah 99441 dari tahun 2016 hingga 2018. Terdapat fitur-titur yang membuat informasi seperti status pemesanan, lokasi, rincian item, jenis pembayaran, serta ulasan.

### **Create Database and ERD**
**Langkah-langkah yang dilakukan meliputi:**
1. Membuat workspace database di dalam pgAdmin dan membuat tabel menggunakan `CREATE TABLE` statement
2. Melakukan import data csv kedalam database
3. Menentukan Primary Key atau Foreign Key enggunakan statement `ALTER TABLE`
4. Membuat dan mengeksport ERD (Entity Relationship Diagram) <br>

<details>
  <summary>Click untuk melihat Queries</summary>
  
  ```sql
-- 1) Membuat database melalui klik kanan Databases > Create > Database.. dengan nama ecommerce_miniproject


-- 2) Membuat tabel dan menentukan foreign key menggunakan statement CREATE TABLE dengan mengikuti penamaan kolom di csv dan memastikan tipe datanya sesuai.

CREATE TABLE customers_dataset (
	customer_id varchar primary key,
	customer_unique_id varchar,
	customer_zip_code_prefix varchar,
	customer_city varchar,
	customer_state varchar
);

CREATE TABLE sellers_dataset (
	seller_id varchar primary key,
	seller_zip_code_prefix varchar,
	seller_city varchar,
	seller_state varchar
);

CREATE TABLE geolocation_dataset (
	geolocation_zip_code_prefix varchar primary key,
	geolocation_lat decimal,
	geolocation_lng decimal,
	geolocation_city varchar,
	geolocation_state varchar
);

CREATE TABLE product_dataset (
	product_id varchar primary key,
	product_category_name varchar,
	product_name_lenght int,
	product_description_lenght int,
	product_photos_qty int,
	product_weight_g decimal,
	product_length_cm decimal,
	product_height_cm decimal,
	product_width_cm decimal
);

CREATE TABLE orders_dataset (
	order_id varchar primary key,
	customer_id varchar,
	order_status varchar,
	order_purchase_timestamp timestamp,
	order_approved_at timestamp,
	order_delivered_carrier_date timestamp,
	order_delivered_customer_date timestamp,
	order_estimated_delivery_date timestamp
);

CREATE TABLE order_items_dataset (
	order_id varchar,
	order_item_id int primary key,
	product_id varchar,
	seller_id varchar,
	shipping_limit_date timestamp,
	price decimal,
	fright_value decimal
);

CREATE TABLE order_payments_dataset (
	order_id varchar,
	payment_sequential int,
	payment_type varchar,
	payment_installments int,
	payment_value decimal
);

CREATE TABLE order_reviews_dataset (
	review_id varchar primary key,
	order_id varchar,
	review_score int,
	review_comment_title varchar,
	review_comment_message varchar,
	review_creation_date timestamp,
	review_answer_timestamp timestamp
);

-- 3) Mengimpor file csv ke dalam masing-masing tabel yang telah dibuat dengan klik kanan pada nama tabel > Import/Export Data.

-- 4). Foreign Key

ALTER TABLE orders_dataset ADD FOREIGN KEY (customer_id) REFERENCES customers_dataset;
ALTER TABLE order_payments_dataset ADD FOREIGN KEY (order_id) REFERENCES orders_dataset;
ALTER TABLE order_reviews_dataset ADD FOREIGN KEY (order_id) REFERENCES orders_dataset;
ALTER TABLE order_items_dataset ADD FOREIGN KEY (order_id) REFERENCES orders_dataset;
ALTER TABLE order_items_dataset ADD FOREIGN KEY (product_id) REFERENCES product_dataset;
ALTER TABLE order_items_dataset ADD FOREIGN KEY (seller_id) REFERENCES sellers_dataset;

-- 5) Membuat ERD dengan cara klik kanan pada database ecommerce_miniproject > Gererate ERD.

  ```
</details>

**Hasil ERD :** <br>
<p align="center">
  <kbd><img src="Assets/fix ga,bar.jpg" width=800px> </kbd> <br>
  <strong> Gambar 1. Entity Relationship Diagram </strong>
</p>
<br>
<br>

---
## 📂 **STAGE 3: Data Analysis**

### **1. Annual Customer Activity Growth**
Pertumbuhan aktivitas pelanggan tahunan dapat dianalisis dari Monthly active user (MAU), pelanggan baru, pelanggan dengan repeat order, dan rata-rata order oleh pelanggan.

<details>
  <summary>Click untuk melihat Queries</summary>
  
  ```sql
 --1 Menampilkan rata-rata jumlah customer aktif bulanan (monthly active user) untuk setiap tahun
SELECT year, FLOOR(AVG(customer_total)) AS avg_mau
FROM (
	SELECT 
		date_part('year', od.order_purchase_timestamp) AS year,
		date_part('month', od.order_purchase_timestamp) AS month,
		COUNT(DISTINCT cd.customer_unique_id) AS customer_total
	FROM orders_dataset AS od
	JOIN customers_dataset AS cd
		ON cd.customer_id = od.customer_id
	GROUP BY 1, 2
	) AS sub
GROUP BY 1
ORDER BY 1
;

--2 Menampilkan jumlah customer baru pada masing-masing tahun
SELECT year, COUNT(customer_unique_id) AS total_new_customer
FROM (
	SELECT
		Min(date_part('year', od.order_purchase_timestamp)) AS year,
		cd.customer_unique_id
	FROM orders_dataset AS od
	JOIN customers_dataset AS cd
		ON cd.customer_id = od.customer_id
	GROUP BY 2
	) AS sub
GROUP BY 1
ORDER BY 1
;

--3 Menampilkan jumlah customer repeat order pada masing-masing tahun
SELECT year, count(customer_unique_id) AS total_customer_repeat
FROM (
	SELECT
		date_part('year', od.order_purchase_timestamp) AS year,
		cd.customer_unique_id,
		COUNT(od.order_id) AS total_order
	FROM orders_dataset AS od
	JOIN customers_dataset AS cd
		ON cd.customer_id = od.customer_id
	GROUP BY 1, 2
	HAVING count(2) > 1
	) AS sub
GROUP BY 1
ORDER BY 1
;

--4 Menampilkan rata-rata jumlah order yang dilakukan customer untuk masing-masing tahun
SELECT year, ROUND(AVG(freq), 3) AS avg_frequency
FROM (
	SELECT
		date_part('year', od.order_purchase_timestamp) AS year,
		cd.customer_unique_id,
		COUNT(order_id) AS freq
	FROM orders_dataset AS od
	JOIN customers_dataset AS cd
		ON cd.customer_id = od.customer_id
	GROUP BY 1, 2
	) AS sub
GROUP BY 1
ORDER BY 1
;

--5 Menggabungkan ke-empat metrik yang telah berhasil ditampilkan menjadi satu tampilan tabel
WITH cte_mau AS (
	SELECT year, FLOOR(AVG(customer_total)) AS avg_mau
	FROM (
		SELECT 
			date_part('year', od.order_purchase_timestamp) AS year,
			date_part('month', od.order_purchase_timestamp) AS month,
			COUNT(DISTINCT cd.customer_unique_id) AS customer_total
		FROM orders_dataset AS od
		JOIN customers_dataset AS cd
			ON cd.customer_id = od.customer_id
		GROUP BY 1, 2
		) AS sub
	GROUP BY 1
),

cte_new_cust AS (
	SELECT year, COUNT(customer_unique_id) AS total_new_customer
	FROM (
		SELECT
			Min(date_part('year', od.order_purchase_timestamp)) AS year,
			cd.customer_unique_id
		FROM orders_dataset AS od
		JOIN customers_dataset AS cd
			ON cd.customer_id = od.customer_id
		GROUP BY 2
		) AS sub
	GROUP BY 1
),

cte_repeat_order AS (
	SELECT year, count(customer_unique_id) AS total_customer_repeat
	FROM (
		SELECT
			date_part('year', od.order_purchase_timestamp) AS year,
			cd.customer_unique_id,
			COUNT(od.order_id) AS total_order
		FROM orders_dataset AS od
		JOIN customers_dataset AS cd
			ON cd.customer_id = od.customer_id
		GROUP BY 1, 2
		HAVING count(2) > 1
		) AS sub
	GROUP BY 1
),

cte_frequency AS (
	SELECT year, ROUND(AVG(freq), 3) AS avg_frequency
	FROM (
		SELECT
			date_part('year', od.order_purchase_timestamp) AS year,
			cd.customer_unique_id,
			COUNT(order_id) AS freq
		FROM orders_dataset AS od
		JOIN customers_dataset AS cd
			ON cd.customer_id = od.customer_id
		GROUP BY 1, 2
		) AS sub
	GROUP BY 1
)

SELECT
	mau.year AS year,
	avg_mau,
	total_new_customer,
	total_customer_repeat,
	avg_frequency
FROM
	cte_mau AS mau
	JOIN cte_new_cust AS nc
		ON mau.year = nc.year
	JOIN cte_repeat_order AS ro
		ON nc.year = ro.year
	JOIN cte_frequency AS f
		ON ro.year = f.year
GROUP BY 1, 2, 3, 4, 5
ORDER BY 1
;
  ```
</details>

<p align="center">
  <kbd><img src="Assets/1.png" width=800px> </kbd> <br>
  <strong> Tabel 1. Hasil Analisis Pertumbuhan Aktivitas Pelanggan Tahunan </strong>
</p>
Tabel tersebut adalah hasil dari penggabungan 4 metrik, berikut keterangan dari kolom
pada table tersebut: <br>
- year : Tahun <br>
- total_revenue : Jumlah Total Revenue pertahun <br>
- top_product : Produk yang memiliki penghasilan terbanyak tiap tahun <br>
- total_revenue_top_product : Jumlah total revenue top produk tiap tahun <br>
- total_canceled : Jumlah total cancel order pertahun
- top_canceled_product : Produk yang memiliki jumlah cancel order terbanyak tiap 
tahun <br>
- total_top_canceled_product : jumlah total cancel order top produk tiap tahun <br>
<br>
<br>

<p align="center">
  <kbd><img src="Assets/avg mau & new customer.png" width=600px> </kbd> <br>
  <strong> Gambar 2. Grafik Rata-rata MAU dan Pelanggan Baru </strong>
</p>

Secara keseluruhan perusahaan mengalami peningkakatan Monthly Active User serta pelanggan baru setiap tahunnya. Peningkatan yang signifikan terjadi pada tahun 2016 ke 2017, hal ini dikarenakan data transaksi pada tahun 2016 dimulai pada bulan September. <br>
<br>
<br>

<p align="center">
  <kbd><img src="Assets/customer repeat.png" width=600px> </kbd> <br>
  <strong> Gambar 3. Grafik Jumlah Pelanggan yang Melakukan Repeat Order </strong>
</p>

Peningkatan yang signifikan juga terjadi pada jumlah pelanggan yang melakukan repeat order pada tahun 2016 hingga 2017. Namun pada tahun 2018 mengalami sedikit penurunan. <br>
<br>
<br>


<p align="center">
  <kbd><img src="Assets/avgcustomer.png" width=600px> </kbd> <br>
  <strong> Gambar 4. Grafik Rata-rata Frekuensi Order Pelanggan </strong>
</p>

Dari analisis dan grafik diatas dapat diketahui bahwa rata-rata pelanggan setiap tahunnya cenderung hanya melakukan order satu kali, artinya mayoritas pelanggan tidak melakukan repeat order.<br>
<br>

### **2. Annual Product Category Quality**

Kualitas kategori produk tahunan dapat dianalisis dari total pendapatan, total pembatalan pesanan, kategori top produk dan kategori produk yang paling banyak dibatalkan.

<details>
  <summary>Click disini untuk melihat Queries</summary>

  ```sql
 --1) Membuat tabel yang berisi informasi pendapatan/revenue perusahaan total untuk masing-masing tahun
CREATE TABLE total_revenue AS
	SELECT
		date_part('year', od.order_purchase_timestamp) AS year,
		SUM(oid.price + oid.fright_value) AS revenue
	FROM order_items_dataset AS oid
	JOIN orders_dataset AS od
		ON oid.order_id = od.order_id
	WHERE od.order_status like 'delivered'
	GROUP BY 1
	ORDER BY 1;

--2) Membuat tabel yang berisi informasi jumlah cancel order total untuk masing-masing tahun
CREATE TABLE canceled_order AS
	SELECT
		date_part('year', order_purchase_timestamp) AS year,
		COUNT(order_status) AS canceled
	FROM orders_dataset
	WHERE order_status like 'canceled'
	GROUP BY 1
	ORDER BY 1;
		
--3) Membuat tabel yang berisi nama kategori produk yang memberikan pendapatan total tertinggi untuk masing-masing tahun
CREATE TABLE top_product_category AS
	SELECT 
		year,
		top_category,
		product_revenue
	FROM (
		SELECT
			date_part('year', shipping_limit_date) AS year,
			pd.product_category_name AS top_category,
			SUM(oid.price + oid.fright_value) AS product_revenue,
			RANK() OVER (PARTITION BY date_part('year', shipping_limit_date)
					 ORDER BY SUM(oid.price + oid.fright_value) DESC) AS ranking
		FROM orders_dataset AS od 
		JOIN order_items_dataset AS oid
			ON od.order_id = oid.order_id
		JOIN product_dataset AS pd
			ON oid.product_id = pd.product_id
		WHERE od.order_status like 'delivered'
		GROUP BY 1, 2
		ORDER BY 1
		) AS sub
	WHERE ranking = 1;
	
--4) Membuat tabel yang berisi nama kategori produk yang memiliki jumlah cancel order terbanyak untuk masing-masing tahun
CREATE TABLE most_canceled_category AS
	SELECT 
		year,
		most_canceled,
		total_canceled
	FROM (
		SELECT
			date_part('year', shipping_limit_date) AS year,
			pd.product_category_name AS most_canceled,
			COUNT(od.order_id) AS total_canceled,
			RANK() OVER (PARTITION BY date_part('year', shipping_limit_date)
					 ORDER BY COUNT(od.order_id) DESC) AS ranking
		FROM orders_dataset AS od 
		JOIN order_items_dataset AS oid
			ON od.order_id = oid.order_id
		JOIN product_dataset AS pd
			ON oid.product_id = pd.product_id
		WHERE od.order_status like 'canceled'
		GROUP BY 1, 2
		ORDER BY 1
		) AS sub
	WHERE ranking = 1;
	
-- Tambahan - Menghapus anomali data tahun
DELETE FROM top_product_category WHERE year = 2020;
DELETE FROM most_canceled_category WHERE year = 2020;

-- Menampilkan tabel yang dibutuhkan
SELECT 
	tr.year,
	tr.revenue AS total_revenue,
	tpc.top_category AS top_product,
	tpc.product_revenue AS total_revenue_top_product,
	co.canceled total_canceled,
	mcc.most_canceled top_canceled_product,
	mcc.total_canceled total_top_canceled_product
FROM total_revenue AS tr
JOIN top_product_category AS tpc
	ON tr.year = tpc.year
JOIN canceled_order AS co
	ON tpc.year = co.year
JOIN most_canceled_category AS mcc
	ON co.year = mcc.year
GROUP BY 1, 2, 3, 4, 5, 6, 7;
  ```
</details>

<p align="center">
  <kbd><img src="Assets/3.png" width=1000px> </kbd> <br>
  <strong> Tabel 2. Hasil Analisis Total Kategori Produk Tahunan </strong>
</p>
Berikut keterangan dari kolom pada table tersebut: <br>
- year : Tahun <br>
- total_revenue : Jumlah Total Revenue pertahun <br>
- top_product : Produk yang memiliki penghasilan terbanyak tiap tahun <br>
- total_revenue_top_product : Jumlah total revenue top produk tiap tahun <br>
- total_canceled : Jumlah total cancel order pertahun <br>
- top_canceled_product : Produk yang memiliki jumlah cancel order terbanyak tiap
tahun <br>
- total_top_canceled_product : jumlah total cancel order top produk tiap tahun <br>

<br>
<p align="center">
  <kbd><img src="Assets/total revenue.png" width=600px> </kbd> <br>
  <strong> Gambar 5. Grafik Total Revenue Pertahun </strong>
</p>

Secara keseluruhan total pendapatan/revenue perusahaan meningkat tiap tahun (terjadi peningkatan signifikan dari tahun 2016 – 2017).<br>
<br>
<br>

<p align="center">
  <kbd><img src="Assets/total revenue top produk.png" width=600px> </kbd> <br>
  <strong> Gambar 6. Grafik Total Revenue Top Produk Pertahun </strong>
</p>

Revenue yang dihasilkan dari top produk juga meningkat untuk setiap tahunnya. Selain itu setiap tahunnya memiliki jenis kategori top produk yang berbeda. Pada tahun 2018, perusahaan menghasilkan revenue paling tinggi dengan jenis karegori top produk kesehatan dan kecantikan (`health_beauty`). <br>
<br>
<br>

<p align="center">
  <kbd><img src="Assets/top produk canceled.png" width=600px> </kbd> <br>
  <strong> Gambar 7. Grafik Total Canceled Top Produk Pertahun  </strong>
</p>

Produk yang sering dibatalkan oleh pelanggan untuk setiap tahunnya juga memiliki jenis kategori yang berbeda dan terus mengalami kenaikan. Tahun 2018 memiliki jumlah produk yang dibatalkan paling banyak dan memiliki jenis kategori yang sama dengan top produk yang paling banyak menghasilkan revenue. Hal tersebut dapat diduga karena jenis kategori kesehatan dan kecantikan sedang mendominasi pasar.  <br>
<br>
### **3. Annual Payment Type Usage**
Tipe pembayaran yang digunakan pelanggan dapat dianalisis dari jenis pembayaran favorit dan jumlah penggunaan untuk setiap jenis pembayaran pertahun. 

<details>
  <summary>Click untuk melihat Queries</summary>

  ```sql
-- 1) Menampilkan jumlah penggunaan masing-masing tipe pembayaran secara all time diurutkan dari yang terfavorit
SELECT payment_type, COUNT(1) 
FROM order_payments_dataset
GROUP BY 1
ORDER BY 2 DESC;

-- 2)Menampilkan detail informasi jumlah penggunaan masing-masing tipe pembayaran untuk setiap tahun
SELECT
	payment_type,
	SUM(CASE WHEN year = 2016 THEN total ELSE 0 END) AS "2016",
	SUM(CASE WHEN year = 2017 THEN total ELSE 0 END) AS "2017",
	SUM(CASE WHEN year = 2018 THEN total ELSE 0 END) AS "2018",
	SUM(total) AS sum_payment_type_usage
FROM (
	SELECT 
		date_part('year', od.order_purchase_timestamp) as year,
		opd.payment_type,
		COUNT(opd.payment_type) AS total
	FROM orders_dataset AS od
	JOIN order_payments_dataset AS opd 
		ON od.order_id = opd.order_id
	GROUP BY 1, 2
	) AS sub
GROUP BY 1
ORDER BY 2 DESC;
  ```
</details>

<p align="center">
  <kbd><img src="Assets/4.png" width=600px> </kbd> <br>
  <strong> Tabel 3. Hasil Analisis Tipe Pembayaran yang Digunakan Pelanggan </strong>
</p>
Berikut keterangan dari kolom pada table tersebut: <br>
- Payment_type: Tipe Pembayaran <br>
- 2016 : Jumlah pengguna pada tahun 2016 <br>
- 2017: Jumlah Pengguna pada tahun 2017 <br>
- 2018: Jumlah Pengguna pada tahun 2018 <br>
- Sum_payment_type: Total jumlah pengguna tipe pembayaran <br>

<br>
<p align="center">
  <kbd><img src="Assets/Picture1.png" width=600px> </kbd> <br>
  <strong> Gambar 8. Grafik Tipe Pembayaran yang Digunakan Pelanggan Pertahun </strong>
</p>

Mayoritas pelanggan melakukan pembayaran menggunakan kartu kredit dan cenderung mengalami peningkatan setiap tahunnya. Pembayaran menggunakan voucher meningkat pada tahun 2017, namun menurun pada tahun 2018. Hal tersebut dapat diduga karena ketersediaan voucher yang diberikan perusahaan lebih sedikit dari tahun lalu. Disisi lain, pelanggan yang melakukan pembayaran dengan kartu debit meningkat secara signifikan pada tahun 2018. Hal tersebut dapat diduga karena kemungkinan terdapat promosi pembayaran untuk kartu debit, sehingga banyak pelanggan yang tertarik untuk menggunakan metode tersebut.
<br>
<br>


---

## 📂 **STAGE 4: Summary**
- Dilihat dari analisis pertumbuhan tahunan pelanggan dapat disimpulkan bahwa **jumlah pelanggan baru dan aktif (MAU) meningkat setiap tahunnya**, namun pelanggan cenderung tidak repeat order atau hanya melakukan pembelian satu kali. Dari hal tersebut perlu adanya strategi bisnis untuk meningkatkan minat pelanggan agar melakukan pembelian misalnya dengan pemberian promo, *call to action*, dan lain sebagainya.
- Dari analisis kualitas produk tahunan, **revenue terus meningkat dengan kategori produk yang berbeda setiap tahunnya**. Kategori **kesehatan dan kecantikan** menjadi produk best seller sekaligus produk yang paling sering dibatalkan pembeliannya pada tahun 2018. Berdasarkan hasil analisis ini dapat dilakukan strategi bisnis berupa riset produk apa yang akan menjadi trend di tahun selanjutnya, sehingga diharapkan dapat memperbesar peluang perusahaan mendapatkan revenue.
- **Kartu kredit** merupakan tipe pembayaran mayoritas yang digunakan oleh pelanggan.
