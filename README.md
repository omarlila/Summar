# 💬 RAG-Powered Chat Server with Memory & Supabase Logging

This project is a FastAPI-based server for an advanced chat assistant that uses RAG (Retrieval-Augmented Generation), conversation memory with Redis, and Supabase for persistent storage. It supports English and Arabic and integrates ChromaDB for document retrieval and SentenceTransformers for embedding.

## 🚀 Features

- 🔥 RAG pipeline using ChromaDB and `jinaai/jina-clip-v2`
- 💾 Short- and long-term memory using Redis
- 📚 Supabase integration for chat and conversation logs
- 🌐 Arabic & English language detection
- 🤖 Automatic model selection and categorization
- 📝 Summarization of long conversation histories
- 🔗 Optional ngrok tunneling for public exposure
- 🧠 Embedding via `SentenceTransformer` (local)
- 📎 Session management & caching

## 🛠️ Technologies Used

- [FastAPI](https://fastapi.tiangolo.com/)
- [Redis](https://redis.io/)
- [ChromaDB](https://www.trychroma.com/)
- [SentenceTransformers](https://www.sbert.net/)
- [Supabase](https://supabase.com/)
- [Ollama](https://ollama.com/)
- [pyngrok](https://github.com/alexdlaird/pyngrok)
- [LangDetect](https://pypi.org/project/langdetect/)

## 📋 Prerequisites

Before installing and running the chat server, ensure you have the following components properly configured:

### 1. 🐳 Redis with Docker
Download and run Redis using Docker with persistent volume:

#### For Linux/macOS:
```bash
# Pull Redis image
docker pull redis:alpine

# Create a volume for Redis data persistence
docker volume create redis-data

# Run Redis container with volume mapping
docker run -d \
  --name redis-server \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:alpine redis-server --appendonly yes

# Verify Redis is running
docker ps
redis-cli ping  # Should return "PONG"
```

#### For Windows:
```cmd
REM Pull Redis image
docker pull redis:alpine

REM Create a volume for Redis data persistence
docker volume create redis-data

REM Run Redis container with volume mapping
docker run -d --name redis-server -p 6379:6379 -v redis-data:/data redis:alpine redis-server --appendonly yes

REM Verify Redis is running
docker ps
docker exec -it redis-server redis-cli ping
```

**Alternative for Windows (using PowerShell):**
```powershell
# Pull Redis image
docker pull redis:alpine

# Create a volume for Redis data persistence
docker volume create redis-data

# Run Redis container with volume mapping
docker run -d `
  --name redis-server `
  -p 6379:6379 `
  -v redis-data:/data `
  redis:alpine redis-server --appendonly yes

# Verify Redis is running
docker ps
docker exec -it redis-server redis-cli ping
```

**Redis Configuration:**
- Default port: `6379`
- Connection URL: `redis://localhost:6379/0`
- Data persistence enabled with volume mapping

### 2. 🌐 ngrok Tunnel Setup
Install and configure ngrok for public API exposure:

#### For Linux:
```bash
# Download ngrok (for Linux)
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update && sudo apt install ngrok
```

#### For macOS:
```bash
# For macOS with Homebrew
brew install ngrok/ngrok/ngrok
```

#### For Windows:
```cmd
REM Option 1: Download from website
REM Visit https://ngrok.com/download and download the Windows version
REM Extract the .exe file to a folder in your PATH

REM Option 2: Using Chocolatey (if installed)
choco install ngrok

REM Option 3: Using Scoop (if installed)
scoop install ngrok
```

**For Windows PowerShell (Manual Installation):**
```powershell
# Create a directory for ngrok
New-Item -ItemType Directory -Path "C:\ngrok" -Force

# Download ngrok (you can also visit https://ngrok.com/download)
Invoke-WebRequest -Uri "https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-windows-amd64.zip" -OutFile "C:\ngrok\ngrok.zip"

# Extract the zip file
Expand-Archive -Path "C:\ngrok\ngrok.zip" -DestinationPath "C:\ngrok"

# Add to PATH (run as Administrator)
$env:PATH += ";C:\ngrok"
[Environment]::SetEnvironmentVariable("PATH", $env:PATH, [EnvironmentVariableTarget]::Machine)
```

#### Authentication (All Platforms):
```bash
# Authenticate with your ngrok token
ngrok config add-authtoken YOUR_NGROK_AUTH_TOKEN

# Test ngrok installation
ngrok version
```

**Get your ngrok auth token:**
1. Sign up at [ngrok.com](https://ngrok.com)
2. Go to [Your Authtoken](https://dashboard.ngrok.com/get-started/your-authtoken)
3. Copy your authtoken and use it in the command above

### 3. 🤖 Ollama Installation & Model Setup
Install Ollama and download required models:

#### For Linux/macOS:
```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Start Ollama service
ollama serve
```

#### For Windows:
```cmd
REM Download Ollama from: https://ollama.com/download/windows
REM Run the installer (OllamaSetup.exe)
REM The installer will automatically add Ollama to your PATH

REM Start Ollama service (run in Command Prompt or PowerShell)
ollama serve

REM Or use the Ollama app from the Start menu
```

**Alternative Windows Installation:**
```powershell
# Using PowerShell (if you have package managers)
# With Chocolatey:
choco install ollama

# With Scoop:
scoop install ollama

# Start Ollama service
ollama serve
```

#### Verify Installation (All Platforms):
```bash
# Verify Ollama is running (in a new terminal/command prompt)
curl http://localhost:11434/api/version

# For Windows Command Prompt (if curl is not available):
# Use PowerShell: Invoke-RestMethod -Uri "http://localhost:11434/api/version"
```

**Download Required Models:**
Use the provided batch script to download all necessary models:

#### For Linux/macOS:
```bash
# Run the provided start_ollama.bat script from the repository
chmod +x start_ollama.bat
./start_ollama.bat
```

#### For Windows:
```cmd
REM Run the provided batch script from the repository
start_ollama.bat

REM Or double-click the start_ollama.bat file in Windows Explorer
```

**The script will automatically:**
- Start Ollama service
- Download smollm2:latest (English model)
- Download prakasharyan/qwen-arabic:latest (Arabic model)
- Verify all models are properly installed
- Test API endpoints for both models

**Note:** If you need to make custom changes to the models or add additional ones, modify the `start_ollama.bat` script in the repository accordingly.

**Verify Ollama API endpoints:**
```bash
# Test Ollama API health
curl http://localhost:11434/api/tags

# Test chat completion
curl -X POST http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smollm2:latest",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": false
  }'
```

### 4. 🧠 Embedding Model Prerequisites
The embedding model `jinaai/jina-clip-v2` will be automatically downloaded when first used. Ensure you have sufficient disk space (approximately 2-3GB).

### 5. 📊 Supabase Setup (Optional)
1. Create a project at [supabase.com](https://supabase.com)
2. Get your project URL and service role key
3. Create required tables (schema provided in `/database` folder)

## 📦 Installation

### 1. Clone the repository
```bash
git clone https://github.com/your-username/chat-rag-server.git
cd chat-rag-server
```

### 2. Create and activate virtual environment

#### For Linux/macOS:
```bash
python -m venv venv
source venv/bin/activate
```

#### For Windows (Command Prompt):
```cmd
python -m venv venv
venv\Scripts\activate
```

#### For Windows (PowerShell):
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1

# If you get execution policy error, run:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 3. Install Python dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables
Create a `.env` file in the root directory with the following configuration:

**Important:** Make sure all values align with your custom setup and the models downloaded by `start_ollama.bat`:

```env
# Ollama Configuration
OLLAMA_BASE_URL=http://localhost:11434
DEFAULT_MODEL=smollm2:latest
ARABIC_MODEL=prakasharyan/qwen-arabic:latest
EMBEDDINGS_MODEL=jinaai/jina-clip-v2

# Redis Configuration (Docker setup from prerequisites)
REDIS_URL=redis://localhost:6379/0

# Supabase Configuration (Optional - configure if using persistent logging)
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-service-role-key

# ngrok Configuration (Optional - set to false if not using public tunneling)
USE_NGROK=true
NGROK_AUTHTOKEN=your-ngrok-token

# Memory Configuration (Advanced settings)
SHORT_TERM_MEMORY_TTL=3600
LONG_TERM_MEMORY_TTL=2592000
MAX_CONVERSATION_LENGTH=10000
SIMILARITY_THRESHOLD=0.7

# ChromaDB Configuration
CHROMA_DB_PATH=./chroma_data_godic

# API Configuration
MAX_TOKENS_DEFAULT=2000
TEMPERATURE=0.7
TOP_P=0.9
```

**Configuration Notes:**
- If you modified the models in `start_ollama.bat`, update `DEFAULT_MODEL` and `ARABIC_MODEL` accordingly
- Set `USE_NGROK=false` if you don't want public URL exposure
- Adjust memory settings based on your system resources
- Ensure `REDIS_URL` matches your Docker Redis container port (default: 6379)
- `SUPABASE_URL` and `SUPABASE_KEY` are optional but required for persistent chat logging

## 🧪 Running the Server

### 1. Start prerequisite services

#### For Linux/macOS:
```bash
# Ensure Redis is running
docker start redis-server

# Ensure Ollama is running
ollama serve

# Verify all services are healthy
curl http://localhost:11434/api/version  # Ollama
redis-cli ping                           # Redis
```

#### For Windows (Command Prompt):
```cmd
REM Ensure Redis is running
docker start redis-server

REM Ensure Ollama is running (in a separate command prompt)
ollama serve

REM Verify all services are healthy
curl http://localhost:11434/api/version
docker exec -it redis-server redis-cli ping
```

#### For Windows (PowerShell):
```powershell
# Ensure Redis is running
docker start redis-server

# Ensure Ollama is running (in a separate PowerShell window)
ollama serve

# Verify all services are healthy
Invoke-RestMethod -Uri "http://localhost:11434/api/version"
docker exec -it redis-server redis-cli ping
```

### 2. Run the chat server

#### For Linux/macOS:
```bash
python server.py
```

#### For Windows:
```cmd
REM Command Prompt
python server.py

REM Or PowerShell
python server.py

REM Or if you have multiple Python versions
py -3 server.py
```

The server will start on `http://localhost:8000` with the following endpoints:
- **API Documentation:** `http://localhost:8000/docs`
- **Health Check:** `http://localhost:8000/api/health`
- **Chat Endpoint:** `POST http://localhost:8000/api/chat`

If ngrok is enabled, you'll also see a public URL in the console output.

## 🔧 Troubleshooting

### Common Issues & Solutions

#### Redis Connection Error:
**Linux/macOS:**
```bash
# Check if Redis container is running
docker ps | grep redis
# Restart if needed
docker restart redis-server
```

**Windows:**
```cmd
REM Check if Redis container is running
docker ps | findstr redis
REM Restart if needed
docker restart redis-server
```

#### Ollama Model Not Found:
**All Platforms:**
```bash
# List installed models
ollama list
# Pull missing model
ollama pull smollm2:latest
```

#### Port Already in Use:
**Linux/macOS:**
```bash
# Check what's using port 8000
lsof -i :8000
# Kill the process if needed
kill -9 <PID>
```

**Windows (Command Prompt):**
```cmd
REM Check what's using port 8000
netstat -ano | findstr :8000
REM Kill the process if needed (replace <PID> with actual process ID)
taskkill /PID <PID> /F
```

**Windows (PowerShell):**
```powershell
# Check what's using port 8000
Get-NetTCPConnection -LocalPort 8000
# Kill the process if needed
Stop-Process -Id <PID> -Force
```

#### ngrok Authentication Error:
**All Platforms:**
```bash
# Re-authenticate ngrok
ngrok config add-authtoken YOUR_TOKEN
```

#### Python Virtual Environment Issues (Windows):
```powershell
# If you get execution policy error when activating venv
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Then try activating again
.\venv\Scripts\Activate.ps1
```

#### Docker Permission Issues (Linux):
```bash
# Add your user to the docker group
sudo usermod -aG docker $USER
# Log out and log back in, or run:
newgrp docker
```

## 📌 Notes

- Embeddings and model inference are performed locally using CPU or CUDA
- Documents used for retrieval are stored in ChromaDB (`chroma_data_godic`)
- Conversations are cached, summarized, and reused for efficient performance
- Supabase stores all conversation and user metadata for long-term analytics

## 🧠 Memory Logic

- **Short-term Memory TTL:** 1 hour (stored in Redis)
- **Long-term Memory TTL:** 30 days (stored in Redis with embeddings)
- **Semantic Recall:** Uses cosine similarity for retrieving relevant past conversations
- **Auto-summarization:** Long conversations are automatically summarized to maintain context

## 🛡️ Authentication

Supabase JWT-based Bearer token required for:
- `/api/user-chats` - Get user's chat history
- `/api/chats/{session_id}` - Get specific chat session  
- `/api/chat-with-db` - Chat with database integration

## 📚 API Documentation

Once the server is running, visit `http://localhost:8000/docs` for interactive API documentation with Swagger UI.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
