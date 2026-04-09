# AutoRender documentation

This repository contains the [Mintlify](https://mintlify.com) site for **AutoRender**. The guides under `essentials/` expand on everything below in more detail.

---

## URL shape

Dynamic images and videos are served from the CDN by embedding **comma-separated** transformation tokens between the workspace id and the asset path:

```text
https://assets.autorender.io/{workspace}/{transformations}/{path/to/file.jpg}
```

Example (workspace `LOKVTtKVGb`, width 360px, sample path):

```text
https://assets.autorender.io/LOKVTtKVGb/w_360/doc1/fujicam.jpg
```

- **Combine** parameters with commas: `w_360,h_240,c_fill`  
- **Order** of tokens does not change the final result in most cases.  
- **Videos** use the same pattern; some params are video-specific (see [Video](#video-transformations)).

When you upload via the API, image **`transform`** strings use the same token syntax (e.g. `w_800,h_600,q_90`).

---

## Resize and dimensions

| Token | Meaning | Example |
|-------|---------|---------|
| `w_{px}` | Width in pixels | `w_360` |
| `h_{px}` | Height in pixels | `h_240` |
| `w_0.5` / `h_0.75` | Fraction of original size | `w_0.5` (= 50% width, aspect preserved) |

```text
https://assets.autorender.io/LOKVTtKVGb/w_360/doc1/gpose.jpg
https://assets.autorender.io/LOKVTtKVGb/h_360/doc1/earring.jpg
https://assets.autorender.io/LOKVTtKVGb/w_360,h_240,c_fill/doc1/paintbrush.jpg
```

## Aspect ratio

Force a ratio `width:height`, usually with a crop mode:

```text
https://assets.autorender.io/LOKVTtKVGb/ar_16:9,w_360,c_fill/doc1/skate.jpg
https://assets.autorender.io/LOKVTtKVGb/ar_1:1,w_360,c_fill,p_c/doc1/gpose.jpg
https://assets.autorender.io/LOKVTtKVGb/ar_4:3,w_360,c_fill/doc1/skate.jpg
```

Custom ratios: `ar_3:2`, `ar_5:4`, `ar_9:16,h_360,c_fill`, etc.

## Crop modes

| Token | Behavior |
|-------|----------|
| `c_crop` | Crop to exact `w_`×`h_` (default often center). |
| `c_fill` | Scale to cover the box; may crop. |
| `c_fit` | Fit inside the box; may letterbox / pad. |

```text
https://assets.autorender.io/LOKVTtKVGb/w_360,h_360,c_crop/doc1/blackporsche.jpg
https://assets.autorender.io/LOKVTtKVGb/w_360,h_360,c_fill/doc1/blackporsche.jpg
https://assets.autorender.io/LOKVTtKVGb/w_360,h_360,c_fit/doc1/blackporsche.jpg
```

## Crop position (`p_`)

Used with `c_crop` / `c_fill` to bias the focal point:

| Token | Position |
|-------|----------|
| `p_c` | Center |
| `p_tl` `p_t` `p_tr` | Top row |
| `p_bl` `p_b` `p_br` | Bottom row |
| `p_l` `p_r` | Left / right center |

```text
https://assets.autorender.io/LOKVTtKVGb/w_360,h_360,c_crop,p_c/doc1/blackporsche.jpg
https://assets.autorender.io/LOKVTtKVGb/w_360,h_360,c_crop,p_tl/doc1/blackporsche.jpg
```

## Zoom

| Token | Meaning |
|-------|---------|
| `z_{factor}` | `1` = no zoom; `>1` zoom in; `<1` zoom out. |

```text
https://assets.autorender.io/LOKVTtKVGb/z_1.5,w_360,h_360/doc1/gpose.jpg
https://assets.autorender.io/LOKVTtKVGb/z_3,w_360,h_360/doc1/gpose.jpg
```

## Rotate

| Token | Meaning |
|-------|---------|
| `r_{degrees}` | Clockwise degrees (e.g. `90`, `180`, `45`; negatives CCW). |

```text
https://assets.autorender.io/LOKVTtKVGb/w_360,r_90/doc1/flight.jpg
https://assets.autorender.io/LOKVTtKVGb/w_360,r_180/doc1/flight.jpg
```

## Flip

| Token | Meaning |
|-------|---------|
| `flip_h` | Mirror left–right |
| `flip_v` | Mirror top–bottom |
| `flip_hv` | Both (like 180° for symmetric cases) |

```text
https://assets.autorender.io/LOKVTtKVGb/w_360,flip_h/doc1/gpose.jpg
https://assets.autorender.io/LOKVTtKVGb/w_360,flip_v/doc1/gpose.jpg
```

## Border radius

| Token | Meaning |
|-------|---------|
| `br_{px}` | Corner radius in pixels. |

```text
https://assets.autorender.io/LOKVTtKVGb/w_360,br_20/doc1/photographer.jpg
```

Circle (square source, radius ≥ half side):

```text
https://assets.autorender.io/LOKVTtKVGb/w_360,h_360,c_crop,br_180/doc1/photographer.jpg
```

---

## Effects (`e_`)

Effects use `e_{name}:{value}` or flag-style names. They can be chained with commas.

| Token | Example | Notes |
|-------|---------|--------|
| Brightness | `e_brightness:20` | Positive / negative adjustment |
| Contrast | `e_contrast:30` | |
| Saturation | `e_saturation:30` / `e_saturation:-50` | |
| Exposure | `e_exposure:25` | |
| Grayscale | `e_grayscale` | No value |
| Sharpen | `e_sharpen` / `e_sharpen:150` | |
| Hue | `e_hue:30` | −180…180 |
| Temperature | `e_temperature:40` | Cool ↔ warm |
| Warmth | `e_warmth:20` | |

```text
https://assets.autorender.io/LOKVTtKVGb/e_saturation:30,w_360/doc1/porsche.jpg
https://assets.autorender.io/LOKVTtKVGb/e_brightness:15,e_contrast:20,e_saturation:10/docs/example/porsche.jpg
https://assets.autorender.io/LOKVTtKVGb/e_grayscale/docs/example/porsche.jpg
```

More filters (structure, vignette, blur, etc.) are documented in **`essentials/effects.mdx`** and the **`essentials/effects/`** pages.

---

## Layers (text and images)

Text overlay pattern:

```text
l_text:{font}_{size}_{style}_{align}:{url-encoded-text},fl_layer_apply/{path}
```

Examples:

```text
https://assets.autorender.io/LOKVTtKVGb/l_text:arial_24_bold_center:Hello%20World,tp_north-east,fl_layer_apply,w_400,h_400/docs/example/porsche.jpg
https://assets.autorender.io/LOKVTtKVGb/l_text:arial_32_bold_center:Sale,co_FF0000,fl_layer_apply,w_400,h_400/docs/example/porsche.jpg
```

- **Color:** `co_{hex}` (e.g. `co_FF0000`)  
- **Position:** `x_100,y_50` or placement `tp_north`, `tp_south`, etc.  

Full layer and image-underlay options: **`essentials/layers.mdx`**.

---

## Recipes (preset tokens)

Workspace-defined shortcuts use the `t_` prefix:

```text
https://assets.autorender.io/LOKVTtKVGb/t_thumbnail-2/docs/example/porsche.jpg
```

Expands to whatever preset is configured (example equivalent: `c Fill,w_200,h_200,p_c`). See **`essentials/recipes.mdx`**.

---

## Video transformations

Same URL layout; video-specific parameters include:

| Category | Token | Example |
|----------|-------|---------|
| Thumbnail (WebP) | `thumb_ar` | `thumb_ar` |
| Width / height | `w_{n}` / `h_{n}` | `w_500` |
| Aspect ratio | `ar_{ratio}` | `ar_1:1` |
| Pad resize | `cm_pad_resize` | |
| Extract region | `cm_extract` | |
| Pad color | `bg_{color}` | `bg_white` |
| Start offset (s) | `so_{sec}` | `so_5` |
| End offset (s) | `eo_{sec}` | `eo_15` |
| Output GIF | `f_gif` | |
| Flip | `flip_h` / `flip_v` / `flip_hv` | |
| Rotate | `r_{deg}` | `r_90` |

```text
https://assets.autorender.io/LOKVTtKVGb/thumb_ar/doc/skateboarding.mp4
https://assets.autorender.io/LOKVTtKVGb/w_500,so_5,eo_15,f_gif/doc/skateboarding.mp4
```

Details: **`essentials/video/overview.mdx`** and sibling video pages.

---

## Automatic behaviour (summary)

Documented in **`essentials/automatic.mdx`**:

- **Width bucketing** — requested widths may round up to cache-friendly sizes (e.g. 320, 480, 720, 1080, 1440, 1920).  
- **Default width** — if you omit `w_` / `h_` / aspect ratio, a default width may be chosen from the **User-Agent** (mobile vs desktop, Save-Data, etc.).

---

## Where to read more

| Topic | Path in this repo |
|-------|-------------------|
| Resize & responsive | `essentials/resize.mdx` |
| Crop & positions | `essentials/crop.mdx` |
| Zoom | `essentials/zoom.mdx` |
| Rotate | `essentials/rotate.mdx` |
| Flip | `essentials/flip.mdx` |
| Border radius | `essentials/border-radius.mdx` |
| Effects index | `essentials/effects.mdx` |
| Layers | `essentials/layers.mdx` |
| Recipes | `essentials/recipes.mdx` |
| Video | `essentials/video/*.mdx` |
| Upload / multipart / files / folders API | `api-reference/*.mdx`, `UPLOAD_API.md` |

---

## Local development (Mintlify)

From the directory that contains `docs.json`:

```bash
npm i -g mint
mint dev
```

Open the local preview (commonly `http://localhost:3000`). See [Mintlify quickstart](https://starter.mintlify.com/quickstart) for deployment and GitHub integration.
