# 🧩 Sort It Out — Product Requirements Document

> **Version:** 1.0  
> **Status:** MVP Live  
> **Live URL:** https://berthojoris.github.io/survai/games/dragdrop-puzzle/  
> **Last Updated:** 2026-07-15

---

## 1. Product Overview

### 1.1 Elevator Pitch
**Sort It Out** adalah game puzzle edukatif berbasis drag & drop. Pemain memilih 3 jawaban benar dari 6 pilihan yang tersedia, lalu menyeretnya ke salah satu kolom target. Game memberi umpan balik instan — animasi sukses atau gagal — sehingga cocok untuk kuis, assessment, atau ice-breaking.

### 1.2 Target Audience
| Segmen | Use Case |
|---|---|
| Guru / Dosen | Kuis interaktif di kelas |
| Trainer / HR | Ice-breaking saat onboarding |
| Content Creator | Konten interaktif di sosmed/story |
| Umum | Game ringan yang bisa dimainkan offline |

### 1.3 Platform
- **Web-first** — static HTML/CSS/JS, no build step
- **PWA-ready** — bisa ditambahkan service worker untuk offline play
- **Responsive** — desktop + mobile (touch drag support)
- **Hosting** — GitHub Pages (free, CDN global)

### 1.4 Key Metrics (Success Criteria)
| Metric | Target |
|---|---|
| Time to first interaction | < 2 detik (static file, no JS bundle) |
| Mobile usability | Touch drag berfungsi di iOS Safari & Android Chrome |
| Completion rate | Pemain bisa menyelesaikan 1 ronde dalam < 60 detik |
| Accessibility | Bisa dimainkan dengan keyboard + screen reader (future) |

---

## 2. Game Mechanics

### 2.1 Core Loop

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────┐
│  Lihat 6    │────▶│  Drag 3 jawaban   │────▶│  Klik tombol │
│  pilihan    │     │  ke 1 kolom target│     │  "Periksa"   │
└─────────────┘     └──────────────────┘     └──────┬───────┘
                                                    │
                                          ┌─────────▼─────────┐
                                          │  Cek kondisi:      │
                                          │  • 3 item di 1     │
                                          │    kolom yang sama?│
                                          │  • Jawaban benar?  │
                                          └────┬─────────┬─────┘
                                               │         │
                                          BENAR│         │SALAH
                                               ▼         ▼
                                         🎉 Sukses   😢 Gagal
                                         + Confetti   + Shake
```

### 2.2 Rules

| # | Aturan | Deskripsi |
|---|---|---|
| R1 | **6 pilihan tersedia** | Panel kiri menampilkan 6 kartu berisi teks pernyataan |
| R2 | **Pilih tepat 3** | Pemain wajib men-drag **tepat 3** kartu ke panel kanan (tidak kurang, tidak lebih) |
| R3 | **Satu kolom saja** | Ke-3 kartu harus berada di **kolom target yang sama** (Kolom A atau Kolom B). Tidak boleh terbagi |
| R4 | **Order-insensitive** | Urutan kartu di dalam kolom tidak memengaruhi penilaian |
| R5 | **Click to undo** | Kartu yang sudah di-drop bisa diklik untuk dikembalikan ke panel kiri |
| R6 | **Tombol baru aktif saat 3 terisi** | Tombol "Periksa Jawaban" hanya enable setelah 3 kartu ditempatkan |

### 2.3 Answer Configuration

Jawaban benar dikonfigurasi oleh admin melalui konstanta JavaScript:

```javascript
// Admin-defined correct answer (indeks 0-based dari array CHOICES)
const CORRECT_ANSWERS = [1, 3, 5];

const CHOICES = [
  { text: "Fotosintesis terjadi di mitokondria",       color: "card-1" },  // 0 — SALAH
  { text: "Air mendidih pada 100°C di tekanan 1 atm",   color: "card-2" },  // 1 — BENAR ✅
  { text: "HTML adalah bahasa pemrograman",             color: "card-3" },  // 2 — SALAH
  { text: "Bumi mengelilingi Matahari",                 color: "card-4" },  // 3 — BENAR ✅
  { text: "Mars memiliki 5 bulan alami",                color: "card-5" },  // 4 — SALAH
  { text: "Atom terdiri dari proton, neutron, elektron", color: "card-6" },  // 5 — BENAR ✅
];
```

**Aturan konfigurasi:**
- `CORRECT_ANSWERS` wajib berisi **tepat 3 indeks**
- Indeks harus unik dan berada dalam rentang `0..CHOICES.length-1`
- `CHOICES` wajib berisi **minimal 6 item**
- Properti `color` menentukan warna aksen kartu (opsional, fallback ke warna default)

### 2.4 Scoring (Future)
| Kondisi | Skor |
|---|---|
| 3/3 benar di satu kolom | +100 pts |
| 3/3 benar tapi split (dicegah sistem) | N/A (warning) |
| 2/3 benar | +0 (gagal) |
| 1/3 benar | +0 (gagal) |
| 0/3 benar | +0 (gagal) |

---

## 3. User Flow

### 3.1 First-Time Player Journey

```
[Landing Page]
  │
  ├─ Lihat judul "🧩 Sort It Out" + subtitle instruksi
  ├─ Panel kiri: 6 kartu pilihan tersusun vertikal
  ├─ Panel kanan: 2 kolom target kosong (Kolom A & B)
  │
  ▼
[Drag & Drop]
  │
  ├─ Desktop: klik + tahan kartu → geser ke kolom target → lepas
  ├─ Mobile:  sentuh + tahan kartu → geser ke kolom target → lepas
  ├─ Kartu sumber berubah jadi redup (opacity 40%, border dashed)
  ├─ Kolom target menampilkan kartu yang sudah di-drop + counter "x/3"
  │
  ▼
[3 Kartu Terkumpul]
  │
  ├─ Tombol "✅ Periksa Jawaban" menyala (animasi pop-in)
  ├─ Jika kartu tersebar di 2 kolom → popup ⚠️ "Satu Kolom Aja!"
  │
  ▼
[Submit]
  │
  ├─── Semua benar ──▶ Overlay 🎉 "Jawaban Benar!" + confetti 100 pieces
  │                    └─ Tombol "🔄 Main Lagi"
  │
  └─── Ada salah ────▶ Shake animasi pada game container
                       └─ Overlay 😢 "Belum Tepat" + tombol "🔙 Coba Lagi"
```

### 3.2 Edge Cases

| Edge Case | Behavior |
|---|---|
| Drag kartu yang sudah di-drop | Tidak bisa — kartu sumber disabled (pointer-events: none) |
| Drag kartu ke area di luar kolom | Kartu kembali ke posisi semula (dragend tanpa drop) |
| Klik kartu di kolom target | Kartu kembali ke panel kiri, counter berkurang |
| Submit tanpa 3 kartu | Tombol disabled, tidak bisa diklik |
| Drag lebih dari 3 kartu | Ditolak — `getPlacedCount() >= 3` mencegah drop ke-4 |
| Double-tap di mobile | Tidak ada efek khusus (tidak ada zoom, tidak ada select) |
| Rotate device di tengah game | Layout auto-adjust via CSS media query (700px breakpoint) |
| Slow network / offline | File static HTML — tetap bisa dibuka jika sudah di-cache browser |

---

## 4. UI/UX Specification

### 4.1 Layout (Desktop ≥ 700px)

```
┌──────────────────────────────────────────────────────────┐
│                     🧩 Sort It Out                        │
│          Drag 3 pernyataan yang BENAR ke kotak di kanan    │
│                                                           │
│            Ditempatkan:  0 / 3                            │
│                                                           │
│  ┌─────────────────────┐  ┌─────────────────────────────┐ │
│  │  📋 Pilihan          │  │  🎯 Target                   │ │
│  │                      │  │                              │ │
│  │  ┌─────────────────┐ │  │  ┌────────────────────────┐ │ │
│  │  │ 1  Fotosintesis..│ │  │  │ Kolom A — drop 3 ...   │ │ │
│  │  └─────────────────┘ │  │  │                        │ │ │
│  │  ┌─────────────────┐ │  │  │  (kosong)              │ │ │
│  │  │ 2  Air mendidih..│ │  │  │                        │ │ │
│  │  └─────────────────┘ │  │  └────────────────────────┘ │ │
│  │  ┌─────────────────┐ │  │                              │ │
│  │  │ 3  HTML adalah.. │ │  │  ┌────────────────────────┐ │ │
│  │  └─────────────────┘ │  │  │ Kolom B — drop 3 ...   │ │ │
│  │  ┌─────────────────┐ │  │  │                        │ │ │
│  │  │ 4  Bumi mengeli..│ │  │  │  (kosong)              │ │ │
│  │  └─────────────────┘ │  │  │                        │ │ │
│  │  ┌─────────────────┐ │  │  └────────────────────────┘ │ │
│  │  │ 5  Mars memiliki │ │  │                              │ │
│  │  └─────────────────┘ │  └─────────────────────────────┘ │
│  │  ┌─────────────────┐ │                                   │
│  │  │ 6  Atom terdiri..│ │   ┌──────────┐ ┌──────────────┐ │
│  │  └─────────────────┘ │   │ 🔄 Reset  │ │✅ Periksa    │ │
│  └─────────────────────┘   └──────────┘ └──────────────┘ │
└──────────────────────────────────────────────────────────┘
```

### 4.2 Layout (Mobile < 700px)

```
┌─────────────────────┐
│   🧩 Sort It Out    │
│   Drag 3 yg BENAR   │
│                     │
│  Ditempatkan: 0 / 3 │
│                     │
│  📋 Pilihan         │
│ ┌───────┐ ┌───────┐ │
│ │1 Foto..│ │2 Air..│ │
│ └───────┘ └───────┘ │
│ ┌───────┐ ┌───────┐ │
│ │3 HTML..│ │4 Bumi.│ │
│ └───────┘ └───────┘ │
│ ┌───────┐ ┌───────┐ │
│ │5 Mars..│ │6 Atom.│ │
│ └───────┘ └───────┘ │
│                     │
│  🎯 Target          │
│ ┌─────────────────┐ │
│ │ Kolom A (0/3)   │ │
│ └─────────────────┘ │
│ ┌─────────────────┐ │
│ │ Kolom B (0/3)   │ │
│ └─────────────────┘ │
│                     │
│ ┌───────┐┌────────┐ │
│ │🔄Reset││✅Periksa│ │
│ └───────┘└────────┘ │
└─────────────────────┘
  ▲ scrollable ▼
```

### 4.3 Design Tokens

| Token | Value | Usage |
|---|---|---|
| Background | `#0a0a1a` | Body background |
| Surface | `#14142b` | Card, drop zone background |
| Surface 2 | `#1e1e3a` | Button secondary |
| Accent | `#6c5ce7` | Primary CTA, borders, glow |
| Accent Glow | `#a29bfe` | Text gradient, highlights |
| Success | `#00cec9` | Success state |
| Success Glow | `#55efc4` | Success text, confetti |
| Danger | `#ff6b6b` | Error state, failure overlay |
| Text Primary | `#e0e0f0` | Body text |
| Text Dim | `#8888aa` | Labels, placeholders |
| Card Colors | 6 distinct colors | `#6c5ce7`, `#e17055`, `#00b894`, `#fdcb6e`, `#0984e3`, `#fd79a8` |
| Border Radius | `12px–24px` | Cards, buttons, drop zones |
| Font | Segoe UI / system-ui | Cross-platform consistent |

### 4.4 Animation Specs

| Animation | Trigger | Duration | Easing |
|---|---|---|---|
| Card hover tilt | Hover (desktop only) | 0.3s | `cubic-bezier(0.4, 0, 0.2, 1)` |
| Card drag state | Drag start | Instant | — |
| Drop zone highlight | Drag over zone | 0.4s | `cubic-bezier(0.4, 0, 0.2, 1)` |
| Pop-in (placed item) | Item dropped in zone | 0.35s | `cubic-bezier(0.175, 0.885, 0.32, 1.275)` |
| Shake (error) | Wrong answer / split warning | 0.5s | Keyframe `shake` |
| Flip-up (overlay card) | Overlay appears | 0.5s | `cubic-bezier(0.4, 0, 0.2, 1)` |
| Bounce-in (icon) | Overlay icon | 0.6s (delay 0.2s) | Keyframe `bounceIn` |
| Confetti fall | Success | 1.2–3.2s (random) | Keyframe `confettiFall` |
| Fade-in (overlay bg) | Overlay appears | 0.3s | `ease` |
| Submit button pop | 3 items placed | 0.4s | `cubic-bezier(0.175, 0.885, 0.32, 1.275)` |

### 4.5 3D Effects
| Effect | Implementation |
|---|---|
| Body perspective | `perspective: 1200px` on body |
| Card 3D tilt | `transform: rotateX(2deg) rotateY(-2deg)` on cards |
| Drop zone depth | `transform: rotateX(1deg)` + `box-shadow: inset` |
| Overlay flip | `transform: rotateX(-60deg) → rotateX(0deg)` |
| Ambient particles | 20 floating orbs, random position/delay/duration |

---

## 5. Technical Architecture

### 5.1 Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| Markup | HTML5 | Semantic, accessible |
| Styling | CSS3 (custom properties) | Dark theme, animations, responsive |
| Logic | Vanilla JavaScript (ES6+) | Zero dependencies, instant load |
| Drag & Drop | HTML5 Drag and Drop API + Touch Events polyfill | Native browser support, no library |
| Hosting | GitHub Pages | Free, CDN, auto-deploy on push |
| PWA | manifest.json (future) | Offline capability, add to home screen |

### 5.2 File Structure

```
games/dragdrop-puzzle/
├── index.html          # Single-file game (HTML + CSS + JS inline)
├── manifest.json       # PWA manifest (planned)
├── sw.js               # Service worker for offline (planned)
├── PRD.md              # This document
└── README.md           # Quick start guide (planned)
```

### 5.3 JavaScript Architecture

```
Global State:
  placedItems: { [zoneId: string]: number[] }
    // e.g., { "1": [1, 3, 5] } — all 3 items in zone 1

Core Functions:
  renderChoices()          — Render 6 choice cards di panel kiri
  renderDropZones()        — Render placed items + counter label di panel kanan
  dropItem(zone, index)    — Handle drop: update state → re-render → update UI
  removeFromZone(key, idx) — Undo: remove item from zone → return to choices
  checkAnswer()            — Validasi: 1 kolom? jawaban benar?
  updateUI()               — Update counter + submit button state

Interaction Handlers:
  Desktop:  dragstart, dragend, dragover, dragleave, drop
  Mobile:   touchstart, touchmove, touchend (dengan ghost clone element)

Animation:
  showSuccess()            — Confetti + overlay "Jawaban Benar!"
  showFailure()            — Shake + overlay "Belum Tepat"
  showSplitWarning()       — Shake + overlay "Satu Kolom Aja!"
  spawnConfetti()          — 100 confetti pieces, random color/position/delay
```

### 5.4 Data Flow

```
User drags card #3
  │
  ▼
touchstart/dragstart: set draggedIndex = 3, add .dragging class
  │
  ▼
touchmove/dragover: highlight drop zone under cursor, move ghost clone
  │
  ▼
touchend/drop: detect which zone → dropItem(zone, 3)
  │
  ▼
dropItem():
  ├─ guard: isPlaced(3)? → reject
  ├─ guard: getPlacedCount() >= 3? → reject
  ├─ push index to placedItems[zoneKey]
  ├─ renderDropZones()   ← re-render target panel
  ├─ renderChoices()     ← re-render source cards (card #3 → .placed)
  └─ updateUI()          ← update counter + submit button
```

---

## 6. Interaction States

### 6.1 Card States

| State | Visual | Trigger |
|---|---|---|
| **Default** | Solid background, border accent color, 3D tilt | Initial render |
| **Hover** | Scale 1.02, translateY -2px, glow border, tilt reset | Mouse over (desktop) |
| **Dragging** | Opacity 0.5, scale 0.95, source ghost | Drag start |
| **Placed** | Opacity 0.4, border dashed, pointer-events none | After successful drop |
| **Active (touch)** | Scale 0.97, darker shadow | Touch hold (mobile) |

### 6.2 Drop Zone States

| State | Visual | Trigger |
|---|---|---|
| **Empty** | Dashed border, centered label text, `min-height: 160px` | Initial render / all items removed |
| **Drag-over** | Solid accent border, purple glow, scale 1.02 | Card hovering over zone |
| **Has items** | Label berubah ke counter "x/3 terisi", `.placed-item` chips | After first drop |
| **Full (3/3)** | Counter shows "3/3", submit button enabled | After third drop |

### 6.3 Overlay States

| State | Icon | Title | Message | Button |
|---|---|---|---|---|
| **Success** | 🎉 | "Jawaban Benar!" | "Kamu berhasil! Semua jawaban yang dipilih tepat." | "🔄 Main Lagi" |
| **Failure** | 😢 | "Belum Tepat" | "Beberapa jawaban masih salah. Coba lagi, bro!" | "🔙 Coba Lagi" |
| **Split Warning** | ⚠️ | "Satu Kolom Aja!" | "Ke-3 jawaban harus di **satu kolom yang sama**, ga boleh nyebar." | "🔙 OK, Siap" |

### 6.4 Button States

| Button | Default | Hover | Active | Disabled |
|---|---|---|---|---|
| **Reset** | Surface2 bg, dim border | Purple border, slight glow | Scale down | N/A (selalu aktif) |
| **Submit** | Purple gradient, glow shadow | TranslateY -2px, bigger glow | Scale down | Opacity 0.4, cursor not-allowed |

---

## 7. Responsive Breakpoints

| Breakpoint | Layout | Card Layout | Drop Zone Height | Card Transform |
|---|---|---|---|---|
| ≥ 700px | Row: left panel \| right panel | Vertical stack, 6 cards | 160px min | 3D tilt active |
| < 700px | Column: choices top, targets bottom | Horizontal wrap, 2 cols | 120px min | No 3D tilt (perf) |
| < 375px | Same as mobile, tighter padding | 2 cols, min-width 130px | 100px min | No 3D tilt |

### Mobile-Specific Behavior
- `overflow-y: auto` + `-webkit-overflow-scrolling: touch` untuk smooth scroll iOS
- `align-items: flex-start` agar konten tidak terpotong di tengah layar
- `padding-bottom: 60px` untuk safe area (iPhone notch / Android nav bar)
- Touch events dengan ghost clone element untuk visual feedback saat drag
- `user-select: none` + `-webkit-tap-highlight-color: transparent` mencegah seleksi teks

---

## 8. Accessibility (Current & Planned)

### 8.1 Current
| Feature | Status |
|---|---|
| Keyboard: `Ctrl+R` reset, `Escape` close overlay | ✅ |
| Semantic HTML (heading, button, generic) | ✅ |
| Color is not sole differentiator (card numbers + color) | ✅ |
| High contrast dark theme | ✅ |

### 8.2 Planned
| Feature | Priority |
|---|---|
| ARIA labels on draggable cards (`aria-grabbed`, `aria-dropeffect`) | Medium |
| Focus management (Tab through cards, Enter to select) | Medium |
| Screen reader announcements ("Card 2 placed in Column A") | Low |
| Reduced motion media query (`prefers-reduced-motion`) | Low |

---

## 9. Performance Budget

| Metric | Target | Current |
|---|---|---|
| Total file size | < 50 KB | ~25 KB |
| Time to Interactive | < 1 detik | < 200ms |
| First Contentful Paint | < 0.5 detik | ~100ms |
| Lighthouse Performance | > 95 | TBD |
| No external dependencies | ✅ | 0 dependencies |

---

## 10. Roadmap

### Phase 1 — MVP (✅ Done)
- [x] 6 choice cards + 2 drop zones
- [x] Drag & drop (desktop + mobile touch)
- [x] 3D effects (perspective, tilt, glow)
- [x] Answer validation (per-zone enforcement)
- [x] Success animation (confetti + overlay)
- [x] Failure animation (shake + overlay)
- [x] Split warning
- [x] Per-zone counter labels
- [x] Reset button
- [x] Responsive design
- [x] Dark theme
- [x] Deploy to GitHub Pages

### Phase 2 — Configuration (Next)
- [ ] Admin panel UI (edit soal tanpa edit kode)
- [ ] JSON config loader (`config.json` → dynamic CHOICES + ANSWERS)
- [ ] Multiple question sets (bank soal)
- [ ] Support variable jumlah jawaban benar (tidak harus 3)

### Phase 3 — Engagement
- [ ] Score system + leaderboard (localStorage)
- [ ] Timer countdown per round
- [ ] Multiple rounds / levels
- [ ] Sound effects (Web Audio API)
- [ ] Streak / combo system

### Phase 4 — Distribution
- [ ] PWA: manifest.json + service worker
- [ ] Add to Home Screen prompt
- [ ] Offline mode (full offline play)
- [ ] Share score to social media
- [ ] Embed mode (iframe-friendly, config via URL params)

---

## 11. Glossary

| Term | Definisi |
|---|---|
| **Choice Card** | Kartu di panel kiri yang bisa di-drag. Berisi teks pilihan + nomor + aksen warna |
| **Drop Zone** | Area target di panel kanan (Kolom A / Kolom B) tempat kartu di-drop |
| **Placed Item** | Kartu yang sudah berhasil di-drop ke salah satu kolom, tampil sebagai chip berwarna |
| **Split** | Kondisi di mana 3 kartu tersebar di 2 kolom berbeda → tidak diizinkan |
| **Confetti** | 100 partikel warna-warni yang jatuh saat jawaban benar |
| **Shake** | Animasi getar pada game container saat jawaban salah atau split |
| **Ghost Clone** | Elemen DOM sementara yang mengikuti jari saat touch-drag di mobile |

---

## 12. Appendix

### A. Customizing Questions

Edit 3 konstanta di dalam `<script>` tag `index.html`:

```javascript
// 1. Tentukan 3 jawaban benar (0-based index)
const CORRECT_ANSWERS = [1, 3, 5];

// 2. Daftar pilihan (minimal 6 item)
const CHOICES = [
  { text: "Pilihan 1", color: "card-1" },
  { text: "Pilihan 2", color: "card-2" },
  { text: "Pilihan 3", color: "card-3" },
  { text: "Pilihan 4", color: "card-4" },
  { text: "Pilihan 5", color: "card-5" },
  { text: "Pilihan 6", color: "card-6" },
];

// 3. (Opsional) Update subtitle di HTML:
// <p class="subtitle">Drag 3 pernyataan yang BENAR ke kotak di kanan</p>
```

### B. Color Variants

6 warna kartu tersedia via class `card-1` s/d `card-6`:

| Class | Hex | Nama |
|---|---|---|
| `card-1` | `#6c5ce7` | Purple |
| `card-2` | `#e17055` | Coral |
| `card-3` | `#00b894` | Green |
| `card-4` | `#fdcb6e` | Gold |
| `card-5` | `#0984e3` | Blue |
| `card-6` | `#fd79a8` | Pink |
