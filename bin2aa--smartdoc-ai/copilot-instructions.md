## smartdoc-ai

> Bạn là một AI Coding Assistant hỗ trợ phát triển hệ thống RAG local theo kiến trúc OOP chuẩn. Hãy tuân thủ nghiêm ngặt các công nghệ sau:

# Context: SmartDoc AI - Intelligent Document Q&A System
# Project Scope: OSSD Course - Spring 2026

## 1. Tech Stack & Environment
Bạn là một AI Coding Assistant hỗ trợ phát triển hệ thống RAG local theo kiến trúc OOP chuẩn. Hãy tuân thủ nghiêm ngặt các công nghệ sau:
* **Frontend**: Streamlit 1.41.1 (hoặc cao hơn)
* **Application Framework**: LangChain 0.3.16 & LangChain Community 0.3.16
* **LLM Engine**: Ollama chạy mô hình `qwen2.5:7b` (được tối ưu cho tiếng Việt)
* **Embedding Model**: HuggingFace `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` (768-dimensional, chạy trên CPU)
* **Vector Database**: FAISS 1.9.0
* **Document Loader**: PDFPlumber (dùng `PDFPlumberLoader`)
* **Python Version**: 3.8+

## 2. Software Architecture: Clean Architecture & MVC Pattern

### 2.1. Architecture Overview
Dự án tuân theo **Clean Architecture** với 4 layer độc lập:

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │  ← Views (Streamlit UI)
├─────────────────────────────────────────┤
│         Application Layer               │  ← Controllers (orchestration)
├─────────────────────────────────────────┤
│         Domain/Business Layer           │  ← Services (business logic)
├─────────────────────────────────────────┤
│         Infrastructure Layer            │  ← Models, Utils, External APIs
└─────────────────────────────────────────┘
```

**Dependency Rule**: Dependencies chỉ được hướng từ ngoài vào trong (Views → Controllers → Services → Models).

### 2.2. Layer Responsibilities

#### **Models Layer** (`src/models/`)
- **Trách nhiệm**: Định nghĩa data structures, domain entities, và business rules cơ bản
- **Không được**: Chứa logic UI, gọi external services, hoặc biết về LangChain
- **Ví dụ class**:
  ```python
  @dataclass
  class Document:
      """Domain entity representing a document"""
      id: str
      content: str
      metadata: Dict[str, Any]
      created_at: datetime
      
  @dataclass
  class ChatMessage:
      """Value object for chat messages"""
      role: Literal["user", "assistant", "system"]
      content: str
      timestamp: datetime
      metadata: Optional[Dict] = None
  ```

#### **Services Layer** (`src/services/`)
- **Trách nhiệm**: Business logic, RAG pipeline, vector store operations, LLM interactions
- **Design Pattern**: Repository Pattern, Strategy Pattern
- **Ví dụ interface**:
  ```python
  class AbstractDocumentService(ABC):
      @abstractmethod
      def load_document(self, file_path: str) -> List[Document]:
          pass
      
      @abstractmethod
      def chunk_document(self, document: Document, 
                         chunk_size: int, chunk_overlap: int) -> List[Document]:
          pass
  
  class AbstractVectorStoreService(ABC):
      @abstractmethod
      def add_documents(self, documents: List[Document]) -> None:
          pass
      
      @abstractmethod
      def similarity_search(self, query: str, k: int) -> List[Document]:
          pass
      
      @abstractmethod
      def clear_store(self) -> None:
          pass
  ```

#### **Controllers Layer** (`src/controllers/`)
- **Trách nhiệm**: Orchestration, input validation, error handling, logging
- **Pattern**: Facade Pattern, Command Pattern
- **Nguyên tắc**: Mỗi controller method phải:
  1. Validate input
  2. Log action
  3. Delegate to services
  4. Handle errors
  5. Return standardized response

#### **Views Layer** (`src/views/`)
- **Trách nhiệm**: Streamlit UI components, user interactions, state management
- **Pattern**: Observer Pattern (Streamlit session state), Component Pattern
- **Nguyên tắc**:
  - Components phải reusable và stateless (dùng session_state)
  - Không được chứa business logic
  - Chỉ gọi controllers, không gọi trực tiếp services

### 2.3. Folder Structure Chi Tiết
```
src/
├── models/
│   ├── __init__.py
│   ├── document_model.py      # Document, Chunk entities
│   ├── chat_model.py          # ChatMessage, ChatHistory
│   └── config_model.py        # AppConfig, RAGConfig
├── services/
│   ├── __init__.py
│   ├── document_service.py    # Document loading & chunking
│   ├── vector_store_service.py # FAISS operations
│   ├── llm_service.py         # Ollama LLM wrapper
│   ├── embedding_service.py   # HuggingFace embeddings
│   └── rag_service.py         # Complete RAG pipeline
├── controllers/
│   ├── __init__.py
│   ├── document_controller.py # Upload, process, manage docs
│   ├── chat_controller.py     # Handle queries, get answers
│   └── config_controller.py   # Manage settings
├── views/
│   ├── __init__.py
│   ├── document_screen.py     # Document upload UI
│   ├── chat_screen.py         # Chat interface
│   ├── settings_screen.py     # Configuration UI
│   └── components.py          # Reusable UI components
└── utils/
    ├── __init__.py
    ├── logger.py              # Logging configuration
    ├── validators.py          # Input validation
    ├── exceptions.py          # Custom exceptions
    └── constants.py           # App-wide constants
```

## 3. Design Patterns & Best Practices

### 3.1. Essential Design Patterns

#### **Repository Pattern** (Vector Store)
```python
class VectorStoreRepository(ABC):
    """Abstract repository for vector store operations"""
    
    @abstractmethod
    def save(self, embeddings: List[np.ndarray], 
             metadata: List[Dict]) -> None:
        pass
    
    @abstractmethod
    def search(self, query_embedding: np.ndarray, 
               k: int) -> List[Tuple[Document, float]]:
        pass

class FAISSRepository(VectorStoreRepository):
    """Concrete implementation using FAISS"""
    pass
```

#### **Factory Pattern** (Document Loaders)
```python
class DocumentLoaderFactory:
    """Factory to create appropriate loader based on file type"""
    
    @staticmethod
    def create_loader(file_path: str) -> AbstractDocumentLoader:
        extension = Path(file_path).suffix.lower()
        if extension == '.pdf':
            return PDFPlumberLoader()
        elif extension == '.docx':
            return DocxLoader()
        else:
            raise UnsupportedFileTypeError(f"Unsupported: {extension}")
```

#### **Strategy Pattern** (Text Splitting)
```python
class TextSplittingStrategy(ABC):
    @abstractmethod
    def split(self, text: str) -> List[str]:
        pass

class RecursiveCharacterSplitter(TextSplittingStrategy):
    def __init__(self, chunk_size: int = 1000, chunk_overlap: int = 100):
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
```

#### **Singleton Pattern** (Configuration)
```python
class ConfigManager:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialize()
        return cls._instance
    
    def _initialize(self):
        self.rag_config = RAGConfig(
            chunk_size=1000,
            chunk_overlap=100,
            temperature=0.7,
            top_p=0.9,
            repeat_penalty=1.1
        )
```

### 3.2. SOLID Principles Implementation

#### **S - Single Responsibility Principle**
Mỗi class chỉ có một lý do để thay đổi:
```python
# ✅ GOOD: Separation of concerns
class DocumentLoader:
    def load(self, path: str) -> str:
        pass

class DocumentChunker:
    def chunk(self, text: str) -> List[str]:
        pass

# ❌ BAD: Multiple responsibilities
class DocumentProcessor:
    def load_and_chunk(self, path: str) -> List[str]:
        pass
```

#### **O - Open/Closed Principle**
Mở cho mở rộng, đóng cho sửa đổi:
```python
# ✅ GOOD: Extensible through inheritance
class AbstractEmbeddingService(ABC):
    @abstractmethod
    def embed(self, texts: List[str]) -> List[np.ndarray]:
        pass

class HuggingFaceEmbedding(AbstractEmbeddingService):
    pass

class OpenAIEmbedding(AbstractEmbeddingService):  # Easy to add new
    pass
```

#### **L - Liskov Substitution Principle**
Subclass phải thay thế được base class:
```python
# ✅ GOOD: Consistent interface
class BaseVectorStore(ABC):
    @abstractmethod
    def add_vectors(self, vectors: np.ndarray) -> None:
        pass

class FAISSStore(BaseVectorStore):
    def add_vectors(self, vectors: np.ndarray) -> None:
        # Implementation that honors the contract
        pass
```

#### **I - Interface Segregation Principle**
Không ép client implement methods không dùng:
```python
# ✅ GOOD: Segregated interfaces
class Searchable(ABC):
    @abstractmethod
    def search(self, query: str) -> List[Document]:
        pass

class Clearable(ABC):
    @abstractmethod
    def clear(self) -> None:
        pass

class VectorStore(Searchable, Clearable):  # Implement what's needed
    pass
```

#### **D - Dependency Inversion Principle**
Depend on abstractions, not concretions:
```python
# ✅ GOOD: Depend on abstraction
class ChatController:
    def __init__(self, llm_service: AbstractLLMService, 
                 vector_service: AbstractVectorStoreService):
        self.llm = llm_service
        self.vector_store = vector_service

# ❌ BAD: Depend on concrete class
class ChatController:
    def __init__(self):
        self.llm = OllamaLLM()  # Tightly coupled
```

### 3.3. Code Quality Standards

#### **Type Hints & Docstrings** (Bắt buộc)
```python
from typing import List, Optional, Dict, Any
from dataclasses import dataclass

def process_query(
    query: str, 
    k: int = 3, 
    filters: Optional[Dict[str, Any]] = None
) -> List[Document]:
    """
    Process user query and retrieve relevant documents.
    
    Args:
        query: User's question in natural language
        k: Number of documents to retrieve (default: 3)
        filters: Optional metadata filters
        
    Returns:
        List of relevant Document objects sorted by relevance
        
    Raises:
        ValueError: If query is empty
        VectorStoreError: If vector store is not initialized
        
    Example:
        >>> docs = process_query("What is RAG?", k=5)
        >>> print(len(docs))
        5
    """
    if not query.strip():
        raise ValueError("Query cannot be empty")
    # Implementation...
```

#### **Error Handling Strategy**
```python
# src/utils/exceptions.py
class SmartDocError(Exception):
    """Base exception for SmartDoc application"""
    pass

class DocumentLoadError(SmartDocError):
    """Raised when document loading fails"""
    pass

class VectorStoreError(SmartDocError):
    """Raised when vector store operations fail"""
    pass

class LLMConnectionError(SmartDocError):
    """Raised when cannot connect to Ollama"""
    pass

# Usage in service
def load_document(self, path: str) -> Document:
    try:
        with open(path, 'rb') as f:
            content = self.loader.load(f)
        return Document(content=content, metadata={'source': path})
    except FileNotFoundError:
        logger.error(f"File not found: {path}")
        raise DocumentLoadError(f"Cannot find file: {path}")
    except Exception as e:
        logger.exception(f"Unexpected error loading {path}")
        raise DocumentLoadError(f"Failed to load document: {str(e)}")
```

#### **Logging Standards**
```python
# src/utils/logger.py
import logging
from typing import Optional

def setup_logger(name: str, level: int = logging.INFO) -> logging.Logger:
    """Configure and return a logger instance"""
    logger = logging.getLogger(name)
    logger.setLevel(level)
    
    if not logger.handlers:
        handler = logging.StreamHandler()
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        handler.setFormatter(formatter)
        logger.addHandler(handler)
    
    return logger

# Usage in all modules
logger = setup_logger(__name__)

class DocumentService:
    def chunk_document(self, doc: Document, chunk_size: int) -> List[Document]:
        logger.info(f"Chunking document {doc.id} with size={chunk_size}")
        chunks = self._split(doc.content, chunk_size)
        logger.info(f"Created {len(chunks)} chunks from document {doc.id}")
        return chunks
```

## 4. Core Architecture & Pipeline Parameters

### 4.1. Default RAG Configuration
```python
@dataclass
class RAGConfig:
    """Default configuration for RAG pipeline"""
    # Text Splitting
    chunk_size: int = 1000
    chunk_overlap: int = 100
    
    # Retrieval
    search_type: str = "similarity"
    retrieval_k: int = 3
    retrieval_fetch_k: int = 20
    
    # LLM Parameters
    temperature: float = 0.7
    top_p: float = 0.9
    repeat_penalty: float = 1.1
    max_tokens: int = 512
    
    # Models
    llm_model: str = "qwen2.5:7b"
    embedding_model: str = "sentence-transformers/paraphrase-multilingual-mpnet-base-v2"
    
    # Prompt Template
    system_prompt: str = """You are a helpful AI assistant. Answer the question based on the provided context.
    If the context doesn't contain relevant information, say "I don't have enough information to answer this question."
    Detect the language of the question and respond in the same language.
    Keep your answer concise (3-4 sentences maximum).
    """
```

### 4.2. Prompt Engineering Template
```python
class PromptTemplate:
    """Professional prompt templates with language detection"""
    
    SYSTEM_PROMPT = """You are SmartDoc AI, an intelligent document assistant.
    
    Instructions:
    1. Analyze the CONTEXT provided below
    2. Answer the QUESTION based ONLY on the context
    3. If context is insufficient, clearly state: "I don't have enough information"
    4. Detect question language (Vietnamese/English) and respond in SAME language
    5. Keep answers concise: 3-4 sentences maximum
    6. Be factual and precise
    
    CONTEXT:
    {context}
    
    QUESTION:
    {question}
    
    ANSWER:"""
    
    @staticmethod
    def format_prompt(context: str, question: str) -> str:
        return PromptTemplate.SYSTEM_PROMPT.format(
            context=context,
            question=question
        )
```

## 5. Development Roadmap & Task Implementation

### Task 1: DOCX Support (OOP Implementation)
```python
# src/services/document_service.py
from langchain_community.document_loaders import Docx2txtLoader
from pathlib import Path

class DocumentLoaderFactory:
    """Factory pattern for document loaders"""
    
    LOADERS = {
        '.pdf': PDFPlumberLoader,
        '.docx': Docx2txtLoader,
        '.txt': TextLoader,
    }
    
    @classmethod
    def create_loader(cls, file_path: str):
        extension = Path(file_path).suffix.lower()
        loader_class = cls.LOADERS.get(extension)
        
        if not loader_class:
            raise UnsupportedFileTypeError(
                f"File type {extension} not supported. "
                f"Supported types: {list(cls.LOADERS.keys())}"
            )
        
        logger.info(f"Creating {loader_class.__name__} for {file_path}")
        return loader_class(file_path)
```

### Task 2 & 3: Chat History & Vector Store Management
```python
# src/models/chat_model.py
@dataclass
class ChatHistory:
    """Manages conversation history"""
    messages: List[ChatMessage] = field(default_factory=list)
    max_history: int = 50
    
    def add_message(self, role: str, content: str) -> None:
        self.messages.append(ChatMessage(
            role=role,
            content=content,
            timestamp=datetime.now()
        ))
        if len(self.messages) > self.max_history:
            self.messages.pop(0)
    
    def clear(self) -> None:
        logger.info("Clearing chat history")
        self.messages.clear()
    
    def to_dict(self) -> List[Dict]:
        return [asdict(msg) for msg in self.messages]

# src/controllers/chat_controller.py
class ChatController:
    def clear_history(self) -> None:
        """Clear chat history with confirmation"""
        if st.session_state.get('chat_history'):
            st.session_state.chat_history.clear()
            logger.info("Chat history cleared by user")
            st.success("✅ Chat history cleared successfully")
    
    def clear_vector_store(self) -> None:
        """Clear vector store with confirmation"""
        try:
            self.vector_service.clear_store()
            logger.warning("Vector store cleared by user")
            st.success("✅ Vector store cleared successfully")
        except Exception as e:
            logger.error(f"Error clearing vector store: {e}")
            st.error(f"❌ Error: {str(e)}")
```

### Task 4: Chunk Tuning UI
```python
# src/views/settings_screen.py
class SettingsScreen:
    """Settings configuration UI"""
    
    def render_chunk_settings(self):
        st.subheader("⚙️ Chunk Configuration")
        
        chunk_size = st.slider(
            "Chunk Size",
            min_value=500,
            max_value=2000,
            value=1000,
            step=100,
            help="Size of text chunks for processing"
        )
        
        chunk_overlap = st.slider(
            "Chunk Overlap",
            min_value=50,
            max_value=200,
            value=100,
            step=10,
            help="Overlap between consecutive chunks"
        )
        
        if st.button("Apply Settings"):
            self.controller.update_chunk_config(chunk_size, chunk_overlap)
            st.success("✅ Settings updated successfully")
```

### Task 5: Citations & Metadata Display
```python
# src/models/document_model.py
@dataclass
class DocumentChunk:
    """Document chunk with metadata for citations"""
    content: str
    metadata: Dict[str, Any]
    
    @property
    def page_number(self) -> Optional[int]:
        return self.metadata.get('page')
    
    @property
    def source_file(self) -> str:
        return self.metadata.get('source', 'Unknown')
    
    def get_citation(self) -> str:
        """Format citation string"""
        page = self.page_number
        source = Path(self.source_file).name
        if page:
            return f"[{source}, page {page}]"
        return f"[{source}]"

# src/views/chat_screen.py
def display_answer_with_citations(self, answer: str, sources: List[DocumentChunk]):
    """Display answer with clickable citations"""
    st.markdown(answer)
    
    with st.expander("📚 View Sources"):
        for idx, source in enumerate(sources, 1):
            st.markdown(f"**Source {idx}:** {source.get_citation()}")
            st.text_area(
                f"Context {idx}",
                value=source.content,
                height=100,
                key=f"source_{idx}",
                disabled=True
            )
```

## 6. Coding Principles & Standards

### 6.1. Local-First Architecture
```python
# ✅ GOOD: Local inference only
class OllamaLLMService(AbstractLLMService):
    def __init__(self, model: str = "qwen2.5:7b"):
        self.llm = Ollama(
            model=model,
            base_url="http://localhost:11434"  # Local Ollama
        )
    
    def generate(self, prompt: str) -> str:
        try:
            return self.llm.invoke(prompt)
        except Exception as e:
            raise LLMConnectionError("Cannot connect to Ollama. Is it running?")

# ❌ BAD: External API calls
class OpenAIService:  # Don't use this!
    def __init__(self, api_key: str):
        self.client = OpenAI(api_key=api_key)  # ❌ Costs money
```

### 6.2. Comprehensive Logging
```python
# Bắt buộc log ở các điểm sau:
logger = setup_logger(__name__)

class RAGService:
    def process_query(self, query: str) -> str:
        # 1. Log input
        logger.info(f"Received query: {query[:50]}...")
        
        # 2. Log retrieval
        docs = self.vector_store.search(query, k=3)
        logger.info(f"Retrieved {len(docs)} documents")
        
        # 3. Log chunks
        context = "\n".join([doc.content for doc in docs])
        logger.debug(f"Context length: {len(context)} characters")
        
        # 4. Log LLM call
        logger.info("Calling LLM for generation")
        answer = self.llm.generate(self.prompt.format(context, query))
        
        # 5. Log output
        logger.info(f"Generated answer: {answer[:50]}...")
        return answer
```

### 6.3. Robust Error Handling
```python
# src/controllers/document_controller.py
class DocumentController:
    def upload_and_process(self, uploaded_file) -> bool:
        """Upload and process document with comprehensive error handling"""
        try:
            # Validation
            if not self._validate_file(uploaded_file):
                st.error("❌ Invalid file format")
                return False
            
            # Save file
            file_path = self._save_uploaded_file(uploaded_file)
            logger.info(f"Saved file to {file_path}")
            
            # Load document
            try:
                document = self.doc_service.load_document(file_path)
            except DocumentLoadError as e:
                st.error(f"❌ Cannot load document: {str(e)}")
                return False
            
            # Process chunks
            try:
                chunks = self.doc_service.chunk_document(document)
                logger.info(f"Created {len(chunks)} chunks")
            except Exception as e:
                st.error(f"❌ Chunking failed: {str(e)}")
                return False
            
            # Store vectors
            try:
                self.vector_service.add_documents(chunks)
                st.success(f"✅ Processed {len(chunks)} chunks successfully")
                return True
            except VectorStoreError as e:
                st.error(f"❌ Vector store error: {str(e)}")
                return False
                
        except Exception as e:
            logger.exception("Unexpected error in upload_and_process")
            st.error(f"❌ Unexpected error: {str(e)}")
            return False
```

## 7. Testing Strategy

### 7.1. Unit Tests Structure
```python
# tests/test_document_service.py
import pytest
from src.services.document_service import DocumentService
from src.utils.exceptions import DocumentLoadError

class TestDocumentService:
    @pytest.fixture
    def doc_service(self):
        return DocumentService()
    
    def test_load_pdf_success(self, doc_service, tmp_path):
        # Arrange
        pdf_path = tmp_path / "test.pdf"
        # Create dummy PDF...
        
        # Act
        document = doc_service.load_document(str(pdf_path))
        
        # Assert
        assert document is not None
        assert document.content
        assert document.metadata['source'] == str(pdf_path)
    
    def test_load_invalid_file_raises_error(self, doc_service):
        # Act & Assert
        with pytest.raises(DocumentLoadError):
            doc_service.load_document("nonexistent.pdf")
    
    def test_chunk_document(self, doc_service):
        # Arrange
        doc = Document(content="A" * 5000, metadata={})
        
        # Act
        chunks = doc_service.chunk_document(doc, chunk_size=1000, chunk_overlap=100)
        
        # Assert
        assert len(chunks) > 1
        assert all(len(chunk.content) <= 1000 for chunk in chunks)
```

### 7.2. Integration Tests
```python
# tests/test_rag_pipeline.py
class TestRAGPipeline:
    def test_end_to_end_query(self, rag_service, sample_document):
        # Arrange
        rag_service.add_document(sample_document)
        query = "What is the main topic?"
        
        # Act
        answer = rag_service.query(query)
        
        # Assert
        assert answer
        assert len(answer) > 0
        assert isinstance(answer, str)
```

## 8. Performance Optimization Guidelines

### 8.1. Caching Strategy
```python
from functools import lru_cache

class EmbeddingService:
    @lru_cache(maxsize=1000)
    def embed_text(self, text: str) -> np.ndarray:
        """Cache embeddings for frequently used texts"""
        return self.model.encode(text)
```

### 8.2. Batch Processing
```python
def add_documents_batch(self, documents: List[Document], batch_size: int = 32):
    """Process documents in batches for efficiency"""
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i+batch_size]
        embeddings = self.embedding_service.embed_batch([d.content for d in batch])
        self.vector_store.add(embeddings, batch)
```

## 9. Streamlit Best Practices

### 9.1. Session State Management
```python
# src/utils/session_state.py
class SessionStateManager:
    """Centralized session state management"""
    
    @staticmethod
    def initialize():
        """Initialize all session state variables"""
        if 'chat_history' not in st.session_state:
            st.session_state.chat_history = ChatHistory()
        
        if 'vector_store_initialized' not in st.session_state:
            st.session_state.vector_store_initialized = False
        
        if 'current_document' not in st.session_state:
            st.session_state.current_document = None
        
        if 'rag_config' not in st.session_state:
            st.session_state.rag_config = RAGConfig()
    
    @staticmethod
    def get(key: str, default: Any = None) -> Any:
        """Safely get session state value"""
        return st.session_state.get(key, default)
    
    @staticmethod
    def set(key: str, value: Any) -> None:
        """Set session state value"""
        st.session_state[key] = value
        logger.debug(f"Session state updated: {key}")
```

### 9.2. Component Architecture
```python
# src/views/components.py
class UIComponents:
    """Reusable UI components following Component Pattern"""
    
    @staticmethod
    def file_uploader(
        label: str = "Upload Document",
        accepted_types: List[str] = ['.pdf', '.docx']
    ) -> Optional[Any]:
        """Reusable file uploader component"""
        return st.file_uploader(
            label,
            type=accepted_types,
            help=f"Supported formats: {', '.join(accepted_types)}"
        )
    
    @staticmethod
    def chat_message(role: str, content: str, avatar: str = None):
        """Render a chat message bubble"""
        with st.chat_message(role, avatar=avatar):
            st.markdown(content)
    
    @staticmethod
    def loading_spinner(message: str = "Processing..."):
        """Context manager for loading spinner"""
        return st.spinner(message)
    
    @staticmethod
    def error_alert(message: str, details: Optional[str] = None):
        """Styled error alert"""
        st.error(f"❌ {message}")
        if details:
            with st.expander("Error Details"):
                st.code(details)
    
    @staticmethod
    def success_alert(message: str):
        """Styled success alert"""
        st.success(f"✅ {message}")
    
    @staticmethod
    def sidebar_section(title: str, icon: str = "📌"):
        """Create sidebar section with title"""
        st.sidebar.markdown(f"### {icon} {title}")
```

### 9.3. Main App Structure (app.py)
```python
# app.py
import streamlit as st
from src.controllers.chat_controller import ChatController
from src.controllers.document_controller import DocumentController
from src.views.chat_screen import ChatScreen
from src.views.document_screen import DocumentScreen
from src.views.settings_screen import SettingsScreen
from src.utils.session_state import SessionStateManager
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

def main():
    """Main application entry point"""
    # Page configuration
    st.set_page_config(
        page_title="SmartDoc AI",
        page_icon="📚",
        layout="wide",
        initial_sidebar_state="expanded"
    )
    
    # Initialize session state
    SessionStateManager.initialize()
    
    # Initialize controllers (dependency injection)
    chat_controller = ChatController()
    document_controller = DocumentController()
    
    # Sidebar navigation
    with st.sidebar:
        st.title("📚 SmartDoc AI")
        st.markdown("---")
        
        page = st.radio(
            "Navigation",
            ["💬 Chat", "📄 Documents", "⚙️ Settings"],
            label_visibility="collapsed"
        )
    
    # Route to appropriate screen
    if page == "💬 Chat":
        chat_screen = ChatScreen(chat_controller)
        chat_screen.render()
    
    elif page == "📄 Documents":
        doc_screen = DocumentScreen(document_controller)
        doc_screen.render()
    
    elif page == "⚙️ Settings":
        settings_screen = SettingsScreen()
        settings_screen.render()

if __name__ == "__main__":
    main()
```

### 9.4. Screen Implementation Example
```python
# src/views/chat_screen.py
from typing import Optional
import streamlit as st
from src.controllers.chat_controller import ChatController
from src.views.components import UIComponents
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class ChatScreen:
    """Chat interface screen following MVC pattern"""
    
    def __init__(self, controller: ChatController):
        self.controller = controller
        self.components = UIComponents()
    
    def render(self):
        """Render the chat screen"""
        st.title("💬 Chat with Your Documents")
        
        # Check if vector store is ready
        if not st.session_state.get('vector_store_initialized'):
            self._render_empty_state()
            return
        
        # Render chat interface
        self._render_chat_history()
        self._render_chat_input()
        
        # Sidebar actions
        self._render_sidebar_actions()
    
    def _render_empty_state(self):
        """Show empty state when no documents loaded"""
        st.info("📄 Please upload documents first in the Documents tab")
        
        col1, col2, col3 = st.columns([1, 2, 1])
        with col2:
            st.image("assets/empty_state.png", use_container_width=True)
    
    def _render_chat_history(self):
        """Display chat history"""
        chat_history = st.session_state.get('chat_history')
        
        if not chat_history or not chat_history.messages:
            st.info("👋 Start a conversation by asking a question below")
            return
        
        for message in chat_history.messages:
            avatar = "🧑" if message.role == "user" else "🤖"
            self.components.chat_message(
                role=message.role,
                content=message.content,
                avatar=avatar
            )
    
    def _render_chat_input(self):
        """Render chat input box"""
        if prompt := st.chat_input("Ask a question about your documents..."):
            # Add user message
            self._add_user_message(prompt)
            
            # Get AI response
            with self.components.loading_spinner("Thinking..."):
                try:
                    response = self.controller.process_query(prompt)
                    self._add_assistant_message(response)
                except Exception as e:
                    logger.error(f"Error processing query: {e}")
                    self.components.error_alert(
                        "Failed to process your question",
                        details=str(e)
                    )
    
    def _add_user_message(self, content: str):
        """Add user message to chat"""
        st.session_state.chat_history.add_message("user", content)
        self.components.chat_message("user", content, avatar="🧑")
    
    def _add_assistant_message(self, content: str):
        """Add assistant message to chat"""
        st.session_state.chat_history.add_message("assistant", content)
        self.components.chat_message("assistant", content, avatar="🤖")
    
    def _render_sidebar_actions(self):
        """Render sidebar action buttons"""
        st.sidebar.markdown("---")
        self.components.sidebar_section("Actions", "🔧")
        
        if st.sidebar.button("🗑️ Clear Chat History", use_container_width=True):
            if st.sidebar.checkbox("Confirm clear history?"):
                self.controller.clear_history()
                st.rerun()
```

## 10. Quick Start Implementation Guide

### Step 1: Create Base Models
```python
# src/models/document_model.py
from dataclasses import dataclass, field
from datetime import datetime
from typing import Dict, Any, Optional
import uuid

@dataclass
class Document:
    """Domain entity for documents"""
    content: str
    metadata: Dict[str, Any] = field(default_factory=dict)
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    created_at: datetime = field(default_factory=datetime.now)
    
    def __post_init__(self):
        """Validation after initialization"""
        if not self.content:
            raise ValueError("Document content cannot be empty")

# src/models/chat_model.py
from dataclasses import dataclass, field
from datetime import datetime
from typing import List, Literal, Optional, Dict
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

@dataclass
class ChatMessage:
    """Value object for chat messages"""
    role: Literal["user", "assistant", "system"]
    content: str
    timestamp: datetime = field(default_factory=datetime.now)
    metadata: Optional[Dict] = None

@dataclass
class ChatHistory:
    """Conversation history management"""
    messages: List[ChatMessage] = field(default_factory=list)
    max_history: int = 50
    
    def add_message(self, role: str, content: str) -> None:
        """Add a new message to history"""
        self.messages.append(ChatMessage(role=role, content=content))
        
        # Trim if exceeds max
        if len(self.messages) > self.max_history:
            removed = self.messages.pop(0)
            logger.debug(f"Removed old message from history: {removed.timestamp}")
    
    def clear(self) -> None:
        """Clear all messages"""
        logger.info(f"Clearing {len(self.messages)} messages from history")
        self.messages.clear()
    
    def get_recent(self, n: int = 10) -> List[ChatMessage]:
        """Get n most recent messages"""
        return self.messages[-n:] if len(self.messages) > n else self.messages
```

### Step 2: Implement Services
```python
# src/services/llm_service.py
from abc import ABC, abstractmethod
from typing import Optional
from langchain_community.llms import Ollama
from src.utils.exceptions import LLMConnectionError
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class AbstractLLMService(ABC):
    """Abstract interface for LLM services"""
    
    @abstractmethod
    def generate(self, prompt: str, **kwargs) -> str:
        """Generate response from prompt"""
        pass

class OllamaLLMService(AbstractLLMService):
    """Ollama LLM service implementation"""
    
    def __init__(
        self,
        model: str = "qwen2.5:7b",
        base_url: str = "http://localhost:11434",
        temperature: float = 0.7,
        top_p: float = 0.9,
        repeat_penalty: float = 1.1
    ):
        logger.info(f"Initializing Ollama with model: {model}")
        
        try:
            self.llm = Ollama(
                model=model,
                base_url=base_url,
                temperature=temperature,
                top_p=top_p,
                repeat_penalty=repeat_penalty
            )
            logger.info("Ollama LLM initialized successfully")
        except Exception as e:
            logger.error(f"Failed to initialize Ollama: {e}")
            raise LLMConnectionError(
                "Cannot connect to Ollama. Ensure Ollama is running on localhost:11434"
            )
    
    def generate(self, prompt: str, **kwargs) -> str:
        """Generate response from prompt"""
        try:
            logger.info(f"Generating response for prompt length: {len(prompt)}")
            response = self.llm.invoke(prompt, **kwargs)
            logger.info(f"Generated response length: {len(response)}")
            return response
        
        except Exception as e:
            logger.error(f"Generation failed: {e}")
            raise LLMConnectionError(f"Failed to generate response: {str(e)}")

# src/services/vector_store_service.py
from abc import ABC, abstractmethod
from typing import List, Optional
import numpy as np
from langchain_community.vectorstores import FAISS
from langchain_community.embeddings import HuggingFaceEmbeddings
from src.models.document_model import Document
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class AbstractVectorStoreService(ABC):
    """Abstract interface for vector store operations"""
    
    @abstractmethod
    def add_documents(self, documents: List[Document]) -> None:
        pass
    
    @abstractmethod
    def similarity_search(self, query: str, k: int = 3) -> List[Document]:
        pass
    
    @abstractmethod
    def clear_store(self) -> None:
        pass

class FAISSVectorStoreService(AbstractVectorStoreService):
    """FAISS vector store implementation"""
    
    def __init__(self, embedding_model: str = "sentence-transformers/paraphrase-multilingual-mpnet-base-v2"):
        logger.info(f"Initializing FAISS with embedding model: {embedding_model}")
        
        self.embeddings = HuggingFaceEmbeddings(
            model_name=embedding_model,
            model_kwargs={'device': 'cpu'},
            encode_kwargs={'normalize_embeddings': True}
        )
        
        self.vector_store: Optional[FAISS] = None
        logger.info("FAISS vector store initialized")
    
    def add_documents(self, documents: List[Document]) -> None:
        """Add documents to vector store"""
        if not documents:
            logger.warning("No documents to add")
            return
        
        logger.info(f"Adding {len(documents)} documents to vector store")
        
        # Convert to LangChain document format
        from langchain.schema import Document as LCDocument
        lc_docs = [
            LCDocument(page_content=doc.content, metadata=doc.metadata)
            for doc in documents
        ]
        
        if self.vector_store is None:
            self.vector_store = FAISS.from_documents(lc_docs, self.embeddings)
            logger.info("Created new FAISS index")
        else:
            self.vector_store.add_documents(lc_docs)
            logger.info("Added documents to existing FAISS index")
    
    def similarity_search(self, query: str, k: int = 3) -> List[Document]:
        """Search for similar documents"""
        if self.vector_store is None:
            logger.warning("Vector store not initialized, returning empty results")
            return []
        
        logger.info(f"Searching for {k} similar documents")
        results = self.vector_store.similarity_search(query, k=k)
        
        # Convert back to domain Document
        docs = [
            Document(content=doc.page_content, metadata=doc.metadata)
            for doc in results
        ]
        
        logger.info(f"Found {len(docs)} similar documents")
        return docs
    
    def clear_store(self) -> None:
        """Clear the vector store"""
        logger.warning("Clearing vector store")
        self.vector_store = None
```

### Step 3: Create Controllers
```python
# src/controllers/chat_controller.py
from typing import List, Optional
import streamlit as st
from src.services.llm_service import AbstractLLMService, OllamaLLMService
from src.services.vector_store_service import AbstractVectorStoreService
from src.models.chat_model import ChatHistory
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class ChatController:
    """Controller for chat operations"""
    
    def __init__(
        self,
        llm_service: Optional[AbstractLLMService] = None,
        vector_service: Optional[AbstractVectorStoreService] = None
    ):
        # Dependency injection with defaults
        self.llm_service = llm_service or OllamaLLMService()
        self.vector_service = vector_service or st.session_state.get('vector_service')
        
        logger.info("ChatController initialized")
    
    def process_query(self, query: str) -> str:
        """Process user query and return response"""
        # Validation
        if not query or not query.strip():
            logger.warning("Empty query received")
            raise ValueError("Query cannot be empty")
        
        logger.info(f"Processing query: {query[:50]}...")
        
        # Retrieve relevant documents
        try:
            relevant_docs = self.vector_service.similarity_search(query, k=3)
            logger.info(f"Retrieved {len(relevant_docs)} relevant documents")
        except Exception as e:
            logger.error(f"Document retrieval failed: {e}")
            raise
        
        # Build context
        context = "\n\n".join([doc.content for doc in relevant_docs])
        logger.debug(f"Context length: {len(context)} characters")
        
        # Create prompt
        prompt = self._build_prompt(context, query)
        
        # Generate response
        try:
            response = self.llm_service.generate(prompt)
            logger.info("Response generated successfully")
            return response
        except Exception as e:
            logger.error(f"Response generation failed: {e}")
            raise
    
    def _build_prompt(self, context: str, question: str) -> str:
        """Build prompt for LLM"""
        return f"""You are SmartDoc AI, an intelligent document assistant.

Answer the QUESTION based ONLY on the CONTEXT below.
If the context doesn't contain enough information, say "I don't have enough information to answer this question."
Detect the question language and respond in the SAME language.
Keep your answer concise (3-4 sentences maximum).

CONTEXT:
{context}

QUESTION:
{question}

ANSWER:"""
    
    def clear_history(self) -> None:
        """Clear chat history"""
        if 'chat_history' in st.session_state:
            st.session_state.chat_history.clear()
            logger.info("Chat history cleared")
```

## 11. Common Pitfalls & Solutions

### Pitfall 1: Tight Coupling
❌ **Bad**:
```python
class ChatScreen:
    def __init__(self):
        self.llm = Ollama(model="qwen2.5:7b")  # Tightly coupled
        self.vector_store = FAISS()
```

✅ **Good**:
```python
class ChatScreen:
    def __init__(self, controller: ChatController):
        self.controller = controller  # Depend on abstraction
```

### Pitfall 2: Business Logic in Views
❌ **Bad**:
```python
# In chat_screen.py
def render(self):
    query = st.text_input("Question")
    docs = vector_store.search(query)  # Business logic in view!
    answer = llm.generate(docs)
    st.write(answer)
```

✅ **Good**:
```python
# In chat_screen.py
def render(self):
    query = st.text_input("Question")
    answer = self.controller.process_query(query)  # Delegate to controller
    st.write(answer)
```

### Pitfall 3: No Error Handling
❌ **Bad**:
```python
def load_document(self, path: str):
    return PDFPlumberLoader(path).load()  # No error handling
```

✅ **Good**:
```python
def load_document(self, path: str) -> Document:
    try:
        loader = PDFPlumberLoader(path)
        content = loader.load()
        return Document(content=content)
    except FileNotFoundError:
        logger.error(f"File not found: {path}")
        raise DocumentLoadError(f"Cannot find file: {path}")
    except Exception as e:
        logger.exception(f"Unexpected error loading {path}")
        raise DocumentLoadError(f"Failed to load: {str(e)}")
```

### Pitfall 4: Missing Type Hints
❌ **Bad**:
```python
def process_query(self, query):
    docs = self.vector_store.search(query)
    return self.llm.generate(docs)
```

✅ **Good**:
```python
def process_query(self, query: str) -> str:
    """Process user query and return AI response"""
    docs: List[Document] = self.vector_store.search(query, k=3)
    response: str = self.llm.generate(self._build_prompt(docs, query))
    return response
```

### Pitfall 5: Streamlit State Mismanagement
❌ **Bad**:
```python
# Direct manipulation
st.session_state['chat_history'] = []
```

✅ **Good**:
```python
# Centralized management
class SessionStateManager:
    @staticmethod
    def initialize():
        if 'chat_history' not in st.session_state:
            st.session_state.chat_history = ChatHistory()
```

## 12. Deployment Checklist

### Pre-Deployment
- [ ] All type hints added
- [ ] Comprehensive docstrings
- [ ] Error handling in all services
- [ ] Logging configured properly
- [ ] Unit tests passing
- [ ] Integration tests passing
- [ ] No hardcoded secrets/API keys
- [ ] requirements.txt updated

### Ollama Setup
- [ ] Ollama installed and running
- [ ] Model `qwen2.5:7b` downloaded
- [ ] Base URL configured correctly
- [ ] Connection test successful

### Performance
- [ ] Embedding caching enabled
- [ ] Batch processing for documents
- [ ] FAISS index optimized
- [ ] Memory usage monitored

### Security
- [ ] No external API calls
- [ ] All data processed locally
- [ ] Input validation on all endpoints
- [ ] File upload size limits set

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bin2aa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bin2aa)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
