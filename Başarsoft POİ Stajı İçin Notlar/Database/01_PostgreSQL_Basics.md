
# 01_PostgreSQL_Basics

## Giriş  
Bu bölümde, hiç veritabanı deneyimi olmayan bir öğrenci için PostgreSQL’in ne olduğunu, nasıl kurulduğunu ve temel SQL komutlarını adım adım “Sebep – Amaç – Sonuç” üçlüsüyle açıklayacağız.

---

## 1. PostgreSQL Nedir ve Kurulumu

### 1.1 Sebep  
Günümüzde uygulamalar kullanıcı verilerini, ayarları ve log kayıtlarını saklamak zorundadır. Bu verileri güvenli, tutarlı ve ölçeklenebilir bir şekilde yönetmek için güçlü bir veritabanı sistemine ihtiyaç duyarız.

### 1.2 Amaç  
Açık kaynak kodlu, endüstri standardı bir ilişkisel veritabanı olan PostgreSQL’i kurarak, SQL komutlarıyla veri saklama ve sorgulama ortamını hazır hale getirmek.

### 1.3 Sonuç  
PostgreSQL sunucusu işletim sisteminde çalışır; `psql` komut satırı aracıyla bağlantı kurup veritabanı oluşturabilir, tablo tanımlayabilir ve veri yönetimine başlayabilirsiniz.

#### 1.3.1 Kurulum (Ubuntu / Debian örneği)
```bash
# Paket listelerini güncelle
sudo apt update

# PostgreSQL’i ve psql istemcisini yükle
sudo apt install -y postgresql postgresql-client

# Sunucunun çalıştığını kontrol et
sudo systemctl status postgresql
````

#### 1.3.2 Kurulum (Windows / macOS)

- **Windows:** [https://www.enterprisedb.com/downloads/postgres](https://www.enterprisedb.com/downloads/postgres) adresinden yükleyici indirin ve sihirbazı takip edin.
    
- **macOS (Homebrew ile):**
    
    ```bash
    brew update
    brew install postgresql
    brew services start postgresql
    ```
    BİR UFAK NOT: 
    Arkadaşlar en basiti windows, zaten çoğunluk da windows kullanıyor aramızdaki arkadaşlardan. GUİ uzantısı olan pgAdmin4 uygulamasını açarak basit bir şekilde next next diyerek sistemi kullanıma hazır hale getirebilriz.
    Ancak aramızda illa ben terminal kullancağım diyen varsa o da gayet tatlı bir yol ^-^

---

## 2. `psql` Kullanımı

### 2.1 Sebep

Grafiksel araçlar yerine terminal üzerinden hızlıca SQL çalıştırmak ve yönetimi komut satırından gerçekleştirmek isteyebiliriz.

### 2.2 Amaç

`psql` ile veritabanı sunucusuna bağlanmak, SQL komutlarını girmek ve temel yönetim görevlerini yapmayı öğrenmek.

### 2.3 Sonuç

Veritabanına doğrudan terminalden erişir; komut geçmişi, komut tamamlama ve sorgu çıktısını düzenli tablolar halinde görme avantajını elde edersiniz.

#### 2.3.1 `psql`’i başlatma

```bash
sudo -u postgres psql
```

- Veya belirli bir veritabanına:
    
    ```bash
    psql -h localhost -U postgres -d my_database
    ```
    

#### 2.3.2 Temel `psql` komutları

|Komut|Açıklama|
|---|---|
|`\l`|Sunucudaki veritabanlarını listeler|
|`\c <db>`|`<db>` veritabanına bağlanır|
|`\dt`|Mevcut tabloları gösterir|
|`\d <table>`|`<table>` yapısını özetler|
|`\q`|`psql`’den çıkar|

---

## 3. Temel DDL ve DML Komutları

### 3.1 CREATE DATABASE

#### Sebep

Uygulama verilerini saklayacak ayrı bir veritabanı olmalı; veri izolasyonu ve yönetimi kolaylaşır.

#### Amaç

Yeni bir veritabanı oluşturmak.

#### Uygulama ve Sonuç

```sql
CREATE DATABASE harita_app;
```

- Veritabanı sunucusunda `harita_app` adında boş bir veritabanı belirir.
    

---

### 3.2 CREATE TABLE

#### Sebep

Veriler satır ve sütunlardan oluşan tablolarda saklanır; her tablo belirli bir varlık tipini temsil eder.

#### Amaç

Örneğin `points` tablosunu, id ve koordinat bilgilerini tutacak şekilde tanımlamak.

#### Uygulama ve Sonuç

```sql
-- points tablosu: id, isim, x, y, son güncelleme zamanı
CREATE TABLE points (
  id           SERIAL PRIMARY KEY,
  name         TEXT    NOT NULL,
  x            DOUBLE PRECISION NOT NULL,
  y            DOUBLE PRECISION NOT NULL,
  last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

- `SERIAL` ile otomatik artan birincil anahtar, `NOT NULL` ile boş değer engellenir, zaman damgası otomatik atanır.
    

---

### 3.3 ALTER TABLE

#### Sebep

Tablo yapısında değişiklik yapmak (yeni sütun eklemek, sütun tipini değiştirmek) gerekebilir.

#### Amaç

Var olan tabloya ek sütun veya constraint eklemek.

#### Uygulama ve Sonuç

```sql
-- points tablosuna aktiflik sütunu ekle
ALTER TABLE points
ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
```

- `is_active` sütunu, yeni oluşturulan tüm kayıtlarda varsayılan olarak TRUE değeri alır.
    

---

### 3.4 INSERT

#### Sebep

Tabloya veri eklemeden hiçbir sorgu anlamlı olmaz; uygulama kullanıcı kaynaklarını kaydetmek ister.

#### Amaç

Yeni bir nokta kaydı eklemek.

#### Uygulama ve Sonuç

```sql
INSERT INTO points (name, x, y)
VALUES ('Okul', 29.02345, 41.01514);
```

- `points` tablosuna “Okul” adında bir satır eklenir; `id` ve `last_updated` otomatik doluşur.
    

---

### 3.5 SELECT

#### Sebep

Eklediğimiz veriyi okumak ve üzerinde işlem yapmak gerekir.

#### Amaç

Tablodaki tüm noktaları listelemek veya koşullu sorgu yapmak.

#### Uygulama ve Sonuç

```sql
-- Tüm noktalar
SELECT * FROM points;

-- X koordinatı 29’dan büyük olanlar
SELECT id, name, x, y
FROM points
WHERE x > 29;
```

- Sonuç: İlgili satırları tablo formatında görürsünüz.
    

---

### 3.6 UPDATE

#### Sebep

Veriler zamanla güncellenmeli; örneğin koordinatlar veya isim değişebilir.

#### Amaç

Bir kaydın alanlarını değiştirmek.

#### Uygulama ve Sonuç

```sql
UPDATE points
SET name = 'Güncellenmiş Okul', last_updated = NOW()
WHERE id = 1;
```

- `id=1` olan satırın `name` ve `last_updated` sütunları güncellenir.
    

---

### 3.7 DELETE

#### Sebep

Artık ihtiyaç kalmayan veya hatalı eklenmiş verileri silmek gerekir.

#### Amaç

Belirli bir satırı veya koşulları sağlayan satırları silmek.

#### Uygulama ve Sonuç

```sql
DELETE FROM points
WHERE id = 1;
```

- `id=1` olan satır tablo içerisinden kalıcı olarak kaldırılır.
    

---

## 4. Transaction, Index, Constraint ve View

### 4.1 Transaction Yönetimi

#### Sebep

Birden fazla deği̇şikli̇ği̇n hepsi̇ bı̇r arada başarılı ya da başarısız olması gėrekır (bankacılık işlemleri gibi).

#### Amaç

BEGIN/COMMIT/ROLLBACK komutlarıyla atomic işlem birimleri oluşturmak.

#### Uygulama ve Sonuç

```sql
BEGIN;

UPDATE points SET x = x + 0.1 WHERE id = 2;
DELETE FROM points WHERE id = 3;

-- Eğer her şey doğruysa
COMMIT;

-- Hata olursa değişiklikleri geri al
ROLLBACK;
```

- COMMIT sonrası değişiklikler kalıcı, ROLLBACK ile tüm değişiklikler iptal edilir.
    

---

### 4.2 Index Oluşturma

#### Sebep

SELECT ve JOIN sorgularını hızlı çalıştırmak için tablo üzerindeki sütunlara index eklemek gerekir.

#### Amaç

`points.x` sütununa hızlandırıcı bir index koymak.

#### Uygulama ve Sonuç

```sql
CREATE INDEX idx_points_x
  ON points (x);
```

- `idx_points_x` index’i, `x` sütunu üzerindeki sorguları belirgin şekilde hızlandırır.
    

---

### 4.3 Constraint’ler

#### Sebep

Veri bütünlüğünü (integrity) korumak; örneğin aynı isimde iki nokta olmasını engellemek.

#### Amaç

PRIMARY KEY, UNIQUE, FOREIGN KEY, NOT NULL, CHECK gibi kuralları tanımlamak.

#### Uygulama ve Sonuç

```sql
-- name sütununa UNIQUE constraint ekle
ALTER TABLE points
ADD CONSTRAINT uq_points_name UNIQUE (name);

-- id zaten PRIMARY KEY olarak tanımlı
-- x ve y sütunları için CHECK constraint
ALTER TABLE points
ADD CONSTRAINT chk_points_coords CHECK (x BETWEEN -180 AND 180 AND y BETWEEN -90 AND 90);
```

- Veri ekleme/güncelleme sırasında bu kurallara uymayan satırlar reddedilir.
    

---

### 4.4 View Oluşturma

#### Sebep

Complex sorguları yeniden kullanmak, güvenlik amacıyla belirli sütunları soyutlamak için.

#### Amaç

Sadece aktif noktaları içeren sanal bir tablo (view) tanımlamak.

#### Uygulama ve Sonuç

```sql
CREATE VIEW active_points AS
SELECT id, name, x, y
FROM points
WHERE is_active = TRUE;
```

- `SELECT * FROM active_points;` diyerek, her seferinde koşulu tekrar yazmadan aktif noktaları alabilirsiniz.
    

---

## Özet

Bu derste PostgreSQL kurulumundan başlayarak `psql` kullanımı, temel DDL/DML komutları, transaction yönetimi, index, constraint ve view kavramlarını “Sebep – Amaç – Sonuç” çerçevesinde öğrendik. Bir sonraki bölümde PostGIS uzantısına giriş yapacağız.


