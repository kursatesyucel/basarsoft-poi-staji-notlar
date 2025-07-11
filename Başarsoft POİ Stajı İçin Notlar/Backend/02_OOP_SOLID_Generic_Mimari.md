## 02_OOP_SOLID_Generic_Mimari

### Giriş

Bu bölümde nesne yönelimli programlamanın temel kavramlarını, SOLID prensiplerini ve gerçek dünya projelerinde kullanabileceğimiz generic mimari yapısını öğreneceğiz. Kod örneklerimiz, bir harita uygulaması senaryosuna uyarlanmış olacak.


---

### 1.1 Sınıf (Class) ve Nesne (Object)

**Neden?**

- Gerçek dünyadaki varlıkları (örneğin bir harita üzerindeki nokta) kod dünyasında temsil etmek için
    
- İlgili veri ve davranışları (metotları) bir arada tutarak modülerlik, okunabilirlik ve yeniden kullanım sağlar
    

```csharp
// 1. MapPoint sınıfı: harita üzerindeki bir noktanın tüm özelliklerini taşır
public class MapPoint
{
    // Alanlar/özellikler
    public int Id { get; set; }                    // Benzersiz kimlik
    public string Name { get; set; }               // Noktanın etiketi
    public double X { get; set; }                  // X koordinatı
    public double Y { get; set; }                  // Y koordinatı
    public DateTime LastUpdated { get; set; }      // Son güncellenme zamanı

    // Davranış / metot
    public string ToGeoJson()
    {
        return $"{{ \"type\": \"Point\", \"coordinates\": [{X}, {Y}] }}";
    }
}

// 2. MapPoint nesnesi oluşturma ve kullanımı
var school = new MapPoint
{
    Id = 1,
    Name = "Okul",
    X = 29.02345,
    Y = 41.01514,
    LastUpdated = DateTime.UtcNow
};

Console.WriteLine($"Nokta: {school.Name}, GeoJSON: {school.ToGeoJson()}");
```

---

### 1.2 Encapsulation (Kapsülleme)

**Neden?**

- Nesnenin iç durumunu (`X`, `Y`, `LastUpdated`) doğrudan dışarıya açmak yerine, kontrol altında tutar.
    
- Geçersiz veya tutarsız veri girişini engeller; değişiklikleri tek bir noktada validasyonla yönetir.
    

```csharp
public class MapPoint
{
    private double _x;
    private double _y;
    private DateTime _lastUpdated;

    // X koordinatını alırken/düzenlerken doğrulama ekleyebiliriz
    public double X
    {
        get => _x;
        set
        {
            if (value < -180 || value > 180)
                throw new ArgumentOutOfRangeException(nameof(X), "X koordinatı geçerli bir boylam olmalı.");
            _x = value;
            UpdateTimestamp();
        }
    }

    public double Y
    {
        get => _y;
        set
        {
            if (value < -90 || value > 90)
                throw new ArgumentOutOfRangeException(nameof(Y), "Y koordinatı geçerli bir enlem olmalı.");
            _y = value;
            UpdateTimestamp();
        }
    }

    public DateTime LastUpdated
    {
        get => _lastUpdated;
        private set => _lastUpdated = value;
    }

    // Her değişiklikte zaman damgasını güncelleyen yardımcı metot
    private void UpdateTimestamp()
    {
        LastUpdated = DateTime.UtcNow;
    }
}
```

- `private` alan ve `public` property ayrımı sayesinde, sadece `X` veya `Y`’ye doğru değerler atanır ve her atama `LastUpdated`’i otomatik yeniler.
    

---

### 1.3 Inheritance (Kalıtım)

**Neden?**

- Ortak özellikleri ve davranışları temel bir sınıfta toplar; tekrar yazımı önler.
    
- Yeni türler oluştururken sadece farklılıkları belirtir, kalan kodu miras alır.
    

```csharp
// 1. Base sınıf: Harita üzerindeki tüm geometrik objeler için ortak alanlar
public abstract class MapFeature
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTime LastUpdated { get; set; }

    // Türetilen sınıflar bu metodu kendine göre uygular
    public abstract string ToGeoJson();
}

// 2. Nokta (Point) sınıfı: MapFeature'den kalıtılır
public class PointFeature : MapFeature
{
    public double X { get; set; }
    public double Y { get; set; }

    public override string ToGeoJson()
    {
        return $"{{ \"type\":\"Point\",\"coordinates\":[{X},{Y}] }}";
    }
}

// 3. Çizgi (LineString) sınıfı örneği
public class LineFeature : MapFeature
{
    public List<(double X, double Y)> Coordinates { get; set; }

    public override string ToGeoJson()
    {
        var coords = string.Join(",", Coordinates.Select(c => $"[{c.X},{c.Y}]"));
        return $"{{ \"type\":\"LineString\",\"coordinates\":[{coords}] }}";
    }
}
```

- `MapFeature` içerisinde ortak `Id`, `Name`, `LastUpdated` tutuluyor; her yeni geometri sınıfı sadece kendi verisini ve `ToGeoJson()` implementasyonunu yazar.
    

---

### 1.4 Polymorphism (Çok Biçimlilik)

**Neden?**

- Farklı tipteki nesneleri, tek bir referans türü üzerinden işleyebilme esnekliği sağlar.
    
- Yeni geometri türleri eklediğinde mevcut kodu bozmadan genişletebilirsin.
    

```csharp
// 1. Farklı feature tiplerini bir listede tutup ortak işlem yapalım
var features = new List<MapFeature>
{
    new PointFeature { Id = 1, Name = "P1", X = 29.0, Y = 41.0, LastUpdated = DateTime.UtcNow },
    new LineFeature
    {
        Id = 2,
        Name = "L1",
        Coordinates = new List<(double X, double Y)> { (29.0,41.0),(29.1,41.1) },
        LastUpdated = DateTime.UtcNow
    }
};

// 2. Hepsini GeoJSON’a çevirip yazdıralım
foreach (var feat in features)
{
    // Burada hangi tip olduğunu kontrol etmeden, override edilmiş metot çalışır
    Console.WriteLine(feat.ToGeoJson());
}
```

- Üst tip (`MapFeature`) üzerinden çağrı yapıyoruz; alttaki `PointFeature` veya `LineFeature`’ın kendi `ToGeoJson()` yöntemini çağırıyor.
    

---

#### 2.1 Interface Nedir ve Neden Kullanılır?

- **Ne işe yarar?**
    
    - Sadece metot imzalarını (signature) tanımlar, gövdesi yoktur.
        
    - Çoklu kalıtıma (multiple inheritance) izin verir; bir sınıf birden fazla interface’i implemente edebilir.
        
    - Sözleşme (contract) mantığı getirir: “Bu sınıfın mutlaka şu metotları içermesi gerekir.”
        
- **Neden?**
    
    - Farklı katmanlar arası bağımlılığı soyutlamak için
        
    - Unit test yazarken “mock” veya “stub” yaratmayı kolaylaştırmak için
        
    - Uygulama içinde değişebilir iş kurallarını birbirinden ayrıştırmak için
        

```csharp
// 1. Nokta verisini okuma/yazma sözleşmesi
public interface IMapPointRepository
{
    MapPoint GetById(int id);
    IEnumerable<MapPoint> GetAll();
    void Add(MapPoint point);
    void Update(MapPoint point);
    void Delete(int id);
}
```

---

#### 2.2 Abstract Sınıf Nedir ve Neden Kullanılır?

- **Ne işe yarar?**
    
    - Hem gövdesi (implementasyonu) hem de metot imzaları barındırabilir.
        
    - Bir temel sınıf olarak ortak kodu bir kere yazıp, alt sınıflarda paylaşmanı sağlar.
        
    - Tekli kalıtım (single inheritance) kısıtlaması vardır: bir sınıf sadece bir abstract sınıftan kalıtım alabilir.
        
- **Neden?**
    
    - Ortak davranış veya yardımcı metotları (logging, validation vb.) somut sınıflara tekrar etmek yerine merkezi bir yerde toplamak için
        
    - Alt sınıfların mutlaka uygulaması gereken metotları `abstract` olarak tanımlayıp, gövdesini alt sınıfa bırakmak için
        

```csharp
// 1. Tüm veri erişim katmanlarında ortak loglama/finalize işlerini barındıran temel sınıf
public abstract class BaseRepository
{
    protected readonly AppDbContext _context;

    public BaseRepository(AppDbContext context)
    {
        _context = context;
    }

    // Ortak loglama
    protected void Log(string message)
    {
        Console.WriteLine($"[RepoLog {DateTime.UtcNow:O}] {message}");
    }

    // Alt sınıfların mutlaka kendi sorgularını yazacağı metot
    public abstract void SaveChanges();
}
```

---

#### 2.3 Interface ile Abstract Arasındaki Farklar

|Özellik|Interface|Abstract Sınıf|
|---|---|---|
|Metot gövdesi|Yok (C# 8.0 öncesi)Varsa default|Var|
|Çoklu kalıtım|Evet|Hayır|
|Alan (field) tanımlama|Hayır|Evet|
|Yapılandırıcı (ctor)|Yok|Evet|
|Kullanım amacı|Sözleşme (contract) oluşturmak|Ortak kod ve davranış paylaşımı|

---

#### 2.4 Örnek Senaryo: MapPointRepository ve BaseService

```csharp
// Domain katmanında: MapPoint modeli
public class MapPoint
{
    public int Id { get; set; }
    public string Name { get; set; }
    public double X { get; set; }
    public double Y { get; set; }
    public DateTime LastUpdated { get; set; }
}

// Infrastructure katmanında: Interface tanımı
public interface IMapPointRepository
{
    MapPoint GetById(int id);
    IEnumerable<MapPoint> GetAll();
    void Add(MapPoint point);
    void Update(MapPoint point);
    void Delete(int id);
}

// Infrastructure katmanında: Abstract temel sınıf
public abstract class BaseRepository
{
    protected readonly AppDbContext _context;
    protected BaseRepository(AppDbContext context) => _context = context;

    protected void Log(string action)
    {
        Console.WriteLine($"[RepoLog] {action} at {DateTime.UtcNow:O}");
    }

    public abstract void SaveChanges();
}

// Infrastructure katmanında: Somut repository
public class MapPointRepository : BaseRepository, IMapPointRepository
{
    public MapPointRepository(AppDbContext context) : base(context) { }

    public MapPoint GetById(int id)
    {
        Log($"GetById({id})");
        return _context.MapPoints.Find(id);
    }

    public IEnumerable<MapPoint> GetAll()
    {
        Log("GetAll()");
        return _context.MapPoints.ToList();
    }

    public void Add(MapPoint point)
    {
        Log($"Add({point.Name})");
        _context.MapPoints.Add(point);
    }

    public void Update(MapPoint point)
    {
        Log($"Update({point.Id})");
        _context.MapPoints.Update(point);
    }

    public void Delete(int id)
    {
        Log($"Delete({id})");
        var entity = _context.MapPoints.Find(id);
        if (entity != null) _context.MapPoints.Remove(entity);
    }

    public override void SaveChanges()
    {
        _context.SaveChanges();
        Log("SaveChanges()");
    }
}
```


---

## 3. SOLID Prensipleri

Bu bölümde SOLID prensiplerini tek tek detaylıca ele alacağız. Her bir ilkeyi **neden** kullanmamız gerektiğini ve **MapPoint** nesnemiz üzerinden nasıl uygulayacağımızı gerçek dünya örnekleriyle inceleyeceğiz.

### 3.1 Single Responsibility Principle (SRP)

**Tanım:** Bir sınıfın veya modülün yalnızca bir sorumluluğu olmalı.

**Neden?**

- Sınıflar küçük ve odaklı olduğunda test yazmak ve bakımı kolaylaşır.
    
- Değişik sebeplerle yapılması gereken değişiklikler birbirini etkilemez.
    

**MapPoint Örneği:**

```csharp
// 1. Veri erişim sorumluluğu: Veritabanından/nokta listesinden okur
public class MapPointRepository
{
    public MapPoint GetById(int id) { /* EF Core ile DB erişimi */ }
    public void Save(MapPoint point) { /* DB yazma işlemi */ }
}

// 2. Validasyon sorumluluğu: Harita noktasının koordinatlarını doğrular
public class MapPointValidator
{
    public bool IsValid(MapPoint point)
    {
        return point.X >= -180 && point.X <= 180
            && point.Y >= -90  && point.Y <= 90;
    }
}

// 3. Formatlama sorumluluğu: MapPoint objesini GeoJSON olarak döndürür
public class MapPointFormatter
{
    public string ToGeoJson(MapPoint point)
    {
        return $"{ { \"type\": \"Point\", \"coordinates\": [{point.X}, {point.Y}] }}";
    }
}
```

### 3.2 Open/Closed Principle (OCP)

**Tanım:** Sınıflar yeni işlevsellik eklemek için **açık**, mevcut kodu **değiştirmek** yerine **genişletmek** için kapalı olmalı.

**Neden?**

- Var olan kodu değiştirmeden yeni özellik eklemek, regresyon riskini azaltır.
    
- Uzun vadede kod stabilitesi ve genişletilebilirlik artar.
    

**MapPoint Örneği:**

```csharp
// 1. Temel kontrol arayüzü
public interface IMapPointRule
{
    bool Check(MapPoint point);
}

// 2. Mevcut sınıfı değiştirmeden yeni kural ekliyoruz
public class MaxDistanceRule : IMapPointRule
{
    private readonly double _maxDistance;
    public MaxDistanceRule(double maxDistance) { _maxDistance = maxDistance; }
    public bool Check(MapPoint point)
    {
        // (0,0) noktasına uzaklık kontrolü örneği
        var distance = Math.Sqrt(point.X*point.X + point.Y*point.Y);
        return distance <= _maxDistance;
    }
}
```

### 3.3 Liskov Substitution Principle (LSP)

**Tanım:** Türetilmiş sınıflar, temel sınıfın yerine her yerde sorunsuz kullanılabilmeli.

**Neden?**

- Polimorfizmi güvenle kullanmamızı sağlar.
    
- Davranışta sürprizler yaşamadan alt sınıfları temel sınıf referansına atayabiliriz.
    

**MapPoint Örneği:**

```csharp
public abstract class BaseFeature
{
    public abstract MapPoint GetCenter();
}

public class PointFeature : BaseFeature
{
    public MapPoint Point { get; set; }
    public override MapPoint GetCenter() => Point;
}

public class PolygonFeature : BaseFeature
{
    public List<MapPoint> Vertices { get; set; }
    public override MapPoint GetCenter()
    {
        // Basit ortalama örneği
        var avgX = Vertices.Average(v => v.X);
        var avgY = Vertices.Average(v => v.Y);
        return new MapPoint { X = avgX, Y = avgY };
    }
}
```

### 3.4 Interface Segregation Principle (ISP)

**Tanım:** Kullanıcılar ihtiyaç duymadıkları metodları içeren büyük ve monolitik arayüzlerden kaçınmalı. Bunun yerine, işlevsel olarak ayrılmış küçük arayüzler tercih edilmeli.

**Neden?**

- Gereksiz bağımlılıkları ortadan kaldırır.
    
- Test ve bakım kolaylığı sağlar.
    

**MapPoint Örneği:**

```csharp
// Büyük bir repository yerine işlevsel parçalara böldük:
public interface IReadOnlyMapPointRepo
{
    MapPoint GetById(int id);
    IEnumerable<MapPoint> GetAll();
}
public interface IWritableMapPointRepo
{
    void Add(MapPoint point);
    void Update(MapPoint point);
    void Delete(int id);
}
// Dilediğimiz yerde sadece okuma veya yazma yetkisi vermek mümkün.
```

### 3.5 Dependency Inversion Principle (DIP)

**Tanım:** Yüksek seviye modüller düşük seviye modüllere değil, **abstraksiyonlara** (interface/abstract) bağımlı olmalı. Detaylar soyutlamalara bağlı olmalıdır.

**Neden?**

- Bağımlılıklar minimal düzeye iner, modüller arası sıkı bağlılık (tight coupling) azalır.
    
- Mock/testing ve modülerlik artar.
    

**MapPoint Örneği:**

```csharp
public class MapPointService
{
    private readonly IReadOnlyMapPointRepo _readRepo;
    private readonly IWritableMapPointRepo _writeRepo;

    public MapPointService(IReadOnlyMapPointRepo readRepo, IWritableMapPointRepo writeRepo)
    {
        _readRepo = readRepo;
        _writeRepo = writeRepo;
    }

    public void MovePoint(int id, double newX, double newY)
    {
        var point = _readRepo.GetById(id);
        point.X = newX;
        point.Y = newY;
        _writeRepo.Update(point);
    }
}
```

---

## 4. Generic Mimari ve Katmanlı Yapı

Bu kısımda **generic mimari** ve **katmanlı proje yapısı** kavramlarını neden ve nasıl kullanmamız gerektiğini detaylıca inceleyeceğiz. Kod örneklerini MapPoint senaryomuz üzerinden uyarlayacağız.

### 4.1 Neden Katmanlı Mimari?

- **Ayrık sorumluluk (Separation of Concerns):** Her katman yalnızca kendi görevine odaklanır.
    
- **Bakım ve Genişletilebilirlik:** Yeni işlev eklerken sadece ilgili katmanı güncelleriz.
    
- **Test Edilebilirlik:** Birim testler (unit test) için bağımlılıkları kolayca mock’layabiliriz.
    

#### Yaygın Katmanlar

1. **Domain (Core)**
    
    - Varlık modelleri (MapPoint, User vb.)
        
    - İş kuralları (domain services)
        
2. **Application**
    
    - İşlemleri koordine eden servisler (MapPointService vb.)
        
    - DTO (Data Transfer Object) tanımları
        
    - Interface tanımları (IMapPointRepository, IMapPointService)
        
3. **Infrastructure**
    
    - Veri erişim (EF Core, PostGIS uyarlaması)
        
    - Generic repository ve UnitOfWork implementasyonları
        
    - Dış sistem entegrasyonları (e‑posta, dosya sistemi vb.)
        
4. **Presentation (API)**
    
    - Web API katmanı: Controller’lar, request/response modelleri
        
    - Dependency Injection konfigürasyonu
        

### 4.2 Generic Repository ve Unit of Work Pattern

- **Generic Repository:** CRUD operasyonlarını tüm varlıklar için tek bir implementasyonla yönetir.
    
- **Unit of Work:** Bir iş birimi (transaction) içinde birden fazla repository çağrısını toplar, `SaveChanges()` tek seferde yapılır.
    

#### 4.2.1 Generic Repository Pattern

**Tanım ve Neden?**  
`Generic Repository` deseni, uygulamadaki her varlık (entity) için tekrar eden CRUD kodlarını tek bir sınıfta toplayarak kod tekrarını önler. Böylece:

- **Tek Sorumluluk:** CRUD işlemleri ayrı bir sınıfta odaklanır.
    
- **Yeniden Kullanılabilirlik:** Yeni bir varlık eklediğimizde, ekleme, güncelleme, silme metotlarını tekrar yazmak yerine generic sınıfı kullanırız.
    
- **Bakım Kolaylığı:** Hata veya optimizasyon gerektiğinde tek bir noktadan müdahale edilir.
    

**Adım 1: Interface Tanımı**

```csharp
public interface IGenericRepository<T> where T : class
{
    // Id ile nesne getirme
    T GetById(int id);
    // Tüm nesneleri listeleme
    IEnumerable<T> GetAll();
    // Yeni nesne ekleme
    void Add(T entity);
    // Mevcut nesneyi güncelleme
    void Update(T entity);
    // Nesne silme
    void Delete(T entity);
    // Koşula göre ilk kaydı getirme (isteğe bağlı genişletme)
    T GetFirstOrDefault(Func<T, bool> predicate);
}
```

**Adım 2: GenericRepository Sınıfı**

```csharp
public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;

    public GenericRepository(AppDbContext context)
    {
        _context = context;
        _dbSet = _context.Set<T>();
    }

    public T GetById(int id)
    {
        // EF Core Find: primary key üzerinden hızlı sorgu
        return _dbSet.Find(id);
    }

    public IEnumerable<T> GetAll()
    {
        // Tüm kayıtları belleğe yükler
        return _dbSet.ToList();
    }

    public void Add(T entity)
    {
        _dbSet.Add(entity);
    }

    public void Update(T entity)
    {
        _dbSet.Update(entity);
    }

    public void Delete(T entity)
    {
        _dbSet.Remove(entity);
    }

    public T GetFirstOrDefault(Func<T, bool> predicate)
    {
        // Belleğe yükleme riskine dikkat; Expression<Func<T,bool>> daha performanslı olabilir
        return _dbSet.FirstOrDefault(predicate);
    }
}
```

**Adım 3: MapPoint Örneği**

````csharp
// MapPointRepository yerine GenericRepository kullanımı
public interface IMapPointRepository : IGenericRepository<MapPoint>
{
    // Spatial sorgular eklenebilir
    IEnumerable<MapPoint> GetWithinDistance(double x, double y, double radius);
}

public class MapPointRepository : GenericRepository<MapPoint>, IMapPointRepository
{
    public MapPointRepository(AppDbContext context) : base(context) { }

    public IEnumerable<MapPoint> GetWithinDistance(double x, double y, double radius)
    {
        // NetTopologySuite kullanıyorsak:
        var center = new NetTopologySuite.Geometries.Point(x, y) { SRID = 4326 };
        return _dbSet
            .Where(p => p.Location.IsWithinDistance(center, radius))
            .ToList();
    }
}

> **Not:** Spatial sorgular için `Location` property’si geometry tipinde EF Core ile map edilmelidir.

#### Kod Örneği: IUnitOfWork & UnitOfWork

`Unit of Work` deseni, bir iş birimi (örneğin bir HTTP isteği veya bir iş akışı) içinde birden fazla veri erişim operasyonunu tek bir transaction çatısı altına toplar. Böylece uyumlu (consistent) bir veri durumu sağlanır: ya tüm değişiklikler başarılı bir şekilde işlenir, ya da bir hata durumunda geri alınır.

**Neden Kullanılır?**
- **Tutarlılık (Consistency):** Birden fazla repository çağrısını tek bir transaction’da yönetir.
- **Performans:** `SaveChanges()` çağrısını iş birimi sonunda bir kez yaparak veritabanı round-trip sayısını azaltır.
- **Atomicity:** İş akışının tamamı başarılı olmazsa, hiçbir değişiklik veritabanına uygulanmaz (rollback).

##### 1. UnitOfWork Interface’i Tanımlama
```csharp
public interface IUnitOfWork : IDisposable
{
    // MapPoint için generic repository
    IGenericRepository<MapPoint> MapPoints { get; }

    // Tüm birim işinin commit edildiği metot
    int Complete();
}
````

##### 2. UnitOfWork Sınıfının Uygulanması

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    // Repository örneğini kapsar
    public IGenericRepository<MapPoint> MapPoints { get; }

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        // Her varlık için tek bir GenericRepository oluşturulur
        MapPoints = new GenericRepository<MapPoint>(context);
    }

    // Değişiklikleri transaction içinde kaydeder
    public int Complete()
    {
        // Burada pre-commit validasyonları veya loglama eklenebilir
        return _context.SaveChanges();
    }

    // Kaynakları serbest bırakır
    public void Dispose()
    {
        _context.Dispose();
    }
}
```

##### 3. Kullanım Örneği: Service Katmanında UnitOfWork

```csharp
public class MapPointService
{
    private readonly IUnitOfWork _unitOfWork;

    public MapPointService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public void CreateMapPoint(MapPointDto dto)
    {
        // 1. MapPoint objesi oluşturulur
        var point = new MapPoint
        {
            Name = dto.Name,
            X = dto.X,
            Y = dto.Y,
            LastUpdated = DateTime.UtcNow
        };

        // 2. Repository üzerinden ekleme yapılır
        _unitOfWork.MapPoints.Add(point);

        // 3. Tüm değişiklikler tek bir transaction’da commit edilir
        _unitOfWork.Complete();
    }

    public void UpdateMapPoint(int id, double newX, double newY)
    {
        // 1. Mevcut nokta veritabanından alınır
        var point = _unitOfWork.MapPoints.GetById(id);

        // 2. Koordinatlar güncellenir
        point.X = newX;
        point.Y = newY;
        point.LastUpdated = DateTime.UtcNow;

        // 3. Repository aracılığıyla update işareti konur
        _unitOfWork.MapPoints.Update(point);

        // 4. Tüm işlemler commit edilir
        _unitOfWork.Complete();
    }
}
```

> **DI Entegrasyonu:**
> 
> ```csharp
> services.AddScoped<IUnitOfWork, UnitOfWork>();
> services.AddScoped(typeof(IGenericRepository<>), typeof(GenericRepository<>));
> ```

### 4.3 Katmanlı Mimari Akış

1. **Controller (API)**: İstek alır, Model Binding ile DTO’yu çözer.
    
2. **Application Service** (`MapPointService`): İş kurallarını uygular, Repository/UnitOfWork’e yönlendirir.
    
3. **Repository** (`GenericRepository<MapPoint>`): EF Core üzerinden DB ile etkileşir.
    
4. **DbContext**: Değişiklikleri takip eder, `SaveChanges()` ile transaction’ı tamamlar.
    

---

## 5. Gerçek Dünya Senaryosu: Harita Nesneleri Gerçek Dünya Senaryosu: Harita Nesneleri

```csharp
// Domain MapObject.cs
a public class MapObject
{
    public int Id { get; set; }
    public Geometry Location { get; set; } // NetTopologySuite
    public string Label { get; set; }
}

// Application IMapObjectService.cs
public interface IMapObjectService
{
    IEnumerable<MapObjectDto> GetAll();
    // ... diğer CRUD
}

// Infrastructure MapObjectRepository.cs
public class MapObjectRepository : GenericRepository<MapObject>, IMapObjectRepository
{
    public MapObjectRepository(AppDbContext ctx) : base(ctx) { }
    // Spatial sorgular burada genişletilebilir
}
```

---

