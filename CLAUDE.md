# BölgeKapma — İş Akışı

Bu projede geliştirme, dört rollü bir onay zinciriyle yürütülür: **Yönetici → Developer / Tester / Tasarımcı**.

## Roller

- **Yönetici** = bu konuşmayı yürüten ana Claude oturumu (ayrı bir alt-ajan değildir). Diğer üç alt-ajanı sırayla çağırır, sonuçları değerlendirir, onay/red kararını verir ve kullanıcıya özetler. **Onay olmadan hiçbir değişiklik finalize edilmez.**
- **Developer** ([.claude/agents/developer.md](../.claude/agents/developer.md)): Kod değişikliklerini `BolgeKapma_Mobil.html` ve ilgili dosyalarda uygular.
- **Tester** ([.claude/agents/tester.md](../.claude/agents/tester.md)): Oyunu PRD'ye göre test eder (Playwright mevcut), bulguları türüne göre developer'a veya tasarımcıya yönlendirir.
- **Tasarımcı** ([.claude/agents/tasarimci.md](../.claude/agents/tasarimci.md)): UX/Türkçe terminoloji kararları verir, ayrıca benzer oyunlara bakarak iyileştirme önerileri üretip yöneticiye sunar.

## Güncel Odak: Türkiye

Kullanıcı talimatıyla (2026-06-08) çalışma odağı **Türkiye haritası ve içeriğine** daraltıldı. ABD/Fransa ile ilgili yeni bulgu, test veya öneri üretilmemeli — bu konudaki mevcut bulgular [TASK_LOG.md](TASK_LOG.md) içinde "beklemede" olarak işaretlendi. Tester/tasarımcı/developer çalışmaları 81 il, Türkçe terminoloji ve oyun mantığı (Türkiye haritası bağlamında) üzerine yoğunlaşmalı.

## ⚠️ Build senkronizasyonu — değişiklik sonrası ZORUNLU adım

`Claude_projeler/files/BolgeKapma_Mobil.html` **kaynak/canonical dosyadır**, ama oyun gerçekte `capacitor.config.json`'daki `webDir: "www"` üzerinden derlenir. Yani `files/BolgeKapma_Mobil.html`'de yapılan bir değişiklik **otomatik olarak** `www/index.html`'e veya Android derlemesine (`android/app/src/main/assets/public/`) yansımaz — bu, 2026-06-08'de "müzik değişikliği uygulandı ama kullanıcı hâlâ eski müziği duyuyor" şeklinde gerçek bir soruna yol açtı (bkz. [TASK_LOG.md](TASK_LOG.md), "eski müzik çalıyor" bulgusu).

**Oyun dosyasında (`files/BolgeKapma_Mobil.html`) veya ilgili statik varlıklarda (örn. `files/audio/`) her değişiklikten sonra:**

1. `files/BolgeKapma_Mobil.html` → `www/index.html` kopyalanmalı (ve değişen statik varlıklar `www/` altındaki karşılıklarına).
2. `npx cap sync android` çalıştırılmalı (Android derlemesini günceller).
3. **Tester** bu senkronizasyonun yapıldığını doğrulamalı — örn. `www/index.html` ve `android/app/src/main/assets/public/index.html` içinde ilgili yeni kodun (`grep`) var olduğunu kontrol etmeli — ve sonucu `[Tester]` etiketiyle `TASK_LOG.md`'ye yazıp yöneticiye haber vermeli. Bu adım atlanırsa kullanıcı/test ortamı her zaman eski derlemeyi görmeye devam eder.

## Tetikleme

Bu alt-ajanlar **manuel** çağrılır — kullanıcı "developer'a sor", "tester çalıştır", "tasarımcının önerisi nedir" gibi açıkça istediğinde devreye girerler. Otomatik tetiklenmezler.

## İletişim: TASK_LOG.md

Alt-ajan çağrıları birbirinden bağımsız başladığı için (önceki konuşmayı görmezler), tüm bulgular/kararlar/onaylar [TASK_LOG.md](TASK_LOG.md) dosyasına yazılır. Her ajan işine başlamadan önce bu dosyayı okur, bitirince kendi girdisini ekler.

Tipik akış örneği:

1. Tester bir hata bulur → türüne göre `[Tester → Developer]` veya `[Tester → Tasarımcı]` etiketiyle TASK_LOG.md'ye yazar.
2. Eğer tasarımcıya yönlendirildiyse: tasarımcı doğru tanımı PRD'ye göre belirler → `[Tasarımcı → Developer]` etiketiyle yazar.
3. Developer değişikliği uygular → `[Developer]` etiketiyle ne yaptığını yazar.
4. Yönetici (ana oturum) günlüğü değerlendirir, onayını `[Yönetici]` etiketiyle ekler ve kullanıcıya özetler.

Tasarımcı ayrıca kendi inisiyatifiyle iyileştirme önerileri de üretebilir — bunlar `[Tasarımcı → Yönetici]` etiketiyle yazılır ve yönetici (gerekirse kullanıcıya danışarak) önceliklendirir.

## Kaynak doküman

Tüm doğruluk/terminoloji kararlarında [BolgeKapma_PRD.md](BolgeKapma_PRD.md) referans alınır — yeni bir "doğru" tanım icat edilmez.
