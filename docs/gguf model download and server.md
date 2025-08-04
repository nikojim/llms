# 🧠 Serve Mistral Q4 GGUF with `llama.cpp`, OpenBLAS, LangChain/RAG, HTTPS, and systemd (Ubuntu)

---

## ✅ Step 1: Install System Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  git \
  cmake \
  build-essential \
  libopenblas-dev \
  libcurl4-openssl-dev \
  aria2 \
  tmux \
  nginx \
  certbot \
  python3-certbot-nginx \
  python3-pip
```

---

## 🧱 Step 2: Clone and Build `llama.cpp` with CMake and BLAS

```bash
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

---

## 📦 Step 3: Download Mistral GGUF Model (Q4\_K\_M)

```bash
mkdir -p ~/models/mistral-gguf
cd ~/models/mistral-gguf

aria2c -x 16 -s 16 https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/resolve/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf
```

---

## 🚀 Step 4: Serve Model with `llama-server`

```bash
cd ~/llama.cpp/build/bin

./llama-server \
  --model ~/models/mistral-gguf/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --ctx-size 4096 \
  --threads $(nproc) \
  --mlock \
  --blas
```

---

## 🧪 Step 5: Test the API

```bash
curl -X POST http://localhost:8080/completion \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the capital of France?", "n_predict": 64}'
```

---

## 🧱 Option 1: 🧩 Connect to LangChain or RAG

### Install LangChain and Dependencies:

```bash
pip install langchain openai requests
```

### Example Python Script:

```python
from langchain.llms import LlamaCpp
llm = LlamaCpp(
    model_path="~/models/mistral-gguf/mistral-7b-instruct-v0.2.Q4_K_M.gguf",
    n_ctx=4096,
    n_threads=8,
    temperature=0.7,
    use_mlock=True
)
print(llm("Explain what a neural network is."))
```

> You can also use the remote server via HTTP by creating a custom `LLM` class in LangChain or using `requests`.

---

## 🔐 Option 2: Secure API with HTTPS and JWT Auth

### A. Setup HTTPS with Nginx + Certbot

1. Point your domain (e.g., `llm.example.com`) to your server IP.
2. Configure Nginx reverse proxy:

```bash
sudo nano /etc/nginx/sites-available/llm
```

```nginx
server {
    listen 80;
    server_name llm.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/llm /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

3. Add SSL:

```bash
sudo certbot --nginx -d llm.example.com
```

### B. Add JWT Authentication (optional)

Wrap with a reverse proxy like `FastAPI` or `Flask` that validates JWT before forwarding to `llama-server`. Let me know if you want this coded.

---

## ⚙️ Option 3: Run `llama-server` as a `systemd` Service

### A. Create the systemd unit:

```bash
sudo nano /etc/systemd/system/llama-server.service
```

```ini
[Unit]
Description=Llama.cpp Mistral Server
After=network.target

[Service]
User=ubuntu
ExecStart=/home/ubuntu/llama.cpp/build/bin/llama-server \
  --model /home/ubuntu/models/mistral-gguf/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --ctx-size 4096 \
  --threads $(nproc) \
  --mlock \
  --blas
Restart=always
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
```

### B. Enable and start:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable llama-server
sudo systemctl start llama-server
sudo systemctl status llama-server
```

---

## ✅ Summary

| Feature                     | Enabled |
| --------------------------- | ------- |
| OpenBLAS optimization       | ✅       |
| Mistral Q4 GGUF inference   | ✅       |
| HTTP API via `llama-server` | ✅       |
| LangChain or local RAG      | ✅       |
| HTTPS secured access        | ✅       |
| JWT-ready architecture      | ✅       |
| Persistent systemd service  | ✅       |

---

Let me know if you want:

* A working Flask/FastAPI JWT wrapper
* LangChain RAG with FAISS integration
* Whisper/Vosk voice input for your LLM server
