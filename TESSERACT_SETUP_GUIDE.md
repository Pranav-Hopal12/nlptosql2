# Tesseract OCR Setup Guide

## ℹ️ What is Tesseract?

Tesseract is an OCR (Optical Character Recognition) engine used to extract text from images. It's **optional** - only needed if you want to use the **Image (OCR)** input feature.

## 📌 Do You Need It?

### ✅ Install Tesseract if:

- You want to use the **Image (OCR)** input mode
- You want to upload images with text and convert to queries
- You need handwriting/scanned document support

### ❌ Skip Installation if:

- You only use **Text** input (typing queries)
- You only use **Voice** input (microphone or audio files)
- You don't need image processing

## 🚀 Installation by Operating System

### Windows

**Step 1: Download Installer**

- Go to: https://github.com/UB-Mannheim/tesseract/wiki
- Download the latest `.exe` installer
- Look for: `tesseract-ocr-w64-setup-vX.X.exe` or similar

**Step 2: Run Installer**

- Run the downloaded `.exe` file
- Accept the license agreement
- Choose installation directory
- **Default path:** `C:\Program Files\Tesseract-OCR` (recommended)

**Step 3: Verify Installation**

```powershell
tesseract --version
```

Should output version information if installed correctly.

**Step 4: Configure Path (if needed)**
If you installed to a custom location, update the pytesseract config in `Input_Processor.py`:

```python
import pytesseract
pytesseract.pytesseract.pytesseract_cmd = r'C:\your\custom\path\tesseract.exe'
```

### macOS

**Step 1: Install via Homebrew**

```bash
brew install tesseract
```

**Step 2: Verify Installation**

```bash
tesseract --version
```

### Linux (Ubuntu/Debian)

**Step 1: Install Package**

```bash
sudo apt-get update
sudo apt-get install tesseract-ocr
```

**Step 2: Verify Installation**

```bash
tesseract --version
```

### Linux (Fedora/RHEL)

```bash
sudo yum install tesseract
```

### Linux (Arch)

```bash
sudo pacman -S tesseract
```

## 🧪 Test Your Installation

### Quick Test Command

```bash
# Create a simple test image with text
# Or use any image with text, then run:

tesseract image.png output
cat output.txt
```

### Python Test Script

```python
import pytesseract
from PIL import Image

# Open an image with text
img = Image.open('path/to/image.png')

# Extract text
text = pytesseract.image_to_string(img)
print(text)
```

## ❌ Troubleshooting

### "Tesseract is not installed" Error

**Problem:** Even though you installed it, Python can't find it.

**Solutions:**

1. **Verify installation path**

   ```bash
   # Windows
   where tesseract

   # macOS/Linux
   which tesseract
   ```

2. **Add to system PATH**

   - Windows: Add `C:\Program Files\Tesseract-OCR` to PATH
   - macOS/Linux: Usually automatic with package managers

3. **Explicitly set path in code**

   ```python
   import pytesseract
   pytesseract.pytesseract.pytesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'
   ```

4. **Restart Python/IDE** after installation

### "No module named 'pytesseract'" Error

**Problem:** Python package not installed.

**Solution:**

```bash
pip install pytesseract
```

Or install all dependencies:

```bash
pip install -r requirements.txt
```

### Image Recognition Not Working Well

**Causes & Solutions:**

1. **Image quality too low**

   - Use high-resolution images (300+ DPI)
   - Ensure text is clear and readable
   - Good contrast between text and background

2. **Tesseract not configured correctly**

   - Verify path is correct
   - Check file permissions
   - Ensure tesseract.exe/tesseract binary is executable

3. **Language support missing**
   - Tesseract can support multiple languages
   - Install language packs if needed:
   ```bash
   # Windows: Language files usually included
   # macOS: brew install tesseract-lang
   # Linux: sudo apt-get install tesseract-ocr-[lang-code]
   ```

## 📊 Supported Image Formats

Tesseract supports:

- ✅ PNG
- ✅ JPG/JPEG
- ✅ TIFF
- ✅ GIF
- ✅ PDF (with additional tools)
- ✅ BMP
- ✅ WebP

## 🎯 Using Image OCR Feature

Once Tesseract is installed:

1. **Launch app**

   ```bash
   streamlit run app/NLP2SQL.py
   ```

2. **Select input mode**

   - Sidebar: Choose "Image (OCR)"

3. **Upload image**

   - Click "Upload" button
   - Select JPG or PNG file
   - Image displays with preview

4. **Get extracted text**
   - Text appears automatically
   - Text is processed for SQL generation

## 🔧 Advanced Configuration

### Set Language

```python
# In Input_Processor.py
text = pytesseract.image_to_string(image, lang='eng+fra')  # English + French
```

### Adjust Recognition Settings

```python
custom_config = r'--oem 3 --psm 6'
text = pytesseract.image_to_string(image, config=custom_config)
```

**Common configs:**

- `--oem 0`: Legacy engine only
- `--oem 1`: Neural nets only
- `--oem 2`: Both engines
- `--oem 3`: Default (auto select)
- `--psm 6`: Assume a single uniform block of text

## 📚 More Information

- **Tesseract Documentation:** https://github.com/UB-Mannheim/tesseract/wiki
- **PyTesseract:** https://github.com/madmaze/pytesseract
- **Tesseract Configs:** https://tesseract-ocr.github.io/

## ✅ Quick Checklist

- [ ] Downloaded and installed Tesseract
- [ ] Verified installation: `tesseract --version`
- [ ] Python pytesseract installed: `pip install pytesseract`
- [ ] Can run `streamlit run app/NLP2SQL.py` without errors
- [ ] Image (OCR) mode available in sidebar
- [ ] Can upload and process images

## 💡 Summary

| Feature                   | Required | Optional                  |
| ------------------------- | -------- | ------------------------- |
| Text Input                | ✅       | -                         |
| Voice Input (Microphone)  | ✅       | -                         |
| Voice Input (File Upload) | ✅       | -                         |
| Image OCR                 | ❌       | ✅ **Requires Tesseract** |

**Tesseract is optional!** Use the app with Text and Voice input without it. Only install if you need Image OCR feature.

---

## 🎉 You're Good to Go!

If you only use **Text** or **Voice** input, you don't need Tesseract. The app works great without it!

If you want Image OCR later, you can install Tesseract anytime and the feature will automatically become available.

Happy querying! 🚀
