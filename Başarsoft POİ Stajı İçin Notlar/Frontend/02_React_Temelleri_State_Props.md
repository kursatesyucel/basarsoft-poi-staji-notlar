# 02_React Temelleri: State & Props

Bu dokümanda React bileşen tipleri (Function vs Class), JSX, props, state ve temel Hook’lar (useState, useEffect) ele alınacaktır.

---

## 1. Bileşen Tipleri

### 1.1 Function Components

```jsx
import React from 'react';

function HelloFunction(props) {
  return <h2>Merhaba, {props.name}!</h2>;
}

export default HelloFunction;
```

- **Avantajlar:** Daha sade, daha az kod. Hook’larla birlikte state ve yan etkiler yönetilebilir.
    
- **Dezavantajlar:** ES6 bilgisi gerektirir; eski React sürümlerinde sınırlıydı.
    

### 1.2 Class Components

```jsx
import React, { Component } from 'react';

class HelloClass extends Component {
  render() {
    return <h2>Merhaba, {this.props.name}!</h2>;
  }
}

export default HelloClass;
```

- **Avantajlar:** `this.state`, `this.setState` ve yaşam döngüsü yöntemleri (`componentDidMount`, vs.) klasik yaklaşımla.
    
- **Dezavantajlar:** Daha fazla boilerplate; `this` bağlamı karmaşıklıkları.
    

---

## 2. JSX

- JavaScript içinde HTML benzeri sözdizimi.
    
- Derlendiğinde `React.createElement` çağrılarına dönüştürülür.
    

```jsx
// JSX
const element = <div className="card">Hoş Geldiniz!</div>;

// Derlendiğinde
// React.createElement('div', { className: 'card' }, 'Hoş Geldiniz!');
```

- **Kurallar:**
    
    - Tek kök eleman (root) olmalı.
        
    - `class` yerine `className`, `for` yerine `htmlFor` kullanılır.
        
    - JavaScript ifadeleri süslü parantez içinde (`{ }`) yazılır.
        

---

## 3. Props

- Bileşene dışardan geçirilen veri.
    
- Bileşen içinde değiştirilemez (read-only).
    

```jsx
function UserCard(props) {
  return (
    <div>
      <h3>{props.username}</h3>
      <p>Yaş: {props.age}</p>
    </div>
  );
}

// Kullanım:
<UserCard username="Ahmet" age={30} />
```

---

## 4. State

- Bileşenin kendi yerel verisi.
    
- Class bileşenlerde `this.state` ve `this.setState` ile, fonksiyonel bileşende `useState` Hook ile yönetilir.
    

### 4.1 Class Component ile State

```jsx
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <button onClick={this.increment}>
        Sayaç: {this.state.count}
      </button>
    );
  }
}
```

---

## 5. Hook’lara Giriş

### 5.1 useState

- Fonksiyonel bileşenlere state ekler.
    
- `const [state, setState] = useState(initialValue)` sözdizimi.
    

```jsx
import React, { useState } from 'react';

function FunctionalCounter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Sayaç: {count}
    </button>
  );
}
```

### 5.2 useEffect

- Yan etkileri (side effects) yönetmek için kullanılır.
    
- Veri çekme, DOM manipülasyonu, abonelikler vs.
    

```jsx
import React, { useState, useEffect } from 'react';

function DataFetcher({ url }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((json) => setData(json));
  }, [url]); // url değiştiğinde yeniden çalışır

  if (!data) return <p>Yükleniyor...</p>;
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

---

> Bu temel kavramlar, React ile interaktif ve modüler kullanıcı arayüzleri oluşturmanın dayanak noktalarıdır. Bir sonraki dokümanda daha ileri Hook’lar ve performans optimizasyonlarına geçeceğiz.