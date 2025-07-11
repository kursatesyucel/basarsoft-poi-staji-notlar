**Editörün Notu**: Arkadaşlar bu ciddi öenmli bir mevzu, birlikte çalışmayı merge conflict falan geçtim kendimiz için bile belli bir mimariyi ve temiz kodu hedeflemeliyiz. İleride projelere bakınca ne olduğunu anlamazsak çok tatsız olur. Tüm projelerinize detaylı bir README ve bol bol yorum satırı eklemeyi ihmal etmeyin.
# 03_BestPractices & Code Style

Bu dokümanda, projenin kod kalitesini, tutarlılığını ve sürdürülebilirliğini artırmak için uymanız gereken en iyi uygulamalar ve kod stili kuralları “Ne? Neden? Nasıl?” ekseninde anlatılmaktadır.

---

## Ne?

- **Kodlama Standartları:** Değişken, fonksiyon, sınıf ve dosya isimlendirmede tutarlılık.  
- **Proje Yapısı:** Mantıksal klasör/dosya organizasyonu.  
- **Bileşen Tasarımı:** Tek sorumluluk prensibine uygun, yeniden kullanılabilir React bileşenleri.  
- **Stil Rehberi:** CSS/SCSS veya CSS-in-JS yaklaşımında ortak kurallar.  
- **Git & Commit:** Anlaşılır, otomatik araçlarla doğrulanabilir mesaj formatları.  
- **Kod İnceleme Süreci:** Standart bir PR şablonu ve kontrol listesi.  
- **Test & Coverage:** Birim testi, entegrasyon testi ve kod kapsama hedefleri.  
- **Performans, Erişilebilirlik, Güvenlik:** Kullanıcı deneyimi ve sağlamlık odaklı yaklaşımlar.  

---

## Neden?

1. **Tutarlılık:** Tüm ekip üyelerinin aynı kuralları takip etmesi, kodun kolay okunmasını sağlar.  
2. **Bakım Kolaylığı:** Standartlara uygun kod, yeni geliştiricilerin projeye hızlı adapte olmasını sağlar.  
3. **Hata Azaltma:** Kod inceleme ve otomatik araçlarla hatalar erken safhada yakalanır.  
4. **Ölçeklenebilirlik:** Sağlam mimari ve konvansiyonlar, projenin büyüdükçe karmaşıklaşmasını önler.  
5. **Profesyonellik:** Temiz, anlaşılır ve iyi dokümante edilmiş kod endüstri standartlarına uygundur.  

---

## Nasıl?

### 1. Kodlama Standartları

#### 1.1. React & JavaScript
- **Dosya İsimlendirme:**  
  - Bileşenler: `PascalCase.jsx` (örn. `MapView.jsx`)  
  - Diğer modüller/hooks: `camelCase.js` (örn. `useFetch.js`)  
- **Değişken & Fonksiyon:**  
  - `camelCase` (örn. `calculateArea`)  
- **React Bileşenleri:**  
  - Fonksiyonel bileşen kullanın, gereksiz sınıf bileşenlerden kaçının.  
  - Prop türlerini `PropTypes` veya TypeScript ile tanımlayın.  
- **Satır Uzunluğu:** 100–120 karakter arası.  
- **Boşluk & Girinti:** 2 boşluk (space) kullanın.  
- **Noktalı Virgül:** Tek tip olması adına, `.prettierrc` ile zorlayın.

#### 1.2. .NET & C#
- **Sınıf & Metot İsimleri:** `PascalCase` (örn. `PointService`, `GetAllPointsAsync`)  
- **Parametre & Yerel Değişken:** `camelCase` (örn. `pointId`)  
- **Namespace:** Proje yapısına göre `Company.Project.Module` formatı  
- **Async Methodlar:** `Async` takısını ekleyin (örn. `GetAllAsync`)  
- **KOD:**
  ```csharp
  public interface IPointService
  {
      Task<IEnumerable<Point>> GetAllPointsAsync();
  }
```

---

### 2. Proje Yapısı & Dosya Organizasyonu

```
src/
├─ assets/             
├─ components/          
├─ contexts/            
├─ hooks/               
├─ pages/               
├─ services/            
├─ utils/               
├─ App.jsx              
└─ main.jsx             
```

- **Mantık Ayrımı:** UI (`components`), veri erişimi (`services`), global state (`contexts`), yardımcı fonksiyonlar (`utils`) ayrı klasörlerde.
    
- **Küçük Modüller:** Her dosya tek bir sorumluluğa (SRP) odaklansın.
    

---

### 3. Bileşen Tasarımı ve Mimari

- **Tek Sorumluluk (SRP):** Her bileşen tek bir görev yapsın.
    
- **Presentational vs Container:**
    
    - **Presentational:** Sadece UI, props ile veri alır.
        
    - **Container:** State yönetimi, veri çekme ve iş mantığı içerir.
        
- **Prop Drilling’den Kaçınma:** Gerektiğinde Context veya custom hook kullanın.
    
- **React.memo:** Ağır render’ları optimize etmek için kullanın.
    

---

### 4. Stil ve CSS

- **Yaklaşım Seçimi:**
    
    - CSS Modules (`.module.css`) veya
        
    - Styled Components / Emotion
        
- **Naming Convention (BEM):** Eğer global CSS kullanıyorsanız:
    
    ```css
    .map-container { … }
    .map-container__marker { … }
    ```
    
- **Değişkenler:** Renk, boşluk ve yazı tipi boyutlarını ortak değişkenlerle yönet (CSS değişkenleri veya theme objesi).
    
- **Responsive:** Mobil, tablet, desktop için break-point’ler açıkça tanımlı olsun.
    

---

### 5. Git ve Commit Mesajları

- **Branching Model:** Git Flow veya GitHub Flow uygulanabilir.
    
- **Commit Formatı:** Conventional Commits:
    
    ```
    feat(component): add MapView component
    fix(api): handle null response in GET /points
    chore: update dependencies
    ```
    
- **PR Şablonu:**
    
    - **Ne yapıldı?**
        
    - **Neden?**
        
    - **Ekran Görüntüleri / Adım Adım Test:**
        
    - **Checklist:** Lint, test, build kontrolleri geçiyor mu?
        

---

### 6. Kod İnceleme Süreci

- **Kontrol Listesi:**
    
    - Kodlama standartlarına uyum
        
    - Gereksiz yorum veya dead-code yok
        
    - Yeni kod için birim testi var mı
        
    - Performans ve güvenlik riskleri incelendi mi
        
- **Araçlar:**
    
    - ESLint + Prettier (JS/TS)
        
    - StyleCop veya EditorConfig (C#)
        

---

### 7. Test Yazma

- **Frontend:**
    
    - Jest + React Testing Library ile komponent ve hook testleri.
        
    - Coverage %80+ hedefleyin.
        
- **Backend:**
    
    - xUnit / NUnit + Moq ile servis ve kontrolcü testleri.
        
- **CI Entegrasyonu:** Her PR’da testler otomatik çalışsın.
    

---

### 8. Performans & Optimizasyon

- **React:**
    
    - Gereksiz render’ları önlemek için `React.memo`, `useCallback`, `useMemo`.
        
- **Harita:**
    
    - GeoJSON katmanlarını parçalı yükleme (veya clustering) ile büyük veri setlerinde performans.
        
- **Network:**
    
    - API isteği debouncing, throttling.
        
    - Prod’ya minify edilmiş bundle ve tree-shaking.
        

---

### 9. Erişilebilirlik (Accessibility)

- **ARIA Rol ve Etiketleri:**
    
    - Harita kontrollerine `aria-label` ekleyin.
        
- **Klavyeyle Erişim:**
    
    - Tüm interaktif öğeler `tabindex` desteklesin.
        
- **Kontrast & Font:** WCAG 2.1 AA standartlarına uyum.
    

---

### 10. Güvenlik (Security)

- **Input Validasyonu:** Hem frontend hem backend’de.
    
- **CORS & CSP:** Sadece güvenilen kaynaklara izin ver.
    
- **HTTPS Zorunluluğu:** API ve frontend HTTPS üzerinden sunulsun.
    
- **Secrets Yönetimi:** `.env` veya Secret Manager kullanın, kod deposuna gömmeyin.
    

---

### 11. Dokümantasyon

- **Inline Yorum:** Karmaşık iş mantıkları kısa, açıklayıcı yorumlarla desteklensin.
    
- **README:** Proje kurulum, geliştirme, test ve deploy adımları eksiksiz yazılsın.
    
- **Yardımcı Belgeler:** `docs/` klasöründe mimari diyagram, API spesifikasyonu (OpenAPI/Swagger).
    

---

### 12. Bağımlılık Yönetimi

- **Sürüm Sabitleme:** `package-lock.json` / `csproj` versiyonlarını güncel tutun.
    
- **Periyodik Güncelleme:** Dependabot veya Renovate ile paketleri takip edin.
    
- **Deprecated Uyarıları:** Konsolda görülen uyarılara hızlıca yanıt verin.
    

---

## Özet

Bu rehberdeki en iyi uygulamalar ve kod stili kurallarına uyarak:

- Kodunuz **tutarlı**,
    
- Takım arkadaşlarınız **dah a hızlı** adapte olabilecek,
    
- Bakım ve test süreçleriniz **kolaylaşacak**,
    
- Prod ortamındaki **riskler** en aza indirgenecek,
    
- ve proje **ölçeklendiğinde** karmaşıklık yönetilebilir kalacaktır.