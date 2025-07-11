
## 05_EFCore_Spatial_PostGIS

### Giriş  
**Sebep:** Coğrafi veriler (nokta, çizgi, poligon) klasik veri tipleriyle işlenemez; harita uygulamalarında “gerçek” spatial sorgular ve indeksler gerekir.  
**Amaç:** EF Core + Npgsql + NetTopologySuite kombinasyonuyla PostGIS spatial uzantısını tam yetenekli kullanmak.  
**Sonuç:** Model sınıflarımız geometry tipine dönüştükten sonra, mesafe hesaplama, içindelik kontrolü, alan/uzunluk ölçümü gibi zengin PostGIS fonksiyonlarını LINQ üzerinden kullanabileceğiz.

---

## 1. PostGIS Uzantısını Etkinleştirme

**Sebep:** PostGIS yüklenmemişse spatial tipler (geometry, geography) ve fonksiyonlar veritabanında mevcut olmaz.  
**Amaç:** EF Core migration’ları çalıştırıldığında `CREATE EXTENSION postgis;` komutu otomatik tetiklensin.  
**Uygulama:**  
```csharp
// AppDbContext.OnModelCreating içinde
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // PostGIS spatial desteğini ekle
    modelBuilder.HasPostgresExtension("postgis");

    // Eğer belirli SRID ile geometry tanımlamak isterseniz:
    modelBuilder.Entity<MapPoint>()
        .Property(p => p.Location)
        .HasColumnType("geometry (Point, 4326)");
}
````

**Sonuç:** Migration’daki ilk SQL script’inde `CREATE EXTENSION IF NOT EXISTS postgis;` yer alır ve geometry sütunları sorunsuz oluşturulur ([npgsql.org](https://www.npgsql.org/efcore/mapping/nts.html?utm_source=chatgpt.com "Spatial Mapping with NetTopologySuite | Npgsql Documentation")).

---

## 2. Spatial Modelleri Tanımlama

**Sebep:** Çift `X,Y` değerinden ziyade tek bir `Point` nesnesi kullanmak, hem kodu sadeleştirir hem de NetTopologySuite’in zengin API’sinden faydalanmamızı sağlar.  
**Amaç:** Model sınıflarında NetTopologySuite tiplerini kullanarak EF Core ile PostGIS “geometry” sütunlarına doğrudan eşleme yapmak.  
**Uygulama:**

```csharp
// Models/MapPoint.cs
using NetTopologySuite.Geometries;

public class MapPoint
{
    public int    Id          { get; set; }
    public string Name        { get; set; } = null!;
    public Point  Location    { get; set; }   // NetTopologySuite.Geometries.Point
    public DateTime LastUpdated { get; set; }
}
```

- `Point` türü longitude=X, latitude=Y olarak ele alınır ([Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/modeling/spatial?utm_source=chatgpt.com "Spatial Data - EF Core | Microsoft Learn")).
    
- SRID: 4326 (WGS84) yaygın coğrafi koordinat sistemidir.
    

**Sonuç:** `MapPoint.Location` property’si, PostGIS’te `geometry(Point,4326)` sütununa karşılık gelir ve CRUD’da `Point` nesneleri otomatik olarak okunup yazılır.

---

## 3. Spatial Fonksiyonlar ve EF Core

**Sebep:** Harita uygulamasında “yakındaki noktaları bul”, “hangi poligon içinde” gibi sorgular performansla yapılmalı.  
**Amaç:** EF Core üzerinden LINQ yazar gibi spatial fonksiyonları (`Distance`, `Contains`, `Intersects`, `Area`) kullanmak.  
**Uygulama:**

```csharp
using NetTopologySuite.Geometries;
using Microsoft.EntityFrameworkCore;

// 3.1: Belirli bir lokasyona en yakın noktayı bulma
var currentLocation = new Point(userLon, userLat) { SRID = 4326 };
var nearest = await context.MapPoints
    .OrderBy(p => p.Location.Distance(currentLocation))
    .FirstOrDefaultAsync();

// 3.2: İçinde olduğumuz poligonu bulma
var containingPolygon = await context.PolygonFeatures
    .FirstOrDefaultAsync(pg => pg.Area.Contains(currentLocation));

// 3.3: 5 kilometre yarıçapındaki noktalar
double radiusInMeters = 5000;
var nearby = await context.MapPoints
    .Where(p => p.Location.IsWithinDistance(currentLocation, radiusInMeters))
    .ToListAsync();
```

- `Distance()`, `Contains()`, `IsWithinDistance()` gibi method’lar EF Core tarafından `ST_Distance`, `ST_Contains`, `ST_DWithin` SQL fonksiyonlarına çevrilir ([Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/modeling/spatial?utm_source=chatgpt.com "Spatial Data - EF Core | Microsoft Learn"), [Stack Overflow](https://stackoverflow.com/questions/61452283/ef-core-spatial-data-query-and-get-distances-in-meters?utm_source=chatgpt.com "EF Core Spatial Data: Query and get distances in meters")).
    
- Birim: geography tipi kullanmadıysanız `Distance()` derecede, `IsWithinDistance()` ise metre/metre cinsinden olabilir; projeksiyon ayarınıza dikkat edin.
    

**Sonuç:** Tek satırlık LINQ sorgularıyla karmaşık spatial işlemler yapılabilir, kod hem okunaklı hem de performanslıdır.

---

## 4. Spatial CRUD Örnekleri

**Sebep:** CRUD işlemlerinde geometry nesnelerinin nasıl oluşturulacağını ve güncelleneceğini bilmek gerekiyor.  
**Amaç:** Yeni bir MapPoint eklerken veya güncellerken `Point` nesnesi yaratma pratiklerini göstermek.  
**Uygulama:**

```csharp
// 4.1: Yeni nokta ekleme
var pt = new MapPoint {
    Name        = "Kütüphane",
    Location    = new Point(29.0200, 41.0128) { SRID = 4326 },
    LastUpdated = DateTime.UtcNow
};
context.MapPoints.Add(pt);
context.SaveChanges();

// 4.2: Nokta güncelleme
var existing = context.MapPoints.Find(pt.Id);
existing.Location = new Point(29.0250, 41.0150) { SRID = 4326 };
existing.LastUpdated = DateTime.UtcNow;
context.SaveChanges();

// 4.3: Nokta silme
context.MapPoints.Remove(existing);
context.SaveChanges();
```

- `new Point(lon, lat) { SRID = 4326 }` ile coğrafi koordinatı doğru SRID’le saklamış oluruz.
    
- `SaveChanges()` tüm değişiklikleri bir transaction içinde uygular.
    

**Sonuç:** CRUD senaryolarında spatial tipler de tıpkı diğer property’ler gibi kolayca yönetilebilir.

---

## 5. Performans ve İpuçları

1. **Spatial Index**  
    **Sebep:** Spatial sorgular (örneğin mesafe, kapsama) büyük veri setlerinde yavaş olabilir.  
    **Amaç:** PostGIS’te GIST index ile sorgu maliyetini düşürmek.  
    **Uygulama SQL:**
    
    ```sql
    CREATE INDEX idx_mappoint_location
      ON map_points
      USING GIST (location);
    ```
    
    **Sonuç:** `ST_DWithin`, `ST_Contains` gibi fonksiyonlar index’i kullanarak çok hızlı çalışır.
    
2. **Geography vs Geometry**
    
    - **Geography**: gerçek dünya eğriliğini dikkate alır, metre cinsinden tamamen doğru sonuç verir.
        
    - **Geometry**: düzlem varsayımı, derecede sonuç üretir; basit uygulamalar için yeterli olabilir.
        
3. **Projeksiyon Dönüşümü**  
    **Sebep:** Dünya eğriselliği hesaba katılmalıdır.  
    **Amaç:** Coğrafi koordinatları proje edilmiş koordinat sistemine (örneğin EPSG:3857) dönüştürüp `Point` oluşturmak.  
    **Sonuç:** Mesafe ölçümleri gerçek dünya metre cinsinden doğru yapılır.
    
4. **EF Core Logging**  
    **Sebep:** Üretilen SQL’i görmek ve optimize etmek.  
    **Amaç:**
    
    ```csharp
    builder.Services.AddDbContext<AppDbContext>(opts =>
      opts.UseNpgsql(connStr, o => o.UseNetTopologySuite())
          .EnableSensitiveDataLogging()
    );
    ```
    
    **Sonuç:** Konsolda `ST_Distance`, `ST_Contains` çağrılarını ve parametrelerini gözlemleyebilirsiniz.
    

---

**Bir Sonraki Adım:**  
EF Core spatial sorgularını ve PostGIS fonksiyonlarını öğrendik.  
Devamında **06_Repository_UnitOfWork_Pattern.md** ile bu spatial işlemleri repository katmanına nasıl taşıyacağımızı göreceğiz.
