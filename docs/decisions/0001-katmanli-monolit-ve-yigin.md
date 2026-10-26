# ADR 0001 — Katmanlı monolit; Python (FastAPI) + vanilla HTML/JS + SQLite

- Status: Accepted
- Date: 2026-10-26

## Context
Bir şirketin izin yönetim sistemi kuruluyor: personel izin talep eder, İK karara bağlar, yıllık
izin bakiyeleri kanuna göre tutulur. Tek ekip, tek ürün, greenfield. Proje sahibi backend için
Python'u, arayüz için framework'süz HTML + JavaScript'i ve modüler yapı yerine basit katmanlı
monoliti seçti (bootstrap mülakatı, 2026-10-26). Ajanlar `scripts/check`'i çok sık çalıştırdığı
için araç zinciri arkada süreç bırakmamalı (ANEW-0007).

## Decision
Uygulama tek bir FastAPI süreci olan **katmanlı monolittir** (`api → services → repositories →
models`); arayüz FastAPI'nin sunduğu build'siz HTML + vanilla JS'tir; veri SQLAlchemy 2 ile
SQLite'ta tutulur. Araçlar: Python 3.12, uv, pytest, ruff, mypy, import-linter, Playwright for
Python, pip-audit.

## Consequences
- Kazanç: tek dil backend, tek süreç, build adımı yok; SQLite sunucu süreci bırakmaz ve testte her
  çalıştırma temiz bir veritabanıyla başlar; FastAPI/Pydantic sınırlarda doğrulamayı hazır verir.
- Kazanç: katman kuralları import-linter ile check'te zorlanır — yapı kâğıt üstünde kalmaz.
- Maliyet: katmanlı yapı alan sınırlarını (izin, hak, takvim) kod düzeyinde ayırmaz; servisler
  zamanla iç içe geçebilir. Review bunu izler.
- Maliyet: SQLite eşzamanlı yazmada sınırlı; çok kullanıcılı canlı kullanımda PostgreSQL'e geçiş
  gerekebilir (SQLAlchemy geçişi kolaylaştırır).
- Maliyet: framework'süz JS'te bileşen yapısı ve durum yönetimi elle kurulur.

## Alternatives considered
- **Modüler monolit (employees / calendar / entitlement / leave)** — sınırlar daha net, ama proje
  sahibi v1 için daha az yapıyı tercih etti.
- **TypeScript (Fastify + React + SQLite)** — tek dil uçtan uca, ama proje sahibi Python ve
  framework'süz arayüzü seçti.
- **.NET 8 / Spring Boot + PostgreSQL** — kurumsal ve olgun, ama daha ağır; build daemon'ları ve
  veritabanı sunucusu residue ve test izolasyonu için ek iş getirir.

## Revisit triggers
- Servisler arasında döngüsel bağımlılık ya da tek bir servisin birden fazla alanın kuralını
  taşıması → modüler monolite geçişi değerlendir.
- Eşzamanlı kullanıcı sayısı veya "database is locked" hataları → PostgreSQL.
- Arayüz karmaşıklığı (çok sayfalı durum, yeniden kullanılan bileşenler) → bir UI kütüphanesi ADR'si.
