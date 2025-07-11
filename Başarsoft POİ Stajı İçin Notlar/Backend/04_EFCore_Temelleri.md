
## 04_EFCore_Temelleri

### Giriş  
**Sebep:** Projemizde veritabanı işlemlerini elle SQL yazmadan, güçlü bir şekilde yönetmek istiyoruz.  
**Amaç:** Entity Framework Core (EF Core) kullanarak model-veritabanı eşleştirmesini, migration’ları ve CRUD işlemlerini kolaylaştırmak.  
**Sonuç:** Kod tarafında tanımladığımız C# sınıfları doğrudan ilişkisel tablo ve sütunlara dönüşecek; şema değişikliklerini versiyonlayabilecek ve basit kodla veri işleyeceğiz.

---

## 1. EF Core Kurulumu

**Sebep:** EF Core paketleri ve araçları kurulu değilse IDE ve CLI komutları çalışmaz; provider’lar yüklenmez.  
**Amaç:** EF Core’la çalışmak için gerekli NuGet paketlerini ve dotnet-ef CLI aracını sisteme eklemek.  
**Uygulama:**
```bash
cd backend/BasarMapApp.Api

# EF Core runtime ve design paketleri
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Tools

# PostgreSQL sağlayıcısı ve spatial desteği
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite

# EF Core komut satırı aracı
dotnet tool install --global dotnet-ef
dotnet ef --version
````

**Sonuç:**

- `dotnet ef` komutları terminalde tanınır.
    
- Projemiz Npgsql ile PostgreSQL’e, NetTopologySuite ile spatial veri tiplerine hazır hale gelir.
    

---

## 2. AppDbContext Yapılandırması

**Sebep:** EF Core’da veritabanı bağlantısı ve model-veritabanı eşleştirmesi `DbContext` sınıfı üzerinden yapılır.  
**Amaç:** `AppDbContext` ile hangi model sınıflarının hangi tablolara karşılık geleceğini ve ek uzantıların (PostGIS) etkinleşmesini tanımlamak.  
**Uygulama:**

```csharp
// backend/BasarMapApp.Api/Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using BasarMapApp.Api.Models;

namespace BasarMapApp.Api.Data
{
    public class AppDbContext : DbContext
    {
        // DI ile options nesnesi alınır => bağlantı dizesi ve provider ayarları burada tanımlı
        public AppDbContext(DbContextOptions<AppDbContext> options)
            : base(options)
        { }

        // Model sınıflarımızı DbSet'lerle tanımlıyoruz
        public DbSet<MapPoint>       MapPoints  { get; set; } = null!;
        public DbSet<LineFeature>    Lines      { get; set; } = null!;
        public DbSet<PolygonFeature> Polygons   { get; set; } = null!;

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // EF Core'a PostGIS uzantısının kullanılacağını bildiriyoruz
            modelBuilder.HasPostgresExtension("postgis");

            // Ek yapılandırmalar (naming conventions, default values) buraya eklenebilir
            base.OnModelCreating(modelBuilder);
        }
    }
}
```

**Sonuç:**

- Uygulama başlatıldığında DI container’dan `AppDbContext` örneği veritabanı bağlantısıyla oluşturulur.
    
- `MapPoint`, `LineFeature`, `PolygonFeature` sınıfları tablolara karşılık gelir.
    
- `postgis` uzantısı her sorguda aktif olur, spatial tipler sorunsuz çalışır.
    

---

## 3. Spatial Veri Desteği (PostGIS + NetTopologySuite)

**Sebep:** Coğrafi veriler (POINT, LINESTRING, POLYGON) özel binary formatta saklanır; normal tipler yetersiz kalır.  
**Amaç:** EF Core’un Npgsql sağlayıcısıyla PostGIS spatial özelliklerini kullanmak için spatial plugin’i etkinleştirmek.  
**Uygulama:**

```csharp
// Program.cs (Minimal API)
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        npgsqlOpts => npgsqlOpts.UseNetTopologySuite()
    )
);

builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

**Sonuç:**

- Spatial tip içeren property’ler (`Geometry`, `Point`, vb.) NetTopologySuite nesnelerine dönüştürülür.
    
- LINQ sorgularında `ST_Distance`, `ST_Intersects` gibi PostGIS fonksiyonları kullanılabilir.
    

---

## 4. Migration (Veritabanı Şeması Yönetimi)

**Sebep:** Proje ilerledikçe model sınıflarında değişiklik olacak; tablo kolonları otomatik güncellenmeli.  
**Amaç:** Kod ile veritabanı şemasını senkronize eden migration’ları versiyonlamak ve gerektiğinde geri almak.  
**Uygulama:**

```bash
# İlk migration
dotnet ef migrations add InitialCreate \
  --project backend/BasarMapApp.Api \
  --startup-project backend/BasarMapApp.Api

# Veritabanını oluştur/güncelle
dotnet ef database update \
  --project backend/BasarMapApp.Api \
  --startup-project backend/BasarMapApp.Api
```

**Sonuç:**

- `Migrations/` klasöründe tarih-içerikli migration sınıfları oluşur.
    
- `dotnet ef database update` ile SQL script’leri otomatik çalışır, tablolar ve uzantılar hazırlanır.
    

---

## 5. Temel CRUD Örnekleri

**Sebep:** Repository veya service katmanına geçmeden önce doğrudan `DbContext` ile veri işlemlerini anlamak yararlı.  
**Amaç:** EF Core API’siyle Create, Read, Update, Delete işlemlerinin temel metotlarını öğrenmek.  
**Uygulama:**

```csharp
using var context = new AppDbContext(/* options */);

// Create
var p = new MapPoint {
    Name        = "Yeni Nokta",
    X           = 28.9784,
    Y           = 41.0082,
    LastUpdated = DateTime.UtcNow
};
context.MapPoints.Add(p);
context.SaveChanges();

// Read
var allPoints = context.MapPoints.ToList();
var single    = context.MapPoints.Find(p.Id);

// Update
single.Name        = "Güncellenmiş Nokta";
single.LastUpdated = DateTime.UtcNow;
context.SaveChanges();

// Delete
context.MapPoints.Remove(single);
context.SaveChanges();
```

**Sonuç:**

- `DbSet<T>` üzerinde `Add`, `Find`, `Update`, `Remove` metotları kullanımı pekişir.
    
- `SaveChanges()` her değişikliği tek bir transaction olarak veritabanına gönderir.
    

---

## 6. LINQ to Entities vs LINQ to Objects

**Sebep:** LINQ sorgularının veritabanında mı yoksa bellekte mi çalıştığını bilmek performans için kritik.  
**Amaç:** İki yaklaşım arasındaki farkı kavrayıp doğru senaryoda tercih etmek.  
**Uygulama:**

```csharp
// LINQ to Entities — SQL’e çevirilir
var distant = context.MapPoints
    .Where(p => p.X > 30)
    .ToList();

// LINQ to Objects — önce tüm veriyi belleğe alır
var inMemory = context.MapPoints
    .ToList()
    .Where(p => p.X > 30);
```

**Sonuç:**

- Büyük veri setlerinde `ToList()` öncesi filtre koymak (LINQ to Entities) daha performant.
    
- Bellek içi filtreleme yalnızca küçük veri listelerinde veya önceden çekilmiş sonuçlarla kullanılmalı.
    

---

## 7. İleri İpuçları

- **DbContext Ömrü**
    
    - **Sebep:** Yanlış lifetime bellek sızıntısı veya bağlantı sorunları yaratabilir.
        
    - **Amaç:** `AddDbContext` ile varsayılan Scoped (her istek) ömrü kullanmak.
        
    - **Sonuç:** Her HTTP isteği temiz bir context örneğiyle çalışır.
        
- **Global Query Filter**
    
    - **Sebep:** Soft-delete veya ortak filtreleri her sorguda tekrar eklemek zahmetli.
        
    - **Amaç:** ModelBuilder ile entity bazlı filtre tanımlamak.
        
    - **Sonuç:** Sorgular otomatik olarak filtrelenir, kodunuzu temiz tutarsınız.
        
- **Performans & İzleme**
    
    - **Sebep:** SQL jenerasyonu ve parametre değerleri gizlenebilir.
        
    - **Amaç:** `EnableSensitiveDataLogging()` ile detaylı logging açmak.
        
    - **Sonuç:** Geliştirme ortamında sorgu içi değerleri görebilir, hataları hızlı tespit edersiniz.
        
- **Spatial Index**
    
    - **Sebep:** Spatial sorgular (yakınlık, kapsama) yavaş çalışabilir.
        
    - **Amaç:** PostGIS’te GIST index oluşturmak.
        
    - **Uygulama SQL:**
        
        ```sql
        CREATE INDEX idx_mappoints_geom
          ON map_points
          USING GIST (geom);
        ```
        
    - **Sonuç:** Spatial sorgular dramatik şekilde hızlanır.
        

---

**Bir Sonraki Adım:**  
Bu derste EF Core’un temellerini ve spatial entegrasyonunu öğrendik.  
Sonraki dosyamız **05_EFCore_Spatial_PostGIS.md** ile PostGIS fonksiyonları ve gelişmiş spatial sorgulamalara geçeceğiz.
