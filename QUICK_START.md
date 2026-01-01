# Quick Start Guide - Multi-Input NLP2SQL

## 🚀 Quick Setup (5 minutes)

### 1. Install Dependencies

```bash
pip install -r requirements.txt
```

### 2. Install Tesseract OCR (Choose your OS)

**Windows:**

- Download: https://github.com/UB-Mannheim/tesseract/wiki
- Run installer
- Default path: `C:\Program Files\Tesseract-OCR`

**macOS:**

```bash
brew install tesseract
```

**Linux (Ubuntu/Debian):**

```bash
sudo apt-get install tesseract-ocr
```

### 3. Update Streamlit (Required for Microphone)

Microphone recording requires Streamlit 1.35.0+:

```bash
pip install streamlit --upgrade
```

### 4. (Optional) Setup OpenAI Whisper

Create `.env` file in project root:

```
OPENAI_API_KEY=your_api_key_here
```

### 5. Run the App

```bash
streamlit run app/NLP2SQL.py
```

---

## 📝 Using the App

### Step 1: Select Input Mode

In the left sidebar, choose:

- **Text** - Type your query
- **Image (OCR)** - Upload image with text
- **Voice** - Use microphone or upload audio (NEW!)

### Step 2: Select Database

- Choose SQLite or PostgreSQL
- Provide connection details

### Step 3: Select Tables

- Pick the tables to query against

### Step 4: Enter Your Query

- **Text**: Type directly
- **Image**: Upload JPG/PNG
- **Voice**:
  - 🎙️ **Live Recording** (NEW): Click microphone to record
  - 📁 **File Upload**: Upload WAV, MP3, etc.

### Step 5: Select Transcription Service

For Voice input, choose:

- **Google Speech Recognition** (free, instant)
- **OpenAI Whisper** (better accuracy, requires API key)

### Step 6: View Results

- See SQL query generated
- View data in table
- Explore visualizations

---

## 💡 Example Queries

### Text Input

```
"Show me all orders from customers in California"
```

### Image Input

Photograph of:

```
Show top 10 customers by total sales
```

### Voice Input

Record yourself saying:

```
"List all products with price above 100 dollars"
```

---

## 📁 New Files

```
NLP2SQL/
├── src/input/
│   ├── __init__.py              ← NEW
│   └── Input_Processor.py       ← NEW
├── MULTI_INPUT_GUIDE.md         ← NEW (detailed guide)
└── IMPLEMENTATION_SUMMARY.md    ← NEW (technical details)
```

---

## ⚠️ Common Issues & Fixes

### "Tesseract is not installed"

**Solution**: Install from https://github.com/UB-Mannheim/tesseract/wiki

### "Could not understand the audio"

**Solution**:

- Speak clearly and slowly
- Use Whisper instead of Google
- Minimize background noise

### "No text found in image"

**Solution**:

- Use clearer images
- Ensure text is dark on light background
- Higher resolution is better

### "OpenAI API key not configured"

**Solution**:

- Create `.env` file with `OPENAI_API_KEY=...`
- Restart the app

---

## 🔧 Architecture Overview

```
Input Mode Selection (Sidebar)
         ↓
    InputProcessor
    ├─ process_text_input()
    ├─ process_image_input() [uses Tesseract]
    └─ process_audio_input() [uses Google or Whisper]
         ↓
    Extracted Text
         ↓
    LLM SQL Generation
         ↓
    Database Execution
         ↓
    Results & Visualizations
```

---

## 📊 Features

| Feature        | Text    | Image | Voice       |
| -------------- | ------- | ----- | ----------- |
| Speed          | Instant | 2-10s | 5-30s       |
| Cost           | Free    | Free  | Free/Paid   |
| Accuracy       | N/A     | High  | High/Better |
| Requires Setup | No      | Yes   | No/Optional |

---

## 🎯 Best Practices

1. **Text Input**

   - Best for quick queries
   - Type naturally

2. **Image Input**

   - Make sure text is clear
   - Good lighting in photos
   - Straight/level text

3. **Voice Input (Google - Free)**

   - Speak clearly
   - Quiet environment
   - Normal pace

4. **Voice Input (Whisper - Better)**
   - Handles accents well
   - Works with background noise
   - More accurate

---

## 📚 Documentation Files

- **MULTI_INPUT_GUIDE.md** - Complete user/developer guide
- **IMPLEMENTATION_SUMMARY.md** - Technical implementation details
- **This file** - Quick reference

---

## 🔗 Useful Links

- Tesseract OCR: https://github.com/UB-Mannheim/tesseract/wiki
- OpenAI API: https://openai.com
- SpeechRecognition Docs: https://pypi.org/project/SpeechRecognition/
- Streamlit Docs: https://docs.streamlit.io/

---

## 💬 Need Help?

1. Check relevant guide file (see Documentation Files above)
2. Review troubleshooting section
3. Check application terminal for error messages
4. Verify library installations: `pip list | grep -E "streamlit|pytesseract|SpeechRecognition"`

---

**Happy Querying! 🎉**
