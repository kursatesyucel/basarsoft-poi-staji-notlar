# 05_Frontend & .NET Backend Entegrasyonu

Bu dokümanda modern bir React + Leaflet ön yüzü ile **ASP.NET Core Web API** arka ucu arasındaki tam entegrasyon akışını ele alacağız. İçerikler:

- CORS (Cross-Origin Resource Sharing) detaylı konfigürasyonu (.NET Core)
    
- Axios ile CRUD (Create, Read, Update, Delete) çağrıları
    
- Harita üzerinde ekleme, güncelleme, silme akışı
    
- Gerçek zamanlı UI güncellemeleri (optimistic updates ve SignalR kullanımı)
    

Her adımda **sebep**, **amaç** ve **sonuç** odaklı açıklamalar yer alacaktır.

---

## 1. CORS Detaylı Konfigürasyonu (.NET Core)

### 1.1 CORS Nedir?

- **Sebep:** Tarayıcı güvenlik kısıtlamaları (same-origin policy), farklı kaynaklardan gelen istekleri engeller.
    
- **Amaç:** React uygulamanızın farklı porta (`https://localhost:5173`) veya domaine (`https://app.example.com`) konuşlanan .NET API ile sorunsuz iletişim kurmasını sağlamak.
    
- **Sonuç:** Backend, uygun başlıkları (`Access-Control-Allow-Origin`) ekleyerek istekleri kabul eder.
    

### 1.2 ASP.NET Core'da CORS Ayarı

`Program.cs` veya `Startup.cs`:

```csharp
// Program.cs (.NET 6+ minimal hosting modeli)
var builder = WebApplication.CreateBuilder(args);

// CORS politikası tanımlama
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReactApp", policy =>
    {
        policy.WithOrigins("https://localhost:5173", "https://app.example.com")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

builder.Services.AddControllers();

var app = builder.Build();

// CORS middleware'i pipeline'a ekleme
app.UseCors("AllowReactApp");

app.MapControllers();
app.Run();
```

- **Sebep:** `UseCors` middleware ile CORS başlıklarını her HTTP yanıtına otomatik eklemek.
    
- **Amaç:** Frontend ve backend farklı origin’lerde olsa bile istekleri sorunsuz hale getirmek.
    
- **Sonuç:** Tarayıcıda `Blocked by CORS policy` hataları ortadan kalkar.
    

---

## 2. Axios ile CRUD Çağrıları

React tarafındaki Axios örnekleri, `.NET Web API` rotalarına göre uyarlanır.

### 2.1 Axios Konfigürasyonu

```js
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://localhost:5001/api', // .NET API URL
  withCredentials: true,                // Cookie veya token bazlı auth için
});

export default api;
```

### 2.2 GET (Read)

```js
api.get('/points')
  .then(res => setPoints(res.data))
  .catch(err => console.error(err));
```

- **Sebep:** API'deki mevcut veriyi çekmek.
    
- **Amaç:** UI state ile arka uç veritabanı eşitlemek.
    
- **Sonuç:** `points` state güncellenir, harita marker’ları render edilir.
    

### 2.3 POST (Create)

```js
const newPoint = { lat: 39.92, lng: 32.86, label: 'Yeni Nokta' };
api.post('/points', newPoint)
  .then(res => setPoints(prev => [...prev, res.data]))
  .catch(err => console.error(err));
```

- **Sebep:** Kullanıcı haritaya yeni bir nokta eklemek istediğinde.
    
- **Amaç:** API üzerinde yeni kayıt oluşturup, UI'ı anında güncellemek.
    
- **Sonuç:** Uygulama state'i ve harita marker'ları eş zamanlı olur.
    

### 2.4 PUT (Update)

```js
api.put(`/points/${id}`, { label: 'Güncel Etiket' })
  .then(res => {
    setPoints(prev => prev.map(p => p.id === id ? res.data : p));
  })
  .catch(err => console.error(err));
```

- **Sebep:** Marker özelliklerinde değişiklik.
    
- **Amaç:** Backend ve frontend arasında state senkronizasyonu sağlamak.
    
- **Sonuç:** Harita üzerindeki marker bilgileri güncellenir.
    

### 2.5 DELETE (Silme)

```js
api.delete(`/points/${id}`)
  .then(() => setPoints(prev => prev.filter(p => p.id !== id)))
  .catch(err => console.error(err));
```

- **Sebep:** Kullanıcı marker'ı kaldırmak istediğinde.
    
- **Amaç:** API üzerinden silme işlemi, UI'da anlık tepki.
    
- **Sonuç:** Harita ve state temizlenir.
    

---

## 3. Harita Üzerinde Ekleme/Güncelleme/Silme Akışı

1. **Kullanıcı Etkileşimi:** Harita tıklaması veya form ile "ekle/düzenle/sil" aksiyonu tetiklenir.
    
2. **Optimistic Update:** Önce UI `setPoints` ile güncellenir; kullanıcı beklemeden görür.
    
3. **API Çağrısı:** `api.post/put/delete` çağrısı yapılır.
    
4. **Sonuç Kontrolü:** Başarıysa güncelleme kalıcılaşır; hata durumunda orijinal state’e geri dönülür (_rollback_).
    
5. **Harita Yenileme:** React-Leaflet bileşenleri, `points` prop’u değişimine tepki vererek render eder.
    

---

## 4. Gerçek Zamanlı UI Güncellemeleri (SignalR)

### 4.1 ASP.NET Core SignalR Ayarları

`Program.cs` içinde:

```csharp
builder.Services.AddSignalR();

app.MapHub<PointsHub>("/hubs/points");
```

```csharp
// Hubs/PointsHub.cs
authorize
using Microsoft.AspNetCore.SignalR;
public class PointsHub : Hub
{
    public async Task BroadcastPointCreated(PointDto p) =>
        await Clients.All.SendAsync("PointCreated", p);
    public async Task BroadcastPointUpdated(PointDto p) =>
        await Clients.All.SendAsync("PointUpdated", p);
    public async Task BroadcastPointDeleted(int id) =>
        await Clients.All.SendAsync("PointDeleted", id);
}
```

### 4.2 React SignalR Client Entegrasyonu

```js
import { HubConnectionBuilder } from '@microsoft/signalr';

useEffect(() => {
  const connection = new HubConnectionBuilder()
    .withUrl('https://localhost:5001/hubs/points', { withCredentials: true })
    .withAutomaticReconnect()
    .build();

  connection.start();

  connection.on('PointCreated', point => setPoints(prev => [...prev, point]));
  connection.on('PointUpdated', updated => setPoints(prev => prev.map(p => p.id === updated.id ? updated : p)));
  connection.on('PointDeleted', id => setPoints(prev => prev.filter(p => p.id !== id)));

  return () => connection.stop();
}, []);
```

- **Sebep:** Çok kullanıcılı ortamlarda değişikliklerin anlık yansıması.
    
- **Amaç:** Kullanıcılar arasında gerçek zamanlı senkronizasyon sağlamak.
    
- **Sonuç:** Harita ve liste bileşenleri, diğer kullanıcıların yaptığı ekleme/güncelleme/silme işlemlerini otomatik alır.
    

---

> Bu doküman ile React + Leaflet ön yüzü ve ASP.NET Core Web API & SignalR tabanlı arka uç arasındaki tüm entegrasyonu öğrendiniz. Sonraki bölümde performans optimizasyonları, lazy loading ve harita eklentileri (cluster, heatmap) konularına odaklanacağız.