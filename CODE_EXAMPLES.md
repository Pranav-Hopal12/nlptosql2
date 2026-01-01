# Code Examples - Multi-Input NLP2SQL

## Using the InputProcessor Module Directly

### Example 1: Process Text Input

```python
from src.input.Input_Processor import InputProcessor

# Direct text processing
success, message = InputProcessor.process_text_input("Show all customers from NYC")
if success:
    print(f"Query: {message}")
else:
    print(f"Error: {message}")
```

### Example 2: Process Image Input

```python
from src.input.Input_Processor import InputProcessor
from PIL import Image

# Load and process image
image = Image.open("query.png")
success, extracted_text = InputProcessor.process_image_input(image)

if success:
    print(f"Extracted text: {extracted_text}")
    # Use extracted_text as a query
else:
    print(f"Error: {extracted_text}")
```

### Example 3: Process Audio with Google Speech Recognition

```python
from src.input.Input_Processor import InputProcessor

# Load audio file
with open("query.wav", "rb") as audio_file:
    success, transcribed_text = InputProcessor.process_audio_input(audio_file)

if success:
    print(f"Transcribed: {transcribed_text}")
else:
    print(f"Error: {transcribed_text}")
```

### Example 4: Process Audio with Whisper

```python
from src.input.Input_Processor import InputProcessor

# Load audio file
with open("query.mp3", "rb") as audio_file:
    success, transcribed_text = InputProcessor.process_audio_with_whisper(audio_file)

if success:
    print(f"Transcribed (Whisper): {transcribed_text}")
else:
    print(f"Error: {transcribed_text}")
```

### Example 5: Use the Unified Interface

```python
from src.input.Input_Processor import InputProcessor

# This is what the app uses internally
input_type = "Text"  # or "Image (OCR)" or "Voice"
success, user_query = InputProcessor.get_input_from_user(input_type)

if success:
    print(f"Ready to process: {user_query}")
else:
    print(f"Input failed: {user_query}")
```

---

## Integration with SQL Generation

### Example 6: Full Pipeline - Text to SQL

```python
from src.input.Input_Processor import InputProcessor
from src.api.LLM_Config import get_completion_from_messages
from src.prompts.Base_Prompt import SYSTEM_MESSAGE
import src.database.DB_Config as DB_Config

# Step 1: Get user input (any mode)
success, user_query = InputProcessor.get_input_from_user("Text")

if not success:
    print(f"Error: {user_query}")
    exit()

# Step 2: Get database schemas
schemas = DB_Config.get_all_schemas("mydb.db", db_type='sqlite')

# Step 3: Generate SQL
formatted_system_message = f"{SYSTEM_MESSAGE}\n\nSchemas: {schemas}"
response = get_completion_from_messages(formatted_system_message, user_query)

print(f"Generated SQL: {response['query']}")

# Step 4: Execute query
results = DB_Config.query_database(response['query'], "mydb.db", 'sqlite')
print(results)
```

### Example 7: Full Pipeline - Image to SQL

```python
from src.input.Input_Processor import InputProcessor
from PIL import Image
import json

# Step 1: Extract text from image
image = Image.open("screenshot.png")
success, query_text = InputProcessor.process_image_input(image)

if not success:
    print(f"OCR Error: {query_text}")
    exit()

print(f"Extracted from image: {query_text}")

# Step 2-4: Same as Example 6 above
# ... rest of SQL generation and execution
```

---

## Error Handling Examples

### Example 8: Comprehensive Error Handling

```python
from src.input.Input_Processor import InputProcessor
import logging

logger = logging.getLogger(__name__)

def process_user_input_safe(input_type: str, user_file=None) -> str:
    """Process input with comprehensive error handling."""
    try:
        # Try to process input
        success, result = InputProcessor.get_input_from_user(input_type)

        if not success:
            logger.error(f"Input processing failed: {result}")
            return None

        if not result or not result.strip():
            logger.warning("Empty input received after processing")
            return None

        logger.info(f"Successfully processed {input_type} input")
        return result

    except ImportError as e:
        logger.error(f"Missing required library: {e}")
        return None
    except Exception as e:
        logger.exception(f"Unexpected error during input processing: {e}")
        return None

# Usage
query = process_user_input_safe("Text")
if query:
    print(f"Query ready: {query}")
```

---

## Streamlit Integration Examples

### Example 9: Custom Input Mode Selector

```python
import streamlit as st
from src.input.Input_Processor import InputProcessor

# Create two-column layout
col1, col2 = st.columns([1, 3])

with col1:
    input_mode = st.radio(
        "Input Mode",
        ["Text", "Image (OCR)", "Voice"],
        help="Choose your input method"
    )

with col2:
    st.write(f"Selected: **{input_mode}**")

    # Get input based on mode
    success, user_query = InputProcessor.get_input_from_user(input_mode)

    if success:
        st.success(f"✅ Input received")
        st.code(user_query, language="sql")
    else:
        st.error(f"❌ {user_query}")
```

### Example 10: Processing Multiple Inputs

```python
import streamlit as st
from src.input.Input_Processor import InputProcessor

# Tab interface for different input methods
tab1, tab2, tab3 = st.tabs(["Text", "Image (OCR)", "Voice"])

queries = []

with tab1:
    success, text_query = InputProcessor.process_text_input(
        st.text_input("Enter your query:")
    )
    if success:
        queries.append(text_query)

with tab2:
    image_file = st.file_uploader("Upload image", type=["jpg", "png"])
    if image_file:
        success, ocr_query = InputProcessor.process_image_input(image_file)
        if success:
            queries.append(ocr_query)

with tab3:
    audio_file = st.file_uploader("Upload audio", type=["wav", "mp3"])
    if audio_file:
        success, voice_query = InputProcessor.process_audio_input(audio_file)
        if success:
            queries.append(voice_query)

# Display all collected queries
if queries:
    st.write("Collected Queries:")
    for i, q in enumerate(queries, 1):
        st.write(f"{i}. {q}")
```

---

## Testing Examples

### Example 11: Unit Test for Input Processor

```python
import unittest
from src.input.Input_Processor import InputProcessor
from PIL import Image
import io

class TestInputProcessor(unittest.TestCase):

    def test_text_input_valid(self):
        """Test valid text input"""
        success, result = InputProcessor.process_text_input("Test query")
        self.assertTrue(success)
        self.assertEqual(result, "Test query")

    def test_text_input_empty(self):
        """Test empty text input"""
        success, result = InputProcessor.process_text_input("")
        self.assertFalse(success)

    def test_text_input_whitespace(self):
        """Test whitespace-only input"""
        success, result = InputProcessor.process_text_input("   ")
        self.assertFalse(success)

    def test_image_input_none(self):
        """Test None image input"""
        success, result = InputProcessor.process_image_input(None)
        self.assertFalse(success)

if __name__ == '__main__':
    unittest.main()
```

---

## Advanced Usage Examples

### Example 12: Custom Input Processor Extension

```python
from src.input.Input_Processor import InputProcessor
import logging

class CustomInputProcessor(InputProcessor):
    """Extended input processor with additional validation."""

    @staticmethod
    def validate_sql_keywords(text: str) -> bool:
        """Check if text contains expected SQL-like keywords."""
        sql_keywords = ['select', 'from', 'where', 'join', 'show', 'list', 'find']
        text_lower = text.lower()
        return any(keyword in text_lower for keyword in sql_keywords)

    @staticmethod
    def process_text_input_with_validation(text: str):
        """Process text with SQL keyword validation."""
        success, message = InputProcessor.process_text_input(text)

        if not success:
            return False, message

        if not CustomInputProcessor.validate_sql_keywords(message):
            logging.warning("Input may not be a valid query (no SQL keywords detected)")

        return True, message

# Usage
success, query = CustomInputProcessor.process_text_input_with_validation(
    "Show me all customers"
)
```

### Example 13: Batch Processing Multiple Files

```python
from src.input.Input_Processor import InputProcessor
from pathlib import Path
import json

def batch_process_images(image_directory: str) -> dict:
    """Process all images in a directory."""
    results = {}
    image_path = Path(image_directory)

    for image_file in image_path.glob("*.png") + image_path.glob("*.jpg"):
        try:
            with open(image_file, 'rb') as f:
                success, text = InputProcessor.process_image_input(f)
                results[str(image_file)] = {
                    "success": success,
                    "text": text
                }
        except Exception as e:
            results[str(image_file)] = {
                "success": False,
                "text": str(e)
            }

    return results

# Usage
batch_results = batch_process_images("./query_images/")
print(json.dumps(batch_results, indent=2))
```

---

## Configuration Examples

### Example 14: Environment Configuration

```python
import os
from dotenv import load_dotenv
from src.input.Input_Processor import InputProcessor

# Load environment variables
load_dotenv()

# Check if Whisper is available
openai_key = os.getenv("OPENAI_API_KEY")

if openai_key:
    print("✅ Whisper API available")
    # Use Whisper for voice input
    audio_file = open("query.mp3", "rb")
    success, text = InputProcessor.process_audio_with_whisper(audio_file)
else:
    print("ℹ️ Using Google Speech Recognition (free)")
    # Fall back to Google
    audio_file = open("query.mp3", "rb")
    success, text = InputProcessor.process_audio_input(audio_file)
```

---

## Performance Monitoring Examples

### Example 15: Input Processing Performance

```python
import time
from src.input.Input_Processor import InputProcessor

def measure_input_processing_time(input_type: str):
    """Measure how long input processing takes."""
    start_time = time.time()

    success, result = InputProcessor.get_input_from_user(input_type)

    processing_time = time.time() - start_time

    return {
        "input_type": input_type,
        "success": success,
        "processing_time": processing_time,
        "result_length": len(result) if success else 0
    }

# Benchmark different input types
benchmarks = {
    "Text": measure_input_processing_time("Text"),
    "Image": measure_input_processing_time("Image (OCR)"),
    "Voice": measure_input_processing_time("Voice")
}

for input_type, metrics in benchmarks.items():
    print(f"{input_type}: {metrics['processing_time']:.2f}s")
```

---

## Notes

- All examples assume proper installation of dependencies
- Error handling should always be implemented in production
- Tesseract must be installed for OCR examples
- OPENAI_API_KEY required for Whisper examples
- Logging is important for debugging

For more details, see:

- `MULTI_INPUT_GUIDE.md` - Complete user guide
- `IMPLEMENTATION_SUMMARY.md` - Technical details
- `src/input/Input_Processor.py` - Source code
