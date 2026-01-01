# Multi-Input Enhancement for NLP2SQL

## Overview

The NLP2SQL application has been enhanced with **multi-input support**, allowing users to provide queries in three different ways:

1. **Text Input** - Traditional text entry
2. **Image (OCR)** - Upload an image containing text
3. **Voice** - Upload or record audio queries

## Features

### 1. Text Input

- **Method**: Direct text entry through a Streamlit text input field
- **Use Case**: Quick query entry for users who prefer typing
- **Validation**: Non-empty text validation

### 2. Image Input (OCR)

- **Method**: Upload JPG or PNG image files
- **Technology**: Pytesseract (wrapper for Tesseract OCR)
- **Supported Formats**: JPG, JPEG, PNG
- **Process**:
  1. User uploads an image
  2. Image preview is displayed
  3. Tesseract extracts text from the image
  4. Extracted text is processed as a query
- **Error Handling**:
  - Validates Tesseract installation
  - Handles images with no readable text
  - Provides helpful error messages

### 3. Voice Input (Audio)

- **Methods**:
  - 🎙️ **Live Microphone Recording** - Real-time recording directly in browser
  - 📁 **File Upload** - Upload pre-recorded audio files
- **Supported Formats**: WAV, MP3, M4A, FLAC, OGG
- **Two Transcription Options**:

#### Live Microphone Recording (NEW!)

- **Method**: Real-time voice recording using browser microphone
- **How It Works**:
  1. Click on the microphone icon in the audio input widget
  2. Record your SQL query
  3. Click stop when finished
  4. Audio is automatically transcribed
  5. Result appears in the chat interface
- **Requirements**:
  - Streamlit 1.35.0 or higher
  - Microphone hardware
  - Browser microphone permission
- **Advantages**:
  - No file upload needed
  - Quick and convenient
  - Similar to Google's voice search

#### Option A: Google Speech Recognition (Free)

- **Library**: SpeechRecognition
- **Advantages**:
  - Completely free
  - No API key required
  - Works offline after initial setup
- **Limitations**:
  - Less accurate than Whisper
  - Works best with clear audio

#### Option B: OpenAI Whisper (Better Accuracy)

- **Library**: OpenAI
- **Advantages**:
  - Superior accuracy
  - Better handling of accents and background noise
  - Supports 99+ languages
- **Requirements**:
  - Valid OPENAI_API_KEY environment variable
  - API cost per request
- **Process**:
  1. User selects Whisper option
  2. Uploads or records audio
  3. Audio is sent to OpenAI API
  4. Transcribed text is returned
  5. Text is processed as a query

## Installation

### Prerequisites

Ensure you have Python 3.8+ installed.

### Step 1: Install Python Dependencies

```bash
pip install -r requirements.txt
```

### Step 2: Install Tesseract OCR (for Image Input)

**Windows:**

1. Download the installer from: https://github.com/UB-Mannheim/tesseract/wiki
2. Run the installer and note the installation path (default: `C:\Program Files\Tesseract-OCR`)
3. Set environment variable or configure in code:
   ```python
   import pytesseract
   pytesseract.pytesseract.pytesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'
   ```

**macOS:**

```bash
brew install tesseract
```

**Linux (Ubuntu/Debian):**

```bash
sudo apt-get install tesseract-ocr
```

### Step 3: Configure API Keys (Optional)

For Whisper voice input, set your OpenAI API key:

```bash
# Create or edit .env file in project root
OPENAI_API_KEY=your_api_key_here
```

## File Structure

```
NLP2SQL/
├── app/
│   └── NLP2SQL.py                 # Main application (updated)
├── src/
│   ├── input/
│   │   ├── __init__.py            # Module initialization
│   │   └── Input_Processor.py     # Multi-input processing logic
│   ├── database/
│   │   └── DB_Config.py
│   ├── api/
│   │   └── LLM_Config.py
│   └── prompts/
│       └── Base_Prompt.py
└── requirements.txt               # Updated with OCR/Voice libraries
```

## Usage

### 1. Start the Application

```bash
streamlit run app/NLP2SQL.py
```

### 2. Select Input Mode

In the sidebar, you'll see a radio button group labeled "📝 Select Input Mode:":

- Choose **Text** for direct text entry
- Choose **Image (OCR)** to upload an image
- Choose **Voice** to upload or record audio

### 3. Select Database and Tables

- Choose your database type (SQLite or PostgreSQL)
- Upload or connect to your database
- Select the tables you want to query

### 4. Provide Your Query

Based on your selected input mode:

**Text Mode:**

- Type your natural language query directly

**Image Mode:**

- Click "Upload an image (JPG, PNG)"
- Image preview will appear
- Text will be automatically extracted
- Processing status displayed as "🔍 Extracting text from image..."

**Voice Mode:**

- Choose transcription method:
  - Google Speech Recognition (Free)
  - OpenAI Whisper (Better accuracy)
- Click "Upload an audio file"
- Or record audio directly
- Processing status displayed as "🎤 Transcribing audio..."

### 5. View Results

Once the query is processed, the application will:

1. Generate SQL from your input
2. Execute the query
3. Display results in a table
4. Show visualizations and analysis

## Input Processor Module

### Class: InputProcessor

#### Methods:

**`process_text_input(text: str) -> Tuple[bool, str]`**

- Validates and processes text input
- Returns: (success, processed_text or error_message)

**`process_image_input(uploaded_image) -> Tuple[bool, str]`**

- Extracts text from uploaded image using OCR
- Returns: (success, extracted_text or error_message)

**`process_audio_input(uploaded_audio) -> Tuple[bool, str]`**

- Transcribes audio using Google Speech Recognition
- Returns: (success, transcribed_text or error_message)

**`process_audio_with_whisper(uploaded_audio) -> Tuple[bool, str]`**

- Transcribes audio using OpenAI Whisper API
- Returns: (success, transcribed_text or error_message)

**`get_input_from_user(input_type: str) -> Tuple[bool, str]`**

- Main method that handles all input types
- Manages UI components based on input_type
- Returns: (success, user_input)

## Error Handling

### Text Input Errors

- Empty or whitespace-only input
- Handled gracefully with user feedback

### Image Input Errors

- **Tesseract Not Found**: Provides download link
- **No Text Found**: Suggests uploading clearer image
- **File Format Issues**: Only JPG/PNG accepted
- **Processing Errors**: Detailed error messages

### Audio Input Errors

- **Unsupported Format**: Accepts WAV, MP3, M4A, FLAC, OGG
- **Audio Quality Issues**:
  - Google: "Could not understand the audio"
  - Whisper: Handles more gracefully
- **API Errors**: Network/service issues
- **Missing Dependencies**: Provides installation instructions

## Logging

The application logs all input processing operations:

- Successful text extractions
- OCR processing status
- Audio transcription status
- Error details for debugging

Check logs for:

```
InputProcessor.process_image_input() -> Extracted 150 characters
InputProcessor.process_audio_input() -> Transcribed query successfully
```

## Performance Considerations

### Text Input

- Instant processing
- Recommended for best performance

### Image Input

- Processing time: 2-10 seconds depending on image size
- Larger images take longer
- Clear text improves accuracy

### Voice Input

- **Google**: ~5-15 seconds per upload
- **Whisper**: ~10-30 seconds (API latency)
- Quality audio = faster processing

## Troubleshooting

### Image (OCR) Issues

**Problem**: "Tesseract OCR is not installed"
**Solution**:

1. Download and install from: https://github.com/UB-Mannheim/tesseract/wiki
2. Set the path in Python if needed

**Problem**: Poor OCR accuracy
**Solution**:

- Use images with clear, black text on white background
- Ensure good image resolution (150+ DPI)
- Avoid rotated or skewed text

### Voice (Audio) Issues

**Problem**: "Could not understand the audio" (Google)
**Solution**:

- Record in a quiet environment
- Speak clearly and at normal pace
- Try Whisper instead for better accuracy

**Problem**: "OpenAI API key not configured"
**Solution**:

1. Get API key from https://openai.com
2. Create `.env` file in project root
3. Add: `OPENAI_API_KEY=your_key_here`

## Technologies Used

### New Libraries Added

- **pytesseract** (≥0.3.10): OCR text extraction
- **Pillow** (≥10.0.0): Image handling
- **SpeechRecognition** (≥3.10.0): Audio transcription
- **pydub** (≥0.25.1): Audio processing

### Existing Libraries Utilized

- **streamlit**: UI and file upload widgets
- **openai**: Whisper API integration
- **python-dotenv**: Environment variable management

## Example Workflow

### Text Query

```
User Input: "Show me all customers from New York"
→ Direct text processing
→ SQL Generation
→ Results displayed
```

### Image Query

```
User uploads image with text: "Show me all customers from New York"
↓
OCR extracts: "Show me all customers from New York"
↓
SQL Generation
↓
Results displayed
```

### Voice Query

```
User records audio: "Show me all customers from New York"
↓
Audio transcribed to: "Show me all customers from New York"
↓
SQL Generation
↓
Results displayed
```

## Security Notes

1. **Image Processing**: All images are processed locally (no external uploads)
2. **Audio Processing**:
   - Google: Sent to Google servers (standard privacy)
   - Whisper: Sent to OpenAI servers (review their privacy policy)
3. **API Keys**: Stored in `.env` file (never commit this to version control)

## Future Enhancements

Potential improvements for future versions:

- Real-time voice recording display
- Audio trimming/editing UI
- Multiple language support for OCR
- Handwriting recognition
- Video input support
- Batch query processing

## Support

For issues or questions:

1. Check the Troubleshooting section above
2. Review application logs in terminal
3. Open an issue on the GitHub repository

---

**Last Updated**: November 13, 2025
**Version**: 2.0 (Multi-Input Enhancement)
