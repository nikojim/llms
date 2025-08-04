Here is the **final updated guide** for serving a **Q4 GGUF Mistral model** using `llama.cpp` on an **Ubuntu server**, with:

* ✅ OpenBLAS optimization via CMake
* ✅ `llama-server` for HTTP API
* ✅ Secure HTTPS via Nginx + Certbot
* ✅ Optional JWT wrapper
* ✅ `systemd` integration
* ✅ Model download and renaming using `aria2c`

---

# 🧠 Serve Mistral Q4 GGUF with `llama.cpp`, OpenBLAS, HTTPS, systemd, and LangChain Support

---

## ✅ Step 1: Install Dependencies

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

## 🧱 Step 2: Clone and Build `llama.cpp` with CMake + OpenBLAS

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

> 🔧 Binaries will be available in `llama.cpp/build/bin/`

---

## 📦 Step 3: Download and Rename Mistral Model with `aria2c`

```bash
mkdir -p ~/models/mistral-gguf
cd ~/models/mistral-gguf

aria2c -x 16 -s 16 \
  -o mistral-q4.gguf \
  https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/resolve/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf
```

> ✅ This downloads the model and renames it to `mistral-q4.gguf`

---

## 🚀 Step 4: Serve the Model

```bash
cd ~/llama.cpp/build/bin

./llama-server \
  --model ~/models/mistral-gguf/mistral-q4.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --ctx-size 4096 \
  --threads $(nproc) \
  --mlock
```

> ⚠️ No `--blas` flag needed — it's already active via CMake if compiled correctly.

---

## 🧪 Step 5: Test the API

```bash
curl -X POST http://localhost:8080/completion \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is a neural network?", "n_predict": 64}'
```

---

## 🧩 Option 1: Use with LangChain or Local RAG

Install LangChain:

```bash
pip install langchain openai requests
```

Example script:

```python
from langchain.llms import LlamaCpp
llm = LlamaCpp(
    model_path="~/models/mistral-gguf/mistral-q4.gguf",
    n_ctx=4096,
    n_threads=8,
    temperature=0.7,
    use_mlock=True
)
print(llm("Explain transformers in AI."))
```

> ✅ You can also call the HTTP API using `requests` if needed.

---

## 🔐 Option 2: Secure API with HTTPS and JWT

### A. Nginx + Certbot HTTPS

1. Create Nginx config:

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

2. Enable and test:

```bash
sudo ln -s /etc/nginx/sites-available/llm /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

3. Add HTTPS with Certbot:

```bash
sudo certbot --nginx -d llm.example.com
```

### B. Add JWT Auth (optional)

Set up a reverse proxy using **FastAPI** or **Flask** that checks a JWT token before forwarding to `llama-server`. Let me know if you want this template.

---

## ⚙️ Option 3: Run `llama-server` as a `systemd` Service

### A. Create systemd unit:

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
  --model /home/ubuntu/models/mistral-gguf/mistral-q4.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --ctx-size 4096 \
  --threads $(nproc) \
  --mlock
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

| Feature                   | Enabled |
| ------------------------- | ------- |
| Mistral GGUF Q4 model     | ✅       |
| aria2c fast download      | ✅       |
| Renamed model file        | ✅       |
| OpenBLAS optimization     | ✅       |
| HTTP API (`llama-server`) | ✅       |
| LangChain compatibility   | ✅       |
| HTTPS via Nginx + Certbot | ✅       |
| JWT-ready API layer       | ✅       |
| systemd startup service   | ✅       |

---
