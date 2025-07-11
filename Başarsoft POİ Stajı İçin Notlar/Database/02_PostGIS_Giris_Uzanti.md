
# 02_PostGIS_Giris_Uzanti

## Giriş  
Bu bölümde, harita uygulamamızda coğrafi verileri işlemek için PostgreSQL’e eklenen PostGIS uzantısını, coğrafi veri tiplerini ve koordinat referans sistemlerini adım adım “Sebep – Amaç – Sonuç” çerçevesinde inceleyeceğiz.  

---

## 1. PostGIS Uzantısının Etkinleştirilmesi

### 1.1 Sebep  
Klasik PostgreSQL, satır ve sütunlardan oluşan ilişkisel veriler için güçlüdür; ancak harita uygulamalarında nokta, çizgi ve poligon gibi geometrik şekilleri saklamak ve bunlar üzerinde uzamsal (spatial) sorgular yapmak için özel fonksiyon ve veri tiplerine ihtiyaç duyarız.

### 1.2 Amaç  
PostGIS uzantısını kurarak PostgreSQL’e spatial veri tipleri ve fonksiyonlarını eklemek; böylece coğrafi verilerimizi doğrudan veritabanında tutup, mesafe, alan, kapsama gibi işlemleri yüksek performansla gerçekleştirebilmek.

### 1.3 Uygulama  
Terminal veya `psql` üzerinden aşağıdaki komutu çalıştırın:
```sql
CREATE EXTENSION IF NOT EXISTS postgis;
````

### 1.4 Sonuç

- Veritabanınızda `geometry`, `geography` gibi spatial tipler ve `ST_Distance()`, `ST_Contains()` gibi PostGIS fonksiyonları kullanıma hazır hale gelir.

- **EDİTÖRÜN NOTU:** ST ile başlayan fonksiyonlar gerçekten çok önemli ve doğru kullanımı bize çok büyük kolaylıklar sağlıyor.
    
- İlk migration veya DDL betiğinizde bu komut otomatik eklenerek uzantı her ortamda yüklenir.
    

---

## 2. Geography vs Geometry Tipleri

### 2.1 Sebep

Coğrafi veriler “dünya üzerindeki eğriliği” göz önünde bulundurularak veya düzlem varsayımıyla ele alınabilir. Yanlış tip seçimi, mesafe ve alan hesaplamalarında büyük hatalara yol açar.

### 2.2 Amaç

- **geometry** tipini düzlemsel projeksiyonlarda (ör. şehir planlaması) kullanmak,
    
- **geography** tipini dünya eğriselliğini hesaba katan kabuksal hesaplamalar (uzun mesafe yol bulma, küresel analiz) için tercih etmek.
    

### 2.3 Uygulama

- **geometry**:
    
    ```sql
    CREATE TABLE geom_points (
      id   SERIAL PRIMARY KEY,
      geom geometry(Point, 3857)  -- Web Mercator projeksiyonu
    );
    ```
    
- **geography**:
    
    ```sql
    CREATE TABLE geo_points (
      id   SERIAL PRIMARY KEY,
      geo  geography(Point, 4326) -- Coğrafi koordinatlar (WGS84)
    );
    ```
    

### 2.4 Sonuç

- Eğer projenizde metre cinsinden doğru mesafe ölçümleri gerekiyorsa `geography` kullanın.
    
- Web haritalarında hız ve yaygın destek önemliyse, `geometry` + uygun projeksiyon (ör. 3857) tercih edin.

Doğru projeksiyon seçimi çok öenmli arkadaşlar, lise coğrafyasına gidersek hatırlarız ki dünyanın geoit şekli dolayısıyla her yerde aynı projeksiyon kullanılmıyordu. Sapmaları en aza indirmek için doğru projeksiyon seçilmeli.
    

---

## 3. SRID Kavramı ve Yaygın Projeksiyonlar

### 3.1 Sebep

Farklı harita servisleri ve coğrafi veri kaynakları, veri noktalarını farklı koordinat sistemlerinde (“projeksiyonlarda”) sunar. Uyumlu analiz için doğru SRID (Spatial Reference Identifier) kullanmak şarttır.

### 3.2 Amaç

- Her spatial sütunun hangi koordinat sistemini kullandığını tanımlamak,
    
- Sorgu ve dönüşümlerde tutarlılığı sağlamak.
    

### 3.3 Yaygın SRID’ler

|SRID|Adı|Kullanım Alanı|
|---|---|---|
|4326|WGS84|GPS, global coğrafi koordinatlar|
|3857|Web Mercator|İnternet harita servisleri (Google)|
|32633|UTM Zone 33N|Dar bölgelerde metre cinsinden analiz|
|27700|OSGB 1936 / British National Grid|Birleşik Krallık ulusal haritalar|

### 3.4 Uygulama Örneği

```sql
-- WGS84 coğrafi koordinat (derece)
CREATE TABLE points_wgs84 (
  id   SERIAL PRIMARY KEY,
  loc  geometry(Point, 4326)
);

-- Web Mercator projeksiyonu (metre)
CREATE TABLE points_merc (
  id   SERIAL PRIMARY KEY,
  loc  geometry(Point, 3857)
);
```

### 3.5 Sonuç

- SRID bilgisi, spatial verilerin doğru yorumlanmasını ve dönüşüm işlemlerinin (ST_Transform) sorunsuz çalışmasını sağlar.
    
- Farklı veri kaynaklarını birleştirirken veya harita katmanları üst üste bindirilirken koordinatlar tutarlı kalır.
    

---

## Özet

Bu bölümde PostgreSQL’e PostGIS uzantısını ekledik, spatial veri tipleri olarak `geometry` ve `geography` arasındaki farkları öğrendik ve SRID kavramını birçok yaygın örnekle pekiştirdik. Bir sonraki adımda **03_Spatial_Fonksiyonlar.md** ile bu spatial veriler üzerinde ST_… fonksiyonlarını uygulamalı olarak keşfedeceğiz.


