# LibreChat Custom Features

This directory contains custom implementations to replace paid API dependencies in LibreChat.

## Directory Structure

### `/search/`
Custom web search implementation to replace paid APIs:
- Serper API
- Firecrawl API  
- Jina API
- Cohere API

**Implementation Strategy:** Aggregated search using free APIs + custom scraper

### `/code-interpreter/`
Custom code execution environment to replace LibreChat's paid Code API:
- Docker-based secure sandbox
- Multi-language support (Python, JavaScript, etc.)
- File persistence and cleanup
- Session management

**Implementation Strategy:** Docker-based execution with security isolation

### `/shared/`
Shared utilities and configurations:
- Common API clients
- Utility functions
- Configuration management
- Testing utilities

## Development Workflow

1. **Local Development**: LibreChat running at `http://localhost:3080`
2. **API Development**: Backend services in `/api/server/services/Tools/`
3. **Frontend Integration**: Client components in `/client/src/`
4. **Custom Features**: Custom implementations in `/custom-features/`

## Integration Points

### Backend Integration
- Replace existing tools in `/api/server/services/Tools/`
- Maintain existing API interfaces for seamless integration
- Add new endpoints as needed

### Frontend Integration  
- Maintain existing UI components
- Extend tool functionality without breaking changes
- Preserve user experience and interface

## Priority Implementation Order

1. **Priority 1**: Custom Web Search (Weeks 1-3)
2. **Priority 2**: Custom Code Interpreter (Weeks 4-6)  
3. **Priority 3**: LLM Integration Enhancement (Weeks 7-8)
4. **Priority 4**: Additional Features (Weeks 9-10)

## Testing Strategy

- Unit tests for individual components
- Integration tests with LibreChat
- End-to-end testing via UI
- Performance and security testing

---

**Created**: June 16, 2025  
**Purpose**: Replace paid API dependencies with custom open-source implementations
