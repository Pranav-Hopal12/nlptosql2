# Microphone Recording Feature Guide

## Overview

The NLP2SQL application now includes **real-time microphone recording** capabilities, similar to Google's voice input interface. Users can record their SQL queries directly using their microphone or upload pre-recorded audio files.

## Features

### 1. **Real-Time Voice Recording** 🎙️

- Click the microphone icon to start recording
- Built-in audio playback to verify recording
- Visual feedback (status, format, file size)
- Requires Streamlit 1.35.0 or higher

### 2. **Audio File Upload** 📁

- Supports multiple formats: WAV, MP3, M4A, FLAC, OGG
- Preview audio before processing
- Traditional file-based input option

### 3. **Dual Transcription Methods** 🔄

Both recording and file upload support:

- **Google Speech Recognition** (Free, instant)
- **OpenAI Whisper** (Higher accuracy, premium)

## Installation & Setup

### 1. Update Streamlit

Microphone recording requires Streamlit 1.35.0 or later:

```bash
pip install streamlit --upgrade
```

Or install all dependencies:

```bash
pip install -r requirements.txt
```

### 2. Verify Dependencies

The following packages are required:

```txt
streamlit>=1.35.0        # Core framework (must have audio_input)
SpeechRecognition>=3.10.0 # Google Speech Recognition
pydub>=0.25.1            # Audio processing
openai>=1.3.0            # Whisper transcription (optional)
```

### 3. System Audio Requirements

#### Windows

- Native audio support (no additional software needed)
- Works with most USB/integrated microphones

#### macOS

- Native audio support
- May require microphone permission in System Preferences

#### Linux

- Install audio libraries:

```bash
sudo apt-get install portaudio19-dev python3-pyaudio
```

## Usage

### Accessing Microphone Feature

1. **Launch the application:**

   ```bash
   streamlit run app/NLP2SQL.py
   ```

2. **Select input mode:** Choose "Voice" from the sidebar dropdown

3. **Select recording method:**
   - **"🎙️ Record Voice"** - Use microphone for real-time recording
   - **"📁 Upload File"** - Use pre-recorded audio file

### Recording a Query

#### Method 1: Live Microphone Recording

```
1. Click "🎙️ Record Voice" tab
2. Select transcription service (Google or Whisper)
3. Click the microphone icon in the audio input widget
4. Speak your SQL query clearly
5. Click stop when finished
6. Review the recording in the audio player
7. Audio is automatically transcribed
8. Query appears in the chat interface
```

#### Method 2: File Upload

```
1. Click "📁 Upload File" tab
2. Select transcription service (Google or Whisper)
3. Click "Browse files" to select audio file
4. Supported formats: WAV, MP3, M4A, FLAC, OGG
5. Audio is automatically transcribed
6. Query appears in the chat interface
```

## Transcription Methods

### Google Speech Recognition (Free)

- **Pros:**
  - Completely free
  - No API key required
  - Instant results
  - Works offline with local audio
- **Cons:**

  - Lower accuracy for accents
  - May have rate limits
  - No conversation context

- **Setup:** No configuration needed

### OpenAI Whisper (Premium)

- **Pros:**
  - Higher accuracy
  - Better accent handling
  - Conversation awareness
  - Supports multiple languages
- **Cons:**

  - Requires OpenAI API key
  - Pay-per-use pricing
  - Requires internet connection

- **Setup:**
  1. Get API key from [OpenAI](https://platform.openai.com/api-keys)
  2. Add to `.env` file:
     ```
     OPENAI_API_KEY=sk-...
     ```

## Code Integration

### Architecture

```
Voice Input
    ├── Real-Time Recording (st.audio_input)
    │   └── Audio Stream → Transcription
    │
    └── File Upload (st.file_uploader)
        └── Audio File → Transcription

        Both methods → TranscriptionService
                    → Text Query
                    → SQL Generation Pipeline
```

### Key Classes

#### InputProcessor

**File:** `src/input/Input_Processor.py`

```python
class InputProcessor:
    @staticmethod
    def get_input_from_user(input_type: str) -> Tuple[bool, str]:
        """
        Main entry point for voice input.
        Handles both microphone recording and file upload.
        """
        if input_type == "Voice":
            # Tab-based interface for microphone and file upload
            # Returns (success, processed_text)
```

**Methods:**

- `process_audio_input()` - Google Speech Recognition
- `process_audio_with_whisper()` - OpenAI Whisper
- `get_input_from_user()` - Unified interface (handles both microphone and file)

### Main Application Integration

**File:** `app/NLP2SQL.py`

```python
def main():
    # Sidebar input mode selector
    input_mode = st.sidebar.radio(
        "Choose input method:",
        ["Text", "Image", "Voice"]
    )

    # Get unified input
    success, user_input = get_user_input_unified(input_mode)

    if success:
        # Process with database
        response = process_with_selected_db(user_input)
```

## Error Handling

### Common Issues & Solutions

#### 1. "Streamlit version too old"

```
Error: AttributeError with audio_input
Solution: pip install streamlit --upgrade
```

#### 2. "Microphone not detected"

```
Error: No microphone available
Solutions:
- Check system audio settings
- Verify microphone is connected
- Try switching to file upload tab
- Restart browser session
```

#### 3. "Transcription failed"

```
Error: Speech Recognition service error
Solutions:
- For Google: Check internet connection, try again
- For Whisper: Verify OPENAI_API_KEY in .env
- Try uploading clearer audio
- Ensure audio is in supported format
```

#### 4. "Permission denied"

```
Error: Browser blocks microphone access
Solutions:
- Allow microphone in browser settings
- Restart browser
- Check browser console for detailed error
- Try incognito/private mode
```

## Performance Optimization

### Audio Quality Settings

For best transcription results:

1. **Recording Quality:**

   - Use 16-bit PCM format
   - Sample rate: 16kHz (Google) or 16kHz-48kHz (Whisper)
   - Mono or stereo (mono is fine)

2. **Microphone Setup:**

   - Use external USB microphone for better quality
   - Position 15-20cm from mouth
   - Minimize background noise
   - Test audio levels before recording

3. **File Format Recommendations:**
   - **WAV** - Highest quality, largest file (use for local files)
   - **MP3** - Good quality, compressed (use for upload)
   - **FLAC** - Lossless compression (use for archival)
   - **OGG** - Good balance (use for web)

## Browser Compatibility

| Browser | Microphone Support | Audio Upload | Status      |
| ------- | ------------------ | ------------ | ----------- |
| Chrome  | ✅ Full support    | ✅ Yes       | Recommended |
| Firefox | ✅ Full support    | ✅ Yes       | Supported   |
| Safari  | ✅ Full support    | ✅ Yes       | Supported   |
| Edge    | ✅ Full support    | ✅ Yes       | Supported   |

## Advanced Configuration

### Custom Audio Parameters

Modify in `src/input/Input_Processor.py`:

```python
# Process audio with custom settings
recognizer = sr.Recognizer()
recognizer.energy_threshold = 4000  # Adjust sensitivity
recognizer.dynamic_energy_threshold = True  # Auto-adjust
```

### Language Support

**Google Speech Recognition:**

```python
# Specify language (default: English)
text = recognizer.recognize_google(audio, language='es-ES')
```

**OpenAI Whisper:**

```python
# Auto-detect language
transcription = client.audio.transcriptions.create(
    model="whisper-1",
    file=audio_file
)

# Or specify language
transcription = client.audio.transcriptions.create(
    model="whisper-1",
    file=audio_file,
    language="en"
)
```

## Testing Microphone Feature

### Manual Testing Checklist

- [ ] Streamlit >= 1.35.0 installed
- [ ] Microphone hardware detected
- [ ] Browser allows microphone access
- [ ] Record 5-10 second test message
- [ ] Playback works correctly
- [ ] Google transcription completes
- [ ] Whisper transcription completes (if enabled)
- [ ] Text appears in chat interface
- [ ] SQL generation works with voice input

### Test Script

```python
# test_microphone.py
import streamlit as st
from src.input.Input_Processor import InputProcessor

st.title("Microphone Test")

# Test real-time recording
st.subheader("Test 1: Live Recording")
audio_data = st.audio_input("Record something:")

if audio_data:
    success, text = InputProcessor.process_audio_input(audio_data)
    st.write(f"Success: {success}")
    st.write(f"Text: {text}")

# Test file upload
st.subheader("Test 2: File Upload")
uploaded_file = st.file_uploader("Upload audio", type=["wav", "mp3"])

if uploaded_file:
    success, text = InputProcessor.process_audio_input(uploaded_file)
    st.write(f"Success: {success}")
    st.write(f"Text: {text}")
```

## API Quotas & Limits

### Google Speech Recognition

- Free service
- No official quota limits (practical: ~1000 requests/day)
- Rate limits apply but rarely hit

### OpenAI Whisper

- **Pricing:** $0.006 per minute of audio
- **Model:** whisper-1 (fixed model)
- **Rate Limits:** Based on organization tier
- **Timeout:** 25MB file size limit

## Troubleshooting

### Debug Mode

Enable debug logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

Check logs for:

```
[DEBUG] Audio data received
[DEBUG] Transcription service: Google
[DEBUG] Processing audio: 15234 bytes
[DEBUG] Transcription result: "..."
```

### Browser Console Errors

Check browser developer console (F12) for WebRTC/audio errors.

### Testing Audio Input

```python
# Verify audio is being captured
if audio_data is not None:
    st.write(f"Audio size: {len(audio_data)} bytes")
    st.write(f"Audio type: {type(audio_data)}")
    st.audio(audio_data, format="audio/wav")
```

## Future Enhancements

Potential improvements for future versions:

1. **Real-time transcription** during recording (streaming)
2. **Multi-language support** with language auto-detection
3. **Audio editing** (trim, normalize) before transcription
4. **Speaker identification** for multi-speaker conversations
5. **Custom vocabulary** for domain-specific terms
6. **Confidence scores** for transcription accuracy
7. **Audio compression** for file uploads
8. **Batch transcription** for multiple files

## FAQ

**Q: Why do I need Streamlit 1.35.0+?**
A: The microphone recording feature uses `st.audio_input()`, which was introduced in Streamlit 1.35.0.

**Q: Can I use both microphone and file upload in the same session?**
A: Yes! You can switch between tabs to use either method.

**Q: Is microphone data stored or logged?**
A: No. Audio is processed in-memory and transcribed. Only the transcribed text is retained.

**Q: What's the maximum recording duration?**
A: No technical limit in Streamlit, but browser memory limitations suggest 10-15 minutes practically.

**Q: Can I use this offline?**
A: Google Speech Recognition requires internet. OpenAI Whisper also requires internet.

**Q: How do I switch between Google and Whisper?**
A: Use the radio button selector before recording/uploading. You can change it between attempts.

**Q: Does voice input support multiple languages?**
A: Google Speech Recognition supports 90+ languages. Specify with `language='xx-XX'`.

**Q: Can I record in background?**
A: No. Recording stops if you switch tabs or close the browser.

## Support & Feedback

For issues or suggestions:

1. Check this guide's Troubleshooting section
2. Review logs in debug mode
3. Test with different audio formats
4. Try alternative transcription service
5. Check browser compatibility
6. Report issue with steps to reproduce

## Version History

### v1.0.0 (Current)

- ✅ Real-time microphone recording
- ✅ Audio file upload (WAV, MP3, M4A, FLAC, OGG)
- ✅ Google Speech Recognition integration
- ✅ OpenAI Whisper integration
- ✅ Tab-based UI for easy switching
- ✅ Audio preview and validation
- ✅ Comprehensive error handling

## Resources

- [Streamlit Audio Documentation](https://docs.streamlit.io/library/api-reference/media/st.audio_input)
- [Google Speech Recognition Docs](https://cloud.google.com/speech-to-text/docs)
- [OpenAI Whisper API](https://platform.openai.com/docs/guides/speech-to-text)
- [WebRTC Standards](https://webrtc.org/)
- [Audio Codecs Guide](https://en.wikipedia.org/wiki/Audio_codec)
