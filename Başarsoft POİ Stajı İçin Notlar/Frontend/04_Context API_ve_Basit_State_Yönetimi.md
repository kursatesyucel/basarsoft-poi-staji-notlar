# 04_Context API ve Basit State Yönetimi

Bu dokümanda **global state** ihtiyacı, Context API ile nasıl karşılanacağı ve örnek bir senaryo üzerinden "seçili objeyi tüm bileşenlerde yönetme" konusu **sebep**, **amaç** ve **sonuç** odaklı olarak anlatılacaktır.

---

## 1. Global State İhtiyacı

### 1.1 Ne Zaman Global State Gerekir?

- **Problem:** Bir uygulamada, birden çok bileşenin aynı veriyi kullanması veya güncellemesi gerektiğinde, prop zincirleri (prop drilling) uzun ve karmaşık hale gelebilir.
    
- **Sebep:** Derin bileşen ağaçlarında, en üst seviyeden alt seviyeye props taşıma ihtiyacı, kodun okunabilirliğini ve bakımını zorlaştırır.
    
- **Amaç:** Ortak veriyi, ihtiyaç duyulan her bileşene doğrudan sunarak, kod şablonunu basitleştirmek.
    
- **Sonuç:** Düzgün ayarlanmış bir global state yapısı, bileşenler arası veri paylaşımını kolaylaştırır ve prop zincirlerini ortadan kaldırır.
    

---

## 2. Context API ile Basit State Yönetimi

### 2.1 Context API Nedir?

- **Sebep:** React, prop drilling yerine merkezi bir veri deposu (store) tanımlama ihtiyacına cevap verir.
    
- **Amaç:** `React.createContext()` ile oluşturulan Context nesnesi, sağlayıcı (Provider) ve tüketici (Consumer) bileşenleri aracılığıyla global state paylaşımını kolaylaştırır.
    
- **Sonuç:** Hem okunan hem de güncellenen ortak veriler, Context Provider ağı içinde kalan tüm bileşenlere erişilebilir hale gelir.
    

### 2.2 Temel Kullanım Adımları

1. **Context Oluşturma**
    
    ```jsx
    import React from 'react';
    
    export const SelectedContext = React.createContext({
      selectedId: null,
      setSelectedId: () => {}
    });
    ```
    
    - **Sebep:** İlk değerler ve imza (shape) burada tanımlanır.
        
    - **Sonuç:** Context, sağlayıcı ve tüketiciler için hazır hale gelir.
        
2. **Provider Bileşeni**
    
    ```jsx
    import React, { useState } from 'react';
    import { SelectedContext } from './SelectedContext';
    
    export function SelectedProvider({ children }) {
      const [selectedId, setSelectedId] = useState(null);
      
      return (
        <SelectedContext.Provider value={{ selectedId, setSelectedId }}>
          {children}
        </SelectedContext.Provider>
      );
    }
    ```
    
    - **Sebep:** Global state ve güncelleme fonksiyonunu context içinde tutmak.
        
    - **Amaç:** Uygulamanın kökünde Provider ile sarmalayarak, tüm alt bileşenlere erişim sağlamak.
        
    - **Sonuç:** `selectedId` ve `setSelectedId`, tüm tüketici bileşenler tarafından kullanılabilir.
        
3. **Tüketici (Consumer) Kullanımı**
    
    ```jsx
    import React, { useContext } from 'react';
    import { SelectedContext } from './SelectedContext';
    
    function Item({ id, label }) {
      const { selectedId, setSelectedId } = useContext(SelectedContext);
      const isSelected = selectedId === id;
    
      return (
        <div
          onClick={() => setSelectedId(id)}
          style={{
            padding: '8px',
            margin: '4px',
            border: isSelected ? '2px solid blue' : '1px solid gray',
            background: isSelected ? '#e0f0ff' : 'white',
          }}
        >
          {label}
        </div>
      );
    }
    ```
    
    - **Sebep:** useContext Hook ile context değerlerine erişmek.
        
    - **Amaç:** Herhangi bir bileşende, global state'i okumak veya güncellemek.
        
    - **Sonuç:** Tıklanan kalemin id'si global state'e yazılır ve tüm bileşenler yeniden render edilerek seçili durum güncellenir.
        
4. **Uygulama Kökünde Provider ile Sarma**
    
    ```jsx
    import React from 'react';
    import ReactDOM from 'react-dom';
    import { SelectedProvider } from './SelectedContext';
    import App from './App';
    
    ReactDOM.createRoot(document.getElementById('root')).render(
      <SelectedProvider>
        <App />
      </SelectedProvider>
    );
    ```
    
    - **Sonuç:** `App` ve tüm alt bileşenleri, `SelectedContext` değerlerine erişebilir.
        

---

## 3. Örnek Senaryo: Seçili Objeyi Tüm Bileşenlerde Yönetme

### 3.1 Problem Tanımı

- Bir liste ve detay bileşeni var. Listeden bir öğe seçildiğinde, detay bileşeninde seçilen öğenin bilgisi gösterilmeli.
    
- Prop drilling yerine Context API kullanarak daha temiz mimari sağlanacak.
    

### 3.2 Komponent Yapısı

```
<App>
  ├─ <List />          // Item bileşenlerini listeler
  │    └─ <Item />     // Tek tek öğeler
  └─ <Detail />        // Seçili öğeyi gösterir
```

### 3.3 Detay Bileşeni Kullanımı

```jsx
import React, { useContext } from 'react';
import { SelectedContext } from './SelectedContext';

function Detail({ items }) {
  const { selectedId } = useContext(SelectedContext);
  const selectedItem = items.find(item => item.id === selectedId);

  if (!selectedItem) return <p>Lütfen bir öğe seçin.</p>;
  
  return (
    <div>
      <h2>Detay: {selectedItem.label}</h2>
      <p>ID: {selectedItem.id}</p>
      <p>Açıklama: {selectedItem.description}</p>
    </div>
  );
}
```

- **Sonuç:** Listeden seçilen öğe, farklı bileşen ağacında kolayca tüketilip gösterilir.
    

---

> Context API, küçük ve orta ölçekli projelerde basit global state yönetimi sağlar. Büyük ve karmaşık state ihtiyaçları için Redux, MobX veya Recoil gibi kütüphaneler tercih edilebilir.