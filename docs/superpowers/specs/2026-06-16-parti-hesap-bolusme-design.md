# Parti Hesap Bölüşme — Tasarım Dokümanı

**Tarih:** 2026-06-16
**Durum:** Onay bekliyor

## Amaç

Cumartesi partilerindeki ortak masrafları kişiler arasında bölmek için çok basit,
şık görünümlü bir uygulama. Her masraf kalemi (tavuk, bira, viski...) sadece o kaleme
ortak olan kişiler arasında **eşit** bölünür. Böylece içki içmeyenlere içki parası
yazılmaz. Sonuçta her kişinin ödemesi gereken **toplam pay** gösterilir.

## Platform

- **Tek bir HTML dosyası** (`index.html`) — tüm HTML, CSS ve JavaScript içinde gömülü.
- Telefon tarayıcısında açılır; "Ana ekrana ekle" ile uygulama gibi kullanılabilir.
- İnternet gerektirmez, çevrimdışı çalışır (harici font/kütüphane yok, sistem fontu).
- Kurulum / derleme aracı gerekmez (Android Studio, SDK vs. yok).

## Ekran Düzeni (tek sayfa, yukarıdan aşağı)

```
┌────────────────────────────┐
│  🎉 Parti Hesabı   [EUR ▾]  │   <- başlık + para birimi seçici
├────────────────────────────┤
│ KİŞİLER                     │
│  [+ Kişi ekle: ____ ]       │
│  • Ali        [sil]         │
│  • Ayşe       [sil]         │
├────────────────────────────┤
│ MASRAFLAR                   │
│  [+ Masraf ekle]            │
│  ┌──────────────────────┐   │
│  │ Tavuk      400   [x] │   │
│  │ ☑Ali ☑Ayşe ☑Mehmet  │   │
│  ├──────────────────────┤   │
│  │ Viski      300   [x] │   │
│  │ ☑Ali ☐Ayşe ☑Mehmet  │   │
│  └──────────────────────┘   │
├────────────────────────────┤
│ SONUÇ                       │
│  Ali     €333.33            │
│  Ayşe    €133.33            │
│  Mehmet  €333.33            │
│  ─────────────────          │
│  Toplam  €700.00            │
├────────────────────────────┤
│         [Hepsini temizle]   │
└────────────────────────────┘
```

- Her masraf kalemi: ad alanı + tutar alanı + her kişi için bir tik kutusu (checkbox).
- Yeni kişi eklendiğinde her masraf kaleminde otomatik olarak yeni bir tik kutusu
  belirir (varsayılan: işaretli).
- Sonuç bölümü, kişiler/masraflar/tikler her değiştiğinde anında yeniden hesaplanır.

## Görsel Stil

- Koyu, modern "parti" teması: koyu arka plan, canlı vurgu (mor → neon gradyan).
- Yuvarlak köşeli kartlar, yumuşak gölgeler.
- Büyük, parmakla dokunması kolay butonlar ve tik kutuları.
- Sonuç bölümünde paylar iri ve net rakamlarla.
- Akıcı küçük geçişler (buton basışı, kart ekleme) — abartısız.

## Para Birimi

Başlıkta açılır menü ile seçim:

| Kod | Ad                | Gösterim örneği |
|-----|-------------------|-----------------|
| EUR | Euro (varsayılan) | `€333.33`       |
| INR | Hindistan Rupisi  | `₹333.33`       |
| MKD | Makedonya Dinarı  | `333.33 ден`    |

Seçilen birim tüm tutar ve sonuçlarda görünür, localStorage'a kaydedilir.

## Hesaplama Mantığı

- Her masraf kalemi, **yalnızca işaretli (ortak) kişiler** arasında eşit bölünür:
  `kişi başı pay = tutar / işaretli kişi sayısı`.
- Bir kişinin toplam payı = ortak olduğu tüm kalemlerdeki paylarının toplamı.
- Tüm tutarların toplamı "Toplam" satırında gösterilir.
- Değerler 2 ondalık ile gösterilir (örn. `333.33`).

## Veri Yapısı

Bellekte tutulan ve localStorage'a serileştirilen durum:

```js
state = {
  paraBirimi: "EUR",                        // "EUR" | "INR" | "MKD"
  kisiler: [ { id, ad } ],
  masraflar: [ { id, ad, tutar, ortaklar: [kisiId, ...] } ]
}
```

- `id`'ler basit artan sayı veya `crypto.randomUUID()`/`Date.now()` tabanlı.
- Her durum değişikliğinde `localStorage`'a otomatik kaydedilir; açılışta okunur.

## Hata / Uç Durumlar

- Tutar boş veya geçersiz (sayı değil) ise o kalem **0** sayılır; uygulama çökmez.
- Bir kalemde hiç kimse işaretli değilse o kalem kimseye bölünmez ve görsel olarak
  uyarı (kırmızımsı kenar/etiket) ile belirtilir.
- Bir kişi silinince, o kişi tüm masrafların `ortaklar` listesinden de çıkarılır.
- "Hepsini temizle" butonu önce onay sorar; onaylanırsa tüm durum sıfırlanır.

## Kapsam Dışı (YAGNI)

- Kim kime borçlu / kimin ödediğini takip etme (sadece kişi başı pay gösterilir).
- Ağırlıklı bölüşme (herkes eşit pay alır).
- Çoklu parti/geçmiş kaydı (tek aktif parti).
- Sunucu, hesap, giriş, senkronizasyon.

## Test Yaklaşımı

Tek dosya olduğu için manuel kabul testi:
1. 3 kişi ekle, 2 masraf gir (biri herkese, biri sadece 2 kişiye), payların doğru
   hesaplandığını doğrula.
2. Para birimini değiştir, sembolün her yerde güncellendiğini gör.
3. Sekmeyi kapat/aç, verilerin korunduğunu doğrula.
4. Kişi sil, ilgili tiklerin ve payların güncellendiğini doğrula.
5. Tutarı boş bırak / hiç kimseyi işaretleme — çökmeden uyarı gösterdiğini doğrula.
