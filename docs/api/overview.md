#  API Reference - Visão Geral

Documentação completa dos endpoints da Klike AI Services API.

---

##  Base URL

```
http://localhost:8000
```

**Produção:** `https://api.klike.ai` (quando disponível)

---

##  Autenticação

Atualmente a API não requer autenticação para desenvolvimento. Em produção, será necessário:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     "https://api.klike.ai/video/task/123"
```

---

## Endpoints Disponíveis

### Análise de Vídeos

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/video/analyze-from-s3` | Análise de vídeo já no S3 |
| `GET` | `/video/task/{task_id}` | Consultar status da tarefa |
| `GET` | `/video/tasks` | Listar todas as tarefas |

### S3 Integration

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `GET` | `/video/list-s3-videos` | Listar vídeos disponíveis no S3 |
| `GET` | `/video/check-s3-video/{file_id}` | Verificar se vídeo existe no S3 |

### Calculadora

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/calculate-budget` | Calcular budget baseado em meta de receita |

---

##  Swagger UI Interativa

Acesse a documentação interativa em:

**[http://localhost:8000/docs](http://localhost:8000/docs)**

Lá você pode:

- ✅ Testar todos os endpoints
- ✅ Ver schemas de request/response
- ✅ Copiar exemplos de código

---

##  Modelos de Dados

### VideoAnalysisRequest

```python
{
  "file": "video.mp4",           # Arquivo MP4
  "audience": ["millennials"],   # Lista de audiências
  "platform_type": ["meta"],     # Lista de plataformas
  "language": "pt"               # pt ou en
}
```

### TaskResponse

```python
{
  "task_id": "uuid",
  "status": "completed",          # pending, processing, completed, failed
  "progress": 100,                # 0-100
  "created_at": "2025-12-08T...",
  "updated_at": "2025-12-08T...",
  "file_name": "s3://video.mp4",
  "result": {                     # Só quando completed
    "score": 75,
    "stage": "B",
    "analysis": {...},
    "actions": {...}
  }
}
```

### VideoAnalysisResult

```python
{
  "score": 75,
  "stage": "B",
  "analysis": {
    "creative_metrics": [
      {
        "metric_name": "3 Second Hook Score",
        "score": 85,
        "stage": "A",
        "reasoning": {
          "what_went_well": ["..."],
          "room_for_improvement": ["..."]
        }
      }
    ],
    "audience_scores": {
      "genz": "Geração Z",
      "score_genz": 55,
      "reasoning_genz": "..."
    },
    "platform_recommendations": [
      {
        "platform": "meta",
        "recommendations": ["..."]
      }
    ],
    "detected_issues": {
      "safezone": true,
      "details_safezone": {...}
    }
  },
  "actions": {
    "actions_creatives": {...},
    "actions_issues": {...},
    "actions_platforms": {...},
    "actions_audiences": {...}
  },
  "warning": {
    "compliance_status": "compliance",
    "issues_found": []
  }
}
```

---

##  Tipos de Audiência

| Valor | Descrição | Características |
|-------|-----------|-----------------|
| `genz` | Geração Z (1997-2012) | Autenticidade, trends, UGC |
| `millennials` | Millennials (1981-1996) | Propósito, storytelling |
| `genx` | Geração X (1965-1980) | Pragmatismo, valor claro |
| `boomer` | Boomers (1946-1964) | Confiança, simplicidade |

---

##  Tipos de Plataforma

| Valor | Plataforma | Formato Ideal |
|-------|------------|---------------|
| `meta` | Facebook/Instagram | 9:16, 1:1, 4:5 |
| `tiktok` | TikTok | 9:16 |
| `linkedin` | LinkedIn | 16:9, 1:1 |

---

##  Idiomas Suportados

| Código | Idioma |
|--------|--------|
| `pt` | Português |
| `en` | Inglês |

---


##  Códigos de Erro

| Código | Descrição | Solução |
|--------|-----------|---------|
| `400` | Bad Request | Verifique os parâmetros |
| `404` | Not Found | Task ID inválido |
| `413` | Payload Too Large | Vídeo > 100MB |
| `422` | Unprocessable Entity | Formato inválido |
| `500` | Internal Server Error | Veja logs do servidor |

### Exemplo de Erro

```json
{
  "detail": "Vídeo muito grande. Máximo: 100MB"
}
```

---

## Guias Detalhados

Explore a documentação específica de cada endpoint:

   [Análise de Vídeos](video-analysis.md)
- [Integração S3](s3-integration.md)
- [Background Tasks](background-tasks.md)
- [Calculadora de Budget](calculator.md)

---

## Exemplos de Integração

### JavaScript/React

```javascript
const analyzeVideo = async (videoFile) => {
  const formData = new FormData();
  formData.append('file', videoFile);
  formData.append('audience', 'millennials');
  formData.append('platform_type', 'meta');
  formData.append('language', 'pt');

  const response = await fetch('http://localhost:8000/video/upload-video-async', {
    method: 'POST',
    body: formData
  });

  const { task_id } = await response.json();
  
  // Polling
  const checkStatus = async () => {
    const res = await fetch(`http://localhost:8000/video/task/${task_id}`);
    const data = await res.json();
    
    if (data.status === 'completed') {
      return data.result;
    } else if (data.status === 'failed') {
      throw new Error(data.error);
    }
    
    await new Promise(r => setTimeout(r, 2000));
    return checkStatus();
  };

  return await checkStatus();
};
```

### Python

```python
import requests
import time

def analyze_video(video_path, audience, platform):
    # Upload
    with open(video_path, 'rb') as f:
        response = requests.post(
            'http://localhost:8000/video/upload-video-async',
            files={'file': f},
            data={
                'audience': audience,
                'platform_type': platform,
                'language': 'pt'
            }
        )
    
    task_id = response.json()['task_id']
    
    # Polling
    while True:
        response = requests.get(
            f'http://localhost:8000/video/task/{task_id}'
        )
        data = response.json()
        
        if data['status'] == 'completed':
            return data['result']
        elif data['status'] == 'failed':
            raise Exception(data['error'])
        
        time.sleep(2)
```

