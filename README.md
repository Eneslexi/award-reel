# Award Reel · Ödüllü Site Hissi Veren İnteraktif Landing Page

**Canlı Demo:** [eneslexi.github.io/award-reel](https://eneslexi.github.io/award-reel/)

"Award-winning" sitelerdeki o farklı his şans değil, büyük bütçe de değil. Scroll'un bir deneyime dönüşmesi, tepki veren bir 3D sahne ve daha içerik yüklenmeden tonu belirleyen bir loader. Bu repo, o hissi veren 5 tekniği tek bir HTML dosyasında, framework kullanmadan gösteriyor.

Aşağıda hem ne öğrenmen gerektiği, hem adım adım yapım sırası, hem de her efekti kendi projende üretmek için kullanabileceğin hazır promptlar var.

---

## Bunu Yapabilmek İçin Ne Öğrenmelisin

Sırayla. Her madde bir sonrakinin temeli:

1. **HTML + CSS temelleri:** flexbox, `position: absolute/sticky`, `transform`, `transition`. Bu projedeki animasyonların yarısı sadece CSS transform.
2. **JavaScript temelleri:** DOM seçme, event dinleme (`mousemove`, scroll), `requestAnimationFrame`. Animasyon döngüsünün kalbi budur.
3. **Scroll matematiği:** bir bölümün ekrana göre konumundan 0..1 arası bir "progress" değeri çıkarmak. Tek formül: `progress = -rect.top / (sectionHeight - viewportHeight)`. Bu sayıyı öğrendiğin an scroll animasyonlarının tamamı elinde.
4. **Lerp (linear interpolation):** `deger += (hedef - deger) * 0.06`. Akıcılığın tüm sırrı bu tek satır. Her frame hedefe biraz yaklaşırsın, animasyon yumuşacık olur.
5. **Three.js temelleri:** Scene, Camera, Renderer, Mesh, Light. İlk hafta sadece küp döndür, yeter.
6. **Işık ve materyal:** MeshStandardMaterial vs MeshPhysicalMaterial, environment map, tone mapping. "Ucuz 3D" ile "pahalı görünen 3D" arasındaki fark modelde değil, ışıkta.

---

## Adım Adım Yapım

### Adım 1: Loader (sayaç 0 → 100)

Siyah tam ekran bir katman, ortada logo, köşede sayaç. Sayaç her adımda kalan mesafenin yüzde 6'sı kadar ilerler, bu yüzden sona doğru kendiliğinden yavaşlar. 100 olunca katman `translateY(-100%)` ile yukarı sıyrılır.

```js
let n = 0;
const tick = () => {
  n += Math.max(1, Math.round((100 - n) * 0.06));
  counter.textContent = Math.min(n, 100);
  if (n < 100) setTimeout(tick, 40 + Math.random() * 70);
  else loader.classList.add('done'); // CSS: transform: translateY(-100%)
};
```

**Prompt:** "Bana siyah tam ekran bir preloader yap. Ortada logo, sol altta 0'dan 100'e sayan büyük bir sayaç olsun. Sayaç easing ile yavaşlayarak dolsun, bitince sayfa yukarı doğru kayarak açılsın. Saf HTML/CSS/JS."

### Adım 2: Canlı mürekkep lekesi (gooey efekt)

SVG içinde birkaç elips çiz, hepsine `feGaussianBlur` + `feColorMatrix` filtresi uygula. Blur, şekillerin kenarını eritir; color matrix alfa kanalını sertleştirir. Sonuç: şekiller birbirine yaklaşınca sıvı gibi birleşir. JS ile elipsleri sinüsle hafifçe gezdir, leke canlanır.

```xml
<filter id="gooey">
  <feGaussianBlur in="SourceGraphic" stdDeviation="12" result="b"/>
  <feColorMatrix in="b" values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 24 -10"/>
</filter>
```

**Prompt:** "SVG gooey filter kullanarak organik, canlı gibi hareket eden siyah bir mürekkep lekesi yap. Birkaç elips feGaussianBlur ve feColorMatrix ile birleşsin, JS ile sinüs fonksiyonuyla yavaşça hareket etsin."

### Adım 3: Mouse ile eğilen 3D kart

Karta `transform-style: preserve-3d` ver, mouse pozisyonunu -1..1 aralığına normalize et, `rotateX/rotateY`'ye çevir. Ham mouse değerini direkt kullanma; lerp'ten geçir, yoksa kart titrer.

```js
window.addEventListener('mousemove', e => {
  mx = (e.clientX / innerWidth - 0.5) * 2;   // -1..1
  my = (e.clientY / innerHeight - 0.5) * 2;
});
function tilt() {
  cmx += (mx - cmx) * 0.06;  // lerp: yumuşatma
  cmy += (my - cmy) * 0.06;
  card.style.transform = `rotateY(${cmx * 22}deg) rotateX(${-cmy * 16}deg)`;
  requestAnimationFrame(tilt);
}
```

**Prompt:** "Ortada duran bir kart mouse hareketine göre 3D eğilsin. CSS perspective + rotateX/rotateY kullan, mouse değerlerini lerp ile yumuşat, kart ayrıca kendi kendine hafifçe yüzsün (float animasyonu)."

### Adım 4: Scroll ile dağılan sahne (sticky + progress)

Bölümün yüksekliğini viewport'tan büyük yap (örn. 280vh), içine `position: sticky; top: 0; height: 100vh` bir sahne koy. Kullanıcı scroll ettikçe sahne yerinde kalır, sen de progress değerini elemanlara dağıtırsın. Her dekor elemanına bir hız katsayısı ver (`data-sx`, `data-sy`), progress ile çarpıp translate et. Aynı progress'le ortadaki objeyi döndürerek aşağı yuvarla.

```js
const p = Math.min(1, Math.max(0, -rect.top / (rect.height - innerHeight)));
colP += (p - colP) * 0.055; // lerp: animasyon scroll'un peşinden süzülür
deco.style.transform = `translate(${sx * colP * 60}vw, ${sy * colP * 50}vh)`;
```

**Prompt:** "Sticky bir hero bölümü yap. Scroll ettikçe etraftaki illüstrasyon öğeleri farklı hızlarda kenarlara savrulsun, ortadaki ürün dönerek aşağı yuvarlansın. Framework yok, sadece scroll progress + lerp + transform."

### Adım 5: 3D intro sahnesi (Three.js)

Koyu zeminde yüzen parlak plastik şekiller. Pahalı görünmesinin üç şartı:

1. `RoomEnvironment` ile environment map (yansımalar buradan gelir)
2. `MeshPhysicalMaterial` + `clearcoat: 1` (vernikli plastik hissi)
3. `ACESFilmicToneMapping` (sinema renk eğrisi)

```js
renderer.toneMapping = THREE.ACESFilmicToneMapping;
scene.environment = new THREE.PMREMGenerator(renderer)
  .fromScene(new RoomEnvironment(), 0.04).texture;
const mat = new THREE.MeshPhysicalMaterial({
  color: 0x1f2ae8, roughness: 0.32, clearcoat: 1, clearcoatRoughness: 0.12
});
```

Şekilleri `Math.sin(time + faz)` ile yüzdür, mouse'u kamera pozisyonuna lerp'le bağla, scroll progress ile kamerayı içeri it.

**Prompt:** "Three.js ile koyu lacivert zeminde yüzen 30 adet parlak plastik artı şekli yap. RoomEnvironment ile environment map, MeshPhysicalMaterial clearcoat, ACES tone mapping kullan. Şekiller sinüsle yüzsün, kamera mouse ile parallax yapsın, scroll ile sahneye dalsın."

### Adım 6: Scroll zoom finali

Apple'ın "3D zoom" efekti gerçek zamanlı 3D değildir; scroll'a bağlı oynatılan bir görüntü dizisidir (image sequence). Aynı hissi gerçek Three.js sahnesiyle de verebilirsin: 500vh'lik sticky bölüm, scroll progress'i easeInOut'tan geçir, kamerayı uzaktan hedefe dolly yap.

```js
const e = p < 0.5 ? 2*p*p : 1 - Math.pow(-2*p + 2, 2) / 2; // easeInOut
camera.position.set(sweep, 12 - e * 10.4, 26 - e * 21.8);
camera.lookAt(0, 2.2 - e * 1.4, 2.6);
```

Sahneyi zenginleştiren detaylar: gerçek gölgeler (`PCFSoftShadowMap`), vertex color ile vadileri koyulaştırılmış zemin, havada süzülen partiküller ve kamerada sürekli çalışan minik "nefes alma" hareketi.

**Prompt:** "Three.js ile gün batımında karlı bir vadi sahnesi yap. Tuğlalardan örülmüş bir iglo, engebeli kar zemini, yumuşak gölgeler ve kar taneleri olsun. 500vh'lik sticky bir bölümde scroll progress'e bağlı olarak kamera yüksekten süzülüp iglonun kapısına zoom yapsın. EaseInOut ve lerp kullan."

### Adım 7: Hepsini tek render döngüsüne bağla

Her sahneye ayrı döngü yazma. Tek `requestAnimationFrame` içinde: hangi bölüm ekrandaysa sadece onu render et. Ekranda olmayan 3D sahne render edilmezse mobil rahatlar.

```js
function loop() {
  if (bolumEkranda(jacksSection)) renderJacks();
  if (bolumEkranda(zoomSection)) renderZoom();
  requestAnimationFrame(loop);
}
```

### Adım 8: Mobilde takılmaması için

3D klonların mobilde kasmasının 4 sebebi ve çözümü:

- `setPixelRatio(Math.min(devicePixelRatio, 2))` yaz. Retina üstü çözünürlüğe render etmek en büyük FPS katilidir
- Ekranda olmayan sahneyi render etme (Adım 7)
- Geometri sayısını düşük tut; derinliği gölge yerine sis + rim light ile ver, gölge gerekiyorsa tek ışıkta tut
- Scroll event'inde iş yapma; scroll değerini oku, işi `requestAnimationFrame` döngüsünde yap

---

## Çalıştırma

```bash
git clone https://github.com/Eneslexi/award-reel.git
cd award-reel
python3 -m http.server 5510
# http://localhost:5510
```

Build yok, bundler yok, bağımlılık yok. Three.js import map ile CDN'den geliyor. `index.html` içindeki her bölüm bu README'deki adımlarla aynı sırada; kodu yukarıdan aşağı okuyarak ilerleyebilirsin.

## Yapı

```
award-reel/
└── index.html   # markup, CSS, JS ve iki Three.js sahnesi, hepsi tek dosyada
```

Tek dosya bilinçli tercih: her efektin kaynağını tek yerde okuyup kendi projene tek tek taşıyabilesin diye.

## Kimin İçin

Three.js'e hiç dokunmamış ama "o siteler nasıl yapılıyor" diyen herkes için. Yukarıdaki promptları istediğin AI aracına verip çıkan kodu bu repodakiyle karşılaştırarak da öğrenebilirsin; en hızlı öğrenme yolu bu.

Instagram: [@enesa.co](https://instagram.com/enesa.co)

## Lisans

MIT
