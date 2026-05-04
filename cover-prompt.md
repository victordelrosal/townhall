# Townhall — cover image prompt

**Generated with:** Google Gemini Nano Banana 2 (`gemini-3.1-flash-image-preview`)
**Date:** 2026-05-04
**Output file:** `cover-final.png` (post-processed: chroma-keyed white background to alpha; diagonal alpha mask applied along the book's perspective base to remove the floating drop-shadow ellipse)

---

## Prompt (verbatim)

```
Hardcover book cover mockup, 3D rendered at slight angle showing the spine on the left, isolated on a pure solid white background (#FFFFFF, completely uniform white, no patterns, no checkerboard, no texture, no gradient — just flat pure white). The 3D book sits on the white background with NO drop shadow, NO cast shadow, NO floor reflection, NO ground shadow ellipse beneath it. The book floats clean against pure white. Portrait orientation hardcover. Deep royal navy blue case binding (almost midnight, saturated). The entire front cover patterned with concentric spiraling rings of hundreds of tiny gold-foil chair silhouettes radiating outward from a central golden circular medallion. The chairs form a hypnotic ornamental mandala texture in metallic gold foil on the deep navy. At the exact centre, a small circular medallion contains a single elegant 19th-century wooden chair silhouette in gold foil, simple profile view. At the very TOP of the cover, in small gold uppercase classical serif type with wide letter-spacing: VICTOR DEL ROSAL. The author's name appears ONLY at the top, never at the bottom. At the bottom third of the cover, a clean cream-coloured rectangular label cartouche with a thin gold border contains EXACTLY TWO LINES of typography in deep navy ink and nothing else: line one is TOWNHALL in large bold uppercase classical serif (Trajan style); line two reads Who Do We Get to Be in the Age of AI in smaller italic serif, with the word Be italicised more strongly. The cartouche has only those two lines. There is NO author name in the cartouche. The book has realistic 3D depth: visible spine on the left side showing TOWNHALL and VICTOR DEL ROSAL set vertically in gold serif lettering on deep navy. Subtle gloss on the cover surface. Classical typography meets modern geometric ornament, museum-quality book design. Pure flat white background only. No shadow of any kind under or around the book.
```

---

## Generation command (CLI)

```bash
python3 ~/.claude/skills/image-gen/scripts/generate_gemini.py "<prompt above>" \
  -o /tmp/townhall-cover-regen/cover-v3-white.png
```

---

## Post-processing (Python / PIL)

1. **Chroma-key white → alpha** with a soft ramp (preserves cast shadow):
   ```python
   from PIL import Image; import numpy as np
   arr = np.array(Image.open('cover-v3-white.png').convert('RGBA'))
   min_ch = arr[..., :3].astype(np.int32).min(axis=-1)
   alpha = np.clip((250 - min_ch) * 255.0 / (250 - 230), 0, 255).astype(np.uint8)
   arr[..., 3] = alpha
   ```

2. **Diagonal alpha mask** along the book's perspective base (using strict-navy detection per column with median-smoothed edge profile, BUF=8 below the binding):
   ```python
   strict_navy = (r < 60) & (g < 60) & (b > 40) & (b < 180) & (alpha > 200)
   # per column, find lowest navy row → median-smooth → wipe below + 8px buffer
   ```

The diagonal mask is what gives the book its clean perspective base on the page, with no detached drop-shadow ellipse below.

---

## Notes

- Earlier attempt with OpenAI GPT Image 1.5 hit a billing hard limit; Gemini was the working fallback.
- The `.cover-stage::after` CSS pseudo-element previously painted a separate radial-gradient shadow under the book in the page layout. That rule has been removed (`townhall/site/index.html`, commit `1ab2f09`); the cover now stands cleanly on the papyrus background with no extraneous shadow.
- Old original cover preserved at `cover-v1-original-backup.png` for fallback.
