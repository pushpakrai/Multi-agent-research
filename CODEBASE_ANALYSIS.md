# Codebase Analysis & Architecture Deep Dive

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Diagram](#architecture-diagram)
3. [File-by-File Analysis](#file-by-file-analysis)
4. [Data Flow](#data-flow)
5. [Security Assessment](#security-assessment)
6. [Performance Analysis](#performance-analysis)
7. [Dependencies & Technology Stack](#dependencies--technology-stack)

---

## Project Overview

**ResearchMind** is a sophisticated multi-agent AI system that leverages:
- **LangChain** for agent orchestration
- **Streamlit** for interactive web UI
- **OpenAI GPT-4o-mini** for language processing
- **Tavily API** for web search
- **BeautifulSoup4** for web scraping

The system conducts automated research through four sequential stages, each handled by a specialized AI agent.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                   STREAMLIT WEB INTERFACE                       │
│                          (app.py)                               │
│  ┌──────────────────┐              ┌──────────────────┐        │
│  │  User Input UI   │              │ Pipeline Monitor │        │
│  │                  │              │                  │        │
│  │ - Topic input    │              │ - Step cards     │        │
│  │ - Submit button  │              │ - Status updates │        │
│  │ - Example chips  │              │ - Real-time view │        │
│  └────────┬─────────┘              └──────────────────┘        │
└───────────┼──────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────┐
│              RESEARCH PIPELINE ORCHESTRATOR                      │
│                      (pipeline.py)                              │
│  run_research_pipeline(topic) → dict                            │
│  ├─ State management                                             │
│  ├─ Error handling                                               │
│  └─ Result aggregation                                           │
└──┬──────┬──────────┬───────────┬──────────────────────────────────┘
   │      │          │           │
   ▼      ▼          ▼           ▼
┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐
│ STEP │ │ STEP │ │ STEP │ │  STEP   │
│  1   │ │  2   │ │  3   │ │    4    │
└──────┘ └──────┘ └──────┘ └──────────┘
  │        │        │          │
  ▼        ▼        ▼          ▼
┌──────────────────────────────────────────────────────────────────┐
│                     AGENTS & CHAINS LAYER                        │
│                        (agents.py)                              │
│                                                                  │
│ ┌─────────────────┐  ┌─────────────────┐                        │
│ │ Search Agent    │  │  Reader Agent   │                        │
│ │ - create_agent()│  │ - create_agent()│                        │
│ │ - tools: [web_s │  │ - tools: [scrape│                        │
│ │   earch]        │  │   _url]         │                        │
│ │ - llm: GPT-4o   │  │ - llm: GPT-4o   │                        │
│ └────────┬────────┘  └────────┬────────┘                        │
│          │                     │                                │
│ ┌─────────────────┐  ┌─────────────────┐                        │
│ │ Writer Chain    │  │ Critic Chain    │                        │
│ │ - ChatPrompt    │  │ - ChatPrompt    │                        │
│ │   Template      │  │   Template      │                        │
│ │ - llm pipe      │  │ - llm pipe      │                        │
│ │   StrOutput     │  │   StrOutput     │                        │
│ └────────┬────────┘  └────────┬────────┘                        │
│          │                     │                                │
└──────────┼─────────────────────┼────────────────────────────────┘
           │                     │
           └──────────┬──────────┘
                      │
                      ▼
         ┌────────────────────────────┐
         │    TOOLS LAYER             │
         │      (tools.py)            │
         │                            │
         │ ┌──────────────────────┐   │
         │ │  web_search(query)   │   │
         │ │  - Tavily API        │   │
         │ │  - LangChain @tool   │   │
         │ │  - Returns: title,   │   │
         │ │    URL, snippet      │   │
         │ └──────────────────────┘   │
         │                            │
         │ ┌──────────────────────┐   │
         │ │  scrape_url(url)     │   │
         │ │  - BeautifulSoup4    │   │
         │ │  - LangChain @tool   │   │
         │ │  - Returns: clean    │   │
         │ │    text (3000 chars) │   │
         │ └──────────────────────┘   │
         └────────────────────────────┘
                      │
                      ▼
         ┌────────────────────────────┐
         │   EXTERNAL SERVICES        │
         │                            │
         │ - Tavily Web Search API    │
         │ - OpenAI GPT-4o-mini       │
         │ - Public Web (for scraping)│
         └────────────────────────────┘
```

---

## File-by-File Analysis

### 1. **app.py** (330+ lines)

#### Purpose
Provides the interactive Streamlit web interface for the research system.

#### Key Sections

**A. Page Configuration**
```python
st.set_page_config(
    page_title="ResearchMind · AI Research Agent",
    page_icon="🔬",
    layout="wide",
    initial_sidebar_state="collapsed",
)
```

**B. Custom CSS Styling** (Glassmorphism Design)
- **Fonts**: Syne, DM Mono, DM Sans
- **Colors**: Orange (#ff8c32), Green (#50c878), Dark (#0a0a0f)
- **Components**: Input cards, step cards, result panels
- **Effects**: Radial gradients, blur effects, hover animations

**C. Helper Functions**
```python
def step_card(num: str, title: str, state: str, desc: str = ""):
    # Renders pipeline step visualization
```

**D. Session State Management**
- Persists state across reruns
- Tracks results, running status, completion

**E. Pipeline Execution**
- 4-step workflow with spinners
- Real-time progress updates

**F. Results Display**
- Expandable sections for outputs
- Markdown rendering
- Download button

#### Strengths
- Professional dark theme
- Real-time visualization
- Responsive design
- Good error handling

---

### 2. **agents.py** (42 lines)

#### Components

**LLM Configuration**
```python
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
```

**Search Agent** - Web information gathering
**Reader Agent** - Content extraction
**Writer Chain** - Report synthesis
**Critic Chain** - Quality assurance

#### Architecture Pattern
Uses **LangChain Expression Language (LCEL)**:
```
Prompt Template → LLM → Output Parser
```

---

### 3. **pipeline.py** (52 lines)

#### Main Function: `run_research_pipeline(topic: str) → dict`

**4-Step Workflow:**
1. Search Agent - Gathers web information
2. Reader Agent - Scrapes top resources
3. Writer Chain - Synthesizes report
4. Critic Chain - Reviews & scores

#### Data Flow
```
Topic → Search → Reader → Writer → Critic → Results
```

---

### 4. **tools.py** (32 lines)

#### Tool 1: `web_search(query)`
- **API**: Tavily Search
- **Features**: 5 results, anti-bot blocking
- **Output**: Title, URL, snippet

#### Tool 2: `scrape_url(url)`
- **Library**: BeautifulSoup4
- **Features**: 8s timeout, tag removal, 3000 char limit
- **Safety**: Error handling, no script injection

---

### 5. **requirements.txt** (30 lines)

Key dependencies:
- **langchain**: Agent framework
- **openai**: GPT-4o-mini access
- **tavily-python**: Web search
- **streamlit**: UI framework
- **beautifulsoup4**: HTML parsing
- **pydantic**: Data validation

---

## Data Flow

```
User Input → Validate → Step 1: Search
→ Step 2: Reader → Step 3: Writer → Step 4: Critic
→ Update UI → Display Results → Download Option
```

---

## Security Assessment

✅ **Implemented:**
- API keys in .env/.gitignore
- Input validation
- Timeout protection (8s)
- HTML tag removal
- Error handling

⚠️ **Recommendations:**
- Rate limiting
- URL validation
- Structured logging
- Database persistence
- User authentication

---

## Performance Analysis

### Execution Time
- Step 1 (Search): 2-5s
- Step 2 (Reader): 3-8s
- Step 3 (Writer): 5-15s
- Step 4 (Critic): 3-8s
- **Total**: ~20-25 seconds

### Token Usage
- **Per Report**: 2,000-3,000 tokens
- **Cost**: ~$0.01-0.02
- **Monthly** (100 queries): ~$1-2

---

## Dependencies & Technology Stack

```
LangChain (Orchestration)
├─ OpenAI (LLM)
├─ Tavily (Search)
├─ BeautifulSoup4 (Scraping)
└─ Streamlit (UI)
```

---

**Technology**: Python, LangChain, Streamlit, OpenAI
**Last Updated**: May 2025
