# Hammer Girl - Mimari Dokümantasyonu

## Genel Bakış

Tek dosyalı (`index.html`) vanilla JavaScript 2D dövüş oyunu. Canvas API kullanılarak render edilir. Framework veya build tool kullanılmaz.

## Dosya Yapısı

```
index.html          # Tüm oyun kodu (HTML + CSS + JS)
img/
  bg.png            # Arka plan görseli
  standing.png      # Idle pozu
  walk1.png, walk2.png
  attack/           # 3 kare: lift, smash, follow-through
  throw/            # 2 kare: lift, throw
  jump/             # 4 kare: crouch, takeoff, mid-air, landing
  items/hammer-upward.png   # Fırlatılan çekiç görseli
  slime/            # Düşman görselleri (idle, move, attack, hurt)
  ui/               # Sağlık barı görselleri (player, enemy)
```

## Mimari Katmanlar

### 1. Konfigürasyon (Satır 73-130)

Tüm oyun sabitleri üst seviyede `const` olarak tanımlı:

| Kategori   | Sabitler                                                                  |
| ---------- | ------------------------------------------------------------------------- |
| Canvas     | `CANVAS_WIDTH` (960), `CANVAS_HEIGHT` (540)                               |
| Karakter   | `CHAR_RENDER_WIDTH/HEIGHT` (180), `MOVE_SPEED` (4)                        |
| Perspektif | `MIN_SCALE` (0.8) - Y ekseninde derinlik hissi                            |
| Animasyon  | `ANIM_FRAME_DURATION` (10), `BREATH_SPEED/AMOUNT`                         |
| Saldırı    | `ATTACK_FRAME_DURATION` (8)                                               |
| Fırlatma   | `THROW_FRAME_DURATION` (8), `HAMMER_SPEED` (8), `HAMMER_SIZE` (200)       |
| Zıplama    | `JUMP_VELOCITY` (-18), `GRAVITY` (0.6)                                    |
| HP         | `PLAYER_MAX_HP` (5), `SLIME_MAX_HP` (5)                                   |
| Düşman     | `SLIME_SPEED` (1.5), `SLIME_CHASE_RANGE` (400), `SLIME_ATTACK_RANGE` (80) |

### 2. Asset Loader (Satır 140-218)

- `loadImage(src)` fonksiyonu ile tüm görseller yüklenir
- `assetsLoaded / assetsTotal` ile yükleme ilerlemesi takip edilir
- `assetsReady` flag'i true olunca oyun başlamaya hazır
- Görseller kategorilere göre yüklenir: bg, standing, walk, attack, throw, jump, slime, UI

### 3. Input Sistemi (Satır 220-393)

**Klavye (Satır 220-269):**

- `keys` objesi ile keydown/keyup takibi
- WASD + Arrow keys: hareket
- E: saldırı, Q: fırlatma, Space: zıplama
- Scroll engelleme (arrow + space)

**Dokunmatik (Satır 271-393):**

- Sol yarı: sanal joystick (multi-touch destekli)
- Sağ alt: aksiyon butonları (E, Q, yukarı ok)
- `touchToCanvas()`: ekran koordinatlarını canvas koordinatlarına çevirir
- Dead zone (15px) ve radius sınırı (60px) ile joystick kontrolü
- `touch.joystickId` ile hangi parmağın joystick olduğu takip edilir

### 4. Oyun Durumu (State) (Satır 395-434)

**`character` objesi:**

- Pozisyon: `x`, `y`
- Hareket: `isMoving`, `facingRight`, `currentFrame`, `animTimer`
- Saldırı: `isAttacking`, `attackFrame`, `attackTimer`
- Fırlatma: `isThrowing`, `throwFrame`, `throwTimer`
- Zıplama: `isJumping`, `jumpPhase` (crouch/air/land), `jumpVelY`, `jumpOffsetY`
- HP: `hp`, `invincible`, `invincibleTimer`
- Efekt: `breathTimer`

**`projectiles` dizisi:** Aktif fırlatılmış çekiçler

**`slime` objesi:** Düşman durumu (pozisyon, state, HP, yön)

### 5. Update Döngüsü (Satır 436-708)

Çalışma sırası:

```
1. Klavye + joystick hareketi uygula
2. Canvas sınırlarına clamp
3. Zıplama fiziği (crouch → air → land)
4. Fırlatma/saldırı animasyon ilerlemesi
5. Yürüme animasyonu veya idle
6. Nefes timer güncelle
7. Invincibility countdown
8. Slime AI (state machine)
9. Çarpışma tespiti (melee + projectile)
10. Projectile güncelleme + temizleme
```

**Slime AI State Machine:**

```
idle → (yakınsa) → chase → (çok yakınsa) → attack
  ↑                                           |
  └──────────── hurt ←────────────────────────┘
```

### 6. Render Sistemi (Satır 710-942)

Render sırası (arkadan öne):

```
1. Arka plan (bg.png veya gradient fallback)
2. Karakter (perspektif + nefes skalası uygulanmış)
3. Slime düşman
4. UI: Sağlık barları (sol üst: oyuncu, sağ üst: düşman)
5. Projectile'lar (dönen çekiçler)
6. Dokunmatik kontroller (sadece touch cihazlarda)
```

**Perspektif sistemi:** Y pozisyonuna göre scale hesaplanır (altta 1.0, üstte 0.8) - sahte derinlik hissi.

**Flip mekanizması:** `ctx.scale(-1, 1)` ile karakter ve slime yön değiştirirken görseller yatay çevrilir.

**Invincibility blink:** `BLINK_SPEED` aralığında karakter görünür/görünmez yapılır.

### 7. Oyun Akışı (Satır 944-1048)

```
Loading Screen → Start Screen → Game Loop
     ↓               ↓              ↓
  renderLoading()  renderStartScreen()  update() + render()
```

- **Loading:** Progress bar + yüzde gösterimi
- **Start:** "Tap or Click to Start" (yanıp sönen)
- **Game:** `requestAnimationFrame` ile ~60fps döngü
- Tam ekran geçişi oyun başlarken tetiklenir

## Responsive Tasarım

- Canvas: `100vw x 100vh` (CSS ile tam ekran stretch)
- `image-rendering: crisp-edges` ile piksel art korunur
- Portrait modda (mobil): `#rotate-warning` gösterilir, canvas gizlenir
- Breakpoint: `orientation: portrait` ve `max-width: 768px`

## Çarpışma Sistemi

- Mesafe tabanlı (AABB değil, dairesel)
- `Math.sqrt(dx*dx + dy*dy)` ile uzaklık hesabı
- Saldırı hit: karakter yönüne göre offset + SLIME_WIDTH yarıçapı
- Projectile hit: `SLIME_WIDTH * 0.7` yarıçapı
- Knockback: vuruş yönünde 30-40px itme
