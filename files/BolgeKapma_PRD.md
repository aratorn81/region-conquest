# Bölge Kapma — Product Requirements Document (PRD)

**Versiyon:** 1.0  
**Tarih:** Haziran 2026  
**Platform:** Flutter (iOS + Android)  
**Durum:** Planlama Aşaması

---

## 1. Ürün Özeti

**Bölge Kapma**, harita tabanlı bir strateji oyunudur. İki oyuncu sırayla haritadaki bölgelere (il, eyalet, departman) tıklayarak o bölgeyi kendi rengine boyar. Oyunun amacı rakibi olabildiğince çok parçaya bölmektir — **en çok parçaya ayrılan oyuncu kaybeder**, az parça = güçlü = kazanan.

### Temel Konsept
- Her tur oyuncular sırayla bir bölge seçer
- Seçilen bölge o oyuncunun rengiyle boyanır
- Tüm bölgeler kaplandığında oyun biter
- Kendi bölgeleri arasında **bağlantısı kopan** oyuncu daha çok parçaya ayrılır
- **Az parça = bütün kalmak = kazanmak**

---

## 2. Hedef Kitle

| Segment | Açıklama |
|---------|----------|
| Birincil | 12–35 yaş, mobil oyun oynayan, strateji oyunlarına ilgi duyan |
| İkincil | Coğrafya meraklıları, eğitim amaçlı kullananlar |
| Platform | iOS ve Android akıllı telefon kullanıcıları |

---

## 3. Platform & Teknoloji

| Kalem | Seçim | Gerekçe |
|-------|-------|---------|
| Framework | **Flutter** | Tek kod → iOS + Android + Web; SVG/harita desteği güçlü |
| Dil | Dart | Flutter'ın native dili |
| Durum Yönetimi | Riverpod veya Bloc | Flutter ekosisteminde en yaygın |
| Harita Render | flutter_svg + CustomPainter | GeoJSON'dan üretilen SVG path'leri |
| Ses | audioplayers veya flame_audio | Ses efektleri ve arka plan müziği |
| Animasyon | Flutter Animation API + Rive | Konfetti, parça büyüme efektleri |
| Yerel Depolama | Hive veya SharedPreferences | Ayarlar, istatistikler |
| Minimum SDK | iOS 13+, Android API 21+ | Geniş cihaz desteği |

---

## 4. Ekranlar ve Navigasyon

```
Splash Screen
    └── Ana Menü
            ├── Ülke Seçimi
            ├── Mod Seçimi (2 Oyuncu / AI Seviyeleri)
            ├── Ayarlar
            └── Hakkında
                    └── Oyun Ekranı
                                └── Sonuç Ekranı
                                        ├── Tekrar Oyna
                                        └── Ana Menüye Dön
```

---

## 5. Ekran Detayları

### 5.1 Splash Screen
- Uygulama logosu + animasyon (1.5–2 saniye)
- Arka plan: koyu gradient (#0f0c29 → #302b63)
- Yükleme sırasında harita verisi ön belleğe alınır

---

### 5.2 Ana Menü

**Görsel Tasarım:**
- Koyu mor gradient arka plan
- Yüzen 🌍 emoji logosu (float animasyonu)
- "Bölge Kapma" başlığı, kalın beyaz font
- "Az parçaya ayrılan kazanır!" alt başlığı

**Ülke Seçimi (kart grid — 3 sütun):**

| Kart | Bayrak | İsim | Bölge Sayısı |
|------|--------|------|-------------|
| Türkiye | 🇹🇷 | Türkiye | 81 il |
| ABD | 🇺🇸 | ABD | 49 eyalet |
| Fransa | 🇫🇷 | Fransa | 96 departman |

- Seçili kart: beyaz border + glow efekti
- Kart arka planı: ilgili ülkenin bayrak renkleriyle hafif gradient

**Not — Gelecek Versiyonlar:**
> V2'de ülke listesi genişletilebilir. Almanya (16 eyalet), İspanya (50 il), İtalya (20 bölge) gibi ülkeler eklenebilir. Mimari buna göre hazırlanmalı (harita verisi dinamik yüklenebilir yapıda olmalı).

**Mod Seçimi (kart grid — 2 sütun):**

| Kart | İkon | İsim | Açıklama | Zorluk |
|------|------|------|----------|--------|
| 2 Oyuncu | 👥 | 2 Oyuncu | Aynı ekranda sırayla | — |
| Kolay AI | 🟢 | Kolay AI | Rastgele oynar | ⭐ |
| Normal AI | 🟡 | Normal AI | Akıllıca oynar | ⭐⭐ |
| Zor AI | 🔴 | Zor AI | Çok zekice oynar | ⭐⭐⭐ |

**Oyuna Başla Butonu:**
- Turuncu gradient, büyük, tam genişlik
- Ülke + mod seçilmeden devre dışı (disabled state)
- Tıklamada haptic feedback

---

### 5.3 Oyun Ekranı

#### Üst Bar
- Sol: Sıra göstergesi ("🔴 Senin sıran" / "🔵 Rakibin sırası" / "🤖 AI oynuyor")
- Sağ: Kalan bölge sayısı ("65 bölge kaldı")

#### Skor Kartları (yan yana, 2 kart)

Her kart şunları gösterir:
- Oyuncu rengi ve etiketi (KIRMIZI / MAVİ / 🤖 AI)
- **İl/bölge sayısı** (büyük rakam)
- **Parça sayısı** (kritik gösterge — az = iyi)
- Aktif oyuncunun kartı: renkli border + glow animasyonu

#### Harita Alanı
- Ekranın büyük bölümünü kaplar
- SVG tabanlı, pinch-to-zoom ve pan desteği (isteğe bağlı V2)
- Her bölge tıklanabilir path
- Renkler:
  - Boş bölge: `#5cb85c` (yeşil)
  - Oyuncu 1: `#e74c3c` (kırmızı)
  - Oyuncu 2: `#2980b9` (mavi)
  - Deniz/arka plan: haritaya özel renk
- Bölge isimleri SVG text olarak üzerinde

#### Bölge Tıklama Davranışı
1. Kullanıcı boş bölgeye dokunur
2. Haptic feedback (light impact)
3. Bölge rengi anlık değişir + parlama (brightness flash) animasyonu
4. Ses efekti çalar (oyuncuya özel ton)
5. Parça sayısı güncellenir
6. **Eğer yeni parça oluştuysa:** konfetti patlaması + ayrılma sesi
7. Sıra değişir

#### AI Davranışı

**Kolay:**
- %100 rastgele seçim
- 400–700ms gecikme

**Normal:**
- Komşu bölgeleri tercih eder (kendi bölgelerine bitişik)
- Bölünmeyi engellemeye çalışır
- 700–1000ms gecikme

**Zor:**
- Kendi bloklarını birleştirmeye öncelik verir
- Rakibin bloklarını bölmeye çalışır
- En az parça oluşturacak hamleyi simüle eder
- 1000–1400ms gecikme (düşünüyor hissi)

**AI göstergesi:** Küçük animasyonlu üç nokta pill → "🤖 AI düşünüyor..."

---

### 5.4 Sonuç Ekranı

**Üst Bölüm — Oynanmış Harita:**
- Oyun bittiğindeki harita durumu (salt okunur, tıklanamaz)
- Küçük boyutta tam harita görünümü

**Alt Bölüm — Sonuç Kartı:**

| Durum | İkon | Başlık Rengi |
|-------|------|-------------|
| Kırmızı Kazandı | 🏆 | #e74c3c |
| Mavi/AI Kazandı | 🏆 / 🤖 | #2980b9 |
| Berabere | 🤝 | #f39c12 |

Sonuç kartı içeriği:
- Kazanan/berabere ikonı (bounce animasyonu)
- Kazanan metni ("Kazandın!" / "AI Kazandı!" / "Berabere!")
- Alt açıklama ("3 parça ile 5 parçayı geçtin!")
- İki skor kartı: Kırmızı X parça | Mavi Y parça
- **Tekrar Oyna** butonu (turuncu)
- **Ana Menü** butonu (şeffaf, outline)

**Konfetti:** Kazananın renginde büyük konfetti yağmuru (yukarıdan aşağı)

---

## 6. Oyun Mekaniği — Teknik Detaylar

### 6.1 Bağlantı Algoritması (Connected Components)

Parça sayısını hesaplamak için **Union-Find (Disjoint Set Union)** veya **BFS/DFS** algoritması kullanılır.

```
Algoritma: DFS ile Bağlı Bileşen Sayımı

1. Oyuncunun sahip olduğu bölge ID'lerini al
2. Ziyaret edilmemiş her bölgeden DFS başlat
3. Komşuluk listesinden bağlı bölgeleri ziyaret et
4. Her bağımsız DFS = 1 parça
5. Toplam parça sayısını döndür
```

**Kritik Kural:** Komşuluk listesi (`adjacency list`) gerçek coğrafi sınırlara dayanmalıdır. İki bölge ancak gerçekte sınır paylaşıyorsa komşu sayılır.

### 6.2 Komşuluk Verisi

Her harita için önceden hesaplanmış komşuluk listesi JSON formatında saklanır:

```json
{
  "1": [8, 37, 42, 44, 47, 58, 62, 64],
  "2": [26, 29, 42, 55, 68],
  ...
}
```

Bu veri harita SVG'si ile birlikte assets klasöründe bulunur.

### 6.3 Harita Verisi Formatı

```json
{
  "id": 1,
  "name": "Adana",
  "path": "M451.3,354.3 L...",
  "cx": 461.6,
  "cy": 336.9
}
```

| Alan | Açıklama |
|------|----------|
| `id` | Benzersiz bölge kimliği |
| `name` | Bölge adı (haritada gösterilir) |
| `path` | SVG path verisi |
| `cx`, `cy` | Etiket merkez koordinatı |

---

## 7. Ses Tasarımı

| Olay | Ses | Açıklama |
|------|-----|----------|
| Bölge alındı (Oyuncu 1) | Yükselen 3 ton (320→480 Hz) | Kısa, neşeli |
| Bölge alındı (Oyuncu 2) | Yükselen 3 ton (220→330 Hz) | Farklı frekans |
| Parça oluştu | Alçalan dramatik ses (140→80 Hz) | Tehlike hissi |
| Oyun kazanıldı | Kısa fanfar (523→659→784→1047 Hz) | Kutlama |
| Oyun kaybedildi | Alçalan melodi (280→220→160 Hz) | Hüzünlü |
| Menü tıklama | Hafif tık sesi | UI feedback |

**Ses Motoru:** `audioplayers` paketi  
**Format:** MP3 (arka plan müziği) + WAV (kısa efektler)  
**Varsayılan:** Sesler açık, kullanıcı ayarlardan kapatabilir

---

## 8. Animasyon Sistemi

### 8.1 Konfetti

| Tetikleyici | Tür | Yoğunluk |
|-------------|-----|----------|
| Yeni parça oluştu | Merkezi patlama | 80 parçacık |
| Oyun kazanıldı | Yukarıdan yağmur | 200 parçacık |

**Parçacık özellikleri:**
- Şekil: dikdörtgen veya daire (rastgele)
- Renk: kazananın rengi (kırmızı/mavi tonları)
- Fizik: yerçekimi, rotasyon, solma

### 8.2 Diğer Animasyonlar

| Animasyon | Tetikleyici | Süre |
|-----------|-------------|------|
| Bölge parlama | Bölge alındığında | 250ms |
| Skor güncelleme | Her hamle | 150ms |
| Kart aktif glow | Sıra değişimi | 300ms |
| Logo float | Menü açık | Sürekli (3s döngü) |
| Sonuç ikonu bounce | Sonuç ekranı | Sürekli (1.5s döngü) |
| Sonuç kartı pop-in | Ekran geçişi | 500ms (spring) |

---

## 9. Renk Paleti & Tasarım Sistemi

### 9.1 Renkler

```
Arka Plan (Derin)   : #0f0c29
Arka Plan (Orta)    : #302b63  
Arka Plan (Açık)    : #24243e
Arka Plan (Kart)    : rgba(255,255,255,0.05)

Oyuncu 1 Kırmızı    : #e74c3c
Oyuncu 1 Koyu       : #c0392b
Oyuncu 2 Mavi       : #2980b9
Oyuncu 2 Koyu       : #1a5fa8

Boş Bölge Yeşil     : #5cb85c
Aksan Turuncu       : #f39c12
Aksan Altın         : #ffd700

Metin Beyaz         : #ffffff
Metin İkincil       : rgba(255,255,255,0.6)
Metin Üçüncül       : rgba(255,255,255,0.35)
```

### 9.2 Tipografi

| Kullanım | Font | Ağırlık | Boyut |
|---------|------|---------|-------|
| Başlık | Nunito | 900 (Black) | 28–36sp |
| Skor rakamı | Nunito | 900 | 32–40sp |
| Parça sayısı | Nunito | 900 | 22–28sp |
| Bölge etiket | Nunito | 700 | 6–8sp (SVG) |
| Buton | Nunito | 900 | 15–18sp |
| Açıklama | Nunito | 700 | 11–14sp |

### 9.3 Border Radius

```
Küçük kart   : 12dp
Orta kart    : 14–16dp
Büyük kart   : 18–20dp
Buton        : 50dp (tam yuvarlak)
```

---

## 10. Dokunmatik & Etkileşim

| Bileşen | Davranış |
|---------|----------|
| Bölge tıklama | Tek dokunuş → renk değişimi |
| Kart seçimi | Tek dokunuş → seçili/seçilmemiş |
| Butonlar | Scale 0.97 on press + haptic |
| Geri tuşu (Android) | Oyun içinde: duraklat/çık dialog |
| Çift dokunuş | Kullanılmıyor (V1) |
| Kaydırma (harita) | V2'de zoom/pan |

**Haptic Feedback Seviyeleri:**

| Olay | Haptic |
|------|--------|
| Bölge alındı | Light impact |
| Parça oluştu | Medium impact |
| Oyun kazanıldı | Heavy impact + success |
| Menü tıklama | Selection |

---

## 11. Ayarlar Ekranı

| Ayar | Tip | Varsayılan |
|------|-----|-----------|
| Ses Efektleri | Toggle | Açık |
| Arka Plan Müziği | Toggle | Kapalı |
| Haptic Feedback | Toggle | Açık |
| Oyun Dili | Seçim (TR/EN) | Cihaz diline göre |

---

## 12. Hata Durumları & Edge Case'ler

| Durum | Davranış |
|-------|----------|
| Zaten alınmış bölgeye tıklama | Hiçbir şey olmaz, feedback yok |
| AI sırası geldiğinde oyuncu tıklar | Tıklama engellenir (busy state) |
| Uygulama arka plana geçer | Oyun durumu korunur, AI durdurulur |
| Uygulama kapanır | Oyun kaybolur (V1'de kaydetme yok) |
| Tüm bölgeler dolduğunda | Otomatik sonuç ekranına geçiş (600ms gecikme) |
| Berabere parça sayısı | "Berabere!" ekranı gösterilir |

---

## 13. Proje Klasör Yapısı (Flutter)

```
lib/
├── main.dart
├── app/
│   ├── app.dart                 # MaterialApp, tema, routing
│   └── router.dart              # Go Router veya Navigator 2.0
├── features/
│   ├── menu/
│   │   ├── menu_screen.dart
│   │   ├── map_selector.dart
│   │   └── mode_selector.dart
│   ├── game/
│   │   ├── game_screen.dart
│   │   ├── game_controller.dart  # Oyun mantığı
│   │   ├── map_widget.dart       # SVG harita render
│   │   ├── score_card.dart
│   │   └── ai_engine.dart        # AI algoritması
│   └── result/
│       ├── result_screen.dart
│       └── confetti_widget.dart
├── models/
│   ├── map_data.dart            # Şehir/bölge modeli
│   ├── game_state.dart          # Oyun durumu
│   └── player.dart
├── services/
│   ├── audio_service.dart       # Ses yönetimi
│   ├── haptic_service.dart      # Titreşim
│   └── settings_service.dart   # Ayarlar (Hive)
└── utils/
    ├── adjacency.dart           # Komşuluk algoritması
    └── connected_components.dart # Parça sayısı hesaplama

assets/
├── maps/
│   ├── turkey.json              # SVG path + komşuluk verisi
│   ├── usa.json
│   └── france.json
├── sounds/
│   ├── claim_red.wav
│   ├── claim_blue.wav
│   ├── split.wav
│   ├── win.wav
│   └── lose.wav
└── fonts/
    └── Nunito/
```

---

## 14. Harita Veri Kaynakları

| Harita | Kaynak | Bölge Sayısı | Koordinat Sistemi |
|--------|--------|-------------|-------------------|
| Türkiye | Özel GeoJSON (gerçek il sınırları) | 81 | Mercator projeksiyonu |
| ABD | PublicaMundi GeoJSON | 49 | Mercator projeksiyonu |
| Fransa | gregoiredavid/france-geojson | 96 | Mercator projeksiyonu |

**SVG ViewBox Boyutları:**

| Harita | viewBox |
|--------|---------|
| Türkiye | 0 0 900 420 |
| ABD | 0 0 900 560 |
| Fransa | 0 0 800 700 |

---

## 15. Performans Gereksinimleri

| Metrik | Hedef |
|--------|-------|
| İlk açılış süresi | < 2 saniye |
| Harita render süresi | < 500ms |
| Bölge tıklama tepkisi | < 100ms |
| Parça sayısı hesaplama | < 16ms (bir frame) |
| APK boyutu | < 30MB |
| RAM kullanımı | < 150MB |
| Minimum FPS | 60fps (animasyonlar) |

---

## 16. Gelecek Versiyon Planı (V2+)

| Özellik | Öncelik | Açıklama |
|---------|---------|----------|
| Yeni haritalar | Yüksek | Almanya, İspanya, İtalya, Dünya haritası |
| Skor sistemi | Orta | Kazanma serisi, toplam oyun sayısı |
| Online çok oyunculu | Yüksek | Gerçek zamanlı Firebase/Supabase ile |
| Kullanıcı hesabı | Orta | Google/Apple ile giriş |
| Liderlik tablosu | Düşük | Online sıralama |
| Harita zoom/pan | Orta | Pinch-to-zoom, özellikle Fransa için |
| Oyun kaydetme | Orta | Yarım kalan oyunu sürdürme |
| Turnuva modu | Düşük | 4+ oyuncu bracket sistemi |
| Renk seçimi | Düşük | Oyuncular kendi renk temasını seçer |
| Zamanlı mod | Düşük | Her hamle için süre sınırı |

---

## 17. Test Gereksinimleri

### Fonksiyonel Testler

- [ ] Tüm ülkelerde oyun başlatılabilir
- [ ] Tüm AI seviyeleri oynanabilir
- [ ] 2 oyuncu modu çalışır
- [ ] Parça sayısı doğru hesaplanır (her harita için birim test)
- [ ] Kazanan doğru belirlenir (az parça = kazanan)
- [ ] Tüm bölgeler doldurulduğunda oyun biter
- [ ] Sonuç ekranında harita doğru gösterilir
- [ ] Ayarlar kaydedilir ve uygulanır

### Parça Sayısı Birim Testleri

```dart
test('Bağlı bölgeler 1 parça sayılır', () {
  final owned = {1, 8, 53}; // Adana, Antalya, Konya - bağlı
  expect(countComponents(owned, turkeyAdj), equals(1));
});

test('Bağlantısız bölgeler ayrı parça sayılır', () {
  final owned = {1, 10, 45}; // Adana, Artvin, Kars - bağlantısız
  expect(countComponents(owned, turkeyAdj), equals(3));
});
```

### Cihaz Testleri

| Cihaz | OS | Ekran Boyutu |
|-------|-----|-------------|
| iPhone SE | iOS 15+ | 4.7" |
| iPhone 14 Pro | iOS 16+ | 6.1" |
| Samsung Galaxy A54 | Android 13 | 6.4" |
| Samsung Galaxy S23 | Android 13 | 6.1" |
| iPad (isteğe bağlı) | iPadOS 16+ | 10.9" |

---

## 18. Yayın Planı

| Aşama | İçerik | Süre |
|-------|--------|------|
| Alpha | Türkiye haritası, AI (3 seviye), temel UI | 3 hafta |
| Beta | ABD + Fransa haritası, ses, animasyon | 2 hafta |
| RC | Bug fix, performans optimizasyonu | 1 hafta |
| V1.0 | App Store + Play Store yayını | — |

---

## 19. App Store Bilgileri

**Uygulama Adı:** Bölge Kapma  
**İngilizce Ad:** Region Conquest  
**Kategori:** Games → Strategy  
**Yaş Sınırı:** 4+ (şiddet içermez)  
**Anahtar Kelimeler:** harita, strateji, coğrafya, bölge, oyun, türkiye

**Kısa Açıklama (30 karakter):**  
Harita tabanlı strateji oyunu

**Uzun Açıklama:**  
Türkiye, ABD ve Fransa haritalarında rakibini parçalara böl! Bölgeleri stratejik olarak seç, rakibinin topraklarını izole et ve en az parçaya ayrılarak oyunu kazan. Arkadaşınla aynı ekranda ya da 3 zorluk seviyesindeki yapay zekaya karşı oyna.

---

*Bu doküman yaşayan bir belgedir. V2 planlaması başladığında güncellenecektir.*
