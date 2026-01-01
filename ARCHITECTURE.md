# Architecture Diagram - NLP2SQL Multi-Input System

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         STREAMLIT APPLICATION                        │
│                      app/NLP2SQL.py (Main App)                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │ SIDEBAR CONTROLS              │
                    │                               │
                    ├─ Database Type Selection      │
                    ├─ Connection Details           │
                    └─ INPUT MODE SELECTOR (NEW)    │
                       ├─ Text Input                │
                       ├─ Image (OCR)               │
                       └─ Voice Input               │
                                    │
                    ┌───────────────┼───────────────┐
                    │                               │
            ┌───────▼────────┐          ┌──────────▼──────────┐
            │ INPUT PROCESSOR│          │  DATABASE SETUP      │
            │ (NEW MODULE)   │          │                      │
            └────────────────┘          ├─ SQLite/PostgreSQL   │
                    │                   ├─ Table Selection     │
    ┌───────────────┼───────────────┐   │ ├─ Schema Loading    │
    │               │               │   └──────────────────────┘
    ▼               ▼               ▼
┌────────┐   ┌──────────┐    ┌──────────┐
│  TEXT  │   │  IMAGE   │    │  VOICE   │
│ INPUT  │   │  (OCR)   │    │  INPUT   │
└────────┘   └──────────┘    └──────────┘
    │            │                │
    │        Uses:            Uses:
    │    Pytesseract      ├─ Google Speech Recognition
    │    + Pillow         │  (Free)
    │                     │
    │                     └─ OpenAI Whisper
    │                        (Premium)
    │            │                │
    └────────────┼────────────────┘
                 │
                 ▼
        ┌────────────────────┐
        │ EXTRACTED TEXT     │
        │ (Unified Format)   │
        └────────────────────┘
                 │
                 ▼
        ┌────────────────────┐
        │  SQL GENERATION    │
        │ (LLM - OpenAI/     │
        │  Google)           │
        └────────────────────┘
                 │
                 ▼
        ┌────────────────────┐
        │  SQL VALIDATION    │
        │  & EXECUTION       │
        └────────────────────┘
                 │
                 ▼
        ┌────────────────────┐
        │ QUERY RESULTS      │
        │ (DataFrame)        │
        └────────────────────┘
                 │
      ┌──────────┼──────────┐
      │          │          │
      ▼          ▼          ▼
   ┌─────┐   ┌──────┐   ┌─────────┐
   │TABLE│   │CHARTS│   │ANALYSIS │
   └─────┘   └──────┘   └─────────┘
```

## Input Processor Module Architecture

```
┌─────────────────────────────────────────────┐
│  InputProcessor Class                       │
│  (src/input/Input_Processor.py)             │
└─────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
    ┌───────┐    ┌────────┐    ┌───────┐
    │ Text  │    │ Image  │    │ Voice │
    └───────┘    └────────┘    └───────┘
        │            │             │
        │        ┌───────────────────┐
        │        │   Audio Methods   │
        │        ├──────┬────────────┤
        │        │      │            │
        │        ▼      ▼            ▼
        │    ┌────┐  ┌────────┐  ┌───────┐
        │    │None│  │Google  │  │Whisper│
        │    └────┘  │Speech  │  │API    │
        │            │Recog.  │  │(OAI)  │
        │            └────────┘  └───────┘
        │
        └─────────────────────────┬──────────┐
                                  │          │
                        ┌─────────▼──────┐  │
                        │  Validation    │  │
                        │  & Processing  │  │
                        └────────────────┘  │
                                            │
                        ┌──────────────────▼─┐
                        │  Error Handling    │
                        │  & User Feedback   │
                        └────────────────────┘
                                  │
                        ┌─────────▼──────────┐
                        │ Return (success,   │
                        │ text_or_error)     │
                        └────────────────────┘
```

## Data Flow - Text Input Example

```
User Types Query
       │
       ▼
┌──────────────────────────────┐
│ st.text_input()              │
│ "Show customers from NYC"    │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ get_user_input_unified()     │
│ (NLP2SQL.py)                 │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ InputProcessor.                │
│ get_input_from_user("Text")  │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ process_text_input()         │
│ - Check if empty             │
│ - Validate format            │
│ - Return (True, text)        │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ user_message variable        │
│ = "Show customers from NYC"  │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ generate_sql_query()         │
│ (with LLM)                   │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ Query Execution & Results    │
└──────────────────────────────┘
```

## Data Flow - Image Input Example

```
User Uploads Image
       │
       ▼
┌──────────────────────────────┐
│ st.file_uploader()           │
│ query_screenshot.png         │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ get_user_input_unified()     │
│ (NLP2SQL.py)                 │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ InputProcessor.              │
│ get_input_from_user("Image") │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ process_image_input()        │
│ - Load with PIL              │
│ - Display preview            │
│ - Run Pytesseract            │
└──────────────────────────────┘
       │
       ├─ Check Tesseract installed
       │
       ├─ Extract text: "Show all sales"
       │
       └─ Return (True, extracted_text)
       │
       ▼
┌──────────────────────────────┐
│ user_message variable        │
│ = "Show all sales"           │
└──────────────────────────────┘
       │
       ▼
(Same as Text Input from here)
```

## Data Flow - Voice Input Example

```
User Uploads/Records Audio
       │
       ▼
┌──────────────────────────────┐
│ st.file_uploader() or        │
│ st.audio_input()             │
│ query.mp3                    │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ get_user_input_unified()     │
│ (NLP2SQL.py)                 │
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ InputProcessor.              │
│ get_input_from_user("Voice") │
└──────────────────────────────┘
       │
       ├─ Choose Method:
       │
       ├─ Google Speech Recognition
       │  │
       │  ├─ Load audio with sr.AudioFile
       │  ├─ Call recognizer.recognize_google()
       │  └─ Get transcription
       │
       └─ OpenAI Whisper
          │
          ├─ Check OPENAI_API_KEY
          ├─ Send to OpenAI API
          ├─ Get transcription
          └─ Return (True, transcribed_text)
       │
       ▼
┌──────────────────────────────┐
│ user_message variable        │
│ = "Show sales by region"     │
└──────────────────────────────┘
       │
       ▼
(Same as Text Input from here)
```

## Module Dependencies

```
app/
├── NLP2SQL.py (Main)
│   ├── imports: src.input.Input_Processor
│   ├── imports: src.database.DB_Config
│   ├── imports: src.api.LLM_Config
│   ├── imports: src.prompts.Base_Prompt
│   ├── imports: streamlit
│   ├── imports: pandas
│   └── imports: plotly
│
src/
├── input/
│   ├── __init__.py
│   └── Input_Processor.py
│       ├── imports: PIL (Pillow)
│       ├── imports: pytesseract
│       ├── imports: speech_recognition
│       ├── imports: streamlit
│       └── imports: logging
│
├── database/
│   └── DB_Config.py
│
├── api/
│   └── LLM_Config.py
│
└── prompts/
    └── Base_Prompt.py
```

## Class Hierarchy

```
┌──────────────────────────┐
│    InputProcessor        │
│  (Static Methods Only)   │
└──────────────────────────┘
         │
         ├─ @staticmethod
         │
         ├─ process_text_input()
         │  └─ Returns: (bool, str)
         │
         ├─ process_image_input()
         │  └─ Uses: Pytesseract, PIL
         │  └─ Returns: (bool, str)
         │
         ├─ process_audio_input()
         │  └─ Uses: SpeechRecognition
         │  └─ Returns: (bool, str)
         │
         ├─ process_audio_with_whisper()
         │  └─ Uses: OpenAI API
         │  └─ Returns: (bool, str)
         │
         └─ get_input_from_user()
            └─ Orchestrator method
            └─ Returns: (bool, str)
```

## Error Flow Diagram

```
User Input
    │
    ▼
Input Processor
    │
    ├─ Is input empty?
    │  ├─ Yes → Return (False, "Empty input")
    │  └─ No → Continue
    │
    ├─ Can file be processed?
    │  ├─ No → Return (False, "Format error")
    │  └─ Yes → Continue
    │
    ├─ Is required library installed?
    │  ├─ No → Return (False, "Install library X")
    │  └─ Yes → Continue
    │
    ├─ Can extraction succeed?
    │  ├─ No → Return (False, "Extraction error")
    │  └─ Yes → Continue
    │
    ├─ Is result valid?
    │  ├─ No → Return (False, "Invalid result")
    │  └─ Yes → Continue
    │
    └─ Return (True, extracted_text)
         │
         ▼
   Continue to SQL Generation
```

## Integration Points

```
Streamlit UI Layer
        │
        ├─ Sidebar: Input Mode Selection
        ├─ File Uploaders: Image/Audio
        ├─ Text Input: Direct Entry
        └─ Output: Results Display
        │
        ▼
Input Processor Layer (NEW)
        │
        ├─ Text Handler
        ├─ Image Handler (OCR)
        └─ Audio Handler (Voice)
        │
        ▼
Unified Text Output
        │
        ▼
LLM Processing Layer
        │
        ├─ Prompt Formation
        ├─ API Calls
        └─ SQL Generation
        │
        ▼
Database Layer
        │
        ├─ Query Validation
        ├─ Execution
        └─ Results
        │
        ▼
Visualization Layer
        │
        ├─ Tables
        ├─ Charts
        └─ Analysis
```

## Performance Layers

```
LAYER 1: Input (Fast)
├─ Text: <100ms
├─ Image: 2-10s
└─ Voice: 5-30s

LAYER 2: SQL Generation (Medium)
├─ API Call: 2-5s
├─ Validation: <100ms
└─ Total: 2-5s

LAYER 3: Database (Medium)
├─ Connection: <100ms
├─ Execution: 1-10s
└─ Total: 1-10s

LAYER 4: Visualization (Medium)
├─ Rendering: <500ms
├─ Analysis: 1-5s
└─ Total: 1-5s

TOTAL: ~5-50 seconds
(Depending on input type and data size)
```

## Security Architecture

```
User Input
    │
    ▼
┌─────────────────────┐
│ Validation Layer    │
├─ Type checking      │
├─ Format validation  │
└─ Size validation    │
    │
    ▼
┌─────────────────────┐
│ Processing Layer    │
├─ Local processing   │
│  (Images)           │
└─ Secure API calls   │
│  (Voice)            │
    │
    ▼
┌─────────────────────┐
│ API Security        │
├─ API key in .env    │
├─ HTTPS only         │
└─ No logging of data │
    │
    ▼
┌─────────────────────┐
│ SQL Injection       │
│ Prevention          │
├─ Input validation   │
└─ Parameterized      │
    queries            │
```

## File Structure Tree

```
NLP2SQL/
│
├── app/
│   └── NLP2SQL.py (modified)
│
├── src/
│   ├── input/ (NEW)
│   │   ├── __init__.py (NEW)
│   │   └── Input_Processor.py (NEW)
│   │
│   ├── database/
│   │   └── DB_Config.py
│   │
│   ├── api/
│   │   └── LLM_Config.py
│   │
│   └── prompts/
│       └── Base_Prompt.py
│
├── requirements.txt (modified)
│
├── Documentation/ (NEW)
│   ├── MULTI_INPUT_GUIDE.md (NEW)
│   ├── IMPLEMENTATION_SUMMARY.md (NEW)
│   ├── QUICK_START.md (NEW)
│   ├── CODE_EXAMPLES.md (NEW)
│   ├── CHANGELOG.md (NEW)
│   └── ARCHITECTURE.md (this file)
│
├── README.md
└── LICENSE
```

---

**Last Updated**: November 13, 2025
**System**: NLP2SQL Multi-Input Enhancement v2.0
