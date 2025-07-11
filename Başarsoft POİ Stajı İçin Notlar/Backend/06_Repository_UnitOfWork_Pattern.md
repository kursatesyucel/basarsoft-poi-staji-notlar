
## 06_Repository_UnitOfWork_Pattern

### Giriş  
**Sebep:** Veritabanı erişim kodunu doğrudan `DbContext` üzerinden yazmak, zamanla controller veya service katmanında tekrar eden, zor test edilebilir kodlara yol açar.  
**Amaç:** Repository deseniyle veri erişimini soyutlamak, Unit of Work deseniyle birden fazla repository’e yapılan değişiklikleri tek bir transaction’da toplamak.  
**Sonuç:** Katmanlı mimari içinde veri erişim kodu merkezi bir noktada toplanır, test ve bakım kolaylaşır, tutarlılık ve atomicity sağlanır.

---

## 1. Repository Deseni

### 1.1 Tanım ve Faydaları  
- **Tanım:** Her bir aggregate root (örneğin `MapPoint`) için CRUD operasyonlarını ve sorguları tanımlayan soyut arayüzler ve somut sınıflar.  
- **Faydalar:**  
  - *Soyutlama:* Veritabanı teknolojisi (`EF Core`) değişse bile repository arayüzü sabit kalır.  
  - *Tekrarsız Kod:* CRUD kodunu her entity için tekrar yazmak yerine generic repository kullanarak tek noktada toplarsınız.  
  - *Test Edilebilirlik:* `IGenericRepository<T>` arayüzleri mock’lanabilir, birim test yazımı kolaylaşır.

### 1.2 `IGenericRepository<T>` Arayüzü

```csharp
// backend/BasarMapApp.Api/Repositories/IGenericRepository.cs
using System.Linq.Expressions;

namespace BasarMapApp.Api.Repositories
{
    public interface IGenericRepository<T> where T : class
    {
        // Tüm kayıtları döner
        Task<IEnumerable<T>> GetAllAsync();
        // Filtreli kayıtlar döner
        Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
        // Tek bir kayıt döner
        Task<T?> GetByIdAsync(int id);
        // Yeni kayıt ekler
        Task AddAsync(T entity);
        // Birden fazla ekleme
        Task AddRangeAsync(IEnumerable<T> entities);
        // Kayıt güncelle
        void Update(T entity);
        // Kayıt sil
        void Remove(T entity);
        // Birden fazla silme
        void RemoveRange(IEnumerable<T> entities);
    }
}
````

- **Açıklama:**
    
    - `Expression<Func<T,bool>> predicate` ile esnek filtreleme.
        
    - `Async` metotlarla I/O bekleme sürelerini non-blocking yönetme.
        

### 1.3 `GenericRepository<T>` Uygulaması

```csharp
// backend/BasarMapApp.Api/Repositories/GenericRepository.cs
using Microsoft.EntityFrameworkCore;
using System.Linq.Expressions;
using BasarMapApp.Api.Data;

namespace BasarMapApp.Api.Repositories
{
    public class GenericRepository<T> : IGenericRepository<T> where T : class
    {
        protected readonly AppDbContext _context;
        private readonly DbSet<T>       _dbSet;

        public GenericRepository(AppDbContext context)
        {
            _context = context;
            _dbSet    = _context.Set<T>();
        }

        public async Task<IEnumerable<T>> GetAllAsync() =>
            await _dbSet.ToListAsync();

        public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate) =>
            await _dbSet.Where(predicate).ToListAsync();

        public async Task<T?> GetByIdAsync(int id) =>
            await _dbSet.FindAsync(id);

        public async Task AddAsync(T entity) =>
            await _dbSet.AddAsync(entity);

        public async Task AddRangeAsync(IEnumerable<T> entities) =>
            await _dbSet.AddRangeAsync(entities);

        public void Update(T entity) =>
            _dbSet.Update(entity);

        public void Remove(T entity) =>
            _dbSet.Remove(entity);

        public void RemoveRange(IEnumerable<T> entities) =>
            _dbSet.RemoveRange(entities);
    }
}
```

- **Nasıl Çalışır?**
    
    - Constructor’da `AppDbContext` ve ilgili `DbSet<T>` alınır.
        
    - CRUD ve sorgu metotları doğrudan `_dbSet` üzerinden çalışır.
        

---

## 2. Unit of Work Deseni

### 2.1 Tanım ve Faydaları

- **Tanım:** Birden çok repository üzerinde yapılan işlemleri tek bir `SaveChanges()` çağrısında toplar; transaction atomicity ve tutarlılık sağlar.
    
- **Faydalar:**
    
    - _Atomicity:_ Birden fazla ekleme/güncelleme/silme işlemi tek bir transaction’da tamamlanır.
        
    - _Performans:_ Birden çok `SaveChanges()` yerine tek çağrı.
        
    - _Daha Net Akış:_ Service katmanındaki operasyonlar mantıksal bir birim olarak gruplanır.
        

### 2.2 `IUnitOfWork` Arayüzü

```csharp
// backend/BasarMapApp.Api/Repositories/IUnitOfWork.cs
using BasarMapApp.Api.Models;

namespace BasarMapApp.Api.Repositories
{
    public interface IUnitOfWork : IDisposable
    {
        IGenericRepository<MapPoint>          MapPoints  { get; }
        IGenericRepository<LineFeature>       Lines      { get; }
        IGenericRepository<PolygonFeature>    Polygons   { get; }
        // Değişiklikleri veritabanına tek seferde uygula
        Task<int> CompleteAsync();
    }
}
```

### 2.3 `UnitOfWork` Uygulaması

```csharp
// backend/BasarMapApp.Api/Repositories/UnitOfWork.cs
using BasarMapApp.Api.Data;
using BasarMapApp.Api.Models;

namespace BasarMapApp.Api.Repositories
{
    public class UnitOfWork : IUnitOfWork
    {
        private readonly AppDbContext _context;

        // Her entity için generic repository örneği
        public IGenericRepository<MapPoint>       MapPoints { get; }
        public IGenericRepository<LineFeature>    Lines     { get; }
        public IGenericRepository<PolygonFeature> Polygons  { get; }

        public UnitOfWork(AppDbContext context)
        {
            _context   = context;
            MapPoints  = new GenericRepository<MapPoint>(_context);
            Lines      = new GenericRepository<LineFeature>(_context);
            Polygons   = new GenericRepository<PolygonFeature>(_context);
        }

        public async Task<int> CompleteAsync() =>
            await _context.SaveChangesAsync();

        // IDisposable implementasyonu
        public void Dispose() =>
            _context.Dispose();
    }
}
```

- **Nasıl Çalışır?**
    
    - `UnitOfWork` constructor’ında tüm repository’leri yaratır.
        
    - `CompleteAsync()` çağrıldığında `SaveChangesAsync()` ile tüm değişiklikler tek bir transaction’da uygulanır.
        

---

## 3. Service Katmanında Kullanım

```csharp
// backend/BasarMapApp.Api/Services/MapPointService.cs
using BasarMapApp.Api.Models;
using BasarMapApp.Api.Repositories;

namespace BasarMapApp.Api.Services
{
    public class MapPointService
    {
        private readonly IUnitOfWork _uow;

        public MapPointService(IUnitOfWork uow) =>
            _uow = uow;

        public async Task<IEnumerable<MapPoint>> GetAllAsync() =>
            await _uow.MapPoints.GetAllAsync();

        public async Task<MapPoint?> GetByIdAsync(int id) =>
            await _uow.MapPoints.GetByIdAsync(id);

        public async Task<MapPoint> CreateAsync(MapPoint point)
        {
            point.LastUpdated = DateTime.UtcNow;
            await _uow.MapPoints.AddAsync(point);
            await _uow.CompleteAsync();
            return point;
        }

        public async Task UpdateAsync(MapPoint point)
        {
            point.LastUpdated = DateTime.UtcNow;
            _uow.MapPoints.Update(point);
            await _uow.CompleteAsync();
        }

        public async Task DeleteAsync(int id)
        {
            var point = await _uow.MapPoints.GetByIdAsync(id);
            if (point == null) return;
            _uow.MapPoints.Remove(point);
            await _uow.CompleteAsync();
        }
    }
}
```

---

## 4. Dependency Injection (DI) Kayıtları

Program.cs içinde DI container’a ekleyin:

```csharp
builder.Services.AddScoped(typeof(IGenericRepository<>), typeof(GenericRepository<>));
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<MapPointService>();
```

- **Sebep:** Scoped ömür (her HTTP isteği için yeni) hem performance hem de bellek yönetimi açısından idealdir.
    
- **Sonuç:** Controller’lar ve diğer servisler `IUnitOfWork` veya `MapPointService` aracılığıyla repository’lere ulaşır.
    

---

**Bir Sonraki Adım:**  
Bu derste Repository ve Unit of Work desenlerini öğrendik.  
Sonraki dosyamız **07_HaritaApi_CRUD_Haritabilgileri.md** ile controller seviyesinde CRUD endpoint’leri ve DTO mapping’e geçeceğiz.

