---
title: "Video Reference"
description: "Quick reference guide for all AutoRender video transformation parameters"
---

## URL Format

Standard AutoRender video URLs follow this structure:

```text
https://assets.autorender.io/{workspace}/{transformations}/{path/to/video.mp4}
```

## Parameter Reference

| Category      | Param        | Meaning                             | Example        |
| ------------- | ------------ | ----------------------------------- | -------------- |
| **Thumbnail** | `thumb_ar`   | Generate WebP thumbnail             | `thumb_ar`     |
| **Size**      | `w_{n}`      | Output width                        | `w_500`        |
| **Size**      | `h_{n}`      | Output height                       | `h_500`        |
| **Ratio**     | `ar_{ratio}` | Force aspect ratio                  | `ar_16:9`      |
| **Mode**      | `cm_pad_resize` | Resize with padding              | `cm_pad_resize`|
| **Trim**      | `so_{sec}`   | Start offset                        | `so_5`         |
| **Trim**      | `eo_{sec}`   | End offset                          | `eo_15`        |
| **Format**    | `f_gif`      | Convert to GIF                      | `f_gif`        |
| **Flip**      | `flip_h`     | Horizontal flip                     | `flip_h`       |
| **Rotate**    | `r_{deg}`    | Rotate (90, 180, 270)               | `r_90`         |

## Common Workflow Examples

### 1. Generating a Video Thumbnail
Returns a high-quality WebP image of the first frame.
```text
https://assets.autorender.io/LOKVTtKVGb/thumb_ar/doc/skateboarding.mp4
```

### 2. Social Media Ready (Square)
Forces a 1:1 ratio and resizes for mobile feeds.
```text
https://assets.autorender.io/LOKVTtKVGb/ar_1:1,w_500/doc/skateboarding.mp4
```

### 3. Creating a 10s Preview Clip
Trims the video from 5 to 15 seconds.
```text
https://assets.autorender.io/LOKVTtKVGb/so_5,eo_15/doc/skateboarding.mp4
```

### 4. Animated GIF Preview
Converts a trimmed segment to a lightweight GIF.
```text
https://assets.autorender.io/LOKVTtKVGb/f_gif,so_0,eo_5,w_300/doc/skateboarding.mp4
```

## Best Practices

- **Caching**: The first request triggers processing; subsequent requests are served from the global CDN.
- **Clip Length**: For GIF conversion, keep durations under 15 seconds for optimal performance.
- **Responsive**: Use `ar_` with `w_` for flexible, responsive video layouts.
