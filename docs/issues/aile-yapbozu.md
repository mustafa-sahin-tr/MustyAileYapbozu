<!-- game: Aile Yapbozu -->
<!-- repo: MustyAileYapbozu -->
# EPIC: Aile Yapbozu — torunun gönderdiği fotoğraf, büyükannenin çözdüğü yapboz, bitince sesli mesaj
labels: epic, enhancement

## Neden
Sahibi (2026-10-14): Fluffy Drop'un 6 ayda 1M+ indirmesi mekanikten değil dağıtım tarifinden geldi: "No Timer" başlık, 60+ kitle, Trivia kategorisi. Aynı kitle mağazada "jigsaw for seniors" ve "puzzles for seniors" arıyor ama bulduğu oyunlar küçük parça, küçük yazı, her seviye sonrası interstitial ve işlevsiz coin. Bu kitlenin asıl eksik motivasyonu içerik değil, bağ: çocuk ve torunlarından gelen bir fotoğraf, bitince dinlenen 15 saniyelik "Anneanne, bu Ege'nin ilk okul günü" mesajı. Aile Yapbozu, 60+ oyuncuya süre ve reklam olmadan büyük parçalı yapboz verir; parayı ödeyen ve içeriği üreten taraf 30–50 yaş aralığındaki çocuk/torundur. Ödeyen ile oynayan ayrıdır; bu yüzden reklam ekonomisi değil aile aboneliği ekonomisi kurulur.

PO kararı (2026-10-16): bağ yalnızca aileyle sınırlı değil. Veri modeli "aile" yerine **çember**dir: bir kullanıcı birden fazla çember kurar ya da davet koduyla katılır ve onlara ad verir ("Aile 👨‍👩‍👧", "Okul arkadaşları 🎒", "Komşular 🏡"). Çemberler kapalıdır; yalnızca davet koduyla girilir, yabancıdan fotoğraf gelmez. Mağaza konumlandırması v1'de aile ve 60+ odaklı kalır; arkadaş çemberlerinin payı soft launch'ta `circle_type` ile ölçülür.

Araştırma raporu ve tasarım taslakları: https://claude.ai/code/artifact/8ab0af47-8a63-4fac-8fd7-aad383f2fe81 (bölüm 5, fikir 3 — "Büyük ve Sakin" serisi) ve https://claude.ai/artifact/3VMA4d5qTn8GF89PDf5Uj9 (Aile Yapbozu · 60+, Aile Yapbozu · Gönder).

Portföydeki rolü: "Büyük ve Sakin" serisinin ilk uygulaması; `MustyCozyCore`'un ilk Firebase'li tüketicisi; 60+ arayüz sistemi, çember bağlantısı, Firebase veri katmanı ve TTS sarmalayıcısı bir sonraki iki uygulamada (Büyük Sudoku, Büyük Kelime Avı) aynen kullanılır ("seri motoru" — bu epic'te kapsam dışı, 24. story'de not).

## Hedef kitle ve konumlandırma
- **Oyuncu (60+):** çoğu Android, 2–5 yaşında orta/alt segment telefon veya hediye edilmiş tablet; gözlük, titreyen el, küçük yazıya tahammülsüzlük. Oyunu kurmaz, kurulur; çoğu zaman davet kodunu çocuğu girer.
- **Gönderen (30–50, çocuk/torun):** uygulamayı mağazada bulan, kuran, ödeyen ve içerik üreten taraf. Motivasyonu: uzaktaki anne/babayla ritüel kurmak, "aradım mı" vicdanını hafifletmek.
- **Arkadaş çemberi (ikincil):** 60+ oyuncunun kendi arkadaşları (komşu, eski iş arkadaşı, okul arkadaşı) ya da birbirine yapboz atan genç arkadaşlar. Arkadaş çemberinde herkes hem gönderir hem çözer. v1'de pazarlanmaz, ürün içinde "Arkadaş çemberi kur" ile açılır.
- Mağaza adı: **Family Jigsaw: Puzzles for Seniors** (EN; Play 30 karakter sınırı için kısa biçim 17. story'de) / **Aile Yapbozu: Büyükler İçin** (TR). Anahtar kelimeler: jigsaw for seniors, family puzzle, puzzles for seniors, büyükanne yapboz, no timer jigsaw.
- Konumlandırma cümlesi: "Fotoğrafı sen gönder, annen yapbozu çözsün, bitince sesini duysun. Süre yok. Reklam yok."
- Rakiplerden farkı: iki rol, aile ve arkadaş çemberlerinden gelen içerik, sesli mesaj ödülü, 88 px dokunma hedefi, hiç interstitial, tek vurgu rengi. Yapboz oyunları arasında "içeriği aile üretir" tek biz.

## Core loop ve kurallar (v1 kararları)
1. **Çember:** kullanıcı bir çember kurar; adını ve simgesini verir (öneriler: Aile 👨‍👩‍👧, Arkadaşlar 🎒, Komşular 🏡; tür `family`|`friends`|`other`), ilk alıcının adını girer ("Kime gönderiyorsun? → Ayşe") ve 6 karakterli davet kodu/link üretir. Kod girilince kişi o çembere katılır. Bir kullanıcı birden fazla çemberde olabilir; rol üyeliğe yazılır (`player` çözer, `sender` gönderir; arkadaş çemberinde varsayılan ikisi birden). Çember en fazla 5 üye; ücretsiz katmanda 3 üye ve kişi başına en fazla 2 kendi kurduğu çember (başkasının çemberine katılmak sınırsız). Çemberler kapalıdır: arama, keşfet, herkese açık profil yok.
2. **Gönderim:** fotoğraf seç → istemcide uzun kenar 1024 px, JPEG q80 → zorluk (Kolay 3×3 = 9, Orta 4×4 = 16, Zor 5×5 = 25 parça; oyuncunun son süresine göre öneri) → 12–30 sn sesli mesaj (AAC/M4A, ≤ 500 KB) → en fazla 80 karakter not → GÖNDER. Gönderimde önce çember, sonra çemberden tek alıcı seçilir (tek çember / tek oyuncu varsa adım atlanır). Firestore `puzzles/{id}` `{circleId, receiverId, status: sent}`.
3. **Oyuncu ana ekranı:** "Günaydın Ayşe Hanım", gönderen avatarıyla gelen yapboz kartı, tek büyük düğme ÇÖZ, altta "Süre yok · Reklam yok" satırı. Oyuncu çember değiştirmez: tüm çemberlerden gelen yapbozlar tek listede, her kartta küçük etiket ("Aile · Emre'den"). Zorunlu eğitim yok; ilk yapboz Kolay gelir.
4. **Çözme:** parçalar alttaki 100 px tepside (tablette sağda); sürükle-bırak; doğru yuvanın 24 dp yakınına bırakınca yapışır; boş yuvalar kesikli çerçeve ve soluk "buraya"; yanlış yere bırakılan parça tahtada kalır (ceza yok). İPUCU GÖSTER (64 px yüksek) elde tutulan ya da bir sonraki parçanın yuvasını 3 kez yakıp söndürür ve TTS söyler: "Parçayı sağ üste bırak". Süre sayacı hiçbir ekranda yok.
5. **Tamamlama anı:** parça kenarları 600 ms'de silinir, fotoğraf netleşir; gönderenin sesli mesajı otomatik çalar (tekrar düğmesi büyük); not okunur; konfeti 2 sn (Reduce Motion açıkken yalnızca parıltı). Oyuncu "Teşekkür" düğmesine basarsa gönderene kalp + "annen yapbozu bitirdi · 14 dk" bildirimi gider (basmazsa da bitirdi bildirimi gider, kalp gitmez).
6. **Ücretsiz katman:** gönderen başına takvim haftasında (Pazartesi 00:00, gönderenin saat dilimi) 2 yapboz gönderimi, tüm çemberleri toplamında (çember açarak kota çoğaltılamaz); sunucu tarafı (Cloud Function) sayar. Oyuncu tarafında hiçbir reklam ve hiçbir satın alma teklifi yok. Gönderen tarafında kota dolunca ve her 5. gönderimde yumuşak Aile Planı kartı; interstitial hiçbir tarafta yok.
7. **Aile Planı** (çalışma adı; nihai ad "Aile Planı" / "Çember Planı" / "Plus" 18. story'de, mağazada "Family" ASO için korunur): 3,99 USD/ay (29,99 USD/yıl), reklamsız garantisi. Plan satın alan kişiye (`users/{uid}.plan`) yazılır ve onun kurduğu tüm çemberleri kapsar: çember başına 5 üye, sınırsız çember, o çemberlere yapılan tüm gönderimler kotasız, o çemberlerin üyelerine tüm nostalji paketleri açık. Plan sahibinin kendi gönderimleri de her çemberde kotasız.
8. **Ücretsiz içerik:** aile fotoğrafı yoksa telifsiz "nostalji" paketleri (Eski İstanbul, Çiçekler, Köy Hayatı; her pakette 12 fotoğraf; 1 paket ücretsiz) ve günün yapbozu (her gün 1, herkese aynı). Bunlarda sesli mesaj yok, "Bugünün yapbozu" notu var.
9. **Veri ve saklama:** fotoğraf ve ses yalnızca yapbozun çemberinin üyelerince okunur; oyuncu ya da gönderen siler; 90 gün sonra Cloud Function otomatik siler. Kişisel veri silme talebi uygulama içinden tek düğmeyle.
10. **Backend kararı:** UGS'nin blob depolaması olmadığı için Firebase (Anonymous Auth + Firestore + Storage + Cloud Functions + FCM + Crashlytics). Supabase değerlendirildi (Auth + Postgres + Storage + Edge Functions) ama Unity SDK olgunluğu, FCM ve Storage kuralları Firebase'i seçtirdi; ücretsiz katman soft launch hacmine yeter. UGS yine Analytics + Remote Config için Core paketinden gelir.
11. **Arayüz sistemi (60+):** Atkinson Hyperlegible; gövde 22–28 px, başlık 32–36 px; dokunma hedefi ≥ 88 px; kontrast ≥ 4,5:1 (metin 7:1 hedef); tek vurgu rengi (koyu turkuaz), kırmızı yalnızca hata; gizli jest yok, her eylem görünür bir düğme; Android geri tuşu her yerde bir geri gider.
12. **Cihaz:** Android min SDK 24, 2 GB RAM hedefi, tablet birinci sınıf (iPad/Android tablet, yatay ve dikey). iOS build soft launch sonrası.

## Kapsam dışı (v1)
- Video mesaj, sesli sohbet, oyuncunun sesli yanıtı (SONRA).
- Tamamlanan yapbozların albümü, baskı/paylaşım (SONRA).
- Büyük Sudoku / Büyük Kelime Avı ile seri geçişi (SONRA; motor bu epic'te çıkarılır ama paket ayrışmaz).
- Reklam: hiçbir yerleşim; Core `AdsService` bağlı ama `sender_ads_enabled=false` (Remote Config), soft launch verisine göre değerlendirilir.
- Döndürmeli parça, serbest şekilli (grid dışı) parça, 6×6 ve üstü.
- Herkese açık çember, çember arama/keşfet, davetsiz mesaj; bir yapbozu tek seferde birden fazla alıcıya gönderme ("herkese gönder", SONRA); gerçek zamanlı birlikte çözme.
- iOS çıkışı MVP'de yok.

## Başarı ölçütleri (kapılar)
- Soft launch (TR, UK, CA; Android; 14 gün): davet kabulü (kabul edilen davet / üretilen davet) ≥ %40; aktif çember başına haftalık gönderilen yapboz ≥ 2; Aile Planı dönüşümü (plan alan kullanıcı / en az bir kabul edilmiş daveti olan çember kurucusu, 14 gün) ≥ %8; gönderilen yapbozun 72 saat içinde tamamlanma oranı ≥ %70; oyuncu tarafında çökme < %0,5 (eski cihazlar dahil). Tüm kapı metrikleri `circle_type` (aile / arkadaş / diğer) kırılımıyla da raporlanır; arkadaş çemberleri belirgin büyürse ayrı mağaza sayfası ya da "Arkadaş Yapbozu" varyantı değerlendirilir.
- Kapı geçilirse: iOS build, yıllık fiyat testi, Canlı; Büyük Sudoku planı açılır.
- Kapı geçilmezse: davet kabulü düşükse 3. story (onboarding) iterasyonu; gönderim düşükse 10–12 (gönderen akışı) ve 21 (hatırlatma); dönüşüm düşükse 14 ve 18; iki hafta sonra yine geçilmezse rafa.

## Temalar ve story'ler (sıra)
**P1 · MVP**
- [ ] Aile Yapbozu 1: Core paketini bağla ve proje iskeleti
- [ ] Aile Yapbozu 2: Firebase projesi, veri modeli ve güvenlik kuralları
- [ ] Aile Yapbozu 3: Rol seçimi, çember kurma ve davet koduyla bağlantı
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
- [ ] Aile Yapbozu 19: Çember yönetimi: üyeler, birden fazla oyuncu, hesap bağlama
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
- **Çoklu çember karmaşıklığı:** 60+ oyuncu için çember kavramı görünmez kalmalı (tek gelen kutusu, küçük etiket); çember yönetimi gönderen tarafındadır. Kota gönderen başına olduğu için çember açmak kota açığı değildir.
- **Anonim hesap kaybı:** Firebase Anonymous Auth cihaza bağlı; gönderen telefon değiştirince çember ve abonelik erişimi kaybolur. v1'de davet koduyla yeniden katılım + Google Play abonelik geri yükleme; Google/Apple hesap bağlama 19. story.
- **Ses formatı:** Unity AAC çözmez; kayıt ve çalma için küçük yerel eklenti (Android MediaRecorder/MediaPlayer, iOS AVAudioRecorder/AVAudioPlayer) şart (10, 7). Editor'de WAV yedek.
- **Eski cihaz belleği:** 5×5'te 25 parça dokusu ≈ 10 MB; 2 GB cihazda kabul edilebilir ama fotoğraf 1024 px üstüne çıkmamalı (9).
- **Çocuk fotoğrafı ve KVKK:** gönderen üçüncü kişilerin fotoğrafını yüklüyor; arkadaş çemberlerinde aile dışı kişiler de görür; rıza ekranı gönderen tarafında, "Kids" kategorisi değil, yaş beyanı 18+ (15). Hukuki metin şablonu dışarıdan alınır; açık soru: TR için VERBİS kaydı gerekiyor mu (yıllık 50 çalışan / 25 M TL eşiği altında muaf; doğrulanacak).
- **TTS kalitesi:** eski Android cihazlarda TR sesi yüklü olmayabilir; TTS yoksa metin balonuyla sessiz yedek (6).
- **Play başlık sınırı:** istenen EN ad 34 karakter; 30 karakterlik kısa biçim 17'de kararlaştırılır.

## Done when
P1 story'leri kapalı, 3 ülkede 14 günlük soft launch tamamlanmış, kapı raporu (davet kabulü, haftalık gönderim, dönüşüm, tamamlama, çökme; `circle_type` kırılımı) epic'e yorum olarak eklenmiş ve karar (Canlı + Büyük Sudoku / iterasyon / raf) yazılmış.

## STORY: Aile Yapbozu 1: Core paketini bağla ve proje iskeleti
labels: tema:backend, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **1. story'si** (epic #EPIC). Tema: Altyapı.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** Unity projesi, `MustyCozyCore` (`com.mtafasahin.cozycore`, monorepo `mtafasahin-unity-packages`) UPM bağımlılığı, sahne düzeni, UGS kurulumu, Android/tablet build ayarları

### Kullanıcı hikâyesi
Bir geliştirici olarak Aile Yapbozu projesinin ortak paketi (UGS, yerelleştirme, ayarlar, büyük yazı modu, Reduce Motion, geri tuşu) ilk günden kullanmasını isterim, böylece 60+ arayüzü ve Firebase katmanı dışında hiçbir altyapıyı yeniden yazmam.

### Kabul kriterleri
- [ ] **Senaryo: Temiz kurulum** · Verilen boş bir Unity 6000.6 projesi (URP 2D) · Eğer `MustyCozyCore` paketi git URL ile eklenirse · O zaman proje derlenir ve `Bootstrap` sahnesi açılınca UGS oturumu (anonim Player Accounts) kurulur.
- [ ] **Senaryo: Sahne akışı** · Verilen `Bootstrap → RoleGate → PlayerHome / SenderHome → Puzzle → Complete` ve `Send` sahneleri · Eğer uygulama açılırsa · O zaman 2 sn içinde rol ekranı ya da kayıtlı rolün ana ekranı görünür; geri tuşu ana ekranlarda uygulamadan çıkar, diğerlerinde bir geri gider.
- [ ] **Senaryo: Yerelleştirme** · Verilen TR ve EN tabloları · Eğer cihaz dili TR ise · O zaman tüm metinler TR, aksi halde EN; dil Ayarlar'dan değiştirilebilir; Atkinson Hyperlegible TR karakter atlası (ğ, ş, ı, İ) tam.
- [ ] **Senaryo: Build** · Verilen Android build ayarları · Eğer `release` build alınırsa · O zaman paket adı `com.mtafasahin.aileyapbozu`, min SDK 24, hedef SDK güncel, 64-bit, boyut < 50 MB, `android:largeHeap=false` ile 2 GB cihazda açılır.
- [ ] **Senaryo: Tablet** · Verilen 10" Android tablet · Eğer uygulama açılırsa · O zaman yatay ve dikey yönelim desteklenir, Canvas Scaler referansı 390×844 (telefon) ve 834×1194 (tablet) profillerine göre seçilir.
- [ ] **Senaryo: Remote Config** · Verilen UGS Remote Config · Eğer `snap_distance_dp` 24'ten 32'ye çekilirse · O zaman yeni oturumda değer uygulanır.

### Teknik görevler (Unity/C#)
- [ ] Repo iskeleti: `Assets/_Game/{Scripts,Prefabs,Art,Audio,Data,Fonts}`, `.gitignore`, `docs/game-design.md` (bu epic'in kuralları), `docs/analytics.md`, `docs/privacy.md` (15. story doldurur).
- [ ] `MustyCozyCore` bağımlılığı (UPM git URL, sürüm etiketi): `UgsBootstrap`, `Localization`, `SettingsScreen`, `BackButtonRouter`, `LargeTextMode`, `ReduceMotion`, `Haptics`, `AudioManager`; `AdsService` ve `IapService` bağlı ama reklam başlatılmaz.
- [ ] Sahne yöneticisi: `SceneFlow` (async yükleme, 200 ms perde), `AppState` (ScriptableObject tabanlı: userId, `circleIds`, son kullanılan çember, cihazdaki baskın rol — RoleGate'in hangi ana ekranı açacağı için).
- [ ] UGS: Player Accounts anonim giriş, Analytics, Remote Config, ortamlar (`dev`, `prod`). Cloud Save kullanılmaz (veri Firestore'da).
- [ ] Remote Config şeması: `snap_distance_dp`, `hint_blink_count`, `tts_rate`, `free_weekly_sends`, `free_owned_circles`, `free_circle_members`, `plan_circle_members`, `premium_card_every_n_sends`, `sender_ads_enabled`, `daily_puzzle_grid`.
- [ ] Font: Atkinson Hyperlegible (OFL) TextMeshPro atlası (Regular/Bold, 22–36 px için SDF 36 pt örnekleme), fallback Noto Sans.
- [ ] CI: GitHub Actions ile Android debug build (GameCI); artefakt APK; tablet ve telefon için iki emülatör profili.
- [ ] Kod stili: `.editorconfig`, Roslyn analizörleri, Assembly Definition dosyaları (Game, Game.Firebase, Game.Editor, Tests).

### Analitik
- `app_open` {first_open: bool, locale: string, build: int, device_class: "phone"|"tablet", ram_mb: int} — her açılışta
- `session_start` {role: string|null} — ana ekran görünür olduğunda

### Notlar
- Core paketi Daily Drop'ta kurulmuş olmalı; değilse bu story L olur.
- Karar: Firebase Unity SDK ayrı assembly'de (`Game.Firebase`), oyun kodu `ICircleBackend` arayüzüyle konuşur; Supabase'e geçiş gerekirse tek katman değişir.
- Reklam SDK'sı (AdMob; `CoreOptions.UseAds = false`) initialize edilmez; `sender_ads_enabled` false olduğu sürece hiçbir reklam çağrısı yapılmaz (APK boyutu ve 60+ güven mesajı).

## STORY: Aile Yapbozu 2: Firebase projesi, veri modeli ve güvenlik kuralları
labels: tema:backend, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **2. story'si** (epic #EPIC). Tema: Altyapı.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** Firebase projesi (`dev`, `prod`), Firestore şeması, Storage kuralları, `FirebaseBackend : ICircleBackend`, Cloud Functions repo klasörü

### Kullanıcı hikâyesi
Bir geliştirici olarak çember, yapboz ve kullanıcı verisinin yalnızca ilgili çemberin üyelerince okunabildiği, istemciden kandırılamayan bir veri katmanı isterim, böylece fotoğraf ve ses gibi hassas içerik güvenle saklanır.

### Kabul kriterleri
- [ ] **Senaryo: Anonim giriş** · Verilen ilk açılış · Eğer Bootstrap tamamlanırsa · O zaman Firebase Anonymous Auth ile `uid` alınır ve `users/{uid}` belgesi `{displayName: null, circleIds: [], plan: "free"|"plus", planUntil: null, tz, locale, weekKey, weekSends: 0, lastActiveAt, fcmTokens: []}` ile oluşur; sonraki açılışlarda aynı uid.
- [ ] **Senaryo: Şema** · Verilen Firestore · Eğer bir yapboz gönderilirse · O zaman `puzzles/{id}` `{circleId, senderId, receiverId, photoPath, pieces: 9|16|25, audioPath, audioSec, note, status: "sent"|"opened"|"completed", createdAt, openedAt, completedAt, durationS, hints, thanked}` alanlarını taşır; `circles/{circleId}` `{name, emoji, type: "family"|"friends"|"other", ownerId, memberIds: [uid], memberCount, createdAt}` ve `circles/{circleId}/members/{uid}` `{roles: ["player"|"sender"], displayName, addedBy, joinedAt}` biçimindedir (rol çember içi; aynı kullanıcı bir çemberde oyuncu, diğerinde gönderen olabilir).
- [ ] **Senaryo: Üye olmayan okuyamaz** · Verilen A çemberinin yapbozu · Eğer A'nın üyesi olmayan bir kullanıcı (başka çemberlerde ortak üyeleri olsa bile) `puzzles/{id}` ya da Storage dosyasını okumayı denerse · O zaman kural reddeder (Firebase Emulator testi).
- [ ] **Senaryo: Çoklu çember** · Verilen kullanıcı X, A ve B çemberlerinin üyesi · Eğer X kendi gelen kutusunu sorgularsa · O zaman iki çemberden gelen yapbozlar tek sorguda (`receiverId == X`) gelir; X, A'daki bir yapbozu B'nin bir üyesine gönderemez (alıcı aynı çemberin üyesi olmalı, kural testi).
- [ ] **Senaryo: Silme yetkisi** · Verilen bir yapboz · Eğer silen kişi `senderId` ya da `receiverId` ise · O zaman silme kabul edilir; diğer üyelerse reddedilir.
- [ ] **Senaryo: Storage sınırları** · Verilen yükleme · Eğer `photo.jpg` > 2 MB ya da `voice.m4a` > 500 KB ya da içerik tipi uymuyorsa · O zaman Storage kuralı reddeder.
- [ ] **Senaryo: Çevrimdışı** · Verilen bağlantı yok · Eğer oyuncu daha önce indirilmiş bir yapbozu açarsa · O zaman Firestore önbelleği ve yerel dosya önbelleğinden açılır; durum güncellemesi bağlantı gelince gider.
- [ ] **Senaryo: Ortam ayrımı** · Verilen `dev` build · Eğer çalışırsa · O zaman `google-services.json` dev projesine işaret eder; `prod` build'de prod (CI'da dosya değişimi).

### Teknik görevler (Unity/C#)
- [ ] Firebase projeleri `aileyapbozu-dev`, `aileyapbozu-prod`; Unity SDK paketleri: Auth, Firestore, Storage, Messaging, Crashlytics (Functions çağrısı için `Functions` paketi).
- [ ] `ICircleBackend` arayüzü: `SignInAsync`, `ListMyCirclesAsync`, `GetCircleAsync(id)`, `CreateCircleAsync(name, emoji, type)`, `JoinCircleAsync(code, roles)`, `CreatePuzzleAsync`, `ListPuzzlesAsync(receiverId)`, `UpdatePuzzleStatusAsync`, `UploadAsync(path, bytes, contentType)`, `DownloadAsync(path)`, `DeletePuzzleAsync`; `FirebaseBackend` uygulaması; `FakeBackend` (Editor ve testler).
- [ ] Firestore kuralları (`firestore.rules`): `isMember(circleId)` (`request.auth.uid in circles/{id}.memberIds`) ve `hasRole(circleId, role)` yardımcıları; `circles` okuma üye, yazma yalnızca Cloud Function (kurma/katılma/çıkarma sunucuda); `puzzles` okuma yalnızca çember üyesi, `create` yalnızca o çemberde `sender` rolü, `senderId == request.auth.uid` ve `receiverId` aynı çemberde `player`, `update` yalnızca `status/openedAt/completedAt/durationS/hints/thanked` alanları ve yalnızca `receiverId`, `delete` gönderen ya da alıcı.
- [ ] Storage kuralları (`storage.rules`): `circles/{circleId}/puzzles/{puzzleId}/{file}`; okuma çember üyesi (`firestore.get` ile `memberIds`); yazma boyut ve `contentType` (`image/jpeg`, `audio/mp4`) kontrolü.
- [ ] Firestore çevrimdışı kalıcılık açık; `LocalCache` sınıfı: `persistentDataPath/puzzles/{id}/{photo.jpg,voice.m4a}` indirme ve temizleme (silinen yapbozda dosyalar da silinir).
- [ ] Cloud Functions klasörü `functions/` (TypeScript, Node 20): `onPuzzleCreated`, `onPuzzleCompleted`, `createCircle`, `joinCircle`, `enforceWeeklyQuota`, `scheduledCleanup`, `deleteMyData` iskeletleri (gövdeler 3, 11, 12, 14, 15'te).
- [ ] Emulator Suite ile kural testleri (`firebase emulators:exec`), CI'da çalışır.
- [ ] Hata eşleme: Firebase hataları `BackendError {Network, Permission, NotFound, Quota}` enum'una çevrilir; UI metinleri 60+ dilinde ("İnternet yok gibi görünüyor. Tekrar deneyelim.").

### Analitik
- `backend_error` {op: string, code: string} — her başarısız çağrıda (kişisel veri yok)

### Notlar
- Karar: Firebase (Unity SDK olgun, FCM, Storage kuralları, ücretsiz katman soft launch hacmine yeter). Supabase alternatifi `docs/adr/0001-backend.md`'de kayıtlı.
- Karar: Cloud Save yok; tüm oyuncu durumu Firestore'da (çember verisiyle aynı yerde, tek kaynak).
- Karar (2026-10-16): `families` yerine `circles`; üyelik alt koleksiyonda, `memberIds` dizisi kural kontrolü ve `array-contains` sorgusu için tekrarlanır (en fazla 5 üye, dizi küçük kalır). Kota ve plan kullanıcı belgesinde (14, 18).
- Firestore belge kimlikleri rastgele; davet kodu belgenin kendisi değil `invites/{code}` koleksiyonu (3. story).
- Soft launch maliyet tahmini: 2.000 çember × haftada 2 yapboz × 400 KB ≈ 1,6 GB/hafta Storage çıkışı; ücretsiz katman (Spark) sınırları `docs/adr/0001-backend.md`'de; Blaze'e geçiş kapı sonrasına.

## STORY: Aile Yapbozu 3: Rol seçimi, çember kurma ve davet koduyla bağlantı
labels: tema:aile, P1, efor:L
milestone: MVP

**Aile Yapbozu** epic'inin **3. story'si** (epic #EPIC). Tema: Çember bağlantısı.

**Öncelik:** P1 · **Efor:** L · **Milestone:** MVP · **Dokunduğu yer:** `RoleGate` sahnesi, `CircleService`, `CreateCircleView`, `InviteCode`, Cloud Functions `createCircle` / `createInvite` / `joinCircle`, App Links

### Kullanıcı hikâyesi
Bir gönderen olarak annemin telefonuna uygulamayı kurup iki dakikada "Aile" çemberimize bağlamak, ayrıca okul arkadaşlarımla ayrı bir "Arkadaşlar" çemberi kurmak isterim; bir oyuncu olarak ise sadece büyük bir kod girip "Hoş geldin Ayşe Hanım" görmek isterim, böylece hiçbir form doldurmadan ilk yapboza ulaşırım.

### Kabul kriterleri
- [ ] **Senaryo: Rol ekranı** · Verilen ilk açılış · Eğer RoleGate görünürse · O zaman iki büyük kart (≥ 160 px yüksek): "Yapboz çözeceğim" ve "Yapboz göndereceğim"; altında küçük "Kodum var" bağlantısı; başka hiçbir şey. Seçim cihazın hangi ana ekranla açılacağını belirler; çember içi rol üyelikte tutulur.
- [ ] **Senaryo: Çember kurma** · Verilen "Göndereceğim" seçildi · Eğer gönderen kendi adını, çember adını ve simgesini (öneri çipleri: "Aile 👨‍👩‍👧", "Arkadaşlar 🎒", "Komşular 🏡"; tür çipten gelir, serbest ad `other`) ve ilk alıcının adını ("Kime gönderiyorsun?") girerse · O zaman `circles/{id}` oluşur, gönderen `owner` + `sender` olur ve 6 karakterli kod (ABCDEFGHJKLMNPQRSTUVWXYZ23456789 alfabesi, 7 gün geçerli) + `https://aileyapbozu.app/j/{kod}` linki gösterilir; "Annenin telefonuna kur" (aile) / "Arkadaşına gönder" (arkadaş) düğmesi link + mağaza adresini paylaşır.
- [ ] **Senaryo: Kod girişi (oyuncu)** · Verilen "Çözeceğim" seçildi · Eğer kod 6 haneli büyük klavyeden girilirse (harf tuşları ≥ 88 px) · O zaman `joinCircle` çağrılır, oyuncu o çembere `roles: ["player"]` ile eklenir, gönderenin yazdığı ad ("Ayşe") oyuncu adı olur ve "Hoş geldin Ayşe Hanım, Emre seni **Aile** çemberine ekledi" ekranı 1 düğmeyle (Başla) görünür.
- [ ] **Senaryo: İkinci çembere katılma** · Verilen Ayşe "Aile" çemberinde · Eğer komşusunun "Komşular" kodunu Ayarlar > "Kodum var" ya da ana ekrandaki küçük "Kod gir" ile girerse · O zaman ikinci çembere katılır; ana ekran değişmez, yeni çemberden gelen yapbozlar aynı listede "Komşular · Fatma'dan" etiketiyle görünür. Oyuncuya çember seçtirilmez.
- [ ] **Senaryo: Arkadaş çemberi rolleri** · Verilen `type: friends` çember · Eğer biri kodla katılırsa · O zaman varsayılan `roles: ["player","sender"]` (herkes gönderir ve çözer); aile çemberinde kodla katılan "Çözeceğim / Göndereceğim" sorusunu görür (kod sahibinin cihazındaki seçime göre önceden seçili).
- [ ] **Senaryo: Gönderenin çemberleri** · Verilen gönderen 2 çember kurmuş · Eğer SenderHome açılırsa · O zaman üstte çember çipleri (ad + simge + üye sayısı) ve "+ Yeni çember"; ücretsiz katmanda kendi kurduğu 3. çemberde "Aile Planı ile sınırsız çember" kartı ve kurma reddi (sunucuda da). Başkasının çemberine katılmak sınırsız.
- [ ] **Senaryo: Linkle katılım** · Verilen link tıklandı, uygulama yüklü · Eğer App Link açılırsa · O zaman kod otomatik dolar ve katılım ekranı çember adını gösterir ("Komşular çemberine katıl"); uygulama yüklü değilse web sayfası kodu büyük gösterir ve mağazaya yönlendirir; ilk açılışta pano kod kalıbına uyuyorsa alan önceden doldurulur.
- [ ] **Senaryo: Geçersiz kod** · Verilen süresi dolmuş ya da yanlış kod · Eğer girilirse · O zaman "Bu kod işe yaramadı. Emre'den yeni kod iste." (gönderen adı biliniyorsa) ve alan temizlenmez. Zaten üyesi olunan çemberin kodu girilirse "Zaten bu çemberdesin" ve o çembere gidilir.
- [ ] **Senaryo: Üye sınırı** · Verilen ücretsiz çemberde 3 üye · Eğer 4. kişi aynı kodla katılmak isterse · O zaman katılım reddedilir; katılan tarafa "Bu çember dolu, Emre'ye haber verdik" ve çember sahibine "Aile Planı ile 5 kişi" kartı; çember sahibinin planı varsa sınır 5.
- [ ] **Senaryo: Yeniden katılım** · Verilen oyuncu uygulamayı sildi ve yeniden kurdu · Eğer gönderen yeni kod üretip oyuncu girerse · O zaman eski oyuncu kaydı (`displayName` eşleşmesi + gönderen onayı) yeni uid ile birleştirilir; oyuncunun diğer çemberleri de aynı birleştirmeyle (her çemberin sahibi kendi onayını verir ya da oyuncu her kodu yeniden girer) geri gelir; açılmamış yapbozlar yeni uid'ye aktarılır.

### Teknik görevler (Unity/C#)
- [ ] `CircleService`: `CreateCircle(ownerName, circleName, emoji, type, firstReceiverName)`, `Join(code, roles)`, `MyCircles` (Firestore `circles where memberIds array-contains me` listener), `RolesIn(circleId)`; `AppState` cihaz rolü kalıcılığı (PlayerPrefs).
- [ ] Cloud Function `createCircle`: ücretsiz katmanda kendi kurduğu çember sayısı ≤ 2 kontrolü (`users/{uid}.plan`), `circles` + `members/{uid}` yazımı, `users.circleIds` arrayUnion.
- [ ] Cloud Function `joinCircle(code, roles, displayName?)`: `invites/{code}` doğrulama (expiresAt, circleId, maxUses), üye sayısı/sahip planı kontrolü (3 / 5), `members/{uid}` + `memberIds`/`memberCount` transaction, `users/{uid}.circleIds` arrayUnion; `createInvite` (yalnızca `sender` rolü; aynı çember için aktif kod tekrar kullanılır).
- [ ] `InviteCode`: alfabe, 6 karakter, çakışma kontrolü sunucuda; `RegexInviteDetector` pano kontrolü (tam eşleşme: 6 karakter, yalnızca A–Z ve 2–9; yalnızca ilk açılışta ve kullanıcı dokunuşuyla).
- [ ] Android App Links (`assetlinks.json`) ve iOS Universal Links için `aileyapbozu.app/j/*`; statik web sayfası (GitHub Pages) kod + çember adı yok (gizlilik) + mağaza linkleri.
- [ ] `RoleGate` UI: iki kart, büyük klavye (`BigKeypad` prefab; 60+ sisteminin ilk bileşeni, 8. story genişletir), karşılama ekranı; TTS "Hoş geldin Ayşe Hanım" (6. story servisi hazırsa).
- [ ] `CreateCircleView` (gönderen teması): ad alanı (en fazla 24 karakter), simge seçici (12 emoji), tür çipleri, ilk alıcı adı; SenderHome çember çipleri.
- [ ] Paylaşım: Core `ShareService` ile metin; aile: "Annem için Aile Yapbozu kurdum. Kod: ABC234 · {link}", arkadaş: "Sana bir yapboz göndermek istiyorum, Komşular çemberine katıl. Kod: ABC234 · {link}".
- [ ] Yeniden katılım birleştirme: `mergeMember` Cloud Function, çember sahibi tarafında onay kartı ("Ayşe yeniden katıldı, eski kayıtla birleştirilsin mi?").
- [ ] Emulator testi: üye sınırı, kurma sınırı, süresi dolmuş kod, zaten üye, iki çembere üyelik.

### Analitik
- `role_selected` {role: string} — kartla seçildiğinde
- `circle_created` {circle_id_hash: string, circle_type: string, owned_count: int} — çember oluştuğunda
- `invite_created` {via: "share"|"screen", circle_type: string} · `invite_accepted` {role: string, via: "code"|"link"|"clipboard", age_h: int, circle_type: string, circles_after: int} — Kapı metriği: `invite_accepted / invite_created ≥ %40`

### Notlar
- Gerçek dünya varsayımı: aile davetlerinin çoğu aynı odada, çocuk annesinin telefonuna kurarken girilir; bu yüzden kod ekranı yazdırılabilir büyüklükte (48 px) ve link ikincil. Arkadaş davetleri çoğunlukla linkle gelir.
- Gönderen kendi adını girer; oyuncunun adını da gönderen girer (oyuncu klavye kullanmaz). Hitap ("Hanım/Bey") TR'de seçimlik, EN'de yok. Ad çember başınadır (Ayşe bir çemberde "Anneannem", diğerinde "Ayşe" olabilir).
- Karar (2026-10-16): çember kapalıdır; çember adı davet sayfasında gösterilmez, yalnız uygulama içinde. Herkese açık çember, arama ve keşfet kapsam dışı.
- Açık soru kapandı: oyuncu "Çözeceğim" seçip kodu yoksa "Henüz kodun yok mu? Şimdilik günün yapbozunu çöz" → 13. story ekranı; kod isteme oyuncuya yüklenmez.

## STORY: Aile Yapbozu 4: Fotoğraftan parça üretimi: 3×3/4×4/5×5 grid ve tab-boşluk maskeleri
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **4. story'si** (epic #EPIC). Tema: Yapboz motoru.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `PuzzleGenerator`, `PieceDefinition`, `EdgeMask`, `PieceMeshBuilder`, `Piece` prefab

### Kullanıcı hikâyesi
Bir oyuncu olarak torunumun gönderdiği fotoğrafın gerçek yapboz parçaları gibi çıkıntılı-girintili, büyük ve net parçalara bölündüğünü görmek isterim, böylece ekranda bir "yapboz" tuttuğumu hissederim.

### Kabul kriterleri
- [ ] **Senaryo: Grid** · Verilen 1024×768 fotoğraf ve zorluk Orta · Eğer yapboz üretilirse · O zaman fotoğraf 4×4 = 16 parçaya bölünür; tahta ekranın en kısa kenarının %92'sini kaplar; 390 px telefonda parça ≈ 90 px, 834 px tablette ≈ 190 px.
- [ ] **Senaryo: Kenar maskeleri** · Verilen iki komşu parça · Eğer üretilirse · O zaman ortak kenarda biri tab (çıkıntı), diğeri boşluk taşır; tab yüksekliği parça kenarının %22'si, genişliği %34'ü; dış kenarlar düz; deterministik (`seed = puzzleId` hash) olduğundan aynı yapboz her açılışta aynı maskeyle gelir.
- [ ] **Senaryo: Doku kesimi** · Verilen parça · Eğer çizilirse · O zaman parça kendi grid hücresi + komşu tab payı kadar fotoğraf dokusunu taşır (UV ofset), piksel kayması ≤ 1 px; parça çevresinde 2 px koyu kenar çizgisi (kontrast).
- [ ] **Senaryo: Yönelim** · Verilen dikey (768×1024) fotoğraf · Eğer yapboz üretilirse · O zaman tahta dikey oranı korur, grid hücreleri kare değil fotoğraf oranında (en-boy 0,6–1,7 arası desteklenir; dışındakiler 9. story'de merkezden kırpılır).
- [ ] **Senaryo: Bellek** · Verilen 5×5 ve 1024 px fotoğraf, 2 GB cihaz · Eğer yapboz üretilirse · O zaman tek `Texture2D` (RGB565 değil, RGBA32 ≤ 4 MB) paylaşılır, parça başına ayrı doku yok; üretim < 400 ms (Profiler, ana iş parçacığında 2 kareden fazla bloklamaz).
- [ ] **Senaryo: Önizleme** · Verilen yapboz ekranı · Eğer "Fotoğrafı gör" düğmesine basılı tutulursa · O zaman tam fotoğraf %40 saydam tahtanın üstünde görünür, bırakınca kaybolur (zorunlu değil, ceza yok).

### Teknik görevler (Unity/C#)
- [ ] `PuzzleGenerator.Build(Texture2D photo, int grid, ulong seed) → PuzzleModel {pieces: PieceDefinition[], boardSize, cellSize}`; `PieceDefinition {row, col, edges: EdgeMask[4], uvRect, correctPos}`.
- [ ] `EdgeMask` enum {Flat, Tab, Slot}; xoshiro256** (Daily Drop'tan paylaşılan `Rng` sınıfı Core'a taşınır) ile her iç kenar için rastgele yön.
- [ ] `PieceMeshBuilder`: hücre dikdörtgeni + her kenar için 8 noktalı Bezier tab/slot profili → `Mesh` (üçgenleme `Triangulator` yardımcı sınıfı, ≈ 60 vertex/parça); UV fotoğraf uzayında; `MeshRenderer` + ortak `Material` (URP Unlit, tek doku).
- [ ] Kenar çizgisi: ikinci alt mesh ya da `LineRenderer` (karar: mesh'e 2 px dış halka eklemek, tek draw call).
- [ ] `Piece` prefab: `MeshFilter`, `MeshRenderer`, `PolygonCollider2D` (profilden), `Piece` bileşeni {definition, state: InTray|OnBoard|Snapped}.
- [ ] Doku yükleme: `ImageConversion.LoadImage` arka planda değil (ana iş parçacığı) ama `Texture2D` oluşturma tek seferde; 1024 px üstü yüklenirse `PhotoDownscaler` (9. story'yle paylaşılır) küçültür.
- [ ] Edit Mode testleri: komşu kenar tutarlılığı (tab ↔ slot), deterministik seed, 3 grid boyutunda vertex sayısı sınırı.
- [ ] Profil: Samsung Galaxy A10 / Redmi 8A sınıfı cihazda 5×5 üretim süresi ve bellek raporu `docs/perf.md`.

### Analitik
- `puzzle_generated` {pieces: int, aspect: float, gen_ms: int} — yapboz ilk açıldığında

### Notlar
- Seri motoru: `PuzzleGenerator` ve `PieceMeshBuilder` oyuna özgü değil; 23. story (SONRA) bunları `MustyBigCalm` paketine taşıyacak; şimdilik `Assets/_Game/Engine/` altında ayrı assembly.
- Karar: parça döndürme yok (v1); tüm parçalar doğru yönde gelir. 60+ için döndürme en büyük hayal kırıklığı kaynağı (rakip yorumları).
- Tab oranları Play modunda ayarlanır; tasarım taslağındaki "3×3 büyük parça" görünümü referans.

## STORY: Aile Yapbozu 5: Sürükle-bırak, yakınlık yapıştırma ve parça tepsisi
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **5. story'si** (epic #EPIC). Tema: Yapboz motoru.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `PieceDragController`, `SnapSystem`, `PieceTray`, `BoardSlots`, `PuzzleSession`

### Kullanıcı hikâyesi
Bir oyuncu olarak parçayı parmağımla tutup tahtaya götürdüğümde yaklaşık doğru yere bıraksam bile yerine "oturmasını" isterim, böylece elim titrese de yapbozu bitirebilirim.

### Kabul kriterleri
- [ ] **Senaryo: Tepsi** · Verilen yeni açılmış yapboz · Eğer ekran çizilirse · O zaman tüm parçalar alttaki 100 px yüksek yatay tepside (tablette yatayda sağ sütun, 160 px) karışık sırada, parça küçültülmeden (tepside %100 ölçek, 3×3'te 3 parça görünür, yatay kaydırma büyük oklarla ve parmakla), boş yuvalar tahtada kesikli çerçeve + soluk "buraya" yazısı (22 px).
- [ ] **Senaryo: Tutma** · Verilen tepsideki parça · Eğer parmak basılı tutulursa (bekleme yok, ilk temas) · O zaman parça parmağın 48 px üstünde taşınır (parmak görüşü kapatmasın), %108 ölçek, hafif gölge; Reduce Motion'da ölçek yok.
- [ ] **Senaryo: Yapışma** · Verilen taşınan parça · Eğer parçanın merkezi doğru yuvanın merkezine 24 dp'den yakın bırakılırsa (`snap_distance_dp`) · O zaman parça 120 ms'de yuvaya kayar, "tık" sesi + Light haptik, `Snapped` olur ve bir daha taşınamaz.
- [ ] **Senaryo: Yanlış yer** · Verilen taşınan parça · Eğer boş bir yuvanın yakınına ama yanlış yuvaya bırakılırsa · O zaman parça orada tahtada serbest durur (ceza, titreme ya da kırmızı yok), tekrar tutulabilir; yuvaya değil tahta dışına bırakılırsa tepsiye döner.
- [ ] **Senaryo: Tepsiye dönüş** · Verilen tahtada serbest duran parça · Eğer tepsiye sürüklenirse · O zaman tepsiye geri girer (boşluk açılır).
- [ ] **Senaryo: Kaldığı yer** · Verilen yarım yapboz · Eğer uygulama kapatılıp 1 gün sonra açılırsa · O zaman yerleşen parçalar yerinde, serbest parçalar konumunda, tepsi sırası aynı (`PuzzleSession` yerel kayıt).
- [ ] **Senaryo: İki parmak** · Verilen ikinci parmak ekrana değdi · Eğer bir parça taşınıyorsa · O zaman ikinci dokunuş yok sayılır; yakınlaştırma/kaydırma jesti yoktur.

### Teknik görevler (Unity/C#)
- [ ] `PieceDragController`: Unity Input System `Pointer`, tek aktif parça, `Physics2D.OverlapPoint` ile en üstteki parça; `EventSystem.IsPointerOverGameObject` ile düğme önceliği; sürükleme ofseti (+48 px).
- [ ] `SnapSystem.TrySnap(piece, dropPos)`: `Vector2.Distance(dropPos, correctPos) <= snapDistance` (dp → px: `Screen.dpi/160`); yapışma tween'i; `Snapped` sonrası collider kapanır, `sortingOrder` tahta katmanına.
- [ ] `BoardSlots`: yuva çerçeveleri (`SpriteRenderer`, 9-slice kesikli), "buraya" etiketi TMP; dolunca gizlenir.
- [ ] `PieceTray`: `ScrollRect` değil, kendi `TrayLayout` (parçalar world-space, tepsi maskeli `SortingGroup`); büyük ok düğmeleri (88 px) ve parmakla kaydırma; parça alınınca boşluk kapanır (160 ms).
- [ ] `PuzzleSession` modeli: `{puzzleId, placed: int[], freePositions: Dictionary<int, Vector2>, trayOrder: int[], hints: int, startedAt, elapsedS}`; `Save()` her yapışmada ve `OnApplicationPause`; JSON `persistentDataPath/sessions/{puzzleId}.json`. Süre yalnızca arka planda ölçülür (öneri ve analitik için), hiçbir yerde gösterilmez.
- [ ] Haptik ve ses: Core `Haptics.Light`, `AudioManager.Play("snap")`; sesler yumuşak (ahşap tık).
- [ ] Play Mode testi: 24 dp içinde bırakma yapışır, 25 dp dışı yapışmaz; kayıt/geri yükleme eşitliği.

### Analitik
- `piece_snapped` {puzzle_id_hash: string, placed: int, total: int, elapsed_s: int} — her yapışmada (3×3'te hepsi, 5×5'te her 5'te bir)
- `puzzle_resumed` {placed: int, total: int} — kaldığı yerden devam edildiğinde

### Notlar
- Yapışma mesafesi 24 dp başlangıç; soft launch verisinde yanlış bırakma oranı yüksekse 32'ye çıkar (Remote Config).
- Karar: parçalar tepside küçültülmez; 5×5'te tepsi yatay kaydırılır. Küçültme 60+ için ayırt edilemez parça demektir.
- Karar: elapsedS ölçülür ama gösterilmez; "Süre yok" vaadi arayüz vaadi, veri vaadi değil (15. story aydınlatma metninde yazılır).

## STORY: Aile Yapbozu 6: İPUCU GÖSTER ve sesli yönlendirme (TTS)
labels: tema:erisilebilirlik, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **6. story'si** (epic #EPIC). Tema: 60+ arayüz ve erişilebilirlik.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `HintSystem`, `TtsService` (Android/iOS sarmalayıcı, Core'a aday), `SlotNamer`, Puzzle HUD

### Kullanıcı hikâyesi
Bir oyuncu olarak takıldığımda büyük bir düğmeye basıp parçanın nereye gideceğini hem görmek hem duymak isterim ("Parçayı sağ üste bırak"), böylece kimseye sormadan, utanmadan devam edebilirim.

### Kabul kriterleri
- [ ] **Senaryo: Düğme** · Verilen Puzzle ekranı · Eğer çizilirse · O zaman sağ altta 64 px yüksek, tam genişlikte "İPUCU GÖSTER" düğmesi (26 px bold, vurgu rengi) her zaman görünür ve her zaman etkin; ipucu sayısı sınırsız, sayaç yok.
- [ ] **Senaryo: Elde parça varken** · Verilen bir parça taşınıyor ya da son dokunulan parça serbest · Eğer İPUCU GÖSTER'e basılırsa · O zaman o parçanın doğru yuvası `hint_blink_count` (3) kez 350 ms aralıkla vurgu rengiyle yanıp söner ve TTS yuva adını söyler.
- [ ] **Senaryo: Elde parça yokken** · Verilen hiçbir parça seçili değil · Eğer düğmeye basılırsa · O zaman tepsideki ilk parça %115 büyüyüp 2 kez zıplar (Reduce Motion'da yalnızca kenar vurgusu) ve TTS "Bu parça sol alta gidiyor" der; yuvası da yanıp söner.
- [ ] **Senaryo: Yuva adları** · Verilen 3×3 · Eğer ipucu verilirse · O zaman adlar: sol üst, üst orta, sağ üst, sol orta, tam orta, sağ orta, sol alt, alt orta, sağ alt; 4×4 ve 5×5'te "üstten 2. sıra, soldan 3." biçimi (TR/EN tablo).
- [ ] **Senaryo: TTS yok** · Verilen cihazda TR TTS sesi yüklü değil · Eğer ipucu verilirse · O zaman metin balonu (26 px) 3 sn gösterilir, hata yok; Ayarlar'da "Sesli yönlendirme" kapatılabilir.
- [ ] **Senaryo: Sesli mesajla çakışma** · Verilen sesli mesaj çalıyor (7. story) · Eğer TTS tetiklenirse · O zaman TTS beklemez, çalmaz (mesaj öncelikli); TTS çalarken müzik %30'a iner.
- [ ] **Senaryo: Karşılama** · Verilen oyuncu ana ekranı açıldı · Eğer Sesli yönlendirme açıksa · O zaman günde bir kez "Günaydın Ayşe Hanım. Emre sana bir yapboz göndermiş." söylenir (saat 12'den sonra "İyi günler").

### Teknik görevler (Unity/C#)
- [ ] `TtsService` arayüzü: `Speak(string, lang)`, `Stop()`, `IsAvailable(lang)`, `OnDone`; Android `android.speech.tts.TextToSpeech` için küçük Java eklentisi (`.aar`, `UtteranceProgressListener`), iOS `AVSpeechSynthesizer` Objective-C eklentisi; Editor'de Debug.Log yedek. `tts_rate` 0,9 (Remote Config).
- [ ] `HintSystem`: hedef parça seçimi (taşınan → son dokunulan serbest → tepsi ilk), `BlinkSlot(slot, count)` coroutine, `BouncePiece`; Reduce Motion dalları.
- [ ] `SlotNamer.Name(row, col, grid, locale)`: 3×3 özel tablo, diğerleri satır/sütun kalıbı; yerelleştirme anahtarları `hint.slot.*`.
- [ ] HUD: `HintButton` prefab (64 px, tam genişlik, `LargeButton` bileşeni 8. story'de genelleşir), "Fotoğrafı gör" basılı tut düğmesi (4. story), geri düğmesi (sol üst 88 px "Ana ekran").
- [ ] Ses öncelik yöneticisi: `AudioFocus {VoiceMessage > Tts > Sfx > Music}`; Android `AudioManager` ducking.
- [ ] TTS dil kontrolü açılışta; yoksa `tts_available=false` ve metin balonu modu.

### Analitik
- `hint_used` {puzzle_id_hash: string, placed: int, total: int, mode: "tts"|"text"} — her basışta
- `tts_availability` {lang: string, available: bool} — açılışta bir kez

### Notlar
- İlke: ipucu asla kısıtlanmaz, satılmaz, reklam karşılığı değildir. Kapı değil köprü.
- Açık soru: iOS'ta TTS için `AVAudioSession` kategorisi Unity'nin ses oturumuyla çakışabilir; eklentide `.playback` + `mixWithOthers`.
- Karşılama cümlesi günde bir; sürekli konuşan uygulama rahatsız eder.

## STORY: Aile Yapbozu 7: Tamamlama anı: fotoğraf netleşir, sesli mesaj, konfeti
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **7. story'si** (epic #EPIC). Tema: Yapboz motoru.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Complete` sahnesi/overlay, `CompletionDirector`, `VoicePlayer` (yerel eklenti), `ConfettiFx`, "Teşekkür" düğmesi

### Kullanıcı hikâyesi
Bir oyuncu olarak son parçayı yerleştirince fotoğrafın bütünleşip netleştiğini görmek, ardından torunumun sesini duymak ve notunu okumak isterim, böylece yapbozun ödülü bir skor değil ailem olsun.

### Kabul kriterleri
- [ ] **Senaryo: Netleşme** · Verilen son parça yapıştı · Eğer tamamlama başlarsa · O zaman girdi kilitlenir, 600 ms'de parça kenar çizgileri ve tab profilleri solup düz fotoğraf kalır (mesh üstüne tam fotoğraf quad'ı fade-in), tahta %4 büyür; Reduce Motion'da 200 ms düz geçiş, büyüme yok.
- [ ] **Senaryo: Konfeti** · Verilen netleşme bitti · Eğer Reduce Motion kapalıysa · O zaman 2 sn konfeti (≤ 120 parçacık, vurgu rengi + beyaz) ve "Bitti!" (36 px); Reduce Motion açıksa yalnızca kenarlarda 1 sn parıltı.
- [ ] **Senaryo: Sesli mesaj** · Verilen yapbozda `audioPath` var ve dosya indirilmiş · Eğer konfeti başlarsa · O zaman 800 ms sonra gönderenin avatarı + adı + "Emre'den sesli mesaj" kartı açılır ve mesaj otomatik çalar; kartta 88 px "Tekrar dinle" düğmesi ve dalga animasyonu (Reduce Motion'da statik çubuk).
- [ ] **Senaryo: Ses dosyası yok** · Verilen indirme başarısız ya da mesaj yok · Eğer tamamlanırsa · O zaman kart yalnızca notu gösterir; "Mesaj gelmedi, tekrar dene" düğmesi (indirme yeniden); nostalji/günün yapbozunda "Bugünün yapbozunu bitirdin" metni.
- [ ] **Senaryo: Not** · Verilen 80 karakterlik not · Eğer kart açılırsa · O zaman 28 px'te, sesli mesajın altında, tam görünür (kaydırma gerekmez; 80 karakter sınırı buna göre).
- [ ] **Senaryo: Teşekkür** · Verilen kart açık · Eğer "Teşekkür ederim" düğmesine (88 px) basılırsa · O zaman `thanked: true` yazılır, düğme "Gönderildi ♥" olur, gönderene kalp bildirimi (12. story); basılmasa da `status: completed` ve `completedAt`, `durationS`, `hints` yazılır.
- [ ] **Senaryo: Çıkış** · Verilen kart · Eğer "Ana ekran" basılırsa · O zaman PlayerHome açılır; tamamlanan yapboz "Bitirdiklerin" listesinde kalır ve fotoğraf + mesaj oradan yeniden açılabilir.

### Teknik görevler (Unity/C#)
- [ ] `CompletionDirector`: sıralı coroutine (lock → reveal → confetti → voice card), Reduce Motion dalları; `PuzzleSession.Complete()`.
- [ ] Netleşme: tam fotoğraf `SpriteRenderer` quad tahta üstünde alpha 0 → 1; parça materyali `_EdgeStrength` 1 → 0 (shader parametresi; URP Unlit'e basit kenar maskesi eklenir).
- [ ] `VoicePlayer` eklentisi: Android `MediaPlayer` (M4A/AAC), iOS `AVAudioPlayer`; Unity tarafı `IVoicePlayer {Play(path), Stop, OnDone, Duration}`; Editor WAV yedeği. Unity `AudioSource` AAC çözmediği için zorunlu.
- [ ] `ConfettiFx` (`ParticleSystem`, havuzlu), parıltı varyantı; `Haptics.Medium` tamamlanınca.
- [ ] Sesli mesaj kartı UI (`VoiceCard` prefab): avatar (gönderen baş harfi + renk), dalga (`AudioWaveformView` basit 24 çubuk, gerçek analiz yok), "Tekrar dinle", not alanı, "Teşekkür ederim", "Ana ekran".
- [ ] Firestore güncellemesi: `UpdatePuzzleStatusAsync(id, completed, {completedAt, durationS, hints})`; çevrimdışıysa kuyruk (2. story).
- [ ] "Bitirdiklerin" listesi veri kaynağı: `status == completed` ve `receiverId == me`, son 20 (8. story UI).

### Analitik
- `puzzle_completed` {puzzle_id_hash: string, pieces: int, duration_s: int, hints: int, source: "circle"|"pack"|"daily", circle_type: string|null, voice: bool} — tamamlanınca
- `voice_played` {replay: bool, sec: int} · `thanks_sent` — düğmelerde

### Notlar
- Kapı metriği girdisi: `puzzle_completed` zamanı − `puzzle_created` ≤ 72 saat oranı ≥ %70.
- Tasarım taslağı: "bitince açılan sesli mesaj" kartı; vurgu rengi yalnızca Tekrar dinle ve Teşekkür'de.
- Konfeti "yoğunluk artsın süre uzamasın" ilkesine uyar: 2 sn, sonra ekran sakin.

## STORY: Aile Yapbozu 8: 60+ arayüz sistemi, oyuncu ana ekranı ve tablet düzeni
labels: tema:erisilebilirlik, P1, efor:L
milestone: MVP

**Aile Yapbozu** epic'inin **8. story'si** (epic #EPIC). Tema: 60+ arayüz ve erişilebilirlik.

**Öncelik:** P1 · **Efor:** L · **Milestone:** MVP · **Dokunduğu yer:** `BigCalmTheme` (SO), `LargeButton`, `LargeCard`, `PlayerHome` sahnesi, `ResponsiveLayout`, tüm oyuncu tarafı ekranlar

### Kullanıcı hikâyesi
Bir 60+ oyuncu olarak her ekranda büyük yazı, büyük düğme, tek anlaşılır vurgu rengi ve "ne yapacağım" sorusuna tek cevap görmek isterim, böylece gözlüksüz ve yardım almadan kullanabileyim.

### Kabul kriterleri
- [ ] **Senaryo: Tema sabitleri** · Verilen `BigCalmTheme` · Eğer herhangi bir oyuncu ekranı çizilirse · O zaman gövde metni ≥ 22 px (varsayılan 24), başlık 32–36 px, düğme etiketi 26 px bold, tüm etkileşimli hedefler ≥ 88×88 px, metin/zemin kontrastı ≥ 4,5:1 (gövde 7:1), tek vurgu rengi `#0E7C86`, kırmızı yalnızca hata; otomatik denetim (`ThemeLint` Editor aracı) ihlal listesi boş.
- [ ] **Senaryo: Ana ekran** · Verilen oyuncu, 1 yeni yapboz · Eğer PlayerHome açılırsa · O zaman üstte "Günaydın Ayşe Hanım" (36 px, saate göre selam), gönderen avatarı + "Emre'den yeni yapboz" kartı ve küçük çember etiketi ("Aile 👨‍👩‍👧", 22 px, soluk; oyuncu tek çemberdeyse gizli) (fotoğraf bulanık önizleme 40 px blur, zorluk etiketi "9 parça"), tek birincil düğme ÇÖZ (tam genişlik, 72 px), altında "Süre yok · Reklam yok" satırı (22 px, soluk); en altta iki ikincil kart: "Bitirdiklerin" ve "Hazır yapbozlar"; Ayarlar dişlisi sağ üst 88 px.
- [ ] **Senaryo: Birden fazla yapboz** · Verilen 3 yeni yapboz · Eğer PlayerHome açılırsa · O zaman en yeni kart büyük, diğerleri altında liste (her satır 96 px, gönderen adı + çember etiketi + "2 gün önce"); farklı çemberlerden gelenler aynı listede tarihe göre; çember sekmesi ya da filtresi yok; hepsi tek dokunuşla açılır.
- [ ] **Senaryo: Yapboz yok** · Verilen hiç yapboz gelmemiş · Eğer PlayerHome açılırsa · O zaman "Henüz yapboz gelmedi" + "Hazır yapbozlardan birini çöz" tek düğme; boş ekran yok.
- [ ] **Senaryo: Tablet yatay** · Verilen 10" tablet yatay · Eğer Puzzle açılırsa · O zaman tahta solda (ekran yüksekliğinin %90'ı), tepsi sağda dikey sütun (160 px), İPUCU GÖSTER sağ altta; dikeyde telefon düzeni ölçeklenir (tepsi altta 140 px).
- [ ] **Senaryo: Sistem yazı boyutu** · Verilen Android "En büyük" yazı boyutu · Eğer ekranlar çizilirse · O zaman metinler bir kademe daha büyür (`LargeTextMode` +4 px), hiçbir düğme etiketi kırpılmaz ve hiçbir düğme ekran dışına taşmaz (ScrollRect ile).
- [ ] **Senaryo: Geri ve ana ekran** · Verilen herhangi bir oyuncu ekranı · Eğer Android geri tuşuna basılırsa · O zaman bir geri gider, Puzzle'da "Ana ekrana dönülsün mü?" (iki düğme: Devam et / Ana ekran); hiçbir ekranda gizli jest (kaydırarak geri, uzun basma) gerekmez.
- [ ] **Senaryo: Çift dokunma koruması** · Verilen ÇÖZ düğmesi · Eğer 400 ms içinde iki kez dokunulursa · O zaman ikinci dokunuş yok sayılır (titreyen el).

### Teknik görevler (Unity/C#)
- [ ] `BigCalmTheme` ScriptableObject: font varlıkları, boyut kademeleri (S/M/L: 22/24/28 gövde), renk paleti (zemin `#FBF7F0`, metin `#1F1F1F`, vurgu `#0E7C86`, hata `#B3261E`), minimum hedef 88, kenar yarıçapı 20; `ThemeApplier` bileşeni tüm TMP ve Button'lara uygular.
- [ ] `LargeButton` (uGUI Button üstüne: min boyut zorunluluğu, 400 ms debounce, basılı durumu %96 ölçek ya da renk koyulaşması, `Haptics.Light`), `LargeCard`, `LargeListRow` prefab'ları.
- [ ] `ThemeLint` Editor aracı: sahnedeki tüm `Selectable` boyutlarını, TMP boyutlarını ve kontrastı (WCAG formülü) tarar, Console'a rapor; CI'da Edit Mode testi olarak koşar.
- [ ] `ResponsiveLayout`: `device_class` (en kısa kenar ≥ 600 dp → tablet) ve yönelime göre `LayoutGroup` anahtarlama; Puzzle için iki prefab varyantı değil, aynı prefab + `LayoutProfile` SO.
- [ ] `PlayerHome` sahnesi: `GreetingView` (saate göre TR/EN selam), `PuzzleInboxView` (Firestore listener `receiverId == me && status != completed`, `createdAt desc`; tüm çemberler tek sorgu, çember adı/simgesi `CircleService.MyCircles` önbelleğinden), `CompletedListView`, `PacksEntryCard`; önizleme blur (`Texture2D` 64 px küçültme + Gaussian, blur çözüm görünmesin diye).
- [ ] Core `LargeTextMode` ile `BigCalmTheme` kademe bağlantısı; `ReduceMotion` okunur.
- [ ] Canvas: tek `Canvas` (Screen Space Overlay) + `CanvasScaler` match 0,5; Safe Area.
- [ ] Erişilebilirlik: Android TalkBack için uGUI'de `AccessibilityNode` yok; yerine her düğmede TTS "dokununca etiketi söyle" seçeneği Ayarlar'da (20. story), bu story'de altyapı (`SpeakOnFocus`).

### Analitik
- `home_view` {role: "player", inbox: int, completed: int, device_class: string} — her açılışta
- `ui_text_scale` {level: string} — oturum başında

### Notlar
- Tasarım taslağı: "Aile Yapbozu · 60+" artboard'u birebir referans (Günaydın Ayşe Hanım, avatar, 3×3, "buraya", 100 px tepsi, İPUCU GÖSTER, "Süre yok · Reklam yok").
- Bu story'nin çıktısı seri motorunun yarısıdır (tema + bileşenler); 23. story taşır.
- Başlamadan refine: PlayerHome ve Puzzle için tablet mockup'ı, onay, sonra kod.

## STORY: Aile Yapbozu 9: Gönderen: fotoğraf seçimi, küçültme ve yükleme
labels: tema:gonderen, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **9. story'si** (epic #EPIC). Tema: Gönderen akışı.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Send` sahnesi adım 1, `PhotoPicker` (yerel eklenti), `PhotoDownscaler`, `CropView`, `UploadService`

### Kullanıcı hikâyesi
Bir gönderen olarak galeriden bir fotoğraf seçip (gerekirse kadrajlayıp) saniyeler içinde yüklemek isterim, böylece akış WhatsApp'a fotoğraf atmak kadar kısa olsun.

### Kabul kriterleri
- [ ] **Senaryo: Seçim** · Verilen Send adım 1 · Eğer "Fotoğraf seç" (tam genişlik düğme) basılırsa · O zaman Android Photo Picker (API 33+) ya da `ACTION_GET_CONTENT` (API 24–32) açılır, depolama izni istenmez; iOS'ta `PHPickerViewController`; "Fotoğraf çek" ikinci seçenek (kamera izni yalnızca o anda).
- [ ] **Senaryo: Küçültme** · Verilen 4032×3024, 3,8 MB HEIC/JPEG · Eğer seçilirse · O zaman istemcide EXIF yönelimi düzeltilir, uzun kenar 1024 px'e küçültülür, JPEG q80 olarak kodlanır (≈ 150–300 KB), EXIF (konum dahil) temizlenir; işlem < 1,5 sn orta sınıf cihazda.
- [ ] **Senaryo: Kadraj** · Verilen en-boy oranı 0,6–1,7 dışı (panorama) · Eğer seçilirse · O zaman `CropView` açılır: oran 4:3 / 3:4 / 1:1 seçimi ve parmakla kaydırma; içindekiler için kadraj isteğe bağlı ("Kadrajla" ikincil düğme).
- [ ] **Senaryo: Önizleme** · Verilen küçültülmüş fotoğraf · Eğer adım 1 çizilirse · O zaman fotoğraf büyük önizlemede, "Değiştir" küçük bağlantı, "Devam" birincil düğme; fotoğraf seçilmeden Devam pasif.
- [ ] **Senaryo: Yükleme** · Verilen GÖNDER'e basıldı (11. story) · Eğer yükleme başlarsa · O zaman `circles/{cid}/puzzles/{pid}/photo.jpg` Storage'a `contentType image/jpeg` ile gider, ilerleme çubuğu, 3 deneme (üstel bekleme 1/2/4 sn), başarısızlıkta taslak korunur.
- [ ] **Senaryo: Uygunsuz içerik uyarısı** · Verilen adım 1 ilk kullanım · Eğer fotoğraf seçildiyse · O zaman bir kez "Fotoğraftaki kişilerin rızası sende olsun; fotoğraf yalnızca seçtiğin çemberle paylaşılır" bilgi satırı (15. story metni) görünür, engellemez.

### Teknik görevler (Unity/C#)
- [ ] `PhotoPicker` eklentisi: Android `PickVisualMedia` (AndroidX Activity) ve geri uyumlu intent; iOS `PHPicker`; dönüş: geçici dosya yolu; `NativeGallery` paketi değerlendirilir (karar: kendi ince eklenti, izin istememek için).
- [ ] `PhotoDownscaler.Process(path) → byte[] jpeg`: EXIF okuma (küçük `ExifReader`), döndürme, `Texture2D` yeniden örnekleme (bilinear, 2 adımda), `EncodeToJPG(80)`; büyük dokuyu hemen `Destroy`; HEIC için Android'de `ImageDecoder` üzerinden bitmap (eklenti).
- [ ] `CropView`: oran seçimi, `RectTransform` kaydırma, sonuç `GetPixels` kırpma.
- [ ] `UploadService.Upload(path, bytes, contentType, IProgress<float>)`: Firebase Storage `PutBytesAsync`, yeniden deneme, iptal.
- [ ] `SendDraft` modeli: `{circleId, receiverId, photoLocalPath, grid, audioLocalPath, audioSec, note}` yerel JSON; uygulama kapanırsa taslak kalır.
- [ ] Bellek: 2 GB cihazda 12 MP fotoğraf yükleme için `Texture2D` yerine Android tarafında `BitmapFactory.inSampleSize` ile küçültülmüş okuma (eklentide), Unity'ye ≤ 2048 px gelir.
- [ ] Gönderen tarafı tema: aynı `BigCalmTheme` ama "M" kademe (22 px gövde, 64 px düğme) — gönderen 30–50 yaş, yine sade.

### Analitik
- `send_step` {step: "photo"|"difficulty"|"voice"|"note"|"review", action: "enter"|"complete"|"back"} — her adımda
- `photo_selected` {src_w: int, src_h: int, out_kb: int, cropped: bool, ms: int} — küçültme bitince

### Notlar
- EXIF konum temizliği KVKK notu (15): fotoğraf meta verisi sunucuya gitmez.
- Karar: 1024 px üst sınır; tablet oyuncuda 5×5 parça 200 px → 1024/5 ≈ 205 px doku/parça, yeterli.
- Fotoğraf çekme seçeneği ikincil; galeri birincil.

## STORY: Aile Yapbozu 10: Gönderen: zorluk seçimi, sesli mesaj kaydı ve küçük not
labels: tema:gonderen, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **10. story'si** (epic #EPIC). Tema: Gönderen akışı.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Send` sahnesi adım 2–4, `DifficultyAdvisor`, `VoiceRecorder` (yerel eklenti), `NoteInput`

### Kullanıcı hikâyesi
Bir gönderen olarak annemin geçmiş performansına göre önerilen zorluğu tek dokunuşla seçmek, 12–30 saniyelik bir ses kaydetmek ve kısa bir not yazmak isterim, böylece gönderdiğim şey bir fotoğraf değil bir an olsun.

### Kabul kriterleri
- [ ] **Senaryo: Zorluk kartları** · Verilen adım 2 · Eğer çizilirse · O zaman üç kart: Kolay (3×3 · 9 parça), Orta (4×4 · 16), Zor (5×5 · 25); her kartta fotoğrafın o grid'e bölünmüş küçük önizlemesi; biri "Önerilen" rozetli.
- [ ] **Senaryo: Öneri kuralı** · Verilen oyuncunun son 5 tamamlanan yapbozu · Eğer öneri hesaplanırsa · O zaman son 3'ün ortalama süresi mevcut zorlukta < 6 dk ve ipucu ≤ 2 ise bir üst zorluk önerilir, > 15 dk ya da ipucu ≥ 6 ise bir alt, aksi halde aynı; hiç veri yoksa Kolay; kartta "Annen son yapbozu 4 dakikada bitirdi" açıklaması (alıcının o çemberdeki adıyla; süre gönderene görünür, oyuncuya değil). İstatistik alıcı başınadır, çemberden bağımsız (aynı oyuncunun tüm çemberlerdeki tamamlamaları).
- [ ] **Senaryo: Kayıt** · Verilen adım 3 · Eğer "Kaydet" düğmesine basılı tutulursa (ya da dokun-başlat/dokun-bitir, Ayarlar'da seçilebilir) · O zaman mikrofon izni (ilk kez) istenir, kayıt başlar, büyük sayaç 0:00 → 0:30, 12 sn altında bırakılırsa "Biraz daha uzun söyle" uyarısı ama kabul; 30 sn'de otomatik durur.
- [ ] **Senaryo: Dinle ve yeniden kaydet** · Verilen kayıt tamamlandı · Eğer "Dinle" basılırsa · O zaman kendi kaydı çalar; "Yeniden kaydet" eskiyi siler; "Mesajsız gönder" küçük bağlantı (izin verilir, uyarı: "Annen bitince sesini duymayı sever").
- [ ] **Senaryo: Dosya** · Verilen 30 sn kayıt · Eğer kodlanırsa · O zaman AAC-LC, mono, 32 kbps, 22,05 kHz, M4A kapsayıcı, ≤ 500 KB (30 sn ≈ 120 KB); `audioSec` tam saniye.
- [ ] **Senaryo: Not** · Verilen adım 4 · Eğer yazılırsa · O zaman en fazla 80 karakter (sayaç), tek satır klavye, emoji serbest; boş bırakılabilir; öneri çipleri ("Seni özledik", "Bu Ege'nin ilk okul günü", "Hafta sonu geliyoruz").
- [ ] **Senaryo: Gürültü/sessizlik** · Verilen kayıt ortalama seviyesi −45 dBFS altında · Eğer kayıt bitirilirse · O zaman "Ses çok kısık görünüyor, tekrar dener misin?" uyarısı (engellemez).

### Teknik görevler (Unity/C#)
- [ ] `DifficultyAdvisor.Suggest(IReadOnlyList<PuzzleStats>) → Grid`; `PuzzleStats {pieces, durationS, hints, completedAt}` Firestore'dan (`receiverId == target && status == completed`, son 5); Edit Mode testleri (eşikler Remote Config: `advisor_fast_s=360`, `advisor_slow_s=900`).
- [ ] `VoiceRecorder` eklentisi: Android `MediaRecorder` (AAC, `setAudioEncodingBitRate(32000)`, `setAudioSamplingRate(22050)`, MPEG_4), iOS `AVAudioRecorder` (`kAudioFormatMPEG4AAC`); Unity tarafı `IVoiceRecorder {Start(path), Stop() → (sec, bytes), Level}`; Editor'de `Microphone` + WAV yedeği (dev build'de WAV kabul edilir, `audioFormat` alanı).
- [ ] İzin: Android `RECORD_AUDIO` çalışma zamanı isteği yalnızca adım 3'te; red → "Mesajsız gönder" yolu.
- [ ] `RecordView` UI: büyük yuvarlak düğme (120 px), sayaç, dalga (seviyeden), Dinle / Yeniden kaydet / Devam; `VoicePlayer` (7. story) yeniden kullanılır.
- [ ] `NoteInput`: TMP InputField, 80 karakter, çip listesi (TR/EN), `SendDraft.note`.
- [ ] Zorluk kartı önizlemesi: `PuzzleGenerator` ile 128 px küçük doku üzerinde grid çizgisi (maske üretimi gerekmez).
- [ ] `SendDraft` adım durumu kalıcı; geri tuşu bir önceki adıma.

### Analitik
- `difficulty_chosen` {grid: int, suggested: int, followed: bool} — adım 2 tamamlanınca
- `voice_recorded` {sec: int, kb: int, retakes: int, skipped: bool} — adım 3 tamamlanınca
- `note_written` {len: int, chip: bool} — adım 4 tamamlanınca

### Notlar
- Tasarım taslağı: "Aile Yapbozu · Gönder" artboard'u (fotoğraf, zorluk, "annenin son süresine göre öneri", 12 sn sesli mesaj, küçük not, GÖNDER).
- Karar: sesli mesaj isteğe bağlı ama varsayılan akışta; atlamak bir bağlantı, düğme değil.
- Açık soru: iOS'ta M4A kaydı Android `MediaPlayer`'da sorunsuz; tersi de (Android AAC → iOS) test edilecek; iki platform arası aile olağan.

## STORY: Aile Yapbozu 11: Gönderim, durum takibi ve "bitirdi" geri bildirimi
labels: tema:gonderen, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **11. story'si** (epic #EPIC). Tema: Gönderen akışı.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Send` adım 5 (özet + GÖNDER), `SendService`, `SenderHome` sahnesi, Cloud Function `onPuzzleCreated`, `onPuzzleCompleted`

### Kullanıcı hikâyesi
Bir gönderen olarak GÖNDER'e bastıktan sonra yapbozun annemin telefonuna ulaştığını, açıldığını ve bitirildiğini (kaç dakikada, teşekkür etti mi) görmek isterim, böylece aramadan da bağ kurduğumu bileyim.

### Kabul kriterleri
- [ ] **Senaryo: Özet ve gönder** · Verilen adım 5 · Eğer çizilirse · O zaman fotoğraf, zorluk, ses süresi ("18 sn"), not, çember + alıcı ("Aile 👨‍👩‍👧 · Annem"), ücretsiz kota satırı ("Bu hafta 1/2") ve tam genişlik GÖNDER; basılınca fotoğraf + ses sırayla yüklenir (ilerleme), sonra `puzzles/{id}` `status: sent` yazılır; toplam < 8 sn 4G'de.
- [ ] **Senaryo: Atomiklik** · Verilen ses yüklemesi başarısız · Eğer 3 deneme de düşerse · O zaman Firestore belgesi yazılmaz, yüklenen fotoğraf silinir, taslak kalır, "Gönderilemedi, tekrar dene" (bağlantı mesajı 60+ dilinde değil; gönderen normal dil).
- [ ] **Senaryo: Gönderen ana ekranı** · Verilen 4 gönderilmiş yapboz, 2 çember · Eğer SenderHome açılırsa · O zaman üstte çember çipleri ("Tümü" varsayılan + her çember; 3. story) ve "Yeni yapboz gönder" birincil düğme, altında liste: her satır fotoğraf küçük resmi + alıcı + çember etiketi + durum rozeti (Gönderildi / Açıldı / Bitirdi · 14 dk ♥) + tarih; çip seçilince liste o çembere süzülür; Aile Planı kartı (14. story) listeden sonra. Gönderen aynı zamanda bir çemberde oyuncuysa üstte "Sana gelenler (2)" kartı oyuncu gelen kutusunu açar.
- [ ] **Senaryo: Canlı durum** · Verilen SenderHome açık · Eğer oyuncu yapbozu bitirirse · O zaman satır 3 sn içinde "Bitirdi" olur (Firestore listener); bildirim 12. story.
- [ ] **Senaryo: Çember ve alıcı seçimi** · Verilen gönderenin `sender` rolü olan çemberleri · Eğer Send başlarsa · O zaman adım 0a çember kartları (ad + simge; tek çemberde atlanır, SenderHome'da çip seçiliyse o çember önceden seçili), adım 0b o çemberdeki `player` rollü üyelerin büyük avatar kartları (kendisi hariç; tek oyuncuda atlanır). v1'de tek alıcı; "herkese gönder" kapsam dışı.
- [ ] **Senaryo: Silme** · Verilen gönderilmiş yapboz · Eğer gönderen satırı uzun basmadan "…" menüsünden "Sil" seçerse · O zaman onay, sonra Firestore + Storage + oyuncu cihaz önbelleği (bir sonraki senkronda) silinir; oyuncu tarafında aynı menü "Bitirdiklerin" listesinde.
- [ ] **Senaryo: Açıldı durumu** · Verilen oyuncu yapboza ilk kez dokundu · Eğer Puzzle açılırsa · O zaman `status: opened`, `openedAt` yazılır (bir kez).

### Teknik görevler (Unity/C#)
- [ ] `SendService.SendAsync(SendDraft) → Result`: sıra (photo → voice → doc), hata geri alma (`DeleteAsync` yüklenenler), `IProgress`, iptal; puzzle id istemcide üretilir (`Guid`), Storage yolları ona göre.
- [ ] Cloud Function `onPuzzleCreated` (Firestore trigger): alan doğrulama (pieces ∈ {9,16,25}, note ≤ 80, audioSec ≤ 30, gönderen ve alıcı aynı çemberde), `users/{sender}.weekSends` artırma (14. story kota kontrolü burada; kota aşımı → belge `status: rejected` + Storage temizliği), FCM (12).
- [ ] Cloud Function `onPuzzleCompleted` (`status` → completed): gönderene FCM, `users/{sender}.stats` güncelleme (12 ve 16).
- [ ] `SenderHome` UI: liste (`LargeListRow` M kademe), durum rozeti, "…" menü (Sil, Tekrar gönder — aynı fotoğrafla yeni yapboz), Aile Planı kartı yeri.
- [ ] `PuzzleRepository` (istemci): Firestore sorguları `senderId == me` ve `receiverId == me` (çember filtresi istemcide `circleId` ile), listener yönetimi, çevrimdışı önbellek.
- [ ] Silme: istemci `DeletePuzzleAsync` (belge) + Cloud Function `onPuzzleDeleted` Storage dosyalarını siler (istemci Storage silme yetkisi gerekmez).
- [ ] Emulator testleri: doğrulama reddi, kota, silme zinciri.

### Analitik
- `puzzle_sent` {puzzle_id_hash: string, pieces: int, voice: bool, note: bool, upload_ms: int, week_sends: int, plan: string, circle_type: string} — belge yazılınca — Kapı metriği: aktif çember başına haftalık ≥ 2
- `puzzle_send_failed` {stage: "photo"|"voice"|"doc", code: string}
- `puzzle_opened` {age_h: int} — oyuncu ilk açtığında
- `puzzle_deleted` {by: "sender"|"player", status: string}

### Notlar
- "Tekrar gönder" aynı fotoğrafla yeni belge üretir (yeni kota kullanımı); eski yapbozu yeniden açılabilir kılmak yerine basit tutuldu.
- Durum rozetinde süre gönderene gösterilir (14 dk); oyuncu tarafında hiçbir yerde süre yok.

## STORY: Aile Yapbozu 12: Bildirimler (FCM): "yeni yapboz" ve "bitirdi"
labels: tema:backend, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **12. story'si** (epic #EPIC). Tema: Bildirimler.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** Firebase Messaging Unity SDK, `PushService`, Cloud Functions `onPuzzleCreated`/`onPuzzleCompleted` FCM gövdeleri, Android bildirim kanalları, derin bağlantı yönlendirmesi

### Kullanıcı hikâyesi
Bir oyuncu olarak torunum yapboz gönderince telefonumda büyük, net bir bildirim görmek ("Emre yeni yapboz gönderdi"); bir gönderen olarak annem bitirince "Annen yapbozu bitirdi · 14 dk" bildirimi almak isterim, böylece ikimiz de uygulamayı açmayı hatırlayalım.

### Kabul kriterleri
- [ ] **Senaryo: Token kaydı** · Verilen giriş yapılmış kullanıcı · Eğer Bootstrap tamamlanırsa · O zaman FCM token `users/{uid}.fcmTokens[]`'a eklenir (en fazla 5, eskisi düşer); Android 13+ `POST_NOTIFICATIONS` izni oyuncu tarafında ilk yapboz kartı göründükten sonra ("Yeni yapboz gelince haber vereyim mi?" büyük kart), gönderen tarafında ilk gönderimden sonra istenir.
- [ ] **Senaryo: Yeni yapboz** · Verilen `puzzles/{id}` `status: sent` yazıldı · Eğer `onPuzzleCreated` çalışırsa · O zaman alıcının tüm token'larına "Emre yeni yapboz gönderdi" başlığı, "Aile · 9 parça · dokun ve çöz" gövdesi (alıcı tek çemberdeyse çember adı yok), `data: {type: "new_puzzle", puzzleId}`; 10 sn içinde ulaşır (`priority: high`).
- [ ] **Senaryo: Bitirdi** · Verilen `status: completed` · Eğer `onPuzzleCompleted` çalışırsa · O zaman gönderene "Annen yapbozu bitirdi" (`displayName` ile "Ayşe yapbozu bitirdi" EN'de), gövde "14 dakika · 2 ipucu" ve teşekkür geldiyse ikinci bildirim yerine aynı bildirim güncellenir ("♥ Teşekkür etti"); `collapseKey = puzzleId`.
- [ ] **Senaryo: Dokunma** · Verilen bildirim · Eğer dokunulursa · O zaman uygulama ilgili ekrana açılır: oyuncu → Puzzle (doğrudan, ana ekranı atlar), gönderen → SenderHome ilgili satır vurgulu; uygulama kapalıyken soğuk başlangıçta da çalışır.
- [ ] **Senaryo: Uygulama açıkken** · Verilen oyuncu PlayerHome'da · Eğer yeni yapboz gelirse · O zaman sistem bildirimi gösterilmez, kart listesi canlı güncellenir ve TTS "Emre yeni yapboz gönderdi" (Sesli yönlendirme açıksa).
- [ ] **Senaryo: Kapalı** · Verilen Ayarlar'da bildirim kapalı · Eğer olay olursa · O zaman token belgeden silinir, Cloud Function göndermez.
- [ ] **Senaryo: Geçersiz token** · Verilen FCM `UNREGISTERED` yanıtı · Eğer gönderim başarısızsa · O zaman token belgeden temizlenir.

### Teknik görevler (Unity/C#)
- [ ] Firebase Messaging Unity SDK; `PushService`: token alma/yenileme (`TokenReceived`), `users/{uid}.fcmTokens` arrayUnion/arrayRemove, izin akışı (Android 13 `POST_NOTIFICATIONS` için eklenti ya da Unity Mobile Notifications izin API'si).
- [ ] Android bildirim kanalı `puzzles` (önem: yüksek, ses: varsayılan), büyük ikon; `MessageReceived` ön plan işleyicisi (sistem bildirimi bastırma + UI olayı).
- [ ] Derin bağlantı: `data.type` ve `puzzleId` → `SceneFlow.OpenPuzzle(id)`; soğuk başlangıçta `AppState.pendingDeepLink`.
- [ ] Cloud Functions: `sendToUser(uid, payload)` yardımcı, yerelleştirilmiş metin (alıcının `users.locale`), `collapseKey`, hatalı token temizliği; `onPuzzleCompleted` teşekkür güncellemesi için `thanked` alan tetikleyicisi.
- [ ] Yerel bildirim yok (MVP); 21. story gönderen hatırlatmasını ekler.
- [ ] iOS hazırlığı: APNs anahtarı ve `UNUserNotificationCenter` izin akışı (build soft launch sonrası ama kod yolu şimdi).
- [ ] Test: Emulator'de Functions + gerçek cihazda dev projesi; token rotasyonu testi.

### Analitik
- `push_permission` {role: string, granted: bool} — izin yanıtında
- `push_received` {type: string, foreground: bool} · `push_opened` {type: string, cold_start: bool}

### Notlar
- Oyuncu izin kartı metni 60+ dilinde, tek soru, iki büyük düğme (Evet / Şimdi değil); "Şimdi değil" 3 yapboz sonra bir kez daha sorar.
- Bildirim gövdesinde oyuncunun süresi gönderene gider; bu bilgi 15. story aydınlatma metninde açıkça yazılır.

## STORY: Aile Yapbozu 13: Nostalji paketleri ve günün yapbozu
labels: tema:meta, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **13. story'si** (epic #EPIC). Tema: Ücretsiz içerik.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `PackCatalog` (SO + Remote Config), `PacksScreen`, `DailyPuzzle`, Addressables/StreamingAssets içerik, PlayerHome "Hazır yapbozlar" kartı

### Kullanıcı hikâyesi
Bir oyuncu olarak ailemden yapboz gelmediği günlerde de çözecek güzel, büyük fotoğraflar (eski İstanbul, çiçekler, köy hayatı) ve her gün herkese aynı "günün yapbozu" bulmak isterim, böylece uygulama boş kalmasın; davetsiz kuran biri de hemen bir şey çözebilsin.

### Kabul kriterleri
- [ ] **Senaryo: Katalog** · Verilen Hazır yapbozlar ekranı · Eğer açılırsa · O zaman 3 paket kartı (Eski İstanbul, Çiçekler, Köy Hayatı), her pakette 12 fotoğraf; "Çiçekler" ücretsiz ve tamamen açık, diğer ikisinde 3 fotoğraf ücretsiz, kalanı Aile Planı rozeti (oyuncu, sahibi planlı herhangi bir çemberin üyesiyse hepsi açık) (oyuncu tarafında satın alma yok: "Ailenden Aile Planı istenebilir" tek satır, düğme yok).
- [ ] **Senaryo: Paket yapbozu** · Verilen bir paket fotoğrafı · Eğer seçilirse · O zaman zorluk üç büyük kartla sorulur (son seçim hatırlanır), aynı yapboz motoru; tamamlanınca sesli mesaj yerine fotoğraf hakkında bir satır ("Galata Köprüsü, 1960'lar") ve konfeti.
- [ ] **Senaryo: Günün yapbozu** · Verilen tarih · Eğer PlayerHome'da "Bugünün yapbozu" kartı açılırsa · O zaman `daily/{yyyy-MM-dd}` fotoğrafı (Remote Config `daily_puzzle_grid` ile zorluk, varsayılan 4×4) herkese aynı; gün değişince yeni; dünkü yarım kaldıysa tamamlanabilir.
- [ ] **Senaryo: İçerik boyutu** · Verilen 36 fotoğraf · Eğer APK'ya gömülürse · O zaman her biri 1024 px JPEG q75 ≤ 180 KB, toplam ≤ 7 MB; günün yapbozu fotoğrafları Storage'dan (`public/daily/`) indirilir ve 7 gün önbellekte.
- [ ] **Senaryo: Telif** · Verilen her fotoğraf · Eğer `docs/content/credits.md` okunursa · O zaman kaynak (Wikimedia Commons PD / CC0 / Unsplash lisansı), lisans ve atıf satırı listelenir; CC-BY varsa uygulama içi "Fotoğraf kaynakları" ekranında atıf.
- [ ] **Senaryo: Davetsiz oyuncu** · Verilen hiçbir çembere bağlı olmayan oyuncu (3. story) · Eğer PlayerHome açılırsa · O zaman "Bugünün yapbozu" ve "Hazır yapbozlar" kartları görünür; kod kartı "Ailenden ya da arkadaşından kod geldiğinde buraya gir" (küçük, ikincil).

### Teknik görevler (Unity/C#)
- [ ] `PackCatalog` SO: paketler, fotoğraf listeleri (`Sprite` değil `Texture2D` yolu), açık/kilitli kuralı; Remote Config `packs_free_count` (3).
- [ ] İçerik: StreamingAssets `packs/{pack}/{n}.jpg` (Addressables MVP'de gereksiz), `PackLoader` ile yükleme + `PhotoDownscaler` yeniden kullanımı.
- [ ] `DailyPuzzle`: `daily/{date}` Firestore belgesi `{photoPath, caption, grid}`; yoksa yerel havuzdan deterministik seçim (`SHA256("aileyapbozu:" + date)` → paket fotoğrafı); önbellek.
- [ ] `PacksScreen` UI (60+ tema): paket kartları 160 px, fotoğraf ızgarası 2 sütun (tablette 3), kilit rozeti.
- [ ] Tamamlama: `CompletionDirector` `source: pack|daily` dalı (caption kartı, sesli mesaj yok).
- [ ] İçerik hazırlama scripti (`tools/prepare_content.py`): küçültme, EXIF temizleme, credits üretimi.
- [ ] Günlük içerik yükleme: Firebase Console'dan ya da küçük admin scripti (`tools/upload_daily.py`), 30 günlük ileri doldurma.

### Analitik
- `pack_view` {pack: string, unlocked: int}
- `puzzle_completed` `source` parametresi (7. story) bu story'yi ölçer; ek: `pack_locked_tap` {pack: string}

### Notlar
- Karar: oyuncu tarafında hiçbir satın alma düğmesi yok; kilitli içerik yalnızca Aile Planı'na soft işaret. Fluffy Drop'un "işlevsiz coin" hatası tersinden: oyuncu hiçbir ekonomiyle muhatap değil.
- Paket seçimi TR odaklı (Eski İstanbul) ama EN soft launch ülkeleri için "Eski Şehirler" (Londra, Toronto dahil) varyantı: Remote Config `pack_set` ile ülkeye göre.
- "Günün yapbozu" seri motorunun ritüel bileşeni; Büyük Sudoku'da "günün sudokusu" olur.

## STORY: Aile Yapbozu 14: Ücretsiz katman kotası ve yumuşak Premium önerisi
labels: tema:monetizasyon, P1, efor:S
milestone: MVP

**Aile Yapbozu** epic'inin **14. story'si** (epic #EPIC). Tema: Monetizasyon.

**Öncelik:** P1 · **Efor:** S · **Milestone:** MVP · **Dokunduğu yer:** Cloud Function `enforceWeeklyQuota` (`onPuzzleCreated` içinde), `QuotaView`, `PremiumCard` (SenderHome ve Send adım 5), `PlanGate`

### Kullanıcı hikâyesi
Bir gönderen olarak haftada 2 ücretsiz gönderimin nerede bittiğini açıkça görmek ve istersem Aile Planı'na geçmek isterim; bir oyuncu olarak ise hiçbir kotadan, kilitten ya da fiyattan haberdar olmamak isterim.

### Kabul kriterleri
- [ ] **Senaryo: Kota sayacı** · Verilen ücretsiz gönderen, bu hafta tüm çemberlerinde toplam 1 gönderim · Eğer Send adım 5 açılırsa · O zaman "Bu hafta 1/2 ücretsiz gönderim · Pazartesi yenilenir" satırı; GÖNDER aktif.
- [ ] **Senaryo: Kota dolu** · Verilen bu hafta 2 gönderim · Eğer "Yeni yapboz gönder" basılırsa · O zaman akış yine açılır, adım 5'te GÖNDER yerine "Aile Planı ile sınırsız gönder" (birincil) ve "Pazartesi tekrar dene" (ikincil, taslağı saklar); sunucu da `users/{sender}.weekSends >= free_weekly_sends` ise belgeyi `rejected` yapar (istemci atlatamaz). Kota gönderen başına ve tüm çemberler toplamıdır; yeni çember açmak kotayı artırmaz.
- [ ] **Senaryo: Hafta anahtarı** · Verilen gönderenin saat dilimi Europe/Istanbul · Eğer Pazartesi 00:00 geçerse · O zaman `users/{sender}.weekKey` (`yyyy-Www`) değişir ve `weekSends` 0'dan başlar; saat dilimi `users/{sender}.tz`'den (cihaz `TimeZoneInfo`), yoksa UTC.
- [ ] **Senaryo: Yumuşak kart** · Verilen ücretsiz gönderen · Eğer SenderHome açılırsa · O zaman listenin altında Aile Planı kartı (3 fayda: sınırsız gönderim, çember başına 5 üye ve sınırsız çember, tüm paketler; fiyat yerel; seçili çember için "3/5 üye" ilerleme satırı) görünür; kapatılabilir ("Şimdi değil" → 7 gün gizli); her `premium_card_every_n_sends` (5) gönderimde bir kez adım 5'te de görünür; hiçbir yerde tam ekran paywall yok.
- [ ] **Senaryo: Oyuncu tarafı temiz** · Verilen oyuncu, ücretsiz çemberler · Eğer tüm oyuncu ekranları gezilirse · O zaman fiyat, abonelik, kota, reklam kelimesi geçmez; kilitli paket yalnızca rozet (13. story).
- [ ] **Senaryo: Plan aktif** · Verilen gönderen planlı (`users/{sender}.plan == "plus"`, `planUntil > now`) ya da yapbozun çemberinin sahibi planlı · Eğer gönderilirse · O zaman kota kontrolü atlanır, kart gizlenir; sahibi planlı çemberde üye sınırı 5, planlı kullanıcının kurabileceği çember sayısı sınırsız.
- [ ] **Senaryo: Planlı çemberde ücretsiz gönderen** · Verilen Emre'nin planı var, Zeynep "Aile" çemberinde ücretsiz gönderen · Eğer Zeynep "Aile"ye gönderirse · O zaman kota sayılmaz ("Emre'nin planı sayesinde sınırsız"); Zeynep kendi "Arkadaşlar" çemberine gönderirse kendi kotası işler.

### Teknik görevler (Unity/C#)
- [ ] `PlanGate` (istemci): `HasPlan`, `IsCircleCovered(circleId)` (çember sahibinin planı; `circles.ownerPlanUntil` Cloud Function'ın yazdığı tekrar alanından), `FreeSendsLeft`, `WeekResetsAt`, `CanCreateCircle`; Firestore `users/{me}` ve `circles` listener'larından; Remote Config `free_weekly_sends` (2).
- [ ] Cloud Function `enforceWeeklyQuota` (`onPuzzleCreated` içinde transaction): `weekKey` hesaplama (gönderen tz, ISO hafta), kapsam kontrolü (gönderen planı ya da çember sahibinin planı), `users/{sender}.weekSends++` ya da `rejected`; `rejected` belgeleri 24 saat sonra `scheduledCleanup` siler.
- [ ] `QuotaView` (adım 5 satırı), `PremiumCard` prefab (SenderHome + adım 5 varyantı), "Şimdi değil" 7 gün bastırma (PlayerPrefs).
- [ ] Satın alma düğmesi 18. story'ye bağlanır; MVP'de kart "Yakında" değil, 18 olmadan kart gizli (Remote Config `premium_card_enabled=false`, soft launch build'inde true).
- [ ] Fiyat metni `IapService.GetLocalizedPrice("plus_monthly")`; ürün yoksa "3,99 USD/ay" yedek yazılmaz, kart gizlenir.

### Analitik
- `quota_view` {sends_left: int, plan: string} — adım 5'te
- `quota_blocked` {week_sends: int} — kota dolu ekranı
- `premium_card_view` {placement: "home"|"send_review", trigger: "quota"|"cadence"} · `premium_card_dismiss`

### Notlar
- İlke: ödeyen taraf gönderen; oyuncu hiçbir ekonomik sinyal görmez. Bu, mağaza açıklamasındaki "Reklam yok, satın alma baskısı yok" vaadinin teknik karşılığı.
- Karar (2026-10-16): kota haftalık ve gönderen başına, tüm çemberler toplamı. Aile başına kota çember modelinde çember açarak çoğaltılabilirdi. Ücretsiz katmanda kişi başına en fazla 2 kendi kurduğu çember, çember başına 3 üye. Eşikler Remote Config: `free_weekly_sends` (2), `free_owned_circles` (2), `free_circle_members` (3), `plan_circle_members` (5).

## STORY: Aile Yapbozu 15: KVKK/GDPR: aydınlatma, rıza, veri silme ve 90 gün saklama
labels: tema:backend, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **15. story'si** (epic #EPIC). Tema: Gizlilik ve uyum.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `ConsentScreen` (gönderen), `PrivacyScreen` (her iki rol), Cloud Functions `deleteMyData`, `scheduledCleanup`, `docs/privacy.md`, mağaza veri güvenliği formu

### Kullanıcı hikâyesi
Bir gönderen olarak başkalarının (özellikle çocukların) fotoğrafını yüklediğimin farkında olmak ve ne kadar süre saklandığını bilmek; bir oyuncu ya da gönderen olarak tek düğmeyle tüm verimi sildirebilmek isterim, böylece uygulama aileme güvenle önerilebilsin.

### Kabul kriterleri
- [ ] **Senaryo: Aydınlatma** · Verilen gönderen rolü ilk kez seçildi · Eğer ilk çember kurma ya da ilk gönderim öncesi `ConsentScreen` açılırsa · O zaman kısa aydınlatma (5 madde: ne toplanır — fotoğraf, ses, not, ad, cihaz token'ı; nerede — Firebase (Google), bölge `europe-west`; ne kadar — 90 gün; kim görür — yalnızca seçtiğin çemberin üyeleri; haklar — silme) ve "Fotoğraftaki kişilerin rızasını almak senin sorumluluğunda; arkadaş çemberinde aile dışından kişiler de görür" uyarısı; "Okudum, kabul ediyorum" onay kutusu + 18+ beyanı; kabul olmadan ilerlenmez; tam metin bağlantısı.
- [ ] **Senaryo: Oyuncu aydınlatma** · Verilen oyuncu rolü · Eğer karşılama ekranı görünürse · O zaman tek paragraf (24 px) "Emre sana fotoğraf ve ses gönderecek; bunlar yalnızca bu çemberde kalır, 90 gün sonra silinir" ve "Tamam" düğmesi; onay kutusu yok (veri sorumluluğu gönderende, oyuncu veri girmez).
- [ ] **Senaryo: Veri silme** · Verilen Ayarlar > Gizlilik > "Tüm verimi sil" · Eğer onaylanırsa (iki adım, 88 px düğmeler) · O zaman `deleteMyData` çağrılır: kullanıcının gönderdiği ve aldığı tüm yapbozlar (Firestore + Storage), `users/{uid}` ve tüm çember üyelikleri silinir; kullanıcının sahibi olduğu çemberlerde sahiplik en eski `sender` üyeye devredilir, `sender` kalmadıysa çember (ve içindeki yapbozlar) silinir; işlem < 60 sn, bitince uygulama RoleGate'e döner; Analytics `user_id` için UGS silme isteği `docs/privacy.md`'deki prosedürle.
- [ ] **Senaryo: Otomatik silme** · Verilen `createdAt` 90 günden eski yapboz · Eğer `scheduledCleanup` günlük çalışırsa · O zaman belge ve Storage dosyaları silinir; 83. günde oyuncu tarafında "Bu yapboz 7 gün sonra silinecek" rozeti (tamamlanmış yapbozlar "Bitirdiklerin"de de silinir; albüm SONRA 26).
- [ ] **Senaryo: Gizlilik ekranı** · Verilen her iki rol · Eğer Ayarlar > Gizlilik açılırsa · O zaman tam aydınlatma metni (TR/EN), "Tüm verimi sil", "Verilerimi indir" (SONRA; v1'de e-posta adresi: `docs/privacy.md`'deki iletişim), üçüncü taraf listesi (Firebase, UGS Analytics, Crashlytics).
- [ ] **Senaryo: Analitik anonimliği** · Verilen tüm olaylar · Eğer incelenirse · O zaman ad, fotoğraf, ses, not, e-posta, kesin konum yoktur; `puzzle_id_hash` ve `circle_id_hash` SHA256 ilk 8 karakter; çember adı analitiğe gitmez (yalnızca `circle_type`).
- [ ] **Senaryo: Mağaza beyanı** · Verilen Play veri güvenliği formu · Eğer doldurulursa · O zaman "Fotoğraflar, ses, cihaz kimliği, uygulama etkileşimi; şifreli aktarım; silme talebi mümkün; çocuklara yönelik değil" beyanı ve `docs/privacy.md` ile tutarlı; kategori Puzzle, "Kids" değil, hedef yaş 18+.

### Teknik görevler (Unity/C#)
- [ ] `ConsentScreen` (gönderen) ve oyuncu karşılama paragrafı; `users/{uid}.consent {version, acceptedAt}`; sürüm değişince yeniden gösterim.
- [ ] Cloud Function `deleteMyData` (callable, auth zorunlu): batch silme (Firestore 500'lük partiler), Storage prefix silme, çember üyelikleri ve sahiplik devri, FCM token; idempotent; tamamlanınca `users/{uid}` silinir ve Anonymous Auth kullanıcısı `admin.auth().deleteUser`.
- [ ] Cloud Function `scheduledCleanup` (Pub/Sub, günlük 03:00 UTC): 90 gün kuralı, `rejected` 24 saat, yetim Storage dosyaları; `retention_days` Functions config.
- [ ] Firebase bölge: Firestore ve Storage `europe-west1/eur3` (GDPR; TR için KVKK yurt dışı aktarım — açık rıza metinde).
- [ ] `PrivacyScreen` UI (60+ tema, kaydırılabilir metin, 24 px), silme onay akışı.
- [ ] `docs/privacy.md`: aydınlatma metni TR/EN, veri envanteri tablosu, saklama süreleri, üçüncü taraflar, silme prosedürü (UGS Analytics için manuel), iletişim adresi; gizlilik politikası sayfası GitHub Pages'te (`aileyapbozu.app/privacy`).
- [ ] Crashlytics'te özel anahtar olarak kişisel veri gönderilmediğinin denetimi.

### Analitik
- `consent_accepted` {version: int, role: string} — kabul anında
- `data_delete_requested` {role: string} · `data_delete_done` {ms: int}

### Notlar
- Hukuki metin şablonu dışarıdan (avukat/şablon hizmeti) alınır; story bunu bloklamaz, placeholder metinle çıkar, soft launch öncesi gerçek metin.
- Açık soru: VERBİS kaydı (TR) — bireysel geliştirici muafiyet eşiği doğrulanacak.
- Çocuk fotoğrafı: uygulama çocuk verisi "toplamaz", gönderen kapalı çember içinde paylaşır; yine de ConsentScreen'de açık uyarı ve "Kids" dışı kategori.
- 90 gün kararı: depolama maliyeti + "uygulama albüm değil" konumlandırması; albüm 26 (SONRA, Aile Planı özelliği olarak düşünülür).

## STORY: Aile Yapbozu 16: Analitik olay şeması ve kapı raporu
labels: tema:analitik, P1, efor:S
milestone: MVP

**Aile Yapbozu** epic'inin **16. story'si** (epic #EPIC). Tema: Analitik.

**Öncelik:** P1 · **Efor:** S · **Milestone:** MVP · **Dokunduğu yer:** UGS Analytics olay tanımları, `docs/analytics.md`, SQL raporları, Firestore toplulaştırma (Cloud Function `nightlyStats`)

### Kullanıcı hikâyesi
Bir geliştirici olarak soft launch kapısını (davet kabulü, çember başına haftalık gönderim, Aile Planı dönüşümü, 72 saat tamamlama, çökme) tek sorguyla görmek isterim, böylece Büyük Sudoku'ya geçme kararı veriyle verilsin.

### Kabul kriterleri
- [ ] **Senaryo: Şema** · Verilen `docs/analytics.md` · Eğer okunursa · O zaman tüm olaylar (app_open, session_start, role_selected, circle_created, invite_created, invite_accepted, backend_error, puzzle_generated, piece_snapped, puzzle_resumed, hint_used, tts_availability, puzzle_completed, voice_played, thanks_sent, home_view, ui_text_scale, send_step, photo_selected, difficulty_chosen, voice_recorded, note_written, puzzle_sent, puzzle_send_failed, puzzle_opened, puzzle_deleted, push_permission, push_received, push_opened, pack_view, pack_locked_tap, quota_view, quota_blocked, premium_card_view, premium_card_dismiss, consent_accepted, data_delete_requested, data_delete_done, settings_change, plan_view, plan_purchase, plan_restore, circle_view, member_removed, account_linked) parametre tipleriyle listelidir ve UGS'de tanımlıdır.
- [ ] **Senaryo: Rol ayrımı** · Verilen her olay · Eğer gönderilirse · O zaman ortak parametreler `role: "player"|"sender"|null` (olayın çemberindeki rol), `circle_id_hash`, `circle_type` (olay bir çembere bağlıysa) ve `circles_count` (kullanıcının üye olduğu çember sayısı) otomatik eklenir (Analytics sarmalayıcısı).
- [ ] **Senaryo: Doğrulama** · Verilen dev build, iki cihaz (gönderen + oyuncu) · Eğer bir yapboz gönderilip çözülürse · O zaman UGS Event Browser'da olaylar iki rolde sırayla ve doğru parametrelerle görünür.
- [ ] **Senaryo: Kapı sorgusu** · Verilen UGS SQL ve Firestore gece istatistikleri · Eğer `gate_report.sql` çalıştırılırsa · O zaman davet kabulü, aktif çember başına haftalık gönderim, Aile Planı dönüşümü (14 gün), 72 saat tamamlama oranı, oyuncu tarafı çökme oranı tek tabloda ve `circle_type` kırılımıyla gelir.
- [ ] **Senaryo: Sunucu tarafı sayım** · Verilen `circles` ve `puzzles` · Eğer `nightlyStats` çalışırsa · O zaman `stats/{yyyy-MM-dd}` belgesinde türe göre çember sayısı, aktif çember (7 gün), kullanıcı başına çember dağılımı, gönderim, tamamlama, plan sayısı; UGS verisiyle çapraz kontrol (istemci olay kaybına karşı).
- [ ] **Senaryo: Gizlilik** · Verilen olaylar · Eğer incelenirse · O zaman 15. story anonimlik kuralı sağlanır.

### Teknik görevler (Unity/C#)
- [ ] `Analytics` (Core) sarmalayıcısı: `Track(string, Dictionary<string, object>)`, ortak parametre enjeksiyonu (`role`, `circle_id_hash`, `circle_type`, `circles_count`, `device_class`), kuyruk + çevrimdışı tampon.
- [ ] UGS Analytics custom event tanımları (dashboard), parametre tipleri.
- [ ] `docs/analytics.md`, `docs/reports/gate_report.sql`, `funnel_sender.sql` (role_selected → circle_created → invite_created → invite_accepted → puzzle_sent), `funnel_player.sql` (invite_accepted → puzzle_opened → puzzle_completed → thanks_sent), `completion_72h.sql`.
- [ ] Cloud Function `nightlyStats` (Pub/Sub günlük) + `tools/export_stats.py` (Firestore → CSV).
- [ ] Crashlytics (Firebase) çökme oranı; eski cihaz segmenti (`ram_mb < 3000`) ayrı satır.

### Analitik
- Bu story şemanın kendisidir.

### Notlar
- MustyBlockBlast ve Daily Drop'taki `docs/UGS SQL` yaklaşımı; farkı iki rollü huni ve Firestore çapraz sayımı.
- Kapı tanımları epic'teki "Başarı ölçütleri" ile birebir; rapor şablonu `docs/reports/gate_report_template.md`.

## STORY: Aile Yapbozu 17: Mağaza sayfası, ASO ve soft launch yapılandırması
labels: tema:magaza, P1, efor:M
milestone: MVP

**Aile Yapbozu** epic'inin **17. story'si** (epic #EPIC). Tema: Mağaza ve çıkış.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** Play Console, mağaza metinleri, telefon + tablet ekran görüntüleri, kapalı test, ülke/fiyat yapılandırması

### Kullanıcı hikâyesi
Bir geliştirici olarak uygulamayı "jigsaw for seniors" arayan çocuk/torunların bulacağı bir mağaza sayfasıyla üç ülkede kapalı test ve soft launch'a çıkarmak isterim, böylece davet ve gönderim verisi gerçek ailelerden gelsin.

### Kabul kriterleri
- [ ] **Senaryo: Başlık ve metin** · Verilen Play listesi · Eğer yayımlanırsa · O zaman EN başlık 30 karakter sınırında "Family Jigsaw: For Seniors" (26; uzun ad "Family Jigsaw: Puzzles for Seniors" kısa açıklamanın başında), TR "Aile Yapbozu: Büyükler İçin" (27); kısa açıklama ≤ 80 karakter ("Send Mom a photo puzzle. No timer, no ads, big pieces, your voice at the end."); tam açıklamada "jigsaw for seniors", "puzzles for seniors", "family puzzle", "no timer", "large pieces", "voice message" geçer ve bir satır arkadaş çemberlerini anar ("Make circles for family and friends"); başlık ve kısa açıklama aile odaklı kalır; TR'de "büyükanne yapboz", "yaşlılar için yapboz", "süre yok".
- [ ] **Senaryo: Görseller** · Verilen ekran görüntüleri · Eğer yüklenirse · O zaman telefon 8 (ilk üçü: oyuncu 3×3 ekranı "Süre yok · Reklam yok" ile, gönder akışı, bitince sesli mesaj), 7" ve 10" tablet 4'er (tablet listesi Play'de ayrı sekme), 1 özellik grafiği (iki telefon: anne ve çocuk), 20 sn video isteğe bağlı; her görselde ≥ 24 px başlık metni, kontrast ≥ 4,5:1.
- [ ] **Senaryo: Kapalı test** · Verilen 12+ test kullanıcısı (6 aile: her biri 1 gönderen + 1 oyuncu, en az 2 oyuncu 65+) · Eğer 14 gün sürerse · O zaman Play üretim erişimi formu doldurulur; test ailelerinden 60+ gözlem notları `docs/usability.md`.
- [ ] **Senaryo: Ülkeler** · Verilen soft launch · Eğer yayımlanırsa · O zaman yalnızca TR, UK, CA; Aile Planı fiyatı yerel (TR kademesi düşük, UK 3,49 GBP, CA 5,49 CAD başlangıç); yaş derecelendirmesi IARC "Herkes", kategori Puzzle, "Kids" programı dışı, hedef kitle 18+ beyanı.
- [ ] **Senaryo: Sürüm bilgisi** · Verilen build · Eğer yayımlanırsa · O zaman versionCode ve versionName CHANGELOG ile eşleşir; gizlilik politikası URL'si, veri güvenliği formu (15. story) ve hesap silme URL'si (Play zorunluluğu; uygulama içi silme + web formu) dolu.
- [ ] **Senaryo: Davet web sayfası** · Verilen `aileyapbozu.app/j/{kod}` · Eğer ziyaret edilirse · O zaman kod büyük (çember adı gösterilmez), mağaza düğmeleri (Play; iOS "yakında") ve 3 adımlı "Telefona nasıl kurulur" görseli; `assetlinks.json` doğrulanır.

### Teknik görevler (Unity/C#)
- [ ] Mağaza metinleri `docs/store/{en,tr}.md`; ASO anahtar kelime listesi ve rakip başlıkları (jigsaw/senior arama sonuçları ilk 20).
- [ ] Ekran görüntüsü şablonu (Design kanvası): telefon 1080×1920, tablet 1600×2560 ve 2560×1600; 60+ tema renkleri.
- [ ] Veri güvenliği formu ve hesap silme sayfası (`aileyapbozu.app/delete`): GitHub Pages statik.
- [ ] Play Console kurulumu: iç test → kapalı test (6 aile) → üretim (3 ülke); Aile Planı abonelik ürünleri Play'de tanımlı (18. story kodu soft launch build'ine girer).
- [ ] Sürümleme: `CHANGELOG.md`, `versionCode` CI'da otomatik; `prod` Firebase projesi build'i.
- [ ] Kullanılabilirlik gözlemi protokolü: 3 görev (kod gir, yapboz çöz, mesajı dinle), süre ve hata sayısı; `docs/usability.md`.

### Analitik
- Mağaza: Play Console dönüşüm oranı (listing → install), arama terimi raporu; davet sayfası ziyaret sayısı (GitHub Pages için basit sayaç yok; UTM ile `invite_accepted.via = link`).

### Notlar
- Başlık testi: soft launch sırasında "Family Jigsaw: For Seniors" ile "Photo Jigsaw for Grandma" A/B (Play listing experiments); TR'de "Aile Yapbozu" sabit.
- Tablet ekran görüntüleri ihmal edilmez: 60+ oyuncuların önemli kısmı hediye tablet kullanır ve Play tablet listesinde görünürlük ayrı.
- iOS: soft launch kapısı geçilince; davet sayfasındaki "yakında" o güne kadar kalır.

## STORY: Aile Yapbozu 18: Aile Planı aboneliği ve geri yükleme
labels: tema:monetizasyon, P2, efor:L
milestone: Soft launch

**Aile Yapbozu** epic'inin **18. story'si** (epic #EPIC). Tema: Monetizasyon.

**Öncelik:** P2 · **Efor:** L · **Milestone:** Soft launch · **Dokunduğu yer:** `IapService` (Core), `PlanScreen`, Cloud Function `verifyPurchase`, `users.plan` + `circles.ownerPlanUntil`, Play Developer API

### Kullanıcı hikâyesi
Bir gönderen olarak aylık küçük bir ücretle sınırsız yapboz göndermek, istediğim kadar çember kurup her birine 5 kişiye kadar üye eklemek ve tüm paketleri açmak isterim; abonelik kurduğum tüm çemberlere uygulanmalı ve telefon değiştirsem de geri gelmeli.

### Kabul kriterleri
- [ ] **Senaryo: Plan ekranı** · Verilen Aile Planı kartı (14. story) · Eğer dokunulursa · O zaman `PlanScreen`: 3 fayda (sınırsız gönderim, sınırsız çember ve çember başına 5 üye, kurduğun çemberlerdeki herkese tüm paketler), "Annen hiçbir zaman reklam ya da satın alma görmez" satırı, aylık (3,99 USD) ve yıllık (29,99 USD, "%37 tasarruf") yerel fiyatlar, 7 gün ücretsiz deneme (yıllıkta), "Satın alımları geri yükle", abonelik koşulları bağlantısı.
- [ ] **Senaryo: Satın alma** · Verilen Google Play Billing · Eğer aylık plan alınırsa · O zaman makbuz `verifyPurchase` ile sunucuda doğrulanır, `users/{uid}.plan = "plus"`, `planUntil` yazılır ve kullanıcının sahibi olduğu tüm `circles/{id}.ownerPlanUntil` güncellenir; bu çemberlerin tüm üyelerinde 5 sn içinde kota kalkar ve paketler açılır (listener). Sonradan kurulan çember de `createCircle`'da kapsama alınır.
- [ ] **Senaryo: Yenileme ve iptal** · Verilen abonelik yenilendi ya da iptal edildi · Eğer Play Real-time Developer Notifications gelirse · O zaman `planUntil` ve sahip olunan çemberlerin `ownerPlanUntil` alanları güncellenir; süre dolunca `plan = "free"`, veri silinmez, 3'ten fazla üyeli ya da 2'den fazla kendi çemberi olan kullanıcıda hiçbir şey kapanmaz (yalnızca yeni üye/çember eklenemez, uyarı).
- [ ] **Senaryo: Geri yükleme** · Verilen yeni cihaz, aynı Google hesabı, yeni anonim uid · Eğer "Geri yükle" basılırsa · O zaman Play'den abonelik bulunur, `verifyPurchase` `purchaseToken`'ı önceki uid ile eşler; önceki uid'nin sahibi olduğu çemberler yeni uid'ye devredilir ve kullanıcı bu çemberlere `owner` + `sender` olarak bağlanır (davet kodu gerekmez; katıldığı başka çemberler için kod gerekir).
- [ ] **Senaryo: Plan kapsamı** · Verilen planlı Emre, Fatma'nın "Komşular" çemberine üye · Eğer Emre "Komşular"a gönderirse · O zaman Emre'nin gönderimi kotasızdır ama "Komşular"ın üye sınırı ve diğer üyelerin kotası değişmez (plan sahibinin kurduğu çemberleri kapsar, katıldıklarını değil); `PlanScreen`'de bu açıkça yazar.
- [ ] **Senaryo: Hediye** · Verilen gönderen A planı aldı · Eğer A'nın kurduğu çemberde gönderen B varsa · O zaman B o çembere kotasız gönderir ve B'nin ekranında o çember için kart "Aile Planı aktif · Emre'nin hediyesi" olur; B'nin kendi çemberleri için satın alma düğmesi görünür kalır.
- [ ] **Senaryo: Oyuncu tarafı** · Verilen plan aktif ya da değil · Eğer oyuncu ekranları gezilirse · O zaman hiçbir değişiklik dışında kilitli paketlerin açılması görülmez (fiyat/abonelik metni yok).

### Teknik görevler (Unity/C#)
- [ ] Unity IAP abonelik ürünleri `plus_monthly`, `plus_yearly` (deneme teklifi Play'de base plan + offer); App Store Connect tanımları iOS build'i için hazır.
- [ ] Cloud Function `verifyPurchase(purchaseToken, sku)` (callable): Google Play Developer API `purchases.subscriptionsv2.get`, `users` + sahip olunan `circles.ownerPlanUntil` güncelleme (batch), `purchases/{token}` kaydı (uid eşlemesi ve geri yükleme için); RTDN Pub/Sub aboneliği `onSubscriptionNotification`.
- [ ] `PlanGate` (14. story) `plan` ve `planUntil` okur; `PlanScreen` UI (gönderen teması, M kademe), fiyat `metadata.localizedPrice`.
- [ ] Geri yükleme akışı: `IapService.Restore()` → token → `verifyPurchase` → sahiplik devri (`transferOwnedCircles(oldUid, newUid)`).
- [ ] Fiyat testi: Remote Config `plan_price_variant` ile iki Play ürünü (A: 3,99 / B: 4,99 aylık) — soft launch'ta yalnızca UK ve CA'da.
- [ ] Hata yolları: doğrulama başarısız (makbuz geçersiz) → kullanıcıya "Doğrulanamadı, tekrar dene", plan yazılmaz; çevrimdışı satın alma tamamlanınca kuyruğa.

### Analitik
- `plan_view` {source: "card_home"|"card_send"|"quota"|"settings"} · `plan_purchase` {sku: string, price_local: string, trial: bool, owned_circles: int, max_circle_members: int} · `plan_restore` {success: bool} · `plan_expired` — Kapı metriği: plan alan kullanıcı / en az bir kabul edilmiş daveti olan çember kurucusu ≥ %8

### Notlar
- Karar: başlangıç fiyatı 3,99 USD/ay, 29,99 USD/yıl; TR'de yerel kademe (Play'in TR fiyat şablonu) ve "yıllık" vurgusu (TR'de aylık abonelik iptali yaygın).
- Hediye akışı bu story'de "plan sahibinin kurduğu çemberdeki gönderenler" ile sınırlı; dış hediye (hediye kodu) SONRA değerlendirilir.
- Karar (2026-10-16): plan kişiye yazılır, sahibi olduğu çemberleri kapsar; adı ("Aile Planı" / "Çember Planı" / "Plus") bu story'nin refine'ında seçilir; Play ürün kimlikleri `plus_monthly` / `plus_yearly` (ad değişse de kimlik sabit). Mağaza metninde "Family" ASO için korunur.
- Bu story kapı sonrası değil, soft launch build'inde olmalı: dönüşüm kapısı bu olmadan ölçülemez. P2 ama soft launch'ın ilk günü canlı.

## STORY: Aile Yapbozu 19: Çember yönetimi: üyeler, birden fazla oyuncu, hesap bağlama
labels: tema:aile, P2, efor:M
milestone: Soft launch

**Aile Yapbozu** epic'inin **19. story'si** (epic #EPIC). Tema: Çember bağlantısı.

**Öncelik:** P2 · **Efor:** M · **Milestone:** Soft launch · **Dokunduğu yer:** `CirclesScreen` + `CircleDetailScreen` (gönderen), `CircleService` genişletme, Firebase Auth hesap bağlama (Google / Apple), Cloud Functions `removeMember`, `leaveCircle`, `updateCircle`

### Kullanıcı hikâyesi
Bir gönderen olarak çemberlerimi (Aile, Arkadaşlar) ve her birindeki üyeleri görmek, çemberi yeniden adlandırmak, yeni oyuncu eklemek (babam da çözsün), bir üyeyi çıkarmak, bir çemberden ayrılmak ve hesabımı Google ile bağlayıp telefon değiştirsem de kaybetmemek isterim.

### Kabul kriterleri
- [ ] **Senaryo: Çember listesi** · Verilen gönderen Ayarlar > Çemberler · Eğer açılırsa · O zaman her çember bir satır (simge + ad + üye sayısı + "sahibi sensin" / "Fatma'nın çemberi"), "+ Yeni çember" (3. story) ve "Kod gir".
- [ ] **Senaryo: Üye listesi** · Verilen `CircleDetailScreen` · Eğer açılırsa · O zaman üyeler avatar + ad + roller + "son aktif: 2 gün önce" (oyuncu için yalnızca gönderenlere görünür), "Üye ekle" (kod üretir, 3. story), plan satırı ("3/5 üye · Ücretsiz" ya da "Aile Planı · 5 üyeye kadar"); sahip için "Adı ve simgeyi değiştir".
- [ ] **Senaryo: İkinci oyuncu** · Verilen çemberde 1 oyuncu · Eğer "Üye ekle" → "Yapboz çözecek biri" seçilip kod başka cihazda `player` rolüyle girilirse · O zaman iki oyuncu olur; Send akışında adım 0b (alıcı seçimi, 11. story) açılır; her oyuncunun kendi gelen kutusu.
- [ ] **Senaryo: Çıkarma** · Verilen bir üye · Eğer çember sahibi "Çıkar" derse (onay) · O zaman `removeMember` üyeliği siler, o üyenin bu çemberde gönderdiği/aldığı yapbozlar kalır (silme 15. story'ye göre); çıkarılan kişinin başka çemberi varsa yalnızca bu çemberin yapbozları gelen kutusundan kalkar, hiç çemberi kalmadıysa RoleGate'e düşer; mesaj "Komşular çemberinden çıkarıldın".
- [ ] **Senaryo: Ayrılma** · Verilen gönderen bir çemberde · Eğer "Çemberden ayrıl" derse (onay) · O zaman `leaveCircle` üyeliği siler; sahip ayrılırsa sahiplik en eski `sender` üyeye geçer (yoksa çember silinir, onay metni bunu söyler). Oyuncu kendi kendini çıkaramaz (yanlışlıkla), gönderen çıkabilir.
- [ ] **Senaryo: Hesap bağlama** · Verilen anonim gönderen · Eğer Ayarlar > "Hesabımı koru" → Google ile giriş · O zaman Firebase `linkWithCredential` ile aynı uid korunur; yeni cihazda Google ile giriş aynı uid'yi ve tüm çemberleri getirir; bağlı hesap zaten başka uid'deyse "Bu Google hesabı başka bir cihazdaki çemberlere bağlı" ve seçim (o hesaba geç / vazgeç).
- [ ] **Senaryo: Oyuncu tarafı** · Verilen oyuncu · Eğer Ayarlar açılırsa · O zaman hesap bağlama önerilmez (karmaşıklık); kaybolursa gönderen yeni kodla yeniden bağlar (3. story birleştirme).
- [ ] **Senaryo: Ad düzenleme** · Verilen gönderen · Eğer oyuncunun bu çemberdeki adını düzenlerse ("Ayşe" → "Anneannem") · O zaman o çemberden gelen karşılama ve bildirim metinleri güncellenir; diğer çemberlerdeki adı değişmez. Karşılama ("Günaydın Ayşe Hanım") oyuncunun ilk katıldığı çemberdeki adı kullanır.

### Teknik görevler (Unity/C#)
- [ ] `CirclesScreen` + `CircleDetailScreen` UI (gönderen teması), çember satırları, üye satırları, çıkarma/ayrılma onayı, ad/simge düzenleme, plan satırı.
- [ ] `CircleService`: `Members(circleId)` listener, `RenameMember`, `UpdateCircle(name, emoji)`, `Leave`; Cloud Functions `removeMember` (yalnızca `ownerId`; oyuncunun kendini çıkarması engelli), `leaveCircle` (sahiplik devri), `updateCircle`; `users/{uid}.circleIds` arrayRemove, FCM "çemberden çıkarıldın" (data-only, istemci gelen kutusunu tazeler ya da RoleGate).
- [ ] Firebase Auth: Google Sign-In (Android) ve Sign in with Apple (iOS build'i için) `linkWithCredentialAsync`; çakışma (`credential-already-in-use`) akışı.
- [ ] Çok oyunculu çember: `PuzzleRepository` sorguları `receiverId` bazlı (zaten), Send adım 0b alıcı kartları.
- [ ] Emulator testi: çıkarma yetkisi, sahiplik devri, son gönderen ayrılınca çember silme, çakışan hesap bağlama.

### Analitik
- `circle_view` {circle_type: string, members: int, players: int, plan: string} · `member_removed` {role: string, by: "owner"|"self"} · `account_linked` {provider: string, conflict: bool}

### Notlar
- Karar: oyuncuya hesap bağlama yok; oyuncunun kimliği çember üyeliğinde gönderenin verdiği ad; kayıp durumunda gönderen yeniden bağlar.
- Açık soru kapandı (2026-10-16): bir kullanıcı birden fazla çemberde, her birinde farklı rollerle olabilir (ör. 50 yaşında biri annesine gönderir, çocuğundan alır). Rol üyeliğe yazılır (2. story).

## STORY: Aile Yapbozu 20: Ayarlar, yazı boyutu kademeleri ve değerlendirme isteği
labels: tema:erisilebilirlik, P2, efor:S
milestone: Soft launch

**Aile Yapbozu** epic'inin **20. story'si** (epic #EPIC). Tema: 60+ arayüz ve erişilebilirlik.

**Öncelik:** P2 · **Efor:** S · **Milestone:** Soft launch · **Dokunduğu yer:** `SettingsScreen` (Core) 60+ varyantı, `BigCalmTheme` kademe seçimi, `SpeakOnFocus`, in-app review tetiği

### Kullanıcı hikâyesi
Bir oyuncu olarak yazıyı daha da büyütmek, sesli yönlendirmeyi açıp kapatmak, dili ve bildirimi tek basit listeden ayarlamak isterim; bir gönderen olarak ise annem üçüncü yapbozu bitirince uygulamayı değerlendirme fırsatı bulmak isterim (oyuncuya hiç sorulmasın).

### Kabul kriterleri
- [ ] **Senaryo: Oyuncu ayar listesi** · Verilen oyuncu Ayarlar · Eğer açılırsa · O zaman yalnızca 7 satır (her biri 96 px): Yazı boyutu (Normal / Büyük / Çok büyük), Sesli yönlendirme (açık/kapalı), Düğmeye dokununca söyle (açık/kapalı), Ses efektleri, Hareketi azalt, Dil, Bildirimler; en altta Gizlilik ve Sürüm; reklam/abonelik/değerlendirme satırı yok.
- [ ] **Senaryo: Yazı boyutu** · Verilen "Çok büyük" seçildi · Eğer ekranlar çizilirse · O zaman gövde 28 px, düğme etiketi 30 px, düğme yüksekliği 96 px; Puzzle'da tepsi 120 px'e çıkar; hiçbir ekran kırpılmaz (ThemeLint üç kademede koşar).
- [ ] **Senaryo: Düğmeye dokununca söyle** · Verilen açık · Eğer bir düğmeye ilk dokunuşta · O zaman TTS etiketi söyler ve eylem ikinci dokunuşta (600 ms içinde) çalışır; kapalıyken tek dokunuş.
- [ ] **Senaryo: Gönderen ayarları** · Verilen gönderen Ayarlar · Eğer açılırsa · O zaman Çemberler, Aile Planı, Hesabımı koru, Bildirimler, Kayıt tarzı (basılı tut / dokun), Dil, Uygulamayı puanla, Gizlilik, Sürüm.
- [ ] **Senaryo: Değerlendirme isteği** · Verilen gönderen, gönderdiği yapbozlardan 3.'sünün tamamlandı bildirimi · Eğer SenderHome açılırsa · O zaman Google Play In-App Review bir kez tetiklenir; oyuncu tarafında asla.
- [ ] **Senaryo: Android geri** · Verilen Ayarlar açık · Eğer geri basılırsa · O zaman ana ekrana dönülür.

### Teknik görevler (Unity/C#)
- [ ] Core `SettingsScreen` iki yapılandırma (`PlayerSettingsConfig`, `SenderSettingsConfig`); oyuna özgü anahtarlar `text_scale`, `tts_enabled`, `speak_on_focus`, `record_style`.
- [ ] `BigCalmTheme` kademe değişimi çalışma zamanında (`ThemeApplier.Reapply`), Puzzle tepsi yüksekliği kademeye bağlı.
- [ ] `SpeakOnFocus`: `LargeButton` içinde ilk dokunuş TTS + 600 ms pencere.
- [ ] In-app review tetiği (Google Play In-App Review; iOS `SKStoreReviewController`) gönderen tarafında `completed_count == 3`.
- [ ] ThemeLint CI'da 3 kademe × 2 cihaz sınıfı.

### Analitik
- `settings_change` {key: string, value: string, role: string} · `review_prompt` {shown: bool}

### Notlar
- Oyuncu ayar listesi bilerek kısa; her ek satır bir destek telefonu demek.
- Büyük yazı modu Core'dan gelir; bu story oyuna özgü üçüncü kademeyi ekler.

## STORY: Aile Yapbozu 21: Gönderen hatırlatması ve haftalık aile özeti
labels: tema:ritual, P3, efor:S
milestone: Canlı

**Aile Yapbozu** epic'inin **21. story'si** (epic #EPIC). Tema: Bildirimler ve ritüel.

**Öncelik:** P3 · **Efor:** S · **Milestone:** Canlı · **Dokunduğu yer:** Cloud Function `weeklyDigest`, `senderReminder`, Ayarlar hatırlatma günü, SenderHome özet kartı

### Kullanıcı hikâyesi
Bir gönderen olarak bir haftadır yapboz göndermediysem nazik bir hatırlatma ("Annen son yapbozu Salı bitirdi, yeni bir tane gönderir misin?") ve Pazar akşamı küçük bir aile özeti almak isterim, böylece ritüel sürsün.

### Kabul kriterleri
- [ ] **Senaryo: Hatırlatma** · Verilen gönderen bir çembere 7 gündür gönderim yapmadı ve o çemberin oyuncusu son 14 günde aktif · Eğer `senderReminder` günlük çalışırsa · O zaman gönderene yerel saat 19:00'da (tz `users.tz`) bir bildirim; 14 gün içinde en fazla 1; Ayarlar'dan kapatılabilir; oyuncuya hiçbir hatırlatma yok.
- [ ] **Senaryo: Haftalık özet** · Verilen Pazar 18:00 yerel · Eğer `weeklyDigest` çalışırsa · O zaman "Bu hafta: 2 yapboz, 31 dk, 1 teşekkür" bildirimi ve SenderHome'da çember başına özet kartı (fotoğraf küçük resimleri); hiç etkinlik yoksa gönderilmez; gönderenin birden fazla çemberi varsa tek bildirimde toplanır.
- [ ] **Senaryo: Kota farkındalığı** · Verilen ücretsiz gönderen ve Pazartesi · Eğer hafta yenilenirse · O zaman özet kartında "2 ücretsiz gönderim yenilendi" satırı (bildirim değil, kart).
- [ ] **Senaryo: Sessiz aile** · Verilen oyuncu 30 gündür aktif değil · Eğer hatırlatma koşulu oluşursa · O zaman farklı metin: "Annenin telefonunda uygulama açılmamış görünüyor; bir bakar mısın?" (bir kez).

### Teknik görevler (Unity/C#)
- [ ] Cloud Functions `senderReminder`, `weeklyDigest` (Pub/Sub, saatlik tarama, tz'ye göre gönderim), `users/{uid}.reminders {enabled, lastSentAt}`.
- [ ] SenderHome `WeeklyDigestCard` (Firestore `circles/{id}/digests/{week}`).
- [ ] Ayarlar: hatırlatma açık/kapalı (gönderen).

### Analitik
- `reminder_sent` {type: "idle"|"silent_player"|"digest"} (sunucu → UGS değil; Firestore `stats`) · `reminder_opened` {type: string} (istemci `push_opened` ile)

### Notlar
- Hatırlatma frekansı kasıtlı düşük; "gönder gönder" diyen uygulama kaldırılır.
- Kapı sonrası: gönderim sıklığı kapısı geçilmediyse önce bu story öne alınır.

## STORY: Aile Yapbozu 22: Video mesaj
labels: tema:gonderen, P3, efor:L, SONRA
milestone: SONRA

**Aile Yapbozu** epic'inin **22. story'si** (epic #EPIC). Tema: Gönderen akışı.

**Öncelik:** P3 · **Efor:** L · **Milestone:** SONRA · **Dokunduğu yer:** `VideoRecorder` eklentisi, Storage kuralları (`video/mp4` ≤ 8 MB), `VideoCard` (tamamlama), transcoding (Cloud Function ya da istemci)

### Kullanıcı hikâyesi
Bir gönderen olarak sesin yerine 15 saniyelik bir video (torunun "Anneanne seni seviyorum" demesi) göndermek isterim, böylece tamamlama anı daha güçlü olsun.

### Kabul kriterleri
- [ ] **Senaryo: Kayıt** · Verilen adım 3'te "Video" seçildi · Eğer kayıt yapılırsa · O zaman ön kamera, ≤ 15 sn, 480p H.264, ≤ 8 MB; istemcide kodlama (Android `MediaCodec`, iOS `AVAssetExportSession`).
- [ ] **Senaryo: Oynatma** · Verilen yapboz tamamlandı · Eğer video varsa · O zaman tam genişlik `VideoPlayer` kartı, büyük "Tekrar izle", ses otomatik açık; eski cihazda donma yoksa (test matrisi).
- [ ] **Senaryo: Plan** · Verilen ücretsiz gönderen ve kapsanmayan çember · Eğer video seçilirse · O zaman yalnızca Aile Planı (Storage maliyeti); kart teklifi.
- [ ] **Senaryo: Saklama** · Verilen 90 gün · Eğer dolarsa · O zaman video da silinir.

### Teknik görevler (Unity/C#)
- [ ] `VideoRecorder` eklentisi, Unity `VideoPlayer` ile oynatma (H.264 baseline, 2 GB cihaz testi), Storage kuralı ve boyut sınırı, `SendDraft.videoLocalPath`.
- [ ] Maliyet modeli: çember başına aylık ≤ 40 MB; Blaze planı gerekir.

### Analitik
- `video_recorded` {sec: int, mb: float} · `video_played` {replay: bool}

### Notlar
- Sesli mesaj verisi (voice_played oranı, replay) güçlüyse öncelik; değilse rafta kalır.

## STORY: Aile Yapbozu 23: Seri motoru: Büyük Sudoku'ya geçiş
labels: tema:backend, P3, efor:L, SONRA
milestone: SONRA

**Aile Yapbozu** epic'inin **23. story'si** (epic #EPIC). Tema: Altyapı.

**Öncelik:** P3 · **Efor:** L · **Milestone:** SONRA · **Dokunduğu yer:** `MustyBigCalm` UPM paketi (yeni repo), `BigCalmTheme`, `LargeButton/Card/Keypad`, `TtsService`, `CircleService`, `ICircleBackend`, Firebase şablonu, `PlayerHome` iskeleti

### Kullanıcı hikâyesi
Bir geliştirici olarak Aile Yapbozu'nda kanıtlanmış 60+ arayüz sistemini, çember bağlantısını, Firebase katmanını, TTS'i ve bildirim akışını ayrı bir pakete çıkarmak isterim, böylece Büyük Sudoku ve Büyük Kelime Avı 1 haftada "oyun motoru + içerik" olarak çıksın ve abonelik seri genelinde geçerli olsun.

### Kabul kriterleri
- [ ] **Senaryo: Paket ayrışması** · Verilen Aile Yapbozu repo'su · Eğer `MustyBigCalm` paketi çıkarılırsa · O zaman Aile Yapbozu paketi UPM ile tüketir, davranış değişmez (regresyon: ThemeLint, Emulator testleri, Play Mode testleri yeşil).
- [ ] **Senaryo: Yeni uygulama** · Verilen boş proje + `MustyCozyCore` + `MustyBigCalm` · Eğer "Büyük Sudoku" iskeleti kurulursa · O zaman RoleGate, çember bağlantısı, PlayerHome, Ayarlar, bildirim ve Firebase (aynı proje, `app` alanı) 1 günde çalışır; yalnızca `IPuzzleEngine` uygulaması (sudoku) yazılır.
- [ ] **Senaryo: Ortak çember ve plan** · Verilen Aile Yapbozu'nda plan almış kullanıcı · Eğer Büyük Sudoku kurulursa · O zaman aynı çemberler ve plan tanınır (`circles` ortak, `apps[]` alanı); gönderen Sudoku'da "zorluk + not" gönderir (fotoğraf yerine bulmaca seed'i).
- [ ] **Senaryo: Çapraz tanıtım** · Verilen oyuncu PlayerHome · Eğer seri uygulaması yüklüyse · O zaman "Hazır yapbozlar" yanında "Büyük Sudoku" kartı (yüklü değilse gönderen tarafında öneri; oyuncu tarafında mağaza linki yok).

### Teknik görevler (Unity/C#)
- [ ] Repo `MustyBigCalm`: `Runtime/{Theme,UI,Tts,Circles,Backend,Push,Privacy}`, `Editor/ThemeLint`, `Tests`; `package.json`, sürümleme.
- [ ] `IPuzzleEngine {Load(payload), OnCompleted, HintAsync()}` arayüzü; Aile Yapbozu `JigsawEngine` olarak uyarlanır.
- [ ] Firestore şema değişikliği: `puzzles.app` alanı, `circles.apps[]`; kurallar güncellenir; plan `verifyPurchase` seri SKU'ları (`plus_monthly` ortak ürün id, Play'de uygulama başına ayrı ama sunucuda tek plan).
- [ ] `docs/series-engine.md`: nelerin paket, nelerin uygulama olduğu.

### Analitik
- `series_app_card_tap` {app: string, role: string}

### Notlar
- Kapı geçilmeden başlanmaz; geçilirse Büyük Sudoku'nun epic'i bu story'nin çıktısına yazılır.
- Aboneliğin seri genelinde geçerli olması Play'de tek uygulama-tek abonelik kuralıyla çelişir; sunucu tarafı eşleme ile çözülür (bir uygulamada alınan plan hepsinde tanınır), hukuki/mağaza politikası doğrulanacak.

## STORY: Aile Yapbozu 24: Sesli yanıt ve sesli sohbet
labels: tema:paylasim, P3, efor:M, SONRA
milestone: SONRA

**Aile Yapbozu** epic'inin **24. story'si** (epic #EPIC). Tema: Paylaşım ve bağ.

**Öncelik:** P3 · **Efor:** M · **Milestone:** SONRA · **Dokunduğu yer:** Tamamlama kartı "Sesli yanıt ver", `VoiceRecorder` oyuncu tarafı, `replies` alt koleksiyonu, SenderHome yanıt kartı

### Kullanıcı hikâyesi
Bir oyuncu olarak torunumun mesajını dinledikten sonra tek büyük düğmeyle ona sesli yanıt bırakmak isterim ("Çok güzeldi, ellerine sağlık"), böylece "Teşekkür" düğmesinden fazlası olsun; bir gönderen olarak annemin sesini dinlemek isterim.

### Kabul kriterleri
- [ ] **Senaryo: Yanıt düğmesi** · Verilen tamamlama kartı · Eğer "Sesli yanıt ver" (88 px, ikincil) basılırsa · O zaman tek dokunuşla kayıt başlar (basılı tutma değil), büyük sayaç, 30 sn sınır, "Gönder" / "Vazgeç"; mikrofon izni 60+ dilinde açıklanır.
- [ ] **Senaryo: Gönderen tarafı** · Verilen yanıt geldi · Eğer SenderHome açılırsa · O zaman satırda "Annenden sesli yanıt" ve FCM bildirimi; dinleme kartı.
- [ ] **Senaryo: Sohbet dizisi** · Verilen bir yapbozda 1 mesaj + 1 yanıt · Eğer gönderen tekrar yanıtlarsa · O zaman aynı yapboz altında en fazla 5 mesajlık dizi (`replies`), oyuncu tarafında "Bitirdiklerin"de rozet.
- [ ] **Senaryo: Saklama** · Verilen 90 gün · Eğer dolarsa · O zaman yanıtlar yapbozla birlikte silinir.

### Teknik görevler (Unity/C#)
- [ ] `puzzles/{id}/replies/{rid} {fromUid, audioPath, sec, createdAt}`; Storage kuralı; `onReplyCreated` FCM.
- [ ] Oyuncu tarafı kayıt UI (60+ tema, tek dokunuş), `VoiceRecorder` yeniden kullanımı.

### Analitik
- `reply_recorded` {role: string, sec: int} · `reply_played` {role: string}

### Notlar
- `thanks_sent` oranı yüksekse (≥ %50) bu story öne alınır; oyuncunun konuşma isteği var demektir.
- Gerçek zamanlı sesli sohbet (arama) kapsam dışı kalır; WhatsApp bunu zaten yapıyor.

## STORY: Aile Yapbozu 25: Huzurevi ve bakım evi lisansı (B2B)
labels: tema:monetizasyon, P3, efor:L, SONRA
milestone: SONRA

**Aile Yapbozu** epic'inin **25. story'si** (epic #EPIC). Tema: Monetizasyon.

**Öncelik:** P3 · **Efor:** L · **Milestone:** SONRA · **Dokunduğu yer:** `organizations/{id}` modeli, kurum paneli (web), tablet kiosk modu, toplu lisans faturalama (uygulama dışı)

### Kullanıcı hikâyesi
Bir bakım evi etkinlik sorumlusu olarak kurumun tabletlerinde sakinler için büyük yapbozlar sunmak, ailelerinin fotoğraf göndermesini sağlamak ve bunu tek bir kurum lisansıyla ödemek isterim.

### Kabul kriterleri
- [ ] **Senaryo: Kurum kurulumu** · Verilen web paneli · Eğer kurum oluşturulursa · O zaman N sakin profili, her sakine bir aile çemberi ve davet kodu, tabletlerde "sakin seç" ekranı (fotoğraf + ad, 120 px kartlar).
- [ ] **Senaryo: Kiosk** · Verilen kurum tableti · Eğer kiosk modu açıksa · O zaman Ayarlar ve rol değişimi PIN'li; uygulama kapanınca sakin seçimine döner; sesli mesajlar kulaklıkla ya da hoparlörle (ayar).
- [ ] **Senaryo: Lisans** · Verilen kurum lisansı (uygulama dışı fatura) · Eğer aktifse · O zaman tüm sakin çemberleri plan kapsamında sayılır; mağaza içi satın alma yok.
- [ ] **Senaryo: Gizlilik** · Verilen kurum · Eğer sakin verisi işlenirse · O zaman kurum veri işleyen sözleşmesi, sakin başına rıza kaydı, kurumdan ayrılınca veri silme.

### Teknik görevler (Unity/C#)
- [ ] `organizations` modeli ve kurallar; kiosk modu (`KioskConfig`, PIN); web panel (Firebase Hosting + basit SPA); lisans alanı `circles.orgId`.
- [ ] Satış materyali ve fiyatlandırma (sakin başına aylık) `docs/b2b.md`.

### Analitik
- `org_session` {org_id_hash: string, residents_active: int}

### Notlar
- Pazar doğrulaması önce: TR'de 3 huzurevi, UK'de 2 care home ile görüşme; teknik iş ondan sonra.
- Solo geliştirici için satış döngüsü ağır; ortak/bayi modeli düşünülür.

## STORY: Aile Yapbozu 26: Fotoğraf albümü (Anılar)
labels: tema:meta, P3, efor:M, SONRA
milestone: SONRA

**Aile Yapbozu** epic'inin **26. story'si** (epic #EPIC). Tema: Meta ve tutma.

**Öncelik:** P3 · **Efor:** M · **Milestone:** SONRA · **Dokunduğu yer:** `AlbumScreen` (oyuncu + gönderen), saklama kuralı istisnası (`keep: true`), Storage maliyeti, baskı/paylaşım

### Kullanıcı hikâyesi
Bir oyuncu olarak bitirdiğim yapbozların fotoğraflarını ve seslerini bir albümde tekrar görmek/dinlemek; bir gönderen olarak Aile Planı ile bu albümün 90 gün sonra silinmemesini isterim.

### Kabul kriterleri
- [ ] **Senaryo: Albüm** · Verilen tamamlanmış yapbozlar · Eğer "Anılar" açılırsa · O zaman aya göre gruplu fotoğraf ızgarası (2 sütun, tablette 3), dokununca tam fotoğraf + sesli mesaj + not + "Emre gönderdi · 12 Mart".
- [ ] **Senaryo: Saklama istisnası** · Verilen Aile Planı aktif · Eğer yapboz tamamlanırsa · O zaman `keep: true` ve `scheduledCleanup` atlar; plan biterse 30 gün uyarı sonra silinir (gönderene bildirim, dışa aktarma seçeneği).
- [ ] **Senaryo: Dışa aktarma** · Verilen gönderen · Eğer "Albümü indir" derse · O zaman ZIP (fotoğraflar + m4a + notlar JSON) Cloud Function ile üretilir, imzalı link 24 saat (GDPR "verilerimi indir" karşılığı da bu).
- [ ] **Senaryo: Oyuncu tarafı sade** · Verilen oyuncu · Eğer Anılar açılırsa · O zaman silme ve indirme yok, yalnızca bakma/dinleme; silme gönderende.

### Teknik görevler (Unity/C#)
- [ ] `AlbumScreen` UI (iki tema), `keep` alanı ve kural, `exportAlbum` Cloud Function (ZIP, Storage geçici), bildirimler.
- [ ] Storage maliyet modeli: Aile Planı başına 500 MB tavan; aşımda eski yapbozlar önce.

### Analitik
- `album_view` {role: string, items: int} · `album_export` {items: int, mb: float}

### Notlar
- Aile Planı değer önerisinin en güçlü adayı ("anılar kaybolmaz"); 18. story dönüşümü düşükse öne çekilir.
- 15. story'nin "verilerimi indir" boşluğunu da kapatır.
