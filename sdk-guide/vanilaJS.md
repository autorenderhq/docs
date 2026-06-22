# @autorender/js

Core Autorender SDK - shared functionality for upload and viewTag features across all framework adapters.

## Installation

```bash
npm install @autorender/js
```

## Features

This package contains:
- **Upload SDK**: Core upload functionality
- **ViewTag SDK**: Image transformation and responsive image generation

For framework-specific adapters, use:

- `@autorender/react` - React adapter
- `@autorender/vue` - Vue adapter
- `@autorender/angular` - Angular adapter
- `@autorender/svelte` - Svelte adapter
- `@autorender/nextjs` - Next.js adapter
- `@autorender/react-native` - React Native adapter

## Upload SDK Usage

```javascript
import { createUploader } from '@autorender/js';
import '@autorender/js/styles';

const uploader = createUploader({
  apiKey: 'your-api-key',
  target: '#uploader-container',
  type: 'inline',
  allowMultiple: true,
  onSuccess: ({ files }) => {
    console.log('Uploaded:', files);
  },
});
```

## ViewTag SDK Usage

### Basic Setup

```javascript
import { createAR } from '@autorender/js';

const AR = createAR({
  baseUrl: 'https://assets.autorender.io',
  workspace: 'ws_123',
});
```

### Generate Image URLs

```javascript
// Simple URL generation
const url = AR.url('products/shoe.jpg', {
  w: 400,
  h: 400,
  fit: 'cover'
});
// Output: https://assets.autorender.io/ws_123/w_400,h_400,c_fill,f_auto,q_auto/products/shoe.jpg

// Get transformation string only
const transformString = AR.transformString({ w: 300, h: 300, fit: 'cover', q: 70 });
// Output: w_300,h_300,c_fill,q_70

```

### Generate Video URLs

```javascript
// MP4
const mp4Url = AR.url('docs/skateboarding.mp4', { w: 960, h: 540 });

// HLS master playlist
const hlsUrl = AR.url('st_240_360_480_720/docs/skateboarding.mp4/ar-master.m3u8');

// DASH manifest
const dashUrl = AR.url('processing/docs/skateboarding.mp4/dash/manifest.mpd');
```

### Responsive Images

```javascript
// Generate responsive image attributes
const attrs = AR.responsiveImageAttributes({
  src: 'hero.jpg',
  width: 400,
  sizes: '(min-width: 800px) 33vw, 100vw'
});

// Returns: { src, srcSet, sizes, width }
```

### Device Detection

```javascript
// Get device pixel ratio
const dpr = AR.getDPR(); // 1 or 2
```

## API Reference

### `createAR(config: CreateARConfig): ARInstance`

Creates a new AutoRender client instance for image transformations.

**Config Options:**
- `baseUrl?: string` - Base URL (default: `'https://assets.autorender.io'`)
- `workspace: string` - Your workspace ID
- `defaults?: { f?: string, q?: string | number }` - Default transformations
- `deviceBreakpoints?: number[]` - Device breakpoints for responsive images
- `imageBreakpoints?: number[]` - Image breakpoints for responsive images

### `ARInstance`

- `url(src: string, transform?: TransformOptions): string` - Generate transformed image URL
  - `src` supports workspace paths (e.g., `products/shoe.jpg`) and absolute remote URLs (e.g., `https://example.com/image.jpg`).
- `transformString(transform: TransformOptions): string` - Get transformation string only
- `responsiveImageAttributes(options: ResponsiveOptions): ResponsiveAttributes` - Generate responsive image attributes
- `getDPR(): number` - Get device pixel ratio (1 or 2)
See the main [full documentation](https://autorender.mintlify.app) documentation for full API reference.


