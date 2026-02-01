# Image Warp JSX Integration Guide

This guide explains how to integrate the ZeroImageWarp component into your React/Three.js projects.

## Prerequisites

Before integrating, ensure your project has:

- **React 18+** with hooks support
- **Three.js 0.150+** for WebGL rendering
- A modern browser with IndexedDB support

## Installation

### Option 1: NPM/Yarn Project

```bash
# Install dependencies
npm install react react-dom three
# or
yarn add react react-dom three
```

Copy `ZeroImageWarp.jsx` to your components directory:

```
src/
├── components/
│   └── ZeroImageWarp.jsx
```

### Option 2: CDN-Based (Vanilla HTML)

Reference the libraries via CDN and use Babel for JSX transformation:

```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/three@0.160.0/build/three.min.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
```

See `demo.html` for a complete working example.

## Basic Integration

### React App (Vite/Create-React-App/Next.js)

```jsx
// App.jsx
import ZeroImageWarp from './components/ZeroImageWarp';

function App() {
  return <ZeroImageWarp />;
}

export default App;
```

### Full-Page Background

```jsx
// pages/index.jsx
import ZeroImageWarp from '../components/ZeroImageWarp';

export default function Home() {
  return (
    <div style={{ position: 'relative', width: '100vw', height: '100vh' }}>
      <ZeroImageWarp />
      
      {/* Your content overlaid on top */}
      <div style={{ 
        position: 'relative', 
        zIndex: 100,
        padding: '2rem',
        color: 'white'
      }}>
        <h1>Welcome</h1>
      </div>
    </div>
  );
}
```

## Configuration Options

### Modifying Speed Settings

Edit the `CONFIG` object in the component:

```javascript
const CONFIG = {
  defaultSpeed: 1.0,    // Initial speed multiplier
  minSpeed: 0.1,        // Slowest allowed speed
  maxSpeed: 5.0,        // Fastest allowed speed
  spawnDistanceMin: -50,
  spawnDistanceMax: -150,
  forwardRecycleZ: 10,
  backwardRecycleZ: -150,
};
```

### Resolution Tiers

The component automatically scales images to three resolution tiers. Modify thresholds:

```javascript
const RESOLUTION_TIERS = {
  HIGH: 1.0,    // 1 megapixel - slow speeds
  MEDIUM: 0.5,  // 0.5 megapixel - normal speeds
  LOW: 0.25,    // 0.25 megapixel - fast speeds
};
```

### Speed-to-Resolution Mapping

```javascript
function getMegapixelTier(speed) {
  if (speed < 0.5) return 'high';   // Detailed when slow
  if (speed > 1.5) return 'low';    // Efficient when fast
  return 'medium';
}
```

### Billboard Density

```javascript
function getBillboardDensity(speed) {
  if (speed < 0.5) return 15;   // Fewer, larger images
  if (speed > 1.5) return 50;   // More, smaller images
  return 30;                     // Default density
}
```

## Particle Themes

Add or modify color themes:

```javascript
const PARTICLE_THEMES = [
  { name: 'Teal', color: 0x00CED1 },
  { name: 'Magenta', color: 0xFF00FF },
  { name: 'Cyan', color: 0x00FFFF },
  // Add your own:
  { name: 'Custom', color: 0xFF5500 },
];
```

## IndexedDB Storage

### Database Structure

Images are stored with this schema:

```javascript
{
  id: number,           // Auto-incremented
  name: string,         // Original filename
  originalWidth: number,
  originalHeight: number,
  aspectRatio: number,
  high: {               // 1.0 MP version
    dataURL: string,
    width: number,
    height: number,
    megapixels: number,
    sizeBytes: number
  },
  medium: { ... },      // 0.5 MP version
  low: { ... }          // 0.25 MP version
}
```

### Accessing Storage Directly

```javascript
// Get all images
const images = await getAllImages();

// Clear storage
await clearAllImages();

// Get count
const count = await getImageCount();
```

### Custom Storage Namespace

To avoid conflicts with other apps, modify the database name:

```javascript
const DB_NAME = 'MyAppImageWarpDB';  // Change this
const DB_VERSION = 1;
const STORE_NAME = 'images';
```

## Zerobytes Position-as-Seed

The component uses deterministic hashing for image selection:

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

This ensures:
- Same seed + position = same image selection
- Reproducible across sessions (if seed is persisted)
- O(1) lookup complexity

### Persisting the Seed

To maintain consistent image layouts:

```javascript
// Save seed to localStorage
localStorage.setItem('warpSeed', seedRef.current);

// On mount, restore seed
useEffect(() => {
  const savedSeed = localStorage.getItem('warpSeed');
  if (savedSeed) {
    seedRef.current = parseInt(savedSeed, 10);
  }
}, []);
```

## Keyboard Controls

Default bindings (modify in `handleKeyPress`):

| Key | Action |
|-----|--------|
| `1` | Slow down (−10%) |
| `2` | Reset speed |
| `3` | Speed up (+10%) |
| `4` | Cycle particle theme |
| `5` | Toggle direction |

### Custom Controls

```javascript
const handleKeyPress = (event) => {
  switch (event.key) {
    case 'ArrowLeft':
      targetSpeedRef.current = Math.max(CONFIG.minSpeed, targetSpeedRef.current * 0.9);
      break;
    case 'ArrowRight':
      targetSpeedRef.current = Math.min(CONFIG.maxSpeed, targetSpeedRef.current * 1.1);
      break;
    case ' ':  // Spacebar
      directionForwardRef.current = !directionForwardRef.current;
      break;
  }
};
```

## UI Customization

### Hiding the Upload Panel

```jsx
{/* Comment out or remove */}
{/* <UploadPanel ... /> */}
```

### Custom Overlay

Replace the controls overlay:

```jsx
<div style={{
  position: 'fixed',
  bottom: 20,
  left: 20,
  // Your custom styles
}}>
  <p>Speed: {uiState.speed}x</p>
  <button onClick={() => targetSpeedRef.current *= 1.1}>Faster</button>
</div>
```

### Removing All UI

Set a `minimal` prop pattern:

```jsx
const ZeroImageWarp = ({ minimal = false }) => {
  // ...
  return (
    <div>
      {!minimal && <UploadPanel ... />}
      {!minimal && <ControlsOverlay ... />}
      <div ref={containerRef} />
    </div>
  );
};
```

## Programmatic Image Loading

### Preload Images via URL

```javascript
async function loadImageFromURL(url) {
  const response = await fetch(url);
  const blob = await response.blob();
  const file = new File([blob], 'image.jpg', { type: blob.type });
  return processImage(file);
}

// Usage
const imageData = await loadImageFromURL('/assets/image1.jpg');
await storeImage(imageData);
```

### Batch Preload

```javascript
async function preloadImages(urls) {
  for (const url of urls) {
    const imageData = await loadImageFromURL(url);
    await storeImage(imageData);
  }
  // Refresh component state
  const images = await getAllImages();
  setStoredImages(images);
}
```

## Performance Optimization

### Texture Compression

For large image libraries, consider lowering WebP quality:

```javascript
function resizeToWebP(img, targetWidth, targetHeight, quality = 0.7) {
  // Lower quality = smaller storage footprint
}
```

### Billboard Pool Size

Reduce maximum density for lower-end devices:

```javascript
function getBillboardDensity(speed) {
  const maxDensity = window.devicePixelRatio > 1 ? 50 : 30;
  if (speed < 0.5) return Math.min(15, maxDensity);
  if (speed > 1.5) return maxDensity;
  return Math.min(30, maxDensity);
}
```

### Frame Rate Control

```javascript
let lastFrameTime = 0;
const targetFPS = 60;
const frameInterval = 1000 / targetFPS;

const animate = (currentTime) => {
  requestAnimationFrame(animate);
  
  const delta = currentTime - lastFrameTime;
  if (delta < frameInterval) return;
  lastFrameTime = currentTime;
  
  // ... render logic
};
```

## TypeScript Support

Create a type definition file:

```typescript
// ZeroImageWarp.d.ts
interface ImageTierData {
  dataURL: string;
  width: number;
  height: number;
  megapixels: number;
  sizeBytes: number;
}

interface StoredImage {
  id: number;
  name: string;
  originalWidth: number;
  originalHeight: number;
  aspectRatio: number;
  high: ImageTierData;
  medium: ImageTierData;
  low: ImageTierData;
}

interface UIState {
  speed: string;
  direction: string;
  theme: string;
  megapixels: number;
  density: number;
}
```

## Troubleshooting

### Images Not Appearing

1. Check browser console for IndexedDB errors
2. Verify images are valid formats (JPEG, PNG, WebP, GIF)
3. Ensure images finished processing (watch for toast notification)

### Performance Issues

1. Reduce `getBillboardDensity` return values
2. Lower resolution tiers (e.g., 0.5 → 0.3 MP)
3. Check for memory leaks in texture disposal

### IndexedDB Quota Exceeded

```javascript
// Estimate storage usage
const estimate = await navigator.storage.estimate();
console.log(`Used: ${estimate.usage} / ${estimate.quota}`);

// Clear if near limit
if (estimate.usage > estimate.quota * 0.9) {
  await clearAllImages();
}
```

## Framework-Specific Notes

### Next.js

Use dynamic import to avoid SSR issues:

```jsx
import dynamic from 'next/dynamic';

const ZeroImageWarp = dynamic(
  () => import('../components/ZeroImageWarp'),
  { ssr: false }
);
```

### Vite

No special configuration needed. Import directly.

### Create-React-App

Ensure Three.js is not tree-shaken:

```javascript
import * as THREE from 'three';
window.THREE = THREE;  // Expose globally if needed
```

## License

MIT License - See repository for full terms.
