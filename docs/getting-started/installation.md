# Instalação

Este guia irá ajudá-lo a configurar o ambiente de desenvolvimento do Klike AI Services.

---

## Pré-requisitos

Antes de começar, certifique-se de ter instalado:

| Ferramenta | Versão Mínima | Download |
|------------|---------------|----------|
| **Python** | 3.10+ | [python.org](https://www.python.org/downloads/) |
| **FFmpeg** | 4.4+ | [ffmpeg.org](https://ffmpeg.org/download.html) |
| **Git** | 2.30+ | [git-scm.com](https://git-scm.com/downloads) |

### Verificar Instalação

```bash
python --version  # Python 3.10.0 ou superior
ffmpeg -version   # ffmpeg version 4.4 ou superior
git --version     # git version 2.30 ou superior
```

---

## Instalação Rápida

### 1. Clone o Repositório

```bash
git clone https://github.com/klike-ai/klike-ai-services.git
cd klike-ai-services
```

### 2. Crie um Ambiente Virtual

=== "Windows"
    ```powershell
    python -m venv env
    .\env\Scripts\activate
    ```

=== "Linux/macOS"
    ```bash
    python3 -m venv env
    source env/bin/activate
    ```

!!! tip "Por que usar ambiente virtual?"
    Ambientes virtuais isolam as dependências do projeto, evitando conflitos com outras instalações Python no seu sistema.

### 3. Instale as Dependências

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

!!! info "Tempo de Instalação"
    A instalação pode levar 5-10 minutos dependendo da sua conexão de internet, pois inclui bibliotecas como OpenCV, FFmpeg-python, e LangChain.

---

### 2. Instale Dependências de Desenvolvimento

```bash
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Se existir
```

### 3. Instale o FFmpeg

=== "Windows"
    **Opção 1: Chocolatey** (Recomendado)
    ```powershell
    choco install ffmpeg
    ```
    
    **Opção 2: Manual**
    1. Baixe de [ffmpeg.org/download.html](https://ffmpeg.org/download.html)
    2. Extraia para `C:\ffmpeg`
    3. Adicione `C:\ffmpeg\bin` ao PATH

=== "macOS"
    ```bash
    brew install ffmpeg
    ```

=== "Ubuntu/Debian"
    ```bash
    sudo apt update
    sudo apt install ffmpeg
    ```

=== "CentOS/RHEL"
    ```bash
    sudo yum install ffmpeg
    ```

### 4. Verificar Instalação do FFmpeg

```bash
ffmpeg -version
```

Você deve ver algo como:

```
ffmpeg version 4.4.2 Copyright (c) 2000-2021 the FFmpeg developers
built with gcc 11.2.0 (GCC)
configuration: --enable-gpl --enable-version3 ...
```

---

## Instalação com Docker

!!! warning "Em Desenvolvimento"
    O suporte completo ao Docker está em desenvolvimento. Use a instalação manual por enquanto.

```dockerfile
# Dockerfile (exemplo futuro)
FROM python:3.10-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
# Construir imagem
docker build -t klike-ai-services .

# Executar container
docker run -p 8000:8000 --env-file .env klike-ai-services
```

---

##  Dependências Principais

### Core Dependencies

| Pacote | Versão | Propósito |
|--------|--------|-----------|
| **fastapi** | 0.104+ | Framework web assíncrono |
| **uvicorn** | 0.24+ | Servidor ASGI |
| **google-genai** | 1.0+ | Cliente Google Gemini |
| **langchain-openai** | 0.0.2+ | Integração OpenAI |
| **boto3** | 1.28+ | SDK AWS (S3, Rekognition) |
| **opencv-python** | 4.8+ | Processamento de vídeo |
| **ffmpeg-python** | 0.2+ | Manipulação de vídeo |
| **supabase** | 2.0+ | Storage de frames |

### Dependências Opcionais

```bash
# Para desenvolvimento
pip install pytest pytest-cov black isort flake8

# Para documentação
pip install mkdocs mkdocs-material
```

---

##  Verificação da Instalação

Execute os testes básicos para garantir que tudo está funcionando:

### 1. Verificar Importações

```python
python -c "
import fastapi
import google.genai
import boto3
import cv2
import langchain_openai
print(' Todas as dependências instaladas com sucesso!')
"
```

### 2. Iniciar o Servidor

```bash
uvicorn app.app:app --reload
```

Você deve ver:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using StatReload
INFO:     Started server process [12346]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

### 3. Testar a API

Abra [http://localhost:8000/docs](http://localhost:8000/docs) no navegador.

Você deve ver a interface Swagger UI interativa.

---

##  Problemas Comuns

### Erro: `ModuleNotFoundError: No module named 'cv2'`

**Solução:**
```bash
pip uninstall opencv-python opencv-python-headless
pip install opencv-python==4.8.1.78
```

### Erro: `FFmpeg not found`

**Solução Windows:**
```powershell
# Adicione ao PATH manualmente
$env:Path += ";C:\ffmpeg\bin"
```

**Solução Linux/macOS:**
```bash
export PATH=$PATH:/usr/local/bin/ffmpeg
```

### Erro: `botocore.exceptions.NoCredentialsError`

**Solução:**
Configure as credenciais AWS no `.env` (veja [Configuração](configuration.md))

### Erro: `google.api_core.exceptions.Unauthenticated`

**Solução:**
Verifique se a `GOOGLE_API_KEY` está correta no `.env`

---

##  Atualização

Para atualizar para a versão mais recente:

```bash
git pull origin main
pip install --upgrade -r requirements.txt
```

---

##  Próximos Passos

Agora que a instalação está completa:

1. [Configure as variáveis de ambiente](configuration.md)
2. [Faça seu primeiro request](quickstart.md)
3. [Explore a API Reference](../api/overview.md)

---
