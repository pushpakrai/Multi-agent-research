# ResearchMind · Multi-Agent AI Research System

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![LangChain](https://img.shields.io/badge/LangChain-0.2.0+-green.svg)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-orange.svg)
![Streamlit](https://img.shields.io/badge/Streamlit-Latest-red.svg)
![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)

## 🔬 Overview

**ResearchMind** is an intelligent multi-agent research system that autonomously conducts in-depth research on any topic by combining four specialized AI agents that work together in a cohesive pipeline:

1. **Search Agent** - Scours the web for recent, reliable information
2. **Reader Agent** - Scrapes and extracts deep content from top resources
3. **Writer Chain** - Synthesizes findings into a polished research report
4. **Critic Chain** - Reviews and scores the report for quality assurance

The system features a beautiful, modern Streamlit web interface with real-time pipeline monitoring and markdown report generation.

## ✨ Key Features

- **Multi-Agent Architecture**: Four specialized agents collaborate for comprehensive research
- **Real-Time Pipeline Monitoring**: Visual step-by-step progress tracking
- **Web Search & Scraping**: Integrated Tavily API for reliable search and BeautifulSoup for content extraction
- **Smart Report Generation**: GPT-4o-mini powered intelligent synthesis of research
- **Quality Assurance**: Built-in critic agent for report validation and scoring
- **Modern UI**: Beautiful dark-themed Streamlit interface with custom CSS styling
- **Markdown Export**: Download research reports in markdown format
- **Async Support**: Built for scalability with async/await capabilities

## 🏗️ Architecture

### System Components

```
┌─────────────────────────────────────────────────┐
│           Streamlit Web Interface               │
│       (app.py - Beautiful Dark Theme UI)        │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│       Research Pipeline Orchestrator            │
│            (pipeline.py)                        │
└──┬──────────────┬──────────────┬────────────────┘
   │              │              │
   ▼              ▼              ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│   Search   │ │   Reader   │ │   Writer   │
│   Agent    │ │   Agent    │ │   Chain    │
└────────────┘ └────────────┘ └────────────┘
   │              │              │
   └──────────────┴──────────────┘
          │
          ▼
    ┌──────────────┐
    │  Critic      │
    │  Chain       │
    └──────────────┘
          │
    ┌─────▼─────────────┐
    │  Tools Module     │
    │  - web_search()   │
    │  - scrape_url()   │
    └───────────────────┘
```

### File Structure

```
multi-agent-research-system/
├── app.py              # Streamlit web interface (330+ lines)
├── agents.py           # LangChain agents & chains (42 lines)
├── pipeline.py         # Research orchestration pipeline (52 lines)
├── tools.py            # Web tools (search & scraping) (32 lines)
├── requirements.txt    # Python dependencies
├── .gitignore          # Git ignore rules (includes .env)
└── README.md           # This file
```

## 📋 Codebase Explanation

### 1. **app.py** - Web Interface Layer
- **Purpose**: Provides an interactive Streamlit web application
- **Features**:
  - Custom dark theme with gradient backgrounds
  - Real-time pipeline visualization with step cards
  - Input validation and error handling
  - Results display with expandable sections
  - Markdown report rendering
  - Download functionality for reports
- **Key Functions**:
  - `step_card()`: Renders pipeline step visualization
  - Session state management for real-time updates
  - Integration with all agents from pipeline

### 2. **agents.py** - AI Agent Configuration
- **Purpose**: Defines LangChain agents and processing chains
- **Components**:

  **Search Agent**:
  ```python
  def build_search_agent():
      return create_agent(model=llm, tools=[web_search])
  ```
  - Uses GPT-4o-mini model
  - Powered by Tavily web search tool
  - Finds recent, reliable information

  **Reader Agent**:
  ```python
  def build_reader_agent():
      return create_agent(model=llm, tools=[scrape_url])
  ```
  - Extracts deep content from URLs
  - Uses BeautifulSoup web scraping

  **Writer Chain**:
  - Structured prompt for professional report writing
  - Outputs: Introduction, Key Findings (3+ points), Conclusion, Sources
  - Ensures detailed, factual, and professional content

  **Critic Chain**:
  - Reviews report quality
  - Provides score (0-10), strengths, areas for improvement
  - Ensures quality assurance

### 3. **pipeline.py** - Orchestration Layer
- **Purpose**: Manages the research workflow
- **Process** (`run_research_pipeline()`):
  1. **Search Phase**: Gathers web information via search agent
  2. **Reading Phase**: Scrapes top resources for deeper content
  3. **Writing Phase**: Synthesizes search + scraped data into comprehensive report
  4. **Criticism Phase**: Evaluates and scores the report
- **State Management**: Tracks results at each stage
- **Output**: Returns dictionary with all intermediate and final results

### 4. **tools.py** - Integration Layer
- **Purpose**: Defines tools for agents to use

  **web_search(query)**:
  - Uses Tavily API for reliable web search
  - Returns: Title, URL, and content snippet (max 5 results)
  - Decorator: `@tool` for LangChain integration

  **scrape_url(url)**:
  - Uses BeautifulSoup to parse HTML
  - Removes script, style, nav, footer tags for clean content
  - Returns: First 3000 chars of extracted text
  - Includes timeout (8s) and User-Agent headers
  - Error handling for failed requests

### 5. **requirements.txt** - Dependencies
All dependencies are production-tested and pinned to secure versions:
- **LangChain Ecosystem**: Core agent framework
- **OpenAI**: GPT-4o-mini model access
- **Tavily**: Reliable web search API
- **Streamlit**: Web interface framework
- **BeautifulSoup4 + Requests**: Web scraping
- **python-dotenv**: Environment variable management
- **Additional**: Rich logging, Pydantic validation, Tenacity retry logic

## 🔐 Security Analysis

### ✅ Security Best Practices Implemented

1. **Environment Variable Management**
   - `.env` file is properly listed in `.gitignore`
   - API keys never hardcoded in source
   - Uses `python-dotenv` for safe loading

2. **API Key Protection**
   - `TAVILY_API_KEY` loaded from `.env` only
   - `OpenAI API key` loaded via LangChain environment
   - No credentials in repository

3. **Input Validation**
   - Topic input validated before processing
   - URL handling with proper try-catch
   - Timeout protection on scraping (8 seconds)
   - Content size limits (3000 chars max)

4. **Web Scraping Safety**
   - User-Agent header to prevent blocking
   - Removes potentially dangerous tags (script, style)
   - BeautifulSoup handles malformed HTML gracefully
   - Timeout prevents hanging requests

5. **Dependencies**
   - All packages from official PyPI
   - Version pinning for reproducibility
   - No vulnerable transitive dependencies known
   - Pydantic for data validation

### ⚠️ Recommendations for Production

1. **Rate Limiting**
   ```python
   # Add rate limiting on web_search and scrape_url
   from tenacity import rate_limit
   ```

2. **Request Timeout Enhancement**
   ```python
   # Increase timeout for slower servers
   resp = requests.get(url, timeout=15, headers={"User-Agent": "Mozilla/5.0"})
   ```

3. **Content Security**
   ```python
   # Add HTML sanitization for rendered output
   from bleach import clean
   output = clean(scraped_content)
   ```

4. **API Rate Limits**
   ```python
   # Monitor Tavily API usage
   # Set query limits per minute
   # Implement fallback search providers
   ```

5. **Error Logging**
   ```python
   # Use structured logging instead of print()
   import logging
   logger = logging.getLogger(__name__)
   logger.error(f"Failed to scrape {url}: {str(e)}")
   ```

6. **Database for Results** (Optional)
   - Store research reports in PostgreSQL/MongoDB
   - Implement user authentication
   - Add request rate limiting per user

## 🚀 Installation & Setup

### Prerequisites
- Python 3.8+
- OpenAI API key
- Tavily API key

### Step 1: Clone Repository
```bash
git clone https://github.com/pushpakrai/Multi-agent-research.git
cd Multi-agent-research
```

### Step 2: Create Virtual Environment
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### Step 3: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 4: Configure Environment Variables
```bash
# Create .env file
echo OPENAI_API_KEY=your_openai_key_here > .env
echo TAVILY_API_KEY=your_tavily_key_here >> .env
```

**Get API Keys:**
- **OpenAI**: https://platform.openai.com/api-keys
- **Tavily**: https://tavily.com/

### Step 5: Run Application
```bash
streamlit run app.py
```

Application will be available at: `http://localhost:8501`

## 📖 Usage Examples

### Basic Research Query
```
Topic: "Latest advances in quantum computing 2025"
```

### Technical Topic
```
Topic: "CRISPR gene therapy applications in cancer treatment"
```

### Trend Research
```
Topic: "AI safety regulations and governance in 2025"
```

## 🔄 Pipeline Flow Example

```
User Input: "What is the latest on fusion energy?"
    ↓
[STEP 1] Search Agent (Tavily)
    ├─ Query: "What is the latest on fusion energy?"
    ├─ Results: 5 web results with titles, URLs, snippets
    └─ Output: Raw search results
    ↓
[STEP 2] Reader Agent (BeautifulSoup)
    ├─ Selects most relevant URL from search results
    ├─ Scrapes content (max 3000 chars)
    └─ Output: Extracted article content
    ↓
[STEP 3] Writer Chain (GPT-4o-mini)
    ├─ Input: Search results + Scraped content
    ├─ Process: Generate professional report
    └─ Output: Markdown report with:
        - Introduction
        - 3+ Key Findings
        - Conclusion
        - Sources cited
    ↓
[STEP 4] Critic Chain (GPT-4o-mini)
    ├─ Input: Generated report
    ├─ Process: Evaluate quality
    └─ Output: Score/10 + Feedback
    ↓
Download Report (.md)
```

## 🎨 UI Features

- **Dark Theme**: Easy on the eyes for long research sessions
- **Real-Time Status**: Watch each agent work in real-time
- **Pipeline Visualization**: 4-step progress cards
- **Color Coding**: Orange (running), Green (completed), Gray (waiting)
- **Responsive Design**: Works on desktop and tablet
- **Fast Results Display**: Expandable sections for detailed exploration

## 📊 Model & API Information

| Component | Model/Service | Purpose |
|-----------|---|---|
| Search | Tavily API | Reliable web search |
| Reader | BeautifulSoup + Requests | Web scraping |
| LLM | GPT-4o-mini | Report writing & criticism |
| UI | Streamlit | Web interface |

## 🔧 Advanced Configuration

### Modify Model
Edit `agents.py`:
```python
llm = ChatOpenAI(model="gpt-4-turbo", temperature=0.1)
```

### Adjust Search Results
Edit `tools.py`:
```python
results = tavily.search(query=query, max_results=10)  # Changed from 5
```

### Customize Prompts
Edit `agents.py` to modify `writer_prompt` or `critic_prompt`:
```python
writer_prompt = ChatPromptTemplate.from_messages([
    ("system", "Your custom system prompt here"),
    ("human", """Your custom human prompt here..."""),
])
```

## 📈 Performance Metrics

- **Average Pipeline Runtime**: 30-60 seconds (varies by network)
- **Search Results**: ~5 per query (Tavily)
- **Scrape Depth**: 3000 character limit per URL
- **Report Length**: 1000-2000 words typically
- **Model Temperature**: 0 (deterministic output)

## 🐛 Troubleshooting

### Issue: "API key not found"
**Solution**: Ensure `.env` file exists in project root with valid keys

### Issue: "Failed to scrape URL"
**Solution**: Website may block scraping; Tavily includes anti-blocking headers

### Issue: "timeout=8" error
**Solution**: Increase timeout in `tools.py` to 15 seconds for slow websites

### Issue: Streamlit not loading
**Solution**: Run `pip install --upgrade streamlit` and clear cache

## 📝 License

MIT License - See LICENSE file for details

## 👨‍💻 Author

**Pushpak Rai**
- GitHub: [@pushpakrai](https://github.com/pushpakrai)
- Repository: [Multi-agent-research](https://github.com/pushpakrai/Multi-agent-research)

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit changes (`git commit -am 'Add improvement'`)
4. Push to branch (`git push origin feature/improvement`)
5. Open a Pull Request

## 📞 Support & Issues

Found a bug or have a feature request?
- Open an issue on GitHub
- Check existing issues first
- Provide detailed reproduction steps

## 🔗 Resources

- **LangChain Documentation**: https://python.langchain.com/
- **Streamlit Documentation**: https://docs.streamlit.io/
- **OpenAI API Docs**: https://platform.openai.com/docs/
- **Tavily Search API**: https://tavily.com/api
- **BeautifulSoup Guide**: https://www.crummy.com/software/BeautifulSoup/

## 📚 Related Projects

- [LangChain](https://github.com/langchain-ai/langchain)
- [Streamlit](https://github.com/streamlit/streamlit)
- [OpenAI Python SDK](https://github.com/openai/openai-python)

---

**Made with ❤️ using LangChain, Streamlit, and OpenAI**

*Last Updated: May 2025*
