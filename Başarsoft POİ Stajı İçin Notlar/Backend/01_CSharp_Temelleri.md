## 01_CSharp_Temelleri

### Giriş

Merhaba arkadaşlar! Bu bölümde C# dilinin temellerini öğreneceğiz. Adım adım ilerleyerek önce .NET CLR mimarisine göz atacak, ardından değişkenlerden döngülere, metotlardan hata yakalamaya kadar pratik örneklerle konuları pekiştireceğiz.

---

## 1. C# ve .NET CLR Mimarisi

- **C# Nasıl Doğdu?**
    
    - Microsoft tarafından 2000 yılında Anders Hejlsberg liderliğinde geliştirildi.
        
    - Günümüzde .NET platformunun en yaygın kullanılan dillerinden biri.
        
- **.NET Common Language Runtime (CLR)**
    
    - C# kodları önce MSIL (Microsoft Intermediate Language) koduna derlenir.
        
    - CLR, bu MSIL’ı çalışma zamanında (JIT derleyici ile) yerel makine koduna çevirir.
        
    - Bellek yönetimi (GC), güvenlik, hata yakalama gibi düşük seviyeli işleri CLR üstlenir.
        

> **Not:** Böylece C# geliştiricisi bellek sızıntısı gibi sorunlarla fazla uğraşmak zorunda kalmaz.

---

## 2. Değişkenler ve Veri Tipleri

```csharp
// Tam sayı tipi
tnt sayi = 42;
// Ondalıklı sayı tipi
double oran = 3.14;
// Metin tipi
string mesaj = "Merhaba C#!";
// Boolean tipi
bool aktifMi = true;

Console.WriteLine(mesaj + " Sayı: " + sayi);
```

- **Değer Tipleri**: int, long, float, double, bool, struct
    
- **Referans Tipleri**: string, class, interface, array, delegate
    

---

## 3. Kontrol Yapıları

### 3.1 `if` - `else`

```csharp
int not = 75;
if (not >= 85)
{
    Console.WriteLine("Pekiyi");
}
else if (not >= 60)
{
    Console.WriteLine("Orta");
}
else
{
    Console.WriteLine("Kaldınız");
}
```

### 3.2 `switch`

```csharp
char durum = 'B';
switch (durum)
{
    case 'A': Console.WriteLine("Harika"); break;
    case 'B': Console.WriteLine("İyi"); break;
    default: Console.WriteLine("Geçersiz durum"); break;
}
```

---

## 4. Döngüler

### 4.1 `for`

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Adım {i}");
}
```

### 4.2 `while` ve `do-while`

```csharp
int sayac = 0;
while (sayac < 3)
{
    Console.WriteLine(sayac);
    sayac++;
}

do
{
    Console.WriteLine("En az bir kere çalışır");
} while (false);
```

### 4.3 `foreach`

```csharp
string[] meyveler = { "Elma", "Armut", "Kiraz" };
foreach (var meyve in meyveler)
{
    Console.WriteLine(meyve);
}
```

---

## 5. Metotlar ve Parametreler

```csharp
// Geri dönüşsüz metot
void Selamla(string isim)
{
    Console.WriteLine($"Merhaba, {isim}!");
}

// Geri dönüşlü metot
double Topla(double a, double b)
{
    return a + b;
}

// Kullanım
Selamla("Ayşe");
var toplam = Topla(3.5, 2.7);
Console.WriteLine(toplam);
```

- **`ref` ve `out` parametreler**
    
    - `ref`: Metota verilen değişkenin önceden bir değeri olmalı.
        
    - `out`: Değeri metot içinde atanmalı.
        

```csharp
void KareAl(ref int x)
{
    x = x * x;
}

void IkiDeger(out int a, out int b)
{
    a = 5;
    b = 10;
}
```

---

## 6. Hata Yakalama (`Exception Handling`)

```csharp
try
{
    int[] sayilar = { 1, 2, 3 };
    Console.WriteLine(sayilar[5]); // Hata!
}
catch (IndexOutOfRangeException ex)
{
    Console.WriteLine("Dizi sınırı aşıldı: " + ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine("Beklenmeyen bir hata oluştu: " + ex.Message);
}
finally
{
    Console.WriteLine("Her durumda çalışır.");
}
```

---

## 7. `IDisposable` ve `using`

- Kaynak (stream, DB bağlantısı vb.) açıldığında mutlaka kapatılmalı.
    
- `IDisposable` arayüzü, `Dispose()` metodu sağlar.
    

```csharp
using (var stream = new FileStream("dosya.txt", FileMode.Open))
{
    // Dosyayla işlemler
} // using bloğu çıkınca stream.Dispose() otomatik çağrılır
```

> **İpucu:** `using var stream = new FileStream(...);` C# 8.0 ile gelen kısa sözdizimi.

---

### Sonraki Adım

Bir sonraki bölümde OOP, SOLID ve Generic mimari yapıları ele alacağız. Bu notu okuduktan sonra bolca kod yazarak pratik yapmayı unutmayın!