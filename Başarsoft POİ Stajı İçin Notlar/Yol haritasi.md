## CS Notları/

├── **Backend/**  
│ ├─ **01_CSharp_Temelleri.md**  
│ │ - C# ve .NET CLR mimarisi  
│ │ - Değişkenler, kontrol yapıları, döngüler  
│ │ - Metotlar, parametre geçişleri (ref, out), exception handling  
│ │ - IDisposable ve `using` örnekleri  
│ │  
│ ├─ **02_OOP_SOLID_Generic_Mimari.md**  
│ │ - Sınıf, obje, encapsulation, inheritance, polymorphism  
│ │ - Interface vs abstract sınıf  
│ │ - SOLID prensipleri tek tek (S, O, L, I, D) örneklerle  
│ │ - Generic mimari nedir, neden kullanılır, kodla örnek  
│ │ - Gerçek proje senaryosunda katmanlı yapı (Domain, Application, Infrastructure)  
│ │  
│ ├─ **03_DotNetCore_WebAPI_DI_Middleware.md**  
│ │ - .NET Core mimarisi & CLI  
│ │ - Dependency Injection temelleri ve `Startup/Program` içi servis kaydı  
│ │ - Middleware kavramı, custom middleware örneği  
│ │ - Controller/Action model binding, routing stratejileri  
│ │  
│ ├─ **04_EFCore_Temelleri.md**  
│ │ - EF Core kurulumu (NuGet paketleri)  
│ │ - DbContext, DbSet, migration  
│ │ - Basit CRUD örnekleri (Console App üzerinden)  
│ │ - LINQ to Entities vs LINQ to Objects  
│ │  
│ ├─ **05_EFCore_Spatial_PostGIS.md**  
│ │ - `Npgsql.EntityFrameworkCore.PostgreSQL` + NetTopologySuite  
│ │ - Spatial tip eşleştirmeleri (Point, LineString, Polygon)  
│ │ - Migration ile geometry sütunu oluşturma  
│ │ - CRUD örnekleri spatial property’lerle  
│ │  
│ ├─ **06_Repository_UnitOfWork_Pattern.md**  
│ │ - Repository pattern kavramı ve implementasyonu  
│ │ - Unit of Work pattern nedir, nasıl entegre edilir  
│ │ - Generic repository sınıfı tasarımı  
│ │ - Örnek: IGenericRepository<TEntity>, IUnitOfWork  
│ │  
│ └─ **07_HaritaApi_CRUD_Haritabilgileri.md**  
│ - Point/LineString/Polygon DTO ⇄ Entity mapping  
│ - Controller’da CRUD endpoint’leri (`/api/points`, `/api/lines`, `/api/polygons`)  
│ - Swagger/OpenAPI entegrasyonu  
│ - Exception & central error handling middleware

├── **Database/**  
│ ├─ **01_PostgreSQL_Basics.md**  
│ │ - PostgreSQL kurulumu ve psql kullanımı  
│ │ - Temel DDL/DML: CREATE/ALTER/INSERT/SELECT/UPDATE/DELETE  
│ │ - Transaction, index, constraint, view  
│ │  
│ ├─ **02_PostGIS_Giris_Uzanti.md**  
│ │ - `CREATE EXTENSION postgis;`  
│ │ - Geography vs Geometry tipleri  
│ │ - SRID kavramı ve yaygın projeksiyonlar  
│ │  
│ ├─ **03_Spatial_Fonksiyonlar.md**  
│ │ - ST_AsText, ST_AsGeoJSON, ST_Distance, ST_Intersects, ST_Within…  
│ │ - Örnek spatial sorgular (yakındaki objeleri bulma, kapsayan poligonlar)  
│ │  
│ └─ **04_Sql_Ornekler_Crud_Spatial.md**  
│ - Nokta ekleme/güncelleme/silme SQL örnekleri  
│ - Spatial index oluşturma ve performans notları

├── **Frontend/**  
│ ├─ **01_Vite_ile_Projeyi_Baslatma.md**  
│ │ - Vite nedir, neden hızlı  
│ │ - Proje oluşturma (`npm create vite@latest`)  
│ │ - ESModule, HMR mekanizması  
│ │  
│ ├─ **02_React_Temelleri_State_Props.md**  
│ │ - Bileşen tipleri (function vs class)  
│ │ - JSX, props, state  
│ │ - Hook’lara giriş: useState, useEffect  
│ │  
│ ├─ **03_Leaflet_Temelleri_Map_Marker.md**  
│ │ - Leaflet kurulumu ve CSS/JS importu  
│ │ - Map objesi oluşturma  
│ │ - Marker, Polyline, Polygon ekleme ve stil verme  
│ │ - Popup/Tooltip kullanımı  
│ │  
│ ├─ **04_Context_API_Veya_BasitStateMgmt.md**  
│ │ - Global state ihtiyacı ve Context API  
│ │ - Örnek: seçili objeyi tüm bileşenlerde yönetme  
│ │  
│ └─ **05_Frontend_Backend_Entegrasyon.md**  
│ - Axios ile CRUD çağrıları  
│ - Harita üzerinde ekleme/güncelleme/silme akışı  
│ - Gerçek zamanlı UI güncellemeleri

└── **Genel/**  
├─ **01_Docker_docker-compose.md** (isteğe bağlı)  
├─ **02_CI_CD_GithubActions.md** (isteğe bağlı)  
└─ **03_BestPractices_CodeStyle.md**