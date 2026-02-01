# 🌀 Zero Image Warp

A React/Three.js component that creates an immersive "warp tunnel" effect using your images. Images fly towards or away from the viewer in a procedurally-generated 3D space with adaptive resolution scaling based on movement speed.

![Zero Image Warp Demo](https://img.shields.io/badge/demo-live-brightgreen) ![React](https://img.shields.io/badge/React-18+-61DAFB?logo=react) ![Three.js](https://img.shields.io/badge/Three.js-0.150+-black?logo=three.js) ![License](https://img.shields.io/badge/license-MIT-blue)

## ✨ Features

- **🖼️ Drag & Drop Image Upload** — Simply drop images into the browser to add them to your warp tunnel
- **📐 Automatic Multi-Resolution Scaling** — Each image is processed into three WebP versions (0.25MP, 0.5MP, 1.0MP) preserving aspect ratios
- **💾 IndexedDB Persistence** — Images survive browser refreshes and sessions
- **⚡ Speed-Adaptive Quality** — Higher resolution at slow speeds, more images at high speeds
- **🎨 8 Particle Themes** — Cycle through Teal, Magenta, Cyan, Amber, Emerald, Coral, Violet, and Gold
- **🔀 Bidirectional Travel** — Toggle between flying forward or in reverse
- **🎲 Deterministic Seeding** — Uses Zerobytes position-as-seed methodology for reproducible image placement

## 🚀 Quick Start

### Demo Preview

Open `demo.html` in any modern browser for an instant preview — no build step required!

```bash
# Clone the repository
git clone https://github.com/MushroomFleet/image-warp-jsx.git

# Open the demo
open demo.html
# or on Windows
start demo.html
```

### React Project Integration

```bash
npm install react react-dom three
```

```jsx
import ZeroImageWarp from './components/ZeroImageWarp';

function App() {
  return <ZeroImageWarp />;
}
```

See the [Integration Guide](image-warp-jsx-integration.md) for detailed setup instructions.

## 🎮 Controls

| Key | Action |
|-----|--------|
| `1` | Slow down (−10%) |
| `2` | Reset speed to 1.0x |
| `3` | Speed up (+10%) |
| `4` | Cycle particle theme |
| `5` | Toggle direction |

## 📁 Project Structure

```
image-warp-jsx/
├── ZeroImageWarp.jsx          # Main React component
├── demo.html                   # Standalone browser demo
├── image-warp-jsx-integration.md  # Developer integration guide
└── README.md                   # This file
```

## 🔧 How It Works

### Adaptive Resolution System

The component dynamically selects image resolution based on travel speed:

| Speed | Resolution | Density |
|-------|------------|---------|
| Slow (< 0.5x) | 1.0 MP | 15 images |
| Normal (0.5x – 1.5x) | 0.5 MP | 30 images |
| Fast (> 1.5x) | 0.25 MP | 50 images |

This balances visual quality with performance — detailed images when you can appreciate them, efficient rendering when flying fast.

### Position-as-Seed (Zerobytes)

Image selection uses deterministic FNV-1a hashing:

```javascript
function positionHash(seed, ...coords) {
  let hash = seed & 0xFFFFFFFF;
  for (const coord of coords) {
    hash ^= coord;
    hash = Math.imul(hash, 0x01000193);
  }
  return Math.abs(hash);
}
```

This ensures the same seed and billboard index always select the same image — reproducible across sessions if you persist the seed.

### IndexedDB Storage

Images are processed client-side and stored in IndexedDB with three resolution variants. No server uploads required.

```javascript
{
  name: "photo.jpg",
  aspectRatio: 1.777,
  high: { dataURL: "...", width: 1333, height: 750 },   // 1.0 MP
  medium: { dataURL: "...", width: 943, height: 530 },  // 0.5 MP
  low: { dataURL: "...", width: 667, height: 375 }      // 0.25 MP
}
```

## 🛠️ Configuration

### Speed Settings

```javascript
const CONFIG = {
  defaultSpeed: 1.0,
  minSpeed: 0.1,
  maxSpeed: 5.0,
};
```

### Particle Themes

```javascript
const PARTICLE_THEMES = [
  { name: 'Teal', color: 0x00CED1 },
  { name: 'Magenta', color: 0xFF00FF },
  // Add your own themes
];
```

## 📖 Documentation

- **[demo.html](demo.html)** — Interactive browser demo
- **[image-warp-jsx-integration.md](image-warp-jsx-integration.md)** — Full integration guide with code examples

## 🤝 Contributing

Contributions welcome! Please open an issue first to discuss proposed changes.

## 📄 License

MIT License — see LICENSE file for details.

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{image_warp_jsx,
  title = {Image Warp JSX: Adaptive Resolution 3D Image Tunnel Component},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/image-warp-jsx},
  version = {1.0.0}
}
```

### Donate

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)
