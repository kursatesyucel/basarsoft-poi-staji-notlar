
# 03_Spatial_Fonksiyonlar

## Giriş  
**Sebep:** Harita uygulamalarında coğrafi veriyi sadece saklamak yetmez; bu verileri insan okuyabilir formata dönüştürmek, uzaklık ve kapsama ilişkilerini hesaplamak çok önemlidir.  
**Amaç:** PostGIS’in sağlayacağı temel spatial fonksiyonları (`ST_AsText`, `ST_AsGeoJSON`, `ST_Distance`, `ST_Intersects`, `ST_Within`) hem ne işe yaradıklarını hem de nasıl kullanıldıklarını anlamak.  
**Sonuç:** Veritabanında saklanan geometrik veriler üzerinde okunabilirlik, analiz ve mekânsal sorgulama kabiliyeti kazanacağız.

---

## 1. ST_AsText(geometry)

### Sebep  
Veritabanındaki `geometry` sütununa doğrudan bakmak insan için anlamsızdır (binary veya hex gösterim).

### Amaç  
Binary veya hex olarak saklanan geometrik objeyi “Well-Known Text” (WKT) formatına dönüştürerek okunabilir bir metin formu elde etmek.

### Uygulama  
```sql
-- points tablosu: id, name, geom (geometry(Point,4326))
SELECT id,
       name,
       ST_AsText(geom) AS wkt
FROM points
LIMIT 5;
````

### Sonuç

|id|name|wkt|
|---|---|---|
|1|Okul|POINT(29.02345 41.01514)|
|2|Kütüphane|POINT(29.02000 41.01280)|

Artık `POINT(lon lat)` formatını görebiliriz.

---

## 2. ST_AsGeoJSON(geometry)

### Sebep

Web uygulamaları genellikle GeoJSON formatını bekler; WKT yerine JSON tercih edilir.

### Amaç

Geometriyi GeoJSON nesnesine dönüştürerek JavaScript ve frontend kütüphanelerine direkt aktarabilmek.

### Uygulama

```sql
SELECT id,
       name,
       ST_AsGeoJSON(geom) AS geojson
FROM points
LIMIT 5;
```

### Sonuç

|id|name|geojson|
|---|---|---|
|1|Okul|{"type":"Point","coordinates":[29.02345,41.01514]}|
|2|Kütüphane|{"type":"Point","coordinates":[29.02,41.0128]}|

Bu JSON, Leaflet veya Mapbox’a direkt beslenebilir.

---

## 3. ST_Distance(geomA, geomB)

### Sebep

İki nokta veya iki obje arasındaki düzlem mesafesini veya coğrafi mesafeyi hesaplamak gerekir (en yakın istasyonu bulma, rota hesaplama).

### Amaç

İki geometry değerinin arasındaki uzaklığı metre veya varsayılan birimde ölçmek.

### Uygulama

```sql
-- Kullanıcı konumu
WITH user_loc AS (
  SELECT ST_SetSRID(ST_MakePoint(29.0250,41.0150),4326) AS geom
)
SELECT p.id,
       p.name,
       ST_Distance(p.geom::geography, u.geom::geography) AS distance_m
FROM points p, user_loc u
ORDER BY distance_m
LIMIT 5;
```

### Sonuç

|id|name|distance_m|
|---|---|---|
|2|Kütüphane|493.21|
|1|Okul|321.45|

Coğrafi (`geography`) tipi metre cinsinden doğru sonuç verir.

---

## 4. ST_Intersects(geomA, geomB)

### Sebep

Bir obje başka bir objeyle kesişiyor mu? Örneğin bir yol çizgisi bir bölge poligonu ile kesişiyorsa trafik analizi yapılabilir.

### Amaç

İki geometry objesinin mekânsal olarak çakışma (intersection) durumunu boolean olarak tespit etmek.

### Uygulama

```sql
-- lines tablosu: yol çizgileri
-- areas tablosu: bölge poligonları
SELECT l.id    AS line_id,
       a.id    AS area_id
FROM lines l
JOIN areas a
  ON ST_Intersects(l.geom, a.geom);
```

### Sonuç

|line_id|area_id|
|---|---|
|5|2|
|7|3|

Yollar hangi bölgelerle kesişiyor, bu eşleştirme tablosundan anlaşılır.

---

## 5. ST_Within(geomA, geomB)

### Sebep

Bir nokta veya çizgi tamamen başka bir poligon içinde mi yer alıyor? Örneğin bir kullanıcı harita içindeki parka girmiş mi kontrolü.

### Amaç

Bir geometry objesinin tamamen diğer geometry objesinin içindeyse `TRUE` döndürmek.

### Uygulama

```sql
-- Kullanıcı konumu: user_geom
-- parklar tablosu: park sınırları
WITH user_loc AS (
  SELECT ST_SetSRID(ST_MakePoint(29.0190,41.0140),4326) AS geom
)
SELECT a.id,
       a.name
FROM areas a, user_loc u
WHERE ST_Within(u.geom, a.geom);
```

### Sonuç

|id|name|
|---|---|
|1|Merkez Parkı|

Kullanıcı koordinatının hangi poligon içinde olduğunu bulduk.

---

## 6. Örnek Spatial Sorgular

### 6.1 Yakındaki Objeleri Bulma

**Senaryo:** Kullanıcının konumuna 1 km yarıçapındaki tüm noktaları listele.

```sql
WITH user_loc AS (
  SELECT ST_SetSRID(ST_MakePoint(29.0250,41.0150),4326) AS geom
)
SELECT p.id,
       p.name,
       ST_Distance(p.geom::geography, u.geom::geography) AS dist_m
FROM points p, user_loc u
WHERE ST_DWithin(p.geom::geography, u.geom::geography, 1000)
ORDER BY dist_m;
```

### 6.2 Bir Poligona Ait Tüm Noktaları Listeleme

**Senaryo:** “Merkez Parkı” poligonunun içinde kalan tüm noktaları çek.

```sql
SELECT p.id,
       p.name
FROM points p
JOIN areas a
  ON a.name = 'Merkez Parkı'
WHERE ST_Within(p.geom, a.geom);
```

### 6.3 Çizgi ile Kesişen Poligonları Bulma

**Senaryo:** Belirli bir yol (line_id=5) ile kesişen tüm park poligonlarını bul.

```sql
SELECT a.id,
       a.name
FROM areas a
WHERE ST_Intersects(
  a.geom,
  (SELECT geom FROM lines WHERE id = 5)
);
```

---

## Özet

Bu bölümde PostGIS’in en temel spatial fonksiyonlarını “Sebep – Amaç – Sonuç” formatında öğrendik ve gerçek dünya senaryolarına uygun örnek SQL sorgularıyla pekiştirdik.  
Bir sonraki adımda **04_Sql_Ornekler_Crud_Spatial.md** ile SQL üzerinden CRUD ve index oluşturmayı detaylandıracağız.

