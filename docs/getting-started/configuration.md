#  Configuração

Este guia explica como configurar todas as variáveis de ambiente e integrações necessárias para o Klike AI Services.

---

##  Visão Geral

O Klike AI Services requer configuração de 4 serviços externos:

| Serviço | Propósito | Obrigatório |
|---------|-----------|-------------|
| **Google Gemini** | Análise de vídeo com IA | ✅ Sim |
| **OpenAI** | Geração de recomendações | ✅ Sim |
| **AWS** | S3 (storage) + Rekognition (OCR) | ✅ Sim |
| **Supabase** | Storage de frames extraídos | ✅ Sim |

---

##  Variáveis de Ambiente

### 1. Crie o arquivo `.env`

Na raiz do projeto, crie um arquivo `.env`:

```bash
touch .env  # Linux/macOS
# ou
New-Item .env  # Windows PowerShell
```

### 2. Template Completo

Copie e preencha com suas credenciais:

```bash
# ======================
# GOOGLE GEMINI API
# ======================
GOOGLE_API_KEY=AIzaSy...xxxxxxxxx

# ======================
# OPENAI API
# ======================
OPENAI_API_KEY=sk-proj-...xxxxxxxxx

# ======================
# AWS CREDENTIALS
# ======================
AWS_ACCESS_KEY_ID=AKIA...xxxxxxxxx
AWS_SECRET_ACCESS_KEY=wJalr...xxxxxxxxx
AWS_REGION=us-east-1
S3_BUCKET_NAME=klike-videos

# ======================
# SUPABASE
# ======================
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=eyJhb...xxxxxxxxx
SUPABASE_BUCKET=video-frames

```

!!! warning "Segurança"
    **NUNCA** commite o arquivo `.env` no Git! Ele já está no `.gitignore`.

---

##  Obter Credenciais

### Google Gemini API

1. Acesse [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Clique em **"Get API Key"**
3. Crie um novo projeto ou selecione existente
4. Copie a chave gerada

```bash
GOOGLE_API_KEY=AIzaSyBxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

!!! tip "Modelo Usado"
    O projeto usa o **Gemini 2.5 Pro** para análise de vídeo multimodal.

---

### OpenAI API

1. Acesse [OpenAI Platform](https://platform.openai.com/api-keys)
2. Faça login ou crie uma conta
3. Clique em **"Create new secret key"**
4. Dê um nome (ex: "Klike AI Services")
5. Copie a chave (ela só aparece uma vez!)

```bash
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

!!! info "Modelo Usado"
    O projeto usa o **GPT-4o-mini** para geração de recomendações via LangChain.

**Custo Estimado:**
- Análise de 1 vídeo ≈ $0.02 - $0.05 USD
- 100 vídeos/mês ≈ $2 - $5 USD

---

### AWS Credentials

#### 1. Criar Usuário IAM

1. Acesse [AWS IAM Console](https://console.aws.amazon.com/iam/)
2. Vá em **Users** → **Add users**
3. Nome: `klike-ai-services`
4. Selecione **Access key - Programmatic access**
5. Clique em **Next: Permissions**

#### 2. Configurar Permissões

Crie ou anexe uma policy com estas permissões:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::seu-bucket-name",
        "arn:aws:s3:::seu-bucket-name/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectText"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 3. Criar Bucket S3

```bash
aws s3 mb s3://klike-videos --region us-east-1
```

Ou via console:

1. Acesse [S3 Console](https://console.aws.amazon.com/s3/)
2. **Create bucket** → Nome: `klike-videos`
3. Region: `us-east-1`
4. **Block all public access**:  (desmarque)
5. **Create bucket**


#### 4. Estrutura do Bucket

O bucket deve seguir esta estrutura:

```
klike-videos/
├── mp4/          # Vídeos originais
├── mp3/          # Áudios extraídos (opcional)
├── outputs/      # Transcrições (futuro)
└── metadata/     # Metadados JSON (futuro)
```

#### 5. Configurar no `.env`

```bash
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_REGION=us-east-1
S3_BUCKET_NAME=klike-videos
```

---

### Supabase

#### 1. Criar Projeto

1. Acesse [Supabase Dashboard](https://app.supabase.com/)
2. **New project** → Nome: `klike-ai-services`
3. **Database Password**: Escolha uma senha forte
4. **Region**: Escolha a mais próxima
5. **Create new project** (leva ~2 min)

#### 2. Criar Bucket de Storage

1. Vá em **Storage** no menu lateral
2. **Create a new bucket**
3. Nome: `video-frames`
4. **Public bucket**:  (marque)
5. **Create bucket**

#### 3. Configurar Políticas de Acesso

No bucket `video-frames`, vá em **Policies** e adicione:

```sql
-- Permitir leitura pública
CREATE POLICY "Public Access"
ON storage.objects FOR SELECT
USING (bucket_id = 'video-frames');

-- Permitir upload de frames
CREATE POLICY "Allow Frame Upload"
ON storage.objects FOR INSERT
WITH CHECK (bucket_id = 'video-frames');
```

#### 4. Obter Credenciais

1. Vá em **Settings** → **API**
2. Copie:
   - **Project URL** → `SUPABASE_URL`
   - **Project API keys** → `anon` `public` → `SUPABASE_KEY`

```bash
SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS...
SUPABASE_BUCKET=video-frames
```

---

## Validar Configuração

Execute este script para validar todas as credenciais:

```python
import os
from dotenv import load_dotenv

load_dotenv()

def validate_config():
    required_vars = [
        "GOOGLE_API_KEY",
        "OPENAI_API_KEY",
        "AWS_ACCESS_KEY_ID",
        "AWS_SECRET_ACCESS_KEY",
        "AWS_REGION",
        "S3_BUCKET_NAME",
        "SUPABASE_URL",
        "SUPABASE_KEY",
        "SUPABASE_BUCKET"
    ]
    
    missing = []
    for var in required_vars:
        if not os.getenv(var):
            missing.append(var)
    
    if missing:
        print("Variáveis faltando:")
        for var in missing:
            print(f"  - {var}")
        return False
    
    print("Todas as variáveis configuradas!")
    
    # Testar conexões
    try:
        from google import genai
        client = genai.Client(api_key=os.getenv("GOOGLE_API_KEY"))
        print(" Google Gemini: Conectado")
    except Exception as e:
        print(f" Google Gemini: {e}")
    
    try:
        import boto3
        s3 = boto3.client('s3',
            aws_access_key_id=os.getenv("AWS_ACCESS_KEY_ID"),
            aws_secret_access_key=os.getenv("AWS_SECRET_ACCESS_KEY"),
            region_name=os.getenv("AWS_REGION")
        )
        s3.head_bucket(Bucket=os.getenv("S3_BUCKET_NAME"))
        print("AWS S3: Conectado")
    except Exception as e:
        print(f"AWS S3: {e}")
    
    try:
        from supabase import create_client
        supabase = create_client(os.getenv("SUPABASE_URL"), os.getenv("SUPABASE_KEY"))
        print("Supabase: Conectado")
    except Exception as e:
        print(f"Supabase: {e}")
    
    return True

if __name__ == "__main__":
    validate_config()
```

Salve como `validate_config.py` e execute:

```bash
python validate_config.py
```

---

##  Configuração por Ambiente

Para ambientes diferentes (dev, staging, prod):

```bash
# .env.development
GOOGLE_API_KEY=dev_key
S3_BUCKET_NAME=klike-videos-dev

# .env.production
GOOGLE_API_KEY=prod_key
S3_BUCKET_NAME=klike-videos-prod
```

Carregue o ambiente correto:

```python
from dotenv import load_dotenv
import os

env = os.getenv("ENV", "development")
load_dotenv(f".env.{env}")
```

---

##  Checklist Final

Antes de continuar, verifique:

- [ ] Arquivo `.env` criado na raiz do projeto
- [ ] Todas as 9 variáveis obrigatórias preenchidas
- [ ] Google Gemini API Key válida
- [ ] OpenAI API Key válida
- [ ] AWS Access Key e Secret válidas
- [ ] Bucket S3 criado e acessível
- [ ] Supabase projeto criado
- [ ] Bucket `video-frames` criado no Supabase
- [ ] Script `validate_config.py` executado com sucesso

---

##  Próximos Passos

Configuração concluída! Agora você pode:

1. ✅ [Fazer seu primeiro request](quickstart.md)
2. ✅ [Explorar a API Reference](../api/overview.md)
3. ✅ [Implementar upload via S3](../api/s3-integration.md)
---

