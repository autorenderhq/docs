# AutoRender Video Transformations

This guide lists the video transformation parameters, what they do, and practical examples.

## URL Format

```text
https://assets.autorender.io/{workspace}/{transformations}/{path/to/video.mp4}
```

Example base:

```text
https://assets.autorender.io/LOKVTtKVGb/{transformations}/doc/skateboarding.mp4
```

- `workspace`: your workspace id
- `transformations`: comma-separated parameters
- `path/to/video.mp4`: source video path

---

## Parameter Reference

| Category | Param | Meaning | Example |
|---|---|---|---|
| Thumbnail | `thumb_ar` | Generate thumbnail (WebP), keep original aspect ratio | `thumb_ar` |
| Size | `w_{n}` | Output width in pixels | `w_500` |
| Size | `h_{n}` | Output height in pixels | `h_500` |
| Aspect ratio | `ar_{ratio}` | Force output ratio | `ar_1:1` |
| Crop/Pad | `cm_pad_resize` | Resize with padding to fit | `cm_pad_resize` |
| Crop/Pad | `cm_extract` | Extract/crop region mode | `cm_extract` |
| Background | `bg_{color}` | Pad color (name or hex) | `bg_white` |
| Trim | `so_{sec}` | Start offset in seconds | `so_5` |
| Trim | `eo_{sec}` | End offset in seconds | `eo_15` |
| Format | `f_gif` | Convert output to GIF | `f_gif` |
| Flip | `flip_h` | Horizontal flip | `flip_h` |
| Flip | `flip_v` | Vertical flip | `flip_v` |
| Flip | `flip_hv` | Horizontal + vertical flip | `flip_hv` |
| Rotate | `r_{deg}` | Rotate by degrees | `r_90` |

---

## Examples

### 1) Thumbnail from video

```text
https://assets.autorender.io/LOKVTtKVGb/thumb_ar/doc/skateboarding.mp4
```

Returns a thumbnail image for preview cards/lists.

### 2) Resize video

```text
https://assets.autorender.io/LOKVTtKVGb/w_500,h_500/doc/skateboarding.mp4
```

Good for standard player viewport sizes.

### 3) Aspect-ratio output

```text
https://assets.autorender.io/LOKVTtKVGb/ar_1:1,w_500/doc/skateboarding.mp4
```

Useful for square feed layouts.

### 4) Pad resize with background

```text
https://assets.autorender.io/LOKVTtKVGb/w_500,h_500,cm_pad_resize,bg_white/doc/skateboarding.mp4
```

Fits full source without cutting content; empty area is padded.

### 5) Trim clip (start/end offsets)

```text
https://assets.autorender.io/LOKVTtKVGb/so_5,eo_15,w_500,h_500/doc/skateboarding.mp4
```

Creates a clip from 5s to 15s, then resizes.

### 6) Convert a trimmed segment to GIF

```text
https://assets.autorender.io/LOKVTtKVGb/w_500,h_500,f_gif,so_5,eo_15/doc/skateboarding.mp4
```

Great for short previews; keep duration small to control file size.

### 7) Flip video

```text
https://assets.autorender.io/LOKVTtKVGb/w_500,flip_h/doc/skateboarding.mp4
```

Mirrors the video horizontally.

### 8) Rotate video

```text
https://assets.autorender.io/LOKVTtKVGb/w_500,r_90/doc/skateboarding.mp4
```

Rotates output by 90 degrees.

---

## Notes / Best Practices

- First request can take longer (processing), later requests are usually cached.
- For GIF output, use small dimensions and short trims to keep response size manageable.
- Prefer combining only the params you need; simpler URLs are easier to debug.
- Use `so_` and `eo_` for trim windows (start/end offsets).

