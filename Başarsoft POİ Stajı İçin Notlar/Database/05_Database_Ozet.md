# Database Bölümü – Genel Özet 

---

## 1. 01_PostgreSQL_Basics.md  
- **Sebep:** Uygulamanın verileri güvenli ve tutarlı bir şekilde saklaması gerekiyor.  
- **Amaç:** PostgreSQL kurulumunu, `psql` ile bağlantıyı, temel DDL/DML (CREATE/ALTER/INSERT/SELECT/UPDATE/DELETE), transaction, index, constraint ve view kavramlarını öğretmek.  
- **Sonuç:** Terminalden veritabanı oluşturup yönetebilen, SQL’in temel komut setine hâkim bir geliştirici yetiştirdik.

---

## 2. 02_PostGIS_Giris_Uzanti.md  
- **Sebep:** Coğrafi veriler (nokta, çizgi, poligon) klasik tablolarla işlenemez; özel spatial fonksiyon ve tipler gerekir.  
- **Amaç:** `CREATE EXTENSION postgis;` ile PostGIS uzantısını aktif etmek; `geometry` ve `geography` tipleri arasındaki farkı, SRID kavramı ve yaygın projeksiyonları (4326, 3857 vb.) anlatmak.  
- **Sonuç:** Veritabanı spatial sorgulara hazır hale geldi; hangi senaryoda hangi tip ve SRID’in tercih edilmesi gerektiğini öğrendik.

---

## 3. 03_Spatial_Fonksiyonlar.md  
- **Sebep:** Harita verileri genellikle veritabanında ikili (binary) veya hex formatında saklanır; bu format insanlar için okunaksızdır.  
- **Amaç:**  
  1. **ST_AsText ve WKT (Well-Known Text)**  
     - WKT, `POINT(lon lat)`, `LINESTRING(x1 y1, x2 y2, …)`, `POLYGON((x1 y1, x2 y2, …, x1 y1))` gibi kolay anlaşılır metin formatıdır.  
     - `ST_AsText(geom)` fonksiyonuyla geometry sütununu WKT’ye dönüştürerek hem dokümantasyon hem de hata ayıklama için okunabilir çıktı elde ederiz.  
  2. **ST_AsGeoJSON** ile GeoJSON’a dönüşüm  
  3. **ST_Distance, ST_Intersects, ST_Within** gibi temel spatial ilişkileri hesaplayan fonksiyonlar  
- **Sonuç:**  
  - WKT sayesinde geometry verisini insan gözüyle inceleyebilir, loglayabilir ve test edebiliriz.  
  - GeoJSON çıktısı web harita kütüphanelerine direkt beslenebilir.  
  - Mesafe ve kapsama sorgularını SQL üzerinden kolayca gerçekleştirebilecek düzeye geldik.

---

## 4. 04_Sql_Ornekler_Crud_Spatial.md  
- **Sebep:** Backend tarafındaki CRUD işlemlerinde doğrudan SQL ihtiyacı olabilir; performans optimizasyonu için spatial index gereklidir.  
- **Amaç:** Nokta ekleme, güncelleme, silme SQL örnekleri; GIST spatial index oluşturma ve `EXPLAIN ANALYZE` ile performans incelemesini göstermek.  
- **Sonuç:** Spatial veriler üzerinde tam kontrol sağladık ve büyük veri setlerinde sorgu hızını artıracak optimizasyonları öğrendik.

---

### Bölüm Sonu Değerlendirmesi  
- **Veritabanı altyapınız**: PostgreSQL + PostGIS ile harita verisine hazır.  
- **SQL yetkinliğiniz**: Temel ve spatial SQL komutlarında yetkin.  
- **Performans bilinciniz**: Index ve transaction stratejileriyle donanımlı.

---


