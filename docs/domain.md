# Domain

> Bootstrap'ta (2026-10-26) dolduruldu. İş, insanlar ve ajanlar arasındaki ortak dil.
> Bir terim burada yoksa, yapay zekânın ona kendi anlamını uydurmasını bekleyin.

## Ubiquitous language
Kodda İngilizce karşılık kullanılır (`docs/conventions.md`, Naming); doküman ve konuşmada Türkçe terim.

| Term | Meaning | Notes / not to be confused with |
|---|---|---|
| Personel (`Employee`) | İzin talep eden şirket çalışanı | Bir İK kullanıcısı da personeldir; kendi izni için Personel olarak davranır |
| İK (`HR`) | Talepleri onaylayan/reddeden; personel kayıtlarını, bakiyeleri ve resmî tatilleri yöneten rol | v1'de yönetici (amir) rolü yoktur |
| İzin talebi (`LeaveRequest`) | Bir personelin belirli bir tarih aralığı (başlangıç–bitiş, iki uç dâhil) için izin isteği | Tarihler takvim günüdür; saat ve yarım gün yoktur |
| İzin türü (`LeaveType`) | `ANNUAL` (yıllık) · `EXCUSE` (mazeret) · `SICK` (hastalık, raporlu) · `UNPAID` (ücretsiz) | Bakiyeden yalnızca `ANNUAL` düşer |
| İzin bakiyesi (`LeaveBalance`) | Personelin içinde bulunulan yılda kullanabileceği kalan yıllık izin günü | Tam sayı gün; hiçbir zaman negatif değil |
| Yıllık izin hakkı (`Entitlement`) | Kıdeme ve yaşa göre bir yıl için tanınan yıllık izin günü (BR-7) | Bakiye ≠ hak: bakiye = hak − kullanılan |
| İş günü (`WorkingDay`) | Cumartesi, pazar ve resmî tatil olmayan gün | İzin süresi iş günüyle sayılır (BR-2) |
| Resmî tatil (`Holiday`) | İK'nın takvime girdiği, çalışılmayan gün | v1'de tam gün; arife yarım günü tam gün tatil sayılmaz, iş günüdür |
| Talep durumu (`LeaveStatus`) | `PENDING` → `APPROVED` / `REJECTED`; `PENDING` veya başlamamış `APPROVED` → `CANCELLED` | Başka geçiş yoktur |
| Denetim kaydı (`AuditEntry`) | Bir talebin her durum değişikliğinin kim, ne zaman, hangi durumdan hangisine bilgisi | Silinmez, güncellenmez |

## Business rules
- **BR-1** Yıllık izin (`ANNUAL`) talebi, talep edilen iş günü sayısı bakiyeyi aşıyorsa oluşturulamaz.
- **BR-2** Hafta sonları ve resmî tatiller izin süresinden düşülmez; süre = aralıktaki iş günü sayısı.
  İş günü içermeyen bir aralık için talep oluşturulamaz.
- **BR-3** Bir personelin `PENDING` veya `APPROVED` durumdaki talepleri tarih olarak çakışamaz.
- **BR-4** Kimse kendi talebini onaylayamaz veya reddedemez; İK personelinin talebini başka bir İK
  kullanıcısı karara bağlar.
- **BR-5** Onaylanan yıllık izin, iş günü sayısı kadar bakiyeden düşer. Başlangıç tarihinden önce
  iptal edilen onaylı izin bakiyeye iade edilir. Başlamış onaylı izin iptal edilemez.
- **BR-6** Başlangıç tarihi geçmişte olan talebi yalnızca İK girebilir.
- **BR-7** Yıllık izin hakkı (4857 s. İş Kanunu md. 53): kıdem 1 yıldan (dâhil) 5 yıla kadar 14 gün,
  5 yıldan (dâhil) 15 yıla kadar 20 gün, 15 yıl (dâhil) ve üzeri 26 gün. 18 yaşından küçük ve
  50 yaşından büyük personele 20 günden az verilemez. Kıdemi 1 yılı doldurmamış personelin
  yıllık izin hakkı yoktur.
- **BR-8** `APPROVED` veya `REJECTED` bir talebin tarihleri ve türü değiştirilemez; değişiklik =
  iptal + yeni talep.
- **BR-9** Onay akışı tek adımlıdır: `PENDING` bir talebi bir İK kullanıcısı onaylar veya reddeder.
  Ret gerekçesi zorunludur.

## Key domain invariants
- Bakiye hiçbir zaman negatif olmaz.
- Her durum değişikliği bir `AuditEntry` üretir (kim, ne zaman, önceki → sonraki durum); denetim
  kayıtları silinmez ve değiştirilmez.
- Bir talebin durumu yalnızca yukarıdaki geçişlerle değişir; `REJECTED` ve `CANCELLED` son durumdur.
- Hastalık izni için teşhis veya rapor içeriği saklanmaz (`docs/security.md`, KVKK).
