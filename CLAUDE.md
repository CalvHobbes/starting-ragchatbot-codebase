# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Dependencies Management
- **Install dependencies**: `uv sync`
- **Add new dependency**: `uv add <package-name>`

### Running the Application
- **Quick start**: `./run.sh` (uses port 8000)
- **Manual start**: `cd backend && uv run uvicorn app:app --reload --port 8000`
- **Web interface**: http://localhost:8000
- **API docs**: http://localhost:8000/docs

### Environment Setup
- Create `.env` file with: `ANTHROPIC_API_KEY=your_key_here`
- Requires Python 3.13+ and uv package manager

### Development Notes
- **No test framework configured** - no pytest, unittest, or testing commands available
- **No linting tools configured** - no flake8, black, ruff, or similar tools setup
- **Document processing**: Place course materials (PDF/DOCX/TXT) in `docs/` folder for automatic processing on startup

## Architecture Overview

This is a **RAG (Retrieval-Augmented Generation) system** for querying course materials with the following architecture:

### Core Components
- **FastAPI Backend** (`backend/app.py`): REST API with CORS enabled, serves both API and frontend
- **RAG System** (`backend/rag_system.py`): Main orchestrator coordinating all components
- **Vector Store** (`backend/vector_store.py`): ChromaDB-based semantic search with course metadata
- **AI Generator** (`backend/ai_generator.py`): Anthropic Claude integration with tool support
- **Document Processor** (`backend/document_processor.py`): Processes PDF/DOCX/TXT into chunks
- **Session Manager** (`backend/session_manager.py`): Conversation history management
- **Search Tools** (`backend/search_tools.py`): Tool-based search system for AI agent

### Data Flow
1. Documents in `docs/` folder are processed into course chunks on startup
2. Content is embedded and stored in ChromaDB (`backend/chroma_db/`)
3. User queries trigger tool-based semantic search via AI agent
4. Claude generates responses using retrieved context and conversation history

### Key Patterns
- **Tool-based Search**: AI uses CourseSearchTool instead of direct retrieval
- **Session-based Conversations**: Each user gets a session for context retention (max 2 messages history)
- **Chunked Processing**: Documents split into 800-char chunks with 100-char overlap
- **Course Metadata**: Separate storage of course titles and descriptions for better search
- **Dual Collections**: ChromaDB uses separate collections for course metadata vs content chunks
- **Smart Course Matching**: Search tool supports partial course name matching (e.g. 'MCP' matches full title)
- **Lesson Filtering**: Can search within specific lesson numbers when provided

### Frontend
- Simple HTML/CSS/JS interface (`frontend/`) served as static files
- Real-time query processing with session management
- Course statistics display

### Configuration
- All settings centralized in `backend/config.py`
- ChromaDB persistence in `backend/chroma_db/`
- Sentence transformers for embeddings (default: all-MiniLM-L6-v2)
- Claude model: claude-sonnet-4-20250514
- Max search results: 5, Max tokens: 800, Temperature: 0

### Data Models
- **Course**: Contains title, lessons list, instructor, course link
- **Lesson**: Has lesson number, title, optional lesson link
- **CourseChunk**: Text content with course title, lesson number, chunk index
- **SearchResults**: Wrapper for ChromaDB results with documents, metadata, distances

### API Endpoints
- `POST /api/query`: Main chat endpoint (QueryRequest → QueryResponse)
- `GET /api/courses`: Get course statistics and available courses
- `GET /`: Serves frontend HTML interface
- `GET /docs`: FastAPI auto-generated API documentation