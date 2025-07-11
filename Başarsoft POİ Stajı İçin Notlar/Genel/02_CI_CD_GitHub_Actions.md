
# 02_CI/CD & GitHub Actions

Bu dokümanda Continuous Integration (CI) ve Continuous Deployment (CD) süreçlerini GitHub Actions kullanarak nasıl otomatikleştireceğinizi “Ne? Neden? Nasıl?” soruları ekseninde anlatıyoruz.

---

## Ne?

- **Continuous Integration (CI):** Kodunuzda yaptığınız her değişikliği otomatik olarak derleyip test eden, hataları erkenden yakalayan süreç.  
- **Continuous Deployment (CD):** Başarılı CI adımlarından sonra uygulamanızı otomatik olarak üretim ortamına veya bir staging sunucusuna gönderen süreç.  
- **GitHub Actions:** GitHub üzerinde yerleşik olarak gelen, YAML dosyalarıyla tanımlanan workflow’larla CI/CD işlemlerini yürüten otomasyon platformu.

---

## Neden?

1. **Erken Hata Yakalama:** Pull request açıldığında veya kod pushlandığında derleme ve testlerin otomatik çalışması yoluyla hataları daha kod repo’suna entegre olmadan önce bulursunuz.  
2. **Tutarlılık & Tekrarlanabilirlik:** Her geliştirme makinesinde aynı şekilde çalışan “clean” bir pipeline sağlar.  
3. **Hızlı Geri Bildirim:** Geliştirici takımına kod kalitesi, güvenlik taramaları ve performans kriterleri hakkında anında bilgi verir.  
4. **Otomatik Dağıtım:** Manuel adımları ortadan kaldırarak sürüm çıkarmayı güvenli ve tekrarlanabilir hâle getirir.

---

## Nasıl?

### 1. Workflow Dosya Konumu

Projenizin kök dizininde `.github/workflows/` klasörü oluşturun. Her workflow bir `.yml` dosyasıdır:

```

.yarn-lock  
package.json  
src/  
.github/  
└─ workflows/  
├─ ci.yml  
└─ cd.yml

````

---

### 2. CI Workflow Örneği (`ci.yml`)

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  # 1. Checkout & Setup
  setup:
    runs-on: ubuntu-latest
    outputs:
      node-cache-key: ${{ steps.cache-node.outputs.cache-hit }}
      dotnet-cache-key: ${{ steps.cache-dotnet.outputs.cache-hit }}
    steps:
      - name: Kodları çek
        uses: actions/checkout@v3

      - name: Node.js önbelleği
        id: cache-node
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('frontend/package-lock.json') }}

      - name: .NET önbelleği
        id: cache-dotnet
        uses: actions/cache@v3
        with:
          path: ~/.nuget/packages
          key: ${{ runner.os }}-dotnet-${{ hashFiles('backend/**/*.csproj') }}

  # 2. Frontend Build & Test
  frontend:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Node.js kur
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Bağımlılıkları yükle
        run: |
          cd frontend
          npm ci
      - name: Testleri çalıştır
        run: |
          cd frontend
          npm run test
      - name: Üretim build
        run: |
          cd frontend
          npm run build

  # 3. Backend Build & Test
  backend:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: .NET SDK kur
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'
      - name: Restore & Test & Publish
        run: |
          cd backend/BasarMapApp.Api
          dotnet restore
          dotnet test --no-build --verbosity normal
          dotnet publish -c Release -o publish

  # 4. Artifacts (isteğe bağlı)
  package:
    needs: [ frontend, backend ]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Frontend artifact
        uses: actions/upload-artifact@v3
        with:
          name: frontend-dist
          path: frontend/dist
      - name: Backend artifact
        uses: actions/upload-artifact@v3
        with:
          name: backend-publish
          path: backend/BasarMapApp.Api/publish
````

> **Açıklamalar:**
> 
> - `on:` ile hangi olaylarda çalışacağını tanımlarsınız (push, pull_request vb.).
>     
> - `jobs:` içinde bağımsız adımları paralel veya dizi hâlinde yürütebilirsiniz.
>     
> - `actions/cache` ile npm ve NuGet paket önbelleğini kullanarak pipeline sürelerini kısaltırsınız.
>     
> - `needs:` ile job’lar arası bağımlılığı belirtirsiniz.
>     

---

### 3. CD Workflow Örneği (`cd.yml`)

```yaml
name: CD

on:
  push:
    branches: [ main ]

permissions:
  contents: read
  packages: write       # Docker package push için
  id-token: write       # Bulut sağlayıcısı kimlik doğrulaması için

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Kodları çek
        uses: actions/checkout@v3

      - name: DockerHub’a giriş
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Backend imajını build & push
        uses: docker/build-push-action@v4
        with:
          context: ./backend
          file: ./backend/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/basarmap-backend:latest

      - name: Frontend imajını build & push
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          file: ./frontend/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/basarmap-frontend:latest

      - name: KubeDeploy (örnek)
        uses: azure/k8s-deploy@v3
        with:
          manifests: |
            k8s/backend-deployment.yaml
            k8s/frontend-deployment.yaml
          images: |
            ${{ secrets.DOCKERHUB_USERNAME }}/basarmap-backend:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/basarmap-frontend:latest
```

> **Açıklamalar:**
> 
> - `on.push.branches: [ main ]` yalnızca main’e yapılan merge sonrası deploy tetikler.
>     
> - Docker imajlarını GitHub Container Registry veya Docker Hub’a gönderir.
>     
> - Kubernetes, Azure, AWS vb. target’lara deploy adımlarını ekleyebilirsiniz.
>     

---

### 4. Secrets & Environment

- **GitHub Secrets:**
    
    - `DOCKERHUB_USERNAME`
        
    - `DOCKERHUB_TOKEN`
        
    - `AZURE_CREDENTIALS` / `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`
        
- **Environment Variables:**
    
    - Build sırasında `NODE_ENV`, `ASPNETCORE_ENVIRONMENT` gibi değişkenleri `env:` altında tanımlayabilirsiniz.
        

---

## İpuçları & En İyi Uygulamalar

- **Minimal Permissions:** Workflow izinlerini sadece gerekli kaynaklarla sınırlandırın.
    
- **Cache Kullanımı:** Hem npm hem NuGet paketlerini önbelleğe alarak pipeline sürelerini kısaltın.
    
- **Concurrency & Cancel:** Aynı branch için paralel workflow’lardansa, önceki workflow’u iptal eden ayarlar ekleyin (örn. `concurrency` özelliği).
    
- **Yapılandırma Ayrımı:** CI ve CD’yi ayrı dosyalarda tutarak okunabilirliği artırın.
    
- **Durum Kontrolü:** `jobs.<job_id>.if` ile koşullu adımlar oluşturun (örn. sadece etiketli release’lerde deploy).
    
- **Bildirimler:** Başarısız workflow’larda ekip kanallarına Slack veya e-posta bildirimleri ekleyin.
    


Editörün Notu: Arkadaşlar bu ileri seviye sayılabilecek bir konu, şu anda bununla kafa yormanıza hiç gerek yok. Genel kültür olması için koydum, CI/CD süreçleri büyük ve devamlı gelişen projelerde asıl önemini kazanmaktadır
---