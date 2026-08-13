# 🏛️ VaultCore — Kurumsal Metadata Doküman Yönetim ve İş Akışı Otomasyon Portalı

> **Staj Projesi — TOFAŞ Bilgi Teknolojileri**

M-Files mimarisinden esinlenerek geliştirilmiş kurumsal düzeyde, metadata (üstveri) güdümlü bir Doküman Yönetim Sistemi (DMS) ve İş Akışı Otomasyon platformudur. Geleneksel ve katı klasör tabanlı depolama yaklaşımını ortadan kaldırarak; çalışma zamanında (runtime) şema konfigürasyonu, dinamik form üretimi, materyalize edilmiş ACL izin motoru, check-out kilitli tam yaşam döngüsü versiyonlama, OCR destekli tam metin arama (FTS) ve kurumsal iş akışı orkestrasyonu sunan nesne odaklı bir mimariyle sıfırdan inşa edilmiştir.

---

## ⚠️ Gizlilik Beyanı (Confidentiality Notice)

Bu proje, Tofaş'taki yazılım mühendisliği stajım sırasında geliştirilmiştir.

Kurumsal gizlilik, fikri mülkiyet ve iç güvenlik gereksinimleri nedeniyle; orijinal kaynak kodları, üretim ortamı verileri, dahili ağ adresleri (URL), sistem kimlik bilgileri ve şirkete özel konfigürasyonlar **kesinlikle paylaşılmamaktadır**.

Bu doküman ve depo, projenin mimari kararlarını, yazılım mühendisliği yaklaşımlarını ve seçilmiş arayüz ekran görüntülerini içeren **arındırılmış teknik bir portföy ve proje sunumudur**.

---

## 📌 Projeye Genel Bakış & Paradigma Değişimi

Geleneksel doküman yönetim sistemleri hiyerarşik klasör ağaçlarına dayanır. Bu durum kurumsal ölçekte ciddi verimsizlikler yaratır: dokümanlar departman klasörleri arasında kopyalanır, yetkilendirme parçalanır ve bir dokümanın birden fazla boyuta (örn. Departman, Marka, Yıl, Proje) göre sınıflandırılması yedekleme olmadan imkansız hale gelir.

**VaultCore katı klasör hiyerarşisini tamamen ortadan kaldırır:**
* **Klasör Yok; Nesneler ve Özellikler Var:** Dokümanlar ve iş nesneleri sadece sahip oldukları metadata özellikleriyle tanımlanır.
* **Sanal Klasör Olarak Kayıtlı Görünümler (`SavedView`):** "Klasör" olarak algılanan yapılar, belirli özellik kriterlerine göre filtrelenen dinamik veritabanı sorgularıdır.
* **Çalışma Zamanında (Runtime) Şema Modelleme:** Sistem yöneticileri; veritabanı migration'ı çalıştırmadan veya kod yazmadan arayüz üzerinden yeni `ObjectType`, `ObjectClass`, `PropertyDefinition` ve `ValueList` tanımlayabilir.
* **Materyalize Edilmiş Erişim Kontrolü (`ObjectAcl`):** Güvenlik kuralları nesne kaydedilirken/güncellenirken değerlendirilip optimize edilmiş tablolara yazılır (materyalize edilir); böylece yüz binlerce kayıtta bile sorgu anında karmaşık kural hesaplama maliyeti oluşmaz.
* **Veri Odaklı İş Akışı Otomasyonu:** İş akışları, SLA süreleri, elektronik onay adımları ve otomatik görev atamaları kod içine gömülü `if/else` blokları yerine veritabanında tanımlı durum makineleri (state machine) üzerinden yürütülür.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                             VaultCore İşleyiş Döngüsü                       │
└─────────────────────────────────────────────────────────────────────────────┘
  1. Dinamik Şema Tanımlama (Arayüzden Kodsuz Sınıf ve Özellik Tanımı)
         ↓
  2. Doküman Girişi & Metadata İndeksleme (Dinamik Form + SHA-256 CAS Dosya Deposu)
         ↓
  3. FTS & OCR İşleme (PostgreSQL turkish_unaccent + Tesseract OCR Motoru)
         ↓
  4. Materyalize Güvenlik Ataması (ObjectAcl Motoru & Kural Değerlendirme)
         ↓
  5. İş Akışı & Görev Otomasyonu (Durum Geçişleri, SLA Takibi, Görev Dağıtımı)
         ↓
  6. Yönetişim & İzlenebilirlik (Check-Out Kilidi, Versiyonlama, Değiştirilemez Denetim İzi)
```

---

# ✨ Temel Sistem Yetenekleri

### 🗂️ Metadata Güdümlü Mimari & Dinamik Şema
* **Migration Gerektirmeyen Şema Yönetimi:** Yeni nesne türleri (Doküman, Sözleşme, Araç, Tedarikçi vb.) ve sınıflar arayüz üzerinden dinamik olarak oluşturulur.
* **Zengin Özellik Tipleri:** Metin, Tamsayı, Ondalık, Tarih/Saat, Mantıksal (Boolean), Değer Listesi (`ValueList` - Lookup) ve Çoklu Seçim alanları.
* **Dinamik Metadata Kartı:** Frontend, seçilen doküman sınıfının kurallarına göre doğrulama kriterlerini, açılır listeleri ve giriş alanlarını runtime'da otomatik render eder.

### 🔒 Materyalize Güvenlik Motoru & Dinamik ACL
* **Kural Tabanlı Yetkilendirme:** Kullanıcı rolleri, grup üyelikleri ve dokümanın metadata özellikleri kesişiminde dinamik izin kuralları (`PermissionRule`).
* **Önceden Hesaplanmış `ObjectAcl`:** İzinler nesne kaydedilirken hesaplanıp materyalize edilir; liste sorguları tek bir indeksli `JOIN` ile milisaniyenin altında yanıt verir.
* **Güvenlik Analitiği & ACL Tanılama:** İzin karmaşıklık haritası, Allow/Deny oranları ve seçilen kullanıcı için simülasyon/kural eşleşme analizi.
* **Sıfır Sızıntı İlkesi:** Yetkisiz erişimlerde (`403 Forbidden`) yanıt gövdesinde (body) hiçbir içerik döndürülmez.

### 🔄 Durum Makinesi İş Akışı Motoru & Otomasyon
* **Görsel İş Akışı Tasarımcısı:** Sürükle-bırak destekli görsel akış şeması, durumlar (States) ve yönlü geçişler (Transitions).
* **Otomatik Görev Üretimi:** Durum geçişi tetiklendiğinde ilgili kullanıcı, grup veya departman sorumlusuna SLA süreli otomatik görev atanır.
* **Elektronik Onay & Parola Doğrulama:** Kritik adımlarda mevzuata uygunluk için parola ile yeniden doğrulama zorunluluğu.
* **Çoklu Kullanıcı Çözümleme Politikaları:** İlk tamamlayanın görevi bitirmesi veya tüm atananların onaylaması zorunluluğu seçenekleri.

### 📄 Versiyon Kontrolü & Check-Out Kilit Mekanizması
* **Değiştirilemez Versiyon Geçmişi:** Her check-in işleminde yeni bir `ObjectVersion` kaydı üretilir; geçmiş versiyonlar asla üzerine yazılmaz veya silinmez.
* **Check-Out Kilidi:** Doküman üzerinde aynı anda düzenleme yapılmasını engeller; TTL süresi dolunca veya yetkili admin müdahalesiyle kilit açılabilir.
* **İçerik-Adresli Depolama (CAS):** Dosyalar fiziksel diskte içeriklerinin SHA-256 hash özetiyle (`ab/cd/abcdef...`) saklanır. Bu yapı mükerrerliği önler ve Path Traversal açıklarını imkansız kılar.

### 🔍 Tam Metin Arama (FTS) & Tesseract OCR
* **PostgreSQL GIN İndeksli Arama:** Türkçe karakter morfolojisine uygun `turkish_unaccent` konfigürasyonu ve unaccent eklentisi.
* **Optik Karakter Tanıma (OCR):** Taranmış doküman, görsel ve PDF'lerden arkaplan işçileri (Background Worker) ile otomatik metin çıkarma.
* **Yapay Zeka Destekli Metadata Önerisi:** Doküman içeriğinden otomatik metadata alan değeri öneren servis entegrasyonu.

### 🌐 Birlikte Çalışabilirlik & Yönetişim
* **WebDAV Uç Noktası:** Basic Auth ile işletim sistemi seviyesinde ağ sürücüsü olarak bağlanma ve dosya kilitleme desteği.
* **WOPI Host Protokolü:** Web tabanlı Office doküman görüntüleme ve düzenleme sağlayıcıları için hazır altyapı.
* **Değiştirilemez Denetim İzi (Audit Trail):** Veritabanı seviyesinde `UPDATE`/`DELETE` engelli, JSON payload farklarını saklayan append-only denetim günlüğü.
* **Görev Vekalet Devri (Delegation):** İzinli veya görevde olan personelin iş akışı görevlerini vekil kullanıcıya devretmesi.

---

# 🖥️ Uygulama Arayüz Kataloğu

## 1. Kimlik Doğrulama & Profil Yönetimi

### 🔐 Kurumsal Giriş Ekranı
Modern cam efekti (glassmorphism) tasarımına sahip, brute-force saldırılarına karşı IP bazlı rate-limiting korumalı, JWT tabanlı kurumsal oturum açma arayüzü.

![Giriş Ekranı](screenshots/01_giris_ekrani.png)

---

### 👤 Kullanıcı Profili & Güvenlik
Kullanıcının sistemdeki aktif rollerini, oturum bilgilerini görüntüleyebildiği ve şifre güncelleyebildiği profil arayüzü.

![Kullanıcı Profili](screenshots/10_profilim.png)

---

### 🛡️ Görev Vekalet Devri & Delegasyon
Kullanıcının yıllık izin veya saha görevi durumlarında iş akışı onay görevlerini denetim izi bozulmaksızın bir çalışma arkadaşına devredebildiği vekalet yapılandırması.

![Vekalet Devri](screenshots/11_profilim_guvenlik_vekalet.png)

---

## 2. Yönetici Gösterge Paneli & İş Akışı Merkezi

### 📊 Yönetici Gösterge Paneli (Dashboard)
Bekleyen görev sayılarını, SLA gecikmelerini, süreç dağılımlarını ve sistem genelindeki kritik KPI metriklerini anlık grafiklerle sunan yönetim merkezi.

![Gösterge Paneli](screenshots/02_gosterge_paneli.png)

---

### 🔀 İş Akışları Genel Bakış (L1 Ana Sayfa)
Kurum genelinde tanımlı tüm iş akışlarının durum bazlı sayaç çipleri, aktif nesne sayıları ve sağlık göstergeleriyle listelendiği ana merkez.

![İş Akışları Merkezi](screenshots/03_akislar_ana_sayfa.png)

---

### 📑 Sözleşme & Mali Onay Süreci (L2 Detay)
Kurumsal sözleşmelerin, satın alma evraklarının ve mali belgelerin çok kademeli onay, hukuki inceleme ve yürürlük durumlarının takibi.

![Sözleşme ve Mali Onay](screenshots/06_akis_2_sozlesme_mali_onay.png)

---

### 🔧 Teknik Servis & Garanti Akışı (L2 Detay)
Servis arıza bildirimlerinin, garanti eksper raporlarının ve teknik operasyon adımlarının iş akışı üzerinden gerçek zamanlı izlenmesi.

![Teknik Servis ve Garanti](screenshots/07_akis_3_teknik_servis_garanti.png)

---

### 📥 Açık Görevlerim
Kullanıcıya veya dahil olduğu gruplara atanmış bekleyen onay taleplerini, kalan SLA sürelerini ve öncelik derecelerini gösteren görev listesi.

![Açık Görevler](screenshots/08_gorevlerim_acik_gorevler.png)

---

### ✅ Tamamlanan Görev Geçmişi
Kullanıcının daha önce onayladığı veya sonuçlandırdığı görevlerin işlem zamanı, eklenen onay notu ve nihai kararlarıyla listelendiği geçmiş ekranı.

![Tamamlanan Görevler](screenshots/09_gorevlerim_tamamlanan_gorevler.png)

---

## 3. Nesne Gezgini & Dinamik Doküman Girişi

### 🗂️ Nesne Gezgini (M-Files Sanal Düzeni)
Sol panelde kayıtlı görünümler (sanal klasör ağacı), orta alanda filtrelenebilir nesne tablosu, sağ tarafta ise anlık metadata kartı ve versiyon geçmişi çekmecesi sunan merkezi gezgin.

![Nesne Gezgini](screenshots/12_nesne_gezgini.png)

---

### 📝 Dinamik Doküman Oluşturma Formu
Kullanıcı doküman sınıfını seçtiğinde; ilgili sınıfa bağlı zorunlu alanları, regex doğrulamalarını, tarih seçicileri ve lookup listelerini çalışma zamanında dinamik üreten form arayüzü.

![Dinamik Doküman Formu](screenshots/13_yeni_dokuman_olusturma_formu.png)

---

## 4. Yönetimsel Vault Yapısı & Dinamik Şema Yönetimi

### 🏷️ Vault Yapısı — Sınıflar & İş Akışı Bağlantısı
Doküman sınıflarının oluşturulduğu, varsayılan iş akışlarına bağlandığı ve operasyonel kuralların migration'sız yönetildiği yapılandırma sekmesi.

![Vault Sınıfları](screenshots/15_admin_vault_yapisi_siniflar.png)

---

### 📦 Vault Yapısı — Nesne Türleri
Temel sistem varlıklarının (Doküman, Sözleşme, Ekipman, Tedarikçi vb.) ve dosya barındırma davranışlarının yönetimi.

![Nesne Türleri](screenshots/16_admin_vault_yapisi_nesne_turleri.png)

---

### ⚙️ Vault Yapısı — Özellik Tanımları
Sistem genelinde kullanılacak metadata alanlarının, veri tiplerinin ve değer listesi bağlantılarının tanımlandığı ekran.

![Özellik Tanımları](screenshots/17_admin_vault_yapisi_ozellik_tanimlari.png)

---

### 📋 Vault Yapısı — Değer Listeleri
Özellik alanları tarafından kullanılan dinamik lookup listelerinin (Departmanlar, Markalar, Belge Tipleri, Gizlilik Dereceleri) merkezi yönetimi.

![Değer Listeleri](screenshots/18_admin_vault_yapisi_deger_listeleri.png)

---

## 5. Güvenlik Motoru, Dinamik ACL & Tanılama

### 🛡️ İzin Kuralları Matrisi
Kullanıcı rolleri, gruplar ve metadata özellikleri bazında nesnelere hangi izin seviyelerinin (Read, Edit, Delete, FullControl) atanacağını belirleyen kural tablosu.

![İzin Kuralları](screenshots/19_admin_izin_kurallari.png)

---

### 🗺️ ACL Karmaşıklık & Analitik Haritası
Sistemdeki izin kurallarının dağılımını, Allow/Deny oranlarını ve en karmaşık yetkilendirmeye sahip nesneleri görselleştiren analitik harita.

![ACL Analitik Haritası](screenshots/26_admin_acl_analitik_haritasi.png)

---

### 🔬 ACL Tanılama & Kullanıcı İzin Simülasyonu
Güvenlik yöneticilerinin herhangi bir kullanıcıyı seçerek hedef nesneler üzerindeki kural eşleşmelerini ve nihai izin sonucunu adım adım inceleyebildiği simülasyon aracı.

![ACL Tanılama](screenshots/27_admin_acl_tanilama_kullanici_sonucu.png)

---

## 6. Görsel İş Akışı Tasarımcısı & Süreç Otomasyonu

### 🎨 Görsel İş Akışı Tasarımcısı
Durum makinesi adımlarının, geçiş yönlerinin ve onay kapılarının görsel bir tuval üzerinde modellendiği tasarımcı arayüzü.

![İş Akışı Tasarımcısı](screenshots/20_admin_is_akislari_tasarimcisi.png)

---

### ⏱️ Akış Adım Detayları & SLA Yapılandırması
İş akışı durumlarına ait otomatik rol/grup atamalarının, zorunlu yorum kurallarının ve SLA sürelerinin yapılandırıldığı detay paneli.

![Akış Adım Detayları](screenshots/21_admin_is_akislari_adim_detaylari.png)

---

## 7. Kurumsal Denetim İzi & Organizasyon Yönetimi

### 📜 Sistem Denetim İzi (Audit Trail)
Sistemde gerçekleşen tüm işlemleri (oturum açma, okuma, check-out, metadata güncelleme, indirme, silme) zaman damgası ve kullanıcı bilgisiyle kaydeden değiştirilemez günlük ekranı.

![Denetim İzi](screenshots/22_admin_denetim_izi.png)

---

### 🔍 Denetim İzi Detay Modalı & JSON İnceleme
İlgili denetim kaydına ait ham JSON istek gövdesini, değişiklik öncesi/sonrası veri farklarını (diff), istemci IP adresini ve tarayıcı detaylarını gösteren modal.

![Denetim İzi Detayı](screenshots/23_admin_denetim_izi_detay_modali.png)

---

### 👥 Kullanıcı Yönetimi
Kullanıcı hesaplarının tanımlandığı, global rollerin (Admin, ContentManager, StandardUser) ve hesap durumlarının yönetildiği idari panel.

![Kullanıcı Yönetimi](screenshots/24_admin_kullanici_yonetimi.png)

---

### 🏢 Grup & Departman Yönetimi
İş akışı atamalarında ve yetkilendirme kurallarında kullanılan organizasyonel birimlerin ve onay komitelerinin yönetimi.

![Grup Yönetimi](screenshots/25_admin_grup_yonetimi.png)

---

# 🏗️ Sistem Mimarisi

VaultCore, sorumlulukların net olarak ayrıldığı ve tek yönlü bağımlılık kuralının uygulandığı **Clean Architecture prensiplerine dayalı Modüler Monolit** mimariyle geliştirilmiştir.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Angular 20 Frontend (SPA)                           │
│                                                                             │
│   • Standalone Bileşenler            • Reaktif Signals & Durum Yönetimi     │
│   • Dinamik Metadata Kart Motoru     • Rol ve İzin Bazlı Route Guard'ları   │
│   • M-Files Gezgin Düzeni            • HTTP Auth & Error Interceptor'ları   │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                               HTTPS / REST / JWT
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          VaultCore.API Katmanı                              │
│                                                                             │
│   • İnce REST Controller'lar         • [RequireObjectPermission] ACL Filtre │
│   • Global Exception Middleware      • Rate Limiting & Güvenlik Header'ları │
│   • Swagger / OpenAPI Dökümantasyonu • Bağımlılık Enjeksiyonu (DI) Zinciri  │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VaultCore.Application Katmanı                          │
│                                                                             │
│   • ObjectService & SearchService    • AclEngineService (Materyalizasyon)   │
│   • WorkflowService & Görev Yöneticisi • MetadataSuggestionService (AI)     │
│   • Fluent Validation & DTO Dönüşümü • Arayüz Sözleşmeleri ve Abstraction  │
└──────────────────┬───────────────────────────────────────────┬──────────────┘
                   │                                           │
                   ▼                                           ▼
┌───────────────────────────────────────────┐ ┌───────────────────────────────┐
│             VaultCore.Core                │ │   VaultCore.Infrastructure    │
│                                           │ │                               │
│  • Domain Varlıkları (Entity)             │ │  • EF Core 9 / PostgreSQL 16  │
│  • Domain Enum'ları (PermissionLevel vb.) │ │  • SHA-256 CAS Dosya Deposu   │
│  • Domain İstisnaları (Saf C#, Dış Bağımsız)│ │ • Tesseract OCR & FTS Worker  │
└───────────────────────────────────────────┘ │  • WebDAV & WOPI Sağlayıcılar │
                                              │  • AclRecalculationWorker     │
                                              └───────────────┬───────────────┘
                                                              │
                                      ┌───────────────────────┴────────┐
                                      ▼                                ▼
                               ┌─────────────┐                  ┌─────────────┐
                               │ PostgreSQL  │                  │ CAS Storage │
                               │  Metadata,  │                  │  SHA-256    │
                               │  ACL & FTS  │                  │  Dosyalar   │
                               └─────────────┘                  └─────────────┘
```

---

# 🧱 Backend Katman Dağılımı

Backend çözümü (`VaultCore.sln`) dört ana katmana ayrılmıştır:

```text
backend/src/
├── VaultCore.Core/                    # Saf Domain Katmanı (Hiçbir dış kütüphaneye bağımlı değildir)
│   ├── Entities/
│   │   ├── User.cs, Group.cs, UserGroup.cs
│   │   └── Vault/                    # ObjectType, ObjectClass, PropertyDefinition,
│   │                                 # ObjectInstance, ObjectVersion, PropertyValue,
│   │                                 # ObjectAcl, Workflow, WorkflowState, AuditEntry
│   ├── Enums/                        # PermissionLevel, PropertyDataType, RoleType
│   └── Exceptions/                   # DomainException hiyerarşisi
│
├── VaultCore.Application/             # İş Mantığı & Servis Orkestrasyonu
│   ├── Interfaces/                   # IObjectService, IAclEngineService, IWorkflowService...
│   ├── Services/                     # Temel iş kuralları implementasyonları
│   ├── DTOs/                         # İstek ve yanıt veri transfer nesneleri
│   ├── Validators/                   # Çalışma zamanı dinamik şema doğrulayıcıları
│   └── Extensions/                   # AddApplicationServices DI kaydı
│
├── VaultCore.Infrastructure/          # Dış Kaynaklar & Veritabanı İmplementasyonları
│   ├── Data/                         # VaultDbContext, EntityTypeConfigurations, SeedData
│   ├── Storage/                      # SHA-256 İçerik-Adresli Dosya Depolama
│   ├── Services/                     # TesseractOcrService, WopiService, WebDavService
│   ├── Workers/                      # AclRecalculationWorker, TextExtractionWorker
│   └── Migrations/                   # PostgreSQL veritabanı migration dosyaları
│
└── VaultCore.API/                     # API Uç Noktaları & Sunum
    ├── Controllers/                  # İnce HTTP uçları (Auth, Objects, Admin, Workflow...)
    ├── Filters/                      # RequireObjectPermissionAttribute
    ├── Middlewares/                  # GlobalExceptionMiddleware, SecurityHeadersMiddleware
    └── Program.cs                    # Minimal hosting yapılandırması
```

---

# 🌐 Frontend Mimarisi (Angular 20 Standalone)

Frontend projesi, modern standalone bileşenler ve reaktif sinyaller (Signals) üzerine kurulmuştur:

```text
frontend/src/app/
├── core/                              # Tekil altyapı servisleri ve güvenlik
│   ├── guards/                       # AuthGuard, RoleGuard (UX yönlendirme korumaları)
│   ├── interceptors/                 # AuthInterceptor (JWT ekleme), ErrorInterceptor
│   ├── services/                     # AuthService (Signal tabanlı), ThemeService, ToastService
│   └── models/                       # TypeScript modelleri ve API sözleşmeleri
│
├── features/                          # Alan modülleri (Lazy-Loaded / Tembel Yüklemeli)
│   ├── auth/login/                   # Kurumsal giriş ekranı
│   ├── vault/
│   │   ├── explorer/                 # M-Files düzeni: Görünüm Ağacı + Nesne Tablosu + Detay Çekmecesi
│   │   ├── objects/                  # Dinamik metadata kartı ve dinamik form bileşenleri
│   │   └── views/                    # SavedView sorgu oluşturucu ve sanal klasörler
│   └── admin/
│       ├── structure/                # Vault yapısı yönetimi (Sınıflar, Türler, Özellikler, Listeler)
│       ├── workflows/                # Görsel akış tasarımcısı ve durum detay paneli
│       ├── permissions/              # İzin kuralları matrisi ve ACL analitik haritası
│       ├── audit/                    # Denetim izi gezgini ve JSON payload modalı
│       └── users/                    # Kullanıcı ve grup yönetim konsolu
│
└── shared/                            # Yeniden kullanılabilir UI bileşenleri
    ├── components/shell/             # Navbar, sidebar, kullanıcı durum başlığı
    └── ui/                           # Modal, badge, onay dialogu, dinamik veri tablosu
```

---

# 🛠️ Teknoloji Yığını

| Alan | Teknoloji / Kütüphane | Sürüm / Açıklama |
| :--- | :--- | :--- |
| **Backend Çatısı** | .NET / ASP.NET Core Web API | .NET 9 (C# 13) |
| **Frontend Çatısı** | Angular | Angular 20 (Standalone Components, Signals) |
| **Veritabanı** | PostgreSQL | PostgreSQL 16 (`unaccent`, `pg_trgm`) |
| **ORM & Veri Erişimi** | Entity Framework Core | EF Core 9 (Npgsql sağlayıcısı) |
| **Kimlik & Yetkilendirme** | JWT Bearer Token | Rol Tabanlı & Materyalize `ObjectAcl` |
| **Tam Metin Arama (FTS)**| PostgreSQL FTS | `turkish_unaccent` özel metin konfigürasyonu |
| **Metin Çıkarma (OCR)**  | Tesseract OCR | Tesseract 5.x C# Sarmalayıcısı |
| **Dosya Depolama** | Content-Addressable Storage (CAS) | SHA-256 hash tabanlı segmente depolama |
| **Protokoller & Entegrasyon** | WebDAV & Microsoft WOPI Host | RFC 4918 WebDAV & WOPI standardı |
| **API Dökümantasyonu** | Swagger / OpenAPI | Swashbuckle ASP.NET Core |
| **Konteynerizasyon** | Docker & Docker Compose | Çok aşamalı (multi-stage) Dockerfile |
| **Test Altyapısı** | xUnit, NSubstitute, Testcontainers | 315 Backend Testi + 112 Frontend Testi |

---

# 🗄️ İlişkisel & Metadata Veri Modeli

Veri mimarisi, ilişkisel veri tabanı bütünlüğü ile dinamik şemanın esnekliğini birleştiren hibrit bir yapıya sahiptir:

```text
┌──────────────────┐            ┌───────────────────┐
│    ObjectType    │◄───────────┤    ObjectClass    │
└────────┬─────────┘            └─────────┬─────────┘
         │                                │
         │                      ┌─────────┴──────────┐
         │                      │   ClassProperty    │
         │                      └─────────┬──────────┘
         │                                │
         │                      ┌─────────┴──────────┐            ┌─────────────────┐
         │                      │ PropertyDefinition │───────────►│    ValueList    │
         │                      └─────────┬──────────┘            └────────┬────────┘
         │                                │                                │
         ▼                                ▼                                ▼
┌──────────────────┐            ┌───────────────────┐            ┌─────────────────┐
│  ObjectInstance  │◄───────────┤   PropertyValue   │            │  ValueListItem  │
└────────┬─────────┘            └───────────────────┘            └─────────────────┘
         │
         ├──────────────────────┬──────────────────────┬──────────────────────┐
         ▼                      ▼                      ▼                      ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│  ObjectVersion   │   │    ObjectAcl     │   │   WorkflowTask   │   │    AuditEntry    │
│ (PropertiesJson, │   │  (Materyalize    │   │ (Durum, Atanan,  │   │ (Append-Only     │
│  VersionNumber)  │   │   İzinler)       │   │  SLA Süreleri)   │   │  JSON Payload)   │
└──────────────────┘   └──────────────────┘   └──────────────────┘   └──────────────────┘
```

### Veri Modeli Temel İlkeleri:
1. **`ObjectInstance` ve `ObjectVersion` Ayrımı:** Bir nesnenin tekil kimliği `ObjectInstance`'ta tutulur. Her check-in işleminde yeni bir `ObjectVersion` üretilerek o anki durum dondurulur.
2. **`PropertyValue` & JSON Okuma Önbelleği:** İlişkisel doğruluk kaynağı `PropertyValue` tablosudur. Arama ve listeleme performansını maksimuma çıkarmak için `ObjectVersion.PropertiesJson` (jsonb) aynı veritabanı transaction'ında otomatik türetilir.
3. **Materyalize `ObjectAcl`:** İzinler doğrudan `(ObjectId, UserId/GroupId, PermissionLevel)` üçlüleri olarak saklanır. Arama sorguları karmaşık izin kurallarını hesaplamak yerine sadece bu tabloya indeksli `JOIN` atar.

---

# 🔒 Güvenlik Mimarisi & Kurumsal Yönetişim

### 1. Sıfır Bilgi Sızıntısı (403 Güvencesi)
Yetkisiz bir kullanıcı bir nesneye erişmeye çalıştığında, API `403 Forbidden` yanıtı döner ve gövde tamamen boştur. Nesnenin listeden düşmesi yeterli görülmez; doğrudan ID sorgusunda dahi nesne varlığına dair ipucu verilmez.

### 2. Merkezi `[RequireObjectPermission]` Filtresi
Controller uç noktalarında yetkilendirme doğrudan filtre seviyesinde uygulanır; controller metodları içerisinde manuel yetki kodları yazılmaz:
```csharp
[HttpGet("{id}")]
[RequireObjectPermission(PermissionLevel.Read)]
public async Task<IActionResult> GetObjectById(Guid id) { ... }
```

### 3. Path Traversal Koruması (İçerik-Adresli Depolama)
Yüklenen dosyalar diskte SHA-256 hash'lerine göre segmente edilerek saklanır (örn. `storage/ab/cd/abcdef123...`). Kullanıcının girdiği orijinal dosya adı yalnızca veritabanında metadata olarak tutulur; böylece dosya sistemi yolu manipülasyonu imkansız hale getirilir.

### 4. Değiştirilemez Denetim İzi (Append-Only)
`AuditEntry` tablosu üzerinde veritabanı seviyesinde `UPDATE` ve `DELETE` operasyonları engellenmiştir. Sistemdeki her hareket geriye dönük değiştirilemez şekilde kayıt altına alınır.

---

# 🧠 Mühendislik Zorlukları & Çözümleri

## 1. Veritabanı Migration'ı Olmadan Dinamik Şema Yönetimi
* **Zorluk:** Kurumsal organizasyonlarda yeni bir doküman tipi veya özellik alanı eklendiğinde yazılım ekibinin veritabanı migration'ı (`ALTER TABLE`) yazıp canlıya alması operasyonel darboğaz yaratır.
* **Çözüm:** `ObjectType` → `ObjectClass` → `PropertyDefinition` soyutlaması kuruldu. İlişkisel tutarlılık için `PropertyValue` tablosu, indeksli hızlı JSON aramaları için `ObjectVersion.PropertiesJson` hibrit yapısı tasarlandı.

---

## 2. 100.000+ Nesnede Milisaniyenin Altında İzin Filtreleme
* **Zorluk:** Dinamik kural bazlı izinlerin (ABAC) her liste ve arama sorgusunda anlık hesaplanması büyük veri setlerinde ciddi gecikmelere yol açar.
* **Çözüm:** **Materyalize ACL Motoru** geliştirildi. İzinler nesne kaydedildiğinde veya kurallar değiştiğinde hesaplanıp `ObjectAcl` tablosuna yazıldı. Kural/grup değişikliklerinde arkaplanda `AclRecalculationWorker` çalıştırıldı. 100.000 nesnelik yük testlerinde sorguların anlık yanıt verdiği doğrulandı.

---

## 3. Koddan Bağımsız Veri Odaklı Durum Makinesi (Workflow)
* **Zorluk:** `if (status == "Onaylandı")` gibi kod içine gömülen akış mantıkları, iş süreçleri her değiştiğinde yeniden derleme ve dağıtım gerektirir.
* **Çözüm:** Durumların, geçişlerin, SLA sürelerinin, zorunlu yorumların ve elektronik parola onaylarının tamamen veritabanında tutulduğu ve arayüzdeki görsel tasarımcıdan yönetildiği veri odaklı bir durum makinesi inşa edildi.

---

## 4. Türkçe Karakter Morfolojisine Uygun Arama ve OCR
* **Zorluk:** Standart SQL `LIKE` aramaları Türkçe karakter uyumsuzluklarına (İ/i, I/ı vb.) takılmakta ve taranmış PDF/görsellerin içindeki metinleri bulamamaktadır.
* **Çözüm:** PostgreSQL'e özel `turkish_unaccent` sözlüğü ve GIN indeksleri kuruldu; arka planda çalışan Tesseract OCR servisi ile taranan evrakların metinleri otomatik çıkarılarak arama dizinine eklendi.

---

## 5. Kapsamlı Otomasyon ve Test Mimarisi
* **Zorluk:** Karmaşık ACL kuralları ve çok adımlı iş akışlarında yetki açıklarının ve regresyonların önlenmesi.
* **Çözüm:** 
  * **315 Backend Testi:** Saf iş mantığını test eden Fake/NSubstitute birim testleri ve gerçek PostgreSQL konteyneri üzerinde çalışan **Testcontainers** + `WebApplicationFactory` API entegrasyon testleri.
  * **112 Frontend Testi:** Dinamik form üretimini, sinyal durum yönetimini ve route guard'larını doğrulayan Jasmine/Karma test seti.

---

# 📚 Mühendislik Kazanımları & Çıkarımlar

VaultCore projesinin geliştirilme süreci, kurumsal yazılım mühendisliği alanında çok yönlü pratik deneyimler sağladı:

* **Kurumsal Mimari Prensipleri:** Clean Architecture, Domain-Driven Design (DDD) yaklaşımları ve modüler monolit sistemlerin ölçeklenebilirliği konusunda derin tecrübe.
* **İleri Veri Modellemesi:** Dinamik şema esnekliğini ACID veritabanı garantileriyle birleştiren hibrit ilişkisel/JSON mimari tasarımı.
* **Güvenlik ve Uyum:** Materyalize izin motorları, parola onaylı durum geçişleri ve değiştirilemez denetim kayıtları ile regülasyonlara uygun sistem tasarımı.
* **Modern Full-Stack Mühendislik:** ASP.NET Core 9 backend ile Angular 20 Standalone / Signals reaktif mimarisinin uyumlu entegrasyonu.
* **Üretim Ortamı Olgunluğu:** Docker Compose ile servis orkestrasyonu, otomatik migration ve seed mekanizmaları, sağlık kontrolü (Health Checks) ve rate-limiting altyapısı.

---

## 📌 Proje Özeti Tablosu

| Nitelik | Detay |
| :--- | :--- |
| **Proje Adı** | VaultCore — Metadata Güdümlü Doküman Yönetim ve İş Akışı Kasası |
| **Geliştirme Bağlamı** | Yazılım Mühendisliği Stajı — Tofaş Bilgi Teknolojileri |
| **Temel Mimari** | Modüler Monolit + Clean Architecture |
| **Frontend Yığını** | Angular 20 (Standalone Components, Signals, TypeScript, SCSS) |
| **Backend Yığını** | ASP.NET Core Web API (.NET 9, C# 13) |
| **Veritabanı** | PostgreSQL 16 (`unaccent`, `pg_trgm`, GIN İndeksleri) |
| **ORM** | Entity Framework Core 9 (Code-First & Migrations) |
| **Kimlik & Yetki** | JWT Bearer, Rol Tabanlı & Materyalize Object ACL |
| **Arama & Çıkarma** | PostgreSQL Türkçe FTS + Tesseract OCR |
| **Protokoller** | WebDAV & Microsoft WOPI Host Protokolü |
| **Otomatik Testler** | 315 Backend Testi (xUnit, Testcontainers) + 112 Frontend Testi (Karma) |
| **Konteynerizasyon** | Docker & Docker Compose |

---

> **Not:** Bu dökümanda yer alan ekran görüntüleri, mimari şemalar ve teknik açıklamalar mühendislik portföyü ve sunum amacıyla hazırlanmıştır. Şirkete ait özel kaynak kodları, sistem şifreleri veya gizli kurumsal veriler içermez.
