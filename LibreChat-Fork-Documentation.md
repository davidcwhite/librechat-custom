# LibreChat Fork Documentation

## Project Overview

This document outlines our fork of LibreChat, an open-source ChatGPT clone that provides a comprehensive AI chat interface with multiple provider support. Our goal is to create a customized version that eventually replaces paid API features with our own implementations.

**Original Repository**: https://github.com/danny-avila/LibreChat.git  
**Fork Date**: June 15, 2025  
**LibreChat Version**: v0.7.8

## LibreChat Architecture Analysis

### Core Components

LibreChat is a full-stack Node.js application with the following architecture:

#### Backend (API)
- **Location**: `/api`
- **Framework**: Express.js with TypeScript support
- **Database**: MongoDB for data persistence
- **Search**: Meilisearch for conversation indexing
- **Structure**:
  - `/api/server` - Main server logic, routes, middleware
  - `/api/models` - MongoDB models and schemas
  - `/api/app/clients` - API client implementations for various LLM providers
  - `/api/server/services` - Business logic services

#### Frontend (Client)
- **Location**: `/client`
- **Framework**: React with TypeScript
- **Build System**: Vite
- **UI Library**: Tailwind CSS

#### Packages
- **Location**: `/packages`
- **data-provider**: Shared types and schemas
- **api**: Core API logic and utilities

### Current Paid API Features Identified

Through codebase analysis, LibreChat integrates several paid external APIs that we plan to replace:

#### 1. **Web Search Functionality**
- **Location**: `/api/server/services/Tools/search.js`, `/packages/data-provider/src/web.ts`
- **Current Providers**:
  - **Serper API** (`SERPER_API_KEY`) - Primary search provider
  - **Jina API** (`JINA_API_KEY`) - Content extraction and processing
  - **Firecrawl API** (`FIRECRAWL_API_KEY`, `FIRECRAWL_API_URL`) - Web scraping service
  - **Cohere API** (`COHERE_API_KEY`) - Search result reranking
- **Features**: Search result aggregation, source mapping, content extraction, result ranking

#### 2. **Code Interpreter**
- **Location**: Multiple files containing "code_interpreter" functionality
- **Current Implementation**: Uses OpenAI's Code Interpreter API
- **Features**: 
  - Code execution in sandboxed environment
  - File persistence between sessions
  - Support for various programming languages
  - Data visualization capabilities

#### 3. **LLM Provider APIs**
- **Providers Supported**:
  - OpenAI GPT models (including web search enhanced models)
  - Anthropic Claude
  - Google Gemini
  - AWS Bedrock
  - Azure OpenAI
- **Location**: `/api/server/services/Endpoints/`

### Docker Infrastructure

LibreChat provides a complete Docker Compose setup:
- **API Container**: Main application (`ghcr.io/danny-avila/librechat-dev:latest`)
- **MongoDB**: Database storage
- **Meilisearch**: Search indexing
- **RAG API**: Vector database for retrieval-augmented generation
- **VectorDB**: Persistent vector storage

## Setup Process

### Dependencies Installed
- [x] Node.js dependencies via `npm install`
- [x] Docker environment identified and configured

### Environment Configuration
- [x] `.env` file created from `.env.example`
- [x] Default configuration includes MongoDB, Meilisearch, and other service URLs

### Current Status
- **Docker Setup**: In progress (Docker Desktop starting)
- **Services**: Ready to deploy via `docker-compose up -d`

## Planned Customizations

### Priority 1: Web Search Replacement
**Target**: Replace Serper, Jina, Firecrawl, and Cohere APIs with custom implementations
- Implement custom web search using open-source search engines
- Create custom web scraping service
- Develop custom content extraction and reranking algorithms

### Priority 2: Code Interpreter Replacement  
**Target**: Replace OpenAI Code Interpreter with custom sandboxed execution environment
- Implement secure code execution sandbox
- Add support for multiple programming languages
- Create file persistence system
- Develop data visualization capabilities

### Priority 3: Custom LLM Integration
**Target**: Optimize for specific use cases and potentially add custom fine-tuned models
- Integrate open-source LLM alternatives
- Implement custom model routing logic
- Add support for local model deployments

## Next Steps

1. Complete Docker setup and verify all services are running
2. Test the default LibreChat functionality
3. Analyze the web search implementation in detail
4. Begin development of custom web search service
5. Create development environment for code interpreter replacement
6. Implement comprehensive testing suite for custom features

## Development Notes

- LibreChat uses a workspace-based architecture with separate API and client packages
- Authentication system supports multiple providers (local, OAuth, SAML)
- Extensive plugin system for extending functionality
- Built-in conversation management and export capabilities
- Comprehensive permission system for feature access control

---

## UI/UX Interface Exploration

### **Authentication System**
- **Login Page**: Clean, modern interface at `/login`
- **Registration Page**: Full registration form at `/register` with fields for name, username, email, password
- **Credentials**: Successfully logged in with user credentials
- **Session Management**: Proper session handling and redirection

### **Main Interface Features**

#### **Navigation & Layout**
- **URL Structure**: Uses `/c/new` for new conversations
- **Sidebar**: Collapsible left sidebar with conversation history
- **Main Chat Area**: Clean, centered chat interface
- **User Menu**: Account settings accessible via user name button

#### **Core Chat Features**
- **Model Selection**: Dropdown showing current model (e.g., `gpt-4o-mini`)
- **Chat Input**: Full-featured textarea with attachment support
- **Conversation History**: Sidebar shows "Today" section with previous chats
- **Message Display**: Clean message formatting with user/AI distinction

#### **Tool Integration (Priority Targets for Replacement)**

1. **🔍 Search Tool**
   - **Current**: Prominent search button in main interface
   - **Function**: Web search integration using paid APIs (Serper, etc.)
   - **Priority**: HIGH - Core functionality for information retrieval

2. **💻 Code Interpreter**
   - **Current**: Dedicated code interpreter button
   - **Function**: Code execution using LibreChat's paid API
   - **Priority**: HIGH - Essential for technical conversations

3. **🤖 Agent Builder**
   - **Current**: Accessible via main toolbar
   - **Function**: Custom AI agent creation and management
   - **Priority**: MEDIUM - Advanced feature for power users

#### **Content Management Features**

4. **📝 Prompts**
   - **Current**: Prompt library and management system
   - **Function**: Save and reuse conversation prompts
   - **Priority**: LOW - Enhancement feature

5. **🧠 Memories**
   - **Current**: Conversation memory system
   - **Function**: Persistent context across conversations
   - **Priority**: MEDIUM - Useful for continuity

6. **⚙️ Parameters**
   - **Current**: Model parameter controls
   - **Function**: Adjust temperature, tokens, etc.
   - **Priority**: LOW - Advanced user feature

#### **File & Bookmark System**

7. **📎 Attach Files**
   - **Current**: File upload functionality
   - **Function**: Document analysis and processing
   - **Priority**: MEDIUM - Useful for document-based conversations

8. **🔖 Bookmarks**
   - **Current**: Save important conversations
   - **Function**: Quick access to key conversations
   - **Priority**: LOW - Convenience feature

### **User Experience Observations**

#### **Strengths**
- **Clean Design**: Modern, intuitive interface with good spacing
- **Responsive Layout**: Works well on different screen sizes
- **Feature Integration**: Tools are seamlessly integrated into chat flow
- **Navigation**: Clear hierarchy and easy access to core features
- **Performance**: Fast loading and responsive interactions

#### **Areas for Custom Enhancement**
- **Search Integration**: Currently relies on paid APIs - our main target
- **Code Execution**: Uses proprietary LibreChat API - our second target
- **Model Selection**: Limited to configured providers - opportunity for local models
- **Tool Accessibility**: Some features behind paywalls - opportunity for open alternatives

### **Technical Implementation Notes**

#### **Frontend Architecture**
- **Framework**: React-based with modern component architecture
- **Styling**: Tailwind CSS with custom design tokens
- **State Management**: Proper state handling for tools and conversations
- **API Integration**: RESTful API calls to backend services
- **Real-time Updates**: WebSocket or polling for live conversations

#### **Integration Points for Custom Features**
- **Tool Buttons**: Easy to modify and extend tool toolbar
- **Model Selector**: Can be enhanced to support local/custom models
- **Search Interface**: Current search UI can be repurposed for custom search
- **Code Execution**: Code interpreter UI can connect to custom sandbox
- **Plugin System**: Extensible architecture for adding new features

---

## Development Environment Setup

### **Prerequisites Verified**
- ✅ Docker & Docker Compose running
- ✅ LibreChat stack operational (API, MongoDB, Meilisearch, etc.)
- ✅ Node.js environment configured
- ✅ npm dependencies installed
- ✅ User authentication working

### **Next Steps for Custom Development**

#### **Phase 1: Custom Web Search Service**
- **Goal**: Replace Serper/Firecrawl APIs with custom search
- **Approach**: Create new API endpoint in `/api/server/services/Tools/`
- **UI Integration**: Modify existing search button to use custom service
- **Backend**: Implement aggregated search using free APIs + custom scraper

#### **Phase 2: Custom Code Interpreter**
- **Goal**: Replace LibreChat Code API with Docker sandbox
- **Approach**: Create secure code execution service
- **UI Integration**: Maintain existing code interpreter interface
- **Backend**: Implement Docker-based execution environment

### **Development Workflow Ready**
1. **Local Testing**: LibreChat running at `http://localhost:3080`
2. **API Development**: Backend services in `/api/server/`
3. **Frontend Modifications**: Client components in `/client/src/`
4. **Testing**: Full stack testing environment operational

---

**Last Updated**: June 16, 2025 08:10 GMT  
**Status**: UI/UX exploration complete, development environment ready

---

## Detailed Paid API Analysis

### Complete List of Paid APIs Identified

#### **PRIORITY 1: Core Functionality APIs (High Impact)**

1. **Web Search & Content Processing APIs**
   - `SERPER_API_KEY` - Primary web search provider
   - `FIRECRAWL_API_KEY` & `FIRECRAWL_API_URL` - Web scraping and content extraction
   - `JINA_API_KEY` - Content processing and extraction
   - `COHERE_API_KEY` - Search result reranking and processing
   - **Location**: `/packages/data-provider/src/web.ts`, `/api/server/services/Tools/search.js`
   - **Replacement Strategy**: Build custom search aggregator using multiple free search APIs + custom scraper

2. **Code Interpreter API**
   - `LIBRECHAT_CODE_API_KEY` - LibreChat's proprietary code execution service
   - **Location**: `/client/src/hooks/Plugins/useAuthCodeTool.ts`
   - **Replacement Strategy**: Implement Docker-based code execution sandbox

#### **PRIORITY 2: Search & Discovery APIs (Medium Impact)**

3. **Search APIs**
   - `GOOGLE_SEARCH_API_KEY` - Google Custom Search API
   - `YOUTUBE_API_KEY` - YouTube Data API
   - `SERPAPI_API_KEY` - SerpAPI for search results
   - `TAVILY_API_KEY` - Tavily search API
   - `TRAVERSAAL_API_KEY` - Traversaal search API
   - **Replacement Strategy**: Aggregate multiple free search sources

#### **PRIORITY 3: LLM Provider APIs (Medium Impact)**

4. **Primary LLM Providers**
   - `OPENAI_API_KEY` - OpenAI GPT models
   - `ANTHROPIC_API_KEY` - Claude models
   - `ASSISTANTS_API_KEY` - OpenAI Assistants API
   - **Location**: `/api/server/services/Endpoints/`
   - **Replacement Strategy**: Add support for open-source alternatives (Ollama, local models)

5. **Alternative LLM Providers**
   - `ANYSCALE_API_KEY`, `APIPIE_API_KEY`, `COHERE_API_KEY`
   - `DEEPSEEK_API_KEY`, `DATABRICKS_API_KEY`, `FIREWORKS_API_KEY`
   - `GROQ_API_KEY`, `MISTRAL_API_KEY`, `PERPLEXITY_API_KEY`
   - `SHUTTLEAI_API_KEY`, `TOGETHERAI_API_KEY`, `UNIFY_API_KEY`, `XAI_API_KEY`

#### **PRIORITY 4: Image Generation APIs (Low Impact)**

6. **Image Generation**
   - `DALLE_API_KEY`, `DALLE3_API_KEY`, `DALLE2_API_KEY` - DALL-E image generation
   - `FLUX_API_KEY` - Flux image generation
   - **Replacement Strategy**: Integrate with Stable Diffusion or local image generation

#### **PRIORITY 5: Voice & Communication APIs (Low Impact)**

7. **Speech & Communication**
   - `STT_API_KEY` - Speech-to-text
   - `TTS_API_KEY` - Text-to-speech
   - `MAILGUN_API_KEY` - Email services
   - **Replacement Strategy**: Use open-source alternatives (Whisper, Coqui TTS)

#### **PRIORITY 6: External Integrations (Low Impact)**

8. **External Tools & Services**
   - `ZAPIER_NLA_API_KEY` - Zapier integration
   - `OPENWEATHER_API_KEY` - Weather data
   - `FIREBASE_API_KEY` - Firebase services
   - `AZURE_AI_SEARCH_API_KEY` - Azure cognitive search
   - **Replacement Strategy**: Build custom integrations or find free alternatives

### Implementation Roadmap

#### Phase 1: Core Web Search Replacement (Weeks 1-3)
- Research and implement aggregated search using free APIs (DuckDuckGo, Bing, etc.)
- Build custom web scraper using Playwright/Puppeteer
- Implement content extraction using open-source tools
- Create search result reranking using lightweight ML models

#### Phase 2: Code Interpreter Replacement (Weeks 4-6)
- Design secure Docker-based code execution sandbox
- Implement support for Python, JavaScript, and other languages
- Add file upload/download capabilities
- Create session persistence and cleanup mechanisms

#### Phase 3: LLM Integration Enhancement (Weeks 7-8)
- Integrate Ollama for local model support
- Add support for Hugging Face transformers
- Implement model switching and routing logic
- Optimize for cost and performance

#### Phase 4: Additional Features (Weeks 9-10)
- Replace image generation with Stable Diffusion
- Implement local speech services
- Add custom external integrations
- Performance optimization and testing

---

**Last Updated**: June 16, 2025  
**Status**: Analysis complete, implementation roadmap defined
