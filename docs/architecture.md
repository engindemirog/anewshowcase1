# Architecture

> Bootstrap'ta (2026-10-26) dolduruldu. Burada kararlaştırılmış mimari anlatılır, hedef değil.
> Ajanlar planlamadan önce bu dosyayı okur.

## System overview
Şirket personelinin izin talep ettiği, İK'nın talepleri karara bağladığı ve yıllık izin
bakiyelerini yönettiği bir web uygulaması. Mimari stil **katmanlı monolit**: tek bir Python
(FastAPI) süreci hem JSON API'yi (`/api/...`) hem de build adımı olmayan HTML + vanilla JavaScript
arayüzünü (statik dosyalar) sunar; veri SQLAlchemy 2 üzerinden SQLite'ta tutulur. Gerekçe: tek ekip,
tek ürün — dağıtık mimarinin maliyeti karşılıksız; katman kuralları test ile zorlanarak yapının
erimesi engellenir ([ADR-0001](decisions/0001-katmanli-monolit-ve-yigin.md)).

## Modules / components and ownership
Bileşenler katmandır; bağımlılık yalnızca tablo sırasıyla aşağı doğru akar.

| Module | Single responsibility | Owns |
|---|---|---|
| `web/` | Kullanıcı arayüzü (HTML sayfaları, ES module JS, CSS); yalnızca `/api` ile konuşur | Sunum durumu; kalıcı veri yok |
| `app/api/` | FastAPI router'ları: HTTP ↔ Pydantic şema dönüşümü, kimlik doğrulama ve rol yetkisi, `DomainError` → HTTP eşlemesi | Şemalar (istek/yanıt) |
| `app/services/` | Tüm iş kuralları (BR-1…BR-9), durum geçişleri, iş günü ve izin hakkı hesabı, transaction sınırları | Kuralların tek yetkili yeri; `DomainError` hiyerarşisi |
| `app/repositories/` | Veritabanı erişimi (sorgular, kalıcılık); iş kuralı içermez | Sorgular |
| `app/models/` | SQLAlchemy modelleri ve enum'lar | Employee, LeaveRequest, LeaveBalance, Holiday, AuditEntry, Session |

## Communication rules
- `api` → `services` doğrudan fonksiyon çağrısı; `services` → `repositories` doğrudan çağrı.
- Bir servis başka bir servisi çağırabilir (örn. talep servisi → takvim/hak hesabı); döngüsel
  import yasaktır.
- Event, mesaj kuyruğu, arka plan işçisi v1'de yok; her şey istek içinde, tek transaction'da.
- `web/` sunucu ile yalnızca `fetch('/api/...')` JSON çağrılarıyla konuşur.

## Forbidden dependencies (make them testable)
1–3 numaralı kurallar import-linter sözleşmeleriyle `scripts/check` içinde zorlanır.
1. **Yalnızca aşağı yönlü:** `app.api → app.services → app.repositories → app.models`. Ters yönde
   import yasak (örn. `app.services` `fastapi` veya `app.api` import etmez; `app.repositories`
   `app.services` import etmez).
2. **`app.api` `app.repositories` ve `app.models`'e doğrudan erişmez** — her istek bir servis
   üzerinden geçer (iş kuralları atlanamaz).
3. **İş kuralı yalnızca `app.services` içinde:** router ve repository'lerde bakiye, çakışma, iş günü
   veya izin hakkı hesabı yapılmaz.
4. **`web/` yalnızca `/api` HTTP uç noktalarını kullanır;** arayüzde iş kuralı tekrar edilmez
   (kolaylık için önden kontrol olabilir, karar her zaman sunucuda).
5. **Harici runtime bağımlılığı yok:** Redis, mesaj kuyruğu, harici kimlik sağlayıcı yok. Yeni bir
   harici sistem ADR gerektirir.
6. **Frontend'de CDN'den script yüklenmez;** gereken her dosya `web/` içinde.

## Deliberately out of scope
- Yönetici (amir) onayı ve çok adımlı onay zinciri — v1'de tek adımlı İK onayı.
- Yarım gün / saatlik izin.
- Kurumsal SSO (LDAP, Entra ID), e-posta/SMS bildirimleri.
- Hastalık raporu dosyası yükleme, bordro entegrasyonu.
- Yıllar arası bakiye devri kuralları ve otomatik yıl sonu işlemleri (ileride change request).
- Çoklu şirket / çoklu kiracı.
