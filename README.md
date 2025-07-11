
---
## 1

Arkadaşlar, merhaba.

Sizler için kapsamlı bir ders notu hazırladım. Bu notların içinde dört ana klasör bulunuyor:

1. **Backend**
    
2. **Frontend**
    
3. **Database**
    
4. **Genel Kavramlar**
    

Her klasör kendi içinde temel bilgilerden ileri konulara kadar sıralandı. Kısaca özetlemek gerekirse:

---

### ✅ Backend:

- C# ve .NET Core temelleri
    
- Konsol uygulamaları ve temel kavramlar
    
- Katmanlı Mimari ve SOLID prensipleri
    
- Entity Framework Core kullanımı
    
- Spatial Fonksiyonlar ve PostGIS entegrasyonu
    

---

### ✅ Database:

- PostgreSQL ve PostGIS kurulumu
    
- Temel SQL komutları
    
- PostGIS Spatial fonksiyonları (Bu kısmı özellikle tavsiye ediyorum!)
    
- Sadece SQL komutlarıyla CRUD işlemleri
    

---

### ✅ Frontend:

- React + Vite + Leaflet kombinasyonu
    
- React temelleri (State, Props, useEffect, useState, Hook’lar)
    
- Leaflet ile harita oluşturma
    
- Context API ve Basit State Management
    
- Backend–Frontend entegrasyonu (Axios + CORS konuları önemli)
    

⚠️ **Not:** Axios ve CORS konusu özellikle dikkat edilmesi gereken noktalardan. Cross-Origin problemi ilk başta karmaşık gelebilir ama temelini anlamanız önemli.

---

### ✅ Genel Kavramlar:

- Docker (Semih bu konuda daha deneyimli. İhtiyaç duyanlar onunla iletişime geçebilir.)
    
- CI/CD süreçleri (Şimdilik detayına girmenize gerek yok, genel kültür amaçlı eklendi.)
    
- Best Practices:
    
    - Temiz kod
        
    - Okunabilirlik
        
    - Readme dosyalarının önemi
        
    - Yorum satırları ve kod standartları
        

Bu Best Practices kısmını özellikle hepinizin detaylı şekilde okumasını rica ediyorum. Hem bireysel hem ekip çalışması için temel niteliğinde bilgiler içeriyor.



---
## 2

Gözlemlerime ve derslerdeki tecrübelerimize dayanarak şunu belirtmek istiyorum: Aramızda deneyim seviyesi henüz başlangıç aşamasında olan arkadaşlar mevcut. Bu kesinlikle olumsuz bir durum değil; özellikle 1. ve 2. sınıf seviyesindeki arkadaşlarımız için gayet doğal.

Bu noktada, eğer ihtiyaç duyan olursa elimden geldiğince yardımcı olmaktan memnuniyet duyarım. Örneğin bugün bir arkadaşımızla yaklaşık iki saatlik birebir bir görüşme yaptık. Konularımız arasında:

- GitHub kullanımı
    
- GitHub Desktop kullanımı
    
- React'ın çalışma mantığı
    
- React’ta neden belirli yöntemler (deflate, byte işlemleri gibi) kullanılıyor
    

gibi başlıklar vardı ve sonunda o arkadaşımız için oldukça verimli geçtiğini düşünüyorum.

Aynı şekilde bu veya benzeri konularda desteğe ihtiyacı olan herkes bana rahatlıkla ulaşabilir. Çekinmeyin.

Ayrıca, projenin kendi tamamladığım haliyle olan commit ID’sini aşağıya bırakacağım. Sizin şu an yapacağınız kısım biraz daha farklı olsa da, o commit’teki dosyaları incelemeniz faydalı olacaktır. Projenin GitHub reposunu da paylaşacağım. Oradan klonlayıp örnek alabilirsiniz.

Bu süreçte özellikle tavsiyem, yapay zekâ araçlarını da (örneğin ChatGPT, Cursor vb.) aktif kullanmanız. Kod yazarken ve hata çözerken bu araçlar gerçekten işinizi kolaylaştıracaktır.


Yapay zekâ konusuyla ilgili küçük bir hatırlatma ve tavsiye paylaşmak istiyorum.

Evet, hepimiz yapay zekâ araçlarını bolca kullanacağız. Ancak burada önemli olan, yapay zekâyı bilinçli ve yönlendirici şekilde kullanmak. Kendi ifademle söyleyeyim: **Yapay zekâ bizim kölemiz olacak, biz onun kölesi olmayacağız.**

Ne demek istiyorum?  
Şu anda bile bu metni doğrudan klavye ile yazmıyorum. ChatGPT’ye sesli anlatıyorum, Speech-to-Text sistemi bunu yazıya döküyor. Ardından metni düzenliyorum.

Aynı mantık projeler için de geçerli.

Bazı arkadaşlarımız yapay zekâya doğrudan şöyle komutlar veriyor:

> “Bir proje yapacağım. CRUD operasyonlarını hazırla.”

Bu tarz genel ve yönsüz komutlarla sağlıklı sonuç almanız zor olur. Yapay zekâdan en iyi verimi almak için önce projenizi kafanızda netleştirmeniz ve ardından net, yönlendirici promptlar kullanmanız gerekiyor. Örneğin:

---

✅ Örnek Etkili Prompt:  
“Bir .NET projesi yapacağım. Backend kısmında CRUD işlemleri olacak. Database olarak PostgreSQL kullanacağım. Frontend kısmı React + Vite + Leaflet olacak. Şimdi backend yapısını düşünelim.

- Hangi katmanlar olmalı?
    
- Hangi classlar ve hangi fonksiyonlar olmalı?
    
- Interfaceler ve SOLID prensiplerine uygun dizin yapısını çıkarır mısın?”
    

Sonrasında çıkan yapıyı kendi bilginizle değerlendirmeniz ve gerekli değişiklikleri yapmanız önemli.

---

**Unutmayın:**  
Yapay zekâ size yardımcı olur ama işi öğrenmek ve tasarlamak size kalır. Bir projeyi sadece yapay zekâya bırakıp “nasıl olsa halleder” mantığıyla ilerlerseniz, uzun vadede hem proje kalitesi düşer hem de siz bir şey öğrenemezsiniz.

Ayrıca, benim paylaştığım not içerisindeki backend notlarında belki ilk bakışta yabancı terimler göreceksiniz. Özellikle o noktalarda yapay zekâya şöyle sorular sormanızı öneriyorum:

- “Burada şu kavramdan bahsedilmiş, bu ne demek?”
    
- “.NET harita uygulaması yapıyorum, bu terimi nasıl kullanırım?”
    
- “Bu fonksiyonun anlamı ve kullanım şekli nedir?”
    

Bu şekilde hem öğrendiğiniz konuları pekiştirirsiniz hem de kendi projenize uygun çözümler üretirsiniz.

Her zaman olduğu gibi takıldığınız noktada bana veya Semih’e ulaşmaktan çekinmeyin.  
Hepimize kolay gelsin!

Github: https://github.com/kursatesyucel/BasarMapApp/tree/7699c61ccb19bc4b699571e6c6901983b6df5e47
