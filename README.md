# Hammer Girl - 2D Fighter

Canvas API ile yapılmış vanilla JavaScript 2D dövüş oyunu. Tek dosyalı, framework kullanmaz.

## Oyna

[https://zaferayan.itch.io/hammer-girl](https://zaferayan.itch.io/hammer-girl)

Veya `index.html` dosyasını tarayıcıda aç.

## Kontroller

### Klavye

| Tuş               | Aksiyon         |
| ----------------- | --------------- |
| WASD / Arrow Keys | Hareket         |
| E                 | Çekiç saldırısı |
| Q                 | Çekiç fırlatma  |
| Space             | Zıplama         |

### Dokunmatik (Mobil)

- **Sol yarı:** Sanal joystick (hareket)
- **Sağ alt:** Aksiyon butonları (saldırı, fırlatma, zıplama)
- Sadece landscape modda çalışır

## Özellikler

- Sprite tabanlı karakter animasyonları (idle, walk, attack, throw, jump)
- Nefes alma efekti ve perspektif skalası (sahte derinlik)
- Slime düşman AI (idle → chase → attack state machine)
- Melee saldırı + çekiç fırlatma (projectile)
- HP sistemi ve invincibility blink efekti
- Dokunmatik joystick + buton desteği
- Tam ekran geçişi
- Loading screen ve start screen

## Proje Yapısı

```
index.html              # Tüm oyun kodu
img/
  bg.png                # Arka plan
  standing.png          # Idle pozu
  walk1.png, walk2.png  # Yürüme kareleri
  attack/               # Saldırı kareleri (3 kare)
  throw/                # Fırlatma kareleri (2 kare)
  jump/                 # Zıplama kareleri (4 kare)
  items/                # Çekiç projectile görseli
  slime/                # Düşman görselleri
  ui/                   # Sağlık barı görselleri
```

## Teknik Detaylar

- **Canvas:** 960x540, CSS ile tam ekran stretch
- **Render:** `requestAnimationFrame` (~60fps)
- **Çarpışma:** Mesafe tabanlı (dairesel)
- **Perspektif:** Y pozisyonuna göre scale (1.0 → 0.8)

Detaylı mimari için [architecture.md](architecture.md) dosyasına bakın.
