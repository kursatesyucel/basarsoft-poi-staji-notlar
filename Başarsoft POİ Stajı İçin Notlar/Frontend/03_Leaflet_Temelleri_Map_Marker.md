# 03_Leaflet Temelleri: Map & Marker

Bu dokümanda Leaflet kütüphanesinin temel kurulumundan başlayarak, harita nesnesi oluşturma, işaretçiler (Marker), çoklu çizgiler (Polyline) ve çokgenler (Polygon) ekleme ile stil verme ve son olarak Popup/Tooltip kullanımını **sebep**, **amaç** ve **sonuç** odaklı açıklamalarla ele alacağız.

---

## 1. Leaflet Kurulumu ve CSS/JS İmportu

Leaflet’i projeye dahil etmenin iki yolu vardır:

### 1.1 NPM ile Kurulum

```bash
npm install leaflet
```

- **Sebep:** Paket yöneticisi ile projeye bağımlılık olarak eklemek.
    
- **Amaç:** Versiyon kontrolü, ağaç sarsıntısı ve modüler import/dependency yönetimi.
    
- **Sonuç:** `import` kullanarak CSS ve JS dosyalarını doğrudan modüllerden çekebiliriz.
    

```js
// JS import (örn. main.js veya App.jsx)
import L from 'leaflet';
// CSS import
import 'leaflet/dist/leaflet.css';
```

### 1.2 CDN ile Dahil Etme

```html
<!-- CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<!-- JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
```

- **Sebep:** Hızlı prototip veya paket kurulumu istemeyen projeler.
    
- **Amaç:** Dosyaları doğrudan bir CDN’den çekmek.
    
- **Sonuç:** HTML içinde `<head>` ve `<body>` sonuna ekleyerek kullanıma hazır hale gelir.
    

---

## 2. Harita (Map) Objesi Oluşturma

Leaflet’te tüm harita işlemleri bir `L.map` nesnesi üzerinden başlar.

```js
// Bir <div id="map"></div> elementi olmalı
const map = L.map('map', {
  center: [39.925533, 32.866287], // Başlangıç koordinatı (Ankara)
  zoom: 13,
  zoomControl: true, // + ve - butonları
});

// OpenStreetMap katmanını ekleyelim
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OSM</a>',
  maxZoom: 19,
}).addTo(map);
```

- **Sebep:** Harita yüklemek ve katmanları yönetmek.
    
- **Amaç:** Kullanıcıya etkileşimli bir harita sunmak.
    
- **Sonuç:** HTML elementine entegre, pan ve zoom özellikli harita.
    

---

## 3. Marker, Polyline, Polygon Ekleme ve Stil Verme

### 3.1 Marker (İşaretçi)

```js
const marker = L.marker([39.925533, 32.866287], {
  draggable: true, // sürüklenebilir
  title: 'Başlangıç Noktası',
}).addTo(map);
```

- **Sebep:** Belirli koordinatları vurgulamak.
    
- **Amaç:** Kullanıcının harita üzerinde nokta seçimine izin vermek.
    
- **Sonuç:** Sürüklenebilir ya da sabit bir işaretçi haritada gözükür.
    

> **Özelleştirme:**
> 
> ```js
> const customIcon = L.icon({
>   iconUrl: 'path/to/icon.png',
>   iconSize: [25, 41],
>   iconAnchor: [12, 41],
> });
> L.marker([..., ...], { icon: customIcon }).addTo(map);
> ```

### 3.2 Polyline (Çoklu Çizgi)

```js
const line = L.polyline([
  [39.92, 32.86],
  [39.93, 32.87],
  [39.94, 32.88],
], {
  color: 'blue',
  weight: 4,
  dashArray: '5, 10',
}).addTo(map);
```

- **Sebep:** Birden fazla noktayı birleştirerek yol veya rota gösterimi.
    
- **Amaç:** Harita üzerinde çizgisel veri sunmak.
    
- **Sonuç:** Stil parametreleriyle kalınlık, renk ve kesikli çizgi efekti uygulanır.
    

### 3.3 Polygon (Çokgen)

```js
const polygon = L.polygon([
  [39.92, 32.86],
  [39.91, 32.87],
  [39.90, 32.85],
], {
  color: 'red',
  fillColor: '#f03',
  fillOpacity: 0.5,
}).addTo(map);
```

- **Sebep:** Alan veya bölge vurgulamak.
    
- **Amaç:** Seçilen alan üzerinde kullanıcı bilgilendirmesi.
    
- **Sonuç:** Kenar rengi, dolgu rengi ve saydamlık özellikleriyle görselleştirme.
    

---

## 4. Popup ve Tooltip Kullanımı

### 4.1 Popup

```js
marker.bindPopup('<b>Merhaba!</b><br>Bu bir pop-up.').openPopup();

line.bindPopup('Bu çizgi bir rota.');
```

- **Sebep:** Harita ögeleri hakkında detaylı bilgi vermek.
    
- **Amaç:** Kullanıcı tıkladığında veya belirtilen event’te bilgi kutusu açmak.
    
- **Sonuç:** HTML içeriği destekleyen, istenilen anda açılabilen bilgi pencereleri.
    

### 4.2 Tooltip

```js
marker.bindTooltip('İşaretçi');
polygon.bindTooltip('Burası ilçe sınırları içinde.', {
  permanent: true,
  direction: 'center',
});
```

- **Sebep:** Sürekli gösterilmesi gereken küçük bilgi etiketleri.
    
- **Amaç:** Harita üzerindeki ögeleri etiketleyerek kullanıcı deneyimini zenginleştirmek.
    
- **Sonuç:** Fareyle üzerine gelindiğinde veya kalıcı olarak gösterilen küçük bilgi balonları.
    

---

> Bu adımlarla Leaflet’in temel işlevlerini (kurulum, harita oluşturma, öge ekleme, bilgi kutuları) öğrenmiş oldunuz. Bir sonraki dokümanda Layer kontrolü, Heatmap, Cluster gibi ileri seviye eklentilerle devam edeceğiz.