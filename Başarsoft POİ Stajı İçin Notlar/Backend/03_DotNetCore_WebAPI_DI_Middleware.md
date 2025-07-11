
---

## 03_DotNetCore_WebAPI_DI_Middleware

### Giriş

Bu bölümde .NET Core Web API projesinin mimarisini, Dependency Injection (DI) altyapısını ve kendi middleware’ını nasıl yazıp devreye alacağımızı öğreneceğiz. Kod örneklerimiz “MapPoint” CRUD senaryosu üzerinde şekillenecek.

---

## 1. .NET Core (ASP.NET Core) Mimarisi

- **Kestirme Yol**: `dotnet new webapi -n MapApi`
    
- **Proje Yapısı**
    
    ```
    MapApi/
    ├─ Controllers/
    │   └─ MapPointsController.cs
    ├─ Models/
    │   └─ MapPoint.cs
    ├─ Data/
    │   └─ AppDbContext.cs
    ├─ Repositories/
    │   └─ IMapPointRepository.cs
    │   └─ MapPointRepository.cs
    ├─ Services/
    │   └─ MapPointService.cs
    ├─ Middleware/
    │   └─ RequestLoggingMiddleware.cs
    ├─ Program.cs
    └─ Startup.cs  *(varsa)*
    ```
    
- **Kilit Bileşenler**
    
    - `Program.cs` / `Startup.cs`
        
    - `AppDbContext` (EF Core)
        
    - Controller → Service → Repository akışı
        
    - Middleware pipeline
        

---

## 2. Dependency Injection (DI)

### 2.1 Neden DI?

- **Loose Coupling**: Sınıflar birbirine doğrudan new’leme ile bağlı değil, soyutlamalar üzerinden çalışır.
    
- **Test Edilebilirlik**: Mock veya stub’lar ile kolayca birim testi yazılabilir.
    
- **Yaşam Döngüsü Yönetimi**: Scoped / Singleton / Transient seçenekleriyle nesnelerin ömrünü kontrol edersin.
    

### 2.2 ASP.NET Core’da DI Kullanımı

#### 2.2.1 `Program.cs` ve `Startup.cs`

```csharp
// Program.cs (NET 6+ minimal hosting model)
var builder = WebApplication.CreateBuilder(args);

// 1) Hizmetleri kaydet (DI container'a)
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddScoped<IMapPointRepository, MapPointRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<MapPointService>();

// 2) Uygulamayı oluştur ve pipeline'ı kur
var app = builder.Build();

app.UseMiddleware<RequestLoggingMiddleware>();    // Custom middleware
app.UseHttpsRedirection();
app.MapControllers();

app.Run();
```

> **Not:** .NET 5 ve öncesi projelerde `Startup.cs` içinde `ConfigureServices` ve `Configure` yöntemleri bulunur; mantık aynı.

---

## 3. Controller → Service → Repository Akışı

1. **Controller** (HTTP isteklerini karşılar)
    
2. **Service** (İş kurallarını uygular)
    
3. **Repository/UnitOfWork** (Veri erişimini yönetir)
    

#### 3.1 `MapPointsController.cs`

```csharp
[ApiController]
[Route("api/[controller]")]
public class MapPointsController : ControllerBase
{
    private readonly MapPointService _service;

    public MapPointsController(MapPointService service)
    {
        _service = service;
    }

    [HttpGet]
    public IActionResult GetAll()
    {
        var points = _service.GetAll();
        return Ok(points);
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var point = _service.GetById(id);
        if (point == null) return NotFound();
        return Ok(point);
    }

    [HttpPost]
    public IActionResult Create([FromBody] MapPointDto dto)
    {
        _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = dto.Id }, dto);
    }

    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] MapPointDto dto)
    {
        _service.Update(id, dto);
        return NoContent();
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        _service.Delete(id);
        return NoContent();
    }
}
```

#### 3.2 `MapPointService.cs`

```csharp
public class MapPointService
{
    private readonly IUnitOfWork _uow;

    public MapPointService(IUnitOfWork uow)
        => _uow = uow;

    public IEnumerable<MapPoint> GetAll()
        => _uow.MapPoints.GetAll();

    public MapPoint GetById(int id)
        => _uow.MapPoints.GetById(id);

    public void Create(MapPointDto dto)
    {
        var point = new MapPoint {
            Name = dto.Name,
            X = dto.X,
            Y = dto.Y,
            LastUpdated = DateTime.UtcNow
        };
        _uow.MapPoints.Add(point);
        _uow.Complete();
        dto.Id = point.Id;
    }

    public void Update(int id, MapPointDto dto)
    {
        var point = _uow.MapPoints.GetById(id);
        point.X = dto.X;
        point.Y = dto.Y;
        point.LastUpdated = DateTime.UtcNow;
        _uow.MapPoints.Update(point);
        _uow.Complete();
    }

    public void Delete(int id)
    {
        var point = _uow.MapPoints.GetById(id);
        _uow.MapPoints.Delete(point);
        _uow.Complete();
    }
}
```

---

## 4. Middleware

### 4.1 Middleware Nedir?

- **Tanım:** HTTP pipeline’ına eklenen, istek (Request) ve yanıt (Response) üzerinde işlem yapan bileşenlerdir.
    
- **Kullanım Alanı:** Logging, hata yakalama, kimlik doğrulama, CORS, rate limiting vb.
    

### 4.2 Kendi Middleware’ını Yazmak

#### 4.2.1 `RequestLoggingMiddleware.cs`

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    public RequestLoggingMiddleware(RequestDelegate next)
        => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        var sw = Stopwatch.StartNew();
        Console.WriteLine($"[Request] {context.Request.Method} {context.Request.Path}");

        await _next(context);  // Bir sonraki middleware'a geç

        sw.Stop();
        Console.WriteLine($"[Response] {context.Response.StatusCode} in {sw.ElapsedMilliseconds}ms");
    }
}
```

#### 4.2.2 Pipeline’a Ekleme

```csharp
// Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

---

## 5. Exception Handling Middleware

### 5.1 Global Hata Yakalama

```csharp
public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;
    public ErrorHandlingMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            Console.Error.WriteLine($"[Error] {ex.Message}");
            context.Response.StatusCode = 500;
            await context.Response.WriteAsJsonAsync(new { error = ex.Message });
        }
    }
}
```

```csharp
// Program.cs içinde logging middleware’den önce:
app.UseMiddleware<ErrorHandlingMiddleware>();
app.UseMiddleware<RequestLoggingMiddleware>();
```

---

## 6. Özet ve İpuçları

- **Service Lifetimes:**
    
    - `AddSingleton` → Uygulama ömrü boyunca tek örnek
        
    - `AddScoped` → Her HTTP isteği için yeni örnek
        
    - `AddTransient` → Her injection’da yeni örnek
        
- **Middleware Sıralaması:** Hata yakalama middleware’ı **en üstte**, sonra logging, ardından routing ve yetkilendirme gelmeli.
    
- **Test:** Controller’ı doğrudan `MapPointService` mock’ları ile test edin; servis katmanını `IUnitOfWork` mock’larıyla izole edin.
    

---
