## uretimapi

> `UretimV4` (CepPatronERP masaustu uygulamasi) ve `WebUretim TabletV2` (tablet/web istemcisi) icin **ortak REST API**'sini saglar. Mikro ERP veritabani ile sirket ic Uretim veritabani arasinda dogrudan SQL erisimiyle (Dapper) calisir; siparis aktarim, stok takibi, uretim hareketleri ve kullanici/login islemleri sunar.

# CLAUDE.md - WebUretim ApiFeza V1 (Backend API)

## Proje Ozeti

`UretimV4` (CepPatronERP masaustu uygulamasi) ve `WebUretim TabletV2` (tablet/web istemcisi) icin **ortak REST API**'sini saglar. Mikro ERP veritabani ile sirket ic Uretim veritabani arasinda dogrudan SQL erisimiyle (Dapper) calisir; siparis aktarim, stok takibi, uretim hareketleri ve kullanici/login islemleri sunar.

ASP.NET Core 6.0 Web API + JWT authentication + Swagger UI ile sunulur. Windows Service olarak da host edilebilir (`UseWindowsService`).

## Iliskili Projeler

Bu API, su iki projeyle birlikte calisir (ayni veritabanlarini paylasir):

| Proje | Rol | Kullanim |
|-------|-----|----------|
| **UretimV4** (`D:\Projelerimmm\UretimV4`) | WinForms masaustu ERP (CepPatronERP.exe) | Ofis kullanicilari icin tam ozellikli ERP arayuzu — bu API'yi **kullanmaz**, DB'ye dogrudan baglanir |
| **WebUretim ApiFeza V1** *(bu proje)* | Backend API (MyApi.dll) | Tablet ve diger web istemcilerinin DB'ye erismek icin kullandigi REST API |
| **WebUretim TabletV2** (`D:\Projelerimmm\WebUretim TabletV2`) | Tablet/web istemcisi | Bu API uzerinden HTTP istekleriyle uretim takibi yapan saha uygulamasi |

Ortak veritabanlari (paylasilan):
- **Uretim DB** (ProConn): `UretimV3_FEZA` - uretim, recete, istasyon, kullanici
- **Mikro DB** (MikroConn): `MikroDB_V16_FEZA24` - Mikro ERP stok, siparis, cari

## Teknoloji Yigini

- **Framework**: .NET 6.0 (ASP.NET Core)
- **API**: ASP.NET Core MVC + Controllers
- **Auth**: JWT Bearer (`Microsoft.AspNetCore.Authentication.JwtBearer 6.0.14`)
- **ORM**: Dapper 2.0.123 (manuel SQL + entity mapping)
- **SQL Client**: Microsoft.Data.SqlClient 5.1.1 + System.Data.SqlClient 4.8.5
- **Dokuman**: Swashbuckle.AspNetCore 6.2.3 (Swagger UI)
- **JSON**: Newtonsoft.Json 13.0.3 (MVC serileştirici)
- **Logging**: Serilog.Sinks.File 5.0.0
- **Hosting**: Microsoft.Extensions.Hosting.WindowsServices 7.0.0 (Windows Service host)

## Cozum Yapisi

```
UretimApi.sln
├── MyApi/                       # Web API (ASP.NET Core 6) → bin\Debug\net6.0\MyApi.dll, MyApi.exe
│   ├── Controllers/             # GenelController, StokTakipController
│   ├── Extentions/              # MyAuthorizationFilter (JWT yetki filtresi)
│   ├── FisKayitLog/             # Fis kayit loglari
│   ├── Properties/              # launchSettings.json (port: 7098/5094, IIS Express: 17378/44383)
│   ├── Program.cs               # Service registration, JWT config, Swagger
│   ├── ZipManager.cs
│   └── appsettings.json         # ConnectionStrings, TokenOptions, LoginUser, Kestrel
└── My/                          # Class Library (.NET 6)
    ├── Business/
    │   ├── Auth/                # MyAuthenticationService (kullanici login, token uretimi)
    │   ├── Geneller/            # GenelService (BaglantiTest)
    │   ├── Managers/            # MikroAktarimManager, MikroConvertManager (siparis aktarim, Mikro ↔ Uretim donusum)
    │   ├── Mikro/               # MikroService (Mikro ERP'den stok/siparis sorgulari)
    │   ├── StokTakipler/        # UretimStokTakipService, SiparisService
    │   ├── Uretimler/           # Uretim emir/hareket service'leri
    │   ├── Users/               # UserService
    │   └── Cariler/             # (csproj'da derlemeden cikarilmis - eski)
    ├── Core/                    # MyGuid, MyLogger, MyResult (sonuc tipleri)
    ├── DataAccess/
    │   ├── Data/                # MyDbContext (ProConn), MyDbContextMikro (MikroConn) - Dapper baglantilar
    │   └── Security/            # TokenHandler, TokenOptions, AccessToken, SingHandler, LoginModel
    └── Entities/                # POCO modeller (Ayarlar, Kullanicilar, Mikro, Siparisler, StokTakipler, UretimEmirleri, Templer)
```

## Derleme ve Calistirma

### Derleme
```powershell
cd "D:\Projelerimmm\WebUretim ApiFeza V1"
dotnet build UretimApi.sln
```

Cikti: `MyApi\bin\Debug\net6.0\MyApi.dll` ve `MyApi.exe` (Windows Service icin).
**Bilinen durum**: Build 0 hata + ~306 uyari (nullable reference warnings). Calismaya engel degil.

### Calistirma (Development)
```powershell
cd "D:\Projelerimmm\WebUretim ApiFeza V1\MyApi"
dotnet run
```
veya
```powershell
& "D:\Projelerimmm\WebUretim ApiFeza V1\MyApi\bin\Debug\net6.0\MyApi.exe"
```

Varsayilan portlar (launchSettings.json):
- HTTPS: `https://localhost:7098`
- HTTP: `http://localhost:5094`
- Swagger UI: kok adreste (RoutePrefix boş) ve `/swagger`

`appsettings.json` icindeki Kestrel ayari `http://*:8199` portunu dinler (Production icin).

### Windows Service Olarak
`Program.cs:51` icinde `builder.Host.UseWindowsService()` aktif. Service kurulumu:
```powershell
sc.exe create MyApiUretim binPath="D:\Projelerimmm\WebUretim ApiFeza V1\MyApi\bin\Release\net6.0\MyApi.exe"
sc.exe start MyApiUretim
```

### Publish
```powershell
dotnet publish UretimApi.sln -c Release -o D:\WebUretimApi_Publish
```

## Konfigurasyon (appsettings.json)

```json
{
  "ConnectionStrings": {
    "ProConn":   "Server=192.168.3.201;Database=UretimV3_FEZA;User Id=ceppatron;Password=1122334416;TrustServerCertificate=True;",
    "MikroConn": "Server=192.168.3.201;Database=MikroDB_V16_FEZA24;User Id=ceppatron;Password=1122334416;TrustServerCertificate=True;"
  },
  "TokenOptions": {
    "Audience": "www.mysite.com",
    "Issuer": "www.myapi.com",
    "AccessTokenExpiration": 24,
    "RefreshTokenExpiration": 48,
    "SecurityKey": "GizliKeyBulunmamasiGerekenKeydirBurayiGizle..*1.."
  },
  "LoginUser": {
    "UserName": "admin",
    "Password": "1234**"
  },
  "Kestrel": {
    "Endpoints": {
      "Http": { "Url": "http://*:8199" }
    }
  }
}
```

| Bolum | Aciklama |
|-------|----------|
| `ConnectionStrings.ProConn` | Yerel Uretim veritabani (UretimV3_FEZA) — UretimV4 ile **ayni** |
| `ConnectionStrings.MikroConn` | Mikro ERP veritabani — UretimV4 ile **ayni** |
| `TokenOptions.SecurityKey` | JWT imzalama anahtari (Production'da degistir) |
| `TokenOptions.Issuer/Audience` | JWT validation degerleri |
| `TokenOptions.*Expiration` | Saat cinsinden token omru |
| `LoginUser` | Sabit kullanici-sifre (alternatif login - varsayilan acik) |
| `Kestrel.Endpoints` | Production HTTP port (8199) |

## Endpoint Yapisi

Tum rotalar `[Route("api/[controller]/[action]")]` kalibinda; varsayilan **tum endpoint'ler JWT gerektirir** (Program.cs:62 - global `MyAuthorizeFilter`). `[AllowAnonymous]` ile isaretli olanlar muaftir.

### Controllers

| Controller | Anonim Endpoint | Yetkili Endpoint Ornekleri |
|------------|----------------|---------------------------|
| `GenelController` (api/Genel/*) | `BaglantiTest` (DB ping), `Login` (JWT olustur) | Kullanici/yetki bilgileri |
| `StokTakipController` (api/StokTakip/*) | `StokFisList` (AllowAnonymous) | Uretim stok fislerini listele, siparis aktar |

Disable edilmis controller'lar (csproj'dan derleme disi tutulmus):
- `CariHesaplarController` (MyApi.csproj:10)
- `MikroAktarimController` (MyApi.csproj:11)

## Authentication Akisi

1. Istemci `POST /api/Genel/Login` ile `{ UserName, Password }` gonderir.
2. `MyAuthenticationService.CreateAccessToken` kullaniciyi dogrular.
3. `TokenHandler` JWT uretir, `TokenOptions.SecurityKey` ile imzalanir.
4. Istemci sonraki isteklerde `Authorization: Bearer <token>` header'i gonderir.
5. `JwtBearer` middleware token'i dogrular; gecerli ise kontroller calisir.
6. JWT validation tum kriterleri kontrol eder: `ValidateAudience`, `ValidateIssuer`, `ValidateLifetime`, `IssuerSigningKey` (`Program.cs:69`).

## Veritabani Erisimi

Iki ayri `DbContext` (Dapper baglanti wrapper, EF Core degil):

- `MyDbContext` → ProConn (Uretim DB) — `[My/DataAccess/Data/MyDbContext.cs](My/DataAccess/Data/MyDbContext.cs)`
- `MyDbContextMikro` → MikroConn (Mikro ERP DB) — `[My/DataAccess/Data/MyDbContextMikro.cs](My/DataAccess/Data/MyDbContextMikro.cs)`

Her ikisi de **Singleton** olarak DI'a kayitli (`Program.cs:26-27`).

Service'lerin dogasi:
- `UserService`, `GenelService`, `UretimStokTakipService`, `SiparisService` → ProConn kullanir
- `MikroService` → MikroConn kullanir
- `MikroAktarimManager` → her ikisini de kullanir (siparis aktarim islemi icin)

## Sonuc Tipi

Tum service metodlari `IMyResult` veya `IMyResult<T>` doner (`My/Core/MyResult.cs`):
```csharp
{ Success: bool, Message: string, Code: int, Data: T }
```
Controller'lar `result.Code` ile HTTP status'u eslestirir (200, 400, 500 vb.).

## Kod Konvansiyonlari

### Adlandirma
- **Turkce tanimlayicilar**: Siparis, UretimEmir, StokTakip, Mikro, Recete, Kullanici, Cari
- **Service kalibi**: `I<Ad>Service` + `<Ad>Service`
- **Manager kalibi**: `<Modul>Manager` (cok service koordine eden)
- **Controller kalibi**: `<Modul>Controller` (kucuk, ince - is mantigi service'lere delege)

### Mimari Katmanlar
1. **Entities** (POCO) - DB ile esitlenmis modeller
2. **DataAccess.Data** (Dapper baglantilar) - DB context wrapper'i
3. **Business.Service** (Service'ler) - Tek bir is alani sorgular/komutlari
4. **Business.Manager** (Manager'lar) - Birden cok service koordine eden orkestrasyon
5. **MyApi.Controllers** - HTTP istek/cevap, validasyon, service delegasyonu

### Kullanim
- `Compile Remove="..."` kullanilarak eski/calsmaz kod derlemeden cikarilmis: `Business\Cariler\**`, `Entities\CariHesaplar\**`
- `Controllers\CariHesaplarController.cs` ve `MikroAktarimController.cs` da derleme disi

## Iliskili Projelerle Etkilesim

### UretimV4 (CepPatronERP.exe) ile
- **Ayni DB'leri kullanir** (Uretim + Mikro). Ayni `Kullanici` tablosu, ayni audit alanlari.
- UretimV4 dogrudan DB'ye baglanir; API'yi cagirmaz.
- Kullanici sifreleri **farkli** sifrelenebilir: UretimV4 AES "OzelAnahtar1234%&", API ise farkli bir mekanizma kullaniyor olabilir (kontrol icin `UserService.cs`'e bak).

### WebUretim TabletV2 ile
- TabletV2 bu API'nin **istemcisidir**. HTTP uzerinden JWT auth ile cagrilar yapar.
- TabletV2 buyuk olasilikla baz URL'yi konfigurasyondan alir (`http://<api-host>:8199` veya `http://localhost:5094` dev'de).
- Ortak entity sozlesmesi yoktur - JSON serileştirme ile kontrat olusur.

## Bilinen Sorunlar / Notlar

- **TrustServerCertificate=True**: Geliştirme/yerel SQL Server icin uygun; production'da SSL sertifikasi dogrulamasi onerilir.
- **SecurityKey appsettings'te aciktir**: Production'da User Secrets / Azure Key Vault / env-var ile saklanmali.
- **LoginUser bolumu**: `{ admin / 1234** }` sabit kullanici aktif olabilir — kontrol et, kapatilmasi gerekebilir.
- **CariHesaplar tamamen kaldirilmis**: csproj `Compile Remove` ile dosyalar derlemeden cikarilmis ama dizinde duruyor.
- **HTTPS redirect**: `app.UseHttpsRedirection()` (Program.cs:129) aktif; HTTP cagrilar otomatik HTTPS'e yonlendirilir. Sadece HTTP host edersen bu satiri kaldir.
- **Nullable warnings**: ~306 uyari mevcut (CS8602, CS8604, vb). Calisma davranisini etkilemiyor; ileride `#nullable disable` ile cesitli dosyalarda susturulabilir.

## Hizli Test

```powershell
# 1. Bagalanti test (anonim)
curl http://localhost:5094/api/Genel/BaglantiTest

# 2. Login al
curl -X POST http://localhost:5094/api/Genel/Login `
  -H "Content-Type: application/json" `
  -d '{ "UserName": "admin", "Password": "1234**" }'

# 3. Yetkili endpoint cagir
curl http://localhost:5094/api/StokTakip/SomeAction `
  -H "Authorization: Bearer <token>"
```

veya Swagger UI ile gorsel: `http://localhost:5094/` (kok adres).

## Hizli Baslangic Kontrol Listesi

- [ ] .NET 6.0 SDK kurulu (`dotnet --list-sdks` ile teyit)
- [ ] SQL Server 192.168.3.201 erisilebilir (veya appsettings duzenle)
- [ ] `ceppatron` SQL kullanicisinin UretimV3_FEZA ve MikroDB_V16_FEZA24 DB'lerine erisim yetkisi var
- [ ] `dotnet restore UretimApi.sln` (NuGet paketleri)
- [ ] `dotnet build UretimApi.sln` (0 hata, ~306 uyari beklenir)
- [ ] `dotnet run --project MyApi` veya `MyApi.exe` calistir
- [ ] `http://localhost:5094/` → Swagger UI acilir
- [ ] `api/Genel/BaglantiTest` 200 OK → DB baglantisi calisiyor

---
> Source: [mb2763/uretimApi](https://github.com/mb2763/uretimApi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
