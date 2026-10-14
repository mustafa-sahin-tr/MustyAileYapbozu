<!-- Kaynak: docs/issues/aile-yapbozu.md (epic bölümü). GitHub'daki epic issue ile aynı metin; karar değişirse önce burası, sonra epic güncellenir. -->

# Aile Yapbozu — torunun gönderdiği fotoğraf, büyükannenin çözdüğü yapboz, bitince sesli mesaj

## Neden
Sahibi (2026-10-14): Fluffy Drop'un 6 ayda 1M+ indirmesi mekanikten değil dağıtım tarifinden geldi: "No Timer" başlık, 60+ kitle, Trivia kategorisi. Aynı kitle mağazada "jigsaw for seniors" ve "puzzles for seniors" arıyor ama bulduğu oyunlar küçük parça, küçük yazı, her seviye sonrası interstitial ve işlevsiz coin. Bu kitlenin asıl eksik motivasyonu içerik değil, bağ: çocuk ve torunlarından gelen bir fotoğraf, bitince dinlenen 15 saniyelik "Anneanne, bu Ege'nin ilk okul günü" mesajı. Aile Yapbozu, 60+ oyuncuya süre ve reklam olmadan büyük parçalı yapboz verir; parayı ödeyen ve içeriği üreten taraf 30–50 yaş aralığındaki çocuk/torundur. Ödeyen ile oynayan ayrıdır; bu yüzden reklam ekonomisi değil aile aboneliği ekonomisi kurulur.

Araştırma raporu ve tasarım taslakları: https://claude.ai/code/artifact/8ab0af47-8a63-4fac-8fd7-aad383f2fe81 (bölüm 5, fikir 3 — "Büyük ve Sakin" serisi) ve https://claude.ai/artifact/3VMA4d5qTn8GF89PDf5Uj9 (Aile Yapbozu · 60+, Aile Yapbozu · Gönder).

Portföydeki rolü: "Büyük ve Sakin" serisinin ilk uygulaması; `MustyCozyCore`'un ilk Firebase'li tüketicisi; 60+ arayüz sistemi, aile bağlantısı, Firebase veri katmanı ve TTS sarmalayıcısı bir sonraki iki uygulamada (Büyük Sudoku, Büyük Kelime Avı) aynen kullanılır ("seri motoru" — bu epic'te kapsam dışı, 24. story'de not).

## Hedef kitle ve konumlandırma
- **Oyuncu (60+):** çoğu Android, 2–5 yaşında orta/alt segment telefon veya hediye edilmiş tablet; gözlük, titreyen el, küçük yazıya tahammülsüzlük. Oyunu kurmaz, kurulur; çoğu zaman davet kodunu çocuğu girer.
- **Gönderen (30–50, çocuk/torun):** uygulamayı mağazada bulan, kuran, ödeyen ve içerik üreten taraf. Motivasyonu: uzaktaki anne/babayla ritüel kurmak, "aradım mı" vicdanını hafifletmek.
- Mağaza adı: **Family Jigsaw: Puzzles for Seniors** (EN; Play 30 karakter sınırı için kısa biçim 17. story'de) / **Aile Yapbozu: Büyükler İçin** (TR). Anahtar kelimeler: jigsaw for seniors, family puzzle, puzzles for seniors, büyükanne yapboz, no timer jigsaw.
- Konumlandırma cümlesi: "Fotoğrafı sen gönder, annen yapbozu çözsün, bitince sesini duysun. Süre yok. Reklam yok."
- Rakiplerden farkı: iki rol, aile içi içerik, sesli mesaj ödülü, 88 px dokunma hedefi, hiç interstitial, tek vurgu rengi. Yapboz oyunları arasında "içeriği aile üretir" tek biz.

## Core loop ve kurallar (v1 kararları)
1. **Aile:** gönderen aileyi kurar, oyuncunun adını girer ("Kime gönderiyorsun? → Ayşe"), 6 karakterli davet kodu/link üretir. Oyuncu cihazında kod girilince aileye katılır; rol o anda seçilir (Çözeceğim / Göndereceğim). Aile en fazla 5 üye (ücretsiz katmanda 3: 1 oyuncu + 2 gönderen).
2. **Gönderim:** fotoğraf seç → istemcide uzun kenar 1024 px, JPEG q80 → zorluk (Kolay 3×3 = 9, Orta 4×4 = 16, Zor 5×5 = 25 parça; oyuncunun son süresine göre öneri) → 12–30 sn sesli mesaj (AAC/M4A, ≤ 500 KB) → en fazla 80 karakter not → GÖNDER. Firestore `puzzles/{id}` `status: sent`.
3. **Oyuncu ana ekranı:** "Günaydın Ayşe Hanım", gönderen avatarıyla gelen yapboz kartı, tek büyük düğme ÇÖZ, altta "Süre yok · Reklam yok" satırı. Zorunlu eğitim yok; ilk yapboz Kolay gelir.
4. **Çözme:** parçalar alttaki 100 px tepside (tablette sağda); sürükle-bırak; doğru yuvanın 24 dp yakınına bırakınca yapışır; boş yuvalar kesikli çerçeve ve soluk "buraya"; yanlış yere bırakılan parça tahtada kalır (ceza yok). İPUCU GÖSTER (64 px yüksek) elde tutulan ya da bir sonraki parçanın yuvasını 3 kez yakıp söndürür ve TTS söyler: "Parçayı sağ üste bırak". Süre sayacı hiçbir ekranda yok.
5. **Tamamlama anı:** parça kenarları 600 ms'de silinir, fotoğraf netleşir; gönderenin sesli mesajı otomatik çalar (tekrar düğmesi büyük); not okunur; konfeti 2 sn (Reduce Motion açıkken yalnızca parıltı). Oyuncu "Teşekkür" düğmesine basarsa gönderene kalp + "annen yapbozu bitirdi · 14 dk" bildirimi gider (basmazsa da bitirdi bildirimi gider, kalp gitmez).
6. **Ücretsiz katman:** aile başına takvim haftasında (Pazartesi 00:00, oyuncunun saat dilimi) 2 yapboz gönderimi; sunucu tarafı (Cloud Function) sayar. Oyuncu tarafında hiçbir reklam ve hiçbir satın alma teklifi yok. Gönderen tarafında kota dolunca ve her 5. gönderimde yumuşak Aile Planı kartı; interstitial hiçbir tarafta yok.
7. **Aile Planı:** 3,99 USD/ay (29,99 USD/yıl), 5 üyeye kadar, sınırsız gönderim, tüm nostalji paketleri, reklamsız garantisi. Ödeyen gönderen; plan aile belgesine yazılır, tüm üyeler yararlanır.
8. **Ücretsiz içerik:** aile fotoğrafı yoksa telifsiz "nostalji" paketleri (Eski İstanbul, Çiçekler, Köy Hayatı; her pakette 12 fotoğraf; 1 paket ücretsiz) ve günün yapbozu (her gün 1, herkese aynı). Bunlarda sesli mesaj yok, "Bugünün yapbozu" notu var.
9. **Veri ve saklama:** fotoğraf ve ses yalnızca aile üyelerince okunur; oyuncu ya da gönderen siler; 90 gün sonra Cloud Function otomatik siler. Kişisel veri silme talebi uygulama içinden tek düğmeyle.
10. **Backend kararı:** UGS'nin blob depolaması olmadığı için Firebase (Anonymous Auth + Firestore + Storage + Cloud Functions + FCM + Crashlytics). Supabase değerlendirildi (Auth + Postgres + Storage + Edge Functions) ama Unity SDK olgunluğu, FCM ve Storage kuralları Firebase'i seçtirdi; ücretsiz katman soft launch hacmine yeter. UGS yine Analytics + Remote Config için Core paketinden gelir.
11. **Arayüz sistemi (60+):** Atkinson Hyperlegible; gövde 22–28 px, başlık 32–36 px; dokunma hedefi ≥ 88 px; kontrast ≥ 4,5:1 (metin 7:1 hedef); tek vurgu rengi (koyu turkuaz), kırmızı yalnızca hata; gizli jest yok, her eylem görünür bir düğme; Android geri tuşu her yerde bir geri gider.
12. **Cihaz:** Android min SDK 24, 2 GB RAM hedefi, tablet birinci sınıf (iPad/Android tablet, yatay ve dikey). iOS build soft launch sonrası.

## Kapsam dışı (v1)
- Video mesaj, sesli sohbet, oyuncunun sesli yanıtı (SONRA).
- Tamamlanan yapbozların albümü, baskı/paylaşım (SONRA).
- Büyük Sudoku / Büyük Kelime Avı ile seri geçişi (SONRA; motor bu epic'te çıkarılır ama paket ayrışmaz).
- Reklam: hiçbir yerleşim; Core `AdsService` bağlı ama `sender_ads_enabled=false` (Remote Config), soft launch verisine göre değerlendirilir.
- Döndürmeli parça, serbest şekilli (grid dışı) parça, 6×6 ve üstü.
- Çoklu aile üyeliği (bir kullanıcı tek ailede), gerçek zamanlı birlikte çözme.
- iOS çıkışı MVP'de yok.

## Başarı ölçütleri (kapılar)
- Soft launch (TR, UK, CA; Android; 14 gün): davet kabulü (kabul edilen davet / üretilen davet) ≥ %40; aktif aile başına haftalık gönderilen yapboz ≥ 2; Aile Planı dönüşümü (plan alan aile / en az bir kabul edilmiş daveti olan aile, 14 gün) ≥ %8; gönderilen yapbozun 72 saat içinde tamamlanma oranı ≥ %70; oyuncu tarafında çökme < %0,5 (eski cihazlar dahil).
- Kapı geçilirse: iOS build, yıllık fiyat testi, Canlı; Büyük Sudoku planı açılır.
- Kapı geçilmezse: davet kabulü düşükse 3. story (onboarding) iterasyonu; gönderim düşükse 10–12 (gönderen akışı) ve 21 (hatırlatma); dönüşüm düşükse 14 ve 18; iki hafta sonra yine geçilmezse rafa.

## Temalar ve story'ler (sıra)
**P1 · MVP**
- [ ] Aile Yapbozu 1: Core paketini bağla ve proje iskeleti
- [ ] Aile Yapbozu 2: Firebase projesi, veri modeli ve güvenlik kuralları
- [ ] Aile Yapbozu 3: Rol seçimi ve davet koduyla aile bağlantısı
- [ ] Aile Yapbozu 4: Fotoğraftan parça üretimi: 3×3/4×4/5×5 grid ve tab-boşluk maskeleri
- [ ] Aile Yapbozu 5: Sürükle-bırak, yakınlık yapıştırma ve parça tepsisi
- [ ] Aile Yapbozu 6: İPUCU GÖSTER ve sesli yönlendirme (TTS)
- [ ] Aile Yapbozu 7: Tamamlama anı: fotoğraf netleşir, sesli mesaj, konfeti
- [ ] Aile Yapbozu 8: 60+ arayüz sistemi, oyuncu ana ekranı ve tablet düzeni
- [ ] Aile Yapbozu 9: Gönderen: fotoğraf seçimi, küçültme ve yükleme
- [ ] Aile Yapbozu 10: Gönderen: zorluk seçimi, sesli mesaj kaydı ve küçük not
- [ ] Aile Yapbozu 11: Gönderim, durum takibi ve "bitirdi" geri bildirimi
- [ ] Aile Yapbozu 12: Bildirimler (FCM): "yeni yapboz" ve "bitirdi"
- [ ] Aile Yapbozu 13: Nostalji paketleri ve günün yapbozu
- [ ] Aile Yapbozu 14: Ücretsiz katman kotası ve yumuşak Premium önerisi
- [ ] Aile Yapbozu 15: KVKK/GDPR: aydınlatma, rıza, veri silme ve 90 gün saklama
- [ ] Aile Yapbozu 16: Analitik olay şeması ve kapı raporu
- [ ] Aile Yapbozu 17: Mağaza sayfası, ASO ve soft launch yapılandırması
**P2 · Soft launch**
- [ ] Aile Yapbozu 18: Aile Planı aboneliği ve geri yükleme
- [ ] Aile Yapbozu 19: Aile yönetimi: üyeler, birden fazla oyuncu, hesap bağlama
- [ ] Aile Yapbozu 20: Ayarlar, yazı boyutu kademeleri ve değerlendirme isteği
**P3 · Canlı**
- [ ] Aile Yapbozu 21: Gönderen hatırlatması ve haftalık aile özeti
**SONRA**
- [ ] Aile Yapbozu 22: Video mesaj
- [ ] Aile Yapbozu 23: Seri motoru: Büyük Sudoku'ya geçiş
- [ ] Aile Yapbozu 24: Sesli yanıt ve sesli sohbet
- [ ] Aile Yapbozu 25: Huzurevi ve bakım evi lisansı (B2B)
- [ ] Aile Yapbozu 26: Fotoğraf albümü (Anılar)

## Riskler ve açık sorular
- **İki taraflı soğuk başlangıç:** oyuncu, gönderen davet etmeden hiçbir şey yapamaz. Azaltma: nostalji paketleri ve günün yapbozu (13) davetsiz de açılır; oyuncu ekranında "Ailenden kod iste" değil, gönderen ekranında "Annenin telefonuna kur" akışı (3).
- **Efor toplamı:** P1'in 17 story'si yaklaşık 27–30 iş günü; "3 haftada MVP" Core paketinin ve Firebase şablonunun hazır olmasına bağlı. 13 ve 15'in bir bölümü soft launch'ın ilk haftasına kayabilir; 4–8 kaymaz.
- **Anonim hesap kaybı:** Firebase Anonymous Auth cihaza bağlı; gönderen telefon değiştirince aile ve abonelik erişimi kaybolur. v1'de davet koduyla yeniden katılım + Google Play abonelik geri yükleme; Google/Apple hesap bağlama 19. story.
- **Ses formatı:** Unity AAC çözmez; kayıt ve çalma için küçük yerel eklenti (Android MediaRecorder/MediaPlayer, iOS AVAudioRecorder/AVAudioPlayer) şart (10, 7). Editor'de WAV yedek.
- **Eski cihaz belleği:** 5×5'te 25 parça dokusu ≈ 10 MB; 2 GB cihazda kabul edilebilir ama fotoğraf 1024 px üstüne çıkmamalı (9).
- **Çocuk fotoğrafı ve KVKK:** gönderen üçüncü kişilerin fotoğrafını yüklüyor; rıza ekranı gönderen tarafında, "Kids" kategorisi değil, yaş beyanı 18+ (15). Hukuki metin şablonu dışarıdan alınır; açık soru: TR için VERBİS kaydı gerekiyor mu (yıllık 50 çalışan / 25 M TL eşiği altında muaf; doğrulanacak).
- **TTS kalitesi:** eski Android cihazlarda TR sesi yüklü olmayabilir; TTS yoksa metin balonuyla sessiz yedek (6).
- **Play başlık sınırı:** istenen EN ad 34 karakter; 30 karakterlik kısa biçim 17'de kararlaştırılır.

## Done when
P1 story'leri kapalı, 3 ülkede 14 günlük soft launch tamamlanmış, kapı raporu (davet kabulü, haftalık gönderim, dönüşüm, tamamlama, çökme) epic'e yorum olarak eklenmiş ve karar (Canlı + Büyük Sudoku / iterasyon / raf) yazılmış.
