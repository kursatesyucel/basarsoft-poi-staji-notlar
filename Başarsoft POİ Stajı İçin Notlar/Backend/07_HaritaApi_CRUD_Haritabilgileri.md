
## 07_HaritaApi_CRUD_Haritabilgileri

### Giriş  
**Sebep:** Frontend uygulaması JSON üzerinden nokta, çizgi ve poligon verilerini gönderecek; backend tarafında bu verileri alıp veritabanına kaydetmek, güncellemek, silmek ve listelemek gerekli.  
**Amaç:** DTO (Data Transfer Object) kullanarak dış dünyaya açılan API modellerini tanımlamak, controller katmanında bu DTO’ları service katmanına yönlendirerek CRUD işlemlerini gerçekleştirmek.  
**Sonuç:** Hem iç modelimizi (entity) hem de dışa dönük JSON yapısını kontrol altında tutar, validation ve versioning gibi ihtiyaçlara kolayca adapte oluruz.

---

## 1. DTO Tanımları

### 1.1 MapPointDto

**Sebep:** Entity modelde `Location: Point` gibi spatial tipler varken, API client genellikle `latitude`/`longitude` değerleri gönderir.  
**Amaç:** Harita noktası için sadece gerekli alanları içeren, JSON serileştirmeye uygun DTO sınıfı tanımlamak.  
```csharp
// backend/BasarMapApp.Api/Repositories/DTOs/MapPointDto.cs
public class MapPointDto
{
    public int?    Id        { get; set; }  // Oluşturma sonrası dolacak
    public string  Name      { get; set; }  = null!;
    public double  Longitude { get; set; }  
    public double  Latitude  { get; set; }  
}
````

**Sonuç:** Client sadece `Name`, `Longitude`, `Latitude` gönderir; `Id` ve `LastUpdated` backend tarafından yönetilir.

### 1.2 LineFeatureDto ve PolygonFeatureDto

Benzer prensiplerle, çizgi ve poligon için koordinat listelerini DTO’da tutarız:

```csharp
// LineFeatureDto.cs
public class LineFeatureDto
{
    public int?           Id          { get; set; }
    public string         Name        { get; set; } = null!;
    public List<PointDto> Coordinates { get; set; } = new();
}

// PolygonFeatureDto.cs
public class PolygonFeatureDto
{
    public int?           Id          { get; set; }
    public string         Name        { get; set; } = null!;
    public List<PointDto> Coordinates { get; set; } = new();
}

// PointDto.cs (yardımcı)
public class PointDto
{
    public double Longitude { get; set; }
    public double Latitude  { get; set; }
}
```

---

## 2. Controller Katmanı

### 2.1 MapPointsController

**Sebep:** HTTP isteklerini karşılayıp, gelen DTO’ları MapPoint entity’sine dönüştürmek ve service katmanını çağırmak.  
**Amaç:** Async CRUD endpoint’leri oluşturmak; model validation, hata yönetimi ve uygun HTTP yanıt kodları ile.

```csharp
// backend/BasarMapApp.Api/Controllers/MapPointsController.cs
using Microsoft.AspNetCore.Mvc;
using NetTopologySuite.Geometries;
using BasarMapApp.Api.Repositories.DTOs;
using BasarMapApp.Api.Services;

[ApiController]
[Route("api/[controller]")]
public class MapPointsController : ControllerBase
{
    private readonly MapPointService _service;
    public MapPointsController(MapPointService service) => _service = service;

    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var points = await _service.GetAllAsync();
        return Ok(points);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var point = await _service.GetByIdAsync(id);
        if (point == null) return NotFound();
        return Ok(point);
    }

    [HttpPost]
    public async Task<IActionResult> Create([FromBody] MapPointDto dto)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);

        var entity = new MapPoint
        {
            Name        = dto.Name,
            Location    = new Point(dto.Longitude, dto.Latitude) { SRID = 4326 },
            LastUpdated = DateTime.UtcNow
        };
        var created = await _service.CreateAsync(entity);
        dto.Id = created.Id;
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, dto);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, [FromBody] MapPointDto dto)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);
        var existing = await _service.GetByIdAsync(id);
        if (existing == null) return NotFound();

        existing.Name        = dto.Name;
        existing.Location    = new Point(dto.Longitude, dto.Latitude) { SRID = 4326 };
        existing.LastUpdated = DateTime.UtcNow;

        await _service.UpdateAsync(existing);
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var existing = await _service.GetByIdAsync(id);
        if (existing == null) return NotFound();

        await _service.DeleteAsync(id);
        return NoContent();
    }
}
```

**Sonuç:**

- Tüm CRUD işlemleri standart HTTP kodlarıyla (200, 201, 204, 400, 404) yapılır.
    
- DTO ⇄ Entity dönüşümü controller’da net olarak yönetilir.
    

### 2.2 LineFeaturesController ve PolygonFeaturesController

**Sebep & Amaç:** Noktalarda olduğu gibi, Lines ve Polygons için de benzer controller’lar oluşturmak; sadece `Location` yerine `LineString` ve `Polygon` inşa etmek.

```csharp
// Örnek: backend/BasarMapApp.Api/Controllers/LineFeaturesController.cs
[ApiController]
[Route("api/[controller]")]
public class LineFeaturesController : ControllerBase
{
    private readonly MapPointService _service; // Burada uygun line service
    // … MapPointController’a benzer CRUD metodları, 
    // dto.Coordinates listesinden new LineString([...]) oluşturulur 
}
```

**Sonuç:**

- Her geometrik tip için ayrı endpoint’ler; frontend’e net sözleşme (contract) sunar.
    
- Kod tekrarını azaltmak için ortak base controller veya generic controller altyapısı geliştirilebilir.
    

---

## 3. Model Validation & Hata Yönetimi

**Sebep:** Gelen verinin (örneğin `Longitude`=200) geçersiz olması durumunda sistemin çökmesini veya tutarsız veri kaydetmesini engellemek.  
**Amaç:** `[Required]`, `[Range]` gibi DataAnnotation attribute’ları ile otomatik model doğrulama, `ModelState.IsValid` kontrolü ve anlamlı hata mesajları.

```csharp
public class MapPointDto
{
    public int? Id { get; set; }

    [Required]
    public string Name { get; set; } = null!;

    [Range(-180, 180)]
    public double Longitude { get; set; }

    [Range(-90, 90)]
    public double Latitude { get; set; }
}
```

**Sonuç:**

- Geçersiz veri girildiğinde 400 Bad Request döner; kullanıcı hatası net anlaşılır.
    
- Service ve database katmanına sadece temiz, valid veri ulaşır.
    

---

## 4. Swagger / OpenAPI Entegrasyonu

**Sebep:** API dokümantasyonunu manuel yazmak zahmetli ve güncel tutması zor.  
**Amaç:** Swagger ile tüm controller’ları otomatik tarayıp interaktif dokümantasyon sunmak.  
**Uygulama:**

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseSwagger();
app.UseSwaggerUI(c => {
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Map API V1");
});
```

**Sonuç:**

- `/swagger` adresinden API dokümantasyonu ve test UI’sine erişilir.
    
- Frontend ekip kendi gönderilerini burada deneyerek doğrulayabilir.
    

---

Arkadaşalr 
