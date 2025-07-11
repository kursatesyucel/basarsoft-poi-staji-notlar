Aşağıda, proje frontend’inin genel mimarisini, kullandığı teknolojileri ve her bir temel parçanın ne işe yaradığını “ne, neden, nasıl” ekseninde detaylı biçimde özetledim.

---

## 1. Proje Temeli: Vite + React

- **Vite**
    
    - **Ne?** Modern, hızlı bir geliştirme sunucusu ve bundler.
        
    - **Neden?** Soğuk başlatma süreleri (cold start) çok kısa, modül tabanlı HMR (Hot Module Replacement) hızlı.
        
    - **Nasıl?** `npm create vite@latest … --template react` ile kurulur; `npm run dev` ile geliştirme sunucusu ayağa kalkar.
        
- **React**
    
    - **Ne?** Bileşen (component) temelli UI kütüphanesi.
        
    - **Neden?** Tekrar kullanılabilir bileşenler, sanal DOM ile hızlı render, zengin ekosistem.
        
    - **Nasıl?** Fonksiyonel bileşen + Hook’lar (useState, useEffect vb.) ile yapılandırılır.
        

---

## 2. Proje Dizin Yapısı

```
src/
│
├─ assets/              # Statik dosyalar (resimler, ikonlar, stil dosyaları)
│
├─ components/          # Tek bir sorumluluğu olan UI bileşenleri
│   ├─ MapView.jsx      # Leaflet harita bileşeni
│   ├─ MarkerList.jsx   # Harita üzerindeki marker’ları listeleyen bileşen
│   └─ …                
│
├─ contexts/            # Global state yönetimi (Context API)
│   └─ SelectedItemContext.jsx
│
├─ hooks/               # Projeye özel yeniden kullanılabilir Hook’lar
│   ├─ useFetch.js      # Axios ile API çağrısı yapan Hook
│   └─ …
│
├─ pages/               # Uygulamanın sayfaları (ana view’lar)
│   ├─ HomePage.jsx     
│   └─ DetailPage.jsx   
│
├─ services/            # API istemcisi ve CRUD işlemleri
│   └─ apiClient.js     # Axios, baseURL ve CORS ayarları
│
├─ utils/               # Ortak yardımcı fonksiyonlar (formatting, validation vb.)
│   └─ calculateArea.js 
│
├─ App.jsx              # Uygulamanın en tepe bileşeni (Router, Context provider)
└─ main.jsx             # Giriş dosyası (Leaflet CSS import, ReactDOM.render)
```

---

## 3. UI Mimarisi ve Bileşen İlişkileri

1. **App.jsx**
    
    - Tüm Context sağlayıcılarını (`<SelectedItemProvider>`) sarar.
        
    - React Router ile sayfalar arası geçişi yönetir.
        
2. **Pages**
    
    - **HomePage:** Haritanın ve yan panelin bulunduğu ana görünüm.
        
    - **DetailPage:** Bir marker/alan seçildiğinde detaylı bilgileri gösterir.
        
3. **Components**
    
    - **MapView:** Leaflet’in `MapContainer`, `TileLayer`, `Marker`, `Polygon` gibi bileşenlerini içerir.
        
        - **Neden Component?** Haritanın kendine ait karmaşık DOM yapısı izole edilsin, farklı parametrelerle yeniden kullanılabilsin.
            
    - **MarkerList / AreaList:** Seçilen datayı listeler, listeden tıklanınca Context üzerinden haritaya odaklanır.
        
4. **Context API**
    
    - **SelectedItemContext:**
        
        - **Ne?** Seçili obje bilgisini (örneğin tıklanan marker ID’si) global state’de tutan context.
            
        - **Neden?** Props drilling’den kaçınmak ve farklı bileşenlerin aynı “seçim” bilgisini kullanması için.
            
        - **Nasıl?** `createContext` + `useReducer` (ya da `useState`) + `Context.Provider`.
            

---

## 4. Durum (State) Yönetimi

- **Local State (useState)**
    
    - Bileşenin kendi içsel durumu için (örn. modal açık/kapalı, form alanı değerleri).
        
- **Side‐effect Yönetimi (useEffect)**
    
    - Bileşen mount edildiğinde API’den veriyi çekmek, cleanup işlemleri (event listener kaldırma) vb.
        
- **Global State (Context API)**
    
    - Uygulama genelinde paylaşılması gereken minimal state’ler (seçili obje, kullanıcı tercihleri vb.).
        
- **Neden Bu Kombinasyon?**
    
    - Local state’ler basit ve performanslı, global state’ler ise sadece gerçekten ihtiyaç duyulan yerlerde kullanılarak render karmaşıklığı azaltılır.
        

---

## 5. Leaflet Entegrasyonu

1. **Kurulum ve Stil**
    
    - `npm install leaflet react-leaflet`
        
    - `import 'leaflet/dist/leaflet.css'`
        
2. **MapContainer Yapılandırması**
    
    - Koordinat, zoom seviyesi, min/max zoom, scroll wheel kontrolü.
        
    - Inline `style={{ height: '100%', width: '100%' }}` veya CSS sınıfı.
        
3. **Katmanlar ve Katman Kontrolleri**
    
    - `TileLayer` (OSM, Satellite vb.).
        
    - İhtiyaca göre `LayersControl` ile kullanıcının katman seçmesini sağlamak.
        
4. **Geometriler**
    
    - **Marker / Popup:** Bilgi baloncuğu.
        
    - **Polyline / Polygon:** Çizgi ve alan gösterimi; `color`, `weight`, `fillOpacity` gibi stil ayarları.
        
    - **GeoJSON:** Harici JSON verilerini doğrudan haritaya bindirme.
        
5. **Etkileşim**
    
    - `onClick`, `onMouseOver` event’leri ile marker/alan seçimleri.
        
    - Seçim sonrası Context güncelleme + yan panel yönlendirme.
        

---

## 6. API Çağrıları ve Veri Akışı

- **axios + apiClient.js**
    
    - `baseURL` (örn. `https://localhost:5001/api`)
        
    - **CORS Ayarları:** Frontend’den gelen isteklerin backend tarafından kabulü için:
        
        ```csharp
        // Startup.cs (ConfigureServices)
        services.AddCors(options => 
          options.AddPolicy("AllowReactApp",
            builder => builder
              .WithOrigins("http://localhost:5173")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials()
          )
        );
        // Configure:
        app.UseCors("AllowReactApp");
        ```
        
- **CRUD İşlemleri**
    
    - `GET /points` → useFetch ile harita verisi çekme
        
    - `POST /points` → formdan ekleme
        
    - `PUT /points/:id` → güncelleme
        
    - `DELETE /points/:id` → silme
        
- **Gerçek Zamanlı Güncellemeler (SignalR)**
    
    - Yatay ölçeklenebilirlik için: yeni ekleme/güncelleme geldiğinde tüm açık istemcilerde UI’yı otomatik yenile.
        

---

## 7. Stil ve Responsive Tasarım

- **CSS-in-JS / CSS Modülleri / Tailwind (isteğe göre)**
    
- **Mobil Düzen**
    
    - Harita ve yan panel arasında kırılma noktaları (breakpoints) tanımlamak.
        
    - Panel “drawer” şeklinde mobilde açılır-kapanır olabilir.
        
- **Tema Desteği (Dark/Light)**
    
    - Context ile tema toggle, stil değişimleri.
        

---

### Özetle

Bu frontend:

1. **Hızlı**: Vite + React
    
2. **Modüler**: Bileşen bazlı yapı
    
3. **Esnek**: Context API + Hook’lar
    
4. **Etkileşimli**: Leaflet map + UI event’leri
    
5. **Sağlam**: CORS & .NET backend ile güvenli entegrasyon
    
6. **Gerçek zamanlı**: SignalR ile anlık güncelleme
    

Bu katmanlı yaklaşım hem geliştirirken hem de ileride bakım ve yeni özellik eklerken projeyi sürdürülebilir kılıyor.