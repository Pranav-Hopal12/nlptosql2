# Microphone Voice Input - Code Examples

## Example 1: Basic Microphone Recording Usage

```python
"""
Simple example of using microphone voice input in your application
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor

def main():
    st.title("🎙️ Voice Query Demo")

    # Get voice input from user (uses microphone + file upload)
    success, user_query = InputProcessor.get_input_from_user("Voice")

    if success:
        st.success(f"✅ Query captured: {user_query}")
        # Now you can use user_query for SQL generation
    else:
        st.warning(f"⚠️ {user_query}")  # error message

if __name__ == "__main__":
    main()
```

**Output:**

```
🎙️ Voice Query Demo
- Two tabs appear: "🎙️ Record Voice" | "📁 Upload File"
- User clicks microphone icon and records query
- Text appears automatically
- Success message shown with query text
```

---

## Example 2: Detailed Microphone Implementation

```python
"""
Complete example with error handling and logging
"""

import logging
import streamlit as st
from src.input.Input_Processor import InputProcessor

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def process_voice_query():
    """Process voice query with comprehensive error handling"""

    st.header("🎤 Voice Query Processing")

    # Get voice input
    success, result = InputProcessor.get_input_from_user("Voice")

    if success:
        # Log the captured query
        logger.info(f"Voice query captured: {result[:50]}...")

        # Display success feedback
        with st.container():
            col1, col2 = st.columns([3, 1])
            with col1:
                st.info(f"**Captured Query:** {result}")
            with col2:
                st.metric("Status", "✅ Ready")

        # Optionally modify or confirm query
        confirmed_query = st.text_area(
            "Edit or confirm your query:",
            value=result,
            height=100
        )

        if st.button("Generate SQL"):
            # Process the query further
            logger.info(f"Processing query: {confirmed_query}")
            st.success("Query submitted for SQL generation!")
            return confirmed_query

    else:
        # Handle error cases
        logger.warning(f"Voice input failed: {result}")
        st.error(f"❌ Voice Input Error: {result}")

        # Suggest alternatives
        st.info("""
        **Suggestions:**
        - Ensure your microphone is connected
        - Check browser microphone permissions
        - Try the file upload tab instead
        - Upgrade Streamlit: `pip install streamlit --upgrade`
        """)

    return None

# Run the function
if __name__ == "__main__":
    query = process_voice_query()
```

**Features:**

- Comprehensive logging
- Error handling
- User feedback
- Query confirmation
- Alternative suggestions

---

## Example 3: Comparing Transcription Methods

```python
"""
Compare Google Speech Recognition vs OpenAI Whisper
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor
import time

def compare_transcription_services():
    """Record once and transcribe with both services"""

    st.header("🔄 Transcription Service Comparison")

    col1, col2 = st.columns(2)

    with col1:
        st.subheader("Google Speech Recognition")
        st.markdown("**Pros:**")
        st.markdown("- Free\n- Fast\n- No API key needed")
        st.markdown("**Cons:**")
        st.markdown("- Less accurate\n- Limited languages")

    with col2:
        st.subheader("OpenAI Whisper")
        st.markdown("**Pros:**")
        st.markdown("- Higher accuracy\n- Multilingual\n- Better accents")
        st.markdown("**Cons:**")
        st.markdown("- Requires API key\n- Slower\n- Paid service")

    # Get audio data
    st.markdown("---")
    st.subheader("🎙️ Record Your Query")

    audio_data = st.audio_input("Record message:")

    if audio_data:
        st.success("✅ Audio recorded successfully!")

        # Display audio
        st.audio(audio_data, format="audio/wav")

        # Compare services
        if st.button("Compare Transcriptions"):
            col1, col2 = st.columns(2)

            with col1:
                st.info("**Google Speech Recognition**")
                with st.spinner("Transcribing with Google..."):
                    start = time.time()
                    success_google, text_google = InputProcessor.process_audio_input(audio_data)
                    google_time = time.time() - start

                if success_google:
                    st.success(f"✅ Time: {google_time:.2f}s")
                    st.write(text_google)
                else:
                    st.error(f"❌ {text_google}")

            with col2:
                st.info("**OpenAI Whisper**")
                with st.spinner("Transcribing with Whisper..."):
                    start = time.time()
                    success_whisper, text_whisper = InputProcessor.process_audio_with_whisper(audio_data)
                    whisper_time = time.time() - start

                if success_whisper:
                    st.success(f"✅ Time: {whisper_time:.2f}s")
                    st.write(text_whisper)
                else:
                    st.error(f"❌ {text_whisper}")

            # Comparison results
            st.markdown("---")
            st.subheader("📊 Comparison Results")

            comparison_data = {
                "Service": ["Google", "Whisper"],
                "Time (seconds)": [f"{google_time:.2f}", f"{whisper_time:.2f}"],
                "Success": ["✅" if success_google else "❌", "✅" if success_whisper else "❌"],
            }

            st.dataframe(comparison_data, use_container_width=True)

# Run
if __name__ == "__main__":
    compare_transcription_services()
```

**Use Cases:**

- Testing which service works better
- Understanding performance differences
- Accuracy comparison
- Cost/benefit analysis

---

## Example 4: Multi-Input Demo Application

````python
"""
Full application demonstrating all input types with microphone recording
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor

def main():
    st.set_page_config(page_title="NLP2SQL Multi-Input", layout="wide")

    st.title("🎯 NLP2SQL - Multi-Input Demo")
    st.markdown("Demonstrate all input methods: Text, Image, Voice")

    # Sidebar configuration
    st.sidebar.title("⚙️ Configuration")
    input_mode = st.sidebar.radio(
        "Choose input method:",
        ["Text 📝", "Image 🖼️", "Voice 🎤"]
    )

    # Main content
    st.markdown("---")

    # Get input based on selected mode
    if input_mode == "Text 📝":
        st.subheader("📝 Text Input")
        st.info("Type your SQL query directly")
        success, result = InputProcessor.get_input_from_user("Text")

    elif input_mode == "Image 🖼️":
        st.subheader("🖼️ Image (OCR) Input")
        st.info("Upload an image containing text (JPG or PNG)")
        success, result = InputProcessor.get_input_from_user("Image")

    else:  # Voice
        st.subheader("🎤 Voice Input")
        st.info("Record voice message or upload audio file")
        success, result = InputProcessor.get_input_from_user("Voice")

    # Display results
    st.markdown("---")
    st.subheader("📋 Processing Result")

    if success:
        st.success("✅ Input processed successfully!")
        st.markdown(f"**Extracted Query:**\n```\n{result}\n```")

        # Show next steps
        st.markdown("---")
        st.info("✨ This query is now ready for SQL generation!")

        if st.button("🚀 Generate SQL"):
            st.success("SQL generation would proceed here...")
    else:
        st.error(f"❌ Processing failed: {result}")

if __name__ == "__main__":
    main()
````

**Features:**

- Multi-input selector
- All three input types
- Clean result display
- Next steps guidance

---

## Example 5: Microphone with Database Integration

````python
"""
Complete example: Voice input → SQL generation → Database query
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor

def main():
    st.title("🗣️ Voice-to-SQL Query System")

    # Configuration
    st.sidebar.header("⚙️ Configuration")

    database_type = st.sidebar.radio(
        "Select Database:",
        ["SQLite", "PostgreSQL"]
    )

    # Voice input
    st.header("1️⃣ Voice Input")
    st.info("🎙️ Record your query or upload an audio file")

    success, voice_query = InputProcessor.get_input_from_user("Voice")

    if success:
        st.success(f"✅ Voice captured!")
        st.write(f"**Query:** {voice_query}")

        # Query confirmation
        st.header("2️⃣ Query Confirmation")

        confirmed_query = st.text_area(
            "Edit or confirm the extracted query:",
            value=voice_query,
            height=100
        )

        # Database selection
        st.header("3️⃣ Database Configuration")

        if database_type == "SQLite":
            db_path = st.text_input(
                "SQLite database path:",
                value="data/example.db"
            )
            st.write(f"📁 Using: {db_path}")

        else:
            col1, col2 = st.columns(2)
            with col1:
                pg_host = st.text_input("Host:", value="localhost")
                pg_port = st.text_input("Port:", value="5432")
            with col2:
                pg_user = st.text_input("User:", value="postgres")
                pg_pass = st.text_input("Password:", type="password")

        # SQL generation
        st.header("4️⃣ SQL Generation")

        if st.button("🔄 Generate SQL", use_container_width=True):
            st.info(f"Would generate SQL from query:\n```\n{confirmed_query}\n```")
            st.success("SQL generation placeholder - integrate LLM here")

        # Results display
        st.header("5️⃣ Results")
        st.info("Query results would display here after SQL execution")

    else:
        st.error(f"Failed to capture voice input: {voice_query}")

if __name__ == "__main__":
    main()
````

---

## Example 6: Error Handling & Recovery

```python
"""
Comprehensive error handling for voice input scenarios
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor
import logging

logger = logging.getLogger(__name__)

def voice_input_with_fallback():
    """Voice input with error recovery mechanisms"""

    st.title("🎤 Voice Input with Error Recovery")

    # Attempt voice input
    success, result = InputProcessor.get_input_from_user("Voice")

    if success:
        st.success(f"✅ Successfully captured: {result}")
        return result

    else:
        # Handle specific error messages
        st.warning(f"⚠️ Voice input failed: {result}")

        # Error-specific guidance
        if "Streamlit version" in result:
            st.error("Streamlit version too old")
            st.code("pip install streamlit --upgrade")
            st.info("Please upgrade Streamlit to 1.35.0 or later")

        elif "Please record" in result:
            st.warning("No audio was recorded")
            st.info("""
            **Troubleshooting:**
            1. Ensure your microphone is plugged in
            2. Check browser permissions (camera icon in address bar)
            3. Try the file upload tab instead
            4. Try a different browser (Chrome recommended)
            """)

        elif "transcription" in result.lower():
            st.error("Transcription service failed")
            st.info("""
            **Solutions:**
            1. Check your internet connection
            2. Try the alternative transcription service
            3. Ensure OpenAI API key is set (if using Whisper)
            4. Try uploading a different audio file
            """)

        else:
            st.error(f"Unknown error: {result}")
            st.info("Please check the application logs for details")

        # Fallback option
        st.markdown("---")
        st.subheader("🔄 Fallback Options")

        fallback_choice = st.radio(
            "How would you like to proceed?",
            [
                "Try voice input again",
                "Switch to text input",
                "Switch to image (OCR) input",
                "Exit and check setup"
            ]
        )

        if fallback_choice == "Try voice input again":
            st.info("Please try recording again...")
            return voice_input_with_fallback()

        elif fallback_choice == "Switch to text input":
            st.info("Switching to text input...")
            return InputProcessor.get_input_from_user("Text")

        elif fallback_choice == "Switch to image (OCR) input":
            st.info("Switching to image input...")
            return InputProcessor.get_input_from_user("Image")

        else:
            st.warning("Please check your setup and try again later")
            return None

if __name__ == "__main__":
    voice_input_with_fallback()
```

---

## Example 7: Advanced - Real-time Transcription Status

````python
"""
Advanced: Show transcription progress and status
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor
import time

def voice_input_with_status():
    """Voice input with real-time status updates"""

    st.title("🎙️ Advanced Voice Input")

    # Recording section
    st.header("Step 1: Record Voice")
    audio_data = st.audio_input("Click microphone to record:")

    if audio_data:
        st.success("✅ Audio captured!")

        # Audio info
        col1, col2, col3, col4 = st.columns(4)
        with col1:
            st.metric("Status", "✅ Ready")
        with col2:
            st.metric("Size", f"{len(audio_data)} bytes")
        with col3:
            st.metric("Format", "WAV")
        with col4:
            st.metric("Duration", "Est. ~30s")

        # Transcription method selection
        st.header("Step 2: Select Transcription Service")

        service = st.radio(
            "Choose service:",
            ["Google Speech Recognition (Free)", "OpenAI Whisper (Premium)"],
            horizontal=True
        )

        # Transcribe
        if st.button("🔄 Transcribe", use_container_width=True):
            # Show progress
            with st.spinner("Processing audio..."):
                progress_bar = st.progress(0)
                status_text = st.empty()

                # Simulated progress
                for i in range(100):
                    progress_bar.progress(i + 1)
                    time.sleep(0.01)

                    if i < 30:
                        status_text.text("🔄 Loading audio...")
                    elif i < 70:
                        status_text.text("🎵 Analyzing audio...")
                    else:
                        status_text.text("✨ Transcribing...")

                # Actual transcription
                if service == "Google Speech Recognition (Free)":
                    success, text = InputProcessor.process_audio_input(audio_data)
                else:
                    success, text = InputProcessor.process_audio_with_whisper(audio_data)

            # Results
            st.header("Step 3: Results")

            if success:
                st.success("✅ Transcription complete!")
                st.markdown(f"**Transcribed Text:**\n```\n{text}\n```")

                # Statistics
                st.markdown("---")
                st.subheader("📊 Statistics")
                col1, col2, col3 = st.columns(3)
                with col1:
                    st.metric("Words", len(text.split()))
                with col2:
                    st.metric("Characters", len(text))
                with col3:
                    st.metric("Service", service.split()[0])

            else:
                st.error(f"❌ Transcription failed: {text}")

if __name__ == "__main__":
    voice_input_with_status()
````

---

## Testing the Microphone Feature

### Quick Test Script

```python
"""
test_microphone.py - Quick verification of microphone feature
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor

st.title("🧪 Microphone Feature Test")

st.markdown("""
## Test Checklist
- [ ] Select "Voice" input mode
- [ ] Click "🎙️ Record Voice" tab
- [ ] Click microphone icon
- [ ] Record a test message (5-10 seconds)
- [ ] Click stop
- [ ] Verify audio playback
- [ ] Verify transcription (Google)
- [ ] Try Whisper transcription (if API key set)
- [ ] Test file upload tab
- [ ] Test both transcription services
""")

st.markdown("---")

# Run test
success, result = InputProcessor.get_input_from_user("Voice")

st.markdown("---")
st.subheader("Test Results")

if success:
    st.success("✅ PASS: Voice input working")
    st.write(f"**Extracted Text:** {result}")
else:
    st.error("❌ FAIL: Voice input failed")
    st.write(f"**Error:** {result}")
```

---

## Deployment Example

```python
"""
Production deployment with microphone feature
"""

import streamlit as st
from src.input.Input_Processor import InputProcessor
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

def main():
    # Page configuration
    st.set_page_config(
        page_title="NLP2SQL",
        page_icon="🗣️",
        layout="wide",
        initial_sidebar_state="expanded"
    )

    # Title and description
    st.title("NLP2SQL: Natural Language to Database Queries")
    st.markdown("""
    Convert natural language (text, images, or voice) into SQL queries.

    **Features:**
    - 🎤 Live microphone recording
    - 🎙️ Audio file upload
    - 🖼️ Image OCR processing
    - 📝 Direct text input
    """)

    # Input selection
    st.sidebar.header("Input Method")
    input_method = st.sidebar.radio(
        "Choose how to provide your query:",
        ["🎤 Voice Recording", "📝 Text Input", "🖼️ Image (OCR)"],
        captions=[
            "Record using your microphone",
            "Type your query",
            "Upload image with text"
        ]
    )

    # Process input
    if input_method == "🎤 Voice Recording":
        success, query = InputProcessor.get_input_from_user("Voice")
    elif input_method == "📝 Text Input":
        success, query = InputProcessor.get_input_from_user("Text")
    else:
        success, query = InputProcessor.get_input_from_user("Image")

    # Display results
    if success:
        st.success("✅ Query captured successfully!")
        st.info(f"**Your Query:** {query}")

        # SQL generation would happen here
        st.markdown("---")
        st.subheader("Next Steps")
        if st.button("Generate SQL", use_container_width=True):
            st.info("SQL generation in progress...")
    else:
        st.error(f"Error: {query}")

if __name__ == "__main__":
    main()
```

---

## Summary

These examples demonstrate:

1. ✅ Basic microphone usage
2. ✅ Error handling and recovery
3. ✅ Service comparison
4. ✅ Full application integration
5. ✅ Advanced features (progress, status)
6. ✅ Testing and validation
7. ✅ Production deployment

All examples are ready to copy and use!
