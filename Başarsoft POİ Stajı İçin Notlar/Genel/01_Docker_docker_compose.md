
# 01_Docker & docker-compose

Bu dokümanda, projenin frontend (React + Vite) ve backend (.NET Core Web API) bileşenlerini Docker konteynerlerine almayı ve birden fazla servisi tek komutla ayağa kaldırmak için docker-compose kullanımını “Ne? Neden? Nasıl?” ekseninde adım adım anlatıyoruz.

---

## Ne?

- **Docker:** Uygulamanızı “imaj” halinde paketleyip izole bir konteynerde çalıştırmanızı sağlayan platform.  
- **docker-compose:** Çoklu konteyner servisini (frontend, backend, veritabanı vb.) tek bir YAML dosyası ile tanımlayıp yönetmenizi sağlayan araç.

---

## Neden?

1. **Taşınabilirlik & Tutarlılık**  
   - Yerel geliştirici makinenizde, CI/CD ortamında veya üretimde aynı davranışı garantiler.  
2. **Bağımlılık İzolasyonu**  
   - Farklı servislerin (Node, .NET, PostgreSQL vb.) kendi konteynerlerinde çalışarak sürüm çakışmalarını önler.  
3. **Kolay Dağıtım**  
   - Tek komutla tüm mimariyi ayağa kaldırabilir, ölçeklendirebilir veya durdurabilirsiniz.  
4. **Temiz Ortam**  
   - Makinenizde yerel bağımlılıklar yüklemeden (“npm install”, “dotnet restore” vb.) çalışabilirsiniz.

---

## Nasıl?

### 1. Ortak: `.dockerignore`

Her servisin kök dizinine bir `.dockerignore` dosyası ekleyin:

```text
# Hem frontend hem backend için örnek .dockerignore
node_modules
bin
obj
.vscode
.env.local
.git
````

---

### 2. Backend için Dockerfile

`backend/Dockerfile`:

```dockerfile
# 1. Aşama: Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Proje dosyalarını kopyala ve restore et
COPY *.sln .
COPY BasarMapApp.Api/*.csproj ./BasarMapApp.Api/
RUN dotnet restore

# Tüm kodu kopyala, publish et
COPY . .
WORKDIR /src/BasarMapApp.Api
RUN dotnet publish -c Release -o /app/publish

# 2. Aşama: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
# Varsayılan dinleme portu
EXPOSE 80
ENTRYPOINT ["dotnet", "BasarMapApp.Api.dll"]
```

> **Amaç:**
> 
> - Çok aşamalı build (multi-stage) ile sadece çalıştırma için gereken dosyaları son imaja dahil ederek boyutu küçültmek.
>     
> - SDK imajı ile derleyip, runtime imajına sadece publish sonuçlarını taşımak.
>     

---

### 3. Frontend için Dockerfile

`frontend/Dockerfile`:

```dockerfile
# 1. Aşama: Dependencies & Build
FROM node:20-alpine AS build
WORKDIR /app

# Bağımlılıkları kopyala ve yükle
COPY package*.json ./
RUN npm ci

# Kod ve Vite build
COPY . .
RUN npm run build

# 2. Aşama: Serve
FROM nginx:stable-alpine
COPY --from=build /app/dist /usr/share/nginx/html
# Opsiyonel: kendi nginx.conf dosyanızı kullanabilirsiniz
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

> **Amaç:**
> 
> - Hafif `alpine` tabanlı imajlar kullanarak küçük boyutlu konteynerler oluşturmak.
>     
> - Build sürecini izole etmek; sadece statik dosyaları bir nginx konteynerinde sunmak.
>     

---

### 4. docker-compose.yml

Proje kökünde `docker-compose.yml` dosyanız:

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "5000:80"                     # Host 5000 → Container 80
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    volumes:
      - ./backend:/src               # Kodu canlı yansıtmak için (dev)
      - ~/.nuget/packages:/root/.nuget/packages
    depends_on:
      - db

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "5173:80"                     # Host 5173 → Container 80 (nginx üzerinden)
    environment:
      - NODE_ENV=development
    volumes:
      - ./frontend:/app               # Kod değişikliklerini anında görmek için (dev)
      - /app/node_modules
    depends_on:
      - backend

  db:
    image: postgres:15-alpine
    restart: always
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: yourpassword
      POSTGRES_DB: basarmapdb
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

> **Açıklamalar:**
> 
> - `volumes` ile kodu ve paket cache’ini mount ederek geliştirme sırasında container restart’ında hızlı feedback.
>     
> - `depends_on` servislere başlama sırasını belirtir.
>     
> - Ortam değişkenleri (`environment`) ile yapılandırmayı (connection string, ASPNETCORE_ENVIRONMENT vb.) kontrol edebilirsiniz.
>     

---

### 5. Çalıştırma ve Yönetim

```bash
# İlk kez build & ayağa kaldır
docker-compose up --build

# Arkaplanda çalıştırmak için
docker-compose up -d

# Logları takip et
docker-compose logs -f frontend
docker-compose logs -f backend

# Tüm servisi durdur & sil
docker-compose down
```

- **Not:** Prod ortam için `volumes` ve canlı bind‐mount’ları (`./frontend:/app` vb.) kaldırıp, sadece imaj üzerinden çalıştırmak; ayrıca gizli bilgileri `.env` dosyasından veya Docker Secrets ile yönetmek güvenlik ve performans açısından daha iyidir.
    

---

## İpuçları & En İyi Uygulamalar

- `.env` dosyası ile hassas bilgileri yönet, `.gitignore`’a ekle.
    
- Her servise `healthcheck` ekleyerek orkestratörün durum tespitini kolaylaştır.
    
- Image boyutunu küçültmek için multi-stage ve `alpine` tabanlı imajlar kullan.
    
- CI/CD pipeline’ınızda `docker-compose` veya Docker BuildKit ile otomatik build & push adımları ekle.
    

---
Editörün Notu: Gördüğüm kadarıyla Semih docker konusunda benden çok daha tecrübeli, bu konuyu onunla da konuşabilirsiniz. Bu not genel bir giriş ve fikir verme amacı ile oldukça faydalı olmakla beraber docker için youtube üzerinden kapsamlı tutorial izlenmesini tavsiye ediyorum.