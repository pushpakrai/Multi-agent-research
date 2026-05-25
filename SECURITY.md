# Security Best Practices & Environment Setup

## 🔐 Environment Variables Setup

### Required API Keys

1. **OpenAI API Key**
   - Get from: https://platform.openai.com/api-keys
   - Format: `sk-...`
   - Usage: GPT-4o-mini model for writing and criticism

2. **Tavily API Key**
   - Get from: https://tavily.com
   - Format: `tvly-...`
   - Usage: Web search functionality

### .env File Template

Create a `.env` file in the project root:

```bash
# OpenAI Configuration
OPENAI_API_KEY=sk-your_openai_key_here

# Tavily Search API
TAVILY_API_KEY=tvly-your_tavily_key_here

# Optional: Model Configuration
OPENAI_MODEL=gpt-4o-mini
OPENAI_TEMPERATURE=0
```

**⚠️ CRITICAL: NEVER commit .env file to Git!**
- It's already in `.gitignore`
- Always keep API keys local and secure
- Use different keys for dev/prod if possible

## 🛡️ Security Checklist

### Before Deploying to Production

- [ ] API keys are in `.env` and `.env` is in `.gitignore`
- [ ] No API keys appear in any Python files
- [ ] Implement rate limiting on API calls
- [ ] Add request timeout protection (currently 8s, consider 15s)
- [ ] Enable HTTPS for any web endpoints
- [ ] Add user authentication if multi-tenant
- [ ] Set up API key rotation policy
- [ ] Monitor API usage for unusual patterns
- [ ] Add request logging and monitoring
- [ ] Implement database for result persistence
- [ ] Add input validation and sanitization
- [ ] Use environment-specific configs

## 📊 API Usage Monitoring

### Tavily Search
```python
# Monitor API calls
from functools import wraps
import time

call_count = 0
call_timestamps = []

def rate_limit_check(max_calls=10, time_window=60):
    global call_count, call_timestamps
    now = time.time()
    call_timestamps = [t for t in call_timestamps if now - t < time_window]
    if len(call_timestamps) >= max_calls:
        raise Exception("Rate limit exceeded")
    call_timestamps.append(now)
```

### OpenAI API
```python
# Check token usage
from langchain.callbacks import get_openai_callback

with get_openai_callback() as cb:
    result = llm.invoke("prompt")
    print(f"Tokens used: {cb.total_tokens}")
    print(f"Cost: ${cb.total_cost}")
```

## 🚨 Common Security Issues & Solutions

### Issue: Exposed API Key in Git History
**Solution**: 
```bash
# If accidentally committed:
git filter-branch --tree-filter 'rm -f .env' HEAD
# Rotate the exposed key immediately
```

### Issue: Unsanitized User Input
**Current**: Topic input validated but not sanitized
**Recommendation**:
```python
from html import escape
topic = escape(st.text_input("Research Topic"))
```

### Issue: No Request Validation
**Recommendation**:
```python
from pydantic import BaseModel, validator

class ResearchRequest(BaseModel):
    topic: str
    
    @validator('topic')
    def topic_not_empty(cls, v):
        if not v.strip() or len(v) > 500:
            raise ValueError('Topic must be 1-500 chars')
        return v.strip()
```

### Issue: Verbose Error Messages
**Current**: Some errors show full exception details
**Recommendation**:
```python
try:
    # operation
except Exception as e:
    logger.error(f"Operation failed: {str(e)}")
    st.error("An error occurred. Please try again.")
```

## 🔐 Secrets Management Options

### Option 1: Environment Variables (Current)
- Simple to use
- Good for local development
- Less ideal for production

### Option 2: GitHub Secrets (Recommended for GitHub Actions)
```yaml
# .github/workflows/deploy.yml
env:
  OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
  TAVILY_API_KEY: ${{ secrets.TAVILY_API_KEY }}
```

### Option 3: AWS Secrets Manager / HashiCorp Vault
For production deployment:
```python
import boto3
secrets_client = boto3.client('secretsmanager')
secret = secrets_client.get_secret_value(SecretId='openai-key')
api_key = secret['SecretString']
```

## 🔗 Network Security

### HTTPS Configuration
If deploying with Streamlit Cloud:
```bash
# Streamlit handles HTTPS automatically
# For custom deployments:
# nginx reverse proxy with SSL certificates
```

### CORS Configuration
```python
# If adding API endpoints
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_methods=["POST"],
    allow_headers=["Content-Type"],
)
```

## 📝 Audit & Logging

### Recommended Logging Setup
```python
import logging
import json
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('research_app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

# Log research requests
def log_research_request(topic, results):
    logger.info(json.dumps({
        'timestamp': datetime.now().isoformat(),
        'action': 'research_completed',
        'topic': topic,
        'status': 'success'
    }))
```

## 🎯 Deployment Security Checklist

### Pre-Deployment
- [ ] All dependencies up to date (`pip list --outdated`)
- [ ] No hardcoded credentials
- [ ] Secrets in environment only
- [ ] Error handling complete
- [ ] Logging configured
- [ ] Tests passing
- [ ] Security audit complete

### Post-Deployment
- [ ] Monitor API usage
- [ ] Check logs regularly
- [ ] Set up alerts for errors
- [ ] Track cost/usage
- [ ] Validate HTTPS working
- [ ] Test rate limiting
- [ ] Document incidents

## 📞 Security Contact

If you discover a security vulnerability:
1. **DO NOT** open a public issue
2. Email security details to repository maintainer
3. Provide reproduction steps
4. Allow time for patch before disclosure

---

**Last Updated: May 2025**
