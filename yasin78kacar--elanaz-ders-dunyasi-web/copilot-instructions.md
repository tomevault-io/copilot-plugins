## elanaz-ders-dunyasi-web

> Bu dosya, Ders Dünyası projesinde çalışan AI agent'ın (Cursor / Claude Code) her oturumda uyması gereken kuralları içerir. Bu kurallar Yasin tarafından belirlenmiştir ve istisnasız uygulanır.

# CLAUDE.md — Ders Dünyası Proje Kuralları

Bu dosya, Ders Dünyası projesinde çalışan AI agent'ın (Cursor / Claude Code) her oturumda uyması gereken kuralları içerir. Bu kurallar Yasin tarafından belirlenmiştir ve istisnasız uygulanır.

## Proje Nedir

İlkokul 1-4. sınıf için Türkçe eğitim web uygulaması. Kullanıcılar: Yasin'in kızları. React + Vite + TypeScript, Vercel'de yayında, localStorage tabanlı çoklu profil (sunucu yok).

Bölümler: Öğrenme Köşesi (Tema 1 Soru Kelimeleri, Tema 2 Saat Okuma, Tema 3 Boşluk Doldurma, Tema 4 Dijital Saat), Hikaye Köşesi, 5N1K, Oyunlar, ders bölümleri.

## Teknik Kısıtlar (İHLAL EDİLMEZ)

- Harici paket/API ekleme YASAK. Mevcut kütüphanelerle çözülür.
- Veri / localStorage / skor yapısına DOKUNMA. İçerik düzeltmek bu yasağa girmez; yapıyı (anahtar, id, şema) değiştirmek girer.
- Mevcut bölümleri bozma. Yeni bir şey eklerken çalışan bölümlere yapısal dokunma.
- Soru yükleme mevcut mekanizmayla (fetch) çalışır. Offline gerekliliği yok.
- Referans/yedek branch'leri silme (timer-yedek, tasarim-cila).

## Çalışma Tarzı

- Tek tur = tek iş. Büyük işleri tek promptta toplama — agent birini atlar.
- Büyük içerik üretiminde partiler halinde çalış (ör. 50'şerli), her partiyi ayrı taslak dosyaya yaz.
- Önce taslak (~/Downloads/), kontrol + onay, sonra canlı veri dosyasına işle.
- Canlı veriye toplu değişiklikten önce yedek al (cp <dosya> ~/Downloads/<dosya>_yedek.json).
- Kayıtları id bazlı eşleştir, sıra bazlı değil.
- Değişiklikten sonra toplam kayıt sayısını doğrula.

## Kalite Kuralı (EN ÖNEMLİ)

- Push'tan önce Yasin gözle test etmeden onay yok.
- AI'ın "kontrol ettim / sorunlu madde yok" demesi kontrol SAYILMAZ. Nihai onay Yasin'in uygulamada gözle görmesiyle olur.
- tsc -b && vite build exit 0 olması "çalışıyor" anlamına GELMEZ — sadece derlendiği anlamına gelir.
- Push korumalı main'e yapılır, onayı (Run) Yasin verir. Agent kendi başına push etmez.
- Her iş bitince git diff --stat ve git status göster. Beklenmedik dosya değiştiyse DUR ve bildir.

## Talimat Formatı

- Her kod bloğu nereye yapıştırılacağını belirtir: CURSOR AGENT'A YAPIŞTIR veya TERMİNALE YAPIŞTIR.

## Eleştirel Yaklaşım

- Yasin'e otomatik "haklısın" DEME. Komutlarını eleştir, hatalarını ve boşluklarını söyle. Komutları her zaman doğru değildir.
- Bir istek proje kuralları, pedagoji ya da teknik gerçeklerle çelişiyorsa, uygulamadan önce açıkça belirt.
- Tempo yüksek; Yasin "dur" demedikçe yavaşlatma önerme. Ancak güvenlik/kalite riski varsa dürüstlük tempodan önce gelir.

## İçerik Standartları

- Hedef yaş 2. sınıf (7-8): basit kelimeler, kısa cümleler, günlük hayattan somut örnekler. Soyut/tanım sorusu yok.
- Kopya/tekrar YASAK. Sorular ve hikayeler birbirinin tekrarı olmaz. Farklı isim, olay, bağlam.
- Cümleler mantıklı olmalı — öğeler tutarlı, çelişki yok.
- Çeldiriciler dengeli: doğru cevap açıkça doğru, yanlış şıklar aynı türden ama uymayan.
- Doğru cevap pozisyonu çeşitli, hep aynı yerde değil.
- Soru kelimesi/tür dağılımı dengeli.
- Karakter ismi metinde "Elanaz" yazılır; uygulama otomatik farklı isme çevirir (dokunma).

## Türkçe Dil Kuralları

- İmla ve cümle başı büyük harf doğru.
- İyelik ekleri doğru: 3. tekil "-si/-sı" (elbisesini, değil elbiseni).
- İsim ekleri ünlü uyumu + kaynaştırma ile doğru (Ece'nin / Ada'ya, değil Ece'ın / Ada'a).
- Üç nokta sonrası küçük harf doğrudur, otomatik "düzeltme" ile bozma.
- Türkçe dil kararları AI tahminine bırakılmaz; belirsizse sorulur.

## Çocuk Güvenliği

- Tüm içerik ilkokul çocuğu için uygun, güvenli, yaşa uygun. Korkutucu/üzücü/uygunsuz tema yok.

---
> Source: [yasin78kacar/elanaz-ders-dunyasi-web](https://github.com/yasin78kacar/elanaz-ders-dunyasi-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
