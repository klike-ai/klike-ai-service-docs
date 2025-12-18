#  Quick Start

Este guia rápido vai te ajudar a fazer sua primeira análise de vídeo em menos de 5 minutos!

---

##  Pré-requisitos

Antes de começar, certifique-se de ter:

- ✅ [Instalação completa](installation.md)
- ✅ [Variáveis de ambiente configuradas](configuration.md)
- ✅ Servidor rodando em `http://localhost:8000`

---

##  Seu Primeiro Vídeo

### Opção 1: Upload Direto (Recomendado para Teste)

#### 1. Prepare um vídeo de teste

Qualquer vídeo MP4 funciona! Recomendações:
- **Duração**: 15-60 segundos (ideal para análise)
- **Formato**: Vertical (9:16) ou quadrado (1:1)
- **Tamanho**: Até 100MB

#### 2. Faça o upload via console do bucket 

1. Acesse o console o serviço s3 da AWS em https://s3.console.aws.amazon.com/s3/
2. Selecione o bucket que você criou (ex: `klike-videos`)
3. Clique em **Upload** → **Add files** → Selecione seu vídeo
4. Clique em **Upload**
5. Aguarde o upload completar
6. Copie o nome do arquivo (ex: `meu_video.mp4`)
7. Acesse o Swagger UI: [http://localhost:8000/docs](http://localhost:8000/docs)
8. No Swagger UI:
    1. Expanda o endpoint `POST /video/analyze-from-s3`
    2. Clique em **Try it out**
    3. No campo `file_id`, cole o nome do arquivo (ex: `meu_video.mp4`)
    4. Preencha os outros campos:
        - `audience`: Ex: `["millennials"]`
        - `platform_type`: Ex: `["meta"]`
        - `language`: Ex: `pt`
    5. Clique em **Execute**

#### 3. Acompanhe o progresso

Você receberá um `task_id`:

```json
{
  "task_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "accepted",
  "message": "Análise iniciada...",
  "check_status_url": "/video/task/a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Copie o `task_id`!**

#### 4. Consulte o resultado

No Swagger UI:

1. Expanda `GET /video/task/{task_id}`
2. **Try it out** → Cole o `task_id`
3. **Execute**

Acompanhe o `progress`:

- `5%` - Upload para Gemini
- `30%` - Aguardando processamento
- `55%` - Processando legendas
- `85%` - Analisando com IA
- `100%` - Concluído! 

---


##  Entendendo o Resultado

### Score Final

O score vai de **0 a 100** e é dividido em stages:

| Score | Stage | Significado |
|-------|-------|-------------|
| 76-100 | **A** | 🟢 Excelente - Pronto para veicular |
| 51-75 | **B** | 🟡 Bom - Pequenos ajustes recomendados |
| 26-50 | **C** | 🟠 Regular - Precisa melhorias |
| 0-25 | **D** | 🔴 Crítico - Refazer criativo |

### Creative Metrics

Seu vídeo é avaliado em **9 métricas**:

```json
{
  "creative_metrics": [
    {
      "metric_name": "3 Second Hook Score",
      "score": 85,
      "stage": "A",
      "reasoning": {
        "what_went_well": [
          "Pergunta instigante nos primeiros 3 segundos",
          "Visual atraente que captura atenção"
        ],
        "room_for_improvement": [
          "Poderia ter um elemento surpresa maior"
        ]
      }
    }
  ]
}
```

### Detected Issues

Issues técnicas que impactam o score:

```json
{
  "detected_issues": {
    "safezone": true,
    "details_safezone": {
      "issue": "Elementos fora da safe zone",
      "impact": "Logo pode ser cortado pela UI",
      "recommendation": "Mover para área central 1080x1420px",
      "discount_applied": 10,
      "frames_example_urls": ["https://..."]
    }
  }
}
```

### Actions (Recomendações)

Ações específicas para melhorar:

```json
{
  "actions": {
    "actions_creatives": {
      "action_hook_score": [
        "Use padrão de interrupção nos 3 primeiros segundos",
        "Faça uma pergunta direta ao espectador"
      ]
    },
    "actions_issues": {
      "action_safezone": [
        "Mova logo para o centro superior do frame",
        "Evite colocar CTA próximo às bordas"
      ]
    },
    "actions_platforms": {
      "action_meta": [
        "Adicione legendas grandes e legíveis",
        "Use música trending do Meta Sound Collection"
      ]
    }
  }
}
```

---

##  Cenários Comuns

### Análise Rápida para Meta

```python
data = {
    "audience": ["millennials"],
    "platform_type": ["meta"],
    "language": "pt"
}
```

### Análise Multi-plataforma

```python
data = {
    "audience": ["genz", "millennials"],
    "platform_type": ["meta", "tiktok", "linkedin"],
    "language": "pt"
}
```

### Análise para LinkedIn B2B

```python
data = {
    "audience": ["genx", "boomer"],
    "platform_type": ["linkedin"],
    "language": "pt"
}
```

---

##  Workflow Recomendado

### Para Agências/Produtoras

```mermaid
graph TD
    A[Receber vídeo do cliente] --> B[Upload para Klike AI]
    B --> C[Análise automática]
    C --> D{Score > 75?}
    D -->|Sim| E[Aprovar para veiculação]
    D -->|Não| F[Implementar ações recomendadas]
    F --> G[Novo upload]
    G --> C
    E --> H[Deploy nas plataformas]
```

### Para Criadores de Conteúdo

```mermaid
graph TD
    A[Criar vídeo] --> B[Upload para Klike AI]
    B --> C[Receber feedback]
    C --> D[Implementar melhorias]
    D --> E{Satisfeito?}
    E -->|Não| B
    E -->|Sim| F[Publicar]
```

---

##  Problemas Comuns

### Upload muito lento

**Solução:** Use a [integração S3](../api/s3-integration.md) para uploads diretos:

```javascript
// Frontend faz upload direto para S3
const s3Upload = await uploadToS3(videoFile);

// Backend só processa
fetch('/video/analyze-from-s3', {
  body: JSON.stringify({ file_id: s3Upload.id })
});
```

### Task fica em "processing" por muito tempo

**Tempos esperados:**

- Vídeo de 30s: ~2-3 minutos
- Vídeo de 60s: ~4-5 minutos

Se ultrapassar 10 minutos:

1. Verifique o `task_id` no endpoint `/video/task/{task_id}`
2. Se `status: "failed"`, veja o campo `error`
3. Verifique os logs do servidor

### Erro: "GOOGLE_API_KEY invalid"

Verifique o `.env`:
```bash
# Deve começar com AIzaSy...
GOOGLE_API_KEY=AIzaSyBxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Teste a chave:
```python
from google import genai
client = genai.Client(api_key="sua_chave")
print(client.models.list())  # Deve listar modelos
```


---

## Dicas Pro

### Batch Processing

Analise múltiplos vídeos em paralelo:

```python
import concurrent.futures

videos = ["video1.mp4", "video2.mp4", "video3.mp4"]
task_ids = []

# Upload em paralelo
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    futures = [
        executor.submit(upload_video, video) 
        for video in videos
    ]
    task_ids = [f.result() for f in futures]

print(f" {len(task_ids)} vídeos em análise!")
```

### Webhook (Futuro)

Em breve você poderá configurar webhooks para notificações:

```python
data = {
    "file": video_file,
    "webhook_url": "https://seu-app.com/webhook",
    "audience": ["millennials"]
}
```

---

** Parabéns! Você completou o Quick Start!**

Agora explore a [documentação completa](../api/overview.md) para recursos avançados.
