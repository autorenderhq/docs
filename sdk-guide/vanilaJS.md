# @autorender/sdk-core

Core Autorender SDK - shared functionality for upload and viewTag features across all framework adapters.

## Installation

```bash
npm install @autorender/sdk-core
```

## Features

This package contains:
- **Upload SDK**: Core upload functionality
- **ViewTag SDK**: Image transformation and responsive image generation

For framework-specific adapters, use:

- `@autorender/sdk-react` - React adapter
- `@autorender/sdk-vue` - Vue adapter
- `@autorender/sdk-angular` - Angular adapter
- `@autorender/sdk-svelte` - Svelte adapter
- `@autorender/sdk-nextjs` - Next.js adapter
- `@autorender/sdk-react-native` - React Native adapter
- `@autorender/sdk-flutter` - Flutter adapter

## Upload SDK Usage

```javascript
import { createUploader } from '@autorender/sdk-core';
import '@autorender/sdk-core/styles';

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
import { createAR } from '@autorender/sdk-core';

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

## Playground

A **vanilla TypeScript + Vite** sample lives in [`playground/html`](../../playground/html). It uses `createUploader` and `createAR` from `@autorender/sdk-core` (no framework adapter).

From the **repository root**:

```bash
npm install --prefix playground/html
npm run dev --prefix playground/html
```

Entry: `playground/html/src/main.ts` and `style.css`.

For richer **ViewTag + uploader** examples (React components wrapping the same core), see [`playground/react`](../../playground/react) and the other framework playgrounds in the [root README](../../README.md) (section **Development → Playgrounds**).

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
See the main [@autorender/sdk](../README.md) documentation for full API reference.


