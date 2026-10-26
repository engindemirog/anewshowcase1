# Conventions

> Bootstrap'ta (2026-10-26) dolduruldu. Buradaki her kural ya araçla zorlanır (tercih edilen) ya
> da review'da kontrol edilir. Temenni buraya girmez.

## Language
<!-- Set at bootstrap (step 0) and mirrored in AGENTS.md. Changing it later is allowed — edit both
     places; existing documents are not translated retroactively. -->
- Chat language (interviews, sessions): Türkçe
- Document language (docs/, specs, plans, ADRs, review and verify reports): Türkçe
- Always English (protocol, not prose): ANEW core files, template headings and field labels,
  `Status:` values (Draft / Approved / In progress / Shipped), `Approved by / on:`, `Source:`.
- Code identifiers, branch names, commit messages: İngilizce. Kullanıcıya görünen arayüz metinleri
  ve API hata `message` alanları Türkçe; hata `code` alanları İngilizce.
- Agent messages (handoff summaries, refusals, `Next:` lines): chat language; command names, file
  paths and protocol fields verbatim.

## Language & framework versions
- Python **3.12** (`pyproject.toml` `requires-python = ">=3.12,<3.13"`); ortam ve kilit: **uv**
  (`uv.lock` commit edilir; check `--frozen` çalışır).
- FastAPI, Pydantic v2, SQLAlchemy **2.x** (2.0 stili: `select()`, `Mapped[...]`), SQLite.
- pytest, pytest-playwright (Chromium), ruff, mypy, import-linter, pip-audit.
- Arayüz: HTML5 + vanilla JavaScript **ES modules** (`<script type="module">`), build adımı ve
  npm yok; güncel Chromium/Edge/Firefox hedeflenir.

## Naming
- Python: PEP 8 — `snake_case` fonksiyon/değişken, `PascalCase` sınıf, `UPPER_SNAKE` sabit (ruff).
- Alan terimleri `docs/domain.md` tablosundaki İngilizce karşılıklarla adlandırılır
  (`LeaveRequest`, `LeaveBalance`…); eş anlamlı uydurulmaz (`Vacation`, `TimeOff` yok).
- Testler: `tests/` altında `test_<konu>.py`, fonksiyonlar `test_<davranış>`; iş kuralı testleri
  adında kuralı taşır (`test_br1_annual_request_rejected_when_balance_insufficient`).
- JavaScript: `camelCase`, `const`/`let` (`var` yok), dosya adları `kebab-case.js`.
- Branch ve commit: `docs/git.md`.

## Error handling
- Servisler iş kuralı ihlalinde `DomainError` alt sınıfı fırlatır; her alt sınıfın sabit bir
  İngilizce `code` değeri vardır (`INSUFFICIENT_BALANCE`, `OVERLAPPING_REQUEST`,
  `SELF_APPROVAL_FORBIDDEN`, …).
- `app/api` bunları **tek bir exception handler**'da HTTP'ye çevirir: kural ihlali / geçersiz durum
  geçişi 409, doğrulama 422, oturum yok 401, yetki 403, bulunamayan veya başkasına ait kayıt 404. Gövde her zaman
  `{"code": "...", "message": "..."}`.
- Beklenmeyen hata → 500 ve genel mesaj; stack trace, SQL ve iç detay kullanıcıya asla sızmaz,
  yalnızca log'a yazılır.
- `except Exception: pass` yasak; yakalanan hata ya ele alınır ya yeniden fırlatılır.

## Data rules
- İzin tarihleri `datetime.date` (takvim günü, saat yok); aralıklar iki uç dâhil.
- Zaman damgaları (`created_at`, denetim kaydı) **UTC**, timezone-aware `datetime`; arayüzde
  `Europe/Istanbul`'a çevrilir. Naive `datetime` yasak.
- İzin süresi, hak ve bakiye **tam sayı gün** (`int`); `float` yasak.
- ID'ler **UUID4**; sıralı tam sayı ID dışarıya açılmaz.
- Durum ve izin türü `StrEnum`; kodda serbest metin durum yazılmaz.

## Enforced by tooling
- **ruff** (lint + format): PEP 8, import sırası, kullanılmayan kod; `ruff format --check`.
- **mypy** (`strict = true`, `app/` için): tip tutarlılığı.
- **import-linter**: `docs/architecture.md` yasak bağımlılık kuralları 1–3.
- **pytest**: `docs/testing.md`.
- **pip-audit**: bilinen güvenlik açığı olan bağımlılık (yalnızca full profil).
- Review'da kontrol edilenler (henüz araç yok): Data rules, `innerHTML` yasağı, Error handling gövde
  biçimi, alan terimi tutarlılığı.
