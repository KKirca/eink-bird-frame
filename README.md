# eink-bird-frame

Bahçedeki kuşları sesinden tanıyıp, o türün 1800'lerden kalma gravürünü e-ink panele basan bir çerçeve.

[fugleramme](https://github.com/arnegiacomo/fugleramme) üzerine kuruluyor: tespit [BirdNET-Go](https://github.com/tphakala/birdnet-go)'nun, gravür eşleştirme ve sayfa dizimi fugleramme'nin işi. Bu depodaki iş, zinciri çalışır hale getirmek ve **panel sürücüsünü kendim yazmak.**

Bu bir derleme günlüğü. Her faz tamamlandıkça tikleniyor.

---

## Durum

| Faz | Konu | Donanım | Durum |
|:---:|---|:---:|:---:|
| 0 | Kurulum ve ilk çıktı | — | ☐ |
| 1 | Gerçek tespit zinciri, mikrofonsuz | — | ☐ |
| 2 | Panel soyutlama katmanı | — | ☐ |
| 3 | Donanım araştırması ve sipariş kararı | — | ☐ |
| — | **🛑 Buradan sonrası satın alma gerektiriyor** | | |
| 4 | Pi kurulumu ve ilk SPI teması | Pi + panel | ☐ |
| 5 | Gerçek panel backend'i | Pi + panel | ☐ |
| 6 | Çerçeve, servis, duvara asma | + çerçeve | ☐ |

**Zaman bütçesi:** 0-3 arası ~3 hafta, haftada ~5 saat.

---

## Faz 0 — Kurulum ve ilk çıktı

**Hedef:** fugleramme'yi kendi makinemde, sahte veriyle çalışır görmek. Elimde güncellenen bir PNG olması.

- [ ] `uv` kurulu mu, değilse kur
- [ ] fugleramme'yi klonla, `uv sync` çalıştır
- [ ] Sahte detector'ı başlat (`fugleramme-fake-detector`)
- [ ] Servisi başlat (`fugleramme-dev`), `localhost:8080` kiosk'u aç
- [ ] `/admin` sayfasını gez — hangi ayarlar var, mod seçenekleri ne
- [ ] `--preview` ile tek seferlik PNG üret, dosyayı aç

**Öğrenilecek:** Projenin veri akışı. Detector nereden okunuyor, hangi ayar neyi değiştiriyor, render ne kadar sürüyor.

**Çıktı:** Diskte bir PNG. Bundan sonraki her şey bu PNG'yi panele götürmekle ilgili.

**Notlar:** `notes/faz-0.md`

---

## Faz 1 — Gerçek tespit zinciri, mikrofonsuz

**Hedef:** Sahte detector'ı atıp gerçek BirdNET-Go'yu devreye almak — mikrofon satın almadan.

BirdNET-Go çevrimdışı dosya analizini destekliyor. Yani kayıtlı kuş seslerini ona analiz ettirip gerçek tespit üretebilirim. Mikrofon sadece sesi *toplama* işi; zincirin geri kalanı mikrofondan bağımsız.

- [ ] BirdNET-Go'yu Docker ile ayağa kaldır
- [ ] Wiki'den dosya/dizin analizi komutunu bul (README'de yazmıyor)
- [ ] [xeno-canto](https://xeno-canto.org/)'dan Türkiye kayıtları indir — serçe, karatavuk, saksağan, baştankara, kumru ile başla
- [ ] Kayıtları analiz ettir, tespitlerin veritabanına düştüğünü doğrula
- [ ] fugleramme'yi sahte yerine gerçek detector'a bağla (`--detector`)
- [ ] `fugleramme-check` ile detector'ın beklenen her şeyi döndürdüğünü doğrula
- [ ] **Ölç:** analiz sırasında CPU ve RAM kullanımı ne? Bu veri Faz 3'te Pi modelini seçerken lazım olacak.

**Öğrenilecek:** BirdNET'in güven eşiği (confidence threshold) nasıl çalışıyor, yanlış pozitifler nasıl görünüyor, sınıflandırma ne kadar pahalı.

**Çıktı:** Gerçek tespitlerle dolu bir sayfa + Pi kararı için somut ölçüm.

**Notlar:** `notes/faz-1.md`

---

## Faz 2 — Panel soyutlama katmanı

**Projenin asıl öğrenme kısmı burası.** Henüz sahip olmadığım panelin göstereceği görüntüyü, yazılımda üretiyorum.

**Hedef:** PNG'yi alıp bir panelin tam olarak basacağı hale getiren kodu yazmak, çıktıyı dosyaya basmak. Donanım gelince sadece arka uç değişecek, üst katman değişmeyecek.

- [ ] Aday panellerin parametrelerini çıkar: çözünürlük, renk paleti (kaç renk, hangi RGB değerleri), arayüz, yenileme süresi
- [ ] Bu parametreleri tek bir "panel profili" olarak kodla
- [ ] Ölçekleme yaz: PNG'yi panelin en-boy oranına oturt — kırpma mı, kenar boşluğu mu? Kararı ver ve nedenini not et
- [ ] fugleramme'nin `render/dither.py`'ını oku (50 satır), ne yaptığını anla
- [ ] Kendi dithering'ini yaz: Floyd–Steinberg hata yayılımı, hedef palete kuantalama
- [ ] Çıktıyı PNG olarak kaydet — "sanal panel"
- [ ] Orijinal ile sanal panel çıktısını yan yana karşılaştır, kayıp nerede?
- [ ] Arka ucu değiştirilebilir yap: `FileBackend` şimdi, `SpiBackend` Faz 5'te

**Öğrenilecek:** Renk kuantalama, hata yayılımı, e-ink'in neden sınırlı paleti olduğu, donanım soyutlaması. Hepsi doğrudan EE konusu.

**Çıktı:** Panel modülü, dosya arka ucuyla çalışır durumda. Donanım gelmeden önce yazılım tarafı biter.

**Notlar:** `notes/faz-2.md`

---

## Faz 3 — Donanım araştırması ve sipariş kararı

**Hedef:** Ne alacağıma karar vermek. Faz 1'in ölçümü ve Faz 2'nin panel profilleri bu kararı besliyor.

- [ ] Pi modeli: Faz 1'deki CPU ölçümüne göre. BirdNET Pi'de mi çalışacak, ayrı makinede mi?
- [ ] Türkiye'de temin edilebilen e-ink modülleri tara (Robot Sepeti, SAMM Market, Amazon.com.tr) — fiyat, stok, çözünürlük, renk sayısı
- [ ] Pimoroni Inky ile karşılaştır: fiyat farkı, kargo süresi, gümrük
- [ ] Mikrofon: ucuz USB ile başla mı, doğrudan Clippy EM272Z1 mi
- [ ] Toplam bütçeyi çıkar, karar ver
- [ ] Sipariş ver

**Çıktı:** Yolda olan bir kutu.

**Notlar:** `notes/faz-3.md`

---

## 🛑 Donanım duvarı

Yukarıdaki dört faz sıfır lira. Aşağısı kutuyu bekliyor.

---

## Faz 4 — Pi kurulumu ve ilk SPI teması

- [ ] Raspberry Pi OS yaz, SSH ile bağlan
- [ ] SPI'ı etkinleştir, pin eşlemesini çıkar
- [ ] Panelin üretici örnek kodunu çalıştır — ekranda *herhangi bir şey* göster
- [ ] Yenileme süresini ölç, busy pin'in ne işe yaradığını gör

**Beklenti:** Bu faz büyük ihtimalle tamamen hata ayıklamaya gidecek. Normal.

---

## Faz 5 — Gerçek panel backend'i

- [ ] Faz 2'nin `SpiBackend`'ini yaz
- [ ] Sanal panel çıktısıyla gerçek panel çıktısını karşılaştır — palet tahminim doğru muydu?
- [ ] Yenileme sıklığını ayarla (e-ink ömrü sınırlı, gereksiz yenileme yapma)

---

## Faz 6 — Çerçeve, servis, duvara asma

- [ ] systemd servisi, açılışta otomatik başlasın
- [ ] Çerçeve, paspartu kesimi, havalandırma
- [ ] Duvara as

---

## Kaynaklar

- [fugleramme](https://github.com/arnegiacomo/fugleramme) — MIT, görseller CC BY-SA 4.0
- [BirdNET-Go](https://github.com/tphakala/birdnet-go) — CC BY-NC-SA 4.0, ticari kullanım yok
- [xeno-canto](https://xeno-canto.org/) — test için kuş sesi kayıtları
