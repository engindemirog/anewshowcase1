# Security

> Bootstrap'ta (2026-10-26) dolduruldu. Ajanların her planda ve review'da uyduğu temel kurallar.
> Sistem kişisel ve özel nitelikli kişisel veri (KVKK) işler.

## Secrets
- Secrets never enter the repo, specs, prompts, or chat. `.env` is gitignored; provide `.env.example`.
- Agents never print secret values, even when debugging.
- Oturum imzalama anahtarı gibi değerler yalnızca ortam değişkeninden okunur; kodda varsayılan
  gerçek değer bulunmaz.

## Input & output
- Tüm API girdisi Pydantic şemalarıyla doğrulanır (tip, aralık, `start <= end`, bilinen enum).
- SQL yalnızca SQLAlchemy ifadeleri/parametreleriyle; string birleştirerek sorgu kurulmaz.
- Arayüzde kullanıcı kaynaklı veri yalnızca `textContent` / `setAttribute` ile basılır;
  `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `eval` yasak (XSS).
- Hata yanıtlarında stack trace, SQL, dosya yolu veya başka bir personelin verisi bulunmaz
  (`docs/conventions.md`, Error handling).
- **Log hijyeni (KVKK):** log'a parola, oturum kimliği, doğum tarihi, izin açıklama metni ve
  ret gerekçesi yazılmaz; personel log'da yalnızca ID ile anılır.
- Hastalık izni (`SICK`) için yalnızca tür ve tarihler saklanır; teşhis, rapor içeriği ve dosya
  yükleme yoktur.

## AuthN / AuthZ
- **Kimlik doğrulama:** yerel kullanıcı adı + parola. Parolalar **argon2id** (`argon2-cffi`) ile
  hash'lenir; düz metin veya geri çözülebilir saklama yasak.
- **Oturum:** sunucu tarafında saklanan rastgele oturum kimliği; cookie `HttpOnly`, `Secure`,
  `SameSite=Strict`. Token `localStorage`/`sessionStorage`'a yazılmaz. Çıkışta oturum silinir.
- **Kaba kuvvet koruması:** art arda 5 başarısız girişten sonra hesap 15 dakika kilitlenir.
- **Varsayılan ret:** `/api/auth/login` dışındaki her uç nokta oturum ister; rol yetkisi `app/api`
  katmanında bir FastAPI dependency ile, uç nokta bazında açıkça verilir.
- **Personel** yalnızca kendi taleplerini, bakiyesini ve resmî tatilleri görür; yalnızca kendi
  adına talep oluşturur/iptal eder.
- **İK** tüm talepleri görür, onaylar/reddeder; personel kayıtlarını, bakiyeleri ve resmî
  tatilleri yönetir. Kendi talebini karara bağlayamaz (BR-4).
- Sahiplik kuralları (BR-4, BR-6) servis katmanında ayrıca doğrulanır — API kontrolüne tek başına
  güvenilmez.
- Yetkisi olmayan birinin başkasına ait bir kayda erişimi **404** döner (kaydın varlığı sızmaz).
- Kurumsal SSO kapsam dışı; eklenmesi ADR gerektirir.

## Dependencies
- Yeni bir runtime bağımlılığı planda gerekçesiyle önerilir ve insan onayıyla eklenir.
- Şartlar: lisans MIT / BSD / Apache-2.0 (başkası ADR ister), aktif bakım (son 12 ayda sürüm),
  bilinen açık yok.
- `pip-audit` full check profilinde çalışır; bulgu = kırmızı check.
- Frontend'e CDN'den script yüklenmez (`docs/architecture.md`, kural 6).

## Review lens
Security is a mandatory dimension of every independent review (see `prompts/review.md`), not a
separate afterthought phase.
