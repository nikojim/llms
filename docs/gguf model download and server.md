# 🧠 Final Full Guide: Serve Mistral Q4 GGUF with OpenAI-Compatible API via `llama-cpp-python` + `llama.cpp` Build (Ubuntu)

---

## ✅ Step 1: Install Required System Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  git \
  build-essential \
  cmake \
  libopenblas-dev \
  python3-pip \
  python3-venv \
  aria2 \
  nginx \
  certbot \
  python3-certbot-nginx \
  tmux
```

---

## 🧱 Step 2: Clone and Build `llama.cpp` with OpenBLAS

```bash
cd ~
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
mkdir build && cd build

cmake .. \
  -DLLAMA_BUILD_SERVER=ON \
  -DLLAMA_CURL=ON \
  -DLLAMA_BLAS=ON \
  -DLLAMA_BLAS_VENDOR=OpenBLAS

cmake --build . --config Release
```

> ✅ This builds optimized C++ binaries with BLAS for CPU performance.
> Optional if you want to run via CLI, benchmarking, or raw `llama-server`.

---

## 🐍 Step 3: Create and Activate Python Virtual Environment

```bash
python3 -m venv ~/llama-env
source ~/llama-env/bin/activate
```

---

## 🧩 Step 4: Install `llama-cpp-python` with OpenBLAS & Server Support

```bash
pip install --upgrade pip

CMAKE_ARGS="-DLLAMA_BLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" \
pip install llama-cpp-python[server]
```

---

## 📦 Step 5: Download and Rename Mistral Q4 GGUF Model

```bash
mkdir -p ~/models/mistral-gguf
cd ~/models/mistral-gguf

aria2c -x 16 -s 16 \
  -o mistral-q4.gguf \
  https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/resolve/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf
```

---

## 🚀 Step 6: Serve Model with OpenAI-Compatible API (via Python)

```bash
source ~/llama-env/bin/activate

python -m llama_cpp.server \
  --model ~/models/mistral-gguf/mistral-q4.gguf \
  --host 0.0.0.0 \
  --port 8000 \
  --n_ctx 4096 \
  --n_threads $(nproc)
```

---

## 🧪 Step 7: Test the Server with `curl`

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral",
    "messages": [{"role": "user", "content": "What is quantum computing?"}],
    "temperature": 0.7
  }'
```

---

## 🧠 Step 8: Use with `ChatOpenAI` from LangChain

```bash
pip install langchain openai

from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage

llm = ChatOpenAI(
    base_url="http://localhost:8000/v1",  # or your public HTTPS endpoint
    api_key="not-needed",                 # required by class, not used here
    model="mistral"
)

response = llm([HumanMessage(content="Summarize the theory of evolution.")])
print(response.content)
```

---

## 🔐 Step 9 (Optional): Secure with Nginx + HTTPS

### A. Create Reverse Proxy

```bash
sudo nano /etc/nginx/sites-available/mistral
```

```nginx
server {
    listen 80;
    server_name llm.example.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/mistral /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### B. Add HTTPS

```bash
sudo certbot --nginx -d llm.example.com
```

---

## ⚙️ Step 10 (Optional): Create a `systemd` Service

```bash
sudo nano /etc/systemd/system/mistral.service
```

```ini
[Unit]
Description=Mistral GGUF Server (llama-cpp-python)
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/llama-env/bin/python -m llama_cpp.server \
  --model /home/ubuntu/models/mistral-gguf/mistral-q4.gguf \
  --host 0.0.0.0 \
  --port 8000 \
  --n_ctx 4096 \
  --n_threads 4
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable mistral
sudo systemctl start mistral
sudo systemctl status mistral
```

---

## ✅ Final Summary

| Step                                   | Status |
| -------------------------------------- | ------ |
| System dependencies installed          | ✅      |
| `llama.cpp` built with OpenBLAS        | ✅      |
| Python venv (PEP 668-safe)             | ✅      |
| `llama-cpp-python[server]` installed   | ✅      |
| Mistral Q4 GGUF downloaded/renamed     | ✅      |
| OpenAI-compatible API served           | ✅      |
| Tested with curl & LangChain           | ✅      |
| HTTPS (via Nginx + Certbot) (optional) | ✅      |
| systemd service (optional)             | ✅      |

---

