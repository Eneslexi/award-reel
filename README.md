# Award Reel · Ödüllü Site Hissi Veren İnteraktif Landing Page

Tek HTML dosyasıyla, framework kullanmadan yazılmış interaktif bir landing page. Amaç, "award-winning" sitelerdeki o farklı hissin aslında birkaç temel tekniğin birleşimi olduğunu göstermek: scroll'un bir deneyime dönüşmesi, tepki veren bir 3D sahne ve daha içerik yüklenmeden tonu belirleyen bir loader.

**Canlı akış:** Loader → Mürekkep Hero → Renkli Scroll Bölümü → 3D Intro → Scroll Zoom Finali

## Bölümler ve Teknikler

### 1. Loader (0 → 100 Sayaç)
Siyah ekran, logo ve köşede büyüyen sayaç. İçerik gelmeden önce tonu kuran klasik "awwwards" açılışı. Sayaç easing ile yavaşlayarak 100'e ulaşır, sonra sayfa yukarı doğru sıyrılır.

- Saf CSS transition + `cubic-bezier` ile wipe efekti
- Sayaç, kalan mesafenin yüzdesi kadar artar, bu yüzden sona doğru doğal şekilde yavaşlar

### 2. Mürekkep Hero
Turuncu zemin üzerinde canlı gibi hareket eden siyah mürekkep lekesi ve mouse ile eğilen 3D kart.

- Leke: SVG elipsler + `feGaussianBlur` + `feColorMatrix` (gooey filter). Elipsler birbirine yaklaşınca sıvı gibi birleşir
- Kart: `perspective` + `rotateX/rotateY`, mouse pozisyonu lerp ile yumuşatılır

### 3. Renkli Scroll Bölümü
Sticky sahne içinde, scroll ilerledikçe illüstrasyonlar kenarlara savrulur ve kutu dönerek aşağı yuvarlanır. Devamında başlık ve tek tek beliren özellik etiketleri gelir.

- `position: sticky` + section yüksekliği 160vh, scroll progress 0..1 normalize edilir
- Her dekor elemanının `data-sx` / `data-sy` hız katsayısı vardır, progress ile çarpılıp `translate` edilir
- Etiketler `IntersectionObserver` ile kademeli açılır

### 4. 3D Intro (Three.js)
Koyu zeminde süzülen mavi, beyaz ve siyah plastik şekiller. Mouse parallax ve scroll ile kameranın içeri dalışı.

- Şekiller: 3 adet `CapsuleGeometry` birleşimi, `MeshStandardMaterial`
- İki directional light (sıcak key + mavi rim) ve sis ile derinlik
- Kamera pozisyonu scroll progress'e bağlı, mouse ile lerp'li parallax

### 5. Scroll Zoom Finali
Apple tarzı "3D zoom" hissi. Orijinal siteler bunu scroll'a bağlı image sequence ile yapar. Burada aynı his gerçek zamanlı Three.js sahnesiyle üretildi: kar zemini, iglo ve scroll'a bağlı kamera dolly.

- Zemin: `PlaneGeometry` vertex'lerine sinüs bazlı yükseklik, iglo çevresi düz bırakılır
- Kamera: easeInOut ile 24 birimden 4.5 birime dolly + hafif yörünge kayması
- `position: sticky` + 320vh section, yani zoom tamamen scroll kontrolünde

## Mobilde Takılmaması İçin

3D klonların çoğu mobilde şu yüzden kasar ve bu repo'da hepsi uygulanmış durumda:

- `setPixelRatio(Math.min(devicePixelRatio, 2))` ile retina üstü çözünürlüğe render etmeyi kes
- Ekranda olmayan sahneyi render etme (her iki canvas da görünürlük kontrolüyle çiziliyor)
- Geometri sayısını düşük tut, gölge haritası yerine sis + rim light ile derinlik ver
- Scroll değerlerini her frame'de değil, tek `requestAnimationFrame` döngüsünde oku

## Çalıştırma

```bash
git clone https://github.com/Eneslexi/award-reel.git
cd award-reel
python3 -m http.server 5510
# http://localhost:5510
```

Herhangi bir statik sunucu yeterli. Build adımı, bundler ve bağımlılık yok. Three.js CDN üzerinden import map ile geliyor.

## Yapı

```
award-reel/
└── index.html   # her şey burada: markup, CSS, JS, iki Three.js sahnesi
```

Tek dosya olması bilinçli bir tercih. Amaç, her efektin kaynağını tek yerde okuyup öğrenebilmek.

## Kimin İçin

Three.js'e hiç dokunmamış ama "o siteler nasıl yapılıyor" diyen herkes için. Her bölüm bağımsız okunabilir, kendi projene tek tek taşıyabilirsin.

Instagram: [@enesa.co](https://instagram.com/enesa.co)

## Lisans

MIT
