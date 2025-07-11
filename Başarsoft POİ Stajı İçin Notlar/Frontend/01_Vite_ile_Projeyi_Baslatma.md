
# 01_Vite_ile_Projeyi_Baslatma.md

## Giriş  
**Sebep:** Günümüzde modern web uygulamaları giderek büyüyor; klasik derleme araçları (webpack, gulp vb.) geliştirme sürecini yavaşlatabiliyor. İhtiyacımız, **hızlı yeniden derleme**, **kolay yapılandırma** ve **modern tarayıcı desteği** sunan bir araç.  
**Amaç:** Vite kullanarak, proje başlangıcından itibaren hem geliştirme hem de üretim aşamasında hızlı, verimli bir iş akışı oluşturmak.  
**Sonuç:** Kod değişiklikleri anında tarayıcıya yansır, yapılandırma dosyaları sadeleşir ve ESModules’un tüm avantajlarını yaşayarak öğrenmiş oluruz.

---

## 1. Vite Nedir ve Neden Hızlı?

### 1.1 Sebep  
Klasik bundler’lar (ör. webpack) tüm modülleri baştan paketleyip tarayıcıya sunar; küçük bir değişiklikte bile tüm projeyi yeniden derler. Bu da “Bekleme süresi” ve “Soğuma (cold start)” problemi yaratır.

### 1.2 Amaç  
- **Geliştirme ortamında**: Değişen modülü izole ederek yalnızca o modülü anında tarayıcıya yüklemek.  
- **Üretime hazır**: Optimize edilmiş, minify edilmiş, cache-bust destekli paketler oluşturmak.  

### 1.3 Sonuç  
- Vite, geliştirme aşamasında **ESModule**’leri tarayıcıya doğrudan sunarak **dev server**’da anlık güncellemeler (HMR) yapar.  
- Üretim aşamasında Rollup tabanlı hızlı paketleme ile optimize edilmiş çıktılar üretir.

---

## 2. Proje Oluşturma

### 2.1 Sebep  
Her proje farklı ihtiyaçlara sahip olabilir (JavaScript vs TypeScript, React vs Vue vs Svelte). Manuel yapılandırma zaman kaybettirir.

### 2.2 Amaç  
Vite’in komut satırı sihirbazıyla birkaç adımda projemizi başlatmak; hem bağımlılıkları hem de temel dosya yapısını otomatik oluşturmak.

### 2.3 Uygulama  
1. Terminali açın ve proje dizinine gidin:  
   ```bash
   cd frontend
```

2. Vite projesi başlatın:
    
    ```bash
    npm create vite@latest
    ```
    
3. Sihrbaz sorularını takip edin:
    
    - **Proje adı**: `harita-uygulama` (örnek)
        
    - **Framework**: `React`
        
    - **Variant**: `JavaScript` veya `TypeScript`
    - 
- **EDİTÖRÜN NOTU: JS seçmenizi şiddetle tavsiye ederim arkadaşlar**
        
4. Oluşturulan klasöre geçin ve bağımlılıkları yükleyin:
    
    ```bash
    cd harita-uygulama
    npm install
    ```
    
5. Geliştirme sunucusunu başlatın:
    
    ```bash
    npm run dev
    ```
    
6. Tarayıcıda `http://localhost:5173` adresini açın. Karşınızda “Vite + React” başlangıç sayfası belirecek.
    

### 2.4 Sonuç

- Proje kökünde `index.html`, `src/` klasörü, `vite.config.js` (veya `.ts`) dosyaları hazır.
    
- Hızlıca kod yazarak `npm run dev` ile anında tarayıcıda sonucu görebilirsiniz.
    

---

## 3. ESModule ve HMR Mekanizması

### 3.1 Sebep

- Klasik bundler’lar geliştirme sunucusunda derlenmiş paketleri sunarken modül değişikliklerinde tüm paketleri yeniden yükler.
    
- ESModules (native `<script type="module">`) tarayıcıda modül bağımlılıklarını ve keşfini kendi yapar; yeniden paketlemeye gerek kalmaz.
    

### 3.2 Amaç

- **ESModule** desteğiyle her dosya bağımsız bir modül olarak servis edilsin.
    
- Değişen modül anında tarayıcıda güncellensin; **Hot Module Replacement (HMR)** ile sayfayı yenilemeden UI durumunu koruyalım.
    

### 3.3 Çalışma Prensibi

1. **Geliştirme sunucusu**:
    
    - Vite, `src/main.jsx` gibi modülleri doğrudan tarayıcıya `<script type="module" src="/src/main.jsx">` olarak sunar.
        
2. **Dosya değişimi algılama**:
    
    - Kaynak dosyada yapılan değişiklikte sunucu yalnızca ilgili modül dosyasını yeniden derler ve tarayıcıya “bu modülü güncelle” mesajı gönderir.
        
3. **HMR**:
    
    - Tarayıcıda ilgili modül doğrudan güncellenir; component state’i korunur, tam sayfa yenileme gerekmez.
        

### 3.4 Sonuç

- **Cold start** süresi neredeyse **anlık** (ms seviyesinde).
    
- Kod yazarken bekleme süresi en aza iner; geliştirici verimliliği artar.
    

---

## 4. Yapılandırma ve Özelleştirme

### 4.1 Sebep

Standart proje yapısı çoğu ihtiyaç için yeterli, ancak bazı durumlarda alias (örn. `@/components`), proxy ayarları veya CSS ön işleyiciler (Sass, Less) gerekebilir.

### 4.2 Amaç

`vite.config.js` içinde basit eklentiler ve ayarlar yaparak projeyi projenize özgü ihtiyaçlara adapte etmek.

### 4.3 Örnek: Path Alias Kurulumu

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  server: {
    proxy: {
      '/api': 'http://localhost:5000'
    }
  }
})
```

- `@/components/Button.jsx` gibi import imkanı
    
- `/api` istekleri backend sunucuya yönlendirilir
    

### 4.4 Sonuç

- Vite konfigürasyonu minimal, anlaşılır ve kolayca genişletilebilir.
    
- Proje ihtiyaçlarına göre hızla özelleştirme yapabilirsiniz.
    

---

## 5. Özet ve İleri Okuma

- **Vite**: ESModule tabanlı, dev’de anlık güncelleme (HMR), prod’da Rollup optimizasyonu.
    
- **Proje Başlatma**: `npm create vite@latest` → temel dosyalar ve komutlar hazır.
    
- **ESModule & HMR**: Saniyeler içinde modül yenileme, state korunumu.
    
- **Config**: `vite.config.js`’le alias, proxy, plugin ekleyerek projenizi büyütebilirsiniz.



## 6. Editörün Notları
 

---

## React'a Giriş ve Tavsiye Edilen Dizin Yapısı Hakkında

Arkadaşlar merhaba,  
Bugüne kadar React konusunda temel konulara değindik:

- **React’ın neden tercih edildiği, hangi ihtiyaçlara çözüm sunduğu**
    
- **ESModule yapısı ve Hot Module Replacement (HMR) mantığı**
    

Ancak farkındayım ki, bu bilgiler şu aşamada biraz soyut veya havada kalıyor olabilir.  
Bu yüzden, size naçizane tavsiyem şu:

---

### 1️⃣ İlk Adım: Dizin Yapısını Oturtun

React dünyasında düzenli bir dosya yapısı kurmak, özellikle proje büyüdükçe çok büyük kolaylık sağlar.  
Bunu, backend tarafındaki OOP prensiplerine benzetebilirsiniz. Örneğin:

- .NET’te nasıl sınıfları, metodları miras alıyor, yeniden kullanıyor ve düzenliyoruz;
    
- React’ta da component’leri, modülleri bu mantıkla düşünüyoruz.
    

Yani yazdığınız bir bileşeni (component) tekrar tekrar kullanabilirsiniz.  
Bu yüzden düzenli bir yapı kurmak önemlidir.

---

### ✅ Örnek Dizin Yapısı Önerisi:

```plaintext
src/
├── assets/         → Görseller, ikonlar, stiller
├── components/     → Tekrar kullanılabilir React bileşenleri
├── pages/          → Sayfa bazlı bileşenler (Home, About, Contact gibi)
├── services/       → API çağrıları ve veri işlemleri
├── hooks/          → Özel React hook’ları
├── contexts/       → Context API ile global state yönetimi
├── utils/          → Yardımcı fonksiyonlar, sabitler
├── App.jsx
├── main.jsx
```

---

### 2️⃣ Öğrenim Eğrisi Hakkında

Evet, React başlangıçta biraz karışık gibi görünebilir:

- JSX yazımı
    
- Props ve State mantığı
    
- Component mantığı
    
- Routing yapısı
    

Ama öğrendikten sonra React gerçekten size çok esnek ve güçlü bir frontend geliştirme imkanı sunar.  
Modern projelerde de yaygın şekilde kullanıldığını göreceksiniz.

---

### 3️⃣ Tavsiye

- React + Vite yapısını kullanmanızı öneriyorum.
    
- Yukarıdaki dizin yapısını baz alarak ilerlerseniz, kodlarınız daha düzenli ve sürdürülebilir olur.
    
- Elbette herkesin yöntemi farklıdır, hocamızın da dediği gibi herkesin yoğurt yiyişi farklı.  
    Ama bir temel oturtmak adına bu yapıyla başlamanızı tavsiye ederim.
    

---

==npm create vite@latest benimReactApp -- --template react==
Yukarıdaki komut satırını terminalde çalıştırarak react yazmaya başlayabiliriz.
Bilgisayara Node kurmayı unutmayın :)

