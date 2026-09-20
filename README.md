AI Video & Meeting Assistant
An AI-powered meeting intelligence application that converts YouTube
videos or local audio/video files into searchable meeting knowledge.

The system processes audio through speech-to-text, generates a
professional meeting summary, extracts action items, key decisions, and
unresolved questions, and provides a RAG-based conversational
interface for asking questions about the meeting transcript.

Built with Python, Whisper, Sarvam AI, LangChain, Mistral AI,
ChromaDB, Sentence Transformers, FFmpeg/Pydub, yt-dlp, and Streamlit.

Overview
The AI Video & Meeting Assistant is designed to turn long meeting
recordings into structured, searchable information.

Input
YouTube URL

Local audio file

Local video file

Output
Meeting title

Full transcript

Bullet-point meeting summary

Action items with owner and deadline when available

Key decisions

Unresolved/open questions

Conversational Q&A over the transcript using Retrieval-Augmented
Generation (RAG)

Key Features
1. YouTube & Local File Processing
The application accepts either a YouTube URL or a local media file.

For YouTube URLs:

YouTube URL
    ↓
yt-dlp
    ↓
Audio extraction
    ↓
WAV
For local files:

Audio / Video File
       ↓
     Pydub
       ↓
16 kHz Mono WAV
Audio is then divided into 10-minute chunks for downstream
transcription.

2. Multilingual Speech-to-Text
The application supports two transcription modes:

Language Mode Engine Processing

English OpenAI Whisper Local transcription
Hinglish Sarvam AI Speech-to-text with English translation

English
Whisper runs locally using the configured Whisper model.

Default model:

small
Hinglish
Sarvam AI's speech-to-text-translate endpoint is used.

Because the synchronous Sarvam API accepts audio of up to 30 seconds,
each chunk is further divided into 25-second pieces before being
sent to the API.

10-minute audio chunk
        ↓
25-second pieces
        ↓
Sarvam AI
        ↓
English transcript
        ↓
Combined transcript
3. Automatic Meeting Title Generation
The transcript is passed to Mistral AI to generate a short professional
meeting title.

The title-generation prompt limits the result to a maximum of 8
words.

4. Hierarchical Meeting Summarization
Long transcripts are processed using a map-and-combine summarization
strategy.

Full Transcript
      ↓
RecursiveCharacterTextSplitter
      ↓
Multiple transcript chunks
      ↓
Individual chunk summaries
      ↓
Combined summaries
      ↓
Mistral AI
      ↓
Final bullet-point meeting summary
The summarizer uses:

Chunk size: 3000
Chunk overlap: 200
This allows longer meeting transcripts to be summarized in stages
instead of sending the entire transcript to the LLM in a single request.

5. Action Item Extraction
The system extracts actionable tasks from the meeting transcript.

For each detected action item, it attempts to identify:

Task description

Owner

Deadline

If a deadline is not mentioned, the system returns:

Not specified
6. Key Decision Extraction
The application identifies important decisions made during the meeting
and presents them as a numbered list.

7. Open Question Extraction
The system identifies:

Unresolved questions

Topics requiring follow-up

Issues that remain unanswered

This helps convert an unstructured meeting transcript into follow-up
items.

Retrieval-Augmented Generation (RAG)
The project implements a complete RAG pipeline for conversational
meeting analysis.

RAG Pipeline
Meeting Transcript
        ↓
Recursive Character Text Splitter
        ↓
500-character chunks
        ↓
Sentence Transformer Embeddings
        ↓
ChromaDB
        ↓
Similarity Retriever
        ↓
Top 4 relevant chunks
        ↓
Mistral AI
        ↓
Context-grounded Answer
Text Chunking
The RAG pipeline uses:

Chunk size: 500
Chunk overlap: 50
Each chunk is stored as a LangChain Document with its chunk index as
metadata.

Embeddings
The project uses:

all-MiniLM-L6-v2
through Hugging Face Sentence Transformers.

Vector Database
The embeddings are stored locally using:

ChromaDB
Vector database directory:

vector_db/
Retrieval
The retriever uses similarity search with:

k = 4
meaning the four most relevant transcript chunks are provided to the LLM
as context.

Grounded Responses
The RAG prompt explicitly instructs the model to answer only from the
retrieved meeting transcript context.

If the information cannot be found, the assistant responds:

I could not find this information in the meeting transcript.
This reduces unsupported answers and keeps responses grounded in the
meeting content.

AI / LLM Architecture
The application uses Mistral AI through LangChain.

Mistral is used for:

Meeting title generation

Meeting summarization

Action-item extraction

Decision extraction

Open-question extraction

RAG question answering

The project uses LangChain LCEL to construct reusable LLM pipelines.

Example architecture:

Input
  ↓
RunnablePassthrough
  ↓
Prompt Template
  ↓
ChatMistralAI
  ↓
StrOutputParser
  ↓
Generated Output
End-to-End Pipeline
                   ┌─────────────────────┐
                   │ YouTube URL / File  │
                   └──────────┬──────────┘
                              ↓
                     Audio Extraction
                     yt-dlp / Pydub
                              ↓
                       WAV Conversion
                              ↓
                       Audio Chunking
                              ↓
                    ┌─────────┴─────────┐
                    ↓                   ↓
              English Mode        Hinglish Mode
                    ↓                   ↓
                Whisper            Sarvam AI
                    └─────────┬─────────┘
                              ↓
                         Transcript
                              ↓
                   ┌──────────┴──────────┐
                   ↓                     ↓
              Summarization        Information
                   ↓                 Extraction
                   ↓            ┌──────┼──────┐
              Mistral AI         ↓      ↓      ↓
                   ↓          Actions Decisions Questions
                   ↓
              Final Summary
                              ↓
                         RAG Pipeline
                              ↓
                 Sentence Transformers
                              ↓
                          ChromaDB
                              ↓
                       Similarity Search
                              ↓
                         Mistral AI
                              ↓
                      Meeting Q&A Chat
Tech Stack
Category Technologies

Language Python
UI Streamlit
Speech-to-Text OpenAI Whisper, Sarvam AI
LLM Mistral AI
LLM Framework LangChain / LCEL
RAG LangChain, ChromaDB
Embeddings Sentence Transformers
Embedding Model all-MiniLM-L6-v2
Audio Processing Pydub, FFmpeg
YouTube Processing yt-dlp
API Requests Requests
Configuration python-dotenv
ML Runtime PyTorch

Project Structure
AI-Video-Assistant/
│
├── app.py
├── main.py
├── test.py
├── Requirements.txt
├── .gitignore
│
├── core/
│   ├── extractor.py
│   ├── rag_engine.py
│   ├── summarizer.py
│   ├── transcriber.py
│   └── vector_store.py
│
└── utils/
    └── audio_processor.py
Module Breakdown
app.py
Streamlit application responsible for:

User input

Language selection

Pipeline execution

Pipeline status visualization

Summary display

Transcript display

Action items

Key decisions

Open questions

Interactive RAG chat

Chat history management

main.py
Provides the command-line pipeline.

The main pipeline:

run_pipeline(source, language)
returns:

title
transcript
summary
action_items
key_decisions
open_questions
rag_chain
It also supports interactive terminal-based meeting Q&A.

core/transcriber.py
Handles speech-to-text.

Responsibilities:

Load Whisper model

Transcribe English audio locally

Send Hinglish audio to Sarvam AI

Split Sarvam requests into API-compatible pieces

Combine transcript chunks

core/summarizer.py
Handles:

Transcript splitting

Chunk-level summarization

Combined final summarization

Meeting title generation

core/extractor.py
Contains separate LLM chains for:

Action Items
Key Decisions
Open Questions
core/vector_store.py
Responsible for:

Transcript chunking

Embedding generation

ChromaDB creation

ChromaDB loading

Retriever configuration

core/rag_engine.py
Builds and executes the RAG pipeline.

Responsibilities:

Build vector store

Create similarity retriever

Retrieve relevant transcript context

Construct grounded prompts

Generate answers with Mistral

utils/audio_processor.py
Handles:

YouTube audio download

Audio/video conversion

Mono conversion

16 kHz resampling

10-minute chunking

Installation
Prerequisites
Install:

Python 3.10+

FFmpeg

Git

You also need API credentials for:

Mistral AI

Sarvam AI (required for Hinglish mode)

Whisper runs locally.

1. Clone the Repository
git clone <your-repository-url>
cd AI-Video-Assistant
2. Create a Virtual Environment
Windows
python -m venv venv
venv\Scripts\activate
macOS / Linux
python3 -m venv venv
source venv/bin/activate
3. Install Dependencies
pip install -r Requirements.txt
4. Install FFmpeg
FFmpeg must be installed separately because Pydub and yt-dlp rely on
FFmpeg for audio processing.

Verify:

ffmpeg -version
Environment Variables
Create a .env file in the project root:

MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key

WHISPER_MODEL=small
SARVAM_STT_MODEL=saaras:v2.5
Environment Variable Reference
Variable Purpose

MISTRAL_API_KEY Mistral AI API authentication
SARVAM_API_KEY Sarvam speech-to-text authentication
WHISPER_MODEL Local Whisper model; defaults to small
SARVAM_STT_MODEL Sarvam model; defaults to saaras:v2.5

Never commit .env or API keys to GitHub.

Running the Application
Streamlit UI
Run:

streamlit run app.py
Then open the Streamlit URL shown in the terminal.

The interface allows you to:

Enter a YouTube URL or local file path.

Select english or hinglish.

Click Analyse.

Wait for the processing pipeline to complete.

Review the generated meeting intelligence.

Ask questions about the meeting using the RAG chat.

CLI Mode
Run:

python main.py
Enter:

YouTube URL or local file path:
Then choose:

english
or:

hinglish
After processing, the CLI displays:

Meeting title

Summary

Action items

Key decisions

Open questions

You can then enter questions and chat with the meeting transcript.

Type:

exit
to leave the chat.

Example Workflow
Input
https://www.youtube.com/watch?v=...
Language:

english
Processing
Download Audio
      ↓
Convert / Normalize Audio
      ↓
10-Minute Chunking
      ↓
Whisper Transcription
      ↓
Meeting Title
      ↓
Hierarchical Summary
      ↓
Action Items
      ↓
Key Decisions
      ↓
Open Questions
      ↓
ChromaDB Vector Store
      ↓
RAG Chat
Example RAG Questions
You can ask questions such as:

What were the main decisions made?
Who was assigned the database migration?
What deadlines were discussed?
What issues are still unresolved?
What did the team decide about the deployment?
The RAG system retrieves relevant transcript chunks before generating
the answer.

Evaluation Results
The following results are the project evaluation metrics reported in
the accompanying resume:

Metric Reported Result

Word Error Rate (WER) 10%
Retrieval Accuracy 90%
RAG Answer Accuracy 92%

These are reported project evaluation results rather than metrics
calculated by the current repository at runtime.

Resume-Aligned Project Description
The project is represented on the resume as:

AI Video & Meeting Assistant | Python, Whisper, Sarvam AI,
LangChain, Mistral, ChromaDB, Sentence Transformers, Streamlit,
FFmpeg, Pydub, yt-dlp

The resume describes the system as a meeting assistant that accepts
YouTube URLs and audio/video files, uses Whisper for English
transcription and Sarvam AI for Hindi/Hinglish speech-to-text, generates
meeting summaries and structured meeting intelligence, and provides
RAG-based meeting Q&A using Sentence Transformer embeddings and
ChromaDB.

Engineering Highlights
This project demonstrates practical experience with:

Speech-to-text systems

Local AI model inference

Multilingual transcription

LLM application development

LangChain LCEL

Prompt engineering

Retrieval-Augmented Generation

Vector databases

Semantic search

Sentence Transformer embeddings

Audio preprocessing

YouTube media extraction

Chunk-based processing

Meeting intelligence

Streamlit application development

Context-grounded question answering

Design Decisions
Why audio chunking?
Long recordings can exceed model/API input limits. The application
therefore converts the source into WAV audio and processes it in
manageable chunks.

Why different transcription engines?
The application uses local Whisper for English and Sarvam AI for
Hinglish so that the selected language mode can use a suitable
transcription/translation pipeline.

Why ChromaDB?
ChromaDB provides local persistent vector storage for transcript
embeddings and enables similarity-based retrieval for meeting Q&A.

Why RAG?
Instead of asking the LLM to answer from the entire transcript directly,
the system retrieves the most relevant transcript sections and supplies
them as context. This helps keep answers grounded in the meeting
content.

Why map-and-combine summarization?
Large transcripts are split into smaller sections, summarized
independently, and then combined into a final summary. This provides a
practical way to process longer meetings.

Current Repository Notes
The current codebase contains PDF/TXT-related packages in
Requirements.txt, but the provided app.py and core pipeline do not
currently implement PDF/TXT export functionality. Therefore, this
README does not claim export as a currently implemented feature.

The repository also contains generated Python __pycache__ files. These
should generally remain excluded from version control through
.gitignore.

Future Improvements
Potential extensions include:

PDF/TXT report export

Speaker diarization

Timestamped transcript segments

Automatic meeting chaptering

Persistent multi-meeting knowledge base

Cross-meeting semantic search

Better citation of retrieved transcript chunks

Streaming transcription

Background processing for long recordings

GPU acceleration for Whisper

Authentication and multi-user workspaces

Meeting analytics dashboard

Action-item tracking

Calendar/task integration

Automated evaluation pipeline for WER, retrieval precision, and RAG
answer quality

Deployment using Docker and cloud infrastructure

Author
Sayyad Yusuff Basha

B.Tech Mechanical & Aerospace Engineering
IIT Hyderabad

Project Focus
Generative AI · RAG · NLP · Speech-to-Text · LLM Applications · Python
· AI Engineering
