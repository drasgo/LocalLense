# LocalLens

**AI photo retouching for everyone – 100% in your browser.**

LocalLens is a privacy-first AI photo editing platform that runs entirely in your browser using ONNX Runtime Web. No uploads, no accounts, no tracking.

![LocalLens Preview](https://via.placeholder.com/800x400?text=LocalLens+Preview)

## Features

- 🖥️ **100% Client-Side** - All AI processing happens on your device
- 🔒 **Privacy First** - Images never leave your browser
- ⚡ **5 AI Tools** - Super-resolution, lighting fix, background removal, cartoonizer, and 3D photo
- 📱 **Responsive** - Works on desktop, tablet, and mobile
- 🎨 **Modern UI** - Clean, beautiful interface with smooth animations

## Tools

| Tool | Description |
|------|-------------|
| **Super-Resolution** | Upscale and enhance image details with AI |
| **Lighting Fix** | Enhance low-light and poorly exposed photos |
| **Background Removal** | Automatically remove image backgrounds |
| **Cartoonizer** | Transform photos into cartoon-style artwork |
| **3D Photo** | Generate depth maps with interactive parallax effects |

## Quick Start

1. Serve the folder locally (required for model downloads):
   ```bash
   python3 -m http.server 8000
   ```
2. Open `http://localhost:8000/index.html` in any modern browser
3. That's it! The UI is fully functional

To enable AI processing, you need to configure the model URLs (see below).

## Configuration

Edit the model URL constants at the top of the `<script>` section in `index.html`:

```javascript
// Replace these with your actual ONNX model URLs
const MODEL_URL_SUPER_RES  = "REPLACE_WITH_SUPER_RES_MODEL_URL.onnx";
const MODEL_URL_LOW_LIGHT  = "REPLACE_WITH_LOW_LIGHT_MODEL_URL.onnx";
const MODEL_URL_BG_REMOVAL = "REPLACE_WITH_BG_REMOVAL_MODEL_URL.onnx";
const MODEL_URL_CARTOON    = "REPLACE_WITH_CARTOON_MODEL_URL.onnx";
const MODEL_URL_DEPTH      = "REPLACE_WITH_DEPTH_MODEL_URL.onnx";

// Your donation link
const DONATE_URL = "https://buymeacoffee.com/REPLACE_USERNAME";
```

### Recommended ONNX Models

Here are some open-source models you can use (convert to ONNX format if needed):

| Tool | Recommended Models |
|------|-------------------|
| Super-Resolution | Real-ESRGAN, ESPCN, EDSR |
| Lighting Fix | Zero-DCE, EnlightenGAN, RetinexNet |
| Background Removal | U2-Net, RMBG-1.4, MODNet |
| Cartoonizer | AnimeGAN, White-box Cartoonization |
| 3D Photo / Depth | MiDaS, DPT, Depth Anything |

### Model Input/Output Format

The code expects models with this default format:
- **Input**: `[1, 3, H, W]` (NCHW), float32, normalized `[0, 1]`
- **Output**: `[1, 3, H, W]` (RGB) or `[1, 1, H, W]` (depth)
- **Default size**: 512×512 (configurable per tool in `toolsConfig`)

If your model uses NHWC format or different dimensions, modify the `imageToTensor()` and `tensorToImageData()` functions.

## Project Structure

```
LocalLens/
├── index.html    # Complete single-page application
└── README.md     # This file
```

Everything is contained in a single `index.html` file:
- HTML structure
- CSS styles (`<style>` block)
- JavaScript logic (`<script>` block)

## Code Architecture

### CSS Organization

| Section | Description |
|---------|-------------|
| Variables & Reset | CSS custom properties, box-sizing reset |
| Navigation | Sticky nav bar with scroll spy highlighting |
| Hero | Animated gradient background, feature pills |
| Sections | Generic section styling, headers |
| Tools Grid | Card grid for tool overview |
| Tool Sections | Individual tool layouts with drop zones |
| Pro & Donate | Feature cards and CTA |
| FAQ | Accordion-style Q&A |
| Footer | Simple footer with dynamic year |
| Responsive | Mobile breakpoint adjustments |

### JavaScript Organization

| Function | Purpose |
|----------|---------|
| `isModelConfigured(url)` | Checks if model URL is valid (not placeholder) |
| `setStatus(toolKey, state, message)` | Updates tool status display |
| `imageToTensor(img, targetSize)` | Converts image to ONNX tensor (NCHW format) |
| `tensorToImageData(tensor, ...)` | Converts tensor back to ImageData |
| `loadModel(toolKey)` | Lazy-loads and caches ONNX sessions |
| `runTool(toolKey)` | Main inference pipeline |
| `downloadResult(toolKey)` | Triggers PNG download |
| `handleFileSelect(toolKey, file)` | Processes file input/drop |
| `clearImage(toolKey)` | Resets tool state |
| `setup*()` | Various initialization functions |

### Tool Configuration

Each tool is configured in the `toolsConfig` object:

```javascript
const toolsConfig = {
    superRes: {
        modelUrl: MODEL_URL_SUPER_RES,
        inputSize: 512,        // Model input dimensions
        name: 'Super-Resolution'
    },
    depth: {
        modelUrl: MODEL_URL_DEPTH,
        inputSize: 512,
        name: '3D Photo',
        isDepth: true          // Single-channel depth output
    },
    bgRemoval: {
        modelUrl: MODEL_URL_BG_REMOVAL,
        inputSize: 512,
        name: 'Background Removal',
        outputAlpha: true      // Output includes alpha channel
    },
    // ...
};
```

## Customization

### Changing Colors

Edit the CSS variables in `:root`:

```css
:root {
    --bg-primary: #f8f9fb;           /* Page background */
    --bg-card: #ffffff;              /* Card background */
    --accent-primary: #0ea5e9;       /* Primary accent (electric blue) */
    --accent-secondary: #a855f7;     /* Secondary accent (purple) */
    /* ... */
}
```

### Adding a New Tool

1. Add the model URL constant:
   ```javascript
   const MODEL_URL_NEW_TOOL = "REPLACE_WITH_NEW_TOOL_MODEL_URL.onnx";
   ```

2. Add to `toolsConfig`:
   ```javascript
   newTool: {
       modelUrl: MODEL_URL_NEW_TOOL,
       inputSize: 512,
       name: 'New Tool'
   }
   ```

3. Add to `sessions` object:
   ```javascript
   const sessions = { ..., newTool: null };
   ```

4. Add HTML for the tool card and section (copy existing pattern)

### Changing Model Input Format

If your model expects NHWC format instead of NCHW, modify `imageToTensor()`:

```javascript
// For NHWC: [1, H, W, 3]
const tensorData = new Float32Array(1 * targetSize * targetSize * 3);
for (let y = 0; y < targetSize; y++) {
    for (let x = 0; x < targetSize; x++) {
        const pixelIdx = (y * targetSize + x) * 4;
        const tensorIdx = (y * targetSize + x) * 3;
        tensorData[tensorIdx + 0] = data[pixelIdx + 0] / 255.0; // R
        tensorData[tensorIdx + 1] = data[pixelIdx + 1] / 255.0; // G
        tensorData[tensorIdx + 2] = data[pixelIdx + 2] / 255.0; // B
    }
}
return new ort.Tensor('float32', tensorData, [1, targetSize, targetSize, 3]);
```

## Browser Support

- ✅ Chrome 90+
- ✅ Firefox 88+
- ✅ Safari 15+
- ✅ Edge 90+

Requires WebGL or WebGPU for GPU acceleration. Falls back to WASM (CPU) on older devices.

## Performance Tips

1. **Use smaller images** - Processing time scales with image size
2. **Close other tabs** - Free up memory for model inference
3. **Use a modern GPU** - WebGL/WebGPU acceleration significantly improves speed
4. **First run is slow** - Models are cached after initial download

## Deployment

LocalLens is a static site. Deploy to any static hosting:

- **GitHub Pages**: Push to `gh-pages` branch
- **Netlify/Vercel**: Connect your repository
- **S3/CloudFlare**: Upload `index.html`

### CORS for Models

If hosting models on a different domain, ensure CORS headers are set:
```
Access-Control-Allow-Origin: *
```

## License

MIT License - Feel free to use, modify, and distribute.

## Contributing

Contributions welcome! Please open an issue or PR.

## Support

If LocalLens saves you time, consider [buying me a coffee](https://buymeacoffee.com/REPLACE_USERNAME)! ☕

---

Built with ❤️ using ONNX Runtime Web



# TODO Futuri

* background removal: https://huggingface.co/onnx-community/BiRefNet-ONNX
* image 2 3d object: https://huggingface.co/ashawkey/LGM
* image 2 anime: https://github.com/TachibanaYoshino/AnimeGANv3?tab=readme-ov-file
*