
# 04_Sql_Ornekler_Crud_Spatial

## Giriş  
**Sebep:** Harita uygulamasında coğrafi verileri SQL ile doğrudan yönetebilmek için `INSERT`/`UPDATE`/`DELETE` komutlarının spatial tiplerle nasıl çalıştığını bilmek gerekiyor.  
**Amaç:** Nokta (Point) verilerini nasıl ekleyeceğimizi, güncelleyeceğimizi ve sileceğimizi adım adım göstermek; ardından spatial index oluşturarak sorgu performansını nasıl iyileştireceğimizi öğrenmek.  
**Sonuç:** Hem veri manipülasyonunu hem de performans optimizasyonunu SQL düzeyinde kavrayarak, backend katmanının ihtiyaç duyduğu spatial CRUD yeteneklerini kazanacağız.

---

## 1. Nokta CRUD İşlemleri

### 1.1 Nokta Ekleme

**Sebep:** Uygulama kullanıcıları yeni bir harita noktası oluşturduklarında, bu noktanın longitude/latitude koordinatlarıyla veritabanına kaydedilmesi gerekir.  
**Amaç:** `points` tablosuna geometry tipinde bir nokta eklemek; SRID değerini doğru ayarlayarak coğrafi sorguların tutarlı çalışmasını sağlamak.  
**Uygulama:**
```sql
-- 1. Yeni nokta ekleme: ST_SetSRID + ST_MakePoint kullanarak
INSERT INTO points (name, geom, last_updated)
VALUES (
  'Yeni Durak',
  ST_SetSRID(ST_MakePoint(29.0250, 41.0150), 4326),
  NOW()
);
````

**Sonuç:**

- `points` tablosunda `geom` sütununa WGS84 (SRID=4326) koordinatlarla yeni bir satır oluşur.
    
- `last_updated` otomatik olarak şu anki zaman damgasını alır.
    

---

### 1.2 Nokta Güncelleme

**Sebep:** Mevcut bir noktanın konumu veya adı değiştiğinde, veritabanındaki kaydı güncellemek gerekir.  
**Amaç:** `UPDATE` komutuyla geometry sütununu ve gerekli metadata’yı (ör. `last_updated`) yenilemek.  
**Uygulama:**

```sql
-- 2. id=5 olan noktanın koordinatlarını ve adını güncelleme
UPDATE points
SET
  name         = 'Güncellenmiş Durak',
  geom         = ST_SetSRID(ST_MakePoint(29.0300, 41.0200), 4326),
  last_updated = NOW()
WHERE id = 5;
```

**Sonuç:**

- `id=5` kayıtlı satırın `geom` ve `name` değerleri yeni bilgilerle değiştirilir.
    
- `last_updated` sütunu, güncelleme zamanını yansıtır.
    

---

### 1.3 Nokta Silme

**Sebep:** Artık gösterilmeyecek veya yanlış eklenmiş bir nokta veritabanından kaldırılmalı.  
**Amaç:** `DELETE` komutuyla istenen satırı silmek; eğer geri almayı düşünüyorsak soft-delete stratejisi de uygulanabilir.  
**Uygulama:**

```sql
-- 3. id=7 olan noktayı fiziksel olarak silme
DELETE FROM points
WHERE id = 7;
```

> **Alternatif (Soft-Delete):** Fiziksel silme yerine `is_active = FALSE` güncellemesi yaparak veri geçmişini koruyabilirsiniz:

```sql
UPDATE points
SET is_active    = FALSE,
    last_updated = NOW()
WHERE id = 7;
```

**Sonuç:**

- Fiziksel silmede satır tamamen yok olur.
    
- Soft-delete yaklaşımında veri saklanır, ancak sorgularda `WHERE is_active = TRUE` eklemeniz gerekir.
    

---

## 2. Spatial Index Oluşturma ve Performans Notları

### 2.1 Spatial Index Oluşturma

**Sebep:** Spatial sorgular (örneğin yakınlık veya kapsama analizleri) büyük veri setlerinde yavaş çalışabilir; geometry sütunları için özel bir index gerekir.  
**Amaç:** PostGIS’in GIST index türünü kullanarak `geom` sütunu üzerinde spatial index oluşturmak.  
**Uygulama:**

```sql
-- 1. points.geom sütunu için GIST spatial index oluşturma
CREATE INDEX idx_points_geom
  ON points
  USING GIST (geom);
```

**Sonuç:**

- `idx_points_geom` adlı index, spatial sorguların (ST_DWithin, ST_Intersects, ST_Distance filter) çok daha hızlı çalışmasını sağlar.
    
- Index, geometrinin bounding box’ını kullanarak öncelikli filtreleme yapar.
    

---

### 2.2 Performans Notları

- **Index Maliyeti vs Kazanımı:**
    
    - **Sebep:** Her insert/update işleminde index de güncellenir; çok sık yazılan tabloya index koymak yazma performansını bir miktar düşürebilir.
        
    - **Amaç:** Okuma (SELECT) işlemlerinin hızını artırmak, yazma maliyetini kabul edilebilir düzeyde tutmak.
        
    - **Sonuç:**
        
        - Yazma yoğunluğu düşük, okuma yoğunluğu yüksek tablolar için spatial index mutlaka kullanılmalı.
            
        - Yüksek yazma yoğunluğu varsa, write-batch stratejisi veya index’i geçici kapatma düşünülebilir.
            
- **EXPLAIN ANALYZE ile Kontrol:**
    
    ```sql
    EXPLAIN ANALYZE
    SELECT id, name
    FROM points
    WHERE ST_DWithin(geom, ST_SetSRID(ST_MakePoint(29.0250,41.0150),4326), 1000);
    ```
    
    - **Sebep:** Gerçek sorgu planını ve index kullanımını görmek.
        
    - **Amaç:** GIST index’in devreye girip girmediğini, sorgu maliyetini ölçmek.
        
    - **Sonuç:**
        
        - `Bitmap Index Scan on idx_points_geom` ifadesi spatial index’in kullanıldığını gösterir.
            
        - Plan ve süre değerlerine bakarak optimizasyon yapabilirsiniz.
            
- **VACUUM ve ANALYZE:**
    
    - **Sebep:** PostgreSQL, index istatistiklerine göre sorgu planı oluşturur; istatistikler güncel olmalı.
        
    - **Amaç:** `VACUUM ANALYZE` komutuyla tablo ve index istatistiklerini güncellemek.
        
    - **Sonuç:**
        
        - Daha doğru plan ve performans ölçümleri elde edilir.
            
        - Sürekli değişen büyük veri setlerinde periyodik çalıştırılmalı.
            

---

## Özet

Bu dosyada SQL kullanarak spatial point verilerini ekleme, güncelleme ve silme işlemlerini “Sebep – Amaç – Sonuç” formatında öğrendik. Ardından GIST spatial index oluşturma ve performans optimizasyonuna dair önemli notları inceledik. Bir sonraki adımda, Database klasöründeki işlemleri tamamlayarak Frontend kısmına geçeceğiz.

